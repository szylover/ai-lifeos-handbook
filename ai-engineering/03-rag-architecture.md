[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# RAG 架构入门

检索增强生成（Retrieval-Augmented Generation, RAG）把「外部知识检索」与「大模型生成」结合，
让模型在回答时引用最新、私有或领域特定的资料，而不是仅依赖训练时的参数记忆。

本章是一份可以照着做的 RAG 工程指南：从空目录开始完成文档加载、chunking、embeddings、向量库、检索、混合检索、重排序、prompt 拼装、带引用生成与检索评估。

## 0. 环境准备

```powershell
cd D:\projects\ai-lifeos-handbook
python -m venv .venv-rag
.\.venv-rag\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install sentence-transformers faiss-cpu rank-bm25 tiktoken numpy pydantic openai python-dotenv
python -c "import faiss, sentence_transformers, rank_bm25; print('ok')"
```

预期输出：`ok`。

## 1. RAG 全流程 pipeline

```text
文档加载 → Document(text, source, metadata)
切分 chunking → Chunk(id, text, source, metadata)
嵌入 embeddings → Vector(id, embedding, metadata)
向量库 FAISS / Chroma / pgvector
检索 retrieval top-k → 混合检索 BM25 + vector
重排序 cross-encoder / Cohere rerank
prompt assembly：编号、截断、去重、引用
generation：只基于 context 回答 → answer + citations
```

```ts
async function rag(query: string) {
  const qVec = await embed(query);
  const chunks = await vectorStore.search(qVec, { topK: 5 });
  const context = chunks.map((c) => c.text).join("\n---\n");
  return llm.complete(`基于以下资料回答：\n${context}\n\n问题：${query}`);
}
```

余弦相似度：

$$
\text{sim}(\mathbf{q}, \mathbf{v}) = \frac{\mathbf{q} \cdot \mathbf{v}}{\lVert \mathbf{q} \rVert \, \lVert \mathbf{v} \rVert}
$$

若 embedding 已归一化，FAISS inner product 等价于 cosine similarity。

## 2. 文档加载 Document Loading

先创建样例文档：

```powershell
New-Item -ItemType Directory -Force docs | Out-Null
"# 退款政策`n`n用户在购买后 30 天内可以申请退款。企业合同以订单条款为准。`n`n## 例外`n`n滥用账号、违反服务条款或已经消耗的一次性服务不支持退款。" | Set-Content -Encoding UTF8 docs\refund.md
"# 安全政策`n`n所有 API key 必须通过环境变量配置，不得写入代码仓库。`n生产环境需要开启审计日志，日志至少保留 180 天。" | Set-Content -Encoding UTF8 docs\security.md
```

最小 loader：

```python
# loader.py
from pathlib import Path
from pydantic import BaseModel

class Document(BaseModel):
    text: str
    source: str
    metadata: dict = {}

def normalize_text(text: str) -> str:
    text = text.replace("\r\n", "\n").replace("\r", "\n")
    return "\n".join(line.rstrip() for line in text.split("\n"))

def load_text_docs(root: str) -> list[Document]:
    docs: list[Document] = []
    for path in Path(root).rglob("*"):
        if path.suffix.lower() not in {".md", ".txt"}:
            continue
        text = normalize_text(path.read_text(encoding="utf-8", errors="ignore"))
        if len(text.strip()) < 20:
            continue
        docs.append(Document(text=text, source=str(path), metadata={"filename": path.name}))
    return docs

if __name__ == "__main__":
    docs = load_text_docs("docs")
    print(f"loaded {len(docs)} documents")
    print(docs[0].source if docs else "no docs")
```

运行 `python loader.py`，预期输出：

```text
loaded 2 documents
docs\refund.md
```

Loader 完成标准：

| 要求 | 做法 | 不满足时的症状 |
| --- | --- | --- |
| 保留来源 | metadata 保存文件名、URL、页码、标题 | 答案无法引用 |
| 清洗噪声 | 去导航、页眉、页脚、重复版权 | top-k 命中模板文本 |
| 保留结构 | Markdown 标题、列表、表格尽量保留 | 模型丢失章节语义 |
| 可增量更新 | 保存 `doc_id`、`mtime`、`checksum` | 每次全量重建索引 |

## 3. 切分 Chunking

切分目标是在召回率与上下文完整性之间取平衡。

| 文档类型 | chunk size | overlap | 说明 |
| --- | ---: | ---: | --- |
| FAQ / 短知识条 | 150–300 tokens | 20–50 tokens | 一问一答完整保留 |
| 技术文档 / 手册 | 400–700 tokens | 80–120 tokens | 保留步骤与代码前后文 |
| 法务 / 政策 | 300–500 tokens | 80–150 tokens | 避免条款边界断裂 |
| 代码文档 | 按函数/类 + 200–500 tokens | 50–100 tokens | 优先结构边界 |
| 长 PDF / 论文 | 600–900 tokens | 100–150 tokens | 章节信息更重要 |

通用起点：`chunk_size=512`，`overlap=min(128, chunk_size*0.15)`，`top_k=5`。

```python
# chunking.py
from dataclasses import dataclass
import tiktoken
from loader import Document

enc = tiktoken.get_encoding("cl100k_base")

@dataclass
class Chunk:
    id: str
    text: str
    source: str
    metadata: dict

def token_len(text: str) -> int:
    return len(enc.encode(text))

def split_by_tokens(docs: list[Document], chunk_size: int = 512, overlap: int = 80) -> list[Chunk]:
    assert 0 <= overlap < chunk_size
    chunks: list[Chunk] = []
    for doc_idx, doc in enumerate(docs):
        tokens = enc.encode(doc.text)
        start = 0
        part = 0
        while start < len(tokens):
            end = min(start + chunk_size, len(tokens))
            text = enc.decode(tokens[start:end]).strip()
            if text:
                chunks.append(Chunk(f"doc{doc_idx}-chunk{part}", text, doc.source, {**doc.metadata, "chunk_index": part}))
            if end == len(tokens):
                break
            start = end - overlap
            part += 1
    return chunks

if __name__ == "__main__":
    from loader import load_text_docs
    chunks = split_by_tokens(load_text_docs("docs"), chunk_size=120, overlap=20)
    for c in chunks:
        print(c.id, token_len(c.text), c.source)
```

结构优先 Markdown 切分：

```python
def split_markdown_sections(text: str) -> list[tuple[str, str]]:
    sections, buf = [], []
    title = "正文"
    for line in text.splitlines():
        if line.startswith("#"):
            if buf:
                sections.append((title, "\n".join(buf).strip()))
            title = line.lstrip("#").strip() or "正文"
            buf = [line]
        else:
            buf.append(line)
    if buf:
        sections.append((title, "\n".join(buf).strip()))
    return [(t, b) for t, b in sections if b]
```

| 策略 | 优点 | 缺点 | 适用 |
| --- | --- | --- | --- |
| 固定 token 窗口 | 简单稳定 | 可能切断表格/步骤 | MVP |
| 标题/段落优先 | 保留语义边界 | 长 section 仍需二次切 | Markdown / HTML |
| 递归分隔符 | 先段落、再句子、再 token | 实现稍复杂 | 多格式文本 |
| 语义切分 | 根据 embedding 相似度找断点 | 成本高、调参多 | 会议纪要、访谈 |

## 4. 嵌入 Embeddings

Embedding 模型把文本映射成向量 $\mathbf{v} \in \mathbb{R}^d$。文档入库和 query 检索必须使用同一 embedding 空间。

| 模型 | 维度 | 运行方式 | 成本 | 适合场景 |
| --- | ---: | --- | --- | --- |
| `sentence-transformers/all-MiniLM-L6-v2` | 384 | 本地 CPU | 免费 | 英文、小 demo |
| `BAAI/bge-small-zh-v1.5` | 512 | 本地 CPU/GPU | 免费 | 中文知识库入门 |
| `BAAI/bge-m3` | 1024 | 本地 GPU 更佳 | 免费 | 多语言、长文本 |
| `text-embedding-3-small` | 1536 | OpenAI API | 低 | 通用 SaaS |
| `text-embedding-3-large` | 3072 | OpenAI API | 较高 | 质量优先 |

```python
# embeddings.py
from sentence_transformers import SentenceTransformer
import numpy as np

class LocalEmbedder:
    def __init__(self, model_name: str = "BAAI/bge-small-zh-v1.5"):
        self.model_name = model_name
        self.model = SentenceTransformer(model_name)

    def embed_documents(self, texts: list[str]) -> np.ndarray:
        vectors = self.model.encode(texts, normalize_embeddings=True, batch_size=32, show_progress_bar=False)
        return np.asarray(vectors, dtype="float32")

    def embed_query(self, query: str) -> np.ndarray:
        q = f"为这个句子生成表示以用于检索相关文章：{query}"
        return self.embed_documents([q])[0]

if __name__ == "__main__":
    emb = LocalEmbedder()
    vectors = emb.embed_documents(["退款政策", "API key 安全"])
    print(vectors.shape)
    print(round(float(np.linalg.norm(vectors[0])), 3))
```

运行 `python embeddings.py`，预期输出：

```text
(2, 512)
1.0
```

OpenAI embedding 版本：

```python
# openai_embeddings.py
from openai import OpenAI
import numpy as np
client = OpenAI()

def embed_openai(texts: list[str], model: str = "text-embedding-3-small") -> np.ndarray:
    resp = client.embeddings.create(model=model, input=texts)
    return np.asarray([item.embedding for item in resp.data], dtype="float32")
```

成本估算：`成本 = 总 token / 1,000,000 * 每百万 token 单价`。向量库 metadata 应记录 `embedding_model` 与 `embedding_dim`；模型变更后重建索引。

## 5. 向量库 Vector Store

### 5.1 FAISS 本地索引

FAISS 适合本地 demo、离线批处理、单机服务。因为前面 embedding 已 normalize，`IndexFlatIP` 可当 cosine search 使用。

```python
# vector_faiss.py
import json
from pathlib import Path
import faiss
import numpy as np
from chunking import Chunk

class FaissStore:
    def __init__(self, dim: int):
        self.index = faiss.IndexFlatIP(dim)
        self.chunks: list[Chunk] = []

    def add(self, vectors: np.ndarray, chunks: list[Chunk]) -> None:
        assert vectors.dtype == np.float32
        assert vectors.shape[0] == len(chunks)
        self.index.add(vectors)
        self.chunks.extend(chunks)

    def search(self, query_vector: np.ndarray, top_k: int = 5) -> list[dict]:
        q = np.asarray([query_vector], dtype="float32")
        scores, idxs = self.index.search(q, top_k)
        out = []
        for score, idx in zip(scores[0], idxs[0]):
            if idx != -1:
                out.append({"score": float(score), "chunk": self.chunks[int(idx)]})
        return out

    def save(self, path: str) -> None:
        root = Path(path)
        root.mkdir(parents=True, exist_ok=True)
        faiss.write_index(self.index, str(root / "index.faiss"))
        payload = [c.__dict__ for c in self.chunks]
        (root / "chunks.json").write_text(json.dumps(payload, ensure_ascii=False, indent=2), encoding="utf-8")

    @classmethod
    def load(cls, path: str):
        root = Path(path)
        index = faiss.read_index(str(root / "index.faiss"))
        payload = json.loads((root / "chunks.json").read_text(encoding="utf-8"))
        store = cls(index.d)
        store.index = index
        store.chunks = [Chunk(**item) for item in payload]
        return store
```

构建索引：

```python
# build_index.py
from loader import load_text_docs
from chunking import split_by_tokens
from embeddings import LocalEmbedder
from vector_faiss import FaissStore

docs = load_text_docs("docs")
chunks = split_by_tokens(docs, chunk_size=256, overlap=40)
embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
vectors = embedder.embed_documents([c.text for c in chunks])
store = FaissStore(dim=vectors.shape[1])
store.add(vectors, chunks)
store.save("rag_index")
print(f"indexed docs={len(docs)} chunks={len(chunks)} dim={vectors.shape[1]}")
```

运行 `python build_index.py`，预期：`indexed docs=2 chunks=2 dim=512`。

### 5.2 Chroma 本地持久化

```powershell
pip install chromadb
```

```python
# chroma_store_example.py
import chromadb
from embeddings import LocalEmbedder
from loader import load_text_docs
from chunking import split_by_tokens

client = chromadb.PersistentClient(path="chroma_db")
collection = client.get_or_create_collection(name="handbook")
embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
chunks = split_by_tokens(load_text_docs("docs"), chunk_size=256, overlap=40)
vectors = embedder.embed_documents([c.text for c in chunks]).tolist()
collection.add(
    ids=[c.id for c in chunks],
    documents=[c.text for c in chunks],
    embeddings=vectors,
    metadatas=[{"source": c.source, **c.metadata} for c in chunks],
)
q = embedder.embed_query("API key 应该如何保存？").tolist()
result = collection.query(query_embeddings=[q], n_results=3)
print(result["documents"][0][0])
print(result["metadatas"][0][0])
```

### 5.3 pgvector 生产示例

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE rag_chunks (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL,
  source TEXT NOT NULL,
  chunk_index INT NOT NULL,
  content TEXT NOT NULL,
  embedding vector(1536),
  metadata JSONB DEFAULT '{}'
);
CREATE INDEX rag_chunks_embedding_idx
ON rag_chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
SELECT id, source, content, 1 - (embedding <=> $1) AS score
FROM rag_chunks
WHERE tenant_id = $2
ORDER BY embedding <=> $1
LIMIT 8;
```

10 万 chunk 以下可先用 Postgres + pgvector；百万级以上评估 Qdrant、Weaviate、Milvus、Pinecone。多租户系统必须在检索阶段用 `tenant_id` 过滤。

## 6. 检索 Retrieval

```python
# retrieve.py
from embeddings import LocalEmbedder
from vector_faiss import FaissStore

def search(query: str, top_k: int = 5):
    embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
    store = FaissStore.load("rag_index")
    results = store.search(embedder.embed_query(query), top_k=top_k)
    for i, r in enumerate(results, start=1):
        c = r["chunk"]
        print(f"[{i}] score={r['score']:.3f} source={c.source} chunk={c.id}")
        print(c.text[:180].replace("\n", " "))

if __name__ == "__main__":
    search("退款窗口是多少天？", top_k=3)
```

`top_k=5` 是 MVP 起点；复杂政策和技术文档可先取 8–12；如果要 rerank，粗召回可取 20，再精排 5。相似度默认用 cosine；如果用 `IndexFlatIP`，必须确保向量已 normalize。

## 7. 混合检索 BM25 + Vector

向量检索擅长语义相似，BM25 擅长关键词、错误码、SKU、函数名。混合检索做法：分别检索，归一化分数，再加权融合。

```python
# hybrid_retrieval.py
from rank_bm25 import BM25Okapi
from loader import load_text_docs
from chunking import split_by_tokens
from embeddings import LocalEmbedder
from vector_faiss import FaissStore

def tokenize(text: str) -> list[str]:
    chars = [c for c in text.lower() if "\u4e00" <= c <= "\u9fff"]
    words = [w for w in text.lower().replace("/", " ").replace("-", " ").split() if w]
    bigrams = ["".join(chars[i:i+2]) for i in range(len(chars)-1)]
    return words + chars + bigrams

def minmax(scores: list[float]) -> list[float]:
    lo, hi = min(scores), max(scores)
    return [0.0 for _ in scores] if hi == lo else [(s - lo) / (hi - lo) for s in scores]

def hybrid_search(query: str, top_k: int = 5, alpha: float = 0.65):
    chunks = split_by_tokens(load_text_docs("docs"), chunk_size=256, overlap=40)
    texts = [c.text for c in chunks]
    bm25 = BM25Okapi([tokenize(t) for t in texts])
    bm25_scores = bm25.get_scores(tokenize(query)).tolist()

    embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
    vectors = embedder.embed_documents(texts)
    store = FaissStore(dim=vectors.shape[1])
    store.add(vectors, chunks)
    dense_results = store.search(embedder.embed_query(query), top_k=len(chunks))
    dense_by_id = {r["chunk"].id: r["score"] for r in dense_results}
    dense_scores = [dense_by_id.get(c.id, 0.0) for c in chunks]

    dense_norm, bm25_norm = minmax(dense_scores), minmax(bm25_scores)
    fused = []
    for i, c in enumerate(chunks):
        score = alpha * dense_norm[i] + (1 - alpha) * bm25_norm[i]
        fused.append((score, dense_norm[i], bm25_norm[i], c))
    fused.sort(key=lambda x: x[0], reverse=True)
    return fused[:top_k]

if __name__ == "__main__":
    for score, dense, bm25, c in hybrid_search("API key 环境变量", top_k=3):
        print(f"score={score:.3f} dense={dense:.3f} bm25={bm25:.3f} source={c.source}")
        print(c.text[:160].replace("\n", " "))
```

调参：`alpha=0.65` 通用；错误码/API 名多时把 BM25 权重提高到 0.4–0.6。也可用 Reciprocal Rank Fusion：

```python
def rrf(rank: int, k: int = 60) -> float:
    return 1.0 / (k + rank)
```

RRF 不依赖不同检索器分数可比性，适合组合多个检索器。

## 8. 重排序 Reranking

第一阶段 retrieval 追求「别漏」，rerank 追求「排前」。典型链路：`vector/BM25 top 20 → cross-encoder 逐对打分 → top 5 放进 prompt`。

```python
# rerank.py
from sentence_transformers import CrossEncoder

class CrossEncoderReranker:
    def __init__(self, model_name: str = "BAAI/bge-reranker-base"):
        self.model = CrossEncoder(model_name)

    def rerank(self, query: str, candidates: list[dict], top_n: int = 5) -> list[dict]:
        pairs = [(query, item["chunk"].text) for item in candidates]
        scores = self.model.predict(pairs)
        enriched = []
        for item, score in zip(candidates, scores):
            enriched.append({**item, "rerank_score": float(score)})
        enriched.sort(key=lambda x: x["rerank_score"], reverse=True)
        return enriched[:top_n]
```

集成：

```python
from embeddings import LocalEmbedder
from vector_faiss import FaissStore
from rerank import CrossEncoderReranker

embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
store = FaissStore.load("rag_index")
query = "退款窗口是多少天？"
candidates = store.search(embedder.embed_query(query), top_k=20)
final_chunks = CrossEncoderReranker("BAAI/bge-reranker-base").rerank(query, candidates, top_n=5)
```

Cohere rerank：

```powershell
pip install cohere
$env:COHERE_API_KEY="..."
```

```python
import os
import cohere
co = cohere.Client(os.environ["COHERE_API_KEY"])

def cohere_rerank(query: str, candidates: list[dict], top_n: int = 5):
    docs = [c["chunk"].text for c in candidates]
    resp = co.rerank(model="rerank-multilingual-v3.0", query=query, documents=docs, top_n=top_n)
    return [{**candidates[r.index], "rerank_score": r.relevance_score} for r in resp.results]
```

## 9. Grounded generation：prompt、引用与拒答

Prompt 模板：

```text
你是一个严谨的问答助手。只能基于 <context> 中的资料回答。
规则：
1. 如果资料不足，回答「根据已提供资料无法确定」，不要编造。
2. 每个关键结论后面必须带引用，格式如 [1] 或 [2]。
3. 引用编号必须来自 <context> 的 chunk 编号。
4. 不要引用没有支持该结论的 chunk。
<context>
[1] source=docs\refund.md chunk=doc0-chunk0
# 退款政策
用户在购买后 30 天内可以申请退款...
</context>
问题：退款窗口是多少天？
```

组装代码：

```python
# prompt_assembly.py
def build_context(results: list[dict], max_chars: int = 6000) -> tuple[str, list[dict]]:
    blocks, citations, used = [], [], 0
    for idx, item in enumerate(results, start=1):
        chunk = item["chunk"]
        block = f"[{idx}] source={chunk.source} chunk={chunk.id}\n{chunk.text.strip()}"
        if used + len(block) > max_chars:
            break
        blocks.append(block)
        citations.append({"n": idx, "source": chunk.source, "chunk_id": chunk.id})
        used += len(block)
    return "\n\n".join(blocks), citations

def build_prompt(question: str, results: list[dict]) -> tuple[str, list[dict]]:
    context, citations = build_context(results)
    prompt = f"""你是一个严谨的问答助手。只能基于 <context> 中的资料回答。
如果资料不足，回答「根据已提供资料无法确定」。每个关键结论后面必须带引用，如 [1]。
<context>
{context}
</context>
问题：{question}
"""
    return prompt, citations
```

OpenAI 生成：

```python
# generate_openai.py
from openai import OpenAI
client = OpenAI()

def generate_answer(prompt: str) -> str:
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0,
    )
    return resp.choices[0].message.content
```

无 API key 的 mock generator：

```python
# generate_mock.py
def generate_answer(prompt: str) -> str:
    if "30 天" in prompt and "退款" in prompt:
        return "用户在购买后 30 天内可以申请退款；企业合同以订单条款为准。[1]"
    if "API key" in prompt and "环境变量" in prompt:
        return "API key 必须通过环境变量配置，不得写入代码仓库。[1]"
    return "根据已提供资料无法确定。"
```

生成阶段规则：`temperature=0` 起步；context 每个 chunk 编号唯一；资料不足时拒答；输出后校验引用编号存在于 `citations`。

## 10. 完整最小端到端 RAG 脚本

复制为 `mini_rag.py`，它会自动创建样例文档、构建索引、混合检索、rerank、生成带引用答案。

```python
# mini_rag.py
from pathlib import Path
from dataclasses import dataclass
import re
import numpy as np
import faiss
import tiktoken
from sentence_transformers import SentenceTransformer, CrossEncoder
from rank_bm25 import BM25Okapi

DOC_DIR = Path("docs")
EMBED_MODEL = "BAAI/bge-small-zh-v1.5"
RERANK_MODEL = "BAAI/bge-reranker-base"

@dataclass
class Document:
    text: str
    source: str
    metadata: dict

@dataclass
class Chunk:
    id: str
    text: str
    source: str
    metadata: dict

def ensure_sample_docs():
    DOC_DIR.mkdir(exist_ok=True)
    (DOC_DIR / "refund.md").write_text(
        "# 退款政策\n\n用户在购买后 30 天内可以申请退款。企业合同以订单条款为准。\n\n"
        "## 例外\n\n滥用账号、违反服务条款或已经消耗的一次性服务不支持退款。\n",
        encoding="utf-8",
    )
    (DOC_DIR / "security.md").write_text(
        "# 安全政策\n\n所有 API key 必须通过环境变量配置，不得写入代码仓库。\n"
        "生产环境需要开启审计日志，日志至少保留 180 天。\n",
        encoding="utf-8",
    )

def load_docs(root: Path) -> list[Document]:
    docs = []
    for path in root.rglob("*"):
        if path.suffix.lower() in {".md", ".txt"}:
            text = path.read_text(encoding="utf-8", errors="ignore").replace("\r\n", "\n")
            docs.append(Document(text=text, source=str(path), metadata={"filename": path.name}))
    return docs

enc = tiktoken.get_encoding("cl100k_base")

def split_docs(docs: list[Document], chunk_size: int = 256, overlap: int = 40) -> list[Chunk]:
    chunks = []
    for doc_idx, doc in enumerate(docs):
        tokens = enc.encode(doc.text)
        start, part = 0, 0
        while start < len(tokens):
            end = min(start + chunk_size, len(tokens))
            text = enc.decode(tokens[start:end]).strip()
            chunks.append(Chunk(f"doc{doc_idx}-chunk{part}", text, doc.source, {**doc.metadata, "chunk_index": part}))
            if end == len(tokens):
                break
            start = end - overlap
            part += 1
    return chunks

class Embedder:
    def __init__(self, model_name: str = EMBED_MODEL):
        self.model = SentenceTransformer(model_name)

    def docs(self, texts: list[str]) -> np.ndarray:
        return np.asarray(self.model.encode(texts, normalize_embeddings=True), dtype="float32")

    def query(self, text: str) -> np.ndarray:
        return self.docs([f"为这个句子生成表示以用于检索相关文章：{text}"])[0]

def tokenize(text: str) -> list[str]:
    chars = [c for c in text.lower() if "\u4e00" <= c <= "\u9fff"]
    words = re.findall(r"[a-zA-Z0-9_\-]+", text.lower())
    bigrams = ["".join(chars[i:i+2]) for i in range(len(chars)-1)]
    return words + chars + bigrams

def normalize(scores: list[float]) -> list[float]:
    lo, hi = min(scores), max(scores)
    return [0.0] * len(scores) if hi == lo else [(s - lo) / (hi - lo) for s in scores]

def retrieve(query: str, chunks: list[Chunk], vectors: np.ndarray, top_k: int = 10):
    embedder = Embedder()
    index = faiss.IndexFlatIP(vectors.shape[1])
    index.add(vectors)
    dense_scores, dense_idxs = index.search(np.asarray([embedder.query(query)], dtype="float32"), len(chunks))
    dense_by_idx = {int(i): float(s) for s, i in zip(dense_scores[0], dense_idxs[0]) if i != -1}
    dense_norm = normalize([dense_by_idx.get(i, 0.0) for i in range(len(chunks))])
    bm25 = BM25Okapi([tokenize(c.text) for c in chunks])
    bm25_norm = normalize(bm25.get_scores(tokenize(query)).tolist())
    fused = []
    for i, c in enumerate(chunks):
        fused.append({"score": 0.65 * dense_norm[i] + 0.35 * bm25_norm[i], "chunk": c})
    return sorted(fused, key=lambda x: x["score"], reverse=True)[:top_k]

def rerank(query: str, candidates: list[dict], top_n: int = 3):
    model = CrossEncoder(RERANK_MODEL)
    scores = model.predict([(query, item["chunk"].text) for item in candidates])
    for item, score in zip(candidates, scores):
        item["rerank_score"] = float(score)
    return sorted(candidates, key=lambda x: x["rerank_score"], reverse=True)[:top_n]

def build_prompt(question: str, results: list[dict]) -> tuple[str, list[dict]]:
    blocks, citations = [], []
    for n, item in enumerate(results, start=1):
        c = item["chunk"]
        blocks.append(f"[{n}] source={c.source} chunk={c.id}\n{c.text}")
        citations.append({"n": n, "source": c.source, "chunk_id": c.id})
    context = "\n\n".join(blocks)
    return f"""你是一个严谨的问答助手。只能基于 <context> 中的资料回答。
如果资料不足，回答「根据已提供资料无法确定」。每个关键结论后面必须带引用，如 [1]。
<context>
{context}
</context>
问题：{question}
""", citations

def mock_generate(prompt: str) -> str:
    if "退款" in prompt and "30 天" in prompt:
        return "用户在购买后 30 天内可以申请退款；企业合同以订单条款为准。[1]"
    if "API key" in prompt and "环境变量" in prompt:
        return "API key 必须通过环境变量配置，不得写入代码仓库。[1]"
    return "根据已提供资料无法确定。"

def main():
    ensure_sample_docs()
    docs = load_docs(DOC_DIR)
    chunks = split_docs(docs)
    embedder = Embedder()
    vectors = embedder.docs([c.text for c in chunks])
    question = "退款窗口是多少天？"
    candidates = retrieve(question, chunks, vectors, top_k=10)
    final = rerank(question, candidates, top_n=3)
    prompt, citations = build_prompt(question, final)
    print(f"docs={len(docs)} chunks={len(chunks)} dim={vectors.shape[1]}")
    print("question:", question)
    print("answer:", mock_generate(prompt))
    print("citations:", citations)

if __name__ == "__main__":
    main()
```

运行：

```powershell
python mini_rag.py
```

预期输出：

```text
docs=2 chunks=2 dim=512
question: 退款窗口是多少天？
answer: 用户在购买后 30 天内可以申请退款；企业合同以订单条款为准。[1]
citations: [{'n': 1, 'source': 'docs\\refund.md', 'chunk_id': 'doc0-chunk0'}, ...]
```

如果只想先验证链路，可把 `final = rerank(...)` 改成 `final = candidates[:3]`，跳过 rerank 模型下载。

## 11. 评估检索质量：recall@k 与 MRR

生成质量难直接归因，先评估 retrieval。最小 eval set：

```csv
question,expected_chunk_id,notes
退款窗口是多少天？,doc0-chunk0,应该命中 refund 政策
API key 应该如何保存？,doc1-chunk0,应该命中 security 政策
审计日志保留多久？,doc1-chunk0,应该命中 security 政策
```

$$
\text{Recall@k} = \frac{\#\{q: rank(q) \le k\}}{\#\{q\}}
$$

$$
\text{MRR} = \frac{1}{N}\sum_{i=1}^{N}\frac{1}{rank_i}
$$

可运行脚本：

```python
# eval_retrieval.py
import csv
from pathlib import Path
from loader import load_text_docs
from chunking import split_by_tokens
from embeddings import LocalEmbedder
from vector_faiss import FaissStore

EVAL_FILE = Path("eval_questions.csv")

def ensure_eval_file():
    if EVAL_FILE.exists():
        return
    EVAL_FILE.write_text(
        "question,expected_chunk_id,notes\n"
        "退款窗口是多少天？,doc0-chunk0,refund\n"
        "API key 应该如何保存？,doc1-chunk0,security\n"
        "审计日志保留多久？,doc1-chunk0,security\n",
        encoding="utf-8",
    )

def rank_of(expected: str, results: list[dict]) -> int | None:
    for i, r in enumerate(results, start=1):
        if r["chunk"].id == expected:
            return i
    return None

def main():
    ensure_eval_file()
    chunks = split_by_tokens(load_text_docs("docs"), chunk_size=256, overlap=40)
    embedder = LocalEmbedder("BAAI/bge-small-zh-v1.5")
    vectors = embedder.embed_documents([c.text for c in chunks])
    store = FaissStore(dim=vectors.shape[1])
    store.add(vectors, chunks)
    rows = list(csv.DictReader(EVAL_FILE.open(encoding="utf-8")))
    hits_at_3, reciprocal_ranks = 0, []
    for row in rows:
        results = store.search(embedder.embed_query(row["question"]), top_k=3)
        rank = rank_of(row["expected_chunk_id"], results)
        hits_at_3 += int(rank is not None)
        reciprocal_ranks.append(0 if rank is None else 1 / rank)
        print(row["question"], "expected=", row["expected_chunk_id"], "rank=", rank)
    print(f"recall@3={hits_at_3 / len(rows):.3f} mrr={sum(reciprocal_ranks) / len(rows):.3f}")

if __name__ == "__main__":
    main()
```

运行 `python eval_retrieval.py`。真实项目建议从搜索日志、客服工单、销售问答抽 50–200 个真实问题；每题人工标注 1–3 个支持答案的 chunk；按定义查询、流程查询、故障排查、政策边界、代码/API 分桶。每次修改 chunk size、embedding 模型、top-k、rerank 参数后跑同一份 eval，并记录 latency、token cost、拒答率、用户反馈。更完整的 LLM 应用评估方法见 [LLM 应用评估](05-evals-for-llm-apps.md)。

## 12. 生产化架构

离线索引流水线：

```text
source connector → parse / clean → doc_id + checksum + mtime
→ chunking → embedding batch job → vector upsert
→ eval smoke test → publish index version
```

在线查询链路：

```text
user question → auth / tenant filter → query rewrite（可选）
→ hybrid retrieval top 20 → rerank top 5 → context compression（可选）
→ grounded generation → citation validation → answer + sources
```

| 环节 | 目标延迟 |
| --- | ---: |
| query embedding | 50–300 ms |
| vector search | 20–100 ms |
| BM25 / metadata filter | 10–80 ms |
| rerank top 20 | 100–800 ms |
| LLM generation | 1–8 s |

幂等与版本化：`checksum = sha256(normalized_text)`，未变化不重复 embedding；`chunk_id = sha256(doc_id + section + chunk_index + text)`，重跑不会重复；索引用版本号，如 `kb_2026_07_07_v3`，支持回滚。

## 13. 常见坑 & 排查表

| 问题 | 症状 | 根因 | 解决办法 |
| --- | --- | --- | --- |
| chunk 太大 | top-k 看似相关但答案找不到细节 | 一个 chunk 混入多个主题 | 从 800 降到 400–512；按标题切 |
| chunk 太小 | 答案缺条件、引用不完整 | 单个 chunk 只有半句话 | 增到 400–700；overlap 80–120 |
| overlap 过低 | 边界问题检索不到 | 关键句被切断 | overlap 调到 chunk size 的 15%–20% |
| embedding 与 query 不匹配 | 关键词相同也搜不到 | 入库和查询模型/指令不同 | 记录模型；重建索引；BGE query 加指令 |
| 中文召回差 | 中文语义匹配弱 | 英文 embedding 不适配 | 换 `bge-small-zh-v1.5`、`bge-m3` |
| 错误码搜不到 | `ERR_042`、函数名不命中 | dense 不擅长精确 token | 加 BM25；提高 BM25 权重 |
| rerank 太慢 | P95 延迟上升 | top 50 全部 cross-encoder | 粗召回 top 20；rerank top 5；缓存 |
| 丢失引用 | 答案对但无 `[1]` | context 没编号或 prompt 不强制 | 编号 context；输出后校验 |
| 幻觉 | 无资料也编答案 | prompt 无拒答规则；top-k 噪声 | 加拒答；提高阈值；引用校验 |
| 引用错位 | `[2]` 支持不了结论 | 模型随意引用 | 逐句检查引用；必要时输出 JSON evidence |
| 多租户越权 | A 客户搜到 B 客户文档 | 检索前未过滤 tenant | metadata filter 必须包含 `tenant_id` |
| 增量更新重复 | 同一段出现多次 | upsert id 不稳定 | 用稳定 `chunk_id` 幂等 upsert |

## 14. 检查清单

- [ ] 每个 chunk 都有 `id`、`source`、`chunk_index`、业务 metadata。
- [ ] 文档入库和 query 使用同一 embedding 模型与 normalize 策略。
- [ ] chunk size、overlap、top-k、rerank top-n 写入配置。
- [ ] 检索结果能展示原文来源，UI 可点击打开原文。
- [ ] prompt 明确「只基于 context 回答」和「资料不足就拒答」。
- [ ] 输出答案带引用，引用编号经过程序校验。
- [ ] 至少有 50 个真实问题组成 eval set，并记录 recall@k、MRR。
- [ ] 多租户或权限场景在检索阶段先过滤。
- [ ] embedding 模型升级有重建索引与回滚方案。
- [ ] 线上记录 query、chunk id、分数、rerank 分数、最终引用、用户反馈。

## 15. 动手练习

### 练习 1：给自己的文档建 RAG

产出物：一个 `docs\` 目录（至少 10 篇 Markdown / txt）；`build_index.py` 输出 docs、chunks、embedding 维度；`ask.py` 输入问题后输出答案与引用；三类终端输出：命中文档、拒答、引用展示。验收：至少 5 个问题命中正确来源，至少 1 个资料外问题拒答，每条答案含 `[n]` 引用。

### 练习 2：比较 chunking 参数

| 实验 | chunk size | overlap | top-k |
| --- | ---: | ---: | ---: |
| A | 256 | 40 | 5 |
| B | 512 | 80 | 5 |
| C | 800 | 120 | 8 |

产出物：`eval_questions.csv` 至少 20 个问题；记录每组 recall@5、MRR、平均 prompt 字符数；写出选择的配置与理由。

### 练习 3：加入混合检索和 rerank

```text
method,recall@5,mrr,avg_latency_ms
vector_only,0.72,0.55,120
hybrid,0.80,0.62,170
hybrid_rerank,0.86,0.74,620
```

你的数字不必与示例一致，但必须来自同一份 eval set。

## 16. 延伸阅读

- [Sentence Transformers Documentation](https://www.sbert.net/)：本地 embedding 与 cross-encoder rerank。
- [FAISS Wiki](https://github.com/facebookresearch/faiss/wiki)：向量索引类型、内积/余弦、ANN 参数。
- [Chroma Docs](https://docs.trychroma.com/)：本地持久化向量库与 metadata filter。
- [pgvector GitHub](https://github.com/pgvector/pgvector)：PostgreSQL 向量检索扩展。
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)：托管 embedding API 与模型选择。
- [Cohere Rerank Docs](https://docs.cohere.com/docs/reranking)：托管 rerank API。
- [RAGAS](https://docs.ragas.io/)：RAG 自动化评估指标与测试集生成。
- [Anthropic: Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval)：在 chunk 前补充上下文以提升召回。

`标签` `RAG` `LLM` `向量检索` `架构` `Embeddings` `FAISS` `混合检索` `Rerank` `评估`

---

[← 上一章](02-llm-app-architecture.md) · [WP-01 目录](README.md) · [下一章 →](04-prompt-engineering-patterns.md)
