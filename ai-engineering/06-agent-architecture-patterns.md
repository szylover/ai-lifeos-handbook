[← 手册首页](../README.md) · [WP-01 AI 工程](README.md)

---

# LLM Agent 架构模式：从工具调用到多智能体

> 结论先行：**Agent = LLM + 工具 + 循环 + 记忆 + 停止条件**。90% 的"Agent 需求"其实用一个带工具调用的确定性工作流就够了；只有当步骤数不可预知、需要动态决策时，才真正需要自主循环。

相关阅读：[Prompt 工程模式](04-prompt-engineering-patterns.md)、[LLM 应用评估](05-evals-for-llm-apps.md)、[LLM 应用架构](02-llm-app-architecture.md)、[RAG 架构](03-rag-architecture.md)。

## 1. 本章定位：建一个受约束的执行系统

Agent 的价值不是让模型自由发挥，而是把模型放进一个可观测、可限速、可恢复、可审计的执行循环。可靠 Agent 至少回答 6 个工程问题：

1. **什么时候行动？** 由 single tool call、ReAct、Plan-and-Execute、Router、Multi-agent 等策略决定。
2. **能做什么？** 由工具白名单和 schema 决定，不由 prompt 口头约束。
3. **什么时候停止？** 由 `MAX_STEPS`、预算、完成条件、重复动作检测、人工接管决定。
4. **如何失败恢复？** 工具错误结构化回灌，循环选择重试、换工具、降级或停止。
5. **如何记忆？** 短期记忆做上下文压缩，长期记忆做向量检索或结构化状态。
6. **如何复盘？** 每一步 thought/action/observation 都记录 trace，可 replay。

### 1.1 前置环境与学习产出

本章代码使用 **Node.js 20+**，无第三方依赖；有 `OPENAI_API_KEY` 时调用 OpenAI-compatible Chat Completions API，没有 key 时使用 deterministic demo planner，方便复制即跑。

```bash
node -v      # 建议 >= v20，内置 fetch
```

学完后你能：

- 从零实现 `plan → act → observe` Agent 核心循环。
- 写出带 JSON Schema 的工具，处理幂等、确认、错误回灌。
- 在 ReAct、Plan-and-Execute、Reflection、Router、Multi-agent 之间取舍。
- 加上步数、预算、超时、循环检测和失败恢复。
- 记录可 replay 的 trace，并接入 LangSmith 或 OpenTelemetry。
- 判断需求应使用 Agent、workflow、RAG 还是确定性代码。

## 2. 心智模型：Agent 是带不确定决策器的状态机

```text
User Goal
  ↓
State = { goal, messages, scratchpad, memory, budget, trace }
  ↓
while not stopped:
  Plan/Think: LLM 根据 goal + state 选择下一步
  Act: 代码层校验 tool schema、权限、预算、确认，再执行工具
  Observe: 把结构化结果/错误写入 state 和 trace
  Check: 是否完成、超步数、超预算、重复、需要人工
  ↓
Final Answer / Escalation / Failure Report
```

普通程序的分支由工程师写死；Agent 的下一步由 LLM 在运行时决定。因此工程重心是：**把可变部分限制在小范围内，把不可变约束放在代码里**。

| 部件 | 由谁控制 | 失败模式 | 工程解法 |
| --- | --- | --- | --- |
| Prompt | 工程师 | 说了但模型不遵守 | 只放策略，不放安全边界 |
| Tool schema | 工程师 | 参数错、语义模糊 | JSON Schema + 运行时校验 |
| Tool execution | 代码 | 副作用、超时、重试风暴 | 幂等 key、dry-run、确认、timeout |
| Loop policy | 代码 + LLM | 死循环、烧钱 | `MAX_STEPS`、预算、重复检测 |
| Memory | 代码 | 上下文爆炸、检索污染 | 摘要、滑窗、向量过滤 |
| Observability | 代码 | 无法复盘 | trace、step log、replay harness |

## 3. Agent 核心循环：从零实现 `plan → act → observe`

先手写一遍循环，再考虑框架。最小状态：

```ts
type AgentState = {
  goal: string;
  messages: Message[];
  observations: Observation[];
  trace: TraceStep[];
  budget: { maxUsd: number; spentUsd: number; maxSteps: number };
  stopReason?: 'done' | 'max_steps' | 'budget' | 'loop_detected' | 'needs_human' | 'error';
};
```

关键：`messages` 给模型看；`trace` 给人和系统复盘。不要把完整 trace 无脑塞回上下文。`stopReason` 必须是枚举，方便 dashboard 聚合。

### 3.1 工具注册表

```ts
type ToolDef = {
  name: string;
  description: string;
  schema: object;
  idempotent: boolean;
  sideEffect: 'none' | 'read' | 'write' | 'external';
  requiresConfirmation: boolean;
  execute: (args: unknown, ctx: ToolContext) => Promise<ToolResult>;
};
```

- `none`：纯计算，如 calculator，可自动重试。
- `read`：读数据库、HTTP GET，可缓存。
- `write`：写库、发邮件、付款草稿，必须有 idempotency key。
- `external`：影响外部世界，如下单、删资源，默认 human-in-the-loop。

### 3.2 循环伪代码

```ts
for step in 1..MAX_STEPS:
  if budget exhausted: stop('budget')
  decision = await model.plan(state, tools)
  if decision.type === 'final': stop('done')
  if repeated(decision): stop('loop_detected')
  if tool requires confirmation and not confirmed: stop('needs_human')
  validatedArgs = validate(decision.args, tool.schema)
  observation = await executeToolWithTimeout(tool, validatedArgs)
  state.observations.push(observation)
  state.trace.push({ thought, action, args, observation, cost, latency })
return failure report with trace summary
```

`thought` 是否记录完整内容取决于供应商政策和公司安全要求；生产系统通常记录 `thought_summary` 或 `decision_rationale`，但 `action/args/observation/error` 必须完整可审计。

## 4. 工具设计：schema、幂等、副作用确认、错误信息

Agent 的上限通常由工具决定。工具描述模糊，模型就会乱用；工具错误不可读，模型就无法自愈。

| 维度 | 坏工具 | 好工具 |
| --- | --- | --- |
| 名称 | `doStuff` | `search_web`, `create_invoice_draft`, `send_invoice` |
| 参数 | `{ input: string }` | 明确字段、类型、枚举、范围、示例 |
| 语义 | "处理用户请求" | "只执行 HTTP GET，不提交表单" |
| 副作用 | 不说明 | 标注 `sideEffect` 与确认要求 |
| 错误 | `failed` | `{ code, retryable, message, details, suggested_fix }` |

### 4.1 精确 schema 示例

```ts
const searchTool = {
  name: 'search_hacker_news',
  description: 'Search public Hacker News stories. Use for recent technical discussions, libraries, papers, and engineering posts. Returns title, url, author, and points. Does not browse arbitrary pages.',
  schema: {
    type: 'object',
    additionalProperties: false,
    required: ['query'],
    properties: {
      query: { type: 'string', minLength: 2, maxLength: 120 },
      limit: { type: 'integer', minimum: 1, maximum: 5, default: 3 }
    }
  }
};
```

Schema 规则：`additionalProperties:false` 拒绝编造字段；字符串给长度上限；数字给范围；枚举替代自由文本；描述写明工具不会做什么。

### 4.2 幂等和副作用确认

有副作用的工具拆成两阶段：先 draft，再 commit。

```ts
const createPaymentDraft = {
  name: 'create_payment_draft',
  sideEffect: 'write',
  requiresConfirmation: false,
  description: 'Create a payment draft only. It does not transfer money.',
  execute: async (args, ctx) => ({
    ok: true,
    data: { draft_id: `draft_${Date.now()}`, status: 'pending_confirmation', args }
  })
};

const confirmPayment = {
  name: 'confirm_payment',
  sideEffect: 'external',
  requiresConfirmation: true,
  description: 'Execute a payment draft. Requires human approval and an idempotency_key.',
  schema: {
    type: 'object',
    additionalProperties: false,
    required: ['draft_id', 'idempotency_key'],
    properties: {
      draft_id: { type: 'string' },
      idempotency_key: { type: 'string', minLength: 12, maxLength: 80 }
    }
  },
  execute: async (args, ctx) => ctx.confirmed
    ? { ok: true, data: { payment_id: 'pay_123', status: 'submitted' } }
    : { ok: false, error: { code: 'CONFIRMATION_REQUIRED', retryable: false, message: 'Human approval required.' } }
};
```

幂等实践：所有 `write/external` 工具参数包含 `idempotency_key`；服务端保存 `idempotency_key → result`；失败续跑复用 key；最终"已完成"必须来自工具返回。

### 4.3 错误要能被模型利用

```ts
type ToolError = {
  code: 'VALIDATION_ERROR' | 'TIMEOUT' | 'RATE_LIMIT' | 'NOT_FOUND' | 'CONFIRMATION_REQUIRED' | 'UPSTREAM_ERROR';
  retryable: boolean;
  message: string;
  details?: unknown;
  suggested_fix?: string;
};
```

例子：

```json
{
  "ok": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "retryable": true,
    "message": "Parameter 'limit' must be between 1 and 5.",
    "details": { "field": "limit", "received": 20 },
    "suggested_fix": "Call search_hacker_news again with limit <= 5."
  }
}
```

不要返回 `Something went wrong`，这会迫使模型猜测。

## 5. 经典模式谱系：什么时候用哪一种

### 5.1 单次工具调用（Tool Use）

适用：步骤固定、只需一次工具结果即可回答。

```text
User: 查订单 123 的物流
LLM: call get_order_tracking({ order_id: "123" })
Code: execute tool
Template: 根据结果生成回答
```

实现要点：不开循环，只允许 0 或 1 次工具调用；最终回答尽量用模板生成；适合客服、订单查询、结构化抽取。**能用它就别上循环。**

### 5.2 ReAct（Reason + Act）

适用：下一步依赖上一步观察，路径不可预知，但工具风险较低。

```ts
async function reactLoop(goal, tools) {
  const state = { goal, observations: [] };
  for (let step = 1; step <= MAX_STEPS; step++) {
    const decision = await llmDecideNextAction(state, tools);
    if (decision.type === 'final') return decision.answer;
    const observation = await tools[decision.tool].execute(decision.args);
    state.observations.push({ action: decision, observation });
  }
  throw new Error('MAX_STEPS exceeded');
}
```

优点：灵活、轨迹可解释。风险：重复搜索、反复修错、成本不可预测。必须配 `MAX_STEPS`、重复动作检测、预算、trace。

### 5.3 Plan-and-Execute

适用：目标可先拆解，执行步骤相对明确；例如"调研 3 个方案并生成表格"。

```ts
let plan = await plannerModel(`Create a JSON plan with 3-6 steps for: ${goal}`);
for (const item of plan.steps) {
  const result = await executorAgent.run(item.task);
  if (!result.ok && item.can_replan) {
    plan = await plannerModel(`Revise plan after failure: ${JSON.stringify(result.error)}`);
  }
}
```

Planner 用强模型，Executor 可用便宜模型或确定性代码。Plan 输出结构化 JSON：`[{id, task, tool_hint, success_criteria}]`，每步有完成标准，否则执行器会"看起来完成"。

### 5.4 Reflection / 自我批评

适用：有可验证反馈的任务，如写代码、生成 SQL、数据清洗、答题。不要对不可验证任务无限反思。

```ts
for (let attempt = 1; attempt <= 3; attempt++) {
  const output = await generator(goal, previousFeedback);
  const test = await runChecks(output);
  if (test.ok) return output;
  previousFeedback = await critic({ goal, output, errors: test.errors });
}
throw new Error('Failed after 3 reflection attempts');
```

反思必须基于外部反馈（测试失败、lint 错误、评估器打分），而不是让模型空想"哪里可以更好"。

### 5.5 Router

适用：用户请求类型差异大，需要分派到不同 workflow 或工具集合。

```ts
const routeSchema = {
  type: 'object',
  required: ['route', 'confidence'],
  properties: {
    route: { enum: ['rag_qa', 'data_analysis', 'code_task', 'human_support'] },
    confidence: { type: 'number', minimum: 0, maximum: 1 },
    reason: { type: 'string', maxLength: 200 }
  }
};

async function route(input) {
  const decision = await classifyWithSchema(input, routeSchema);
  if (decision.confidence < 0.7) return humanSupport(input);
  return handlers[decision.route](input);
}
```

Router 的好处是把复杂系统切小：每条路径可以有独立 prompt、工具、评估集和预算。

### 5.6 Multi-agent：orchestrator-workers

适用：任务确实可拆给不同角色，且每个角色有独立上下文与产物。例如研究员搜集资料、工程师实现、评审员找 bug。

```ts
async function orchestrate(goal) {
  const tasks = await planner(goal); // [{role:'researcher', task:'...'}, ...]
  const results = [];
  for (const task of tasks) {
    const worker = workers[task.role];
    results.push(await worker.run(task.task));
  }
  return await synthesizer({ goal, results });
}
```

不要一开始就多智能体。它会放大延迟、成本、故障面和调试难度。先用单 agent + 好工具打透，确有必要再拆。

## 6. 记忆：短期上下文管理 + 长期向量记忆

Agent 记忆不是"把所有历史塞进 prompt"。记忆要服务于决策。

### 6.1 短期记忆：滑窗 + 摘要 + scratchpad

```ts
function compactContext(messages, observations) {
  const recent = messages.slice(-8);
  const importantObservations = observations
    .filter(o => o.keep)
    .slice(-10)
    .map(o => ({ tool: o.tool, summary: o.summary }));
  return { recent, importantObservations };
}
```

压缩策略：保留最近 N 条消息和最近 K 个工具观察；每 5-10 步把 trace 压缩成 `{facts, decisions, open_questions}`；HTTP HTML、长日志只保留摘要和可追溯 URI；摘要字段区分 `facts[]` 和 `assumptions[]`。

### 6.2 长期记忆：向量检索 + 结构化过滤

长期记忆适合保存跨会话知识：用户偏好、项目约定、历史解决方案、文档片段。RAG 的检索、切分、embedding、rerank 设计见 [RAG 架构](03-rag-architecture.md)。

```text
事件/文档 → 清洗 → 切分 chunk → embedding → vector store
运行时：goal → embedding → top_k 检索 → metadata filter → rerank → 注入 prompt
```

过滤条件包括 `tenant_id/user_id`、`source_type`、`freshness`、`confidence`。长期记忆写入要谨慎：保存来源引用、过期时间、用户删除、管理员审计，避免错误记忆反复污染未来任务。

## 7. 停止条件、循环检测、预算护栏、失败恢复

| 停止条件 | 触发例子 | 返回给用户/上游 |
| --- | --- | --- |
| `done` | 模型输出 final 且满足完成标准 | 最终答案 + 关键证据 |
| `max_steps` | 步数达到 `MAX_STEPS` | 已完成步骤、卡住位置、建议人工 |
| `budget` | 估算花费超过 `maxUsd` | 花费摘要、降级建议 |
| `loop_detected` | 同一工具同参重复 2-3 次 | 重复动作、最后观察 |
| `needs_human` | 高风险工具需要确认 | 待确认 draft 与风险说明 |
| `timeout` | 单步或全局超时 | 超时工具、可重试状态 |

### 7.1 循环检测

```ts
function actionKey(action) {
  return `${action.tool}:${JSON.stringify(action.args, Object.keys(action.args).sort())}`;
}

function isRepeated(action, seen) {
  const key = actionKey(action);
  const count = (seen.get(key) ?? 0) + 1;
  seen.set(key, count);
  return count >= 3;
}
```

生产里还要检测语义重复：连续搜索同义 query、连续修改同一文件但测试不变、连续调用失败 API。

### 7.2 预算护栏

```ts
const PRICE_PER_1K = { input: 0.005, output: 0.015 };
function estimateCost(inputTokens, outputTokens) {
  return inputTokens / 1000 * PRICE_PER_1K.input + outputTokens / 1000 * PRICE_PER_1K.output;
}
```

建议设置单任务 `maxUsd`、单用户 quota、工具级 rate limit、模型降级策略。预算剩余低于 30% 时禁用 Reflection 或切换便宜模型。

### 7.3 失败恢复：把 error feed back 给模型

```json
{
  "tool": "http_get_json",
  "ok": false,
  "error": {
    "code": "TIMEOUT",
    "retryable": true,
    "message": "GET https://example.com timed out after 5000ms",
    "suggested_fix": "Retry once with a narrower URL, or use search_hacker_news for metadata."
  }
}
```

错误不要直接抛到最外层。把结构化 error 当 observation 回灌，然后让模型选择重试、换工具、缩小参数、停止并说明限制。代码层仍限制最大重试次数。

## 8. 可观测性：记录 thought/action/observation、trace、replay

```ts
type TraceStep = {
  run_id: string;
  step: number;
  timestamp: string;
  model: string;
  thought_summary?: string;
  action?: { tool: string; args: unknown };
  observation?: ToolResult;
  latency_ms: number;
  estimated_cost_usd: number;
  stop_reason?: string;
};
```

每一步至少记录：输入目标和压缩上下文版本号、模型名、温度、prompt 版本、工具名、参数、返回摘要、错误码、latency、token、cost、重试次数、stop reason。

### 8.1 Trace 示例

```json
{
  "run_id": "run_20260707_001",
  "step": 2,
  "model": "demo-planner",
  "thought_summary": "Need current discussion references before calculating average points.",
  "action": { "tool": "search_hacker_news", "args": { "query": "LLM agent tracing", "limit": 3 } },
  "observation": { "ok": true, "data": [{ "title": "...", "points": 128 }] },
  "latency_ms": 421,
  "estimated_cost_usd": 0.0002
}
```

### 8.2 Replay harness

```ts
async function replay(trace, tools) {
  for (const step of trace) {
    if (!step.action) continue;
    const actual = await tools[step.action.tool].execute(step.action.args, { replay: true });
    console.log(step.step, { expected: step.observation, actual });
  }
}
```

Replay 用于验证工具返回兼容结构、比较新 prompt 是否减少步数、对黄金任务集做回归测试，连接 [LLM 应用评估](05-evals-for-llm-apps.md)。

### 8.3 工具选择

- **LangSmith**：适合 LangChain/LangGraph 生态，能看 run tree、prompt、tool call、dataset eval。
- **OpenTelemetry**：适合已有分布式追踪体系，把 agent step 当 span，接入 Jaeger、Grafana Tempo、Honeycomb。
- **自建 JSONL trace**：早期足够用。每行一个 step，后续再导入分析系统。

OpenTelemetry span 命名建议：`agent.run`、`agent.plan`、`agent.tool.search_hacker_news`、`agent.finalize`。

## 9. 完整最小可运行 Agent：计算器 + 搜索/HTTP 工具

下面是一个单文件 Node.js Agent，具备 `MAX_STEPS`、预算护栏、ReAct 风格循环、工具 schema、运行时校验、超时、结构化错误、calculator、Hacker News search、HTTP JSON GET、trace 输出。

### 9.1 创建 `agent.mjs`

```js
// agent.mjs - Node.js 20+
const MAX_STEPS = Number(process.env.MAX_STEPS ?? 6);
const MAX_USD = Number(process.env.MAX_USD ?? 0.05);
const MODEL = process.env.OPENAI_MODEL ?? 'gpt-4o-mini';
const API_URL = process.env.OPENAI_BASE_URL ?? 'https://api.openai.com/v1/chat/completions';

const goal = process.argv.slice(2).join(' ') ||
  'Search Hacker News for LLM agent tracing, take the top 3 story points, and calculate their average.';

function ok(data) { return { ok: true, data }; }
function err(code, message, retryable = false, details = undefined, suggested_fix = undefined) {
  return { ok: false, error: { code, retryable, message, details, suggested_fix } };
}

function stableJson(value) {
  if (Array.isArray(value)) return `[${value.map(stableJson).join(',')}]`;
  if (value && typeof value === 'object') {
    return `{${Object.keys(value).sort().map(k => `${JSON.stringify(k)}:${stableJson(value[k])}`).join(',')}}`;
  }
  return JSON.stringify(value);
}

function validate(schema, args) {
  if (!args || typeof args !== 'object' || Array.isArray(args)) return err('VALIDATION_ERROR', 'Arguments must be an object.', true);
  for (const key of schema.required ?? []) {
    if (!(key in args)) return err('VALIDATION_ERROR', `Missing required parameter '${key}'.`, true, { field: key });
  }
  if (schema.additionalProperties === false) {
    for (const key of Object.keys(args)) {
      if (!schema.properties?.[key]) return err('VALIDATION_ERROR', `Unknown parameter '${key}'.`, true, { field: key });
    }
  }
  for (const [key, rule] of Object.entries(schema.properties ?? {})) {
    if (!(key in args)) continue;
    const value = args[key];
    if (rule.type === 'string' && typeof value !== 'string') return err('VALIDATION_ERROR', `'${key}' must be string.`, true);
    if (rule.type === 'integer' && !Number.isInteger(value)) return err('VALIDATION_ERROR', `'${key}' must be integer.`, true);
    if (rule.minLength && value.length < rule.minLength) return err('VALIDATION_ERROR', `'${key}' is too short.`, true);
    if (rule.maxLength && value.length > rule.maxLength) return err('VALIDATION_ERROR', `'${key}' is too long.`, true);
    if (rule.minimum !== undefined && value < rule.minimum) return err('VALIDATION_ERROR', `'${key}' must be >= ${rule.minimum}.`, true);
    if (rule.maximum !== undefined && value > rule.maximum) return err('VALIDATION_ERROR', `'${key}' must be <= ${rule.maximum}.`, true);
  }
  return ok(args);
}

async function withTimeout(promiseFactory, ms) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);
  try { return await promiseFactory(controller.signal); }
  finally { clearTimeout(timer); }
}

const tools = {
  calculator: {
    name: 'calculator',
    description: 'Evaluate basic arithmetic with numbers, +, -, *, /, parentheses, decimals. No variables or functions.',
    idempotent: true,
    sideEffect: 'none',
    requiresConfirmation: false,
    schema: { type: 'object', additionalProperties: false, required: ['expression'], properties: { expression: { type: 'string', minLength: 1, maxLength: 120 } } },
    async execute(args) {
      if (!/^[0-9+\-*/().\s]+$/.test(args.expression)) {
        return err('VALIDATION_ERROR', 'Expression contains unsupported characters.', true, { expression: args.expression }, 'Use only numbers and + - * / parentheses.');
      }
      try {
        const value = Function(`"use strict"; return (${args.expression});`)();
        if (!Number.isFinite(value)) return err('VALIDATION_ERROR', 'Expression did not produce a finite number.', true);
        return ok({ expression: args.expression, value });
      } catch (e) {
        return err('VALIDATION_ERROR', `Invalid arithmetic expression: ${e.message}`, true);
      }
    }
  },

  search_hacker_news: {
    name: 'search_hacker_news',
    description: 'Search public Hacker News stories through the Algolia API. Returns title, url, author, points.',
    idempotent: true,
    sideEffect: 'read',
    requiresConfirmation: false,
    schema: {
      type: 'object',
      additionalProperties: false,
      required: ['query'],
      properties: { query: { type: 'string', minLength: 2, maxLength: 120 }, limit: { type: 'integer', minimum: 1, maximum: 5 } }
    },
    async execute(args) {
      const limit = args.limit ?? 3;
      const url = `https://hn.algolia.com/api/v1/search?tags=story&query=${encodeURIComponent(args.query)}`;
      try {
        const data = await withTimeout(async signal => {
          const res = await fetch(url, { signal, headers: { 'user-agent': 'minimal-agent-demo' } });
          if (!res.ok) throw new Error(`HTTP ${res.status}`);
          return res.json();
        }, 8000);
        const hits = (data.hits ?? []).slice(0, limit).map(h => ({ title: h.title, url: h.url, author: h.author, points: h.points ?? 0 }));
        return ok({ query: args.query, results: hits });
      } catch (e) {
        return err(e.name === 'AbortError' ? 'TIMEOUT' : 'UPSTREAM_ERROR', `HN search failed: ${e.message}`, true, { url }, 'Retry with a shorter query or lower limit.');
      }
    }
  },

  http_get_json: {
    name: 'http_get_json',
    description: 'Perform HTTPS GET for a JSON URL. It never submits forms or sends credentials.',
    idempotent: true,
    sideEffect: 'read',
    requiresConfirmation: false,
    schema: { type: 'object', additionalProperties: false, required: ['url'], properties: { url: { type: 'string', minLength: 8, maxLength: 300 } } },
    async execute(args) {
      if (!/^https:\/\//.test(args.url)) return err('VALIDATION_ERROR', 'Only https:// URLs are allowed.', true);
      try {
        const json = await withTimeout(async signal => {
          const res = await fetch(args.url, { signal, headers: { accept: 'application/json' } });
          if (!res.ok) throw new Error(`HTTP ${res.status}`);
          return res.json();
        }, 8000);
        return ok({ url: args.url, json });
      } catch (e) {
        return err(e.name === 'AbortError' ? 'TIMEOUT' : 'UPSTREAM_ERROR', `HTTP JSON GET failed: ${e.message}`, true, { url: args.url });
      }
    }
  }
};

function toolSpecs() {
  return Object.values(tools).map(t => ({ name: t.name, description: t.description, schema: t.schema }));
}

function estimateCost(inputTokens, outputTokens) {
  return inputTokens / 1000 * 0.00015 + outputTokens / 1000 * 0.0006;
}

async function callOpenAIPlanner(state) {
  const system = `You are a tool-using agent. Return ONLY compact JSON.
Schema: {"type":"tool","thought_summary":"...","tool":"tool_name","args":{...}} or {"type":"final","thought_summary":"...","answer":"..."}.
Use tools when needed. Stop when the goal is satisfied. Available tools: ${JSON.stringify(toolSpecs())}`;
  const user = JSON.stringify({ goal: state.goal, observations: state.observations.slice(-5) });
  const res = await fetch(API_URL, {
    method: 'POST',
    headers: { 'content-type': 'application/json', authorization: `Bearer ${process.env.OPENAI_API_KEY}` },
    body: JSON.stringify({ model: MODEL, temperature: 0, messages: [{ role: 'system', content: system }, { role: 'user', content: user }] })
  });
  if (!res.ok) throw new Error(`Planner API failed: HTTP ${res.status} ${await res.text()}`);
  const data = await res.json();
  return { decision: JSON.parse(data.choices?.[0]?.message?.content ?? '{}'), cost: estimateCost(data.usage?.prompt_tokens ?? 1000, data.usage?.completion_tokens ?? 200) };
}

function demoPlanner(state) {
  const obs = state.observations;
  if (obs.length === 0) {
    return { decision: { type: 'tool', thought_summary: 'Need search results before calculating average points.', tool: 'search_hacker_news', args: { query: 'LLM agent tracing', limit: 3 } }, cost: 0.0001 };
  }
  const search = obs.find(o => o.tool === 'search_hacker_news' && o.result.ok);
  if (search && !obs.find(o => o.tool === 'calculator')) {
    const points = search.result.data.results.map(r => r.points ?? 0);
    return { decision: { type: 'tool', thought_summary: 'Calculate average points from top results.', tool: 'calculator', args: { expression: `(${points.join('+')})/${points.length}` } }, cost: 0.0001 };
  }
  const calc = obs.find(o => o.tool === 'calculator' && o.result.ok);
  if (search && calc) {
    const titles = search.result.data.results.map((r, i) => `${i + 1}. ${r.title} (${r.points} points)`).join('\n');
    return { decision: { type: 'final', thought_summary: 'Search and calculation complete.', answer: `Top results:\n${titles}\nAverage points: ${calc.result.data.value}` }, cost: 0.0001 };
  }
  return { decision: { type: 'final', thought_summary: 'Cannot make progress after tool errors.', answer: 'I could not complete the task. See trace for tool errors.' }, cost: 0.0001 };
}

async function plan(state) {
  if (process.env.OPENAI_API_KEY) return callOpenAIPlanner(state);
  return demoPlanner(state);
}

async function runAgent(goal) {
  const state = { goal, observations: [], trace: [], spentUsd: 0, seenActions: new Map(), runId: `run_${Date.now()}` };
  for (let step = 1; step <= MAX_STEPS; step++) {
    if (state.spentUsd >= MAX_USD) return finalize(state, 'budget', `Budget exceeded: $${state.spentUsd.toFixed(4)}`);
    const started = Date.now();
    let decision, cost;
    try {
      ({ decision, cost } = await plan(state));
      state.spentUsd += cost;
    } catch (e) {
      state.trace.push({ step, error: String(e), latency_ms: Date.now() - started });
      return finalize(state, 'error', `Planner failed: ${e.message}`);
    }
    if (decision.type === 'final') {
      state.trace.push({ step, thought_summary: decision.thought_summary, stop_reason: 'done', latency_ms: Date.now() - started, estimated_cost_usd: cost });
      return finalize(state, 'done', decision.answer);
    }
    const tool = tools[decision.tool];
    if (!tool) {
      state.observations.push({ tool: decision.tool, args: decision.args, result: err('VALIDATION_ERROR', `Unknown tool '${decision.tool}'.`, true, { available: Object.keys(tools) }) });
      continue;
    }
    if (tool.requiresConfirmation) return finalize(state, 'needs_human', `Tool ${tool.name} requires confirmation.`);
    const actionKey = `${decision.tool}:${stableJson(decision.args)}`;
    const seen = (state.seenActions.get(actionKey) ?? 0) + 1;
    state.seenActions.set(actionKey, seen);
    if (seen >= 3) return finalize(state, 'loop_detected', `Repeated action: ${actionKey}`);
    const validation = validate(tool.schema, decision.args);
    const result = validation.ok ? await tool.execute(validation.data, { runId: state.runId }) : validation;
    const observation = { tool: tool.name, args: decision.args, result };
    state.observations.push(observation);
    state.trace.push({
      run_id: state.runId,
      step,
      timestamp: new Date().toISOString(),
      model: process.env.OPENAI_API_KEY ? MODEL : 'demo-planner',
      thought_summary: decision.thought_summary,
      action: { tool: tool.name, args: decision.args },
      observation: result,
      latency_ms: Date.now() - started,
      estimated_cost_usd: cost
    });
  }
  return finalize(state, 'max_steps', `Stopped after MAX_STEPS=${MAX_STEPS}.`);
}

function finalize(state, stopReason, answer) {
  return { stopReason, answer, spentUsd: Number(state.spentUsd.toFixed(6)), trace: state.trace };
}

const result = await runAgent(goal);
console.log(JSON.stringify(result, null, 2));
```

### 9.2 运行

无 API key 的 demo 模式：

```bash
node agent.mjs
```

预期输出形态（搜索结果会随时间变化）：

```json
{
  "stopReason": "done",
  "answer": "Top results:\n1. ...\nAverage points: 123.33",
  "spentUsd": 0.0003,
  "trace": [
    { "step": 1, "action": { "tool": "search_hacker_news" } },
    { "step": 2, "action": { "tool": "calculator" } },
    { "step": 3, "stop_reason": "done" }
  ]
}
```

使用真实 LLM：

```bash
export OPENAI_API_KEY="sk-..."
node agent.mjs "Find three recent discussions about OpenTelemetry LLM tracing and calculate the average points."
```

Windows PowerShell：

```powershell
$env:OPENAI_API_KEY = "sk-..."
node .\agent.mjs "Find three recent discussions about OpenTelemetry LLM tracing and calculate the average points."
```

### 9.3 你应该检查什么

- `trace` 中每步都有 action、args、observation、latency。
- `MAX_STEPS=1 node agent.mjs` 会以 `max_steps` 停止。
- `MAX_USD=0 node agent.mjs` 会以 `budget` 停止。
- 把 demo planner 的 `limit` 改成 `20`，应返回结构化 `VALIDATION_ERROR`，而不是崩溃。

## 10. 何时"别用 Agent"：决策指南

| 问题 | 如果答案是"是" | 推荐方案 |
| --- | --- | --- |
| 步骤是否固定、可枚举？ | 是 | 普通 workflow / 状态机 |
| 是否只需一次检索再回答？ | 是 | RAG，见 [RAG 架构](03-rag-architecture.md) |
| 是否对延迟极敏感？ | 是 | 确定性服务 + 缓存，避免自主循环 |
| 是否有高风险副作用？ | 是 | draft/confirm workflow + human-in-the-loop |
| 是否需要运行时动态选择工具和步骤？ | 是 | Agent，但加硬护栏 |

```text
if fixed_steps:
  use_workflow()
elif single_retrieval:
  use_rag()
elif high_risk_side_effect and no_human_confirmation:
  reject_or_draft_only()
elif dynamic_steps and tool_results_change_next_action:
  use_agent_with_guardrails()
else:
  start_with_router_or_single_tool_call()
```

例子："每天 9 点拉报表、生成摘要、发邮件"用 workflow；"根据用户问题从知识库回答"用 RAG；"调研陌生库、查文档、写示例、根据错误修正"可用 Agent；"自动卖出股票"不要让 Agent 直接执行，最多生成建议或订单草稿。

## 11. 常见坑 & 排查表

| 症状 | 常见原因 | 排查方法 | 修复 |
| --- | --- | --- | --- |
| 死循环烧钱 | 没有 `MAX_STEPS`；重复搜索同一 query | 看 trace 中 actionKey 是否重复 | 加步数上限、重复检测、搜索次数上限 |
| 工具选错 | 工具描述重叠或太抽象 | 统计黄金集 tool accuracy | 改名、收窄描述、拆分工具集合 |
| 参数经常错 | schema 太宽、缺少范围 | 查看 `VALIDATION_ERROR` 分布 | `additionalProperties:false`、加 enum/range |
| 幻觉"已完成" | 最终回答未绑定工具结果 | 对比 answer 与 observation | 用模板引用工具返回，禁止模型编造状态 |
| 错误无法自愈 | 工具只返回 `failed` | 看 observation.error 是否有 code | 统一 `{code,retryable,suggested_fix}` |
| 成本失控 | Reflection/多智能体无限重试 | 统计每 run token/step | 预算、重试上限、模型降级 |
| 线上不可复盘 | 没有 trace 或日志缺 args | 随机抽失败 run | JSONL trace + run_id + step span |
| Prompt injection 成功 | 工具把网页内容当指令 | 检查 prompt 是否隔离 untrusted text | 把检索内容标为 data，不让其覆盖 system |
| 副作用重复执行 | 重试时无幂等 key | 查外部 API 重复记录 | idempotency key + draft/confirm |
| 多智能体互相污染 | worker 输出无结构化边界 | 看 orchestrator 输入 | worker 产物 schema 化，失败隔离 |

## 12. 检查清单

- [ ] 我能说明为什么当前需求需要 Agent，而不是 workflow/RAG/单次工具调用。
- [ ] 每个工具都有清晰 name、description、JSON Schema、参数范围和 `additionalProperties:false`。
- [ ] 有副作用工具拆成 draft/confirm，并带 `idempotency_key`。
- [ ] 循环实现了 `MAX_STEPS`、预算、超时、重复动作检测。
- [ ] 工具错误统一为 `{code,retryable,message,details,suggested_fix}`。
- [ ] 每一步记录 trace：model、action、args、observation、latency、cost、stop reason。
- [ ] 短期记忆不会无限增长，有滑窗或摘要。
- [ ] 长期记忆有 tenant/user 过滤、来源引用、删除/过期机制。
- [ ] 有一组黄金任务评估轨迹质量：工具选择、步数、成本、最终答案。
- [ ] 高风险动作有 human-in-the-loop，安全边界在代码层而不是 prompt。

## 13. 动手练习 / 里程碑作业

### 练习 A：跑通最小 Agent

Deliverables：

1. `agent.mjs` 能在无 API key 模式下运行。
2. 输出包含 `stopReason: "done"`、`answer`、至少 2 个 trace step。
3. 截图或保存一次 JSON trace。

验收命令：

```bash
node agent.mjs
MAX_STEPS=1 node agent.mjs
```

### 练习 B：新增一个工具

新增 `read_json_file` 工具：读取本地 JSON 文件并返回指定 key。

要求：

- schema：`{ path: string, key?: string }`。
- 只允许读取当前目录下 `.json` 文件，禁止 `..`。
- 错误返回 `NOT_FOUND`、`VALIDATION_ERROR` 或 `UPSTREAM_ERROR`。
- 加一条 demo planner 分支，让 Agent 先读 JSON，再计算其中数字平均值。

### 练习 C：加 trace replay

Deliverables：

1. 每次运行把 trace 追加到 `agent-trace.jsonl`。
2. 实现 `node agent.mjs --replay agent-trace.jsonl`，重放工具调用。
3. 输出 expected vs actual 的差异。

### 练习 D：做一个小评估集

创建 10 条任务：3 条只需要 calculator；3 条需要 search + calculator；2 条故意让工具参数错误，验证自愈；2 条应该触发 `max_steps` 或 `budget`。统计成功率、平均步数、平均成本、重复动作次数。

## 14. 延伸阅读

- ReAct paper: [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
- Anthropic: [Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- Model Context Protocol: [MCP documentation](https://modelcontextprotocol.io/)
- OpenAI: [Function calling / tools guide](https://platform.openai.com/docs/guides/function-calling)
- LangChain: [LangGraph documentation](https://langchain-ai.github.io/langgraph/)
- LangSmith: [Observability and evaluation](https://docs.smith.langchain.com/)
- OpenTelemetry: [Specification](https://opentelemetry.io/docs/specs/otel/)
- OWASP: [Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

## 小结

Agent 的工程难点不在"让它更聪明"，而在**约束它**：清晰的工具、硬性的停止条件、代码层护栏、可回放的可观测性。先从单次工具调用或普通 workflow 开始；当任务确实需要运行时动态决策，再引入 ReAct、Plan-and-Execute、Reflection、Router 或 Multi-agent。每增加一个自由度，就同步增加一个可验证的护栏和一条评估用例。


`标签` `Agent` `工具调用` `ReAct` `Plan-and-Execute` `Reflection` `Router` `多智能体` `可观测性` `可靠性` `LLM`

---

[← 上一章](05-evals-for-llm-apps.md) · [WP-01 目录](README.md) · [下一章 →](07-finetune-vs-rag-vs-prompt.md)
