[← 手册首页](../README.md) · [WP-04 个人投资](README.md)

---

# FIRE 基础

FIRE（Financial Independence, Retire Early）的核心问题：**需要多少资产，被动收益就能覆盖开支？**

## 4% 法则

经验法则：若每年提取投资组合的 4%，在多数历史区间内本金可长期维持。
因此目标资产为年开支的 25 倍：

$$
\text{FIRE Number} = \frac{\text{年开支}}{0.04} = 25 \times \text{年开支}
$$

例如年开支 40 万，则 FIRE 目标为 $40 \times 25 = 1000$ 万。

## 复利增长

从当前资产 $P$ 出发，每年再投入 $C$，年化收益 $r$，$n$ 年后：

$$
FV = P(1+r)^n + C \cdot \frac{(1+r)^n - 1}{r}
$$

## 快速估算

```ts
function futureValue(P: number, C: number, r: number, n: number) {
  const g = Math.pow(1 + r, n);
  return P * g + C * ((g - 1) / r);
}
// 当前 100 万，每年投 30 万，年化 7%，20 年后：
futureValue(1_000_000, 300_000, 0.07, 20); // ≈ 1706 万
```

> AI LifeOS 的 **Investing** 模块（P7.7）会把这些公式做成交互式计算器。


`标签` `FIRE` `财务自由` `4%法则`

---

[WP-04 目录](README.md) · [下一章 →](02-asset-allocation-rebalancing.md)
