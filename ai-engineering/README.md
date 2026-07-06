# WP-01 · AI 工程与转型白皮书

> 资深软件工程师构建生产级 LLM 应用的能力体系与方法论

**版本** v1.0 · **更新** 2026-07-06

## 摘要

面向已有扎实工程能力、计划在 12 个月内成为能独立交付 AI 应用的资深工程师，系统阐述 LLM 应用的能力地图、架构范式（RAG / Agent）、Prompt 工程、评估体系、定制路径（Prompt/RAG/微调）与成本延迟优化，形成一套可落地、可迭代、可度量的工程方法论。

## 执行摘要

对资深软件工程师而言，**AI 工程 ≈ 系统工程 + 概率直觉 + LLM 应用范式**，无需先啃完深度学习理论。本白皮书的核心论点是：AI 应用的竞争力不在"更聪明的模型"，而在**围绕模型的工程纪律**——结构化的 Prompt、可靠的检索、严格的评估、受约束的 Agent、以及对成本与延迟的持续治理。

我们主张三条基本原则：(1) **评估先行**——没有黄金集与可信指标就没有可迭代的产品；(2) **知识用 RAG、行为用微调、其余用 Prompt**——按成本从低到高选择定制路径；(3) **约束 Agent 而非放任其"更自主"**——清晰的工具、硬性的停止条件、代码层护栏与可回放的可观测性。沿着本白皮书的章节推进，读者可在 6–12 个月内建立从能力地图到生产落地的完整闭环。

## 关键要点

- AI 应用 90% 的工作在 LLM 应用层（Prompt / 检索 / 评估 / 编排），而非模型训练。
- 评估体系（Evals）是 LLM 团队成熟度的核心标志，等价于单元测试之于传统软件。
- 定制路径优先级：先 Prompt，再 RAG，最后才微调；知识问题永远先用 RAG 而非微调。
- Agent 的工程难点在于约束它：工具设计、停止条件、代码层护栏、可观测性。
- 成本与延迟优化按『别调用 → 用对模型 → 调短 → 调快 → 改基座』的顺序推进。

## 章节

1. [AI 工程师能力地图与学习路线](01-ai-engineer-roadmap.md)
2. [LLM 应用架构：从 Prompt 到 Agent](02-llm-app-architecture.md)
3. [RAG 架构入门：从检索到生成](03-rag-architecture.md)
4. [LLM 应用的 Prompt 工程模式手册](04-prompt-engineering-patterns.md)
5. [LLM 应用的评估（Evals）实战](05-evals-for-llm-apps.md)
6. [LLM Agent 架构模式：从工具调用到多智能体](06-agent-architecture-patterns.md)
7. [微调 vs RAG vs Prompt：如何选对 LLM 定制路径](07-finetune-vs-rag-vs-prompt.md)
8. [LLM 应用的成本与延迟优化](08-llm-cost-latency-optimization.md)

## 参考文献

1. [roadmap.sh · AI Engineer 路线图](https://roadmap.sh/ai-engineer)
2. [Prompt Engineering Guide](https://www.promptingguide.ai)
3. [OpenAI Platform Docs](https://platform.openai.com/docs)
4. [Hugging Face](https://huggingface.co)
5. [LangChain 文档](https://python.langchain.com)
6. [Microsoft · RAG 设计模式](https://learn.microsoft.com/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide)

---

[← 返回手册首页](../README.md)
