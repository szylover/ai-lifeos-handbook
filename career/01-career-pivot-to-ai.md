[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 资深工程师转型 AI：12 个月行动计划

> 核心判断：资深 SWE **不需要**从零变成 ML 研究员。市场最缺的是**能把 AI 可靠地做进产品**的工程师：能识别业务场景、拆系统边界、控制质量/成本/延迟、把不稳定的模型能力变成可上线的用户体验。

本章是一份 hands-on playbook：帮你把「多年系统工程经验」重新定位成「AI 应用交付能力」，用目标岗位 JD 逆向补齐缺口，用 90 天做出一个有真实用户、真实指标、真实复盘的作品集，并把它转化成内部转岗、跳槽或独立产品的机会。

## 读完本章你应该产出的东西

请不要只阅读。读完后你至少要交付 6 个文件/链接：

1. **目标岗位 JD 分析表**：3 个目标岗位，每个拆出 8–12 条能力要求。
2. **能力缺口 worksheet**：每条要求对应证据、缺口、补齐动作、截止日期。
3. **转型路线选择表**：在「内转 / 跳槽 / 独立产品」中选一条主线、一条备线。
4. **作品集项目 brief**：问题、用户、scope、技术栈、指标、用户获取、复盘计划。
5. **90 天周计划**：每周 deliverable 可检查，不写「学习 AI」这种不可验收目标。
6. **改写后的简历 bullet**：至少 8 条，把既有工程经历重写成「能把 AI 做进产品」。

前置能力假设：你已经有 5 年以上软件工程经验，熟悉后端/平台/全栈/数据/DevOps 中至少一个方向；你不需要先掌握深度学习论文推导，但需要能写 API、读日志、做评估、上线服务。

---

## 一句话定位：资深工程能力 × AI 应用交付 = 稀缺性

不要把自己定位成「正在学习 AI 的老工程师」。更有市场价值的定位是：

> 我是能把 AI 能力安全、可观测、可迭代地接入真实产品流程的资深工程师。

公式：

```text
个人稀缺性 = 资深工程能力 × AI 应用交付 × 业务场景理解 × 可证明证据
```

拆开看：

| 维度 | 你已有的资产 | 需要叠加的 AI 能力 | 可证明证据 |
| --- | --- | --- | --- |
| 系统设计 | 服务拆分、缓存、异步任务、SLA、容灾 | LLM gateway、RAG pipeline、agent tool boundary、human-in-the-loop | 架构图、延迟/成本指标、故障复盘 |
| 工程质量 | 测试、CI/CD、监控、回滚、灰度 | evals、prompt regression、模型版本对比、幻觉 case bank | eval report、失败样本集、上线 checklist |
| 数据能力 | ETL、日志、指标、搜索、权限 | embedding、chunking、retrieval quality、feedback loop | 检索命中率、人工评分、A/B 前后对比 |
| 产品判断 | 需求拆解、优先级、用户访谈 | 判断何时用 LLM，何时不用；设计 AI UX fallback | 用户访谈记录、转化率、节省时间 |
| 团队协作 | 跨团队推进、review、事故处理 | 和 PM/ML/安全/法务协作上线 AI 功能 | RFC、PR、上线公告、风险说明 |

### 不要竞争「模型研究员」，要竞争「AI product engineer」

很多资深工程师转型失败，是因为误把目标设成「补完所有 ML 基础」。更现实的岗位画像是：

- **AI Product Engineer**：快速把 LLM/RAG/Agent 做进产品，关注用户价值、质量和迭代速度。
- **LLM Application Engineer**：设计提示词、工具调用、检索、评估、监控和成本控制。
- **AI Platform Engineer**：建设模型网关、评估平台、特征/向量数据管道、权限与审计。
- **Developer Productivity AI Engineer**：把 AI 用在代码审查、测试生成、知识库问答、内部自动化。

如果你原来是后端/平台工程师，优先瞄准 **AI 应用工程**和 **AI 平台工程**，而不是纯算法岗。学习路线可配合 [AI 工程师能力地图](../ai-engineering/01-ai-engineer-roadmap.md)。

---

## 用目标岗位 JD 逆向对齐能力缺口

转型不是「先学一年再说」，而是从目标岗位倒推：岗位要什么证据，你就补什么证据。

### Step 1：收集 3 类 JD

用 LinkedIn、BOSS 直聘、拉勾、公司官网、GitHub Jobs/Wellfound 等渠道，各找 3 个岗位，共 9 个 JD：

1. **理想岗**：你最想去的 AI 产品/平台团队。
2. **可达岗**：你当前背景匹配 60% 以上的岗位。
3. **跳板岗**：标题不一定 AI，但工作内容包含 LLM/RAG/automation。

搜索关键词：

```text
AI Product Engineer
LLM Application Engineer
RAG Engineer
AI Platform Engineer
GenAI Engineer
Developer Productivity AI
Applied AI Engineer
AI Full-stack Engineer
```

筛 JD 时不要只看标题，重点看 responsibilities：

- 是否要上线用户可见功能？
- 是否提到 RAG、agent、evals、prompt engineering、vector database？
- 是否强调 observability、latency、cost、security、privacy？
- 是否接受 software engineering 背景，而不是要求 PhD / publication？

### Step 2：把 JD 拆成能力原子

把每条 JD 复制到表格里，用下面 6 类标注。

| 类别 | 典型关键词 | 你要准备的证据 |
| --- | --- | --- |
| AI 基础 | LLM, prompt, embeddings, RAG, function calling, agent | 小项目、代码、设计文档、评估报告 |
| 工程交付 | production, scalable, reliable, API, backend, CI/CD | 过往系统经历 + AI 项目上线记录 |
| 评估质量 | evals, metrics, hallucination, safety, guardrails | 测试集、评分 rubrics、失败案例修复 |
| 数据检索 | search, vector DB, indexing, ranking, knowledge base | chunking 策略、检索指标、权限设计 |
| 产品业务 | customer, workflow, UX, adoption, impact | 用户访谈、使用数据、前后对比 |
| 协作影响 | cross-functional, mentor, lead, roadmap | RFC、技术方案、带人/推进案例 |

### Step 3：能力缺口 worksheet（可直接复制）

把下面模板复制到 Notion、Google Sheet 或 Markdown。每个目标岗位填一张。

```markdown
# JD Gap Analysis - <公司/岗位/链接>

## 1. 岗位摘要
- 公司/团队：
- 岗位标题：
- JD 链接：
- 我为什么想要这个岗位：
- 该岗位最可能解决的业务问题：

## 2. JD 原子能力拆解
| # | JD 原文要求 | 能力类别 | 重要度(1-5) | 我已有证据 | 缺口 | 补齐动作 | 产出物 | 截止日期 |
|---|---|---|---:|---|---|---|---|---|
| 1 | Build LLM-powered features in production | AI+工程 | 5 | 做过高并发 API、灰度发布 | 缺 LLM 上线案例 | 做一个 RAG 问答并部署 | Demo + README + eval | W4 |
| 2 | Design evaluation metrics for AI outputs | 评估质量 | 5 | 做过单元测试/监控 | 缺 eval set 和评分 | 建 50 条 golden set | eval_report.md | W6 |
| 3 | Work with PM/design to iterate UX | 产品协作 | 4 | 做过需求评审 | 缺 AI UX fallback 案例 | 访谈 5 个用户，记录失败路径 | interview notes | W5 |

## 3. 匹配度评分
- 必备项数量：
- 我已满足：
- 通过 90 天可补齐：
- 需要 6–12 个月补齐：
- 不准备补齐/不是目标：
- 当前匹配度 = (已满足 + 0.5 × 90天可补齐) / 必备项数量 = __%

## 4. 90 天补齐优先级
P0（必须补，否则无法投）：
1.
2.
3.

P1（能显著提高面试转化）：
1.
2.

P2（锦上添花）：
1.
2.

## 5. 面试证据包
- 作品链接：
- 架构图：
- 指标截图：
- 复盘文章：
- 简历 bullet：
- STAR 故事：
```

### Step 4：给每个缺口定义「证据」，不要定义「学习项」

错误写法：

```text
缺口：不了解 RAG
补齐动作：学习 RAG
```

正确写法：

```text
缺口：没有证明我能把私有知识库接入 LLM 并评估答案质量
补齐动作：用 200 篇团队文档做 RAG，设计 chunking、metadata filter、reranker、eval set
产出物：线上 Demo + 50 条 golden Q&A + 检索命中率/答案正确率报告 + 失败 case 复盘
```

判断一条补齐动作是否合格，用 4 个问题：

1. 面试官能打开链接看到吗？
2. 能用数字说明前后变化吗？
3. 能解释 trade-off 吗？
4. 能讲一个失败 case 和修复过程吗？

---

## 三种转型路线：内转、跳槽、独立产品

三条路不是互斥的。建议设计成「主线 + 备线」：例如主线内转，备线跳槽；或主线跳槽，备线独立产品。

### 路线 A：在现公司内转

适合：你所在公司已经有 AI initiative、数据/用户/业务场景可用，且你有内部信誉。

| 维度 | 优势 | 风险 | 降低风险的做法 |
| --- | --- | --- | --- |
| 资源 | 能接触内部业务、数据、用户 | 权限/合规限制 | 先做低风险内部工具，不碰敏感数据 |
| 信任 | 过往绩效可背书 | 被旧角色绑定 | 用小项目证明你能独立负责 AI 交付 |
| 成本 | 不必立刻换工作 | AI 项目可能只是口号 | 设 6 周验证点，没机会就转 B |

#### 内转 6 步

1. **找场景**：列出 10 个团队内重复、高频、文本密集任务。
   - 例：客服工单归类、PR review checklist、内部知识库问答、销售邮件草稿、测试用例生成。
2. **选低风险切口**：避开直接自动决策，先做 copilot，而不是 autopilot。
   - 合格切口：人确认后执行；失败成本低；数据权限清晰；每天有人用。
3. **写一页 RFC**：用业务语言，不用炫技。

```markdown
# RFC: 用 LLM 辅助 <任务>

## 问题
- 当前流程：
- 每周频次：
- 单次耗时：
- 主要痛点：

## 方案
- 输入：
- 输出：
- 人工确认点：
- 不做什么：

## 成功指标
- 时间节省：从 __ 分钟降到 __ 分钟
- 采纳率：目标 __% 用户每周使用
- 质量：人工评分 ≥ __/5
- 成本：每次调用 ≤ $__

## 风险与控制
- 数据权限：
- 幻觉：
- 回滚：
- 日志与审计：

## 2 周 PoC 计划
- Week 1：
- Week 2：
```

4. **找 sponsor**：不是找「对 AI 感兴趣的人」，而是找 KPI 会被这个工具影响的人。
5. **2 周 PoC + 4 周试点**：PoC 只证明可行，试点才证明有用。
6. **申请角色迁移**：拿指标谈，不拿热情谈。

内转话术：

```text
我希望把 30% 时间投入到 <AI 项目>。过去 4 周试点中，8 位同事使用了 126 次，平均每次节省 12 分钟，人工评分 4.2/5。下一步我想把它产品化：接入权限、日志、eval set 和灰度发布。我可以继续负责现有系统的稳定性，同时把这个方向作为下季度目标。
```

### 路线 B：跳槽到 AI 团队

适合：现公司没有真实 AI 机会，或你希望更快进入 AI-native 团队。

| 维度 | 优势 | 风险 | 降低风险的做法 |
| --- | --- | --- | --- |
| 成长速度 | 直接进入 AI 项目密度高的环境 | 竞争者作品更多 | 用作品集+系统经验差异化 |
| 薪酬机会 | AI 团队预算可能更高 | 岗位泡沫/Title 混乱 | 看 JD 和业务，不只看标题 |
| 学习曲线 | 同事可能有 ML/AI 背景 | 面试会问 AI design | 准备 RAG/eval/agent 设计题 |

#### 跳槽 7 步

1. **定义目标岗位池**：20 个岗位，分成 A/B/C 档。
2. **对每个岗位做 JD gap analysis**：不要海投前才发现证据不够。
3. **作品集先行**：至少 1 个主作品 + 2 个小证据。
4. **简历重写**：顶部 1 行定位 + 作品链接 + 4 条 AI 相关 bullet。
5. **准备 AI 系统设计题**：RAG 问答、文档总结、客服 Copilot、代码助手、数据分析 Agent。
6. **定向 networking**：找目标团队工程师，请教具体工程问题，不要上来要内推。
7. **两轮投递节奏**：先用 5 个 B 档岗位校准，再投 A 档。

联系目标团队工程师的话术：

```text
你好，我是做后端/平台的资深工程师，最近在把 RAG 和 evals 做进一个真实用户工具里。看到你们团队在做 <具体产品/方向>，我对「如何在生产环境评估 LLM 输出质量」很感兴趣。想请教一个具体问题：你们更看重 offline eval、用户反馈还是人工抽检？如果方便，我可以发一页我的项目复盘，请你帮我指出最不像 production 的地方。
```

### 路线 C：独立做 AI 产品

适合：你有明确细分痛点、能直接触达用户、可以承担收入不稳定。

| 维度 | 优势 | 风险 | 降低风险的做法 |
| --- | --- | --- | --- |
| 自主性 | 选题和节奏完全可控 | 容易闭门造车 | 先卖/访谈，再写代码 |
| 作品强度 | 真实用户和收入是最强证据 | 获客难 | 从你熟悉的行业/社区切入 |
| 技术自由 | 可快速试错 | 过度工程化 | 2 周内必须让用户用到 |

#### 独立产品 8 步

1. **选一个你有渠道的窄人群**：例如独立开发者、跨境电商运营、留学顾问、律师助理、内部 SRE 团队。
2. **访谈 10 人**：只问现在怎么做、多久做一次、花多少钱、哪里痛。
3. **写 landing page**：一句话价值主张 + 等候名单，不先写完整产品。
4. **做 concierge MVP**：前 5 个用户可以半自动，后台人工补。
5. **收钱或换强承诺**：哪怕 9.9 美元/99 元试用费，也比「我会用」更真实。
6. **只自动化最频繁步骤**：不要一开始做平台。
7. **记录使用指标**：激活、留存、任务完成率、节省时间、退款原因。
8. **把复盘写成作品集**：即使产品失败，复盘也是强证据。

独立产品的阶段门：

| 时间 | 必须证明 | 没证明怎么办 |
| --- | --- | --- |
| 第 7 天 | 10 个目标用户愿意聊 | 换人群或渠道 |
| 第 14 天 | 3 个用户愿意试用 | 缩小问题，不写更多功能 |
| 第 30 天 | 至少 1 个用户愿意付费/强使用 | 重新定位痛点 |
| 第 60 天 | 周留存或重复使用出现 | 才考虑扩功能 |

---

## 作品集策略：一个真实作品 > 十个教程 demo

作品集不是 GitHub 上堆 10 个 `chat-with-pdf`。一个强作品必须有：真实问题、真实用户、真实指标、真实失败、真实迭代。

### 强作品的评分标准

| 维度 | 0 分 | 1 分 | 2 分 | 3 分 |
| --- | --- | --- | --- | --- |
| 用户 | 只有自己用 | 朋友试过 | 5+ 目标用户用过 | 20+ 用户或团队持续使用 |
| 问题 | 教程题 | 模糊痛点 | 高频具体任务 | 和业务指标/收入/效率挂钩 |
| 技术 | 只调 API | 有 RAG/prompt | 有 eval/监控/权限 | 有灰度、回滚、成本优化 |
| 指标 | 无 | 只有截图 | 有使用/质量/成本指标 | 有前后对比和失败分析 |
| 复盘 | README | 简单介绍 | 讲 trade-off | 有 case bank、迭代记录、下一步 |

目标：主作品达到 12 分以上。低于 8 分时，不要急着投递，把项目做深。

### 作品集 README 模板

```markdown
# <项目名>：一句话说明帮谁解决什么问题

## Problem
- 目标用户：
- 原流程：
- 高频程度：
- 痛点证据：访谈/截图/工单/数据

## Solution
- 用户流程：
- AI 负责什么：
- 人负责什么：
- 明确不做什么：

## Architecture
- Frontend：
- Backend：
- Model provider：
- Retrieval / tools：
- Storage：
- Observability：
- Deployment：

## Metrics
| 指标 | Baseline | 当前 | 目标 | 采集方式 |
|---|---:|---:|---:|---|
| 任务耗时 | | | | |
| 采纳率 | | | | |
| 答案正确率 | | | | |
| P95 latency | | | | |
| 每次成本 | | | | |

## Evaluation
- Golden set 数量：
- 评分 rubrics：
- 失败类别：
- 最近一次 eval 结果：

## Iterations
| 日期 | 发现 | 改动 | 指标变化 |
|---|---|---|---|

## Lessons Learned
- 最大 trade-off：
- 最难 debug 的失败：
- 如果重做会改什么：
```

### Portfolio 项目点子 1：团队知识库 RAG Copilot

**适合背景**：后端、平台、DevOps、技术负责人。

- **用户**：新入职工程师、on-call 工程师、客服/售前同事。
- **问题**：内部文档分散，问一个问题要翻 Confluence/Notion/GitHub/Slack。
- **Scope（4 周版）**：
  1. 导入 100–300 篇公开或脱敏文档。
  2. 支持自然语言问答，返回答案 + 引用来源。
  3. 支持用户反馈：有用/无用/错误原因。
  4. 做 50 条 golden Q&A eval set。
- **技术栈**：Next.js 或 Streamlit；FastAPI/Node.js；PostgreSQL + pgvector / Chroma / Qdrant；OpenAI/Anthropic/本地模型任选；LangChain/LlamaIndex 可用但要能解释内部流程。
- **如何拿真实用户**：
  - 内部：找 5 个新同事或 on-call 同事，每人给 3 个真实问题。
  - 外部：用开源项目文档，如 Kubernetes/Next.js/FastAPI 文档，邀请社区用户试用。
- **度量**：
  - Retrieval hit rate@5：golden answer 的来源文档是否出现在 top 5。
  - Answer correctness：人工 1–5 分，≥4 记正确。
  - Citation accuracy：答案引用是否真的支持结论。
  - P95 latency 和平均 token cost。
- **复盘怎么写**：
  - 展示 5 个失败问题：chunk 太大、权限过滤缺失、问题需要最新版本、答案引用不充分、多跳问题失败。
  - 写出每次迭代：metadata filter、chunk overlap、reranker、query rewriting、拒答策略。

### Portfolio 项目点子 2：PR Review / 测试生成 Copilot

**适合背景**：资深后端、Tech Lead、DevEx、测试平台。

- **用户**：工程团队，每天提交 PR 的开发者。
- **问题**：PR review 容易漏边界条件，测试用例不完整，新人不了解团队 checklist。
- **Scope（3–5 周版）**：
  1. GitHub App 或 CLI 读取 diff。
  2. 根据团队规则生成 review checklist。
  3. 对高风险变更提出测试建议，不自动阻塞 merge。
  4. 把误报/有效建议记录下来做 eval。
- **技术栈**：GitHub API、Node.js/TypeScript 或 Python、LLM function calling、SQLite/PostgreSQL、GitHub Actions。
- **如何拿真实用户**：
  - 先在自己的 5 个 PR 上跑。
  - 邀请 3 位同事/开源维护者对建议打标签：useful / noisy / wrong。
- **度量**：
  - Useful suggestion rate = useful / total suggestions。
  - Noise rate = noisy / total suggestions。
  - Review time saved：人工估计或问卷。
  - 漏测发现数：被采纳的测试建议数量。
- **复盘怎么写**：
  - 说明为什么不追求「自动 approve/reject」。
  - 展示 prompt regression：某次 prompt 改动让 noise rate 上升，如何回滚。
  - 写清权限和安全：不把私有代码发到不合规模型；支持脱敏/本地模型选项。

### Portfolio 项目点子 3：客服/销售工单分流与回复草稿

**适合背景**：业务后端、SaaS、CRM、数据工程。

- **用户**：客服、售前、客户成功团队。
- **问题**：重复问题多，分类和初稿耗时，但直接自动回复风险高。
- **Scope（4 周版）**：
  1. 导入历史 FAQ 或公开 issue。
  2. 工单分类：billing / bug / how-to / feature request / urgent。
  3. 生成回复草稿 + 引用知识库来源。
  4. 人工确认后发送，记录采纳/修改。
- **技术栈**：FastAPI、React、PostgreSQL、vector DB、LLM structured output、简单后台管理。
- **如何拿真实用户**：
  - 如果无公司数据，用开源项目 GitHub issues 模拟客服工单。
  - 找项目维护者或客服朋友试用 20 条历史问题。
- **度量**：
  - 分类准确率。
  - 草稿采纳率：无需大改即可使用的比例。
  - 平均处理时间下降。
  - 升级/拒答正确率：高风险问题是否交给人工。
- **复盘怎么写**：
  - 对比「只用 prompt」与「RAG + 分类 + 模板」效果。
  - 展示高风险 guardrail：退款、法律、医疗、安全事故一律不自动回复。

### Portfolio 项目点子 4：个人/团队数据分析 Agent

**适合背景**：数据平台、BI、全栈、内部工具。

- **用户**：PM、运营、团队负责人。
- **问题**：简单数据问题要排队等分析师；但让 LLM 直接跑 SQL 有安全风险。
- **Scope（4–6 周版）**：
  1. 准备一个脱敏 demo 数据库，如电商订单/产品使用事件。
  2. 自然语言转 SQL，但只允许 SELECT，只读账号，限制表和行数。
  3. SQL 执行前展示解释，用户确认后执行。
  4. 生成图表和结论，并标注不确定性。
- **技术栈**：Python/FastAPI、DuckDB/PostgreSQL、SQLGlot、Plotly、LLM tool calling、OpenTelemetry。
- **如何拿真实用户**：
  - 找 3 个 PM/运营朋友提供真实问题，不提供敏感数据。
  - 用公开数据集复现他们的问题结构。
- **度量**：
  - SQL execution success rate。
  - Correct answer rate：人工校验结果是否正确。
  - Unsafe query block rate。
  - Time-to-insight：从问题到图表的时间。
- **复盘怎么写**：
  - 展示 3 类失败：schema 理解错、指标口径错、相关性被说成因果。
  - 说明如何用 semantic layer、metric definitions、query validation 降低风险。

---

## 90 天转型冲刺计划：学习 + 建作品 + 公开输出 + 投递

这 90 天不是「学完 AI」。目标是产出可展示证据：一个主作品、一个 eval report、三篇公开文章、十次真实用户反馈、一次简历改造、至少 20 次定向触达。

学习主线参考 [AI 工程师能力地图](../ai-engineering/01-ai-engineer-roadmap.md)，但每周都要绑定作品产出。

### 总体节奏

| 阶段 | 周数 | 目标 | 核心 deliverable |
| --- | --- | --- | --- |
| 定位与选题 | W1–W2 | 明确目标岗位和项目 | JD gap analysis、项目 brief、用户名单 |
| MVP | W3–W5 | 做出可用版本 | Demo、部署链接、基础日志 |
| 评估与迭代 | W6–W8 | 从 demo 变成作品 | eval set、指标报告、用户反馈 |
| 包装与投递 | W9–W13 | 转化为机会 | 简历、复盘文章、面试故事、投递 pipeline |

### Week-by-week plan

| 周 | 学习重点 | 建作品 | 公开输出 | 求职/转岗动作 | 验收标准 |
| --- | --- | --- | --- | --- | --- |
| W1 | LLM 应用全景、目标岗位类型 | 收集 9 个 JD，填 3 张 gap analysis | 发一条公开学习路线/转型目标 | 列 20 个目标岗位/团队 | 有明确主线和备线 |
| W2 | Prompt、structured output、API 调用 | 选定 1 个主作品，写 README skeleton 和架构草图 | 写《我为什么选择这个 AI 项目》 | 约 5 个潜在用户访谈 | 访谈记录 ≥5 条，scope 被砍到 4 周可做 |
| W3 | RAG/工具调用基础（二选一） | 完成端到端 happy path | 发 2 分钟 demo 视频 v0 | 找 2 位同事/朋友试用 | 用户能独立完成 1 个任务 |
| W4 | 数据、权限、日志 | 部署到可访问环境，记录 usage events | 写技术笔记：架构和 trade-off | 开始联系目标团队工程师 | Demo 链接可打开，日志可查 |
| W5 | AI UX：fallback、拒答、人工确认 | 加入反馈按钮、错误收集、成本统计 | 发一次用户反馈截图/总结 | 更新简历顶部定位草稿 | 收到 ≥10 条真实反馈 |
| W6 | Evals：golden set、rubrics | 建 30–50 条 eval set，跑 baseline | 写《我如何评估这个 AI 功能》 | 找 1 位 AI 工程师 review 项目 | 有 eval_report，能复现结果 |
| W7 | 检索/提示词/工具边界优化 | 根据失败 case 做 2 次迭代 | 发失败案例复盘 | 准备 3 个 STAR 故事 | 指标至少 1 项明显改善 |
| W8 | Observability、成本、延迟 | 加入 P95 latency、token cost、error taxonomy | 写《从 demo 到 production 缺什么》 | 做一次 mock AI system design | 能讲清质量/成本/延迟 trade-off |
| W9 | Agent/RAG/LLM design 面试题 | 冻结主作品 v1，完善 README | 录 5 分钟 walkthrough 视频 | 简历改版完成 | 作品页 10 分钟内可被看懂 |
| W10 | 系统设计复习 | 准备架构图、sequence diagram | 发项目复盘长文 | 先投 5 个 B 档岗位 | 收到市场反馈，不急着投 A 档 |
| W11 | 行为面试 + AI 风险 | 整理失败 case bank | 写《我踩过的 5 个 AI 应用坑》 | 根据反馈调整简历/作品 | 至少 2 次面试/coffee chat |
| W12 | 针对性补缺口 | 做一个小增强或第二小作品 | 开源一个可复用组件/脚本 | 投 10–15 个目标岗位 | pipeline 有 20+ 触达记录 |
| W13 | 复盘与下一轮计划 | 归档指标、路线图 | 写 90 天总结 | 决定继续内转/跳槽/独立 | 有下一阶段 30 天计划 |

### 每周固定时间块

按每周 10 小时设计，工作忙时也能执行：

| 时间块 | 时长 | 内容 | 产出 |
| --- | ---:| --- | --- |
| 深度学习 | 2h | 读官方文档/课程，只学本周要用的 | 10 条笔记 + 1 个代码实验 |
| 项目开发 | 4h | 实现本周核心功能 | PR/commit/deploy |
| 用户反馈 | 1h | 访谈、试用、问卷、看日志 | 反馈表更新 |
| 评估复盘 | 1h | 跑 eval、看失败、定改动 | eval_report 更新 |
| 公开输出 | 1h | 博客、demo 视频、短帖 | 1 个公开链接 |
| 求职动作 | 1h | JD 分析、networking、简历 | pipeline 更新 |

如果只有每周 5 小时，砍学习时间，不砍用户反馈和评估。作品集的可信度来自真实反馈，不来自你看了多少教程。

---

## 个人品牌与公开影响力：节奏比平台重要

目标不是成为 AI 网红，而是让目标团队在 10 分钟内判断：这个人真的做过 AI 应用交付。

### 三层内容矩阵

| 内容类型 | 频率 | 长度 | 例子 | 目的 |
| --- | --- | --- | --- | --- |
| Build log | 每周 1 次 | 300–800 字 | 本周做了什么、失败了什么、下周改什么 | 证明持续推进 |
| 技术深文 | 每月 1 次 | 1500–3000 字 | RAG eval、成本优化、AI UX fallback | 证明深度思考 |
| Demo 视频 | 每 2–3 周 1 次 | 2–5 分钟 | 从用户问题到系统输出的完整流程 | 降低观看成本 |
| 开源贡献 | 每月 1 个 PR/issue | 小而真实 | 修文档、补测试、复现 bug | 证明协作能力 |

### 技术博客选题模板

不要写《RAG 入门》。写你亲手遇到的问题：

```text
标题模板 1：我如何把 <任务> 的 AI 答案正确率从 <A> 提到 <B>
标题模板 2：一个 <用户类型> AI Copilot 的 5 个失败案例
标题模板 3：从 demo 到 production：<项目名> 的成本、延迟和评估
标题模板 4：为什么我在 <场景> 放弃了 Agent，改用 workflow + human review
```

每篇文章结构：

1. 背景：谁的什么问题，原来怎么做。
2. 方案：架构图 + 关键设计选择。
3. 指标：baseline、当前值、采集方式。
4. 失败：至少 3 个具体 bad cases。
5. 迭代：你改了什么，指标怎么变。
6. 结论：什么时候适用，什么时候不适用。

### Demo 视频脚本

```text
0:00–0:20  说明用户和痛点：这个工具帮 <用户> 在 <场景> 节省 <时间/成本>
0:20–1:20  展示真实输入，不用 toy prompt
1:20–2:20  展示系统输出、引用、人工确认/拒答
2:20–3:20  展示后台：日志、eval、成本、失败分类
3:20–4:30  讲一个失败案例和修复
4:30–5:00  总结指标和下一步
```

发布渠道建议：

- 中文：掘金、知乎、微信公众号、即刻、B 站。
- 英文/国际：GitHub README、LinkedIn、Dev.to、Medium、X。
- 面试最重要：GitHub repo + 一页项目主页 + 5 分钟视频。

---

## 简历改造：把系统工程经验重写成「能把 AI 做进产品」

简历不是把技能栏塞满 `LLM, RAG, LangChain, Vector DB`。你要证明三件事：

1. 你能交付 production system。
2. 你理解 AI 功能的不确定性，并能评估/监控/迭代。
3. 你能把 AI 连接到业务指标。

### 顶部定位示例

Before：

```text
Senior Software Engineer with 8 years of backend experience. Familiar with Java, Go, Kubernetes, microservices.
```

After：

```text
Senior Software Engineer focused on production AI applications: built scalable backend/platform systems, now applying LLM/RAG/evals to ship reliable AI features with measurable user impact.
```

中文版本：

```text
资深软件工程师，专注 production AI applications。具备 8 年后端/平台系统交付经验，正在将 LLM/RAG/evals 接入真实产品流程，关注质量、成本、延迟与用户采纳。
```

### STAR + 指标 bullet 公式

```text
[动作动词] + [做了什么系统/功能] + [技术/方法] + [规模/约束] + [结果指标]
```

AI 转型版：

```text
[将 AI 能力接入什么业务流程] + [如何控制质量/风险] + [如何上线/评估] + [业务或工程结果]
```

可用动词：

```text
Designed, shipped, productionized, evaluated, instrumented, reduced, improved, automated, led, migrated, built, scaled, optimized
```

### Before → After 示例

| 原 bullet | 问题 | 改写后 bullet |
| --- | --- | --- |
| 负责订单系统后端开发，支持业务增长。 | 太泛，没有规模和 AI 迁移价值。 | Designed and scaled order-processing APIs handling 8M+ monthly orders with 99.95% availability; experience directly applicable to productionizing AI workflows with strict latency, retry, and observability requirements. |
| 搭建日志监控平台。 | 没有说明对 AI 质量的关联。 | Built centralized observability platform across 40+ services, reducing MTTR by 35%; reused the same tracing/alerting patterns to instrument LLM latency, token cost, error taxonomy, and fallback rates in AI prototypes. |
| 优化数据库查询性能。 | 只是技术点。 | Optimized PostgreSQL query paths and caching strategy, cutting P95 latency from 780ms to 210ms; later applied retrieval profiling to reduce RAG answer latency by 42% while preserving answer quality. |
| 参与需求评审和系统设计。 | 没有领导力。 | Led cross-functional design reviews with PM, design, security, and data teams for customer-facing workflows; translated ambiguous user needs into measurable acceptance criteria and rollout plans. |
| 做过 CI/CD。 | 太普通。 | Built CI/CD pipelines with canary deploy and automated rollback; adapted the pipeline for prompt/eval regression so LLM changes are tested before release. |
| 写了一个 ChatGPT demo。 | demo 感强。 | Shipped a RAG-based knowledge assistant to 12 beta users, built a 50-question golden eval set, improved answer correctness from 62% to 81%, and reduced average cost per query to $0.018. |
| 熟悉微服务。 | 技能堆砌。 | Decomposed monolith into event-driven services processing 20M events/day; relevant to designing AI workflows with async jobs, queue-based retries, idempotent tool calls, and human approval checkpoints. |
| 做过权限系统。 | 没有连接 AI 风险。 | Implemented RBAC and audit logging for internal admin tools; applied the same controls to RAG document access so generated answers only cite sources the user is authorized to view. |

### 简历结构建议

1. **Header**：姓名、城市/远程、邮箱、GitHub、LinkedIn、作品集主页。
2. **Positioning summary**：2–3 行，突出 production AI applications。
3. **Selected AI Projects**：放在工作经历前或后，取决于作品强度。
4. **Experience**：每段经历 3–5 条 bullet，至少 1 条能连接 AI 交付能力。
5. **Skills**：分组写，不堆名词。

技能栏示例：

```text
AI Applications: LLM APIs, RAG, embeddings, structured output, function calling, evals, prompt regression, AI observability
Backend/Platform: Go, Python, TypeScript, PostgreSQL, Redis, Kafka, Kubernetes, CI/CD, OpenTelemetry
Product Delivery: system design, RFC, user interviews, metrics, rollout, incident review, cross-functional leadership
```

### 面试 STAR 故事模板

```markdown
# STAR Story: <故事名>

Situation：
- 背景是什么？用户/业务/系统处在什么状态？

Task：
- 你负责什么？成功标准是什么？约束是什么？

Action：
- 你做了哪 3–5 个关键动作？
- 技术 trade-off 是什么？
- 如何处理质量、成本、延迟、权限、失败？

Result：
- 数字结果：
- 用户反馈：
- 你学到了什么：
- 如果重做会改什么：

AI relevance：
- 这个故事如何证明你能做 AI 应用交付？
```

---

## 常见坑 & 排查表

| 坑 | 症状 | 根因 | 排查问题 | 修复动作 |
| --- | --- | --- | --- | --- |
| 只学不做 | 收藏很多课，3 个月没有可打开的 Demo | 把学习当成安全区 | 本周有没有用户能打开的链接？ | 每周强制发布一个小版本；学习只服务当前功能 |
| 追热点不落地 | 一会儿 Agent，一会儿 fine-tuning，一会儿多模态 | 没有目标岗位和用户问题 | 这个技术解决哪个用户任务？不用它会怎样？ | 回到 JD gap 和用户访谈，砍掉无关技术 |
| 简历堆术语 | 技能栏很多 AI buzzwords，经历没有指标 | 想用关键词替代证据 | 每个关键词是否有项目/指标支撑？ | 删除无证据术语，改成项目 bullet |
| demo 没用户 | GitHub repo 很多 star 之外没人用 | 没有分发和访谈 | 谁在过去 7 天使用过？ | 先找 5 个真实用户，不加功能 |
| 没有 eval | 只凭感觉说效果好 | 不知道 AI 质量怎么量化 | 是否有 golden set？是否能复现实验？ | 建 30–50 条测试集，定义 rubrics |
| 忽略成本/延迟 | Demo 可用，上线不可用 | 没有 production 思维 | P95 latency 和每次成本是多少？ | 加缓存、流式输出、模型分级、token budget |
| 自动化过度 | 让 AI 直接执行高风险操作 | 为了炫技牺牲安全 | 失败会造成什么损失？ | 改成 copilot + human approval + audit log |
| 作品太大 | 6 周还在搭平台 | 企图一次做完整产品 | 用户最小闭环是什么？ | 砍到 1 个用户、1 个任务、1 个指标 |
| 不会讲故事 | 项目做了但面试讲不清 | 没有复盘材料 | 能否 3 分钟讲清问题→方案→指标→失败？ | 写 README、录视频、准备 STAR |

---

## 检查清单：做到这些再说「我准备好了」

定位与目标：

- [ ] 我能用一句话说清自己的 AI 转型定位。
- [ ] 我分析过至少 9 个 JD，而不是凭感觉学习。
- [ ] 我知道目标岗位的 P0 缺口，并为每个缺口定义了证据。
- [ ] 我选择了一条主路线和一条备线。

作品集：

- [ ] 我有一个可访问的主作品链接。
- [ ] 主作品有真实用户或目标用户试用记录。
- [ ] README 写清 Problem、Solution、Architecture、Metrics、Evaluation、Lessons Learned。
- [ ] 我有 30–50 条 eval set 或等价评估样本。
- [ ] 我记录了至少 5 个失败案例和修复动作。
- [ ] 我能说清 P95 latency、平均成本、质量指标和用户采纳情况。

公开影响力：

- [ ] 我发布过至少 3 篇 build log 或技术复盘。
- [ ] 我录过一个 2–5 分钟 demo 视频。
- [ ] 我至少参与过 1 个开源 AI 项目的 issue/PR，或公开复现过一个 bug。

简历与面试：

- [ ] 简历顶部有 AI application positioning 和作品集链接。
- [ ] 至少 8 条 bullet 使用 STAR + 指标重写。
- [ ] 我准备了 3 个 AI 相关 STAR 故事。
- [ ] 我能白板设计一个 RAG/Agent/AI Copilot 系统，并讨论 eval、权限、成本、延迟。
- [ ] 我用 5 个 B 档岗位测试过简历和作品集反馈。

---

## 动手练习：48 小时启动转型

### 练习 1：完成 JD gap analysis（2–3 小时）

Deliverables：

- 3 个目标岗位链接。
- 每个岗位拆出至少 8 条能力原子。
- 一张合并后的 P0/P1/P2 缺口表。

操作步骤：

1. 搜索 9 个 JD，保留链接和截图。
2. 对每条 requirement 标注类别：AI 基础、工程交付、评估质量、数据检索、产品业务、协作影响。
3. 删除你不打算竞争的纯研究要求，如「发表顶会论文」。
4. 选出出现频次最高的 10 个能力。
5. 为每个能力写一个可展示证据。

完成标准：你能回答「为什么接下来 90 天要做这个作品，而不是学另一个课程」。

### 练习 2：启动一个作品（4–6 小时）

Deliverables：

- GitHub repo 或私有项目目录。
- README skeleton。
- 5 个目标用户问题。
- 第一个端到端 happy path。

操作步骤：

1. 从 4 个 portfolio 点子中选一个，或用自己的公司场景替换。
2. 写一句话问题定义：

```text
我帮 <目标用户> 在 <具体场景> 把 <任务> 从 <当前耗时/成本/痛点> 改善到 <目标指标>。
```

3. 写不做清单：

```text
第一版不做：
- 不做多租户；
- 不做自动执行高风险操作；
- 不支持所有文件格式；
- 不追求 100% 准确，只追求可评估和可迭代。
```

4. 找 5 个真实问题，不要自己编 toy prompt。
5. 做最小闭环：输入 → AI 处理 → 输出 → 用户反馈记录。

完成标准：另一个人不看你解释，也能根据 README 跑起来或打开 Demo。

### 练习 3：写第一条简历 bullet（30 分钟）

选择一段你过去的系统工程经历，按下面模板改写：

```text
Before：

After：
- Designed/led/built <系统/功能> for <用户/业务>, handling <规模>, improving <指标>; this experience maps to production AI delivery through <质量/成本/延迟/权限/可观测性的连接点>.

证据链接/截图：
面试可讲的失败案例：
```

完成标准：这条 bullet 不出现空泛形容词，至少有 1 个数字和 1 个 AI 交付连接点。

---

## 延伸阅读

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)：官方 prompt engineering 基础与示例。
- [OpenAI Evals](https://github.com/openai/evals)：了解如何构建 LLM evals 和测试集。
- [Anthropic: Building effective agents](https://www.anthropic.com/research/building-effective-agents)：区分 workflow 与 agent，避免为了 agent 而 agent。
- [Google: People + AI Guidebook](https://pair.withgoogle.com/guidebook/)：AI UX、用户信任、反馈与失败处理。
- [LangChain RAG documentation](https://python.langchain.com/docs/tutorials/rag/)：RAG 基础实现参考；学习后要能解释抽象层背后的流程。
- [LlamaIndex Documentation](https://docs.llamaindex.ai/)：数据连接、索引、检索和评估工具参考。
- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)：把 agent 当作可维护软件系统设计的实践清单。
- [GitHub Docs: Building GitHub Apps](https://docs.github.com/en/apps/creating-github-apps)：做 PR Review Copilot 时可参考的官方文档。

> 配套章节：[AI 工程师能力地图](../ai-engineering/01-ai-engineer-roadmap.md)、[海外与 Remote 求职策略](02-remote-overseas-job-strategy.md)、[系统设计面试](03-system-design-interview.md)、[STAR 行为面试](04-behavioral-interview-star.md)。


`标签` `职业转型` `AI` `资深工程师` `规划` `作品集` `简历` `个人品牌`

---

[WP-02 目录](README.md) · [下一章 →](02-remote-overseas-job-strategy.md)
