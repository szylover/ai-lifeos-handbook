[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 动态规划专题：从状态定义到优化

> DP 的难点不是「会不会」，而是**能不能稳定地把新问题拆成状态 + 转移**。本文给你一套可复用的思考流程。

## 通用五步法

1. **状态定义**：`dp[i]` / `dp[i][j]` 表示什么？（最关键的一步）
2. **转移方程**：`dp[i]` 由哪些更小的状态推出？
3. **边界/初始化**：最小子问题的值。
4. **计算顺序**：保证算 `dp[i]` 时依赖项已算好。
5. **答案位置**：最终答案在哪个状态。

## 一维 DP：打家劫舍（LC198）

- 状态：`dp[i]` = 前 i 间房能偷的最大金额。
- 转移：$dp[i] = \max(dp[i-1],\; dp[i-2] + nums[i])$
- 边界：`dp[0]=nums[0]`，`dp[1]=max(nums[0],nums[1])`。

```cpp
int rob(vector<int>& nums) {
    int prev2 = 0, prev1 = 0;
    for (int x : nums) {
        int cur = max(prev1, prev2 + x);
        prev2 = prev1; prev1 = cur;
    }
    return prev1;
}
```

## 背包模型：零钱兑换（LC322，完全背包）

- 状态：`dp[a]` = 凑出金额 a 的最少硬币数。
- 转移：$dp[a] = \min_{c \in coins}(dp[a-c] + 1)$
- 边界：`dp[0]=0`，其余初始化为 ∞。
- 注意：数组按 **amount**（≤10⁴）开，不是按硬币面额，不会爆。

```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (c <= a) dp[a] = min(dp[a], dp[a - c] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

## 子序列：LIS 与 LCS

- **最长递增子序列（LIS）**：贪心 + 二分做到 $O(n\log n)$，见 [专文](/knowledge/longest-increasing-subsequence)。
- **最长公共子序列（LCS，LC1143）**：二维 DP，$O(nm)$。
  - 状态：`dp[i][j]` = A 前 i、B 前 j 的 LCS 长度。
  - 转移：相等则 $dp[i][j]=dp[i-1][j-1]+1$，否则 $\max(dp[i-1][j], dp[i][j-1])$。
  - 说明：一般 LCS 没有通用的 $O(n\log n)$ 解；只有当元素互异可映射为排列时才能转成 LIS 求解。

## 编辑距离（LC72）

- 状态：`dp[i][j]` = word1 前 i 转成 word2 前 j 的最少操作数。
- 转移：若 `word1[i-1]==word2[j-1]`，$dp[i][j]=dp[i-1][j-1]$；
  否则 $dp[i][j] = 1 + \min(dp[i-1][j],\; dp[i][j-1],\; dp[i-1][j-1])$（删/插/替）。
- 边界：`dp[i][0]=i`，`dp[0][j]=j`。

## 区间 DP 与二维路径

- **区间 DP**（如戳气球、回文）：`dp[i][j]` 表示区间 `[i,j]`，按区间长度从小到大枚举。
- **路径 DP**（最小路径和 LC64）：`dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`。

## 优化技巧

| 技巧 | 场景 |
| --- | --- |
| 滚动数组 | 转移只依赖上一行/前两个状态 → 降维省空间 |
| 单调队列 | 滑动窗口最值型转移 → 降到 O(n) |
| 二分 | LIS 这类「维护有序结构」的转移 |
| 记忆化搜索 | 状态转移图复杂、顺序难定时，DFS + 缓存 |

## 高频题清单（建议顺序）

1. 打家劫舍 198 · 爬楼梯 70
2. 零钱兑换 322 · 完全平方数 279
3. 最长递增子序列 300 · 最长公共子序列 1143
4. 编辑距离 72 · 不同路径 62/63
5. 最长回文子串 5 · 最小路径和 64
6. 买卖股票系列 121/122/123/188
7. 戳气球 312 · 最长有效括号 32（进阶）

> 记忆化搜索与自底向上 DP 是**等价**的：想不清顺序时先写记忆化 DFS，再翻译成迭代。


`标签` `DP` `动态规划` `面试` `刷题`

---

[← 上一章](05-coding-interview-patterns.md) · [WP-02 目录](README.md) · [下一章 →](07-longest-increasing-subsequence.md)
