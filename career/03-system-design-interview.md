[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 系统设计面试框架（面向资深工程师）

> 结论先行：系统设计面试不是背组件名，而是在 45 分钟内把一个模糊业务题变成**有边界、有数字、有取舍、有演进路径**的方案。你要像 Tech Lead 一样主持讨论：先对齐目标，再用估算驱动架构，最后深挖最关键的 1–2 个风险点。

相关阅读：[技术面试英语](../english/01-technical-interview-english.md)、[Remote / 海外求职策略](02-remote-overseas-job-strategy.md)、[转型 AI 的职业路径](01-career-pivot-to-ai.md)。

## 本章定位：把“会画架构图”升级成“能主导设计评审”

适用对象：有后端、平台、数据、移动端或全栈经验，正在准备 Senior / Staff / Tech Lead 级别系统设计面试的工程师。

前置知识：HTTP、REST/gRPC、SQL 基础、缓存、消息队列、分布式系统基本概念。如果某个组件不熟，先用本章的“组件工具箱”建立可面试的最小模型。

学完本章，你应该能独立完成：

- 在 45 分钟内稳定走完“澄清 → 估算 → API → 数据模型 → 架构 → 深挖 → 权衡 → 总结”。
- 用公式估出 QPS、存储、带宽、缓存容量，并解释这些数字如何影响架构。
- 对负载均衡、缓存、DB、分片、复制、MQ、CDN、一致性、限流、幂等做“何时用 + 代价是什么”的判断。
- 从头到尾讲清两个经典题：短链接系统、新闻 Feed 系统。
- 展示 senior signal：主动定义边界、识别瓶颈、讲失败路径、给演进计划、把业务约束转成技术取舍。

---

## 1. 45 分钟固定时间盒脚本

### 1.1 总体节奏

| 阶段 | 时间 | 目标产出 | 面试官在看什么 |
| --- | ---: | --- | --- |
| 0. 开场对齐 | 0–1' | 复述题目，声明流程 | 是否能掌控讨论 |
| 1. 澄清需求 | 1–6' | 功能 / 非功能 / Out of Scope | 是否先问问题而不是先画图 |
| 2. 容量估算 | 6–12' | QPS、存储、带宽、峰值 | 是否能用数字驱动决策 |
| 3. API 设计 | 12–17' | 3–5 个核心接口 | 是否理解产品边界和错误场景 |
| 4. 数据模型 | 17–22' | 核心实体、索引、分区键 | 是否能落到数据层 |
| 5. 高层架构 | 22–32' | 组件图 + 请求流 | 是否能先做简单可工作的系统 |
| 6. 深挖 1–2 组件 | 32–42' | 缓存 / 分片 / MQ / 一致性等 | 是否能处理规模、故障、取舍 |
| 7. 权衡与总结 | 42–45' | Trade-off、演进、监控 | 是否有生产经验和收尾能力 |

> 使用方法：你不需要机械报时，但要在脑中设闹钟。若 15 分钟还在澄清需求，后面必然失控；若 30 分钟还没画高层架构，深挖时间会被挤掉。

### 1.2 逐分钟 talking-track（可直接背）

#### 0–1'：开场对齐

你说：

> “我会先澄清功能和非功能需求，然后做一个 back-of-envelope 估算，用数字决定数据存储和缓存策略。接着定义 API、数据模型和高层架构，最后我会挑 1–2 个最有风险的点深入，比如缓存一致性、分片或消息队列。可以吗？”

完成标准：面试官点头，或主动给出范围提示。

#### 1–6'：澄清需求（功能 / 非功能 / Out of Scope）

按三个层次问，不要一次抛 20 个问题。

**功能需求**：

- “核心用户是谁？生产者和消费者是否不同？”
- “必须支持哪 2–3 个核心 use cases？”
- “读写路径分别是什么？读多还是写多？”
- “是否需要搜索、推荐、权限、审计、统计？”

**非功能需求**：

- “规模假设：DAU / MAU / 请求量大概多少？”如果面试官不提供，你主动假设。
- “延迟目标：P95 需要在 100ms、300ms 还是 1s 内？”
- “可用性目标：99.9%、99.99% 还是更高？”
- “一致性要求：强一致、读己之写、还是秒级最终一致可接受？”
- “数据保留多久？是否有隐私、合规、地域要求？”

**Out of Scope**：

- “为了 45 分钟内讲深，我会先不做 X，只保留 Y。比如短链接先不做复杂风控和广告归因，只做创建、跳转、统计。”

一句话确认模板：

> “我先聚焦 A、B 两个核心功能，规模按 N DAU、读写比 R、P95 延迟 T、可用性 99.9%，暂时不做 C 和 D。这个范围合理吗？”

完成标准：你得到一个可设计的题目，而不是泛泛的“设计 Twitter”。

#### 6–12'：容量估算（带公式）

你说：

> “我先用粗略数字估算，不追求精确，目的是决定是否需要缓存、分片、异步化和 CDN。”

固定公式：

```text
平均 QPS = 每日请求数 / 86,400 ≈ 每日请求数 / 100,000
峰值 QPS = 平均 QPS × 峰值系数（2–5，社交热点可用 10）
写入存储/天 = 写入条数/天 × 单条记录大小
总存储 = 写入存储/天 × 保留天数 × 副本系数
读带宽 = 读 QPS × 单次响应大小
写带宽 = 写 QPS × 单次写入大小
缓存容量 ≈ 热点对象数 × 对象大小 × overhead 系数（1.2–2）
```

估算 talking-track：

> “假设 1,000 万 DAU，每人每天 20 次读取，所以读请求是 2 亿/天，平均约 2,000 QPS，峰值乘 5 是 1 万 QPS。写入如果每人每天 1 次，就是 1,000 万/天，平均 100 QPS，峰值 500 QPS。这个读写比说明我们优先优化读路径，缓存和 CDN 的价值很高。”

完成标准：至少给出读 QPS、写 QPS、存储增长、带宽中的 3 个数字，并用它们解释一个架构选择。

#### 12–17'：API 设计

你说：

> “我先定义外部 API，确保系统边界清楚。接口只列核心字段、错误码和幂等方式，不展开所有业务细节。”

API 模板：

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

要覆盖：

- 创建：是否需要 `Idempotency-Key`。
- 读取：按 ID、分页、排序、游标。
- 更新 / 删除：权限和并发控制。
- 错误：`400` 参数错、`401/403` 权限、`404` 不存在、`409` 冲突、`429` 限流、`5xx` 服务异常。

完成标准：3–5 个接口足够；每个接口都服务于前面确认的核心 use case。

#### 17–22'：数据模型

你说：

> “我会先给逻辑模型，再说明物理层如何索引、分片和复制。”

数据模型模板：

```text
Entity: Resource
- id: string / bigint, primary key
- owner_id: string, index
- status: enum
- created_at: timestamp, index for pagination
- updated_at: timestamp

Primary access patterns:
1. get by id              -> primary key
2. list by owner + time   -> composite index (owner_id, created_at desc)
3. search by keyword      -> search index, not OLTP DB
```

选择 SQL / NoSQL 的话术：

> “如果核心是事务、关系和灵活查询，我会从 PostgreSQL/MySQL 起步；如果核心是超大规模 key-value、固定访问模式和水平扩展，我会选 DynamoDB/Cassandra 这类 NoSQL。这里我选 X，因为访问模式是 Y，代价是 Z。”

完成标准：你明确了主键、关键索引、分区键，以及为什么这样支持 API。

#### 22–32'：高层架构

你说：

> “先画能工作的最小系统，再按估算加组件。每个箭头我都会说明数据怎么流动。”

通用骨架：

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

讲图顺序：

1. 用户请求从 Client 到 LB。
2. API Gateway 做认证、限流、路由。
3. Service 无状态，方便水平扩容。
4. 读路径先查 Cache，miss 后查 DB，再回填。
5. 写路径先写 DB，再发事件到 MQ，异步更新搜索、统计、通知。
6. DB 用主从复制；读多时加 read replicas；写多时考虑分片。

完成标准：面试官能听懂系统如何处理一次读请求和一次写请求。

#### 32–42'：深挖 1–2 个组件

优先深挖“被数字证明有风险”的点，而不是你最熟的点。

选择规则：

- 读 QPS 高：深挖缓存、CDN、热点 key、降级。
- 写 QPS 高：深挖分片、写入扩展、MQ 削峰、幂等。
- 数据量大：深挖分区键、归档、索引膨胀、冷热分层。
- 一致性敏感：深挖事务、Outbox、Saga、读己之写。
- 全球化：深挖 multi-region、延迟、数据主权、冲突解决。

深挖 talking-track：

> “这个系统最大的风险是 X，因为估算里 Y 已经到 Z 量级。我会用方案 A 解决，代价是 B。备选方案 C 在某些条件下更好，但这里我不选它，因为 D。”

完成标准：至少讲一个失败场景和对应补救，例如缓存穿透、消息重复消费、分片热点、主库故障。

#### 42–45'：权衡与总结

你说：

> “总结一下：这个设计优先保证 A，因此选择了 B；牺牲的是 C，通过 D 缓解。当前规模下瓶颈会先出现在 E；如果流量增长 10 倍，我会先做 F，再做 G。生产上我会监控 P95/P99 latency、error rate、cache hit rate、queue lag、DB CPU/IO 和 replication lag。”

完成标准：你把方案、取舍、演进、监控压缩成 60–90 秒，给面试官一个清晰收尾。

---

## 2. Back-of-envelope 估算速查

### 2.1 常用数字

| 项目 | 粗略数字 | 面试用法 |
| --- | ---: | --- |
| 1 天秒数 | 86,400 ≈ 10^5 | 日请求转 QPS |
| 1 KB | 10^3 bytes | 文本、URL、metadata |
| 1 MB | 10^6 bytes | 图片缩略图、小响应批量 |
| 1 GB | 10^9 bytes | 单机内存 / 小表 |
| 1 TB | 10^12 bytes | 日志、对象存储、历史数据 |
| 内存访问 | ~100 ns | Cache 极快 |
| SSD 随机读 | ~100 μs | 比内存慢约 1000× |
| 同机房网络 | ~0.1–1 ms | 服务间 RPC |
| 跨区域网络 | ~50–200 ms | multi-region 成本 |
| DB 单行点查 | ~1–10 ms | 有索引时可接受 |
| Redis 点查 | ~0.5–2 ms | 热点读、计数 |
| 用户可感知延迟 | 100–300 ms | P95 目标常用范围 |

> 面试中数字只需量级正确。说清“我用 10^5 秒近似一天，方便心算”比假装精确更专业。

### 2.2 常用公式

```text
DAU = Daily Active Users
MAU = Monthly Active Users
读请求/天 = DAU × 人均读次数/天
写请求/天 = DAU × 人均写次数/天
平均 QPS = 请求/天 / 86,400 ≈ 请求/天 / 10^5
峰值 QPS = 平均 QPS × 2–5（热点产品用 10）
读写比 = 读 QPS : 写 QPS
单日存储 = 新记录数/天 × 单条大小
总存储 = 单日存储 × 保留天数 × 副本数
带宽 = QPS × 响应大小
缓存命中后 DB QPS = 总读 QPS × (1 - cache hit rate)
分片数 ≈ 峰值写 QPS / 单分片安全写 QPS，再留 2× 余量
```

### 2.3 Worked 估算示例：照片 metadata 服务

题目：用户上传照片，首页读取最近 20 张照片 metadata；图片本体在 object storage，只估 metadata。

假设：

- 5,000 万 DAU。
- 每人每天上传 2 张照片。
- 每人每天刷新首页 10 次。
- 单条 photo metadata：`photo_id` 16B、`user_id` 16B、URL 200B、caption 200B、时间和状态 50B，加索引 overhead 后按 1 KB。
- 数据保留 5 年，副本系数 3。

计算：

```text
写入照片/天 = 50M × 2 = 100M/day
平均写 QPS = 100M / 100,000 = 1,000 QPS
峰值写 QPS = 1,000 × 5 = 5,000 QPS

首页读请求/天 = 50M × 10 = 500M/day
平均读 QPS = 500M / 100,000 = 5,000 QPS
峰值读 QPS = 5,000 × 5 = 25,000 QPS

metadata 存储/天 = 100M × 1KB = 100GB/day
5 年原始存储 = 100GB × 365 × 5 = 182.5TB
含 3 副本 = 547.5TB ≈ 0.55PB

首页响应大小 = 20 × 1KB = 20KB
峰值读带宽 = 25,000 × 20KB = 500,000KB/s ≈ 500MB/s ≈ 4Gbps
```

由数字推出决策：

- 25k 峰值读 QPS：Service 无状态横向扩容，首页列表加缓存。
- 5k 峰值写 QPS：单主关系型 DB 可能吃紧，按 `user_id` 或时间分区，或使用可水平扩展 NoSQL。
- 0.55PB metadata：需要冷热分层，最近 90 天放在线库，历史归档到 cheaper storage / analytical store。
- 4Gbps 读带宽：metadata 可承受；图片本体必须走 CDN/Object Storage，不能经应用服务转发。

---

## 3. 组件工具箱：何时用 + 权衡

| 组件 / 模式 | 何时用 | 关键做法 | 主要权衡 / 风险 |
| --- | --- | --- | --- |
| Load Balancer | 多个服务实例、需要高可用入口 | L4/L7 LB，health check，least connections / round-robin | LB 配置错误会放大故障；sticky session 会削弱扩容能力 |
| API Gateway | 多服务统一入口，需要 auth、rate limit、routing | JWT 校验、路由、限流、request id | 可能成为延迟和故障中心；业务逻辑不要塞进 gateway |
| Cache-aside | 读多写少、热点数据、可容忍短暂不一致 | 读：先 cache，miss 查 DB 回填；写：更新 DB 后删 cache | 缓存穿透、击穿、雪崩；删除失败会产生脏读 |
| Write-through Cache | 写入少但希望读总是命中缓存 | 写 cache 同步写 DB | 写延迟增加；cache 故障影响写路径 |
| Write-back Cache | 极高写吞吐、可容忍丢失风险 | 先写 cache，异步落 DB | 数据丢失和一致性风险高，面试中谨慎使用 |
| Cache TTL | 数据可过期、允许分钟级不一致 | 设置随机 TTL，避免同一时间过期 | TTL 太长脏数据，太短命中率低 |
| 缓存失效 | 写后需要读到新值 | update DB → delete cache；必要时双删 / versioned key | 删除 cache 与 DB commit 的竞态；可用 Outbox 修复 |
| SQL DB | 事务、关系、join、灵活查询 | PostgreSQL/MySQL，索引、读副本、分区 | 水平写扩展较难；schema migration 要谨慎 |
| NoSQL KV / Wide-column | 固定访问模式、超大规模 key-value、水平扩展 | DynamoDB/Cassandra，按 partition key 查询 | ad-hoc query 弱；分区键选错会热点 |
| Document DB | JSON 文档、schema 变化快、按文档聚合 | MongoDB 等，按业务聚合建模 | 跨文档事务弱；过大文档影响更新 |
| Sharding | 单库容量或写 QPS 达瓶颈 | 选 shard key，routing layer，一致性哈希或范围分片 | re-sharding 成本高；热点 shard；跨 shard 查询复杂 |
| Replication | 读扩展、高可用、灾备 | primary-replica，同步 / 异步复制 | replica lag；故障切换可能丢数据或产生 split-brain |
| Message Queue | 异步、削峰、解耦、重试 | Kafka/SQS/RabbitMQ，consumer group，DLQ | 至少一次投递导致重复；需要幂等 consumer；queue lag 可见性 |
| CDN | 静态资源、全球读、热点内容 | Cache-Control、URL versioning、edge purge | 动态个性化内容难缓存；失效传播延迟 |
| CAP / 最终一致 | 分区容忍下在可用性和一致性之间取舍 | 强一致用于钱/库存；最终一致用于 feed/计数 | 用户可能读到旧数据；需要补偿和可观测性 |
| Rate Limiting | 防滥用、保护下游、区分租户配额 | token bucket、sliding window、per user/IP/API key | 误伤正常用户；分布式限流需要近似或中心化计数 |
| Idempotency | 重试、支付、创建资源、MQ 消费 | Idempotency-Key、唯一约束、去重表、状态机 | key 存储有成本；key 语义必须定义清楚 |
| Circuit Breaker | 下游不稳定时保护系统 | timeout、熔断、半开探测、fallback | 过早熔断影响可用功能；fallback 可能返回旧数据 |
| Backpressure | 消费速度低于生产速度 | 队列长度阈值、拒绝、降级、限流 | 用户体验下降；必须暴露 queue lag 和 dropped count |
| Observability | 定位延迟、错误、容量问题 | logs、metrics、traces、SLO dashboard | 指标太多无用；必须围绕用户路径定义 |

### 3.1 组件选择的面试话术

不要说：“这里加 Redis、Kafka、Elasticsearch。”

要说：

> “读峰值 25k QPS，而 DB 希望控制在 5k 以下，所以我用 Redis 做 cache-aside。假设命中率 80%，DB 读 QPS 会从 25k 降到 5k。代价是写后短暂不一致，我会在写成功后删除 cache，并用 TTL 兜底。”

### 3.2 常见组件组合

| 场景 | 推荐组合 | 说明 |
| --- | --- | --- |
| 短链接跳转 | CDN/Edge + Redis + KV DB | 读多写少，key-value 访问 |
| 新闻 Feed | Fanout worker + Redis feed cache + Graph DB/NoSQL | 读写扩散取舍明显 |
| 聊天 | WebSocket gateway + MQ + message store | 在线实时 + 离线可靠 |
| 支付 | SQL + 幂等键 + Outbox + Saga | 强一致边界清楚，异步通知 |
| 搜索 | OLTP DB + CDC/MQ + Elasticsearch | 写入主库，异步建索引 |
| 计数器 | Redis + periodic flush + correction job | 高吞吐近实时，最终校正 |

---

## 4. Worked Design #1：设计一个短链接系统

题目：设计 TinyURL / bit.ly 这类短链接系统。用户提交长 URL，系统返回短 URL；访问短 URL 时跳转到原始 URL。

### 4.1 需求澄清

功能需求：

1. 创建短链接：输入 long URL，返回 short code。
2. 访问短链接：根据 short code 302 redirect 到 long URL。
3. 自定义 alias：可选；若冲突返回错误。
4. 过期时间：可选；过期后返回 404 或 410。
5. 点击统计：异步记录点击事件，展示基础统计。

非功能需求：

- 读多写少：跳转远多于创建。
- 跳转 P95 < 100ms；创建 P95 < 300ms。
- 可用性：跳转路径 99.99%，创建路径 99.9%。
- 一致性：创建后应立即可读；统计允许分钟级延迟。
- 安全：恶意 URL 检测可作为异步增强，面试中先简化。

Out of Scope：

- 复杂反作弊、广告归因、企业权限、多租户账单。
- 短链接域名管理先不展开。

确认话术：

> “我会重点设计 create 和 redirect 两条路径，统计异步化；读写比按 100:1，跳转低延迟优先。自定义 alias 支持但不是主路径。可以吗？”

### 4.2 容量估算

假设：

- 1,000 万 DAU。
- 每人每天创建 0.1 个短链 → 100 万新短链/天。
- 每人每天点击 10 次 → 1 亿跳转/天。
- 峰值系数 5。
- 单条 URL 记录：short_code 16B、long_url 平均 500B、user_id 16B、时间和状态 100B、索引 overhead 后按 1KB。
- 数据保留 5 年，副本系数 3。

计算：

```text
创建/天 = 10M × 0.1 = 1M/day
平均写 QPS = 1M / 100,000 = 10 QPS
峰值写 QPS = 10 × 5 = 50 QPS

跳转/天 = 10M × 10 = 100M/day
平均读 QPS = 100M / 100,000 = 1,000 QPS
峰值读 QPS = 1,000 × 5 = 5,000 QPS

URL 存储/天 = 1M × 1KB = 1GB/day
5 年原始存储 = 1GB × 365 × 5 = 1.8TB
含 3 副本 = 5.4TB

跳转响应很小，主要是 302 header。按 1KB 响应：
峰值带宽 = 5,000 × 1KB = 5MB/s ≈ 40Mbps
```

结论：

- 写 QPS 很低，创建路径不需要复杂分片起步。
- 读峰值 5k QPS，Redis cache + read replicas 足够；更大规模可上 edge cache。
- 存储 5.4TB 可由分区后的 SQL 或 KV store 承担；访问模式是 `short_code -> long_url`，KV 很自然。

### 4.3 API 设计

```http
POST /v1/links
Idempotency-Key: 7f3a-client-generated
Content-Type: application/json

{
  "long_url": "https://example.com/articles/system-design",
  "custom_alias": "sys-design",
  "expires_at": "2027-07-07T00:00:00Z"
}

201 Created
{
  "short_url": "https://sho.rt/abc123",
  "short_code": "abc123",
  "long_url": "https://example.com/articles/system-design",
  "expires_at": "2027-07-07T00:00:00Z"
}
```

错误：

- `400` invalid URL。
- `409` custom alias already exists。
- `429` user create rate exceeded。
- `503` create service unavailable。

跳转：

```http
GET /{short_code}

302 Found
Location: https://example.com/articles/system-design
Cache-Control: private, max-age=60
```

统计查询：

```http
GET /v1/links/{short_code}/stats?from=2026-07-01&to=2026-07-07

200 OK
{
  "short_code": "abc123",
  "clicks": 12345,
  "top_referrers": [{"referrer": "x.com", "count": 321}],
  "top_countries": [{"country": "US", "count": 456}]
}
```

### 4.4 数据模型

核心表 / KV：

```text
links
- short_code: string primary key
- long_url: string
- user_id: string, index (user_id, created_at desc)
- status: active | expired | blocked
- created_at: timestamp
- expires_at: timestamp nullable
- custom_alias: boolean

idempotency_keys
- user_id: string
- idempotency_key: string
- short_code: string
- request_hash: string
- created_at: timestamp
unique(user_id, idempotency_key)

click_events（append-only，可进 Kafka/S3/ClickHouse）
- event_id: string
- short_code: string
- ts: timestamp
- ip_hash: string
- user_agent: string
- referrer: string
- country: string
```

索引 / 分区：

- `links.short_code`：跳转主路径，必须点查快。
- `(user_id, created_at desc)`：用户管理自己创建的短链。
- `click_events` 按日期分区，按 `short_code` 聚合。
- 若用 NoSQL，partition key = `short_code`；若统计写入巨大，点击事件按 `date + hash(short_code)` 分区。

短码生成方案对比：

| 方案 | 做法 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 自增 ID + Base62 | DB/ID service 生成整数，再 Base62 | 短、无碰撞、可控 | 可预测；需要 ID service |
| 随机 Base62 | 随机生成 6–8 位，碰撞重试 | 简单、不可预测 | 需要唯一检查；高占用率时碰撞增加 |
| Hash long URL | 对 long URL hash 后截断 | 同 URL 可复用 | 碰撞处理复杂；泄漏模式 |

推荐：普通短链用 ID service + Base62；如果担心可枚举，在 ID 上加 salt / shuffle，或用 7–8 位随机码。

Base62 容量：

```text
62^6 ≈ 56.8B（568 亿）
62^7 ≈ 3.5T
```

每天 100 万新短链，6 位理论可用 56,800 天，但考虑冲突、保留、恶意扫描，实际可用 7 位更安全。

### 4.5 高层架构图（文字）

```text
Create path:
Client
  → API Gateway（auth、rate limit）
  → Link Service
      → Idempotency Store（去重）
      → ID Generator（Snowflake / sequence range）
      → Link DB / KV Store（short_code -> long_url）
      → Redis（可选预热）
  ← short_url

Redirect path:
Browser
  → Edge / CDN（可缓存热门 short_code）
  → Load Balancer
  → Redirect Service
      → Redis cache
      → Link DB / KV Store（cache miss）
      → Kafka click_events（异步）
  ← 302 Location: long_url

Analytics path:
Kafka click_events
  → Stream Processor / Worker
  → Aggregation Store（ClickHouse / Druid / BigQuery）
  → Stats API
```

### 4.6 深挖 1：跳转低延迟与缓存一致性

读路径伪流程：

```text
GET /abc123
1. Validate short_code format；不合法直接 404，避免打 DB。
2. Redis GET link:abc123。
3. Cache hit：检查 status/expires_at，返回 302。
4. Cache miss：DB/KV get short_code。
5. DB miss：写 negative cache，TTL 30–60s，返回 404。
6. DB hit：Redis SET link:abc123 value TTL 1–24h（加随机抖动），返回 302。
7. 异步发送 click event；发送失败不阻塞跳转。
```

缓存策略：

- Cache-aside：简单可靠，适合读多写少。
- 热门短链：可延长 TTL，甚至放 CDN/Edge KV。
- Negative cache：防止随机 short_code 扫描打爆 DB。
- TTL jitter：例如 `ttl = 3600 + random(0, 600)`，避免雪崩。

一致性处理：

- 创建短链后写 DB 成功，再删除或写入 cache。因为创建后需要立即可读，可直接 `SET cache` 预热。
- 更新 long URL 或封禁短链：先更新 DB，再删除 cache；若删除失败，通过 Outbox 事件重试删除。
- 过期链接：cache value 必须包含 `expires_at`，即使 cache 未过期，服务也能判断链接已过期。

权衡话术：

> “我不把点击统计放在同步路径，因为跳转 P95 < 100ms 比统计实时性更重要。统计丢少量事件可接受，但跳转失败不可接受。所以 click event 写 Kafka 失败时只记录 error metric，不阻塞 302。”

### 4.7 深挖 2：短码唯一性、幂等与防滥用

创建流程：

```text
POST /v1/links
1. 校验 URL scheme 只允许 http/https。
2. 检查用户 rate limit：例如 free user 10/min。
3. 检查 Idempotency-Key：同 key + 同 request_hash 返回已有结果。
4. 若 custom_alias：尝试 insert short_code=alias；唯一冲突返回 409。
5. 若系统生成：向 ID Generator 取 ID，Base62 encode。
6. Insert links(short_code, long_url, user_id, ...)；short_code unique。
7. 写 idempotency_keys。
8. 返回 short_url。
```

ID Generator 选择：

- 单 DB sequence：简单，50 QPS 写入足够；但可用性依赖 DB。
- Segment/号段：服务一次取 `[start, end]`，本地发号，减少 DB 压力。
- Snowflake：时间戳 + 机器 ID + 序列号，适合多机高吞吐。

防滥用：

- 创建限流：per user / per IP token bucket。
- 跳转限流：对随机扫描 IP 做 404 rate limit。
- URL 安全扫描：创建后异步扫描；命中恶意则 `status=blocked` 并删除 cache。
- 自定义 alias 保留词：`admin`, `api`, `login` 等禁止。

### 4.8 权衡与演进

当前设计优先：跳转低延迟、高可用、统计异步。

牺牲：统计不是强实时；cache 可能有短暂脏数据；随机扫描需要额外防护。

10× 流量演进：

1. Redirect Service 扩容，Redis cluster 分片。
2. 热门短链放 CDN/Edge cache。
3. Link Store 从单库迁移到按 `hash(short_code)` 分片的 KV store。
4. Click events 从单 Kafka topic 拆分区，聚合层改 ClickHouse / Druid。
5. 多区域部署：short_code 数据异步复制，跳转走最近区域；创建可用 home region 或全局 ID。

关键监控：

- redirect P95/P99 latency、302 rate、404 rate、5xx rate。
- cache hit rate、negative cache hit rate、Redis CPU/memory。
- DB QPS、replication lag、hot key。
- Kafka produce error、queue lag、analytics freshness。

---

## 5. Worked Design #2：设计一个新闻 Feed 系统

题目：设计类似 Twitter/X、Instagram 或 LinkedIn 的 home feed。用户关注其他人，打开首页看到按时间或相关性排序的动态。

### 5.1 需求澄清

功能需求：

1. 用户发布 post。
2. 用户关注 / 取关其他用户。
3. 用户读取 home feed，分页加载。
4. 支持点赞 / 评论计数展示。
5. feed 排序先按时间倒序；ranking 作为扩展。

非功能需求：

- Feed 读取 P95 < 300ms。
- 发布 P95 < 500ms；名人用户发布可稍慢但不能拖垮系统。
- 最终一致可接受：新 post 出现在粉丝 feed 中允许延迟几秒。
- 高可用：读路径 99.9%+。

Out of Scope：

- 复杂 ML ranking、广告、推荐、内容审核细节、私信。

确认话术：

> “我先做关注关系下的 home timeline，排序按时间倒序。普通用户采用写扩散，超大 V 采用读扩散或混合模式，ranking 和广告先不展开。”

### 5.2 容量估算

假设：

- 5,000 万 DAU。
- 每人每天打开 feed 10 次。
- 每次返回 20 条，每条 feed item metadata 1KB。
- 10% 用户每天发 2 条 post → 1,000 万 posts/day。
- 平均粉丝数 200；大 V 可达 1,000 万粉丝。
- 峰值系数 5。

计算：

```text
feed read/day = 50M × 10 = 500M/day
平均读 QPS = 500M / 100,000 = 5,000 QPS
峰值读 QPS = 5,000 × 5 = 25,000 QPS

post write/day = 50M × 10% × 2 = 10M/day
平均发帖 QPS = 10M / 100,000 = 100 QPS
峰值发帖 QPS = 100 × 5 = 500 QPS

如果全部写扩散：
fanout writes/day = 10M posts × 200 followers = 2B feed items/day
平均 fanout 写 QPS = 2B / 100,000 = 20,000 QPS
峰值 fanout 写 QPS = 100,000 QPS

feed 响应大小 = 20 × 1KB = 20KB
峰值读带宽 = 25,000 × 20KB = 500MB/s ≈ 4Gbps

feed item 存储/天 = 2B × 100B（只存 post_id/author_id/ts）= 200GB/day
保留 30 天 = 6TB；3 副本 = 18TB
```

结论：

- 读 QPS 高，home feed 需要预计算或缓存，不能每次在线 join 所有关关注对象。
- 全量写扩散对普通用户可行，但大 V 会造成 fanout 爆炸。
- feed item 只存轻量引用，不复制完整 post 内容。

### 5.3 API 设计

发布 post：

```http
POST /v1/posts
Idempotency-Key: post-uuid-123
{
  "text": "system design notes",
  "media_ids": ["m_1", "m_2"],
  "visibility": "public"
}

201 Created
{
  "post_id": "p_987",
  "created_at": "2026-07-07T10:20:53Z"
}
```

读取 feed：

```http
GET /v1/feed/home?limit=20&cursor=eyJ0cyI6...

200 OK
{
  "items": [
    {
      "post_id": "p_987",
      "author_id": "u_1",
      "text": "system design notes",
      "created_at": "2026-07-07T10:20:53Z",
      "like_count": 123,
      "comment_count": 4
    }
  ],
  "next_cursor": "eyJ0cyI6..."
}
```

关注：

```http
POST /v1/users/{target_user_id}/follow
DELETE /v1/users/{target_user_id}/follow
```

错误：

- `401/403` 未登录或不可见。
- `404` post/user 不存在。
- `409` 重复 follow 或状态冲突。
- `429` 读取或发布过快。

### 5.4 数据模型

```text
users
- user_id: string primary key
- handle: string unique
- created_at: timestamp

posts
- post_id: string primary key
- author_id: string, index(author_id, created_at desc)
- text: string
- media_ids: array
- visibility: enum
- created_at: timestamp
- deleted_at: timestamp nullable

follows
- follower_id: string
- followee_id: string
- created_at: timestamp
primary key(follower_id, followee_id)
index(followee_id, follower_id)  -- fanout 时查粉丝

home_feed_items
- user_id: string
- sort_ts: timestamp
- post_id: string
- author_id: string
primary key(user_id, sort_ts desc, post_id)
TTL: 30–90 days

engagement_counts
- post_id: string primary key
- like_count: bigint
- comment_count: bigint
- updated_at: timestamp
```

存储选择：

- `users` / `posts`：SQL 或 document DB 都可；如果关系和权限复杂，SQL 更舒服。
- `follows`：访问模式明确、规模大，可用 wide-column / KV；也可 SQL 分区起步。
- `home_feed_items`：按 `user_id` 查询最近 N 条，适合 Cassandra/DynamoDB/Redis sorted set。
- `engagement_counts`：高频计数可先 Redis，异步 flush 到持久库。

### 5.5 高层架构图（文字）

```text
Post write path:
Client
  → API Gateway
  → Post Service
      → Post DB（写原始 post）
      → Outbox / Kafka: PostCreated(post_id, author_id, ts)

Fanout path:
Kafka PostCreated
  → Fanout Worker
      → Follow Store（查粉丝）
      → Home Feed Store（批量写入 follower feed）
      → Redis Feed Cache（可选预热活跃用户）

Read feed path:
Client
  → Feed Service
      → Redis home_feed:{user_id}（最近 items）
      → Home Feed Store（cache miss / pagination）
      → Post Service / Post Cache（批量取 post 内容）
      → Engagement Cache（批量取计数）
  ← feed items + next_cursor

Follow path:
Client
  → Follow Service
      → Follow Store
      → Kafka FollowChanged（异步修补 feed）
```

### 5.6 深挖 1：推 / 拉 / 混合 Feed

| 模式 | 做法 | 优点 | 缺点 | 适用 |
| --- | --- | --- | --- | --- |
| Push / Fanout-on-write | 发帖时写入每个粉丝的 home feed | 读很快，首页直接取自己的 inbox | 大 V 发帖写爆；关注关系变化要修补 | 普通用户、读多写少 |
| Pull / Fanout-on-read | 读 feed 时拉取关注者最新 posts 合并排序 | 写很轻，不怕大 V | 读时 join 多个作者，延迟高 | 关注数少、写多读少 |
| Hybrid | 普通用户 push，大 V pull；读时 merge | 平衡读延迟和写放大 | 逻辑复杂，排序和去重更难 | 大多数社交 feed |

推荐方案：Hybrid。

规则示例：

```text
if author.follower_count < 100,000:
    fanout post_id to followers' home_feed_items
else:
    mark author as celebrity
    do not fanout to all followers
    read path pulls celebrity recent posts and merges with precomputed feed
```

读路径 merge：

1. 从 `home_feed_items[user_id]` 取普通作者推送的 50 条候选。
2. 找出用户关注的大 V 列表，批量取每个大 V 最近 5–20 条 post。
3. 合并候选，按时间或 ranking score 排序。
4. 去重、过滤删除 / blocked / visibility 不可见内容。
5. 返回前 20 条和 cursor。

权衡话术：

> “我不用纯 push，因为 1,000 万粉丝的大 V 发一条会制造 1,000 万次写入，热点时会拖垮 fanout worker。Hybrid 让普通用户读很快，同时把大 V 的成本转移到读路径；代价是读路径需要 merge 和缓存。”

### 5.7 深挖 2：Fanout 可靠性、幂等与延迟控制

Fanout worker 流程：

```text
1. Consume PostCreated(post_id, author_id, ts, event_id)。
2. 用 author_id 查询粉丝列表，按分页批量读取。
3. 对每批 follower 生成 feed item：
   key = (follower_id, ts, post_id)
4. 批量写 Home Feed Store。
5. 记录 progress checkpoint：event_id + follower_page_cursor。
6. 失败重试；超过阈值进入 DLQ。
```

幂等设计：

- `home_feed_items` 主键包含 `(user_id, post_id)` 或写入条件 `insert if not exists`，重复消费不会重复出现。
- `PostCreated.event_id` 用于 worker checkpoint。
- `FollowChanged` 修补 feed 时也按 `post_id` 去重。

延迟控制：

- 普通用户 fanout 目标：P95 在 5 秒内进入粉丝 feed。
- Worker 根据 queue lag 自动扩容。
- 对超大 fanout 任务切成小 batch，例如每批 1,000 粉丝。
- 如果 lag 超阈值，降级：新 post 暂时通过 pull merge 出现在活跃用户 feed，而不是等待 fanout 完成。

删除 / 隐私变更：

- post 删除后，不必同步删除所有 feed item；读路径取 post 内容时发现 deleted/不可见，过滤掉。
- 若法律或隐私要求强删除，可以异步 tombstone + compaction。

### 5.8 深挖 3：缓存、分页与 Ranking 扩展

缓存：

- `Redis sorted set home_feed:{user_id}` 存最近 500–1000 个 post_id，score = timestamp 或 ranking score。
- 活跃用户缓存，长尾用户走 Home Feed Store。
- 计数缓存 `post_counts:{post_id}` 批量 MGET，避免 N+1。

分页：

- 使用 cursor，不用 offset。
- cursor 内容：`last_score`, `last_post_id`, `ranking_version`。
- offset 在深页会变慢，且新内容插入会导致重复或漏读。

Ranking 扩展：

```text
score = w1 * recency_score
      + w2 * affinity(user, author)
      + w3 * engagement_score
      - w4 * negative_feedback
```

面试中处理 ranking：

> “我先按时间倒序保证基础 feed 可用。Ranking 可以作为独立服务，在候选集生成后打分，不改变底层 fanout 模型。这样基础架构和 ML 迭代解耦。”

### 5.9 权衡与演进

当前设计优先：读低延迟、普通用户体验、最终一致。

牺牲：写扩散带来额外存储；大 V 内容在读路径 merge 增加复杂度；ranking 初版较简单。

10× 流量演进：

1. Follow Store 和 Home Feed Store 按 `user_id` hash 分片。
2. Fanout Worker 按 author_id/event_id 分区，批量写入。
3. Feed Cache 只覆盖活跃用户，长尾走持久 store。
4. Engagement count 使用近实时流处理，最终校正。
5. Ranking 独立服务化，支持 feature store 和在线实验。
6. 多区域：读本地 feed cache，写入 home region，异步复制；跨区域一致性按秒级 SLA。

关键监控：

- feed read P95/P99、empty feed rate、duplicate item rate。
- fanout queue lag、fanout success rate、DLQ count。
- cache hit rate、Home Feed Store read/write latency。
- celebrity merge latency、ranking service timeout。
- 数据新鲜度：post 创建到出现在粉丝 feed 的 P50/P95。

---

## 6. 评分维度：面试官到底在看什么

| 维度 | 弱信号 | 强信号 |
| --- | --- | --- |
| 需求澄清 | 一上来画图 | 主动确认功能、非功能、Out of Scope |
| 数字估算 | “流量很大” | 给 QPS、存储、带宽，并用数字解释组件选择 |
| API / 模型 | 只画组件 | 给接口、字段、索引、分区键、错误码 |
| 架构分层 | 堆中间件 | 从最小可用系统演进到缓存、队列、分片 |
| 深挖能力 | 只讲 happy path | 讲故障、重试、幂等、热点、降级 |
| 取舍 | “这样最好” | 明确为了 A 牺牲 B，并说明何时换方案 |
| 沟通 | 长时间沉默 | 边想边讲，阶段性确认，接受面试官约束 |
| 生产经验 | 不提监控 | 给 SLO、metrics、alert、runbook 方向 |

### 6.1 Senior signal 怎么展示

1. **主动设置边界**：
   > “为了讲深，我把推荐算法和广告先放到 out of scope，但保留事件流接口，后续可接 ranking service。”

2. **用数字裁剪设计**：
   > “写峰值只有 50 QPS，所以创建路径先不需要复杂分片；真正瓶颈是 5k 跳转 QPS 和热点链接。”

3. **先简单后复杂**：
   > “第一版用 SQL + Redis 足够；当数据到 10TB 或写 QPS 到 10k，再迁移到按 key 分片的 KV store。”

4. **讲失败路径**：
   > “Kafka 重复投递是正常情况，所以 consumer 必须幂等；我用 `(user_id, post_id)` 唯一键去重。”

5. **讲运维指标**：
   > “上线后我会看 cache hit rate、DB QPS、queue lag、P99 latency。如果 cache hit rate 从 90% 掉到 50%，需要检查 key 失效或热点迁移。”

6. **识别组织约束**：
   > “如果团队缺少运维 Kafka 的经验，早期可以用托管队列如 SQS/Pub/Sub，牺牲部分吞吐换可靠交付。”

---

## 7. 常见坑 & 排查表

| 坑 | 面试表现 | 后果 | 修正话术 / 动作 |
| --- | --- | --- | --- |
| 跳过澄清 | “我先画一个微服务架构” | 设计错方向 | “我先确认核心 use case 和规模，避免过度设计。” |
| 不做估算 | “加缓存因为系统很大” | 决策无依据 | 用 2 分钟算读写 QPS、存储、带宽，再决定组件 |
| 只堆组件 | Redis、Kafka、ES 全上 | 像背答案 | 每加一个组件，说清解决的瓶颈和引入的代价 |
| API 缺失 | 只有框图 | 系统边界模糊 | 至少列创建、读取、分页、错误码、幂等 |
| 数据模型缺失 | 不知道怎么查 | 架构无法落地 | 给主键、索引、访问模式、分区键 |
| 忽略写路径 | 只优化读 | 数据不一致或写爆 | 对每个写操作说明事务边界、异步事件、重试 |
| 忽略失败 | happy path 讲完 | senior 信号不足 | 主动讲缓存 miss、MQ 重复、DB failover、限流 |
| 一致性说不清 | “保证一致” | 不可信 | 明确强一致范围，其余最终一致 + 补偿 |
| 分片键随便选 | “按时间分片” | 热点或跨分片查询 | 根据主访问模式选 key；说明热点和 re-sharding |
| 分页用 offset | `?page=10000` | 深页慢、重复漏读 | 用 cursor：`last_score + last_id` |
| 不收尾 | 时间到戛然而止 | 面试官记不住亮点 | 最后 60 秒总结取舍、瓶颈、监控、10× 演进 |
| 英语表达卡住 | 长时间沉默 | 沟通分下降 | 提前准备阶段话术；参考 [技术面试英语](../english/01-technical-interview-english.md) |

---

## 8. 面试中可复用的检查清单

### 8.1 开始前 60 秒

- [ ] 我复述了题目，确认自己理解正确。
- [ ] 我声明了 45 分钟流程，让面试官知道我会覆盖哪些阶段。
- [ ] 我没有立刻画最终架构图。

### 8.2 需求澄清

- [ ] 功能需求限制在 2–4 个核心 use cases。
- [ ] 非功能需求包括规模、延迟、可用性、一致性、保留时间。
- [ ] 我明确说了 Out of Scope。
- [ ] 我用一句话向面试官确认范围。

### 8.3 估算

- [ ] 我计算了平均 QPS 和峰值 QPS。
- [ ] 我计算了存储增长或缓存容量。
- [ ] 我计算了带宽或响应大小。
- [ ] 我把数字转化成架构决策，而不是算完就丢。

### 8.4 API 与数据模型

- [ ] API 覆盖创建、读取、分页 / 查询、错误场景。
- [ ] 写接口考虑了幂等键。
- [ ] 数据模型列出主键、二级索引、分区键。
- [ ] 我解释了 SQL vs NoSQL 的选择。

### 8.5 架构与深挖

- [ ] 高层架构能讲通一次读请求和一次写请求。
- [ ] 每个组件都有明确理由和代价。
- [ ] 深挖至少包含一个失败场景和补救方案。
- [ ] 我没有同时深挖超过 2 个主题导致失焦。

### 8.6 收尾

- [ ] 我总结了核心取舍。
- [ ] 我说明了 10× 流量时的演进路径。
- [ ] 我给了关键监控指标。
- [ ] 我留出最后 1 分钟，而不是被面试官打断。

---

## 9. 动手练习：把框架练成肌肉记忆

### 9.1 练习方式

每题严格 45 分钟，用手机计时。不要暂停查资料；面试就是在不完美信息下做合理假设。结束后用录音或文字复盘。

每题产出物（deliverables）：

1. 需求澄清清单：功能 / 非功能 / Out of Scope。
2. 估算草稿：读 QPS、写 QPS、存储、带宽，写出公式。
3. API：3–5 个核心接口。
4. 数据模型：实体、主键、索引、分区键。
5. 架构图：文字箭头图即可。
6. 深挖笔记：1–2 个组件，包含失败场景。
7. 60 秒总结稿：取舍、瓶颈、监控、10× 演进。

### 9.2 三个计时题

#### 题 1：设计 Rate Limiter（难度：中）

约束：

- 支持 per user / per IP / per API key。
- 10 万 QPS。
- 限流策略支持 token bucket 和 sliding window。
- 多区域部署时允许轻微误差。

必须深挖：

- Redis 计数 vs 本地计数 vs 分布式近似。
- 误杀正常用户如何缓解。
- Redis 故障时 fail open 还是 fail closed。

#### 题 2：设计聊天系统（难度：中高）

约束：

- 1:1 聊天和群聊。
- 在线消息 P95 < 200ms。
- 离线消息可靠送达。
- 支持已读回执，最终一致可接受。

必须深挖：

- WebSocket gateway 如何保持无状态或弱状态。
- Message Store 的分区键。
- 重试导致重复消息时如何用 `client_msg_id` 幂等。

#### 题 3：设计通知系统（难度：高）

约束：

- Email、Push、SMS 三个渠道。
- 每天 10 亿通知。
- 需要用户偏好、退订、频控。
- 第三方 provider 不稳定。

必须深挖：

- MQ、重试、DLQ、provider failover。
- 幂等发送和去重。
- 用户频控与优先级队列。

### 9.3 复盘评分表

每项 0–2 分，总分 20：

| 项目 | 0 分 | 1 分 | 2 分 |
| --- | --- | --- | --- |
| 需求澄清 | 基本没有 | 问了但未确认 | 有边界、有确认 |
| 估算 | 无数字 | 有数字无决策 | 数字驱动架构 |
| API | 缺失 | 粗略 | 字段、错误、幂等清楚 |
| 数据模型 | 缺失 | 有实体 | 有索引和分区键 |
| 架构 | 堆组件 | 能跑通 | 数据流清楚、有演进 |
| 深挖 | 泛泛 | 讲一个点 | 有失败路径和权衡 |
| 一致性 | 含糊 | 有结论 | 强/最终一致边界清楚 |
| 可扩展性 | 没提 | 水平扩容 | 分片、热点、10× 路径 |
| 可观测性 | 没提 | 提指标 | 指标对应风险 |
| 沟通 | 失控 | 基本清楚 | 主导节奏、阶段确认 |

目标：连续 3 题达到 16 分以上，再开始 mock interview。

---

## 10. 延伸阅读

- [System Design Primer](https://github.com/donnemartin/system-design-primer)：经典开源资料，适合补齐组件基础和常见题。
- [ByteByteGo Blog](https://blog.bytebytego.com/)：大量可视化系统设计案例，适合学习表达方式。
- [High Scalability](http://highscalability.com/)：真实大规模系统案例和架构演进。
- [AWS Architecture Center](https://aws.amazon.com/architecture/)：云上架构、可靠性、安全和成本设计参考。
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)：SLO、错误预算、可靠性工程的权威读物。
- [Martin Fowler: Patterns of Distributed Systems](https://martinfowler.com/articles/patterns-of-distributed-systems/)：分布式系统模式，适合深挖一致性和复制。
- [Designing Data-Intensive Applications](https://dataintensive.net/)：数据系统设计经典书，重点读复制、分区、事务、流处理。
- [技术面试英语](../english/01-technical-interview-english.md)：把本章脚本翻成英语并练到能边画边讲。

---

## 小结

系统设计面试的稳定打法是：**先定边界，再用数字推架构，最后深挖取舍**。45 分钟里不要追求“完美系统”，而要展示你能在约束下做工程决策：什么先做、什么不做、为什么这样做、失败了怎么兜底、规模扩大后如何演进。

把本章的时间盒脚本背熟，再用短链接、Feed、聊天、限流、通知等题反复计时练习。真正的目标不是记住某个答案，而是让面试官相信：把一个真实系统交给你，你能带团队把它设计、上线并持续演进。

`标签` `系统设计` `面试` `分布式` `架构` `海外求职` `容量估算` `Senior Engineer`

---

[← 上一章](02-remote-overseas-job-strategy.md) · [WP-02 目录](README.md) · [下一章 →](04-behavioral-interview-star.md)
