[← 手册首页](../README.md) · [WP-03 英语能力](README.md)

---

# 技术面试英语：高频句型与表达

> 原则：面试英语不需要华丽，需要**清晰、结构化、边想边说（think aloud）**。背下面的句型骨架，把内容填进去；每天录音复盘，把表达练到在压力下也能自动说出来。

## 本章定位：把技术能力翻译成面试官能跟上的英语

本章不是通用口语课，而是一份给软件工程师的**技术面试英语作战手册**。目标是：在 coding、system design、behavioral 三类面试里，用稳定、简洁、结构化的英语表达你的思路、权衡、代码和影响力。

你需要的前置条件：

- 已经熟悉常见技术面试形式：coding interview、system design interview、behavioral interview。
- 能读懂英文技术文档，但开口时容易卡顿、语序混乱或过度依赖中文思维。
- 愿意每天拿手机录音 10–20 分钟，并按 rubric 自查。
- 如果你还在补系统设计框架，可配合阅读 [系统设计面试](../career/03-system-design-interview.md)。
- 如果你还在整理 STAR 故事，可配合阅读 [行为面试 STAR](../career/04-behavioral-interview-star.md)。

学完本章后，你应该能独立完成以下动作：

1. 用 60–90 秒完成一段自然的英文自我介绍。
2. 在 coding 面试中持续 think aloud，不让面试官面对沉默。
3. 用固定模板澄清需求、确认输入输出、讨论复杂度和边界条件。
4. 在 system design 面试中用英语组织 requirements → APIs → data model → high-level design → bottlenecks → trade-offs。
5. 在 behavioral 面试中用 STAR 结构讲 2–3 分钟完整故事。
6. 用录音、自评表和 shadowing 建立每日闭环，持续减少 filler words。

---

## 1. 核心心智模型：面试英语是协议，不是表演

技术面试英语的关键不是“像母语者一样说话”，而是让面试官实时知道三件事：

| 面试官想知道 | 你需要说出口的内容 | 典型句型 |
|---|---|---|
| 你是否理解问题 | 复述目标、约束、输入输出 | "Let me restate the problem in my own words." |
| 你是否有结构 | 分阶段、列选项、解释取舍 | "I’ll first solve the simple case, then optimize it." |
| 你是否能合作 | 提问、接受提示、调整方案 | "That’s a good point. I’ll update the approach accordingly." |

### 1.1 面试中的英语优先级

按优先级排序：

1. **Correctness**：表达必须准确，不能让人误解技术含义。
2. **Structure**：先讲框架，再讲细节；不要一边想一边散射式输出。
3. **Brevity**：能用一句话说清，就不要用三句话绕。
4. **Fluency**：不卡顿当然好，但不是第一优先级。
5. **Accent**：口音不是问题；模糊、吞音、重音错到影响理解才是问题。

### 1.2 面试沟通的 5 步循环

在任何问题里，都可以使用这个循环：

1. **Clarify**：澄清问题和约束。
2. **Plan**：先给方案，不急着写代码或画图。
3. **Execute**：边写边解释当前动作。
4. **Validate**：用例子、测试、复杂度、风险验证。
5. **Iterate**：根据反馈调整。

可背模板：

> "Let me first clarify the requirements. Then I’ll outline a baseline solution, discuss the complexity, and implement it. After that, I’ll test it with a few edge cases."

中文拆解：

- Let me first clarify...：我先澄清，避免误解。
- outline a baseline solution：先给基础方案，展示结构。
- discuss the complexity：主动覆盖面试评分点。
- test it with edge cases：主动验证，而不是等面试官追问。

---

## 2. 60–90 秒自我介绍模板

### 2.1 万能结构：Present → Past → Fit

不要从学校、城市、兴趣开始。技术面试自我介绍最稳的结构是：

1. **Present**：现在是谁，几年经验，主技术栈。
2. **Past**：过去做过什么有代表性的项目，最好有数字。
3. **Fit**：为什么这个岗位/团队与你匹配。

模板：

```text
I'm a [role] with [X years] of experience, mainly working on [domain / stack].
In my recent role, I focused on [area], where I [achievement with metric].
Before that, I worked on [another relevant area], which gave me experience in [skill].
I'm interested in this role because [team/product/problem] matches my background in [relevant strength], and I’d like to contribute to [specific outcome].
```

### 2.2 软件工程师版本

```text
I'm a backend engineer with six years of experience, mainly working on distributed systems and data-intensive applications.
In my recent role, I led the redesign of a job scheduling service, reducing p95 latency by 38% and cutting operational incidents by half.
Before that, I worked on API platforms and internal developer tools, which gave me strong experience in service reliability, observability, and cross-team collaboration.
I'm excited about this role because your team is solving large-scale infrastructure problems, and I think my background in scalable backend systems would be directly relevant.
```

### 2.3 AI 工程师版本

```text
I'm a software engineer focusing on LLM applications and backend systems.
Recently, I've been building retrieval-augmented generation workflows, including document ingestion, evaluation pipelines, and latency optimization.
One project I’m proud of is an internal knowledge assistant where we improved answer acceptance rate from 62% to 81% by adding better chunking, hybrid retrieval, and offline evals.
I'm interested in this team because the role combines product-oriented AI work with strong engineering requirements, which is exactly the intersection I enjoy.
```

### 2.4 自我介绍的常见坑

| 坑 | 症状 | 改法 |
|---|---|---|
| 太长 | 讲 3 分钟还没到重点 | 控制在 4 句，每句一个功能 |
| 太抽象 | "I’m passionate and hardworking" | 改成项目、数字、技术栈 |
| 像背稿 | 单调、停不下来 | 每句之间加自然停顿，允许微调 |
| 没有岗位关联 | 只讲自己，不讲为什么匹配 | 最后一句必须连接 team / product / problem |

---

## 3. Coding interview：从澄清到提交的全流程英语

Coding 面试最怕沉默。你不需要每秒都说话，但每次进入新阶段都要告诉面试官你在做什么。

### 3.1 开场澄清需求

常用句型：

- "Let me make sure I understand the problem correctly."
- "So the goal is to return [output] given [input], right?"
- "Are there any constraints on the input size?"
- "Can the input contain duplicates?"
- "Can the numbers be negative?"
- "Should I optimize for time complexity, space complexity, or readability?"
- "If there are multiple valid answers, can I return any of them?"
- "What should we return for an empty input?"

模板：

```text
Let me make sure I understand the problem correctly.
We are given [input], and we need to return [output].
A few clarifying questions:
1. Can [edge condition] happen?
2. What are the constraints on [size / range]?
3. If [ambiguous case], should we [expected behavior]?
```

### 3.2 复述问题：避免一上来写错题

模板：

```text
Let me restate it in my own words.
For each [item], we need to [operation], and the final result should be [result].
The key constraint seems to be [constraint], so a naive solution may be too slow.
```

例子：Two Sum

```text
Let me restate the problem.
We have an array of integers and a target value.
We need to return the indices of two numbers that add up to the target.
I’ll assume there is exactly one valid pair unless you want me to handle the no-solution case as well.
```

### 3.3 先给 brute force，再优化

面试官喜欢看到你从正确性走向优化，而不是直接跳到神秘答案。

```text
A brute-force approach would be to check every pair, which takes O(n squared) time and O(1) extra space.
That’s simple and correct, but we can do better.
If we store previously seen values in a hash map, then for each number we only need to check whether target minus current value has been seen.
That gives us O(n) time and O(n) space.
```

### 3.4 写代码时的旁白

写循环：

- "I’ll iterate through the array once."
- "For each element, I compute the complement."
- "Then I check whether the complement already exists in the map."
- "If it does, we can return the stored index and the current index."
- "Otherwise, I store the current value and continue."

写函数边界：

- "I’ll handle the empty input first."
- "This condition prevents an out-of-bounds access."
- "I’m using a dictionary here because lookup is expected O(1)."
- "I’ll keep the variable names explicit so the logic is easy to follow."

发现 bug：

- "I just noticed a bug here: I should check the map before inserting the current value, otherwise I might use the same element twice."
- "Let me fix that and rerun the example mentally."

### 3.5 复杂度表达

| 中文意思 | 推荐表达 | 不推荐 |
|---|---|---|
| 时间复杂度是 O(n) | "The time complexity is O(n)." | "The time cost is O(n)." |
| 空间复杂度是 O(n) | "The space complexity is O(n)." | "The space is O(n) complexity." |
| 平均 O(1) 查询 | "The lookup is expected O(1)." | "It is always O(1)." |
| 最坏情况 | "In the worst case..." | "At worst situation..." |
| 常数空间 | "It uses constant extra space." | "It uses O(1) memory space cost." |

### 3.6 边界条件 phrase bank

- "Let me test it with an empty array."
- "Now let’s test a single-element input."
- "This case has duplicate values."
- "This case includes negative numbers."
- "This case checks whether we accidentally reuse the same index."
- "The algorithm still works because the map only contains elements we have already visited."

### 3.7 卡住时的救场语言

不要说 "I have no idea" 然后沉默。用下面的顺序：承认 → 缩小问题 → 暴露思路 → 请求小提示。

```text
I’m not immediately seeing the optimal solution, but let me reason from a simpler version.
If the input were smaller, brute force would work by [method].
The bottleneck is [bottleneck], so I’m looking for a way to avoid [repeated work].
One possible direction is [data structure / pattern].
Does that sound like a reasonable direction, or would you like me to think about another approach?
```

更短版本：

- "I’m not fully sure yet, but I’ll think out loud."
- "The bottleneck seems to be repeated scanning."
- "Maybe a hash map / heap / two-pointer approach can remove that repeated work."
- "Could you give me a small hint about the expected direction?"

### 3.8 完整 coding interview 对话脚本：Two Sum

> 练习方式：先读一遍；第二遍遮住中文解释，只跟读英文；第三遍把题目换成 Valid Parentheses 或 Top K Frequent Elements，套同样结构。

```text
Interviewer: Let's solve Two Sum. Given an array of integers and a target, return the indices of two numbers that add up to the target.

Candidate: Sure. Let me make sure I understand the problem correctly.
We are given an array of integers and a target value, and we need to return two indices whose values sum to the target.
Can I assume there is exactly one valid answer?

Interviewer: Yes, exactly one answer exists.

Candidate: Great. Can the input contain negative numbers or duplicates?

Interviewer: Yes, both are possible.

Candidate: Got it. I’ll first describe a brute-force solution, then optimize it.
A brute-force solution would check every pair of numbers. That would be O(n squared) time and O(1) extra space.
Since we only need to find a complement for each number, we can use a hash map from value to index.
For each number x, I’ll check whether target minus x has already appeared. If yes, I return those two indices. If not, I store x with its index.
This should run in O(n) time and O(n) space.

Interviewer: Sounds good. Please implement it.

Candidate: I’ll write a function called twoSum. I’ll create an empty map called seen.
Then I’ll iterate through the array with both index and value.
For each value, I compute complement = target - value.
If complement is in seen, I return [seen[complement], current index].
Otherwise, I store the current value.
One detail: I should check before inserting, so I don’t use the same element twice.

Candidate: Here is the code.

Candidate: Let me test it quickly with nums = [2, 7, 11, 15] and target = 9.
At index 0, value is 2, complement is 7, not found, so we store 2 -> 0.
At index 1, value is 7, complement is 2, found at index 0, so we return [0, 1].
That matches the expected output.

Interviewer: What about duplicates?

Candidate: Good question. For nums = [3, 3] and target = 6, at index 0 we store 3 -> 0.
At index 1, complement is 3, and it exists in the map, so we return [0, 1].
So duplicates are handled correctly.

Interviewer: What is the complexity?

Candidate: The time complexity is O(n), because we scan the array once and each hash map lookup is expected O(1).
The space complexity is O(n) in the worst case, because we may store up to n values in the map.
```

### 3.9 可套用的 coding 结束语

```text
To summarize, the algorithm uses [data structure / pattern] to avoid [bottleneck].
It handles [edge cases], and the final complexity is [time] time and [space] space.
```

---

## 4. System design interview：用英语组织大题

系统设计英语的目标是“可跟踪”。面试官应该能清楚知道你现在处于哪个阶段：需求、容量估算、API、数据模型、架构、扩展、权衡。

更多系统设计框架见 [系统设计面试](../career/03-system-design-interview.md)。本章专注英文表达。

### 4.1 45 分钟系统设计时间盒

| 时间 | 目标 | 推荐英文 |
|---|---|---|
| 0–5 分钟 | 澄清需求 | "Before jumping into the design, I’d like to clarify the requirements and scale." |
| 5–10 分钟 | 容量估算 | "Let me do a rough back-of-the-envelope estimate." |
| 10–15 分钟 | API / 数据模型 | "I’ll define the main APIs and data entities first." |
| 15–25 分钟 | 高层架构 | "At a high level, the system has these components." |
| 25–35 分钟 | 深挖瓶颈 | "The likely bottleneck is [X], so let’s zoom in there." |
| 35–42 分钟 | 权衡和故障 | "Here are the trade-offs and failure modes." |
| 42–45 分钟 | 总结 | "Let me summarize the final design and why I made these choices." |

### 4.2 需求澄清 phrase bank

功能需求：

- "Who are the main users of the system?"
- "What are the core actions users need to perform?"
- "Should we support real-time updates, or is eventual consistency acceptable?"
- "Do we need search, filtering, recommendations, or analytics?"
- "Is this for a global audience or a single region?"

非功能需求：

- "What is the expected scale in terms of daily active users or QPS?"
- "What latency target should we design for?"
- "What availability target are we aiming for?"
- "How important is consistency compared with availability?"
- "Are there compliance or privacy constraints?"

确认范围：

- "To keep the discussion focused, I’ll prioritize [core feature] and treat [secondary feature] as out of scope unless you want to include it."
- "I’ll assume read traffic is much higher than write traffic. Please correct me if that assumption is wrong."

### 4.3 容量估算表达

```text
Let me make a rough estimate.
Assume we have 10 million daily active users.
If each user creates 2 requests per day on average, that is 20 million requests per day.
Dividing by 86,400 seconds, the average QPS is about 230.
If peak traffic is 10 times the average, we should design for roughly 2,300 QPS.
```

常用数字读法：

- 1,000 = one thousand
- 10,000 = ten thousand
- 100,000 = one hundred thousand
- 1,000,000 = one million
- 10,000,000 = ten million
- 1,000,000,000 = one billion
- 2,300 QPS = twenty-three hundred QPS / two thousand three hundred QPS

### 4.4 API 设计表达

```text
I’ll define a small set of APIs first, because that helps clarify the data flow.
For creating a resource, we can use POST /v1/items.
For reading a resource, we can use GET /v1/items/{id}.
For listing resources, we need pagination, so I’d include limit and cursor parameters.
```

可套用 API 模板：

```text
POST /v1/[resource]
Request: { [field]: [type], ... }
Response: { id: string, status: string }

GET /v1/[resource]/{id}
Response: { [field]: [type], ... }

GET /v1/[resource]?limit=50&cursor=...
Response: { items: [...], nextCursor: string | null }
```

### 4.5 数据模型表达

- "The core entities are User, Item, and Event."
- "I’d use a relational database for strongly consistent metadata."
- "For high-volume append-only events, a log-based or NoSQL store may be more appropriate."
- "The primary key is [id], and we need an index on [field] because the main query pattern is [query]."
- "I’ll avoid premature denormalization, but I may add a read-optimized table for the hot path."

### 4.6 架构组件表达

- "Clients send requests to an API gateway."
- "The gateway handles authentication, rate limiting, and routing."
- "The application service contains the business logic."
- "We use a cache to reduce read latency and database load."
- "A message queue decouples write requests from asynchronous processing."
- "Workers consume messages and update derived data."
- "Object storage is used for large binary files."
- "A CDN serves static content close to users."

### 4.7 权衡表达：不要只说 trade-off，要说清代价

模板：

```text
There is a trade-off between [A] and [B].
If we choose [option 1], we get [benefit], but we pay the cost of [cost].
If we choose [option 2], we get [benefit], but [risk / limitation].
Given the requirement of [requirement], I would choose [decision].
```

常见权衡：

| 权衡 | 英文表达 |
|---|---|
| 一致性 vs 可用性 | consistency versus availability |
| 延迟 vs 成本 | latency versus cost |
| 简单性 vs 可扩展性 | simplicity versus scalability |
| 写入性能 vs 查询性能 | write performance versus query performance |
| 同步处理 vs 异步处理 | synchronous processing versus asynchronous processing |
| 规范化 vs 反规范化 | normalization versus denormalization |

### 4.8 完整 system design 对话脚本：设计 URL Shortener

```text
Interviewer: Design a URL shortener like bit.ly.

Candidate: Sure. Before jumping into the design, I’d like to clarify the requirements and scale.
The core feature is that users submit a long URL and receive a short URL. When someone visits the short URL, the system redirects them to the original URL. Is that correct?

Interviewer: Yes.

Candidate: Do we need custom aliases, expiration dates, or analytics such as click counts?

Interviewer: Custom aliases are optional. Basic click counts would be nice to have.

Candidate: Got it. I’ll focus on URL creation, redirection, and basic click tracking. For scale, should I assume read traffic is much higher than write traffic?

Interviewer: Yes, reads are much higher.

Candidate: Let me make a rough estimate. Suppose we have 10 million new URLs per month. That’s about 4 writes per second on average. Redirections may be much higher, say 10,000 QPS at peak. So the read path needs to be highly optimized.

Candidate: For APIs, I’d define POST /v1/urls with a longUrl and optional customAlias. The response returns the shortCode.
For redirection, GET /{shortCode} returns a 301 or 302 redirect to the long URL.
For analytics, GET /v1/urls/{shortCode}/stats returns basic click counts.

Candidate: The main data model is a UrlMapping table with shortCode, longUrl, createdAt, expiresAt, and ownerId.
The shortCode is the primary key. We also need an index on ownerId if users list their URLs.
For click tracking, I wouldn’t update the main row synchronously for every redirect because that would add latency and create a write hotspot.
Instead, I’d publish click events to a queue and aggregate them asynchronously.

Interviewer: How do you generate the short code?

Candidate: There are a few options. One is to generate a random base62 string and check for collisions. Another is to use an auto-increment ID and encode it in base62.
The random approach is simple and avoids exposing sequence information, but we need collision checks.
The sequential approach is deterministic and compact, but it may reveal traffic volume.
Given this is a public URL shortener, I’d choose random base62 with a uniqueness check.

Interviewer: What about the read path?

Candidate: The read path should be very fast. A request goes through the load balancer to the redirect service.
The redirect service first checks a distributed cache, such as Redis, using shortCode as the key.
If the mapping is found, it returns a redirect immediately.
If it is a cache miss, the service reads from the database, populates the cache, and then redirects.
For very hot links, the cache will absorb most traffic.

Interviewer: What are the failure modes?

Candidate: If Redis is down, the system should fall back to the database, though latency and database load will increase.
If the database is temporarily unavailable, new URL creation may fail, but cached redirections can still work for hot URLs.
For analytics, if the queue is delayed, click counts may be stale, but redirection should not be blocked.
So I’d treat analytics as eventually consistent.

Interviewer: Can you summarize the design?

Candidate: Sure. The design has a write path for creating short URLs, a read-optimized redirect path using Redis cache, and an asynchronous analytics pipeline using a queue and workers.
I chose random base62 codes to avoid exposing sequence information.
The main trade-off is that analytics are eventually consistent, but the redirect path remains low-latency and highly available.
```

---

## 5. Behavioral interview：STAR 英语答案

行为面不是闲聊，而是在验证你是否能在真实团队里解决复杂问题。准备方法见 [行为面试 STAR](../career/04-behavioral-interview-star.md)，本章提供可直接背诵的英语骨架。

### 5.1 STAR 的英文骨架

```text
Situation: At [company/team], we were facing [problem/context].
Task: My responsibility was to [your responsibility], with the goal of [goal].
Action: I took three steps. First, [action 1]. Second, [action 2]. Third, [action 3].
Result: As a result, [measurable outcome]. I also learned [lesson], which I later applied to [later situation].
```

### 5.2 STAR 时间分配

| 部分 | 时长 | 内容 |
|---|---:|---|
| Situation | 20–30 秒 | 背景，不要讲公司八卦 |
| Task | 10–20 秒 | 你的责任，而不是团队泛泛目标 |
| Action | 60–90 秒 | 你具体做了什么，按 2–3 步讲 |
| Result | 20–30 秒 | 数字结果、业务影响、学习 |

### 5.3 常见行为题 phrase bank

冲突：

- "We had different opinions, but we aligned on the shared goal first."
- "I brought data into the discussion instead of making it personal."
- "I proposed a small experiment so we could validate the trade-off quickly."

失败：

- "The initial approach did not work as expected."
- "I took responsibility for the gap and changed the plan in two ways."
- "The main lesson I learned was to validate assumptions earlier."

领导力：

- "I did not have formal authority, so I focused on creating clarity and momentum."
- "I wrote a short design proposal and asked for feedback from the stakeholders."
- "I broke the work into milestones and made progress visible."

压力：

- "I separated the urgent issue from the important long-term fix."
- "First, we mitigated the customer impact. Then we worked on the root cause."
- "After the incident, I documented the timeline and created follow-up actions."

### 5.4 完整 behavioral STAR 脚本：处理技术分歧

```text
Interviewer: Tell me about a time when you had a disagreement with a teammate.

Candidate: Sure. In my previous team, we were redesigning a background job processing system. The system had frequent delays during peak hours, and we needed to improve reliability before a major customer launch.

My teammate wanted to rewrite the worker service from scratch, while I believed we should first identify the bottleneck and make targeted changes. My responsibility was to lead the technical investigation and propose a plan that the team could execute within three weeks.

I took three steps.
First, I collected data from logs, metrics, and queue latency dashboards. The data showed that the main bottleneck was not the worker code itself, but uneven partitioning and a few slow downstream API calls.
Second, I wrote a short design proposal comparing two options: a full rewrite and an incremental fix. For each option, I listed the expected impact, delivery risk, and operational risk.
Third, I suggested a compromise: we would rebalance partitions, add timeout and retry controls for the slow API calls, and isolate the highest-volume jobs into a separate worker pool. At the same time, we documented the rewrite idea as a longer-term improvement.

As a result, we reduced p95 job completion time by about 45% and completed the work before the customer launch. More importantly, the disagreement became productive because we focused on evidence and shared goals rather than personal preferences.
The lesson I learned was that technical disagreements are easier to resolve when we make assumptions explicit and compare options with data.
```

### 5.5 STAR 自检清单

- [ ] 我有没有说明具体场景，而不是抽象描述？
- [ ] 我有没有说明“我”的责任，而不是只说“我们”？
- [ ] Action 是否有 2–3 个清晰步骤？
- [ ] Result 是否有数字、范围或可验证结果？
- [ ] 最后一两句是否体现学习和成长？
- [ ] 全文是否控制在 2–3 分钟？

---

## 6. Clarifying requirements：澄清问题句库

### 6.1 通用澄清

- "Could you clarify what you mean by [term]?"
- "When you say [X], do you mean [A] or [B]?"
- "Is [case] in scope for this problem?"
- "Should I handle [edge case], or can I assume [simpler assumption]?"
- "What behavior would you expect if [condition]?"
- "Are we optimizing for [latency / throughput / memory / simplicity]?"

### 6.2 确认理解

```text
Just to confirm, the expected behavior is:
- If [condition A], we should [behavior A].
- If [condition B], we should [behavior B].
Is that correct?
```

### 6.3 主动声明假设

```text
I’ll make two assumptions to move forward.
First, [assumption 1].
Second, [assumption 2].
If either assumption is wrong, I can adjust the design.
```

### 6.4 面试官给模糊问题时

```text
This is a broad problem, so I’ll narrow it down first.
I’ll focus on [scope], and I’ll treat [out-of-scope item] as a future extension.
```

---

## 7. Thinking out loud：边想边说模板

### 7.1 从观察开始

- "The first thing I notice is..."
- "The constraint suggests that..."
- "The tricky part is..."
- "This looks similar to [known pattern], but with one difference: [difference]."

### 7.2 从瓶颈开始

```text
The brute-force solution repeats [operation] many times.
If we can avoid that repeated work, we can improve the time complexity.
A useful data structure here might be [hash map / heap / stack / trie], because it supports [operation] efficiently.
```

### 7.3 从不变量开始

```text
I want to maintain an invariant:
At each step, [condition] remains true.
This helps because [reason].
```

### 7.4 从例子开始

```text
Let me walk through a small example.
If the input is [example], then after the first step we have [state].
After the second step, [state].
This suggests that the algorithm should [idea].
```

### 7.5 停顿时的桥接句

- "Let me pause for a second and check the logic."
- "I want to make sure I’m not missing an edge case."
- "I’m going to trace the example once more."
- "The current approach works for [case], but I’m not sure about [case] yet."

---

## 8. Narrating code：代码旁白句库

### 8.1 变量和数据结构

- "I’ll use `left` and `right` pointers to represent the current window."
- "This set stores values we have already visited."
- "This map stores the latest index for each character."
- "This heap keeps the smallest element at the top."
- "This queue is used for breadth-first traversal."

### 8.2 条件判断

- "If this condition is true, we have found a valid answer."
- "Otherwise, we need to keep searching."
- "This branch handles the empty input case."
- "This check prevents duplicate processing."

### 8.3 循环和递归

- "The loop runs once for each element."
- "The recursion stops when we reach a null node."
- "After processing the current node, we recursively process the left and right children."
- "We update the window until it becomes valid again."

### 8.4 测试代码

- "Now I’ll test the normal case."
- "Next, I’ll test the smallest input."
- "This test covers duplicates."
- "This test covers the no-solution case, if that is allowed."

---

## 9. Discussing trade-offs：权衡句库

### 9.1 三句式

```text
Option A gives us [benefit], but it has [cost].
Option B gives us [different benefit], but it has [different cost].
Given [requirement], I would choose [option] because [reason].
```

### 9.2 技术取舍例句

- "A relational database gives us strong consistency and flexible queries, but horizontal scaling can be harder."
- "A NoSQL store can handle high write throughput, but we may lose some query flexibility."
- "Caching reduces latency, but it introduces cache invalidation complexity."
- "Asynchronous processing improves responsiveness, but users may see eventual consistency."
- "Precomputing results makes reads faster, but writes become more expensive."
- "A monolith is simpler to operate at the beginning, but service boundaries may become harder to manage as the team grows."

### 9.3 不确定时如何表达

```text
I don’t think there is a universally best choice here.
It depends on [requirement].
If [condition], I would choose [A].
If [different condition], I would choose [B].
For the current requirements, my recommendation is [choice].
```

---

## 10. Handling "I don't know"：不知道也要有结构

面试里承认不知道是可以的，但必须展示推理能力和学习能力。

### 10.1 不要这样说

- "I don’t know." 然后沉默。
- "I’ve never used it." 然后结束。
- "Maybe it’s not important." 否定问题。

### 10.2 推荐三段式

```text
I haven’t worked with [technology] directly.
However, based on my understanding of [related concept], I would expect it to [reasonable inference].
If I needed to use it in production, I would verify [specific thing] by [documentation / prototype / benchmark].
```

### 10.3 例子：没用过 Kafka

```text
I haven’t operated Kafka in production directly.
However, I have worked with message queues and event-driven systems, so I understand the general concerns: delivery semantics, ordering, retries, and consumer lag.
If I were designing with Kafka, I would verify partitioning strategy, retention settings, and consumer group behavior before making production decisions.
```

### 10.4 例子：不会某个算法

```text
I don’t remember the exact algorithm name right now, but I can reason from the constraints.
The input size suggests that O(n squared) may be too slow.
I’ll first build a correct baseline, then look for repeated work that can be eliminated with a better data structure.
```

---

## 11. Asking the interviewer questions：把面试官变成协作者

### 11.1 Coding 中可以问的问题

- "Would you like me to implement the optimized solution directly, or start with the brute-force version?"
- "Should I write production-style error handling, or focus on the core algorithm?"
- "Do you want me to run through more test cases?"
- "Is there a particular constraint you’d like me to optimize for?"

### 11.2 System design 中可以问的问题

- "Which part would you like me to go deeper into: storage, caching, or failure handling?"
- "Should we assume a single region or multi-region deployment?"
- "Is strong consistency required for this operation?"
- "Would it be acceptable if analytics are delayed by a few minutes?"

### 11.3 Behavioral 中可以问的问题

- "Would you like an example focused on technical leadership or cross-team collaboration?"
- "Should I go deeper into the technical details or the communication side?"
- "Does that answer your question, or would you like another example?"

### 11.4 反问面试官

一定要问，展示投入：

- "What does success look like for this role in the first six months?"
- "How does the team balance shipping speed with technical quality?"
- "What are the biggest technical challenges the team is facing right now?"
- "How are design decisions usually made on the team?"
- "What does the onboarding process look like for a new engineer?"
- "How does the team measure the impact of engineering work?"
- "What would make someone really successful in this role?"

---

## 12. Pronunciation and pitfall notes：常见技术词发音与误区

> 目标不是消除口音，而是避免把关键词说到面试官需要猜。

| 词 | 推荐读法提示 | 常见坑 | 练习句 |
|---|---|---|---|
| cache | "cash" | 读成 "catch" | "The cache reduces database load." |
| queue | "kyoo" | 读出多余音节 | "The queue decouples producers and consumers." |
| archive | 名词常见 "AR-kive" | 重音漂移 | "Old logs are moved to archive storage." |
| hierarchy | "HI-er-ar-kee" | 漏掉中间音 | "The hierarchy is represented as a tree." |
| idempotent | "eye-dem-PO-tent" | 不会读就跳过 | "The retry operation should be idempotent." |
| throughput | "THROO-put" | 和 "output" 混 | "We optimize throughput, not just latency." |
| latency | "LAY-ten-see" | 重音不稳 | "The p95 latency increased during peak traffic." |
| consistency | "con-SIS-ten-see" | 和 availability 混说 | "This is a consistency trade-off." |
| availability | "a-vail-a-BIL-i-tee" | 过快吞音 | "The service has high availability." |
| schema | "SKEE-ma" 或 "SHEE-ma" | 两种均可，不要犹豫 | "The schema includes userId and createdAt." |
| daemon | "DEE-mun" | 读成 day-mon | "A background daemon processes jobs." |
| regex | "REG-ex" | 读太快不清楚 | "We can validate the input with a regex." |
| SQL | "S-Q-L" 或 "sequel" | 同场景来回切换 | "I’ll use SQL for relational queries." |
| tuple | "TOO-pul" 或 "TUH-pul" | 两种均可 | "The function returns a tuple." |
| boolean | "BOO-lee-an" | 读成 boo-LEN | "This flag is a boolean value." |

### 12.1 中文母语者常见语法坑

| 想表达 | 不自然表达 | 推荐表达 |
|---|---|---|
| 我有 6 年经验 | I have 6 years experiences | I have 6 years of experience |
| 这个需求是什么 | What is the requirement means? | What does this requirement mean? |
| 我负责做后端 | I’m responsible to backend | I’m responsible for the backend service |
| 这个方案更好 | This way is more better | This approach is better |
| 我们讨论一下复杂度 | Let’s discuss about complexity | Let’s discuss the complexity |
| 我遇到一个问题 | I met a problem | I ran into a problem |
| 代码可以工作 | The code can work | The code works |
| 数据很多 | The data is very many | There is a large amount of data |

### 12.2 重音练习

每天选 5 个词，按以下节奏读：

```text
cache, cache, cache.
The cache reduces database load.
A distributed cache reduces database load.
A distributed cache reduces database load during peak traffic.
```

完成标准：录音回听时，关键词不含糊，句尾不吞掉。

---

## 13. Filler-word reduction drills：减少 "um / like / you know"

Filler words 不是完全不能有，但密度太高会削弱结构感。目标：每 60 秒不超过 3 个明显 filler。

### 13.1 常见 filler 替代表达

| 场景 | 不推荐 | 推荐 |
|---|---|---|
| 争取时间 | "um... like..." | "Let me think for a moment." |
| 换方向 | "so yeah" | "Let me approach it from another angle." |
| 修正 | "I mean..." | "Let me rephrase that." |
| 检查 | "you know" | "Let me verify the assumption." |
| 卡住 | 长时间沉默 | "I’m not fully sure yet, but here is how I’m thinking about it." |

### 13.2 30 秒静默停顿训练

步骤：

1. 打开手机录音。
2. 选择一个题目："Explain how a cache works."
3. 讲 30 秒。
4. 每次想说 "um" 时，改成停顿 1 秒。
5. 回听并计数：um、uh、like、you know、so yeah。
6. 重录一次，目标减少 50%。

脚本：

```text
A cache is a faster storage layer placed in front of a slower system.
The goal is to reduce latency and load.
For example, instead of reading user profiles from the database every time, we can store frequently accessed profiles in Redis.
The trade-off is that cached data may become stale, so we need an invalidation or expiration strategy.
```

### 13.3 结构词替代 filler

背这些词，让大脑有“轨道”：

- "First,... Second,... Finally,..."
- "At a high level,..."
- "The main trade-off is..."
- "One edge case is..."
- "The reason is..."
- "To summarize,..."

---

## 14. 每日 practice loop：shadowing → recording → self-review

每天 20 分钟即可。关键是闭环，不是单纯听英语。

### 14.1 20 分钟训练流程

| 时间 | 动作 | 产出 |
|---|---|---|
| 0–3 分钟 | 读今天的脚本 | 标出不会读的词 |
| 3–8 分钟 | shadowing：跟读 3 遍 | 语速和停顿更接近原文 |
| 8–13 分钟 | 关掉原文，按模板复述 | 1 段自由输出 |
| 13–17 分钟 | 录音并回听 | 找 3 个问题 |
| 17–20 分钟 | 重录一版 | 保存最佳版本 |

### 14.2 Shadowing 方法

1. 选 60–90 秒英文材料：可以是本章脚本，也可以是技术 talk 的一小段。
2. 第一遍只听节奏，不追求完全跟上。
3. 第二遍延迟 0.5 秒跟读，模仿停顿和重音。
4. 第三遍看稿跟读，修正发音。
5. 第四遍不看稿复述，允许换词，但保留结构。

### 14.3 录音命名规则

用固定命名，便于看到进步：

```text
YYYY-MM-DD_coding_twosum_v1.m4a
YYYY-MM-DD_systemdesign_urlshortener_v1.m4a
YYYY-MM-DD_behavioral_conflict_v1.m4a
```

### 14.4 自评 rubric

每项 1–5 分：

| 维度 | 1 分 | 3 分 | 5 分 |
|---|---|---|---|
| Structure | 散乱 | 有阶段但转场生硬 | 开头、过程、总结清晰 |
| Clarity | 需要猜 | 大意清楚 | 技术含义准确 |
| Conciseness | 绕、重复 | 偶尔重复 | 每句都有功能 |
| Fluency | 长时间卡顿 | 有停顿但能继续 | 停顿自然，不影响理解 |
| Filler control | filler 很多 | 偶尔出现 | 每分钟不超过 3 个 |
| Technical precision | 术语混乱 | 基本准确 | 复杂度、权衡、边界清楚 |

记录格式：

```text
Date:
Prompt:
Score: Structure __ / Clarity __ / Conciseness __ / Fluency __ / Filler __ / Precision __
Three issues:
1.
2.
3.
Next attempt focus:
```

---

## 15. 一周训练计划

### Day 1：自我介绍

- 产出：60–90 秒 self-introduction。
- 练习：录 3 版，每版删掉一句废话。
- 完成标准：不看稿能讲完；有岗位连接句。

### Day 2：Coding 澄清 + brute force + optimize

- 题目：Two Sum / Valid Parentheses / Merge Intervals。
- 产出：每题 2 分钟英文思路。
- 完成标准：每题包含 clarifying question、baseline、optimized idea、complexity。

### Day 3：Coding 代码旁白

- 题目：任选一道已会的题。
- 产出：边写代码边说 5–8 分钟。
- 完成标准：没有超过 10 秒沉默；能解释每个关键变量。

### Day 4：System design 需求澄清

- 题目：URL shortener / chat app / notification system。
- 产出：5 分钟 requirements + scale。
- 完成标准：至少问 5 个澄清问题，明确 scope。

### Day 5：System design trade-offs

- 题目：继续 Day 4。
- 产出：5 分钟架构权衡。
- 完成标准：说出至少 3 个 trade-off，每个包含 benefit 和 cost。

### Day 6：Behavioral STAR

- 题目：conflict / failure / leadership。
- 产出：2–3 分钟 STAR 故事。
- 完成标准：Action 有三步，Result 有数字或明确影响。

### Day 7：Mock interview

- 产出：30 分钟模拟面试录音。
- 完成标准：复盘 3 个高频问题，写下下周要修的 3 个表达。

---

## 16. 可打印速查卡

### 16.1 Coding 速查卡

```text
Clarify:
Let me make sure I understand the problem correctly...
Can the input contain...?
What should we return if...?

Plan:
A brute-force approach would be...
We can optimize it using...
The key idea is...

Implement:
I’ll iterate through...
I’ll store...
This condition handles...

Validate:
Let me test it with...
The time complexity is...
The space complexity is...
```

### 16.2 System design 速查卡

```text
Before jumping into the design, I’d like to clarify the requirements and scale.
I’ll assume [assumption]. Please correct me if that’s wrong.
At a high level, the system has [components].
The main bottleneck is likely [bottleneck].
There is a trade-off between [A] and [B].
Given [requirement], I would choose [decision].
```

### 16.3 Behavioral 速查卡

```text
At [team], we faced [problem].
My responsibility was to [task].
I took three steps.
First, [action 1].
Second, [action 2].
Third, [action 3].
As a result, [metric / impact].
The lesson I learned was [lesson].
```

---

## 17. 完整实操：一次 45 分钟 mock 面试怎么做

### 17.1 准备材料

- 一道你已会的 coding 题。
- 一个 system design 题目。
- 一个 STAR 故事。
- 手机录音或电脑录屏。
- 本章速查卡。

### 17.2 时间安排

| 时间 | 内容 |
|---|---|
| 0–5 分钟 | 自我介绍 + 面试开场 |
| 5–25 分钟 | Coding：澄清、思路、代码、测试、复杂度 |
| 25–38 分钟 | System design：需求、架构、权衡 |
| 38–43 分钟 | Behavioral STAR |
| 43–45 分钟 | 反问 + 总结 |

### 17.3 复盘方法

回放录音，做三轮标注：

1. 第一轮只听结构：有没有开头、过渡、总结。
2. 第二轮只听 filler：每出现一次 um / uh / like 就画一笔。
3. 第三轮只听技术准确性：复杂度、术语、权衡是否说对。

输出一份复盘：

```text
Mock date:
Strongest segment:
Weakest segment:
Top 3 repeated fillers:
One unclear technical explanation:
One sentence to memorize for next time:
Next mock focus:
```

---

## 18. 常见坑 & 排查表

| 症状 | 可能原因 | 立即修复 |
|---|---|---|
| 一紧张就沉默 | 没有阶段句 | 背 "Let me think out loud" 和 "The bottleneck is..." |
| 讲很多但没结构 | 没有编号 | 强制使用 First / Second / Finally |
| 复杂度说错 | 没有提前准备术语 | 背 complexity 表达，并在刷题后口头总结 |
| 系统设计像背八股 | 没有连接需求 | 每个组件后说 "because the requirement is..." |
| STAR 像流水账 | Action 没有拆步骤 | 固定说 "I took three steps" |
| 发音影响理解 | 高频词没练 | 每天 5 个技术词，短句 repeat |
| filler 太多 | 害怕停顿 | 用 1 秒静默替代 um |
| 回答太绝对 | 没有 trade-off | 加 "It depends on..." 和条件判断 |
| 听不懂问题 | 不敢澄清 | 说 "Could you rephrase that?" |
| 反问太泛 | 没准备 | 背 3 个团队、技术、成功标准问题 |

---

## 19. 检查清单：做到这些才算掌握

- [ ] 我能 90 秒内完成自我介绍，并自然连接岗位。
- [ ] 我能在 coding 题开始前问至少 3 个澄清问题。
- [ ] 我能用英语解释 brute force 和 optimized solution 的区别。
- [ ] 我能边写代码边解释变量、循环、条件和边界。
- [ ] 我能说出 time complexity 和 space complexity，语法正确。
- [ ] 我能在卡住时继续 think aloud，而不是沉默。
- [ ] 我能在 system design 中说清 requirements、scale、API、data model、architecture、trade-offs。
- [ ] 我能讲一个 2–3 分钟 STAR 故事，并包含 quantified result。
- [ ] 我能主动问面试官高质量问题。
- [ ] 我每周至少完成一次录音复盘，并记录下一次改进点。

---

## 20. 里程碑作业

### Level 1：建立最小表达库

产出物：

- 1 段 self-introduction。
- 10 句 clarifying questions。
- 10 句 code narration。
- 5 句 trade-off sentences。
- 1 个 STAR 故事。

完成标准：不看本章正文，只看速查卡能说出来。

### Level 2：完成三段录音

产出物：

1. Coding：Two Sum 8 分钟 walkthrough。
2. System design：URL shortener 10 分钟 discussion。
3. Behavioral：conflict STAR 3 分钟 answer。

完成标准：每段按 rubric 自评，所有维度至少 3 分。

### Level 3：模拟真实面试

产出物：

- 45 分钟 mock interview 录音。
- 一页复盘：3 个优势、3 个问题、3 个下周训练目标。

完成标准：filler words 每分钟不超过 3 个；没有超过 10 秒的无解释沉默。

---

## 21. 延伸阅读与配套章节

配套章节：

- [海外与 Remote 求职策略](../career/02-remote-overseas-job-strategy.md)
- [系统设计面试](../career/03-system-design-interview.md)
- [行为面试 STAR](../career/04-behavioral-interview-star.md)
- [英语口语流利度训练](02-english-speaking-fluency.md)
- [技术写作英语](03-technical-writing-english.md)

外部资源：

- [Google Technical Development Guide](https://techdevguide.withgoogle.com/)
- [Interviewing at Meta](https://www.metacareers.com/careerprograms/pathways/interviewing/)
- [Amazon Leadership Principles](https://www.amazon.jobs/content/en/our-workplace/leadership-principles)
- [Pramp: Practice Mock Interviews](https://www.pramp.com/)
- [MIT Communication Lab](https://mitcommlab.mit.edu/)
- [The System Design Primer](https://github.com/donnemartin/system-design-primer)

`标签` `英语` `面试` `技术沟通` `口语`

---

[WP-03 目录](README.md) · [下一章 →](02-english-speaking-fluency.md)
