[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# LLM 应用架构：从 Prompt 到 Agent

大多数 LLM 应用可归为四种范式，复杂度递增。**先用能满足需求的最简范式**。

## 范式一：直接调用（Prompt → Completion）

最简单：给 prompt，拿输出。适合翻译、摘要、分类、改写。

```ts
const res = await llm.complete({
  system: "你是严格的分类器，只输出 JSON。",
  user: `把邮件分类为 {work, personal, spam}：\n${email}`,
  temperature: 0,
});
```

**工程要点**：低温、明确输出 schema、给 few-shot 例子、对输出做校验与重试。

## 范式二：RAG（检索增强生成）

当答案依赖**私有/最新/大量**知识时，先检索再生成。

```
query → embed → 向量检索 top-k → 拼进 prompt → 生成（带引用）
```

**工程要点**：切分粒度、混合检索（关键词 + 向量）、重排序、把「来源」带回答案里以便核查。详见 [RAG 架构入门](/knowledge/rag-architecture)。

## 范式三：工具调用（Function Calling）

让模型**决定**调用哪个函数、传什么参数，由你的代码执行。

```ts
const tools = [{
  name: "get_weather",
  parameters: { type: "object", properties: { city: { type: "string" } } },
}];
const call = await llm.complete({ user: "北京天气？", tools });
if (call.toolCall) {
  const result = await getWeather(call.toolCall.args.city);
  // 把 result 回传给模型生成最终回答
}
```

**工程要点**：工具描述要精确；参数用 schema 约束；对副作用工具（写操作）加确认与幂等。

## 范式四：Agent 循环（Plan → Act → Observe）

多步任务：模型在「思考 → 调用工具 → 观察结果」间循环，直到完成。

```ts
let context = [userGoal];
for (let step = 0; step < MAX_STEPS; step++) {
  const decision = await llm.decide(context, tools);
  if (decision.done) return decision.answer;
  const observation = await execute(decision.toolCall);
  context.push(decision, observation);
}
```

**工程要点**：
- **步数上限**与预算护栏，防止死循环烧钱。
- **可观测**：记录每步的 thought/action/observation，便于调试。
- **失败恢复**：工具报错时把错误喂回模型让它换策略。
- **确定性优先**：能用普通代码编排的流程，别全交给模型自由发挥。

## MCP：标准化工具与资源接入

Model Context Protocol 把「工具、资源、提示」抽象成标准接口，让不同客户端/模型复用同一批能力。价值在于**解耦**：工具提供方与模型使用方各自演进。

## 选型决策树

```
需要外部/私有知识？ ──否──> 直接调用
        │是
        v
需要执行动作/多步？ ──否──> RAG
        │是
        v
任务步骤固定？ ──是──> 工具调用 + 你写编排
        │否
        v
     Agent 循环
```

## 通用护栏（所有范式都需要）

- 输入/输出**校验**（schema、长度、注入检测）
- **成本控制**：缓存、模型分级（便宜模型做简单活）、限流
- **评估**：离线 eval 集 + 线上采样人评
- **可观测**：日志、trace、token 计量

> 相关：[AI 工程师能力地图与学习路线](/knowledge/ai-engineer-roadmap)。


`标签` `LLM` `Agent` `Prompt` `架构` `Function Calling`

---

[← 上一章](01-ai-engineer-roadmap.md) · [WP-01 目录](README.md) · [下一章 →](03-rag-architecture.md)
