[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# RAG 架构入门

检索增强生成（Retrieval-Augmented Generation, RAG）把「外部知识检索」与「大模型生成」结合，
让模型在回答时引用最新、私有或领域特定的资料，而不是仅依赖训练时的参数记忆。

## 核心数据流

1. **切分（Chunking）**：把文档切成语义完整的片段。
2. **向量化（Embedding）**：用 embedding 模型把每个 chunk 映射为向量 $\mathbf{v} \in \mathbb{R}^d$。
3. **检索（Retrieval）**：对查询向量 $\mathbf{q}$，用余弦相似度找 top-k：

$$
\text{sim}(\mathbf{q}, \mathbf{v}) = \frac{\mathbf{q} \cdot \mathbf{v}}{\lVert \mathbf{q} \rVert \, \lVert \mathbf{v} \rVert}
$$

4. **生成（Generation）**：把 top-k chunk 拼进 prompt，交给 LLM 生成答案。

## 最小实现骨架

```ts
async function rag(query: string) {
  const qVec = await embed(query);
  const chunks = await vectorStore.search(qVec, { topK: 5 });
  const context = chunks.map((c) => c.text).join("\n---\n");
  return llm.complete(`基于以下资料回答：\n${context}\n\n问题：${query}`);
}
```

## 工程权衡

| 维度 | 选择 | 说明 |
| --- | --- | --- |
| 切分粒度 | 200–500 token | 太大稀释相关性，太小丢上下文 |
| 向量库 | pgvector / Qdrant | v1 预留，v2 接入 |
| 重排序 | Cross-encoder | 提升 top-k 精度 |

> 在 AI LifeOS 中，RAG 属于 **v2**：v1 先把知识库内容与元数据做扎实，为向量化预留 `Embedding` 模型。


`标签` `RAG` `LLM` `向量检索` `架构`

---

[← 上一章](02-llm-app-architecture.md) · [WP-01 目录](README.md) · [下一章 →](04-prompt-engineering-patterns.md)
