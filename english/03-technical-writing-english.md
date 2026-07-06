[← 手册首页](../README.md) · [WP-03 英语能力](README.md)

---

# 工程师的英文技术写作与异步沟通

> 结论先行：在 Remote/海外团队，**写作是你最高杠杆的英语技能**——PR 描述、设计文档、issue、Slack、邮件占据日常沟通的大头，且异步、留痕、可被反复阅读。写得清楚，你的专业度和影响力就被看见。

相关阅读：[Remote / 海外求职策略](/knowledge/remote-overseas-job-strategy)、[英语口语流利度](/knowledge/english-speaking-fluency)、[系统设计面试框架](/knowledge/system-design-interview)。

## 1. 核心原则：BLUF（结论先行）

**Bottom Line Up Front**：第一句就给结论/请求，再展开细节。读者忙，别让他读到第三段才知道你要什么。

- ❌ "So I was looking into the issue and found several things..."
- ✅ "**We should roll back v2.3; it doubles p95 latency.** Details below."

## 2. 可扫读的结构

英文技术写作追求**可扫读（scannable）**：
- 短段落（2–4 行），一段一个观点。
- 多用**要点列表**和**小标题**。
- 关键信息**加粗**。
- 有取舍就上**表格**。
- 长文档开头放 **TL;DR / Summary**。

## 3. 常用文体模板

### 3.1 PR 描述
```text
## What
一句话说明这个 PR 做了什么。
## Why
背景 / 关联 issue / 动机。
## How
关键实现决策与取舍（reviewer 最关心）。
## Testing
怎么验证的；有无风险与回滚方案。
```

### 3.2 设计文档（RFC）
```text
# Title
## Summary (TL;DR)
## Problem & Goals / Non-goals
## Proposal
## Alternatives considered  ← 体现深度
## Trade-offs & Risks
## Rollout / Metrics
```
"Alternatives considered" 和 "Trade-offs" 是资深度的体现，别省。

### 3.3 请求帮助 / 提问（尊重对方时间）
```text
Context: 我在做 X，卡在 Y。
What I tried: A、B（都失败，附现象）。
Ask: 具体想请教的一个问题。
```

### 3.4 状态同步 / 邮件
先结论后细节，明确 **action item** 和 **owner/deadline**。

## 4. 让语气专业又礼貌

- 软化直接否定：`I'd push back on...`、`Have we considered...?`、`One concern is...`。
- 请求用问句而非命令：`Could you...?` 好过 `You should...`。
- 给反馈对事不对人：`This function could be simpler` 而非 `You wrote this badly`。
- 不确定就标注：`I might be missing context, but...`。

## 5. 常见中式英语坑

- 冗长客套开场（"Hope this email finds you well, I am writing to..."）→ 直接说事。
- 一句话塞太多从句 → 拆成短句。
- 滥用 "very / actually / basically" → 删掉更有力。
- 时态混乱、单复数/冠词漏 → 用工具（Grammarly 类）过一遍。
- "please kindly" 叠用、过度道歉 → 一次礼貌即可。

## 6. 怎么练

- 每个 PR/文档写完**自己读一遍**：结论在开头吗？能扫读吗？能再删 20% 吗？
- 收集团队里写得好的 doc/PR 当范本模仿。
- 用 AI 帮你润色并**对比学习**它改了什么（学模式，不是代写）。
- 坚持用英文写 commit/PR/issue，把日常工作变成练习。

## 小结

技术写作是可训练的工程技能：**BLUF 开头、可扫读结构、套用文体模板、专业礼貌的语气、删掉中式冗余**。在异步团队里，清晰的写作就是你被看见、被信任、产生影响力的方式。


`标签` `英语` `技术写作` `异步沟通` `Remote` `PR` `文档`

---

[← 上一章](02-english-speaking-fluency.md) · [WP-03 目录](README.md)
