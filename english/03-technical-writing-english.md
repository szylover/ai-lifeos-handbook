[← 手册首页](../README.md) · [WP-03 英语能力](README.md)

---

# 工程师的英文技术写作与异步沟通

> 结论先行：在 Remote/海外团队，**写作是你最高杠杆的英语技能**。PR、RFC、incident update、Slack thread、邮件会被搜索、转发、引用。写得清楚，你的判断力和 ownership 才会被看见。

相关阅读：[Remote / 海外求职策略](../career/02-remote-overseas-job-strategy.md)、[英语口语流利度](02-english-speaking-fluency.md)、[系统设计面试框架](../career/03-system-design-interview.md)。

## 1. 本章定位

本章把英文技术写作当作工程交付能力训练。

你会产出 6 类可复制文本：

1. PR description。
2. Commit message。
3. Design doc / RFC。
4. Incident postmortem。
5. Slack / async update。
6. Professional email。

学完后你能：

- [ ] 用 BLUF 写第一段。
- [ ] 把流水账改成金字塔结构。
- [ ] 给 PR 写 review guide、testing、risk、rollback。
- [ ] 写可搜索的 commit message。
- [ ] 写含 goals、non-goals、alternatives 的 RFC。
- [ ] 写 blameless postmortem。
- [ ] 写清楚 async ask、owner、deadline。

准备 3 个素材：

- 最近一个 PR。
- 最近一次方案讨论。
- 最近一次线上问题或协作卡点。

## 2. 核心原则

技术英文不是文学。

目标是：让读者更快做正确动作。

### 2.1 Clarity：具体到对象和动作

差：

```text
We need to improve the API because it has some problems.
```

好：

```text
We need to add request-level timeouts to the Orders API because 3% of checkout requests exceed 10 seconds during peak traffic.
```

检查 5 件事：

1. 主语是谁？
2. 动作是什么？
3. 对象是什么？
4. 证据是什么？
5. 读者下一步是什么？

### 2.2 Concision：删掉不改变含义的词

| 冗余 | 更好 | 原因 |
|---|---|---|
| basically | 删除 | 无信息 |
| actually | 删除 | 显得防御 |
| very important | critical + 证据 | 强调不如证据 |
| in order to | to | 更短 |
| due to the fact that | because | 更短 |
| at this point in time | now | 更短 |

Before：

```text
I just wanted to check if maybe you could take a look at this PR when you have time.
```

After：

```text
Could you review this PR by Wednesday EOD? It blocks the billing rollout.
```

### 2.3 Structure：默认读者在扫读

- 标题写状态或请求。
- 第一段写结论。
- 小标题回答读者问题。
- 列表写并列事实。
- 表格写取舍。
- 代码块放可复制模板。
- 数字替代形容词。

### 2.4 Specificity：判断后面接证据

弱：

```text
The new approach is better and more scalable.
```

强：

```text
The queue-based approach removes synchronous retries from checkout and reduces p95 latency from 1.8s to 1.1s in the load test.
```

可用证据：p50 / p95、error rate、affected users、regions、reproduction steps、dashboard、issue、runbook、trade-off。

### 2.5 Reader-first：按读者任务排序

| 读者 | 先看 | 你先写 |
|---|---|---|
| Reviewer | 范围、风险、测试 | Summary + How to review |
| Manager | 状态、影响、阻塞 | Status + Ask |
| On-call | 影响、缓解、ETA | Impact + Mitigation |
| Customer | 结果、补救 | What happened + Next |
| Maintainer | 原因、约束 | Why + Trade-off |

## 3. BLUF 与金字塔

BLUF = Bottom Line Up Front。

金字塔 = answer → reasons → evidence → details。

通用结构：

```text
Bottom line: <结论 / 请求 / 状态>
Why:
- <理由 1>
- <理由 2>
Evidence:
- <指标 / 日志 / 链接>
Next step:
- <owner + action + deadline>
```

可直接套用的第一句：

```text
Recommendation: ship the queue-based retry path behind a feature flag.
Decision needed: choose Postgres advisory locks or Redis locks for job deduplication.
Status: the migration is blocked by missing backfill metrics.
Risk: the current implementation can double-charge users if the retry job runs twice.
Ask: please review the data model by Friday 12:00 UTC.
Update: p95 latency is back to normal after rolling back v2.3.
```

Before：

```text
I looked into the latency issue and checked logs. There are many slow database queries. Alice thinks the new index is not used. We can roll back or add another index.
```

After：

```text
Recommendation: roll back the search ranking change in v2.3 today.
Why:
- p95 latency increased from 420ms to 1.9s after the deploy.
- The new query does not use `idx_items_created_at` in production.
- Adding a new index takes ~45 minutes and does not fix the current incident.
Next step: I can start the rollback unless there are objections by 14:30 UTC.
```

改进：第一行给行动；指标替代 many；默认推进时间减少等待。

## 4. 写作工作流

把写作当作代码重构。

### 4.1 Draft：先写事实

```text
What changed?
Why did it change?
Who is affected?
When did it happen?
Where is the evidence?
How was it tested?
What action is needed?
```

### 4.2 Diagnose：诊断读者

- 读者是谁？
- 读者要 approve、decide、debug、schedule 还是 reply？
- 读者有多少上下文？
- 读者最担心什么？

### 4.3 Rewrite：重排

1. Bottom line。
2. Context。
3. Evidence。
4. Options / trade-offs。
5. Ask / next step。

### 4.4 Verify：7 项检查

- [ ] 第一段是否给结论或请求？
- [ ] 每段是否只有一个观点？
- [ ] 是否有数字、链接或复现步骤？
- [ ] 是否说明测试和风险？
- [ ] 是否删除 filler words？
- [ ] 是否有 owner、deadline、timezone？
- [ ] 只看 bullet 是否懂 80%？

## 5. PR Description

PR 描述服务 reviewer；目标是降低 review 成本。

### 模板

```markdown
## Summary
- <One-sentence bottom line.>

## Why
- <Problem / issue / incident / motivation.>

## What changed
- <Change 1>
- <Change 2>

## How to review
- <Start with file/module X.>
- <Skip generated files Z.>

## Testing
- [ ] <command or manual test>: <result>

## Risks and rollback
- Risk: <what could break>
- Mitigation: <flag / monitor / fallback>
- Rollback: <how to disable or revert>
```

### Before → After

Before：

```markdown
## Description
This PR changes the payment code and fixes some bugs. I refactored retry logic and updated tests. Please review.
```

After：

```markdown
## Summary
Fixes duplicate payment retries by moving retry deduplication from the API handler to the `payment_jobs` table.

## Why
- Users can be charged twice if the API times out after Stripe accepts the payment.
- The current in-memory dedupe key is lost when the pod restarts.
- Related issue: #1842.

## What changed
- Added a unique `(payment_id, retry_type)` constraint to `payment_jobs`.
- Changed `POST /payments/:id/retry` to enqueue a job instead of retrying synchronously.
- Added an idempotency check before calling Stripe.

## How to review
- Start with `payments/retry_service.ts`.
- Then review the migration and worker.
- Ignore generated schema snapshots.

## Testing
- `npm test -- payments/retry_service.test.ts`: passed.
- Manual: retried the same payment 5 times; one Stripe request was sent.

## Risks and rollback
- Risk: delayed retries if the worker queue is unhealthy.
- Mitigation: queue depth alert at 500 jobs.
- Rollback: disable `payment_retry_queue_enabled`.
```

完成标准：

- [ ] Summary 能独立存在。
- [ ] Why 写“不改会怎样”。
- [ ] How to review 指路。
- [ ] Testing 可复现。
- [ ] Rollback 可执行。

## 6. Commit Message

Commit message 服务未来维护者、release owner 和 `git bisect`。

### 模板

```text
<type>(<scope>): <imperative summary>

<Why this change is needed.>
<How this change solves it.>
<Important trade-off or migration note.>

Validation: <command or check>
Refs: #<issue>
```

常用 type：

| Type | 用途 |
|---|---|
| feat | 新能力 |
| fix | 修 bug |
| refactor | 不改行为 |
| test | 测试 |
| docs | 文档 |
| perf | 性能 |
| chore | 维护 |

### Before → After

Before：

```text
update stuff
```

After：

```text
fix(payments): dedupe retry jobs by payment id

Retries were deduped in memory, so a pod restart could send the same
payment retry to Stripe twice. This commit adds a database-level unique
constraint on (payment_id, retry_type) and skips enqueueing when a retry
job already exists.

Validation: npm test -- payments/retry_service.test.ts
Refs: #1842
```

完成标准：

- [ ] summary 用祈使句。
- [ ] scope 可搜索。
- [ ] body 解释 why。
- [ ] validation 写明范围。
- [ ] 一个 commit 只表达一个完整意图。

## 7. Design Doc / RFC

RFC 服务决策：基于共同事实选择方案。

### 模板

```markdown
# RFC: <decision title>

## Summary
<Recommendation + reason + trade-off.>

## Problem
<Current pain, impact, metrics, constraints.>

## Goals
- <goal 1>
- <goal 2>

## Non-goals
- <out of scope 1>
- <out of scope 2>

## Proposal
<architecture, API, data model, rollout.>

## Alternatives considered
| Option | Pros | Cons | Decision |
|---|---|---|---|
| A | | | |
| B | | | |

## Risks and mitigations
| Risk | Impact | Mitigation | Owner |
|---|---|---|---|

## Rollout and metrics
- Phase 1: <small scope>
- Phase 2: <larger scope>
- Success metrics: <numbers>
- Abort criteria: <numbers>
```

### Before → After

Before：

```markdown
# Improve notification system

We have many issues with notifications. This doc proposes to use a queue and make the system better.
```

After：

```markdown
# RFC: Move email notifications to an async queue

## Summary
Recommendation: move email notification sending from the request path to an async queue, starting with password reset and invoice emails.

This reduces checkout and account API latency, isolates third-party email provider failures, and gives us retry visibility through queue metrics. The trade-off is eventual delivery: emails may be delayed by up to 2 minutes during provider degradation.

## Alternatives considered
| Option | Pros | Cons | Decision |
|---|---|---|---|
| Keep synchronous sending and add retries | Small code change | Still blocks API requests | Rejected |
| Add async queue with worker retries | Removes provider latency; visible metrics | Adds worker operations | Recommended |
| Use provider webhooks only | Less internal retry logic | Does not solve request latency | Rejected |
```

完成标准：

- [ ] Summary 有推荐方案。
- [ ] Problem 不提前写解法。
- [ ] Non-goals 防止跑题。
- [ ] Alternatives 至少 2 个。
- [ ] Risks 有 owner。
- [ ] Rollout 有成功和中止指标。

## 8. Incident Postmortem

Postmortem 服务学习和预防复发；语言必须 blameless。

### Blameless 写法

差：

```text
Bob forgot to update the config, causing the outage.
```

好：

```text
The deploy process allowed the service to start with a stale config because config validation did not check the required `PAYMENT_QUEUE_URL` value.
```

### 模板

```markdown
# Incident Postmortem: <service / symptom / date>

## Summary
<What happened, impact, duration, current status.>

## Customer impact
- Affected users: <number or percentage>
- Affected functionality: <what failed>
- Duration: <start-end, timezone>

## Timeline
| Time | Event |
|---|---|

## Root cause
<System condition that allowed the incident.>

## What went well
- <detection / rollback / communication>

## What went poorly
- <monitoring / runbook / test gap>

## Action items
| Action | Owner | Due date | Priority |
|---|---|---|---|
```

### Before → After

Before：

```markdown
Yesterday the payment service was down because the deploy was bad. We fixed it by rolling back. Need to be more careful next time.
```

After：

```markdown
## Summary
On 2026-07-06 from 09:14 to 09:47 UTC, the Payments API returned elevated 5xx responses after version `2026.07.06.3` was deployed.

Customer impact: 18% of checkout attempts failed for 33 minutes in the US region. No duplicate charges were detected.

Current status: the service recovered after rollback to `2026.07.06.2`. The root cause was missing startup validation for the new `PAYMENT_QUEUE_URL` configuration.

## Action items
| Action | Owner | Due date | Priority |
|---|---|---|---|
| Add startup validation for `PAYMENT_QUEUE_URL` and fail deployment if missing | Payments Platform | 2026-07-10 | P0 |
| Add a canary enqueue check before 100% rollout | Release Engineering | 2026-07-17 | P1 |
| Update the Payments rollback runbook | On-call rotation | 2026-07-12 | P1 |
```

完成标准：

- [ ] 有时间、影响、当前状态。
- [ ] root cause 是系统条件。
- [ ] action item 可验证。
- [ ] 不出现 blame。
- [ ] 每个行动项有 owner、due date、priority。

## 9. Slack / Async Update

Slack 目标是减少打断，减少追问，让异步协作继续前进。

### 状态模板

```text
Status: <green/yellow/red + one-line summary>

Done:
- <completed work>

Next:
- <owner + action + ETA>

Blocked:
- <blocker or none>

Ask:
- <specific request + deadline>
```

### 求助模板

```text
Context: <what you are doing>
Problem: <expected vs actual>
What I tried:
- <attempt + result>
Ask: <one specific question>
Deadline: <when this blocks work>
```

### Before → After：状态同步

Before：

```text
Hi team, I am still working on the migration. There are some issues but I think it should be okay. Will update later.
```

After：

```text
Status: yellow — the user table migration is 70% complete, but the backfill is slower than expected.

Done:
- Schema change deployed behind `user_profile_v2_enabled=false`.
- Backfilled 12M / 18M rows.

Next:
- I will reduce batch size from 5,000 to 2,000 and continue the backfill.
- New ETA: 18:00 UTC today.

Blocked:
- None. The app is still reading from the old columns.

Ask:
- @data-platform could you confirm whether the 15:00 UTC warehouse job can tolerate +20% DB read load?
```

### Before → After：求助

Before：

```text
Does anyone know why the test is failing?
```

After：

```text
Context: I am updating the payment retry worker in #1842.
Problem: `retry_service.test.ts` passes alone but fails in the full suite. Expected one retry job; actual two jobs.
What I tried:
- Cleared Jest mocks in `beforeEach`; no change.
- Checked DB cleanup; `payment_jobs` is empty before this test.
- Reproduced with `npm test -- --runInBand`.
Ask: has anyone seen shared queue state between worker tests?
Deadline: this blocks tomorrow's release PR.
```

完成标准：

- [ ] Status 有颜色和一句话。
- [ ] Done / Next / Blocked 分开。
- [ ] Ask 是一个具体问题。
- [ ] Deadline 写 timezone 或 release 约束。
- [ ] 不用 `any update???`。

## 10. Professional Email

邮件适合跨团队、客户、招聘、正式决策和需要留档的沟通。

### 模板

```text
Subject: <action / topic / deadline>

Hi <Name>,

<BLUF: why you write and what you need.>

Context:
- <background 1>
- <background 2>

Request:
- <specific ask>
- <deadline or format>

Next step:
- <what happens after reply>

Best,
<Name>
```

### Subject 公式

```text
Review request: Billing retry RFC by Jul 12
Decision needed: Queue provider for notification migration
Follow-up: API contract changes for partner integration
Incident update: Payments 5xx on Jul 6 resolved
Referral request: Senior Backend Engineer, Payments Platform
```

### Before → After：跨团队请求

Before：

```text
Subject: Question

Hi,
Hope this email finds you well. I am writing to ask about the database thing we talked about before. We need some help and it would be great if you can look at it when you have time. Thanks.
```

After：

```text
Subject: Review request: user profile backfill plan by Jul 10

Hi Data Platform team,

Could you review the user profile backfill plan by Jul 10? We want to confirm the batch size and read-load assumptions before starting production migration.

Context:
- The migration backfills 18M `users` rows into `user_profiles`.
- The proposed batch size is 2,000 rows with a 200ms pause.
- Estimated additional read load is +20% for about 4 hours.

Request:
- Please confirm whether the warehouse sync job can tolerate this load.
- If not, please suggest a safe time window or maximum batch size.

Next step:
- If approved by Jul 10, we will start the backfill on Jul 11 at 02:00 UTC.

Best,
Sha
```

### Before → After：招聘/内推

Before：

```text
Subject: Job

Hi, I saw your company has jobs. I am interested. Can you refer me? My resume is attached.
```

After：

```text
Subject: Referral request: Senior Backend Engineer, Payments Platform

Hi Maya,

I am interested in the Senior Backend Engineer role on the Payments Platform team. Would you be open to referring me if my background looks relevant?

Why I may be a fit:
- 7 years of backend experience with payment, billing, and distributed job systems.
- Led a retry queue redesign that reduced checkout p95 latency by 38%.
- Comfortable writing RFCs, PR descriptions, and postmortems in English.

Links:
- Resume: <link>
- GitHub / portfolio: <link>
- Role: <link>

If helpful, I can send a 5-bullet summary tailored to the referral form.

Best,
Sha
```

## 11. ESL 常见坑

| 错误 | 正确 | 规则 |
|---|---|---|
| We need add index. | We need to add an index. | need to + 动词；可数单数要冠词 |
| This is root cause. | This is the root cause. | 特定对象用 the |
| several issue | several issues | several 后复数 |
| each users | each user | each 后单数 |
| discuss about the plan | discuss the plan | discuss 直接接宾语 |
| explain me the issue | explain the issue to me | explain sth to sb |
| reply me | reply to me / reply in the thread | reply to |
| request for review | request a review / ask for a review | 搭配要自然 |
| please kindly review | Could you review...? | 不叠加礼貌词 |

时态模板：

```text
Past: The incident started at 09:14 UTC.
Present perfect: We have rolled back the deploy.
Present: The service is healthy now.
Future: We will add startup validation by Friday.
```

语气强度：

| 强度 | 表达 | 场景 |
|---|---|---|
| 强 | must | 安全、合规、明确要求 |
| 中 | should | 推荐方案、工程判断 |
| 弱 | could | 建议探索 |
| 不确定 | may / might | 假设 |

专业替换：

| 过度软 | 更专业 |
|---|---|
| Sorry to bother you | Could you help confirm...? |
| I am not sure if this is stupid | I may be missing context, but... |
| Any update??? | Could you share an update by 16:00 UTC? |
| Please kindly help review | Could you review this by Friday? |

## 12. 风格检查清单

句子：

- [ ] 用具体动词。
- [ ] 少用名词化。
- [ ] 一句不超过 25 个词。
- [ ] 删除 basically / actually / very / really。
- [ ] 用数字替代 vague adjectives。

段落：

- [ ] 第一段回答“所以呢”。
- [ ] 一段一个观点。
- [ ] 长背景放后面。
- [ ] 风险单独列出。
- [ ] 回滚单独列出。

语气：

- [ ] 对事不对人。
- [ ] 不隐藏不确定性。
- [ ] 不夸大确定性。
- [ ] 反对时给替代方案。
- [ ] 请求时给 deadline 和原因。

执行：

- [ ] 每个 ask 有 owner。
- [ ] 每个 deadline 有 timezone。
- [ ] 每个测试有命令或步骤。
- [ ] 每个指标有当前值和目标值。

## 13. 自我编辑循环

### 13.1 0 分钟：写 reader outcome

```text
After reading this, <reader> should <action>.
```

示例：

```text
After reading this, the reviewer should understand the retry behavior change and know which files to review first.
```

### 13.2 1–3 分钟：写粗稿

```text
Changed retry service. Added DB constraint. Tests pass. Risk queue delay. Need review before release.
```

### 13.3 4–6 分钟：套 BLUF

```text
Summary: This PR prevents duplicate payment retries by adding DB-level dedupe.
Why: In-memory dedupe is lost on pod restart.
Testing: npm test -- payments/retry_service.test.ts passed.
Ask: Please review the retry service and migration by Thursday EOD.
```

### 13.4 7–8 分钟：删 20%

删掉：

- 重复背景。
- 无证据评价。
- 客套开场。
- maybe / just / actually / basically。

### 13.5 9–10 分钟：朗读检查

问：

1. 第一口气能读完吗？
2. 哪句话没有信息增量？
3. 哪个代词指代不清？
4. 读者下一步是否明确？

AI 辅助 prompt：

```text
Rewrite this technical update using BLUF.
Keep the meaning unchanged.
Make it concise, specific, and professional.
Return the improved version, changes made, and missing information.
Draft:
<PASTE TEXT>
```

## 14. 可复制模板库

### 14.1 PR Review Request

```text
Could you review <PR link> by <deadline + timezone>?
Focus areas:
- <file/module/logic>
You can skip:
- <generated files / snapshots>
This blocks <release / migration / customer fix>.
```

### 14.2 Code Review Comment

```text
Question: <specific behavior or trade-off>
Concern: <risk or bug>
Suggestion: <concrete alternative>
Example: <small scenario>
```

### 14.3 Pushback

```text
I’d push back on <proposal> because <specific risk>.
A lower-risk option is <alternative>.
Trade-off: <what we give up>.
```

### 14.4 Decision Log

```text
Decision: <what we chose>
Context: <why needed>
Options considered:
- <A>: <accepted/rejected reason>
- <B>: <accepted/rejected reason>
Consequences:
- <benefit>
- <trade-off>
```

### 14.5 Weekly Update

```text
Week of <date>
Highlights:
- <impactful result>
Progress:
- <workstream + status>
Risks:
- <risk + mitigation>
Next week:
- <planned work>
Asks:
- <owner + request + deadline>
```

### 14.6 Incident Update

```text
Update <time UTC>: <status>
Impact:
- <users / endpoints / regions>
Mitigation:
- <what has been done>
Next update:
- <time>
Owner:
- <incident commander / on-call>
```

### 14.7 Follow-up Email

```text
Subject: Follow-up: <topic>

Hi <Name>,

Following up on <topic>. Could you share <specific response> by <deadline>?
This matters because <impact or dependency>.
If I do not hear back by then, I will <reasonable default action>.

Best,
<Name>
```

## 15. 完整实操：一个改动写成 5 类文本

素材：支付重试从 API 同步调用迁移到异步队列，防止重复扣款并降低延迟。

原始草稿：

```text
We changed payment retry. Old code was bad because it retried in API. Sometimes duplicate. New worker uses queue and db unique key. Tests passed. Need review. Risk queue delay. Rollback by flag.
```

PR、Commit、RFC、Slack、Email 都应共享同一组事实：问题、方案、测试、风险、下一步。

不要让 PR 说“无风险”，Slack 却说“queue backlog may delay retries”。

写作一致性就是工程一致性。

## 16. 常见坑与排查表

| 症状 | 原因 | 修复 |
|---|---|---|
| 别人问“你要我做什么？” | 没有 ask | 加 `Ask:` |
| Reviewer 只评论格式 | 没有 review guide | 加 `How to review` |
| RFC 讨论发散 | 没有 non-goals | 加 `Non-goals` |
| Postmortem 像甩锅 | 主语是个人 | 改成系统条件 |
| Slack 来回追问 | 缺 expected / actual / tried | 用求助模板 |
| 邮件没人回 | subject 无动作 | 写 action + deadline |
| 英文不自信 | 过度道歉 | 删 sorry / just |
| 句子很长 | 一句多个动作 | 拆 bullet |
| 结论在最后 | 按时间顺序写 | 改 BLUF |
| 判断不可信 | 没证据 | 加指标和链接 |

## 17. 练习

### 17.1 PR 重写

输入：`This PR updates user settings. I changed API and UI and fixed validation. Please review.`

交付：含 Summary、Why、What changed、Testing、Risks 的 PR 描述。

### 17.2 Slack 求助

输入：`The migration is failing. Anyone can help?`

交付：补齐 Context、Problem、What I tried、Ask、Deadline。

### 17.3 RFC 摘要

输入：`We want to improve search. Current search is slow. Maybe add cache.`

交付：Title、Summary、Goals、Non-goals、Alternatives 表格。

### 17.4 Postmortem Action Items

输入：`Someone forgot config. Need better process.`

交付：3 条 blameless action items，每条有 owner、due date、priority。

## 18. 每周训练闭环

周一：选一个真实文本。

周二：用 BLUF 重写。

周三：和团队优秀样例对比。

周四：问同事或 AI 三个问题：

```text
1. What is unclear?
2. What can be shorter?
3. What action do you think I want the reader to take?
```

周五：沉淀一个模板、一个句型、一个 before→after。

## 19. 小结

英文技术写作是工程技能。

最重要的 5 个动作：

1. BLUF 开头。
2. 金字塔组织证据。
3. 高频文体套模板。
4. 用数字、测试、owner 增加可信度。
5. 自我编辑删掉 20% 噪音。

当你的 PR、RFC、incident update 和邮件都能做到“第一屏给结论，正文给证据，结尾给行动”，你就在英语异步团队中拥有了更高的可见度和影响力。

`标签` `英语` `技术写作` `异步沟通` `Remote` `PR` `文档`

---

[← 上一章](02-english-speaking-fluency.md) · [WP-03 目录](README.md)
