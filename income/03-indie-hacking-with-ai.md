[← 手册首页](../README.md) · [WP-05 独立变现](README.md)

---

# 独立开发者：用 AI 快速做小产品并变现

> 从想法到收款：用 AI 加速验证、构建、上线小产品，跑通第一笔睡后收入。

本文仅供教育与参考，非专业法律、税务、投资建议；涉及收款、税务、公司主体、跨境支付、消费者权益与数据合规时，请结合所在地法规咨询专业人士。

## 本章定位：在“逃离阶梯”里把工程能力变成可复用资产

独立开发不是“辞职做一个改变世界的大产品”，而是用工程能力连续做小实验：发现一个窄问题，验证有人愿意付费，用 AI 降低构建成本，用分发能力把产品送到需要它的人面前。

本章服务《逃离坐班白皮书》的“变现”层：它连接 [变现地图](01-monetization-landscape.md) 的收入模型、[内容与个人品牌](05-content-personal-brand.md) 的获客能力，以及 AI 工程册里的 [LLM 应用架构](../ai-engineering/02-llm-app-architecture.md) 与 [成本/延迟优化](../ai-engineering/08-llm-cost-latency-optimization.md)。

读完并照做后，你应该能完成以下可度量产出：

- 在 48 小时内列出 20 个可验证的小产品想法，并筛出 3 个候选。
- 在不写核心产品代码前，用 landing page、waitlist、访谈或 pre-sale 验证需求。
- 用 Next.js、Postgres、Auth、Stripe 和 AI coding 工具搭建一个最小 SaaS。
- 跑通 Stripe test mode：创建 checkout session、接收 webhook、解锁付费功能。
- 用 MRR、churn、LTV、CAC 判断产品是否值得继续投入。
- 制定 6–8 周从 0 到 first paying customer 的执行计划。

## 前置条件与工具清单

你不需要先辞职，也不需要融资；你需要的是一个能持续 6–8 周投入的时间盒。

### 技术前置

- 熟悉 TypeScript、React、HTTP API、SQL。
- 能使用 Git、GitHub、命令行、环境变量。
- 了解 SaaS 的基本概念：用户、订阅、权限、发票、退款。
- 如果产品包含 LLM 功能，先读 [LLM 应用架构](../ai-engineering/02-llm-app-architecture.md)，避免把 prompt demo 当成产品架构。

### 推荐栈

| 层 | 推荐选择 | 为什么适合独立开发 |
|---|---|---|
| Web 框架 | Next.js App Router | 前后端一体，部署和 SEO 友好 |
| 语言 | TypeScript | 让 AI 生成代码时更容易被类型系统约束 |
| UI | Tailwind CSS + shadcn/ui | 快速做出可销售的界面 |
| 数据库 | Postgres | 关系数据、订阅状态、审计记录都适合 |
| ORM | Prisma 或 Drizzle | schema 可读，迁移清晰 |
| Auth | Clerk、Auth.js、Supabase Auth | 少写登录注册和安全边界 |
| 支付 | Stripe | test mode、Checkout、Customer Portal、Webhook 完整 |
| 部署 | Vercel + Neon/Supabase | 前期省运维，能快速上线 |
| 分析 | Plausible、PostHog、Umami | 跟踪漏斗和留存 |
| AI coding | GitHub Copilot、Cursor、Claude Code 等 | 生成脚手架、测试、SQL、重构 |

### 工作节奏

- 每天 90–150 分钟深度工作。
- 每周至少 5 次公开输出：日志、截图、demo、学习笔记、案例拆解。
- 每周至少 20 次分发动作：私信、评论、邮件、社区回复、冷启动邀约。
- 每周只推进 1 个主想法，避免同时做 5 个半成品。

## 核心心智：AI 降低构建成本，但抬高分发权重

过去的独立开发瓶颈常常是“写不完”；现在 AI 能帮你写脚手架、CRUD、测试、文案、SQL、落地页，瓶颈变成“有没有人要”。

### 小赌注，而不是大迁徙

把每个产品看成一个带上限亏损的小赌注：

| 维度 | 大迁徙式创业 | 小赌注式独立开发 |
|---|---|---|
| 起点 | 辞职、融资、做完整产品 | 保持现金流，做 2–6 周实验 |
| 成功标准 | 大规模增长 | 第一个愿意付费的窄人群 |
| 风险 | 时间、现金流、身份压力同时暴露 | 每次实验亏损可控 |
| 构建方式 | 先产品后市场 | 先分发和验证，再做 MVP |
| 退出机制 | 不好停 | 数据不达标就 kill |

一个小赌注的预算可以写成：

- 时间：不超过 40–80 小时。
- 现金：不超过 500–2000 元人民币或等值美元。
- 心理：允许失败，但必须产出复用资产，如 landing page、文章、组件、邮件模板、访谈记录。

### Distribution-first 的实际含义

“先分发”不是先发广告，而是在写代码前确认三件事：

1. 你知道目标用户在哪里聚集。
2. 你能用他们的语言描述痛点。
3. 你能在不骚扰的前提下触达 50–200 个潜在用户。

如果你找不到 50 个潜在用户，就不要先写产品；先写用户名单、社区地图、关键词列表。

### Build in public 不是表演，而是复利记录

Build in public 的目标不是获得点赞，而是积累信任与可被搜索的上下文。

可发布内容包括：

- 你发现的痛点截图：隐藏隐私信息。
- 你如何验证需求：问卷、访谈、waitlist 数字。
- 你做出的取舍：为什么砍掉某个功能。
- 你学到的工程坑：Stripe webhook、LLM 成本、部署问题。
- 你获得的用户反馈：原话、截图、修复前后。

对工程师来说，这些内容也是未来咨询、外包、课程、模板、微型 SaaS 的资产，和 [内容与个人品牌](05-content-personal-brand.md) 的路径互相增强。

## 一条可执行总流程

把独立开发拆成 8 个闸门，每个闸门都有“继续/停止”标准。

```text
痛点池
  ↓
问题筛选
  ↓
需求验证
  ↓
预售或 waitlist
  ↓
MVP 构建
  ↓
支付上线
  ↓
分发发布
  ↓
指标迭代 / kill / double down
```

每个阶段都写出一个 artifact：

| 阶段 | 产出物 | 不合格信号 |
|---|---|---|
| 痛点池 | 20 条具体问题 | 只有“AI 写作工具”这种泛泛分类 |
| 问题筛选 | 3 个候选 + 评分 | 不知道付费者是谁 |
| 需求验证 | 10 次访谈或 100 个 waitlist 曝光 | 只有朋友说“挺好” |
| 预售 | 1–3 个付款或强承诺 | 用户只愿意免费试 |
| MVP | 1 个核心 job-to-be-done | 功能超过 5 个 |
| 支付上线 | Stripe test + live checklist | 权限和订阅状态手工改 |
| 分发发布 | 1 个 launch page + 20 个触达动作 | 只发一条朋友圈 |
| 迭代 | 周报指标 | 不看数据，凭感觉加功能 |

## 第 1 阶段：找到痛到愿意付费的问题

### 1. 从你有优势的场景开始

资深工程师最容易变现的不是“所有人都用的工具”，而是你理解很深的窄流程。

优先从这 5 类找：

1. 你工作中重复做过 20 次以上的任务。
2. 你所在行业有大量 Excel、截图、复制粘贴、人工审核。
3. 你能接触到真实用户并拿到反馈。
4. 你知道现有工具贵、慢、复杂或不适合小团队。
5. AI 可以把原来 30 分钟的任务压到 3 分钟以内。

### 2. 用“痛点句式”写问题

不要写：

> 做一个 AI 简历优化工具。

改成：

> 北美求职的后端工程师每投递 20 个岗位就要手动改 20 次简历关键词；他们愿意花 9–19 美元买一个能根据 JD 自动生成 diff 的工具。

痛点句式：

```text
谁（具体人群）
在什么场景
为了完成什么任务
现在用什么低效替代方案
每次损失多少时间/钱/机会
为什么现有方案不够好
他可能为什么付费
```

### 3. 建一个 20 条痛点池

用下面表格记录，不要只放在脑子里。

| # | 人群 | 场景 | 现有替代方案 | 频率 | 付费理由 | 触达渠道 |
|---|---|---|---|---|---|---|
| 1 | 独立咨询顾问 | 写 proposal | 复制旧文档 | 每周 | 提高成交率 | LinkedIn、X |
| 2 | 小团队 CTO | 审 PR 风险 | 人肉扫 diff | 每天 | 降低事故 | GitHub 社区 |
| 3 | 跨境卖家 | 生成商品 SEO 文案 | 外包/表格 | 每天 | 节省运营时间 | Facebook group |

完成标准：

- 至少 20 条。
- 每条都有明确人群和触达渠道。
- 至少 5 条来自真实对话、社区帖子或你亲身经历。

### 4. 用评分矩阵筛选

每项 1–5 分，总分低于 18 不进入验证。

| 指标 | 1 分 | 3 分 | 5 分 |
|---|---|---|---|
| 痛感 | 只是 nice-to-have | 偶尔抱怨 | 影响收入/效率/合规 |
| 频率 | 年度 | 月度 | 每周/每天 |
| 预算 | 个人无预算 | 小额可付 | 公司或专业人士预算 |
| 触达 | 不知道去哪找 | 有 1 个渠道 | 有 3 个以上渠道 |
| 构建难度 | 需大团队 | 2–4 周 | 2–7 天 MVP |
| 差异化 | 完全红海 | 有细分角度 | 你有独特经验/数据/渠道 |

Go 标准：

- 总分 ≥ 22。
- 你能写出 50 个潜在用户名单或关键词。
- MVP 能在 7 天内实现核心价值。

No-go 标准：

- 必须先做移动端、浏览器插件、复杂协同、重合规才能验证。
- 目标用户没有明显付费习惯。
- 你只能通过买广告触达用户。

## 第 2 阶段：先验证，再构建

需求验证的目标不是证明你聪明，而是找到“现在就愿意行动”的人。

### 验证方法 A：社区信号

找 3–5 个目标用户聚集地：

- Reddit、Hacker News、Indie Hackers、Product Hunt。
- X、LinkedIn、即刻、小红书、公众号评论区。
- Discord、Slack、Telegram、微信群、垂直论坛。
- GitHub issues、Stack Overflow、Chrome Web Store 评论。
- 竞品的 G2、Capterra、AppSumo、Trustpilot 评论。

搜索关键词模板：

```text
"I hate" + 任务关键词
"how do you" + 任务关键词
"alternative to" + 竞品名
"too expensive" + 竞品名
"manually" + 任务关键词
"spreadsheet" + 任务关键词
"有没有工具" + 任务关键词
"太贵了" + 竞品名
```

记录证据：

| 证据 | 链接/截图 | 用户原话 | 频率 | 可产品化机会 |
|---|---|---|---|---|
| Reddit thread | URL | “I spend 2 hours every Friday...” | 12 comments | 自动生成报告 |
| 竞品差评 | URL | “Pricing is insane for solo users” | 8 reviews | 低价轻量版 |

完成标准：

- 找到 10 条以上来自陌生人的痛点原话。
- 至少 3 条提到时间、金钱、风险或替代方案。
- 至少 1 个现有替代品有人付费。

### 验证方法 B：Landing page + Waitlist

不要先做完整产品；先做一个能收集承诺的页面。

页面结构：

1. Hero：一句话说明结果。
2. Problem：列出 3 个具体痛点。
3. Demo：GIF、截图、假门 demo 或 Loom 视频。
4. Outcome：使用前后对比。
5. Pricing anchor：写出预计价格，例如 `$19/mo early access`。
6. CTA：加入 waitlist、预约访谈、预购。
7. FAQ：数据安全、退款、上线时间、适合谁。

Hero 模板：

```text
把 [目标用户] 每周 [耗时任务] 从 [当前耗时] 缩短到 [目标耗时]。
上传 [输入]，获得 [可交付结果]，无需 [讨厌的步骤]。
```

示例：

```text
把独立咨询顾问每周 3 小时的 proposal 初稿缩短到 10 分钟。
粘贴客户需求，生成报价结构、风险条款和交付里程碑，无需翻旧文档。
```

最小实现可以用：

- Carrd、Framer、Webflow。
- Next.js 单页 + Supabase waitlist 表。
- Tally/Typeform + Stripe Payment Link。

Waitlist 表结构：

```sql
create table waitlist_signups (
  id uuid primary key default gen_random_uuid(),
  email text not null unique,
  role text,
  company_size text,
  pain_level int check (pain_level between 1 and 5),
  willingness_to_pay text,
  source text,
  created_at timestamptz not null default now()
);
```

Waitlist CTA 不要只写“Join waitlist”，加一个筛选问题：

```text
你现在如何解决这个问题？
[ ] 手工处理
[ ] 用竞品，但不满意
[ ] 外包给别人
[ ] 还没解决，只是好奇

如果能节省 2 小时/周，你愿意支付？
[ ] 不愿意
[ ] $9/mo
[ ] $19/mo
[ ] $49/mo+
```

Go 标准：

- 100 个目标曝光内拿到 ≥ 10 个 waitlist。
- waitlist 中 ≥ 30% 填写了高痛感或付费意愿。
- 至少 3 人愿意接受 15 分钟访谈。

No-go 标准：

- 只有泛流量点赞，没有目标用户留下联系方式。
- 用户只想看免费内容，不愿意描述当前痛苦。
- 你无法找到第二个有效获客渠道。

### 验证方法 C：Pre-sale

预售比 waitlist 更强，因为它验证支付行为。

简单预售路径：

1. 写 landing page，说明 early access 价格和退款承诺。
2. 使用 Stripe Payment Link 或 Lemon Squeezy 创建一次性付款。
3. 明确交付时间：例如 14 天内给 beta 访问。
4. 如果未交付，主动退款。

预售文案：

```text
我正在做一个面向 [人群] 的 [结果型工具]。
目标是把 [任务] 从 [当前成本] 降到 [目标成本]。

早鸟价：$19，一次性购买，包含：
- beta 访问权；
- 30 分钟 onboarding；
- 未来 3 个月所有更新；
- 如果 14 天内没有可用版本，自动退款。

如果你愿意试，我发你付款链接；如果不愿意，也想请你告诉我最大顾虑是什么。
```

Go 标准：

- 7 天内拿到 1–3 个预售付款；或
- 5 个以上明确说“上线后按这个价格买”的目标用户，并留下联系方式；或
- 有一个团队愿意付费试点。

No-go 标准：

- 所有人都说“做好告诉我”，但没人愿意留邮箱、约访谈、付定金。
- 需要大量教育市场，用户还不知道为什么要解决这个问题。

### 访谈脚本：不要推销，问过去行为

访谈 15 分钟即可。

开场：

```text
我不是来卖东西的。我想理解你现在怎么处理 [任务]。
如果中途你觉得不相关，可以直接打断。
```

问题顺序：

1. 你上一次处理这个任务是什么时候？
2. 当时从哪里开始？用了哪些工具？
3. 最耗时/最烦/最容易出错的是哪一步？
4. 如果这一步失败，会造成什么损失？
5. 你现在为相关工具或服务付费吗？多少钱？
6. 如果有一个工具能解决 70%，你希望它怎么工作？
7. 我可以上线 beta 后邀请你试用吗？

不要问：

- “你会不会用？”
- “你觉得这个想法好吗？”
- “如果免费你愿意试吗？”

完成标准：

- 10 次访谈后，你能写出用户的原话，而不是你的转述。
- 你能画出当前 workflow。
- 你知道用户愿意替换哪一个现有工具或人工步骤。

## 第 3 阶段：用 AI 在数天内构建 MVP

MVP 只做一个 job-to-be-done：输入一份材料，输出一个用户可使用的结果，并通过支付权限控制访问。

### MVP 范围切割

以“AI proposal generator”为例：

| 功能 | MVP 做 | 暂不做 |
|---|---|---|
| 输入 | 粘贴客户需求文本 | 文件上传、多格式解析 |
| 输出 | 生成 proposal 草稿 | 团队协作、版本管理 |
| 用户 | 登录后保存 10 条历史 | 企业 SSO |
| 支付 | Stripe 订阅解锁 | 优惠券、复杂套餐 |
| AI | 单次 LLM 调用 + 模板 prompt | RAG、多 agent、fine-tune |
| 分析 | 记录 signup、checkout、generation | 完整 BI |

如果产品涉及复杂 LLM 流程，先用 [LLM 应用架构](../ai-engineering/02-llm-app-architecture.md) 的分层方法拆开：UI、API、orchestration、model gateway、storage、evals；上线后再按 [成本/延迟优化](../ai-engineering/08-llm-cost-latency-optimization.md) 做缓存、批处理、模型降级。

### AI coding 工作流

不要让 AI “从零写一个 SaaS”；给它边界和验收标准。

推荐提示词：

```text
你是一个资深 Next.js/TypeScript 工程师。
请在现有项目中实现 [功能]。
约束：
- 使用 Next.js App Router。
- 所有服务端逻辑放在 app/api 或 server-only module。
- 数据库通过 Prisma。
- 需要错误处理和最小测试。
- 不要引入新依赖，除非说明原因。

验收标准：
- npm run build 通过。
- 未登录用户不能访问 /app。
- 未订阅用户看到 upgrade CTA。
- Stripe webhook 能把 subscription status 写入数据库。
```

AI 适合做：

- 生成样板代码和类型定义。
- 把接口拆成小函数。
- 根据 Stripe 文档生成 webhook 骨架。
- 写 seed 数据、单元测试、Playwright happy path。
- 重构 UI 状态和错误信息。

你必须亲自把关：

- 支付状态和权限边界。
- Webhook 签名校验。
- 环境变量和 secret 管理。
- 数据删除、隐私、日志脱敏。
- 账单、退款、税务提示。

## 最小 SaaS 实操：Next.js + Prisma + Stripe

下面示例是“登录用户提交文本，付费用户才能生成结果”的最小骨架。代码片段是代表性实现，你可以放进一个 Next.js App Router 项目中改造。

### 1. 创建项目

```bash
npx create-next-app@latest indie-ai-saas \
  --ts \
  --tailwind \
  --eslint \
  --app \
  --src-dir=false \
  --import-alias="@/*"

cd indie-ai-saas
npm install prisma @prisma/client stripe zod
npm install -D tsx
npx prisma init
```

完成标准：

- `npm run dev` 能启动。
- 项目根目录存在 `prisma/schema.prisma`。
- `.env` 中有 `DATABASE_URL`。

### 2. 定义数据模型

`prisma/schema.prisma`：

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                   String   @id @default(cuid())
  email                String   @unique
  stripeCustomerId      String?  @unique
  subscriptionStatus    String   @default("free")
  subscriptionCurrentEnd DateTime?
  createdAt             DateTime @default(now())
  generations           Generation[]
}

model Generation {
  id        String   @id @default(cuid())
  userId    String
  input     String
  output    String
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
}
```

迁移：

```bash
npx prisma migrate dev --name init
```

完成标准：

- 数据库里出现 `User` 和 `Generation` 表。
- Prisma Client 生成成功。

### 3. 封装 Prisma 和 Stripe

`lib/db.ts`：

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma?: PrismaClient;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" ? ["query", "error", "warn"] : ["error"],
  });

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}
```

`lib/stripe.ts`：

```ts
import Stripe from "stripe";

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error("Missing STRIPE_SECRET_KEY");
}

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: "2024-06-20",
  typescript: true,
});
```

`.env` 示例：

```bash
DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/DB"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
STRIPE_SECRET_KEY="sk_test_xxx"
STRIPE_WEBHOOK_SECRET="whsec_xxx"
STRIPE_PRICE_ID="price_xxx"
```

不要把 `.env` 提交到 Git。

### 4. 实现当前用户获取

真实项目建议接入 Clerk、Auth.js 或 Supabase Auth。这里用一个极简接口表示“已登录用户”，便于聚焦付费逻辑。

`lib/current-user.ts`：

```ts
import { prisma } from "@/lib/db";

export async function getCurrentUser() {
  const email = "demo@example.com";

  return prisma.user.upsert({
    where: { email },
    update: {},
    create: { email },
  });
}
```

接入真实 Auth 后，把 `email` 替换成 session 中的用户邮箱或 user id。

完成标准：

- 调用 `getCurrentUser()` 会返回数据库用户。
- 未登录状态在真实项目中必须返回 `null` 并跳转登录。

### 5. 创建 Stripe Checkout handler

`app/api/stripe/checkout/route.ts`：

```ts
import { NextResponse } from "next/server";
import { getCurrentUser } from "@/lib/current-user";
import { prisma } from "@/lib/db";
import { stripe } from "@/lib/stripe";

export async function POST() {
  const user = await getCurrentUser();

  if (!process.env.NEXT_PUBLIC_APP_URL || !process.env.STRIPE_PRICE_ID) {
    return NextResponse.json({ error: "Payment is not configured" }, { status: 500 });
  }

  let stripeCustomerId = user.stripeCustomerId;

  if (!stripeCustomerId) {
    const customer = await stripe.customers.create({
      email: user.email,
      metadata: { userId: user.id },
    });

    stripeCustomerId = customer.id;

    await prisma.user.update({
      where: { id: user.id },
      data: { stripeCustomerId },
    });
  }

  const session = await stripe.checkout.sessions.create({
    mode: "subscription",
    customer: stripeCustomerId,
    line_items: [{ price: process.env.STRIPE_PRICE_ID, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/app?checkout=success`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing?checkout=cancelled`,
    allow_promotion_codes: true,
    subscription_data: {
      metadata: { userId: user.id },
    },
  });

  if (!session.url) {
    return NextResponse.json({ error: "Unable to create checkout session" }, { status: 500 });
  }

  return NextResponse.json({ url: session.url });
}
```

前端按钮：

```tsx
"use client";

import { useState } from "react";

export function UpgradeButton() {
  const [loading, setLoading] = useState(false);

  async function startCheckout() {
    setLoading(true);
    const res = await fetch("/api/stripe/checkout", { method: "POST" });
    const data = (await res.json()) as { url?: string; error?: string };

    if (!res.ok || !data.url) {
      setLoading(false);
      alert(data.error ?? "Checkout failed");
      return;
    }

    window.location.href = data.url;
  }

  return (
    <button
      onClick={startCheckout}
      disabled={loading}
      className="rounded bg-black px-4 py-2 text-white disabled:opacity-50"
    >
      {loading ? "Redirecting..." : "Upgrade"}
    </button>
  );
}
```

完成标准：

- 点击按钮跳转到 Stripe Checkout test 页面。
- Stripe Dashboard 中能看到 checkout session。

### 6. 接收 Stripe webhook 并更新订阅状态

Webhook 是支付系统的事实来源，不要只依赖 checkout success URL。

`app/api/stripe/webhook/route.ts`：

```ts
import { headers } from "next/headers";
import { NextResponse } from "next/server";
import Stripe from "stripe";
import { prisma } from "@/lib/db";
import { stripe } from "@/lib/stripe";

export const runtime = "nodejs";

async function syncSubscription(subscription: Stripe.Subscription) {
  const customerId =
    typeof subscription.customer === "string"
      ? subscription.customer
      : subscription.customer.id;

  const currentPeriodEnd = new Date(subscription.current_period_end * 1000);

  await prisma.user.updateMany({
    where: { stripeCustomerId: customerId },
    data: {
      subscriptionStatus: subscription.status,
      subscriptionCurrentEnd: currentPeriodEnd,
    },
  });
}

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get("stripe-signature");

  if (!signature || !process.env.STRIPE_WEBHOOK_SECRET) {
    return NextResponse.json({ error: "Missing Stripe signature" }, { status: 400 });
  }

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET,
    );
  } catch (error) {
    const message = error instanceof Error ? error.message : "Invalid webhook";
    return NextResponse.json({ error: message }, { status: 400 });
  }

  switch (event.type) {
    case "customer.subscription.created":
    case "customer.subscription.updated":
    case "customer.subscription.deleted":
      await syncSubscription(event.data.object as Stripe.Subscription);
      break;
    default:
      break;
  }

  return NextResponse.json({ received: true });
}
```

本地测试：

```bash
stripe login
stripe listen --forward-to localhost:3000/api/stripe/webhook
```

把 CLI 输出的 `whsec_...` 填入 `.env`。

完成标准：

- Stripe CLI 转发 webhook 成功。
- 用户的 `subscriptionStatus` 从 `free` 变成 `active` 或对应状态。
- 取消订阅后状态会更新为 `canceled`。

### 7. 实现 gated feature

`lib/entitlement.ts`：

```ts
import type { User } from "@prisma/client";

const PAID_STATUSES = new Set(["active", "trialing"]);

export function canUsePaidFeature(user: User) {
  if (!PAID_STATUSES.has(user.subscriptionStatus)) {
    return false;
  }

  if (!user.subscriptionCurrentEnd) {
    return true;
  }

  return user.subscriptionCurrentEnd.getTime() > Date.now();
}
```

`app/api/generate/route.ts`：

```ts
import { NextResponse } from "next/server";
import { z } from "zod";
import { getCurrentUser } from "@/lib/current-user";
import { prisma } from "@/lib/db";
import { canUsePaidFeature } from "@/lib/entitlement";

const schema = z.object({
  input: z.string().min(20).max(5000),
});

async function generateProposal(input: string) {
  return [
    "## Proposal Draft",
    "",
    "### Client context",
    input.slice(0, 300),
    "",
    "### Suggested scope",
    "- Discovery call",
    "- Milestone-based delivery",
    "- Weekly progress update",
    "",
    "### Risks",
    "- Clarify data access before implementation",
    "- Confirm acceptance criteria before kickoff",
  ].join("\n");
}

export async function POST(req: Request) {
  const user = await getCurrentUser();

  if (!canUsePaidFeature(user)) {
    return NextResponse.json(
      { error: "Upgrade required", code: "UPGRADE_REQUIRED" },
      { status: 402 },
    );
  }

  const json = await req.json();
  const parsed = schema.safeParse(json);

  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 });
  }

  const output = await generateProposal(parsed.data.input);

  const generation = await prisma.generation.create({
    data: {
      userId: user.id,
      input: parsed.data.input,
      output,
    },
  });

  return NextResponse.json({ generation });
}
```

完成标准：

- 免费用户请求 `/api/generate` 得到 `402 UPGRADE_REQUIRED`。
- 付费用户得到生成结果。
- 生成记录保存到数据库。

接入真实 LLM 时，把 `generateProposal` 替换成模型调用，并记录 token、latency、model、cost。LLM 成本开始增长后，按 [成本/延迟优化](../ai-engineering/08-llm-cost-latency-optimization.md) 做：

- prompt 压缩。
- 输入去重和缓存。
- 小模型先行，大模型兜底。
- 异步队列处理慢任务。
- 按用户套餐限制月度额度。

### 8. 加一个最小产品页

`app/app/page.tsx`：

```tsx
import { getCurrentUser } from "@/lib/current-user";
import { canUsePaidFeature } from "@/lib/entitlement";
import { UpgradeButton } from "@/components/upgrade-button";

export default async function AppPage() {
  const user = await getCurrentUser();
  const paid = canUsePaidFeature(user);

  return (
    <main className="mx-auto max-w-3xl space-y-6 p-8">
      <div>
        <p className="text-sm text-gray-500">Logged in as {user.email}</p>
        <h1 className="text-3xl font-bold">Proposal Generator</h1>
        <p className="text-gray-600">
          Paste a client brief and generate a structured proposal draft.
        </p>
      </div>

      {!paid ? (
        <section className="rounded border p-6">
          <h2 className="text-xl font-semibold">Upgrade to generate proposals</h2>
          <p className="mt-2 text-gray-600">
            Paid users can create proposal drafts and save generation history.
          </p>
          <div className="mt-4">
            <UpgradeButton />
          </div>
        </section>
      ) : (
        <section className="rounded border p-6">
          <h2 className="text-xl font-semibold">You are on a paid plan</h2>
          <p className="mt-2 text-gray-600">
            Connect this page to a client component that posts to /api/generate.
          </p>
        </section>
      )}
    </main>
  );
}
```

完成标准：

- 免费用户看到 Upgrade。
- 付费用户看到可用状态。
- 页面文案能被截图用于 launch。

### 9. 上线前的安全和合规最小清单

涉及支付、合同、税务、跨境收款和用户数据时，再次提醒：本文仅供教育与参考，非专业法律、税务、投资建议，请结合自身情况咨询专业人士。

- [ ] `.env` 未提交。
- [ ] Webhook 使用 `constructEvent` 校验签名。
- [ ] 付费权限只在服务端判断，不只在前端隐藏按钮。
- [ ] 记录 Stripe customer id 和 subscription status。
- [ ] 提供 Terms、Privacy、Refund policy 页面。
- [ ] 明确 AI 输出可能出错，用户需自行审核。
- [ ] 日志不打印用户敏感输入和 secret。
- [ ] 删除账户或删除数据有人工流程或自助入口。
- [ ] 生产环境使用 Stripe live key 和 live webhook secret。

## 第 4 阶段：定价与变现模型

定价不是“我觉得多少钱合适”，而是对用户价值、替代成本、获客成本和支持成本的假设。

### 常见收费模型

| 模型 | 适合产品 | 优点 | 风险 |
|---|---|---|---|
| Subscription | 持续使用的 SaaS、监控、生成额度 | MRR 可预测 | 用户会持续比较价值 |
| One-time | 模板、插件、小工具、lifetime deal | 购买决策简单 | 收入不稳定，后续支持成本高 |
| Usage-based | API、LLM 调用、批处理 | 成本和收入更匹配 | 用户账单不确定 |
| Seat-based | 团队协作工具 | 随团队扩张增长 | 早期销售摩擦更大 |
| Service + product | AI 自动化 + 顾问交付 | 早期现金流强 | 不是真正纯软件，需要交付 |

早期建议：

- B2C/Prosumer：$9–29/mo。
- Solo professional：$19–79/mo。
- Small team：$49–199/mo。
- API/usage：成本上加 3–10 倍毛利，再设最低消费。

如果你还不确定定价，从 [变现地图](01-monetization-landscape.md) 选择一个主模型，不要同时做订阅、咨询、课程、广告四种收入。

### Free trial vs Freemium

| 策略 | 适合 | 设置方式 |
|---|---|---|
| Free trial | 用户能在 7–14 天内感知价值 | 需要信用卡或无需信用卡二选一 |
| Freemium | 边际成本低、传播性强 | 免费额度严格限制核心成本 |
| No free plan | B2B 强痛点、交付价值明确 | Demo + money-back guarantee |

AI 产品要谨慎 freemium，因为每次试用都可能产生推理成本。

可用限制：

- 每月 5 次免费生成。
- 输出带水印或不能导出。
- 只支持短文本，不支持批量。
- 高级模型只给付费用户。

### Stripe 从 test 到 live 的步骤

1. 在 Stripe Dashboard 创建 Product。
2. 创建 Price：月付、年付或一次性。
3. 复制 `price_...` 到 `STRIPE_PRICE_ID`。
4. 使用 test key 本地跑通 Checkout。
5. 使用 Stripe CLI 跑通 webhook。
6. 在 Vercel 设置生产环境变量。
7. 在 Stripe Dashboard 添加 live webhook endpoint。
8. 切换 live key，做一笔小额真实支付。
9. 确认发票、邮件、订阅状态、取消流程。
10. 把退款和客服邮箱写进页面。

Live 前检查：

- [ ] test key 和 live key 没混用。
- [ ] webhook endpoint 指向生产域名。
- [ ] 成功和取消 URL 是生产域名。
- [ ] Customer Portal 可让用户自助取消或改卡。
- [ ] 账单税务设置已按所在地要求处理。

### 单位经济模型：一个可计算例子

假设你做的是面向独立咨询顾问的 proposal generator。

| 指标 | 数值 | 说明 |
|---|---:|---|
| 月订阅价格 | $29 | 单用户 plan |
| 付费用户 | 50 | launch 后第 3 个月 |
| 月流失率 | 5% | 每月取消比例 |
| 毛利率 | 85% | 扣除 LLM、数据库、支付费 |
| 每月内容/外联成本 | $300 | 工具、赞助、时间折算不计 |
| 新增付费 CAC | $20 | 平均每个付费用户获客成本 |

MRR：

$$
MRR = 付费用户数 \times 月价格 = 50 \times 29 = 1450
$$

月毛利：

$$
Gross\ Profit = MRR \times 毛利率 = 1450 \times 0.85 = 1232.5
$$

简化 LTV：

$$
LTV = \frac{月价格 \times 毛利率}{月流失率}
     = \frac{29 \times 0.85}{0.05}
     = 493
$$

LTV/CAC：

$$
\frac{LTV}{CAC} = \frac{493}{20} = 24.65
$$

这组数字表示：如果 churn 和 CAC 真实可靠，产品值得继续加码分发。但早期样本小，不能因为 3 个用户没取消就假设 churn 很低。

### 定价决策表

| 观察结果 | 解释 | 调整 |
|---|---|---|
| 很多人注册，没人升级 | 免费层太慷慨或价值不清 | 缩免费额度，强化付费结果 |
| 用户嫌贵但继续用 | 价格可能合理 | 不急降价，增加年付 |
| 用户说贵并流失 | 价值没覆盖价格 | 找高价值人群或改功能 |
| LLM 成本吃掉毛利 | 用量和价格错配 | 加 usage cap 或 usage-based |
| 支持成本很高 | 产品不够自解释 | 改 onboarding、FAQ、模板 |

## 第 5 阶段：Launch 与分发

一个产品不是“上线”后自动被看见；上线只是你开始分发的理由。

### 发行资产包

Launch 前准备这些素材：

- 一句话定位。
- 3 张截图：输入、输出、支付/结果页。
- 30–60 秒 demo 视频。
- 5 条用户痛点原话。
- 3 个使用案例。
- 1 篇长文：你为什么做、适合谁、不适合谁。
- 10 条短帖：功能点、前后对比、构建过程、指标。
- 1 个 FAQ 页面。
- 1 个退款承诺。

### SEO / 内容路径

SEO 适合有明确搜索意图的工具。

关键词分层：

| 类型 | 示例 | 页面 |
|---|---|---|
| 问题词 | how to write consulting proposal | 指南文章 |
| 替代词 | better alternative to proposal software | 对比页 |
| 模板词 | consulting proposal template | 免费模板页 |
| 工具词 | AI proposal generator | 产品页 |
| 行业词 | proposal for software consulting | 垂直 landing page |

内容结构：

```text
/blog/how-to-write-consulting-proposal
/templates/software-consulting-proposal
/alternatives/expensive-proposal-tool
/use-cases/freelance-developer-proposal
```

每篇内容都要有一个可执行 CTA：

- 下载模板。
- 生成一份样例。
- 加入 waitlist。
- 预约 15 分钟 onboarding。

内容长期复利很强，和 [内容与个人品牌](05-content-personal-brand.md) 的方法一致：先用真实案例建立信任，再把流量导向产品。

### Product Hunt 路径

Product Hunt 不适合所有产品，但适合设计完整、英文市场、可演示的小工具。

准备：

- 注册并提前 2–4 周参与社区。
- 找 20–50 个真实支持者，不要买票。
- 准备 maker comment：问题、解决方案、适合谁、优惠。
- 准备首日回复节奏。
- 提前确认时区和发布时间。

Maker comment 模板：

```text
Hey Product Hunt 👋

I built [Product] for [specific audience] who struggle with [pain].

It helps you:
1. [Outcome 1]
2. [Outcome 2]
3. [Outcome 3]

Why I built it:
[Short origin story with a real pain signal]

For PH users, I prepared [offer].
I’d love feedback on [specific question].
```

### 社区分发路径

社区不是广告栏。先贡献，再展示。

操作顺序：

1. 列出 20 个相关讨论帖。
2. 回复具体解决方案，不贴链接。
3. 当有人继续追问时，再给 demo 或资源。
4. 把重复问题整理成文章。
5. 文章末尾放产品入口。

评论模板：

```text
我最近也在处理这个问题。一个可行流程是：
1. 先把 [输入] 标准化；
2. 用 [规则/工具] 生成初稿；
3. 人工只审 [高风险部分]。

如果你愿意，我可以把我用的 checklist 发你。
```

### Cold outreach 路径

冷启动不是群发垃圾邮件，而是高度相关的一对一邀约。

名单来源：

- 最近发帖抱怨该问题的人。
- 使用竞品但公开表达不满的人。
- 你的二度人脉。
- GitHub/LinkedIn 上符合画像的人。
- 行业 newsletter 作者、社区管理员。

邮件模板：

```text
Subject: quick question about [specific workflow]

Hi [Name],

I saw your post about [specific pain].
I’m building a small tool that helps [audience] turn [input] into [outcome] in [time].

Not asking you to buy anything.
Could I ask 2 questions?
1. How do you handle [workflow] today?
2. What would make it worth paying for?

If useful, I can send a private demo when it is ready.

Best,
[Your name]
```

指标：

- 100 封高度相关外联。
- 30% 打开率以上。
- 5–15% 回复率。
- 3–10 次访谈。
- 1–3 个试用或付款。

### Programmatic distribution

如果你的产品能覆盖很多长尾场景，可以程序化生成页面。

示例：

- `/templates/{industry}-proposal-template`
- `/tools/{role}-email-generator`
- `/compare/{competitor}-alternative-for-{persona}`

注意：

- 每页必须有独特内容和真实价值，不能生成垃圾页。
- 用用户案例、行业字段、样例输出增加差异。
- 先手工写 10 页验证，再自动化扩展到 100 页。

### Launch checklist

- [ ] Landing page 有清晰定位和 CTA。
- [ ] 支付流程 test 和 live 都已验证。
- [ ] 付费权限服务端生效。
- [ ] 有 demo 视频或 GIF。
- [ ] 有 3 个使用案例。
- [ ] 有退款和隐私说明。
- [ ] 有 analytics：visit、signup、checkout、paid、activation。
- [ ] 准备 10 条短内容。
- [ ] 准备 50 个外联对象。
- [ ] 准备 3 个社区发布版本，按社区语气改写。
- [ ] 准备首批用户 onboarding 邮件。
- [ ] 准备问题反馈入口。

## 第 6 阶段：指标与迭代闭环

早期不要追 DAU、总访问量、点赞数；追能指导决策的漏斗。

### 最小指标面板

| 指标 | 定义 | 早期目标 |
|---|---|---|
| Target visits | 目标用户访问数 | 每周 100+ |
| Signup rate | 注册 / 目标访问 | 5–20% |
| Activation rate | 完成核心动作 / 注册 | 30–60% |
| Checkout start | 发起支付 / 激活 | 5–20% |
| Paid conversion | 付费 / 注册 | 1–10% |
| Week-1 retention | 7 天后仍使用 | 20–40% |
| Churn | 月取消 / 付费用户 | 早期样本小，逐月观察 |

核心事件：

```text
visit_landing
signup
complete_onboarding
generate_started
generate_succeeded
checkout_started
subscription_active
subscription_cancelled
```

每周复盘问题：

1. 哪个渠道带来最多目标用户？
2. 哪个页面让用户点击 CTA？
3. 用户在哪一步卡住？
4. 付费用户使用了什么功能？
5. 取消用户的原话是什么？
6. 下周只改一个最大瓶颈，应该改什么？

### Kill vs Double down

Kill 不是失败，而是保住注意力。

Kill 标准：

- 2 周内触达 100 个目标用户，访谈或 waitlist 反应极弱。
- 3 次改变定位后，仍没人愿意付费或投入时间。
- 用户痛点真实，但你没有触达渠道。
- 产品需要长期合规、数据、合作才能有价值，超出个人资源。
- 你对目标人群没有持续兴趣。

Double down 标准：

- 陌生用户愿意付款。
- 用户主动追问上线时间。
- 用户把产品推荐给同事或朋友。
- 付费用户每周重复使用。
- 你能找到可重复的获客动作。

Pivot 方式：

| 现象 | Pivot |
|---|---|
| 人群不买 | 保留问题，换更有预算的人群 |
| 功能不用 | 保留人群，换更痛的 workflow |
| 获客太难 | 保留产品，换渠道或内容角度 |
| 成本太高 | 保留价值，改套餐和额度 |
| 竞争太强 | 切更窄垂直场景 |

## 6–8 周：从 0 到第一位付费用户

下面计划适合兼职推进。每周结束必须有可检查 deliverable。

### Week 1：痛点池和人群选择

目标：找到 3 个高分候选。

任务：

- 写 20 条痛点池。
- 每条补齐人群、场景、替代方案、触达渠道。
- 用评分矩阵筛选前 3。
- 为每个候选写 1 句定位和 1 个假想价格。

Deliverable：

- `ideas.csv` 或 Notion 表。
- 3 个候选产品 brief。
- 50 个潜在用户或社区链接。

Go 标准：

- 至少 1 个候选总分 ≥ 22。
- 你知道下周要联系谁。

### Week 2：验证和访谈

目标：证明有真实痛点。

任务：

- 做 10 次访谈邀约，完成至少 5 次。
- 搜集 10 条社区/竞品痛点证据。
- 建一个 landing page。
- 发布 3 条 build-in-public 内容。

Deliverable：

- 访谈记录。
- Landing page URL。
- Waitlist 表单。
- 痛点原话库。

Go 标准：

- 100 个目标曝光中 ≥ 10 个 waitlist；或
- 3 个高质量访谈明确表示愿意试用。

### Week 3：预售或强承诺

目标：验证付费意愿。

任务：

- 创建 Stripe Payment Link 或手动 invoice。
- 给访谈用户发送 early access 邀请。
- 做一个 60 秒假门 demo。
- 继续 20 次外联。

Deliverable：

- 预售链接。
- Demo 视频。
- 价格页草稿。
- 反对意见列表。

Go 标准：

- 至少 1 个付款；或
- 3 个明确价格承诺和 beta 预约。

### Week 4：MVP 核心功能

目标：完成一个可用的端到端流程。

任务：

- 创建 Next.js 项目。
- 接入数据库和 Auth。
- 实现核心生成/处理逻辑。
- 保存用户输入和输出。
- 添加基础错误处理。

Deliverable：

- 可部署 demo。
- 1 条 happy path 视频。
- 核心功能通过手动测试。

完成标准：

- 用户能登录、提交输入、获得输出。
- 你能解释每次生成的成本和延迟。

### Week 5：支付、权限、上线

目标：让用户可以真实付款并使用。

任务：

- 接入 Stripe Checkout。
- 接入 webhook。
- 实现 gated feature。
- 增加 Terms、Privacy、Refund policy。
- 部署到生产环境。

Deliverable：

- 生产 URL。
- Stripe live 流程截图。
- 付费用户 onboarding 邮件。

完成标准：

- 一笔真实小额支付能解锁功能。
- 取消订阅会关闭权限。

### Week 6：公开发布和一对一分发

目标：拿到第一批真实用户。

任务：

- 发布长文和 10 条短帖。
- 做 50 次一对一外联。
- 在 3 个社区用不同角度发布。
- 给 waitlist 发 beta 邀请。
- 安排 5 次 onboarding call。

Deliverable：

- Launch post。
- 外联表。
- 用户反馈列表。
- 漏斗数据。

Go 标准：

- 至少 1 个付费用户；或
- 5 个活跃 beta 用户每周重复使用。

### Week 7：激活和留存

目标：让已来的人成功一次。

任务：

- 找到用户第一次使用的最大阻塞。
- 改 onboarding。
- 增加样例数据、模板、空状态提示。
- 给未激活用户发 1 封帮助邮件。
- 访谈取消或沉默用户。

Deliverable：

- 激活率对比。
- 3 个产品改动。
- 5 条用户原话。

完成标准：

- 新注册用户更快完成核心动作。
- 至少 1 个用户愿意提供 testimonial。

### Week 8：决策：kill、pivot 或 double down

目标：做出理性决策。

任务：

- 汇总 8 周投入、收入、用户、渠道数据。
- 计算 MRR、付费转化、激活、留存、毛利。
- 写一页复盘。
- 决定下一轮 4 周实验。

Deliverable：

- 8 周复盘文档。
- 下一轮 backlog。
- Kill/pivot/double down 决策。

决策标准：

- 有付费、有重复使用、有渠道：double down。
- 有痛点、无支付：改人群或定价。
- 有流量、无激活：改 onboarding 或核心价值。
- 无流量、无支付、无热情：kill。

## 常见坑与排查表

| 症状 | 可能原因 | 解决办法 |
|---|---|---|
| 做了 4 周还没上线 | MVP 范围太大 | 砍到 1 个输入、1 个输出、1 个付费墙 |
| 很多点赞没人注册 | 内容受众不是买家 | 改渠道，直接触达目标用户 |
| 用户注册后不用 | 首次价值太慢 | 提供样例输入、一键 demo、模板 |
| Stripe success 后没解锁 | 依赖前端回调 | 用 webhook 更新订阅状态 |
| LLM 成本高 | 免费额度过大或 prompt 太长 | 加额度、缓存、模型分层 |
| 用户说输出不准 | 缺少上下文或评估 | 增加输入结构、样例、人工可编辑 |
| 取消率高 | 持续价值不足 | 增加周期性任务、提醒、历史复用 |
| 害怕公开发布 | 把发布当成考试 | 先发过程、问题、学习，不必一次完美 |
| 不知道定价 | 没有替代成本参照 | 问用户当前花多少时间和钱 |
| 一直加功能 | 没有指标约束 | 每周只改最大漏斗瓶颈 |

## 可直接复制的模板

### 产品 brief

```text
产品名：
目标用户：
高频场景：
当前替代方案：
核心痛点：
承诺结果：
MVP 输入：
MVP 输出：
预计价格：
主要渠道：
Go 标准：
Kill 标准：
```

### 用户反馈记录

```text
用户：
角色：
来源：
当前 workflow：
最痛步骤：
原话：
愿意支付：
反对意见：
下一步：
```

### 每周复盘

```text
本周目标：
本周完成：
关键指标：
- visits:
- signups:
- activations:
- checkout starts:
- paid:
- revenue:

学到的事实：
最大瓶颈：
下周唯一重点：
需要 kill/pivot/double down 吗：
```

### Onboarding 邮件

```text
Subject: Welcome to [Product] — here is the fastest way to get value

Hi [Name],

Thanks for trying [Product].

Fastest path:
1. Open [URL]
2. Paste [input]
3. Click [action]
4. Review [output]

If you send me one example of [workflow], I can help you set up the first run.

Question: what would make this worth paying for every month?

Best,
[Your name]
```

## 掌握检查清单

- [ ] 我能用一句话说清目标用户、任务、结果、价格。
- [ ] 我有 20 条痛点池，不是 20 个功能点。
- [ ] 我完成了至少 5 次真实用户访谈。
- [ ] 我在构建前做了 landing page 或 pre-sale。
- [ ] 我知道 3 个目标用户聚集渠道。
- [ ] 我的 MVP 只有一个核心 job-to-be-done。
- [ ] 我跑通了 Stripe test mode。
- [ ] 我的付费权限在服务端判断。
- [ ] 我能计算 MRR、毛利、LTV、CAC。
- [ ] 我有 launch 素材包和 50 个外联对象。
- [ ] 我每周根据漏斗数据做一次取舍。
- [ ] 我有明确 kill/pivot/double down 标准。

## 延伸阅读

- Stripe Checkout 文档：https://docs.stripe.com/checkout
- Stripe Webhooks 文档：https://docs.stripe.com/webhooks
- Next.js App Router 文档：https://nextjs.org/docs/app
- Prisma 文档：https://www.prisma.io/docs
- Product Hunt Launch Guide：https://www.producthunt.com/launch
- Indie Hackers：https://www.indiehackers.com/
- The Mom Test：https://momtestbook.com/
- Lenny's Newsletter：https://www.lennysnewsletter.com/

标签 `Indie Hacker` `独立开发` `AI 产品` `SaaS` `变现`

---

[← 上一章](02-freelancing-remote-contracting.md) · [WP-05 目录](README.md) · [下一章 →](04-consulting-expertise.md)
