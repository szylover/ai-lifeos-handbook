[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# LLM 应用的成本与延迟优化

> 结论先行：LLM 的两大生产成本是**token 费用**和**用户等待**。优化顺序不是先换最强基建，而是沿着瀑布做：**别调用（缓存/规则） → 用对模型（分级） → 调短（prompt/context 压缩） → 调快（流式/并发/批处理） → 改基座（小模型/自托管）**。

相关阅读：[LLM 应用架构](02-llm-app-architecture.md)、[微调 vs RAG vs Prompt](07-finetune-vs-rag-vs-prompt.md)、[Agent 架构模式](06-agent-architecture-patterns.md)、[LLM 应用评估](05-evals-for-llm-apps.md)。

## 0. 本章定位：把“感觉贵、感觉慢”变成可计算的工程问题

本章面向已经有线上 LLM 接口的工程师：你不需要先重写系统，只要能拿到一次 LLM 调用的 `model / prompt / completion / latency / user_id / feature`，就能开始优化。

前置环境：

- Python 3.10+；示例只依赖 `openai`、`numpy`、`redis`、`fastapi`、`uvicorn`。
- 一个可替换的 LLM provider；代码以 OpenAI SDK 风格演示，换成 Anthropic、Azure OpenAI、Gemini、本地 OpenAI-compatible server 时接口思路相同。
- 至少 50 条真实请求样本；如果没有，先从日志中抽样或在 staging 压测生成。

学完后你能独立完成：

1. 用公式估算单请求、单功能、月度 token 成本，并拆出 input/output 成本占比。
2. 给任意 LLM 接口加 token、延迟、缓存命中、用户维度成本埋点。
3. 实现精确缓存、语义缓存、provider prompt caching，并知道各自适用边界。
4. 写一个“便宜模型先做、困难任务升级”的 routing/cascade。
5. 通过压缩上下文、限制输出、结构化输出减少 token，并用评估防止质量掉线。
6. 用流式、并发、批处理、超时降级优化端到端延迟和吞吐。

## 1. 成本模型：LLM 账单到底怎么来的

### 1.1 按 token 计费公式

绝大多数托管 LLM 的在线调用按 input tokens 和 output tokens 分别计费：

$$
Cost_{request}=\frac{InputTokens}{1,000,000}\times Price_{input}+\frac{OutputTokens}{1,000,000}\times Price_{output}
$$

其中：

- `InputTokens`：system prompt、developer prompt、用户问题、RAG context、对话历史、工具结果等全部输入。
- `OutputTokens`：模型生成的回答、JSON、tool call arguments、reasoning visible output（不同 provider 对 hidden reasoning 计费口径不同，按账单为准）。
- `Price_input / Price_output`：每 1M token 单价。输出通常比输入贵 3–5 倍。

把一次功能调用拆成多次模型调用时：

$$
Cost_{feature}=\sum_{i=1}^{n} Cost_{request_i}+Cost_{embedding}+Cost_{rerank}+Cost_{infra}
$$

如果 RAG 每次还做 embedding：

$$
Cost_{embedding}=\frac{EmbeddingTokens}{1,000,000}\times Price_{embedding}
$$

> 实操建议：所有价格都放到配置表，不要写死在业务代码。provider 价格会变，模型也会迁移。

### 1.2 单请求成本 worked example

假设一个“客服问答”接口使用：

| 项目 | 数值 |
|---|---:|
| 模型 | `gpt-4.1-mini` |
| 示例输入价 | `$0.40 / 1M input tokens` |
| 示例输出价 | `$1.60 / 1M output tokens` |
| system + policy | 500 tokens |
| RAG 片段 | 2,200 tokens |
| 用户问题 + 历史 | 300 tokens |
| 模型输出 | 450 tokens |

则：

```text
InputTokens = 500 + 2200 + 300 = 3000
OutputTokens = 450
InputCost  = 3000 / 1_000_000 * 0.40 = $0.00120
OutputCost =  450 / 1_000_000 * 1.60 = $0.00072
RequestCost = $0.00192
```

如果每天 80,000 次调用：

```text
DailyCost = 80_000 * 0.00192 = $153.60
MonthlyCost(30d) = $4,608
```

注意输出只占 token 数的 13%，但占成本的 37.5%。所以“减少废话输出”经常比想象中值钱。

### 1.3 月度成本预测公式

按功能预测比按全站平均更准确：

$$
MonthlyCost=\sum_{feature} DAU_{feature}\times CallsPerUser_{feature}\times CostPerCall_{feature}\times 30
$$

再加上缓存命中率：

$$
EffectiveCostPerCall=(1-CacheHitRate)\times CostPerMiss+CacheHitRate\times CostPerHit
$$

通常 `CostPerHit` 近似为 Redis/DB 成本，可先按 `$0.000001` 或 0 估算。

示例：

| 功能 | DAU | 人均调用 | miss 单价 | 缓存命中 | 月成本 |
|---|---:|---:|---:|---:|---:|
| 客服问答 | 20,000 | 4 | `$0.00192` | 35% | `$2,995` |
| 简历改写 | 3,000 | 2 | `$0.01200` | 5% | `$2,052` |
| 标题生成 | 50,000 | 1 | `$0.00035` | 60% | `$210` |

计算客服问答：

```text
20_000 * 4 * 0.00192 * (1 - 0.35) * 30 = $2,995.20
```

### 1.4 用 Python 做一张成本表

依赖与运行：

```bash
pip install tabulate
python cost_model.py
```

```python
# cost_model.py
from tabulate import tabulate

PRICES = {
    # 示例价：真实生产请替换为 provider 当前价格
    "gpt-4.1": {"input": 2.00, "output": 8.00},
    "gpt-4.1-mini": {"input": 0.40, "output": 1.60},
    "gpt-4.1-nano": {"input": 0.10, "output": 0.40},
}

def request_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    p = PRICES[model]
    return input_tokens / 1_000_000 * p["input"] + output_tokens / 1_000_000 * p["output"]

def monthly_cost(dau: int, calls_per_user: float, cost_per_miss: float, cache_hit_rate: float) -> float:
    return dau * calls_per_user * cost_per_miss * (1 - cache_hit_rate) * 30

rows = []
for model in PRICES:
    c = request_cost(model, input_tokens=3000, output_tokens=450)
    rows.append([model, f"${c:.6f}", f"${monthly_cost(20000, 4, c, 0.35):,.0f}"])

print(tabulate(rows, headers=["model", "single request", "monthly @80k/day, 35% hit"]))
```

预期输出类似：

```text
model          single request    monthly @80k/day, 35% hit
-------------  ----------------  ---------------------------
gpt-4.1        $0.009600         $14,976
gpt-4.1-mini   $0.001920         $2,995
gpt-4.1-nano   $0.000480         $749
```

完成标准：你能对每个线上 LLM feature 输出 `p50/p95 tokens`、`p50/p95 latency`、`monthly forecast`、`cost by user/tenant`。

## 2. 优化优先级瀑布：先砍调用，再砍 token，最后砍基础设施

| 优先级 | 杠杆 | 典型收益 | 风险 | 最小验证 |
|---:|---|---:|---|---|
| 1 | 别调用：缓存、规则、模板 | 20–80% | 过期/错误复用 | 缓存命中答案通过人工抽检 |
| 2 | 用对模型：routing/cascade | 30–90% | 简单误判成困难或反之 | 黄金集准确率不降超 1–2pt |
| 3 | 调短：prompt/context/output | 15–60% | 丢关键信息 | eval + 失败样本 diff |
| 4 | 调快：streaming/concurrency/batch | 体感 30–70% | 复杂度、重试风暴 | p95 latency 与错误率 |
| 5 | 改基座：小模型、自托管、蒸馏 | 50–95% | 运维和质量 | TCO + 长周期压测 |

为什么这个顺序有效：

1. **不调用**的成本最低、延迟最低，质量风险可通过 TTL、版本号、置信度控制。
2. **模型分级**通常不改产品体验，只是把“所有请求都坐头等舱”改为“短途经济舱、复杂问题商务舱”。
3. **上下文瘦身**直接减少 input tokens，也缩短模型 prefill 时间。
4. **流式/并发**更多改善体感和吞吐，不一定减少账单。
5. **自托管/微调/蒸馏**是组织级工程，只有稳定高 QPS 或强隐私需求时才优先。

## 3. 别调用：精确缓存、语义缓存、prompt caching

### 3.1 精确缓存：相同请求直接返回

适用场景：标题生成、分类、摘要、FAQ、规则说明、同一文档的重复问答。关键是 cache key 必须包含会影响答案的所有因素。

推荐 key：

```text
llm:{feature}:{model}:{prompt_version}:{locale}:{sha256(normalized_input)}
```

包含 `prompt_version` 是为了防止 prompt 改了还复用旧答案；包含 `model` 是为了防止迁移模型后的风格/能力差异。

可运行示例（Redis 精确缓存 + OpenAI SDK）：

```bash
pip install openai redis
# Windows 可用 Docker Desktop 跑 Redis：docker run -p 6379:6379 redis:7
python exact_cache.py
```

```python
# exact_cache.py
import hashlib
import json
import os
from openai import OpenAI
import redis

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY", "sk-demo"))
r = redis.Redis(host="localhost", port=6379, decode_responses=True)

PROMPT_VERSION = "support-v3"
MODEL = "gpt-4.1-mini"

def normalize(text: str) -> str:
    return " ".join(text.strip().lower().split())

def cache_key(feature: str, user_input: str, locale: str = "zh-CN") -> str:
    digest = hashlib.sha256(normalize(user_input).encode("utf-8")).hexdigest()
    return f"llm:{feature}:{MODEL}:{PROMPT_VERSION}:{locale}:{digest}"

def call_llm(question: str) -> dict:
    # 把这段替换为真实 provider 调用；为避免误运行扣费，这里支持 MOCK=1
    if os.environ.get("MOCK", "1") == "1":
        return {"answer": f"这是缓存演示答案：{question}", "tokens": {"input": 120, "output": 30}}
    resp = client.chat.completions.create(
        model=MODEL,
        messages=[
            {"role": "system", "content": "你是客服助手。回答必须简洁、准确。"},
            {"role": "user", "content": question},
        ],
        max_tokens=200,
    )
    return {
        "answer": resp.choices[0].message.content,
        "tokens": {
            "input": resp.usage.prompt_tokens,
            "output": resp.usage.completion_tokens,
        },
    }

def answer(question: str) -> dict:
    key = cache_key("support_qa", question)
    cached = r.get(key)
    if cached:
        data = json.loads(cached)
        data["cache"] = "hit"
        return data

    data = call_llm(question)
    data["cache"] = "miss"
    r.setex(key, 60 * 60 * 24, json.dumps(data, ensure_ascii=False))  # TTL 1 天
    return data

if __name__ == "__main__":
    print(answer("如何修改发票抬头？"))
    print(answer(" 如何 修改 发票抬头？ "))  # normalize 后命中
```

完成标准：日志里能看到 `cache=hit/miss`；缓存命中时 provider 调用数不增加；prompt 改版后不会命中旧结果。

### 3.2 语义缓存：意思相同也复用

精确缓存无法处理“怎么改发票抬头”和“发票抬头在哪里修改”。语义缓存用 embedding 相似度判断是否复用历史答案。

适用条件：

- 答案对轻微措辞变化稳定，如 FAQ、政策解释、错误码说明。
- 有明确 TTL；库存、价格、用户私有状态不适合长 TTL。
- 命中阈值经人工/评估集调过，而不是拍脑袋。

最小可运行示例（内存向量库，便于理解；生产换 pgvector/Redis Vector/Pinecone/Milvus）：

```bash
pip install openai numpy
MOCK=1 python semantic_cache.py
```

```python
# semantic_cache.py
import hashlib
import math
import os
from dataclasses import dataclass
from typing import List
import numpy as np
from openai import OpenAI

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY", "sk-demo"))

@dataclass
class CacheItem:
    question: str
    answer: str
    embedding: np.ndarray
    prompt_version: str
    tenant_id: str

STORE: List[CacheItem] = []
PROMPT_VERSION = "support-v3"

def mock_embedding(text: str, dim: int = 64) -> np.ndarray:
    # 可跑的 deterministic mock：真实环境请调用 embeddings API
    h = hashlib.sha256(text.lower().encode()).digest()
    values = [(h[i % len(h)] / 255.0) for i in range(dim)]
    v = np.array(values, dtype=np.float32)
    return v / np.linalg.norm(v)

def embed(text: str) -> np.ndarray:
    if os.environ.get("MOCK", "1") == "1":
        return mock_embedding(text)
    resp = client.embeddings.create(model="text-embedding-3-small", input=text)
    v = np.array(resp.data[0].embedding, dtype=np.float32)
    return v / np.linalg.norm(v)

def cosine(a: np.ndarray, b: np.ndarray) -> float:
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

def llm_answer(question: str) -> str:
    return f"请在 设置 → 账务 → 发票信息 中修改。问题：{question}"

def semantic_answer(question: str, tenant_id: str, threshold: float = 0.88) -> dict:
    qv = embed(question)
    best = None
    best_score = -math.inf
    for item in STORE:
        if item.tenant_id != tenant_id or item.prompt_version != PROMPT_VERSION:
            continue
        score = cosine(qv, item.embedding)
        if score > best_score:
            best, best_score = item, score

    if best and best_score >= threshold:
        return {"answer": best.answer, "cache": "semantic_hit", "score": round(best_score, 4), "matched": best.question}

    answer = llm_answer(question)
    STORE.append(CacheItem(question, answer, qv, PROMPT_VERSION, tenant_id))
    return {"answer": answer, "cache": "miss", "score": None, "matched": None}

if __name__ == "__main__":
    print(semantic_answer("如何修改发票抬头？", tenant_id="t1"))
    print(semantic_answer("发票抬头在哪里改？", tenant_id="t1", threshold=0.80))
```

生产实现要补齐：

| 设计点 | 推荐做法 |
|---|---|
| 隔离 | `tenant_id/user_scope/prompt_version/model` 都参与过滤 |
| 阈值 | 从 0.82、0.86、0.90、0.94 做离线评估，选“错误命中率 < 0.5%”的点 |
| TTL | FAQ 1–7 天；价格/库存 0–5 分钟；合规答案按版本失效 |
| 审计 | 命中时记录 `matched_question`、similarity、cache_item_id |
| 回退 | 相似度在灰区（如 0.82–0.88）时不直接复用，可让小模型判等价 |

### 3.3 Provider prompt caching：缓存稳定长前缀

很多 provider 支持 prompt caching：当 system prompt、工具说明、few-shot、长文档前缀重复时，provider 对这部分 input tokens 给折扣或更低延迟。通用原则：

1. 把稳定内容放在 prompt 前缀：system policy、工具 schema、few-shot、固定产品说明。
2. 把每次变化的内容放后面：用户问题、当前检索片段、当前时间。
3. 不要把随机 request id、时间戳、用户昵称塞进前缀，否则缓存键被打碎。
4. 监控 provider 返回的 cached tokens 字段；不同 SDK 字段名不同。

Prompt 排列示例：

```text
[稳定，可缓存]
- system policy v12
- 输出 JSON schema
- 3 个 few-shot examples
- 产品帮助中心核心规则

[变化，不利于缓存]
- user_id = 123
- now = 2026-07-07T10:20:53+08:00
- 用户本轮问题
- 本轮检索出来的片段
```

如果 provider 支持显式 cache control，可以把长前缀标出来；如果只支持自动缓存，就保证 byte-level 前缀稳定。

## 4. 用对模型：模型分级 routing 与 cascade

### 4.1 路由策略

| 请求类型 | 推荐模型 | 升级条件 |
|---|---|---|
| 分类、抽取、短 JSON | nano/small | JSON parse 失败、置信度低 |
| FAQ、轻量改写 | mini | 命中敏感主题、用户追问复杂 |
| 多文档推理、代码生成 | strong | 默认直接强模型或 mini 先草稿再强模型审稿 |
| 高风险合规/金融/医疗 | strong + guardrail | 不用纯成本路由 |

常见两种做法：

1. **预路由**：规则/小模型先判断难度，再选模型。优点是省钱；缺点是路由错会影响质量。
2. **级联 cascade**：小模型先答；如果失败、低置信度、格式错误、触发风险，再升级强模型。优点是稳；缺点是升级请求多一次延迟。

### 4.2 可运行 router 示例

下面示例不真实扣费，`MOCK=1` 默认打开；接入 provider 时实现 `call_provider` 即可。

```bash
python router.py
```

```python
# router.py
import json
import os
import re
from dataclasses import dataclass

@dataclass
class LLMResult:
    model: str
    text: str
    confidence: float
    input_tokens: int
    output_tokens: int

PRICES = {
    "gpt-4.1-nano": {"input": 0.10, "output": 0.40},
    "gpt-4.1-mini": {"input": 0.40, "output": 1.60},
    "gpt-4.1": {"input": 2.00, "output": 8.00},
}

def estimate_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    p = PRICES[model]
    return input_tokens / 1_000_000 * p["input"] + output_tokens / 1_000_000 * p["output"]

def classify_complexity(user_text: str) -> str:
    text = user_text.lower()
    if len(text) < 80 and any(k in text for k in ["分类", "提取", "json", "标题"]):
        return "simple"
    if any(k in text for k in ["合同", "法律", "医疗", "财务审计", "多文档", "架构设计"]):
        return "hard"
    if len(text) > 600 or text.count("?") + text.count("？") >= 3:
        return "hard"
    return "medium"

def call_model(model: str, prompt: str) -> LLMResult:
    # MOCK 版本：真实环境替换为 SDK 调用，并填 usage tokens
    input_tokens = max(1, len(prompt) // 2)
    if model.endswith("nano"):
        confidence = 0.72 if "架构" not in prompt else 0.45
        text = '{"label":"general","confidence":0.72}'
        output_tokens = 18
    elif model.endswith("mini"):
        confidence = 0.84
        text = "这是 mini 模型的答案。"
        output_tokens = 80
    else:
        confidence = 0.95
        text = "这是强模型的高置信答案，包含更完整推理。"
        output_tokens = 180
    return LLMResult(model, text, confidence, input_tokens, output_tokens)

def needs_escalation(result: LLMResult, task: str) -> bool:
    if result.confidence < 0.80:
        return True
    if task == "simple":
        try:
            json.loads(result.text)
        except json.JSONDecodeError:
            return True
    if re.search(r"不知道|无法确定|可能不准确", result.text):
        return True
    return False

def route_and_answer(user_text: str) -> dict:
    task = classify_complexity(user_text)
    first_model = {"simple": "gpt-4.1-nano", "medium": "gpt-4.1-mini", "hard": "gpt-4.1"}[task]
    first = call_model(first_model, user_text)
    calls = [first]

    if first_model != "gpt-4.1" and needs_escalation(first, task):
        calls.append(call_model("gpt-4.1", user_text))

    final = calls[-1]
    total_cost = sum(estimate_cost(c.model, c.input_tokens, c.output_tokens) for c in calls)
    return {
        "task": task,
        "final_model": final.model,
        "calls": [c.model for c in calls],
        "answer": final.text,
        "estimated_cost": round(total_cost, 6),
    }

if __name__ == "__main__":
    for q in ["把这句话分类成 JSON：我要退款", "请设计一个多租户 RAG 架构，包含权限和评估"]:
        print(route_and_answer(q))
```

上线前用黄金集评估：

```text
baseline: 全部 gpt-4.1，准确率 92.0%，平均成本 $0.0100
router:   nano/mini/4.1，准确率 91.4%，平均成本 $0.0031
结论：准确率 -0.6pt，成本 -69%，若业务容忍阈值为 -1pt，可上线灰度。
```

## 5. 调短：context、prompt、output 都要瘦身

### 5.1 Context 压缩：只把有用材料放进窗口

RAG 常见浪费是“top_k=10，每段 800 token，全部塞入 prompt”。改法：

1. 先用检索召回 `top_k=20`。
2. 用 reranker 选 `top_n=4`。
3. 每段只取命中窗口附近 1–3 个段落。
4. 对长段落做 extractive compression：只保留包含关键词/实体/答案候选的句子。
5. 记录 `context_tokens`，超过预算直接拒绝或二次压缩。

简单压缩函数：

```python
# context_compress.py
import re

def compress_passage(query: str, passage: str, max_chars: int = 900) -> str:
    terms = [t for t in re.split(r"\W+", query.lower()) if len(t) >= 2]
    sentences = re.split(r"(?<=[。！？.!?])\s*", passage.strip())
    scored = []
    for i, s in enumerate(sentences):
        score = sum(1 for t in terms if t in s.lower())
        if score:
            scored.append((score, i, s))
    picked = [s for _, _, s in sorted(scored, key=lambda x: (-x[0], x[1]))[:5]]
    result = "".join(picked) or passage[:max_chars]
    return result[:max_chars]

if __name__ == "__main__":
    q = "企业版如何修改发票抬头"
    p = "个人版不支持发票。企业版管理员可在设置-账务-发票信息中修改发票抬头。修改后下个账期生效。其他说明略。"
    print(compress_passage(q, p))
```

完成标准：`context_tokens p50/p95` 下降；答案引用仍能覆盖黄金集证据；无答案问题不会因压缩而 hallucinate。

### 5.2 Prompt 压缩：删掉“礼貌废话”和重复规则

压缩前：

```text
你是一个非常专业、非常耐心、非常善于帮助用户的 AI 助手。请你一步一步思考，尽可能详细地回答用户问题，必要时解释背景、原理、注意事项，并在最后总结。
```

压缩后：

```text
你是客服助手。只基于给定资料回答；无依据时说“不确定”。输出不超过 120 字。
```

对比：前者 50+ 中文字且鼓励长输出；后者约 30 字且有质量边界和长度约束。

可执行 prompt review 清单：

- 删除重复身份描述：`专业/耐心/友好` 通常不会改善准确率。
- 删除泛化过程要求：不需要时不要写 `step by step`，它会诱导长输出。
- 把长规则改成表格或枚举短句。
- few-shot 从 8 个降到 2–3 个代表性样例；剩余样例放到 eval，不放 prompt。
- 给 prompt 加版本号，便于缓存与回滚。

### 5.3 减少 output tokens：限制长度、停止符、结构化输出

输出优化通常最划算，因为 output token 单价更高、还直接影响生成时延。

做法：

```python
response = client.chat.completions.create(
    model="gpt-4.1-mini",
    messages=[...],
    max_tokens=160,              # 硬上限
    temperature=0.2,
    response_format={"type": "json_object"},  # 支持时使用结构化输出
)
```

把自然语言改为 JSON 也能省 token：

```text
长回答：
该用户的意图是申请退款，情绪比较着急，建议转人工并优先处理。

短 JSON：
{"intent":"refund","sentiment":"urgent","handoff":true}
```

如果 UI 只需要字段，不要让模型生成完整句子。让后端模板渲染：

```python
TEMPLATES = {
    "refund": "我已识别到退款诉求，将为你转接退款流程。",
    "invoice": "你可以在 设置 → 账务 → 发票信息 修改。",
}
```

### 5.4 对话历史压缩

策略：

| 历史类型 | 保留方式 |
|---|---|
| 最近 2–4 轮 | 原文保留 |
| 更早事实 | 滚动摘要 |
| 已完成工具结果 | 保存结构化状态，不保存长日志 |
| 闲聊/确认语 | 丢弃 |

滚动摘要模板：

```text
把以下对话压缩成状态摘要，最多 120 字。只保留：用户目标、关键约束、已确认事实、未解决问题。不要保留寒暄。
```

完成标准：历史 token 降低 50%+；多轮指代问题的通过率不明显下降。

## 6. 调快：首字、并发、批处理、部署、超时降级

### 6.1 延迟公式

一次 LLM 调用的用户可见延迟可拆为：

$$
Latency = QueueTime + NetworkRTT + PrefillTime(InputTokens) + DecodeTime(OutputTokens) + PostProcess
$$

流式场景下：

$$
TTFT = QueueTime + NetworkRTT + PrefillTime
$$

$$
TotalTime = TTFT + OutputTokens \times TimePerOutputToken
$$

优化映射：

| 指标 | 主要受什么影响 | 优化手段 |
|---|---|---|
| TTFT | 队列、网络、input tokens | 就近部署、prompt caching、减少 context |
| 总耗时 | output tokens、模型速度 | 限制输出、换小模型、streaming |
| p95/p99 | 重试、排队、慢依赖 | 超时、熔断、并发隔离 |
| 吞吐 | rate limit、batch size | 批处理、异步队列、连接复用 |

### 6.2 流式输出：改善体感，不一定省钱

FastAPI SSE 示例：

```bash
pip install fastapi uvicorn openai
uvicorn stream_app:app --reload
curl -N "http://127.0.0.1:8000/chat?q=解释LLM成本优化"
```

```python
# stream_app.py
import os
import time
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from openai import OpenAI

app = FastAPI()
client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY", "sk-demo"))

@app.get("/chat")
def chat(q: str):
    def events():
        if os.environ.get("MOCK", "1") == "1":
            for token in ["先", "缓存", "，再", "分级", "，最后", "压缩。"]:
                time.sleep(0.15)
                yield f"data: {token}\n\n"
            yield "data: [DONE]\n\n"
            return

        stream = client.chat.completions.create(
            model="gpt-4.1-mini",
            messages=[{"role": "user", "content": q}],
            stream=True,
            max_tokens=200,
        )
        for chunk in stream:
            delta = chunk.choices[0].delta.content or ""
            if delta:
                yield f"data: {delta}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(events(), media_type="text/event-stream")
```

完成标准：前端能在 1 秒内看到首字；服务端记录 `ttft_ms` 和 `total_ms`。

### 6.3 并发：独立子任务同时跑

如果一个请求需要“检索用户资料、检索文档、做安全检查、调用 LLM”，不要串行等待所有 I/O。

```python
# concurrent_io.py
import asyncio
import time

async def fetch_profile():
    await asyncio.sleep(0.2)
    return {"plan": "pro"}

async def retrieve_docs():
    await asyncio.sleep(0.35)
    return ["doc1", "doc2"]

async def safety_check():
    await asyncio.sleep(0.15)
    return "ok"

async def main():
    started = time.perf_counter()
    profile, docs, safety = await asyncio.gather(fetch_profile(), retrieve_docs(), safety_check())
    print(profile, docs, safety, "elapsed", round(time.perf_counter() - started, 3))

asyncio.run(main())
```

串行约 0.70s，并发约 0.35s。注意给每个下游设置 timeout，避免一个慢依赖拖死全链路。

### 6.4 批处理：离线任务用吞吐换延迟

适用：批量摘要、离线标签、日报生成、数据清洗。不要把用户在线等待的请求塞进大 batch。

简化 batch worker：

```python
# batch_worker.py
from collections import deque

queue = deque()

def enqueue(item):
    queue.append(item)

def flush_batch(max_batch_size=20):
    batch = []
    while queue and len(batch) < max_batch_size:
        batch.append(queue.popleft())
    if not batch:
        return []
    # 真实环境：调用 provider batch API 或把多个短任务合成一个请求
    return [{"id": x["id"], "summary": x["text"][:30]} for x in batch]

for i in range(5):
    enqueue({"id": i, "text": "这是一段需要摘要的长文本" * 10})
print(flush_batch())
```

### 6.5 投机解码、预取与 speculative execution

投机解码（speculative decoding）通常由推理服务实现：小模型先生成 draft tokens，大模型验证接受，从而提高 tokens/s。托管 API 未必暴露开关；自托管 vLLM/TGI/llama.cpp 可能支持类似能力。

应用层的 speculative execution：

- 用户停留在输入框时预取 embedding 或检索候选。
- 对高概率下一步工具调用提前准备数据。
- 对低成本小模型先生成草稿，同时等待强模型或检索结果。

风险：预测错会浪费钱。只在命中率高、单次成本低、延迟收益明确的路径使用，并记录 `wasted_speculation_cost`。

### 6.6 就近部署、连接复用、超时与降级

部署清单：

- 选择离用户和 provider endpoint 更近的 region；跨洲 RTT 会直接加到 TTFT。
- 复用 HTTP client；不要每次请求新建 TLS 连接。
- 对 LLM 调用设置分层 timeout：在线客服 8–12s，后台摘要 60–120s。
- 超时后降级：返回缓存旧答案、较短答案、转人工、异步通知，而不是无限等待。

超时降级示例：

```python
# timeout_fallback.py
import asyncio

async def call_llm():
    await asyncio.sleep(2.0)
    return "fresh answer"

async def answer_with_timeout():
    try:
        return await asyncio.wait_for(call_llm(), timeout=0.8)
    except asyncio.TimeoutError:
        return "系统繁忙，先返回已缓存的简版答案；完整结果稍后更新。"

print(asyncio.run(answer_with_timeout()))
```

## 7. 可观测与预算护栏：没有账本就没有优化

### 7.1 每次调用必须记录的字段

| 字段 | 用途 |
|---|---|
| `request_id` | 串联日志、trace、用户反馈 |
| `tenant_id/user_id` | 成本归因、限流、滥用排查 |
| `feature` | 找出最贵功能 |
| `model` | 模型迁移与路由效果 |
| `prompt_version` | prompt 改动对成本/质量影响 |
| `input_tokens/output_tokens` | 成本计算 |
| `cached_tokens` | provider prompt caching 效果 |
| `cache_status` | exact/semantic/provider hit rate |
| `ttft_ms/total_ms` | 延迟优化 |
| `estimated_cost_usd` | 实时预算 |
| `quality_signal` | thumbs up/down、eval score、人工审核 |

### 7.2 成本埋点 wrapper

```python
# metered_llm.py
import time
from dataclasses import dataclass, asdict

PRICES = {"gpt-4.1-mini": {"input": 0.40, "output": 1.60}}

@dataclass
class LLMMetric:
    feature: str
    user_id: str
    model: str
    prompt_version: str
    input_tokens: int
    output_tokens: int
    latency_ms: int
    estimated_cost_usd: float
    cache_status: str

def estimate(model: str, input_tokens: int, output_tokens: int) -> float:
    p = PRICES[model]
    return input_tokens / 1_000_000 * p["input"] + output_tokens / 1_000_000 * p["output"]

def emit_metric(metric: LLMMetric):
    # 生产中写入 OpenTelemetry/Prometheus/ClickHouse/BigQuery
    print(asdict(metric))

def metered_call(feature: str, user_id: str, prompt: str) -> str:
    started = time.perf_counter()
    model = "gpt-4.1-mini"
    prompt_version = "v3"
    # mock usage
    answer = "ok"
    input_tokens = len(prompt) // 2
    output_tokens = 20
    latency_ms = int((time.perf_counter() - started) * 1000)
    emit_metric(LLMMetric(
        feature=feature,
        user_id=user_id,
        model=model,
        prompt_version=prompt_version,
        input_tokens=input_tokens,
        output_tokens=output_tokens,
        latency_ms=latency_ms,
        estimated_cost_usd=estimate(model, input_tokens, output_tokens),
        cache_status="miss",
    ))
    return answer

metered_call("support_qa", "u123", "如何修改发票？")
```

### 7.3 按用户/功能限流与预算护栏

Redis 滑动窗口简化示例：

```python
# budget_guard.py
import time
from collections import defaultdict, deque

WINDOW_SECONDS = 60 * 60
USER_BUDGET_USD = 0.50
FEATURE_BUDGET_USD = {"support_qa": 50.0, "resume_rewrite": 20.0}

user_spend = defaultdict(deque)      # user_id -> deque[(ts, cost)]
feature_spend = defaultdict(deque)   # feature -> deque[(ts, cost)]

def _prune(q, now):
    while q and now - q[0][0] > WINDOW_SECONDS:
        q.popleft()

def _sum(q):
    return sum(cost for _, cost in q)

def allow(user_id: str, feature: str, estimated_cost: float) -> tuple[bool, str]:
    now = time.time()
    uq = user_spend[user_id]
    fq = feature_spend[feature]
    _prune(uq, now)
    _prune(fq, now)

    if _sum(uq) + estimated_cost > USER_BUDGET_USD:
        return False, "user_hourly_budget_exceeded"
    if _sum(fq) + estimated_cost > FEATURE_BUDGET_USD.get(feature, 10.0):
        return False, "feature_hourly_budget_exceeded"

    uq.append((now, estimated_cost))
    fq.append((now, estimated_cost))
    return True, "ok"

print(allow("u1", "support_qa", 0.02))
```

生产护栏：

- **硬限流**：单用户/租户每小时 token 或美元上限。
- **软告警**：某功能小时成本超过 7 日同期均值 2 倍时通知 Slack/飞书。
- **熔断**：provider 错误率或 p95 延迟过高时切备用模型/返回缓存。
- **配额展示**：企业客户在后台看到本月 token 用量，减少账单争议。

### 7.4 成本告警 SQL 示例

假设日志表 `llm_calls` 有 `created_at, feature, user_id, estimated_cost_usd, input_tokens, output_tokens`：

```sql
-- 最近 1 小时成本最高的功能
SELECT feature,
       COUNT(*) AS calls,
       SUM(estimated_cost_usd) AS cost_usd,
       AVG(input_tokens) AS avg_input,
       AVG(output_tokens) AS avg_output
FROM llm_calls
WHERE created_at >= NOW() - INTERVAL '1 hour'
GROUP BY feature
ORDER BY cost_usd DESC
LIMIT 20;
```

```sql
-- 疑似滥用用户：1 小时成本超过 $5
SELECT user_id,
       COUNT(*) AS calls,
       SUM(estimated_cost_usd) AS cost_usd
FROM llm_calls
WHERE created_at >= NOW() - INTERVAL '1 hour'
GROUP BY user_id
HAVING SUM(estimated_cost_usd) > 5
ORDER BY cost_usd DESC;
```

## 8. 完整案例：把“客服问答”接口成本降 70%

### 8.1 初始状态

某 B2B SaaS 的 `/api/support/answer`：

| 指标 | Before |
|---|---:|
| 日调用量 | 80,000 |
| 模型 | 全部 `gpt-4.1` |
| 平均 input tokens | 3,000 |
| 平均 output tokens | 450 |
| 单请求成本 | `$0.00960` |
| 月成本 | `$23,040` |
| p95 总延迟 | 9.2s |
| 答案通过率（黄金集） | 92.0% |

计算：

```text
gpt-4.1: input $2/M, output $8/M
3000/1M*2 + 450/1M*8 = 0.006 + 0.0036 = $0.0096
80,000 * 30 * 0.0096 = $23,040/月
```

### 8.2 优化手段表

| 步骤 | 改动 | 成本影响 | 延迟影响 | 质量验证 |
|---:|---|---:|---:|---|
| 1 | FAQ 精确缓存，TTL 24h，prompt_version 入 key | 18% 请求不调用 | 命中 <50ms | 命中样本 200 条人工抽检 0 错 |
| 2 | 语义缓存，阈值 0.90，仅公开 FAQ | 额外 12% 不调用 | 命中 <80ms | 错误命中率 0.3% |
| 3 | RAG top_k 10→4，段落压缩 800→350 tokens | input 3000→1700 | TTFT -28% | 黄金集召回 -0.4pt |
| 4 | 输出 JSON + 模板渲染，max_tokens 450→220 | output 450→220 | 总耗时 -31% | 人工满意度持平 |
| 5 | Router：70% miss 用 `gpt-4.1-mini`，30% 升级 `gpt-4.1` | 模型单价下降 | p95 -22% | 准确率 91.5% |
| 6 | 流式 + 8s timeout + 旧缓存降级 | 账单不变 | 体感 TTFT <1s | 超时投诉下降 |

### 8.3 After 数字

假设总缓存命中 30%，剩余 70% miss 中：70% 走 mini，30% 走强模型。压缩后平均 `input=1700`、`output=220`。

```text
mini 单价:
1700/1M*0.40 + 220/1M*1.60 = $0.001032
strong 单价:
1700/1M*2.00 + 220/1M*8.00 = $0.005160
miss 平均:
0.70 * 0.001032 + 0.30 * 0.005160 = $0.0022716
含缓存有效单价:
0.70 * 0.0022716 = $0.001590
月成本:
80,000 * 30 * 0.001590 = $3,816
降幅:
1 - 3816 / 23040 = 83.4%
```

为了保守估算，加入 embedding、Redis、重试、监控等额外 `$2,800/月`，总成本约 `$6,616/月`，仍下降：

```text
1 - 6616 / 23040 = 71.3%
```

最终状态：

| 指标 | Before | After |
|---|---:|---:|
| 月成本 | `$23,040` | `$6,616` |
| 单请求有效成本 | `$0.00960` | `$0.00276` |
| p95 总延迟 | 9.2s | 4.8s |
| TTFT | 2.1s | 0.8s |
| 黄金集通过率 | 92.0% | 91.5% |
| 用户差评率 | 3.1% | 3.0% |

上线顺序：先埋点 → 只读影子计算成本 → 5% 流量启用缓存 → 20% 启用 router → 50% 压缩 context → 全量 → 每日回看失败样本。

## 9. 常见坑 & 排查表

| 症状 | 可能原因 | 排查 SQL/日志 | 修复 |
|---|---|---|---|
| 成本突然翻倍 | prompt 加了长 few-shot 或 RAG top_k 变大 | 看 `avg_input_tokens by prompt_version` | 回滚 prompt；给 context token budget |
| p95 延迟很高但平均正常 | provider 排队、重试、慢用户输入 | 看 `p95 by model/region/retry_count` | timeout、熔断、备用模型 |
| 缓存命中率低 | key 含时间戳/随机数；normalize 不足 | 抽样比较相似请求 key | 稳定前缀；规范化输入 |
| 语义缓存答错 | 阈值太低；跨租户复用；答案过期 | 查 `matched_question/similarity/tenant_id` | 提高阈值；加 tenant 过滤；缩 TTL |
| 换小模型后投诉增加 | router 把困难任务分给小模型 | 对差评样本看 `task/model/confidence` | 加升级规则；困难类直接强模型 |
| JSON parse 失败重试多 | prompt 不严格或模型不支持 JSON mode | 看 `parse_error_rate` | 使用 structured output；失败升级 |
| 输出越来越长 | prompt 鼓励详细解释；没设 max_tokens | 看 `avg_output_tokens by version` | 加 `max_tokens` 和模板渲染 |
| RAG 答案 hallucinate | 压缩丢证据；无答案也强答 | 看引用覆盖率 | 要求引用 doc_id；无证据返回不确定 |
| 月账单和估算不一致 | hidden reasoning、cached token 折扣、重试漏记 | 对账 provider usage export | 以账单校准价格表和 usage 字段 |
| 批处理影响在线体验 | 在线请求进入 batch 队列 | 看 queue wait time | 在线/离线队列隔离 |

## 10. 检查清单

- [ ] 每个 LLM 调用都有 `feature/model/prompt_version/input_tokens/output_tokens/latency/cost/user_id`。
- [ ] 价格配置可热更新，且每月和 provider 账单对账一次。
- [ ] 已按功能计算 `monthly forecast`，知道 Top 5 成本来源。
- [ ] 精确缓存 key 包含 `model/prompt_version/locale/tenant_scope/input_hash`。
- [ ] 语义缓存有阈值评估、TTL、租户隔离、命中审计。
- [ ] 稳定长前缀放在 prompt 前面，避免随机字段破坏 provider prompt caching。
- [ ] Router 有黄金集评估；准确率下降阈值预先定义。
- [ ] RAG context 有 token budget；超过预算会压缩或拒绝。
- [ ] 输出设置 `max_tokens`；能用 JSON/枚举就不用长自然语言。
- [ ] 在线请求支持 streaming 或至少记录 TTFT。
- [ ] 所有下游 I/O 有 timeout；超时有降级而不是无限重试。
- [ ] 离线任务走 batch/队列，不挤占在线 rate limit。
- [ ] 有按用户/租户/功能的预算护栏和异常告警。
- [ ] 每次优化都跑 eval，不只看成本下降。

## 11. 动手练习：给你的一个 LLM 调用做成本审计并优化

选择一个真实接口，例如 `/api/chat`、`/api/summarize`、`/api/extract`。不要先改代码，先审计。

### 练习 A：成本账本

产出物：`cost_audit.md` 或 issue/comment（如果团队不允许新文件，就贴到工单）。内容必须包含：

1. 最近 7 天调用量、p50/p95 input tokens、p50/p95 output tokens、p50/p95 latency。
2. 当前模型价格、单请求均价、月度预测。
3. Top 10 用户/租户成本。
4. 失败率、重试率、缓存命中率（没有缓存就写 0）。

计算模板：

```text
Feature: support_qa
Model: gpt-4.1-mini
Avg input/output: 3000 / 450
Unit cost: 3000/1M*0.40 + 450/1M*1.60 = $0.00192
Monthly forecast: DAU * calls/user/day * unit_cost * 30
```

### 练习 B：提出 3 个优化 PRD

每个 PRD 用同一张表：

| 优化 | 预期成本下降 | 预期延迟下降 | 质量风险 | 验证方法 | 回滚条件 |
|---|---:|---:|---|---|---|
| exact cache | 20% | 命中 <50ms | 旧答案 | 200 条命中抽检 | 错误命中 >0.5% |

至少包含一个“不调用”方案、一个“调短”方案、一个“用对模型”方案。

### 练习 C：上线一个最小优化

建议顺序：

1. 加埋点 wrapper，不改变回答。
2. 上 exact cache，仅对公开 FAQ 或 deterministic 任务开启。
3. 灰度 5% 流量，观察 24 小时。
4. 输出 before/after：成本、p95 latency、错误率、质量反馈。

验收标准：

- 成本或延迟至少一个指标下降 20%+。
- 质量指标未超过预设劣化阈值。
- 有 dashboard 或 SQL 能持续跟踪。
- 有一键回滚：关闭缓存、关闭 router、恢复旧 prompt_version。

## 12. 延伸阅读

- OpenAI API Pricing：https://openai.com/api/pricing/
- OpenAI Prompt Caching：https://platform.openai.com/docs/guides/prompt-caching
- Anthropic Prompt Caching：https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- Google Gemini Context Caching：https://ai.google.dev/gemini-api/docs/caching
- OpenTelemetry：https://opentelemetry.io/docs/
- Redis Vector Similarity Search：https://redis.io/docs/latest/develop/interact/search-and-query/advanced-concepts/vectors/
- pgvector：https://github.com/pgvector/pgvector
- vLLM Optimization and Serving：https://docs.vllm.ai/

## 小结

LLM 成本与延迟优化的核心不是“找一个神奇便宜模型”，而是建立可观测账本，然后按瀑布逐层处理：能缓存就别调用，能路由就别全走强模型，能压缩就别塞满上下文，能流式/并发就别让用户干等，最后再评估蒸馏、自托管等基座改造。每一步都必须用成本、延迟、质量三张表验收；只省钱但质量失控，等于把成本转移给用户和客服。


`标签` `成本` `延迟` `性能` `缓存` `LLM` `生产` `可观测性` `模型路由`

---

[← 上一章](07-finetune-vs-rag-vs-prompt.md) · [WP-01 目录](README.md)
