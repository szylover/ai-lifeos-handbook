[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 系统设计能力：像架构师一样思考

> 结论先行：系统设计能力不是“知道很多中间件”，而是把模糊目标转成**边界清楚、数字可验证、风险可控、能持续演进**的工程方案。它让你在远程团队、咨询项目、独立产品和架构评审中，不靠坐班曝光也能证明自己的判断力。

相关阅读：[转型 AI 的职业路径](01-career-pivot-to-ai.md)、[LLM 应用架构](../ai-engineering/02-llm-app-architecture.md)、[行为面试 STAR](04-behavioral-interview-star.md)。

## 本章定位：从“会画图”到“能主导架构决策”

系统设计是《逃离坐班白皮书》里的职业杠杆：你能接手复杂问题、拆出约束、做出取舍，并把方案写成别人愿意执行的设计文档。它适用于远程 Senior / Staff / Tech Lead 工作、咨询交付、独立产品技术选型，以及 AI 应用的延迟、成本、可靠性设计。

前置知识：HTTP、REST/gRPC、SQL 基础、缓存、消息队列、分布式系统基本概念。学完本章，你应该能：

- 用“澄清 → 估算 → API → 数据模型 → 架构 → 深挖 → 权衡”的流程设计中大型系统。
- 用 back-of-envelope 数字估出 QPS、存储、带宽、缓存容量，并让数字驱动架构。
- 对缓存、分片、复制、队列、负载均衡、一致性、CAP 做“何时用 + 代价是什么”的判断。
- 写出一页高信号设计文档，拿去做 architecture review。
- 从头到尾讲清短链接系统和新闻 Feed 系统的关键设计。

---

## 1. 架构师工作流：先定边界，再做取舍

系统设计最常见的失败不是“不懂 Kafka”，而是过早跳到组件。正确顺序：

```text
1. 澄清需求：功能、非功能、Out of Scope
2. 容量估算：QPS、存储、带宽、热点、增长
3. 定义接口：外部 API、错误码、幂等、分页
4. 数据建模：实体、主键、索引、分区键、生命周期
5. 高层架构：组件图、读写路径、同步/异步边界
6. 深挖风险：缓存、分片、复制、队列、一致性、故障
7. 权衡演进：当前取舍、10× 流量路径、监控和回滚
```

### 1.1 需求澄清：把“大题”变成可落地问题

按三个层次问，不要一次抛 20 个问题。

| 层次 | 要确认什么 | 示例问题 |
| --- | --- | --- |
| 功能 | 核心用户、2–3 个 use cases、读写路径 | “必须支持哪些动作？读多还是写多？是否需要搜索、推荐、权限、统计？” |
| 非功能 | 规模、延迟、可用性、一致性、保留周期 | “P95 要 100ms、300ms 还是 1s？新数据必须强一致还是秒级最终一致可接受？” |
| Out of Scope | 暂时不做什么 | “先不做复杂风控、广告、企业权限，只保留主路径和可扩展接口。” |

一句话确认模板：

> “我先聚焦 A、B 两个核心功能，规模按 N DAU、读写比 R、P95 延迟 T、可用性 99.9%，暂时不做 C 和 D。这个范围合理吗？”

### 1.2 容量估算：用数字决定架构

```text
平均 QPS = 每日请求数 / 86,400 ≈ 每日请求数 / 100,000
峰值 QPS = 平均 QPS × 峰值系数（2–5，热点社交可用 10）
写入存储/天 = 写入条数/天 × 单条记录大小
总存储 = 写入存储/天 × 保留天数 × 副本系数
带宽 = QPS × 单次响应大小
缓存容量 ≈ 热点对象数 × 对象大小 × overhead 系数（1.2–2）
缓存命中后 DB QPS = 总读 QPS × (1 - cache hit rate)
分片数 ≈ 峰值写 QPS / 单分片安全写 QPS，再留 2× 余量
```

示例：1,000 万 DAU，每人每天 20 次读取，读请求 2 亿/天，平均约 2,000 QPS，峰值乘 5 是 1 万 QPS。写入若每人每天 1 次，就是 1,000 万/天，平均 100 QPS，峰值 500 QPS。这个读写比说明优先优化读路径，缓存和 CDN 的价值高于写入分片。

| 项目 | 粗略数字 | 用法 |
| --- | ---: | --- |
| 1 天秒数 | 86,400 ≈ 10^5 | 日请求转 QPS |
| Redis 点查 | ~0.5–2 ms | 热点读、计数 |
| DB 单行点查 | ~1–10 ms | 有索引时可接受 |
| 同机房网络 | ~0.1–1 ms | 服务间 RPC |
| 跨区域网络 | ~50–200 ms | multi-region 成本 |
| 用户可感知延迟 | 100–300 ms | P95 目标常用范围 |
| 1 KB / 1 TB | 10^3 bytes / 10^12 bytes | metadata / 历史数据 |

### 1.3 API、数据模型与高层架构

API 要覆盖创建、读取、分页、错误码和幂等：

```http
POST /v1/resources
Idempotency-Key: <client-generated-key>
{
  "field": "value"
}

201 Created
{
  "id": "res_123",
  "created_at": "2026-07-07T10:20:53Z"
}
```

数据模型先写访问模式，再决定主键、索引、分区键：

```text
Entity: Resource
- id: string / bigint, primary key
- owner_id: string, index
- status: enum
- created_at: timestamp, index for pagination

Primary access patterns:
1. get by id              -> primary key
2. list by owner + time   -> composite index (owner_id, created_at desc)
3. search by keyword      -> search index, not OLTP DB
```

SQL / NoSQL 判断：事务、关系和灵活查询优先 PostgreSQL/MySQL；超大规模 key-value、固定访问模式和水平扩展优先 DynamoDB/Cassandra。任何选择都要说出代价：SQL 的水平写扩展难，NoSQL 的 ad-hoc query 弱。

通用高层架构：

```text
Client / Mobile / Web
        │
        ▼
 CDN / Edge Cache（静态资源、可缓存读请求）
        │
        ▼
 Load Balancer
        │
        ▼
 API Gateway（auth、rate limit、routing）
        │
        ▼
 Stateless Services（水平扩容）
        │
        ├── Redis / Memcached（热点读、session、计数）
        ├── Primary DB + Read Replicas（事务、持久化）
        ├── Message Queue（异步任务、削峰、重试）
        └── Workers（发送通知、生成 feed、统计聚合）
```

讲图顺序：Client 到 LB；Gateway 做认证/限流/路由；Service 无状态；读路径 cache-aside；写路径写 DB 后发事件；读多加 replicas，写多考虑分片。

### 1.4 深挖风险与收尾

优先深挖“被数字证明有风险”的点：读 QPS 高看缓存/CDN/热点；写 QPS 高看分片/MQ/幂等；数据量大看分区键/归档；一致性敏感看事务/Outbox/Saga；全球化看 multi-region/数据主权/冲突解决。

深挖模板：

> “这个系统最大的风险是 X，因为估算里 Y 已经到 Z 量级。我会用方案 A 解决，代价是 B。备选方案 C 在某些条件下更好，但这里不选它，因为 D。”

收尾模板：

> “这个设计优先保证 A，因此选择 B；牺牲 C，通过 D 缓解。当前瓶颈会先出现在 E；如果流量增长 10 倍，我会先做 F，再做 G。生产上监控 P95/P99 latency、error rate、cache hit rate、queue lag、DB CPU/IO 和 replication lag。”

---

## 2. 组件工具箱：何时用 + 权衡

| 组件 / 模式 | 何时用 | 关键做法 | 主要权衡 / 风险 |
| --- | --- | --- | --- |
| Load Balancer | 多实例、高可用入口 | L4/L7 LB，health check | 配置错误放大故障；sticky session 削弱扩容 |
| API Gateway | 统一入口、auth、rate limit、routing | JWT、路由、限流、request id | 可能成为延迟和故障中心 |
| Cache-aside | 读多写少、热点数据 | 读 cache miss 查 DB 回填；写 DB 后删 cache | 穿透、击穿、雪崩；短暂脏读 |
| SQL DB | 事务、关系、join、灵活查询 | 索引、读副本、分区 | 水平写扩展难；迁移要谨慎 |
| NoSQL KV / Wide-column | 固定访问模式、超大 key-value | 按 partition key 查询 | ad-hoc query 弱；分区键选错会热点 |
| Sharding | 单库容量或写 QPS 达瓶颈 | shard key、routing layer、一致性哈希 | re-sharding 贵；跨 shard 查询复杂 |
| Replication | 读扩展、高可用、灾备 | primary-replica，同步/异步复制 | replica lag；failover 可能丢数据 |
| Message Queue | 异步、削峰、解耦、重试 | Kafka/SQS/RabbitMQ，consumer group，DLQ | 至少一次投递；consumer 必须幂等 |
| CDN | 静态资源、全球读、热点内容 | Cache-Control、URL versioning、edge purge | 个性化内容难缓存；失效传播延迟 |
| CAP / 最终一致 | 分区容忍下做 C/A 取舍 | 钱/库存强一致；feed/计数最终一致 | 需要补偿和可观测性 |
| Rate Limiting | 防滥用、保护下游、租户配额 | token bucket、sliding window | 误伤用户；分布式限流需近似或中心化 |
| Idempotency | 重试、创建资源、MQ 消费 | Idempotency-Key、唯一约束、去重表 | key 存储有成本；语义要清楚 |
| Circuit Breaker | 下游不稳定 | timeout、熔断、半开探测、fallback | fallback 可能返回旧数据 |
| Backpressure | 消费慢于生产 | 队列阈值、拒绝、降级、限流 | 用户体验下降；要暴露 queue lag |
| Observability | 定位延迟、错误、容量问题 | logs、metrics、traces、SLO dashboard | 指标必须围绕用户路径定义 |

高信号表达：

> “读峰值 25k QPS，而 DB 希望控制在 5k 以下，所以我用 Redis 做 cache-aside。假设命中率 80%，DB 读 QPS 会从 25k 降到 5k。代价是写后短暂不一致，我会在写成功后删除 cache，并用 TTL 兜底。”

| 场景 | 推荐组合 | 说明 |
| --- | --- | --- |
| 短链接跳转 | CDN/Edge + Redis + KV DB | 读多写少，key-value 访问 |
| 新闻 Feed | Fanout worker + Redis feed cache + NoSQL | 读写扩散取舍明显 |
| 支付 | SQL + 幂等键 + Outbox + Saga | 强一致边界清楚，异步通知 |
| 搜索 | OLTP DB + CDC/MQ + Elasticsearch | 写入主库，异步建索引 |

---

## 3. Worked Design #1：短链接系统

目标：用户提交长 URL，系统返回短 URL；访问短 URL 时 302 redirect 到原始 URL。

### 3.1 需求与估算

功能：创建短链接、访问跳转、自定义 alias、过期时间、异步点击统计。非功能：跳转 P95 < 100ms；创建 P95 < 300ms；跳转路径 99.99%；创建后立即可读；统计允许分钟级延迟。

假设：1,000 万 DAU；每人每天创建 0.1 个短链 → 100 万新短链/天；每人每天点击 10 次 → 1 亿跳转/天；峰值系数 5；单条 URL 记录按 1KB；数据保留 5 年，副本系数 3。

```text
创建/天 = 10M × 0.1 = 1M/day
平均写 QPS = 1M / 100,000 = 10 QPS；峰值写 QPS = 50 QPS
跳转/天 = 10M × 10 = 100M/day
平均读 QPS = 100M / 100,000 = 1,000 QPS；峰值读 QPS = 5,000 QPS
URL 存储/天 = 1M × 1KB = 1GB/day
5 年原始存储 = 1GB × 365 × 5 = 1.8TB；含 3 副本 = 5.4TB
跳转响应按 1KB：峰值带宽 = 5,000 × 1KB = 5MB/s ≈ 40Mbps
```

结论：写 QPS 很低，创建路径不需要复杂分片；读峰值 5k QPS，Redis cache + read replicas 足够；访问模式是 `short_code -> long_url`，KV store 很自然。

### 3.2 API、模型与短码

```http
POST /v1/links
Idempotency-Key: 7f3a-client-generated
{
  "long_url": "https://example.com/articles/system-design",
  "custom_alias": "sys-design",
  "expires_at": "2027-07-07T00:00:00Z"
}

201 Created
{
  "short_url": "https://sho.rt/abc123",
  "short_code": "abc123"
}
```

```http
GET /{short_code}

302 Found
Location: https://example.com/articles/system-design
Cache-Control: private, max-age=60
```

```text
links
- short_code: string primary key
- long_url: string
- user_id: string, index (user_id, created_at desc)
- status: active | expired | blocked
- created_at: timestamp
- expires_at: timestamp nullable

idempotency_keys
- user_id: string
- idempotency_key: string
- short_code: string
- request_hash: string
unique(user_id, idempotency_key)

click_events（append-only，可进 Kafka/S3/ClickHouse）
- event_id, short_code, ts, ip_hash, user_agent, referrer, country
```

| 短码方案 | 优点 | 缺点 |
| --- | --- | --- |
| 自增 ID + Base62 | 短、无碰撞、可控 | 可预测；需要 ID service |
| 随机 Base62 | 简单、不可预测 | 需要唯一检查；高占用率时碰撞增加 |
| Hash long URL | 同 URL 可复用 | 碰撞处理复杂；泄漏模式 |

`62^6 ≈ 56.8B`，`62^7 ≈ 3.5T`。每天 100 万新短链，6 位理论可用很久；考虑冲突、保留、恶意扫描，实际用 7 位更安全。

### 3.3 架构与深挖

```text
Create path:
Client → API Gateway（auth/rate limit）→ Link Service
  → Idempotency Store → ID Generator → Link DB / KV Store → Redis（可选预热）

Redirect path:
Browser → Edge/CDN → Load Balancer → Redirect Service
  → Redis cache → Link DB/KV（cache miss）→ Kafka click_events（异步）
  ← 302 Location: long_url

Analytics path:
Kafka click_events → Stream Processor → ClickHouse/Druid/BigQuery → Stats API
```

读路径伪流程：

```text
GET /abc123
1. Validate short_code；不合法直接 404，避免打 DB。
2. Redis GET link:abc123。
3. Hit：检查 status/expires_at，返回 302。
4. Miss：DB/KV get short_code。
5. DB miss：写 negative cache，TTL 30–60s，返回 404。
6. DB hit：Redis SET link:abc123 TTL 1–24h（加随机抖动），返回 302。
7. 异步发送 click event；失败不阻塞跳转。
```

关键权衡：

- 点击统计不放同步路径，因为跳转 P95 < 100ms 比统计实时性更重要。
- 创建后需要立即可读，可在写 DB 成功后直接 `SET cache` 预热。
- 更新或封禁：先更新 DB，再删除 cache；删除失败用 Outbox 事件重试。
- cache value 包含 `expires_at`，即使 cache 未过期，服务也能判断链接已过期。

10× 演进：Redirect Service 扩容；Redis cluster 分片；热门短链放 CDN/Edge cache；Link Store 迁移到按 `hash(short_code)` 分片的 KV store；click events 拆 Kafka 分区，聚合层改 ClickHouse / Druid。

监控：redirect P95/P99、302/404/5xx rate、cache hit rate、negative cache hit rate、DB QPS、replication lag、hot key、Kafka produce error、analytics freshness。

---

## 4. Worked Design #2：新闻 Feed 系统

目标：用户关注其他人，打开首页看到按时间或相关性排序的动态。

### 4.1 需求与估算

功能：发布 post、关注 / 取关、读取 home feed、分页加载、点赞 / 评论计数展示。非功能：Feed 读取 P95 < 300ms；发布 P95 < 500ms；新 post 进入粉丝 feed 允许延迟几秒；读路径 99.9%+。

假设：5,000 万 DAU；每人每天打开 feed 10 次；每次返回 20 条，每条 feed item metadata 1KB；10% 用户每天发 2 条 post → 1,000 万 posts/day；平均粉丝数 200；大 V 可达 1,000 万粉丝；峰值系数 5。

```text
feed read/day = 50M × 10 = 500M/day
平均读 QPS = 5,000 QPS；峰值读 QPS = 25,000 QPS
post write/day = 50M × 10% × 2 = 10M/day
平均发帖 QPS = 100 QPS；峰值发帖 QPS = 500 QPS
如果全部写扩散：fanout writes/day = 10M × 200 = 2B feed items/day
平均 fanout 写 QPS = 20,000 QPS；峰值 fanout 写 QPS = 100,000 QPS
feed 响应大小 = 20 × 1KB = 20KB；峰值读带宽 ≈ 500MB/s ≈ 4Gbps
feed item 存储/天 = 2B × 100B = 200GB/day；保留 30 天，3 副本 = 18TB
```

结论：读 QPS 高，不能每次在线 join 所有关注对象；全量写扩散对普通用户可行，但大 V 会造成 fanout 爆炸；feed item 只存轻量引用，不复制完整 post 内容。

### 4.2 API、数据模型与架构

```http
POST /v1/posts
Idempotency-Key: post-uuid-123
{
  "text": "system design notes",
  "media_ids": ["m_1", "m_2"],
  "visibility": "public"
}

201 Created
{"post_id": "p_987", "created_at": "2026-07-07T10:20:53Z"}
```

```http
GET /v1/feed/home?limit=20&cursor=eyJ0cyI6...

200 OK
{
  "items": [{"post_id": "p_987", "author_id": "u_1", "text": "...", "like_count": 123}],
  "next_cursor": "eyJ0cyI6..."
}
```

```text
posts: post_id PK, author_id index(author_id, created_at desc), text, media_ids, visibility, created_at, deleted_at
follows: primary key(follower_id, followee_id), index(followee_id, follower_id) -- fanout 查粉丝
home_feed_items: primary key(user_id, sort_ts desc, post_id), author_id, TTL 30–90 days
engagement_counts: post_id PK, like_count, comment_count, updated_at
```

```text
Post write path:
Client → API Gateway → Post Service → Post DB → Outbox/Kafka: PostCreated(post_id, author_id, ts)

Fanout path:
Kafka PostCreated → Fanout Worker → Follow Store（查粉丝）
  → Home Feed Store（批量写 follower feed）→ Redis Feed Cache（可选预热）

Read feed path:
Client → Feed Service → Redis home_feed:{user_id} → Home Feed Store（miss/pagination）
  → Post Service/Post Cache（批量取内容）→ Engagement Cache（批量取计数）
  ← feed items + next_cursor
```

### 4.3 推 / 拉 / 混合 Feed

| 模式 | 做法 | 优点 | 缺点 | 适用 |
| --- | --- | --- | --- | --- |
| Push / Fanout-on-write | 发帖时写入每个粉丝的 home feed | 读很快 | 大 V 发帖写爆；关注变化要修补 | 普通用户、读多写少 |
| Pull / Fanout-on-read | 读 feed 时拉取关注者最新 posts 合并 | 写很轻 | 读时 join 多作者，延迟高 | 关注数少、写多读少 |
| Hybrid | 普通用户 push，大 V pull；读时 merge | 平衡读延迟和写放大 | 排序和去重更复杂 | 大多数社交 feed |

推荐 Hybrid：

```text
if author.follower_count < 100,000:
    fanout post_id to followers' home_feed_items
else:
    mark author as celebrity
    read path pulls celebrity recent posts and merges with precomputed feed
```

读路径 merge：从 `home_feed_items[user_id]` 取普通作者候选；批量取用户关注的大 V 最近 posts；合并、排序、去重、过滤删除/不可见内容；返回前 20 条和 cursor。这样避免 1,000 万粉丝大 V 发一条造成 1,000 万次写入，代价是读路径需要 merge 和缓存。

### 4.4 Fanout 可靠性、缓存与 ranking

Fanout worker：消费 `PostCreated(post_id, author_id, ts, event_id)`；分页读取粉丝；批量写 `home_feed_items`；记录 `event_id + follower_page_cursor` checkpoint；失败重试，超过阈值进入 DLQ。

幂等设计：`home_feed_items` 主键包含 `(user_id, post_id)` 或写入条件 `insert if not exists`；`PostCreated.event_id` 用于 worker checkpoint；`FollowChanged` 修补 feed 时也按 `post_id` 去重。

延迟控制：普通用户 fanout P95 目标 5 秒；Worker 根据 queue lag 自动扩容；超大任务每批 1,000 粉丝；lag 超阈值时，新 post 暂时通过 pull merge 出现在活跃用户 feed。

缓存与分页：`Redis sorted set home_feed:{user_id}` 存最近 500–1000 个 post_id，score = timestamp 或 ranking score；活跃用户缓存，长尾走 Home Feed Store；计数缓存批量 MGET；分页用 cursor：`last_score`, `last_post_id`, `ranking_version`，不用 offset。

```text
score = w1 * recency_score
      + w2 * affinity(user, author)
      + w3 * engagement_score
      - w4 * negative_feedback
```

Ranking 可作为独立服务，在候选集生成后打分，不改变底层 fanout 模型；基础架构和 ML 迭代解耦。

10× 演进：Follow Store 和 Home Feed Store 按 `user_id` hash 分片；Fanout Worker 按 author_id/event_id 分区；Feed Cache 只覆盖活跃用户；Engagement count 使用近实时流处理；Ranking 独立服务化；多区域读本地 feed cache、写 home region、异步复制。

监控：feed read P95/P99、empty feed rate、duplicate item rate、fanout queue lag、fanout success rate、DLQ count、cache hit rate、celebrity merge latency、数据新鲜度。

---

## 5. 把能力转成可见成果：设计文档与架构评审

系统设计能力只有被写下来、被评审、被复盘，才会变成职业杠杆。每个真实项目沉淀一份 1–3 页设计文档。

```markdown
# Design: <系统 / 功能名>
## Context：业务目标、用户路径、当前痛点
## Goals / Non-goals：本期做什么、不做什么
## Assumptions & Estimates：DAU、QPS、storage、bandwidth、latency、availability、consistency
## API & Data Model：核心接口、实体、索引、分区键
## Architecture：文字箭头图或 Mermaid 图
## Key Trade-offs：Decision / Why / Cost / When to revisit
## Failure Modes：Failure / Symptom / Mitigation / Metric
## Rollout Plan：Phase 1 / Phase 2 / Rollback
```

架构评审要展示的判断力：

| 维度 | 强信号 |
| --- | --- |
| 需求澄清 | 主动确认功能、非功能、Out of Scope |
| 数字估算 | 给 QPS、存储、带宽，并用数字解释组件选择 |
| API / 模型 | 给接口、字段、索引、分区键、错误码 |
| 深挖能力 | 讲故障、重试、幂等、热点、降级 |
| 取舍与运维 | 明确牺牲什么、何时换方案，并给 SLO、metrics、runbook 方向 |

把这些判断力写成项目故事，可以用于绩效、作品集、咨询销售页，也可以在职业面试中用 [STAR 方法](04-behavioral-interview-star.md) 讲清楚：Situation 是约束，Task 是目标，Action 是架构取舍，Result 是指标改善。

### 5.1 常见坑 & 修正动作

| 坑 | 后果 | 修正动作 |
| --- | --- | --- |
| 跳过澄清 | 设计错方向 | 先确认核心 use case 和规模 |
| 不做估算 | 决策无依据 | 用 5 分钟算读写 QPS、存储、带宽 |
| 只堆组件 | 复杂度失控 | 每加一个组件，说清瓶颈和代价 |
| API / 数据模型缺失 | 架构无法落地 | 给接口、字段、主键、索引、分区键 |
| 忽略写路径 | 数据不一致或写爆 | 说明事务边界、异步事件、重试 |
| 忽略失败 | 生产风险不可控 | 讲 cache miss、MQ 重复、DB failover、限流 |
| 一致性说不清 | 不可信 | 明确强一致范围，其余最终一致 + 补偿 |
| 分片键随便选 | 热点或跨分片查询 | 根据主访问模式选 key；说明 re-sharding |
| 分页用 offset | 深页慢、重复漏读 | 用 cursor：`last_score + last_id` |

---

## 6. 掌握检查清单与练习

### 6.1 掌握检查清单

- [ ] 我能把模糊需求压缩成 2–4 个核心 use cases，并在 5 分钟内估出读 QPS、写 QPS、存储、带宽中的至少 3 个数字。
- [ ] 我能定义 3–5 个核心 API，覆盖错误码、分页、幂等，并给出实体、主键、索引、分区键和数据生命周期。
- [ ] 我能讲通一次读请求和一次写请求的数据流，并为每个组件说出“解决什么瓶颈 + 引入什么代价”。
- [ ] 我能主动讲失败场景、补救方案、当前取舍、10× 流量演进和关键监控指标。
- [ ] 我能把方案整理成 1–3 页设计文档，接受他人评审。

### 6.2 三个里程碑作业

1. **Rate Limiter**：per user / per IP / per API key；10 万 QPS；支持 token bucket 和 sliding window；多区域允许轻微误差。深挖 Redis 计数 vs 本地计数 vs 分布式近似、误杀缓解、Redis 故障 fail open 还是 fail closed。
2. **聊天系统**：1:1 和群聊；在线消息 P95 < 200ms；离线消息可靠送达；已读回执最终一致。深挖 WebSocket gateway 状态、Message Store 分区键、`client_msg_id` 幂等。
3. **通知系统**：Email、Push、SMS；每天 10 亿通知；用户偏好、退订、频控；第三方 provider 不稳定。深挖 MQ、重试、DLQ、provider failover、幂等发送和优先级队列。

每个作业产出：需求澄清清单、估算草稿、API、数据模型、文字架构图、深挖笔记、权衡表、监控指标。目标不是“答题”，而是积累可复用的架构作品集。

---

## 7. 延伸阅读

- [System Design Primer](https://github.com/donnemartin/system-design-primer)：经典开源资料，适合补齐组件基础和常见题。
- [ByteByteGo Blog](https://blog.bytebytego.com/)：大量可视化系统设计案例，适合学习表达方式。
- [High Scalability](http://highscalability.com/)：真实大规模系统案例和架构演进。

---

## 小结

系统设计能力的稳定打法是：**先定边界，再用数字推架构，最后深挖取舍**。它不是应试技巧，而是资深工程师脱离坐班依赖后仍能创造信任的能力：你能把复杂系统讲清楚、写清楚、上线后看住风险，并在约束变化时持续演进。

`标签` `系统设计` `架构思维` `可扩展性` `权衡` `容量估算` `分布式系统`

---

[← 上一章](02-remote-overseas-job-strategy.md) · [WP-02 目录](README.md) · [下一章 →](04-behavioral-interview-star.md)
