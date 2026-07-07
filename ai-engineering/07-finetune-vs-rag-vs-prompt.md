[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# 微调 vs RAG vs Prompt：如何选对 LLM 定制路径

> 结论先行：**先 Prompt，再 RAG，最后才微调**。90% 的“要不要微调”其实是“知识没喂对”或“prompt 没写好”。微调解决的是**行为/风格/格式**问题，不是**知识新鲜度**问题——后者是 RAG 的活。

相关阅读：[RAG 架构](03-rag-architecture.md)、[Prompt 工程模式](04-prompt-engineering-patterns.md)、[LLM 应用评估](05-evals-for-llm-apps.md)、[LLM 应用架构](02-llm-app-architecture.md)、[成本与延迟优化](08-llm-cost-latency-optimization.md)。

## 1. 本章定位：把“定制模型”拆成三种工程选择

你不需要一开始就训练模型。LLM 定制通常有三条路：

1. **Prompt / few-shot**：通过指令、上下文、示例、工具调用约束模型行为。
2. **RAG（Retrieval-Augmented Generation）**：把外部知识检索出来，再交给模型生成答案。
3. **Fine-tuning（微调）**：用训练数据改变模型在某类输入上的默认输出分布。

学完本章，你应该能独立完成：

- 用一个决策树判断需求该走 Prompt、RAG、微调，还是组合方案。
- 为一个业务任务先建 evals，再量化三条路径的收益。
- 写出可上线的 few-shot prompt 模板，而不是把所有规则堆进 system prompt。
- 搭建一个最小 RAG 流程，判断“知识问题”是否已被检索解决。
- 准备 SFT / DPO 数据集，理解 LoRA / QLoRA 的适用边界。
- 用 Hugging Face PEFT + TRL 跑一个最小 LoRA 微调流程，并知道如何评估、合并、服务。

前置建议：

- 先读 [Prompt 工程模式](04-prompt-engineering-patterns.md)，理解 role、few-shot、结构化输出、反例约束。
- 先读 [RAG 架构](03-rag-architecture.md)，理解 chunk、embedding、retriever、reranker、citation。
- 先读 [LLM 应用评估](05-evals-for-llm-apps.md)，因为没有 evals 的微调基本是在烧钱。

## 2. 一句话决策框架

把问题先分成三类：

| 需求本质 | 首选路径 | 为什么 | 典型例子 |
| --- | --- | --- | --- |
| **知识问题**：模型缺少私有、最新、可追溯资料 | **RAG** | 知识经常变化，应该放在外部索引里，而不是塞进参数 | 公司制度问答、产品文档助手、法务条款检索 |
| **行为/风格/格式问题**：模型知道信息，但输出不稳定 | **微调** | 微调擅长改变默认输出习惯，让模型更像“受过训练的员工” | 固定客服话术、代码审查风格、严格 JSON schema、分类标签 |
| **其余问题**：通用能力 + 少量上下文即可解决 | **Prompt** | 最快、最便宜、可逆，适合先验证需求 | 摘要、改写、翻译、一次性分析、轻量抽取 |

成本优先级：

```text
Prompt < RAG < 微调
```

对应的工程含义：

- **Prompt**：几小时到几天；主要成本是 prompt 设计、评估样例、token。
- **RAG**：几天到几周；主要成本是清洗文档、索引、检索质量、权限、引用、缓存。
- **微调**：数周起步；主要成本是数据标注、训练、评估、模型托管、版本回滚、持续重训。

## 3. 决策树：从需求到方案

```text
输入：一个“模型效果不好”的需求

1. 是否主要因为模型缺少业务知识、私有文档、最新信息？
   ├─ 是 → 先做 RAG
   │      ├─ 检索能找到正确证据但回答风格差 → RAG + Prompt
   │      └─ 检索能找到正确证据但输出格式长期不稳 → RAG + 轻量微调
   └─ 否 → 继续

2. 是否可以通过更清晰的指令、few-shot、schema、工具调用解决？
   ├─ 是 → 先做 Prompt / few-shot
   └─ 否 → 继续

3. 是否是稳定、重复、高频的行为/风格/格式任务？
   ├─ 是 → 考虑微调
   └─ 否 → 保持 Prompt，不要过早工程化

4. 是否已有可复用 eval set 和高质量训练数据？
   ├─ 是 → 做微调实验，并与 Prompt/RAG baseline 对比
   └─ 否 → 先补 evals 和数据，不要开训
```

更工程化的决策表：

| 判断问题 | 如果答案是“是” | 推荐动作 |
| --- | --- | --- |
| 答案必须来自内部文档、数据库或实时状态吗？ | 是 | RAG / tool calling |
| 用户需要 citation、出处、审计链路吗？ | 是 | RAG，返回 chunk id / URL / 行号 |
| 文档每周甚至每天变化吗？ | 是 | RAG，不要把知识微调进参数 |
| 输出格式需要长期稳定，如 JSON、SQL、标签？ | 是 | 先 Prompt + schema；不稳再 SFT / LoRA |
| 需要固定语气、品牌话术、专家风格？ | 是 | few-shot 起步；高频场景可微调 |
| 只是想减少长 prompt 成本？ | 可能 | 先算 token 成本；高频稳定任务可 LoRA / SFT |
| 数据少于 100 条且质量一般？ | 是 | 不要微调；先写 prompt、补数据、建 evals |
| 无法衡量“变好”还是“变坏”？ | 是 | 先建 evals，见 [LLM 应用评估](05-evals-for-llm-apps.md) |

## 4. 先用数据决定：不要凭感觉选路径

三条路径都要对同一套 eval set 跑分。一个最小评估闭环：

1. **定义任务**：例如“从客服对话中抽取退款原因和责任方”。
2. **收集 50–200 条代表性输入**：覆盖短文本、长文本、脏数据、多语言、边界情况。
3. **写 gold answer**：最好由业务专家给出标准答案或评分标准。
4. **跑 baseline**：最简单 prompt 的结果。
5. **逐步实验**：Prompt v2、RAG、微调模型分别跑同一套 eval。
6. **比较指标**：准确率、格式合法率、拒答正确率、citation 命中率、延迟、成本。

建议记录表：

| 实验 | 方法 | Accuracy | JSON valid | Citation hit | P95 latency | Cost / 1k req | 结论 |
| --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| baseline | zero-shot prompt | 72% | 86% | N/A | 1.2s | $0.40 | 可用但不稳 |
| prompt-v2 | few-shot + schema | 84% | 98% | N/A | 1.6s | $0.55 | 先上线 |
| rag-v1 | top-5 retrieval | 88% | 97% | 76% | 2.4s | $0.80 | 适合知识问答 |
| lora-v1 | 1k SFT samples | 91% | 99% | N/A | 1.3s | $0.35 | 适合高频抽取 |

如果没有 evals，任何“微调后更聪明了”的说法都不可信。evals 的设计方法见 [LLM 应用评估](05-evals-for-llm-apps.md)。

## 5. 路径一：Prompt / few-shot——最快的可逆实验

Prompt 适合以下情况：

- 任务是通用能力：总结、改写、翻译、解释、轻量分类。
- 输入上下文较短，不需要查大型知识库。
- 输出要求可以用明确结构、few-shot、JSON schema 或函数调用约束。
- 需求还在探索期，未来可能频繁改规则。

### 5.1 Prompt 快速胜利清单

参考 [Prompt 工程模式](04-prompt-engineering-patterns.md)，先做这些：

1. **把任务拆清楚**：目标、输入、输出、约束、反例。
2. **给 3–5 个 few-shot 示例**：覆盖普通、边界、失败样例。
3. **输出结构化**：JSON / Markdown table / XML tags，而不是自然语言随意发挥。
4. **把知识与规则分开**：规则写 prompt，动态知识走 RAG 或 tool。
5. **增加自检步骤**：让模型在输出前验证 schema、字段、引用是否存在。

### 5.2 可直接套用的 few-shot 模板

```text
System:
你是一个严格的信息抽取器。只根据用户输入抽取字段，不要猜测。
如果字段不存在，输出 null。必须返回合法 JSON，不要输出 Markdown。

User:
任务：从客服对话中抽取退款信息。

字段：
- refund_reason: string | null
- responsible_party: "merchant" | "customer" | "logistics" | "unknown"
- confidence: number, 0 到 1

示例 1:
输入：用户说“快递把杯子摔碎了，我要退款。”
输出：{"refund_reason":"商品运输破损","responsible_party":"logistics","confidence":0.92}

示例 2:
输入：用户说“我不想要了，七天无理由可以退吗？”
输出：{"refund_reason":"用户主观退货","responsible_party":"customer","confidence":0.86}

现在处理：
输入：{{conversation}}
输出：
```

完成标准：

- 在 eval set 上 JSON 合法率 ≥ 98%。
- 关键字段准确率比 zero-shot 至少提升 5–10 个百分点。
- prompt 的 token 成本仍可接受。

### 5.3 什么时候 Prompt 不够

出现以下症状时，Prompt 可能不是最终答案：

| 症状 | 更可能的路径 |
| --- | --- |
| 模型不知道公司最新政策 | RAG |
| 每次都要塞 20 页规则导致成本高 | RAG 或微调 |
| 少数格式错误造成下游系统失败 | 先 schema/validator；仍不稳再微调 |
| 需要稳定模仿品牌语气，且调用量很高 | few-shot → 微调 |
| 需要引用证据、可审计 | RAG |

## 6. 路径二：RAG——当答案藏在外部知识里

RAG 适合“模型不是不会推理，而是没有正确材料”的场景。典型需求：

- 企业知识库问答：制度、SOP、FAQ、合同、产品文档。
- 内容经常更新：每周变更价格、政策、API 文档。
- 必须给出处：链接、段落、文件名、版本号、时间戳。
- 需要权限控制：不同用户只能检索不同文档。

### 6.1 最小 RAG 执行步骤

完整架构见 [RAG 架构](03-rag-architecture.md)，这里给执行顺序：

1. **准备文档**：PDF/HTML/Markdown/数据库记录统一转成文本。
2. **切 chunk**：常见起点是 300–800 tokens，overlap 50–100 tokens。
3. **写 metadata**：`source`, `url`, `updated_at`, `owner`, `acl`, `section`。
4. **生成 embedding**：使用 OpenAI text-embedding-3-small、bge-m3、e5-large 等。
5. **入库**：pgvector、Qdrant、Milvus、Weaviate、Pinecone 都可以。
6. **检索**：top-k 先设 5–10；必要时加 BM25 hybrid search。
7. **rerank**：用 cross-encoder 或 hosted reranker 提升证据排序。
8. **生成**：把证据块放入 prompt，要求“只基于证据回答，不足则拒答”。
9. **评估**：answer correctness + citation correctness + refusal correctness。

### 6.2 RAG prompt 模板

```text
System:
你是企业知识库助手。只能根据 <context> 中的证据回答。
如果证据不足，回答“资料中没有足够信息”，并说明缺少什么。
每个关键结论后必须给出 citation，格式为 [source:chunk_id]。

User:
问题：{{question}}

<context>
{{retrieved_chunks}}
</context>

请输出：
1. 直接答案
2. 依据列表
3. 如果不确定，列出需要补充的资料
```

### 6.3 RAG 最小伪代码

```python
from openai import OpenAI

client = OpenAI()

def answer_with_rag(question: str, vector_store) -> str:
    query_emb = client.embeddings.create(
        model="text-embedding-3-small",
        input=question,
    ).data[0].embedding

    chunks = vector_store.search(query_emb, top_k=5)
    context = "\n\n".join(
        f"[{c['source']}:{c['chunk_id']}]\n{c['text']}"
        for c in chunks
    )

    messages = [
        {
            "role": "system",
            "content": "只能根据提供的 context 回答；证据不足则拒答；关键结论必须带 citation。",
        },
        {
            "role": "user",
            "content": f"问题：{question}\n\n<context>\n{context}\n</context>",
        },
    ]
    return client.chat.completions.create(
        model="gpt-4.1-mini",
        messages=messages,
        temperature=0,
    ).choices[0].message.content
```

完成标准：

- 检索 top-5 recall 达到业务可接受线，例如 ≥ 85%。
- 有答案问题的 citation 命中率 ≥ 80%。
- 无答案问题的拒答正确率 ≥ 90%。
- 文档更新后无需重训模型，只需重建相关索引。

### 6.4 RAG 不是万能药

RAG 解决“给模型正确材料”，不保证模型自动有好行为。常见补强：

- 回答风格不一致：用 prompt 的 role / tone / few-shot。
- 长文档检索不准：改 chunk、metadata filter、hybrid search、reranker。
- 引用错：让模型只引用 chunk id，并在后处理验证引用是否出现在检索结果中。
- 格式不稳：用 JSON schema / function calling；高频任务再考虑微调。

## 7. 路径三：微调——改变模型的默认行为

微调适合以下条件同时成立：

- 任务高频、稳定、重复，值得把行为固化进模型。
- 你有高质量样本，而不是“随便导出一堆聊天记录”。
- Prompt / few-shot 已经接近上限，继续加 prompt 会变贵、变慢、变脆。
- 已有 eval set，能证明微调比 baseline 更好。

### 7.1 微调不适合什么

| 不适合的需求 | 原因 | 替代方案 |
| --- | --- | --- |
| 注入最新政策、产品库存、价格 | 参数更新慢，重训成本高 | RAG / tool calling |
| 让模型“少幻觉” | 幻觉常来自无证据生成，不是缺少风格 | RAG + citation + refusal eval |
| 只有十几条样本 | 数据量太少，容易过拟合 | few-shot |
| 需求每周变 | 微调迭代慢 | prompt 配置化 |
| 没有评估集 | 无法判断是否退化 | 先建 evals |

### 7.2 SFT、LoRA/QLoRA、DPO/RLHF 怎么选

| 方法 | 训练数据 | 解决什么 | 适用场景 | 风险 |
| --- | --- | --- | --- | --- |
| **SFT（Supervised Fine-Tuning）** | `prompt → ideal completion` 或 chat messages | 学习标准答案、格式、风格 | 抽取、分类、客服话术、代码模板 | 数据差会直接学坏 |
| **LoRA** | 同 SFT | 只训练低秩 adapter，便宜、可切换 | 中小团队最常用的开源微调 | adapter 与 base model 版本强绑定 |
| **QLoRA** | 同 SFT | 4-bit 量化加载 base model，再训练 LoRA | 显存有限，如单卡 24GB | 训练速度较慢，量化配置要测 |
| **DPO（Direct Preference Optimization）** | `prompt + chosen + rejected` | 学习“更喜欢哪种答案” | 有成对偏好反馈，想改善语气/安全/排序 | 不能替代知识注入 |
| **RLHF** | reward model + policy optimization | 大规模偏好对齐 | 平台级模型训练 | 工程复杂，不是普通应用首选 |

实战建议：

- 第一轮微调优先选 **SFT + LoRA**。
- 显存紧张选 **QLoRA**。
- 当 SFT 已能完成任务，但“哪个答案更好”难用单一标准答案表达，再考虑 **DPO**。

## 8. 微调数据：格式、质量、数量级

### 8.1 Chat SFT JSONL 格式

每行是一条训练样本：

```json
{"messages":[{"role":"system","content":"你是一个严格的客服质检分类器，只输出 JSON。"},{"role":"user","content":"客户说：物流三天没更新，我要投诉。"},{"role":"assistant","content":"{\"category\":\"logistics_complaint\",\"priority\":\"medium\",\"needs_human\":false}"}]}
```

保存为 `data/train.jsonl`，验证集保存为 `data/valid.jsonl`。

### 8.2 Instruction SFT JSONL 格式

部分开源训练脚本使用 `instruction/input/output`：

```json
{"instruction":"把客服对话分类为 refund、logistics、quality、other，并输出 JSON。","input":"用户：杯子裂了，拍照给你们看。","output":"{\"category\":\"quality\",\"confidence\":0.91}"}
```

训练前通常会转成 chat template：

```python
def to_messages(row):
    return {
        "messages": [
            {"role": "system", "content": "你是一个严格的客服质检分类器，只输出 JSON。"},
            {"role": "user", "content": row["instruction"] + "\n\n" + row["input"]},
            {"role": "assistant", "content": row["output"]},
        ]
    }
```

### 8.3 DPO 偏好数据格式

DPO 需要 chosen / rejected：

```json
{"prompt":"请用公司客服语气回复：用户收到破损商品。","chosen":"非常抱歉给您带来不便。请您上传破损照片，我们会优先为您补发或退款。","rejected":"东西坏了就退吧，发照片。"}
```

适合训练“哪种回答更符合偏好”，不适合记住“最新退款政策第 17 条”。

### 8.4 数据量级经验

| 目标 | 可尝试数量 | 更稳数量 | 备注 |
| --- | ---: | ---: | --- |
| 固定输出格式 / 标签分类 | 100–300 条 | 1k–5k 条 | 样本覆盖边界比数量更重要 |
| 品牌语气 / 客服话术 | 300–1k 条 | 3k–10k 条 | 需要反例和高质量改写 |
| 复杂领域任务 | 1k–5k 条 | 10k+ 条 | 先确认 RAG 是否更适合 |
| DPO 偏好优化 | 500–2k 对 | 5k+ 对 | chosen/rejected 差异要清晰 |

数据质量规则：

- 去重：重复样本会让模型过度偏向模板化输出。
- 覆盖边界：空输入、冲突信息、无答案、用户诱导、脏格式都要有。
- 输出一致：同一标签体系不能前后写法不同，如 `refund` 与 `Refund` 混用。
- 拆分验证集：至少 10–20% 留作 validation，不参与训练。
- 保留 holdout eval：最终评估集不要在调参过程中反复看答案。

## 9. 最小 LoRA 微调流程：Hugging Face PEFT + TRL

下面示例面向单机 GPU。模型用占位的开源 chat model，你可以替换为团队允许使用的 base model，例如 Qwen、Llama、Mistral 系列。生产前确认模型 license。

### 9.1 环境准备

```powershell
cd D:\projects\your-finetune-lab
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install "torch" "transformers>=4.44.0" "datasets" "accelerate" "peft" "trl" "bitsandbytes" "safetensors"
```

如果 Windows 上 `bitsandbytes` 不可用，建议在 Linux GPU 机器或 WSL2 CUDA 环境运行训练；Windows 仍可用于准备数据和跑 API 微调。

### 9.2 准备数据

目录：

```text
finetune-lab/
  data/
    train.jsonl
    valid.jsonl
  train_lora.py
  eval_json_valid.py
```

`data/train.jsonl` 示例：

```json
{"messages":[{"role":"system","content":"你是一个严格的客服质检分类器，只输出 JSON。"},{"role":"user","content":"客户：快递一直没到，物流停在三天前。"},{"role":"assistant","content":"{\"category\":\"logistics\",\"priority\":\"medium\",\"needs_human\":false}"}]}
{"messages":[{"role":"system","content":"你是一个严格的客服质检分类器，只输出 JSON。"},{"role":"user","content":"客户：杯子碎了，我要退款。"},{"role":"assistant","content":"{\"category\":\"quality\",\"priority\":\"high\",\"needs_human\":true}"}]}
```

真实训练请至少准备数百条，并保证 validation 中有训练集没有出现过的表达。

### 9.3 训练脚本：`train_lora.py`

```python
import torch
from datasets import load_dataset
from peft import LoraConfig
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from trl import SFTConfig, SFTTrainer

MODEL_NAME = "Qwen/Qwen2.5-1.5B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME, trust_remote_code=True)
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

def format_example(example):
    return tokenizer.apply_chat_template(
        example["messages"],
        tokenize=False,
        add_generation_prompt=False,
    )

dataset = load_dataset(
    "json",
    data_files={
        "train": "data/train.jsonl",
        "validation": "data/valid.jsonl",
    },
)

quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
)

model = AutoModelForCausalLM.from_pretrained(
    MODEL_NAME,
    quantization_config=quantization_config,
    device_map="auto",
    trust_remote_code=True,
)

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
)

training_args = SFTConfig(
    output_dir="outputs/qwen-classifier-lora",
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    learning_rate=2e-4,
    num_train_epochs=3,
    logging_steps=10,
    eval_strategy="steps",
    eval_steps=50,
    save_steps=50,
    max_seq_length=1024,
    packing=False,
    bf16=torch.cuda.is_available(),
    report_to="none",
)

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["validation"],
    peft_config=lora_config,
    formatting_func=format_example,
    tokenizer=tokenizer,
)

trainer.train()
trainer.save_model("outputs/qwen-classifier-lora/final")
tokenizer.save_pretrained("outputs/qwen-classifier-lora/final")
```

运行：

```powershell
accelerate config
accelerate launch train_lora.py
```

预期结果：

- `outputs/qwen-classifier-lora/final` 下出现 adapter 文件，如 `adapter_model.safetensors`。
- training loss 下降，但不要只看 loss；最终以 eval set 指标为准。

### 9.4 推理与评估

最小推理脚本：

```python
import json
import torch
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer

BASE_MODEL = "Qwen/Qwen2.5-1.5B-Instruct"
ADAPTER = "outputs/qwen-classifier-lora/final"

tokenizer = AutoTokenizer.from_pretrained(BASE_MODEL, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(
    BASE_MODEL,
    torch_dtype=torch.bfloat16 if torch.cuda.is_available() else torch.float32,
    device_map="auto",
    trust_remote_code=True,
)
model = PeftModel.from_pretrained(model, ADAPTER)
model.eval()

def classify(text: str) -> dict:
    messages = [
        {"role": "system", "content": "你是一个严格的客服质检分类器，只输出 JSON。"},
        {"role": "user", "content": text},
    ]
    prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    with torch.no_grad():
        output = model.generate(
            **inputs,
            max_new_tokens=128,
            temperature=0,
            do_sample=False,
        )
    generated = output[0][inputs["input_ids"].shape[-1]:]
    raw = tokenizer.decode(generated, skip_special_tokens=True).strip()
    return json.loads(raw)

print(classify("客户：我收到的杯子裂开了，麻烦处理退款。"))
```

评估脚本核心逻辑：

```python
import json

def is_valid_json(text: str) -> bool:
    try:
        json.loads(text)
        return True
    except json.JSONDecodeError:
        return False

def exact_category(pred: dict, gold: dict) -> bool:
    return pred.get("category") == gold.get("category")
```

至少记录：

- `json_valid_rate`
- `category_accuracy`
- `priority_accuracy`
- `needs_human_f1`
- `p50/p95 latency`
- `tokens/request`

### 9.5 合并 adapter 与服务

如果需要把 LoRA adapter 合并到 base model：

```python
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer

base = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-1.5B-Instruct",
    device_map="auto",
    trust_remote_code=True,
)
model = PeftModel.from_pretrained(base, "outputs/qwen-classifier-lora/final")
merged = model.merge_and_unload()
merged.save_pretrained("outputs/qwen-classifier-merged", safe_serialization=True)

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-1.5B-Instruct", trust_remote_code=True)
tokenizer.save_pretrained("outputs/qwen-classifier-merged")
```

用 vLLM 服务：

```powershell
pip install vllm
python -m vllm.entrypoints.openai.api_server --model outputs/qwen-classifier-merged --port 8000
```

调用：

```powershell
curl http://localhost:8000/v1/chat/completions `
  -H "Content-Type: application/json" `
  -d "{\"model\":\"outputs/qwen-classifier-merged\",\"messages\":[{\"role\":\"user\",\"content\":\"客户说杯子碎了要退款\"}],\"temperature\":0}"
```

上线前必须做：

- 与 Prompt baseline 跑同一套 eval。
- 与 base model 跑同一套 eval。
- 加入 canary：抽 1–5% 流量观察格式合法率、人工升级率、投诉率。
- 保留回滚：base model + prompt 方案必须随时可切回。

## 10. Hosted API 微调最小流程：OpenAI 示例

如果团队不想维护训练集群，可以使用 hosted fine-tuning。示例格式：

`train.jsonl`：

```json
{"messages":[{"role":"system","content":"你是一个严格的客服质检分类器，只输出 JSON。"},{"role":"user","content":"客户：快递一直没到，物流停在三天前。"},{"role":"assistant","content":"{\"category\":\"logistics\",\"priority\":\"medium\",\"needs_human\":false}"}]}
```

上传并训练：

```powershell
pip install --upgrade openai
$env:OPENAI_API_KEY="sk-..."
openai files create --purpose fine-tune --file data\train.jsonl
openai files create --purpose fine-tune --file data\valid.jsonl
openai fine_tuning.jobs.create `
  -m gpt-4.1-mini-2025-04-14 `
  -t file-TRAIN_ID `
  --validation-file file-VALID_ID
```

查看任务：

```powershell
openai fine_tuning.jobs.retrieve -i ftjob-...
```

调用微调模型：

```python
from openai import OpenAI

client = OpenAI()
resp = client.chat.completions.create(
    model="ft:gpt-4.1-mini-2025-04-14:org:classifier:abc123",
    messages=[
        {"role": "system", "content": "你是一个严格的客服质检分类器，只输出 JSON。"},
        {"role": "user", "content": "客户：杯子碎了，我要退款。"},
    ],
    temperature=0,
)
print(resp.choices[0].message.content)
```

注意：

- 具体可微调模型名会变化，以上命令以官方文档当前支持列表为准。
- 不要上传敏感数据；必要时做脱敏、权限审批、数据保留策略。
- hosted 微调也需要 validation 与 holdout eval，不是“提交训练就完事”。

## 11. 成本、延迟、维护对比

| 维度 | Prompt / few-shot | RAG | 微调 |
| --- | --- | --- | --- |
| 初始成本 | 最低，主要是 prompt 和 eval | 中，需要文档管道、向量库、检索调参 | 最高，需要数据、训练、评估、部署 |
| 迭代速度 | 分钟到小时 | 小时到天 | 天到周 |
| 单次延迟 | 低到中；长 prompt 会变慢 | 中到高；多一次检索/rerank | 低到中；可减少 prompt token |
| 知识更新 | 改 prompt 或输入上下文 | 更新索引即可 | 通常要重训，不适合频繁知识变化 |
| 输出风格稳定性 | 中，依赖 prompt | 中，仍依赖 prompt | 高，适合固化行为 |
| 可解释性 | 低到中 | 高，可返回 citation | 低，参数内行为难解释 |
| 运维复杂度 | 低 | 中，高质量 RAG 很吃工程 | 高，涉及模型版本、adapter、回滚 |
| 最适合 | 探索期、低频任务、通用能力 | 知识问答、可追溯答案 | 高频稳定任务、格式/风格一致性 |

一个粗略成本公式：

```text
月成本 = 请求量 × (输入 token × 输入单价 + 输出 token × 输出单价)
       + RAG 检索/存储成本
       + 微调训练/托管成本
       + 人工标注与评估成本
```

不要只比较推理 token。微调可能降低每次请求 token，但增加训练、运维、回滚成本。RAG 可能增加延迟，但减少知识维护和错误回答风险。

## 12. 组合策略：不要把三者当互斥选项

真实系统经常组合：

### 12.1 Prompt + RAG

适用：企业知识库助手。

```text
用户问题 → 查询改写 prompt → 检索 → rerank → 带 citation 的回答 prompt → 后处理校验
```

价值：

- RAG 提供事实。
- Prompt 规定回答结构、拒答策略、引用格式。

### 12.2 Prompt + 微调

适用：高频结构化抽取。

```text
短 system prompt + 微调模型 → JSON 输出 → validator → fallback 到强模型
```

价值：

- 微调让格式更稳。
- Prompt 保留少量可配置规则。

### 12.3 RAG + 轻量微调

适用：既需要最新知识，又需要固定行为。

例子：售后政策助手。

- RAG 检索最新政策条款。
- LoRA 微调客服语气、拒答方式、升级人工的判断风格。
- eval 同时测 citation correctness 和 tone compliance。

典型 prompt：

```text
你必须基于 <policy_context> 回答，不得编造政策。
你的语气应符合客服规范：先致歉，再给步骤，再说明时效。
如果政策没有覆盖，输出 {"needs_human": true, "reason": "..."}。
```

## 13. 常见坑 & 排查表

| 坑 | 症状 | 根因 | 排查/解决 |
| --- | --- | --- | --- |
| 用微调解决知识新鲜度 | 新政策仍答错，重训后过几周又错 | 知识应该在外部系统更新 | 改 RAG；索引记录 `updated_at`；答案带 citation |
| 数据太少就开训 | 训练集表现好，真实流量崩 | 过拟合，样本覆盖不足 | 扩充到数百/数千条；加 validation 和 holdout |
| 无 eval 盲目微调 | 团队争论“感觉更好” | 没有量化标准 | 先建 50–200 条 eval；定义指标 |
| 把脏日志直接当训练集 | 模型学会错误语气、错标签 | 数据未清洗 | 人审样本；统一标签；删除低质和冲突样本 |
| RAG chunk 太大 | 检索返回长段落但答案抓不到关键句 | embedding 被噪声稀释 | 降到 300–800 tokens；加标题和 metadata |
| RAG chunk 太小 | 答案缺上下文 | 上下文被切断 | 增加 overlap；按标题层级切分 |
| Prompt 越写越长 | 成本高、延迟高、规则互相冲突 | 没有抽象规则和示例 | 合并规则；用 few-shot；稳定高频部分考虑微调 |
| 微调后通用能力下降 | 非目标任务变差 | 训练分布太窄或学习率过高 | 降学习率；混入通用保持数据；只在目标路由使用 |
| JSON 仍偶尔坏 | 下游解析失败 | 生成式模型无法 100% 保证格式 | schema constrained decoding、validator、retry、fallback |
| DPO 被当成 SFT | 模型偏好变了但任务不会做 | DPO 需要已有可接受能力 | 先 SFT，再 DPO |

## 14. 上线检查清单

### 14.1 Prompt 检查

- [ ] system prompt 不超过必要长度，规则无冲突。
- [ ] 有 3–5 个覆盖边界的 few-shot。
- [ ] 输出格式有 schema 或解析器验证。
- [ ] eval set 上指标超过 baseline。
- [ ] 有 fallback：解析失败、置信度低、证据不足时怎么处理。

### 14.2 RAG 检查

- [ ] 文档清洗流程可重复运行。
- [ ] chunk 有 `source/chunk_id/updated_at/acl` metadata。
- [ ] top-k recall、citation hit、refusal correctness 已量化。
- [ ] 检索结果受权限控制。
- [ ] 文档更新后索引能增量刷新。
- [ ] 回答中的 citation 会被后处理验证。

### 14.3 微调检查

- [ ] 已证明 Prompt / RAG baseline 不够。
- [ ] 训练集、validation、holdout eval 三者分离。
- [ ] 样本数量和覆盖面满足目标任务。
- [ ] 训练数据无敏感信息或已脱敏。
- [ ] 与 base model 和 prompt baseline 跑同一套 eval。
- [ ] 有模型版本、adapter 版本、数据版本记录。
- [ ] 有 canary、监控、回滚方案。

## 15. 动手练习：同一任务跑三条路径并对比

任务：构建“客服对话分类器”，输入一段对话，输出：

```json
{
  "category": "refund | logistics | quality | other",
  "priority": "low | medium | high",
  "needs_human": true
}
```

### 15.1 Deliverable A：Prompt baseline

- 写一个 zero-shot prompt。
- 写一个 few-shot prompt。
- 准备 50 条 eval case。
- 输出表：`json_valid_rate`, `category_accuracy`, `priority_accuracy`。

通过标准：

- few-shot 比 zero-shot 至少提升一个关键指标。
- JSON 合法率 ≥ 95%。

### 15.2 Deliverable B：RAG 版本

假设分类规则来自一份内部 SOP：

- 把 SOP 切成 chunk。
- 建立向量索引。
- 检索 top-5 SOP 段落。
- 回答中返回引用，如 `[sop-refund:003]`。

通过标准：

- 对“规则依赖型 case”，RAG 版本准确率高于 prompt-only。
- citation 指向的 chunk 真能支持结论。

### 15.3 Deliverable C：微调版本

- 准备至少 300 条训练样本、60 条 validation、100 条 holdout。
- 用 LoRA 或 hosted API 跑 SFT。
- 与 Prompt baseline 跑同一套 holdout。

通过标准：

- JSON 合法率 ≥ 99%。
- 目标指标比 few-shot 提升 ≥ 3 个百分点，或在同等指标下降低 token 成本 ≥ 30%。
- 记录失败样例并决定下一轮是补数据、改 prompt、改 RAG，还是不继续微调。

### 15.4 最终实验报告模板

```markdown
# LLM customization experiment

## Task
- 输入：
- 输出：
- 业务风险：

## Dataset
- train:
- validation:
- holdout:
- edge cases:

## Experiments
| id | method | model | accuracy | JSON valid | latency | cost | notes |
| --- | --- | --- | ---: | ---: | ---: | ---: | --- |

## Decision
- 采用方案：
- 不采用方案：
- 原因：
- 回滚方案：
```

## 16. 最终选择建议

按下面顺序推进：

1. **先 Prompt**：最快拿到 baseline，暴露任务定义问题。
2. **知识不足就 RAG**：只要答案依赖私有/实时/可追溯资料，就不要先微调。
3. **行为稳定且高频才微调**：格式、风格、标签、话术可以通过 SFT/LoRA 固化。
4. **先 evals 后训练**：每一轮实验都要能回答“提升了多少、成本变了多少、失败在哪里”。
5. **组合而不是站队**：Prompt 负责可配置规则，RAG 负责知识，微调负责稳定行为。

## 17. 延伸阅读

- [Hugging Face PEFT documentation](https://huggingface.co/docs/peft/index)
- [Hugging Face TRL documentation](https://huggingface.co/docs/trl/index)
- [Hugging Face Transformers chat templating](https://huggingface.co/docs/transformers/chat_templating)
- [OpenAI fine-tuning guide](https://platform.openai.com/docs/guides/fine-tuning)
- [OpenAI evals and model optimization docs](https://platform.openai.com/docs/guides/evals)
- [Qdrant RAG documentation](https://qdrant.tech/documentation/rag/)
- [vLLM serving documentation](https://docs.vllm.ai/)

## 小结

把三者当**互补层**而非竞争项：Prompt 打底、RAG 供知识、微调定行为。决策时先问“这是知识问题还是行为问题”，再按 `Prompt < RAG < 微调` 的成本顺序推进。每一步都用同一套 evals 验证收益；如果指标没有显著提升，就不要把复杂度带进生产。


`标签` `微调` `RAG` `Prompt` `决策` `LLM` `成本` `LoRA` `SFT` `DPO` `Evals`

---

[← 上一章](06-agent-architecture-patterns.md) · [WP-01 目录](README.md) · [下一章 →](08-llm-cost-latency-optimization.md)
