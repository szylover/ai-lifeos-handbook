[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 动态规划专题：从状态定义到优化

> DP 的难点不是「会不会」，而是**能不能稳定地把新问题拆成状态 + 转移**。本文把动态规划拆成可执行流程：识别问题、定义状态、写出转移、选择实现方式、验证复杂度，并用 Python 跑通经典题型。

## 0. 本章定位：把 DP 变成可复用的工程套路

动态规划（Dynamic Programming, DP）适合解决「大问题可以由小问题答案组合而来」的优化、计数、可行性与序列匹配问题。面试里，DP 不是背模板，而是快速建立一个**有向无环的状态图**：每个状态只依赖更小、更早或更短的状态。

### 前置知识与环境

- 会写 Python 函数、列表、字典、递归与循环。
- 理解复杂度 $O(n)$、$O(n^2)$、$O(nW)$、$O(2^n n)$。
- 能区分子数组（连续）、子序列（不一定连续）、子集（顺序通常不重要）。
- 本章代码只依赖 Python 标准库，建议使用 Python 3.10+。

```bash
python --version
python 06-dp-playground.py
```

你可以把每段 `python` 代码复制到本地单独运行；代码都带有 `assert` 或 `print`，能立即验证。

### 学完后你应该能做到

- 看到题目后用 60 秒判断它是否可能是 DP。
- 用一句话定义 `dp[...]` 的含义，并写出 base case。
- 在白板上写出状态转移方程，而不是只背代码。
- 在 top-down memoization 与 bottom-up tabulation 之间做选择。
- 对 0/1 背包、完全背包、零钱兑换、LIS、LCS、编辑距离、路径 DP、区间 DP、树 DP、状压 DP、数位 DP 给出可运行实现。
- 解释什么时候能滚动数组、什么时候不能随便降维。

## 1. 什么是 DP：缓存状态图上的重复子问题

一句话：DP = 把指数级递归中的重复子问题缓存起来，或按依赖顺序把每个子问题只算一次。

以斐波那契为最小例子：

$$
F(n)=F(n-1)+F(n-2),\quad F(0)=0,\quad F(1)=1
$$

朴素递归会重复计算 `fib(3)`、`fib(4)`；DP 把每个 `fib(i)` 存下来。

```python
from functools import lru_cache

@lru_cache(None)
def fib_top_down(n: int) -> int:
    if n < 2:
        return n
    return fib_top_down(n - 1) + fib_top_down(n - 2)


def fib_bottom_up(n: int) -> int:
    if n < 2:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev1 + prev2
    return prev1

assert fib_top_down(10) == 55
assert fib_bottom_up(10) == 55
print("fib ok")
```

### DP 与递归、贪心、搜索的区别

| 方法 | 核心动作 | 适用信号 | 失败信号 |
| --- | --- | --- | --- |
| DFS / 回溯 | 枚举所有选择 | 要求列出方案、搜索空间不大 | 子问题大量重复但不缓存 |
| 贪心 | 每步做局部最优 | 可证明局部最优推出全局最优 | 需要回看多个历史选择 |
| DP | 定义状态并复用子问题 | 最优值/计数/可行性依赖较小状态 | 状态定义不含足够信息 |
| BFS | 分层扩展 | 最短步数、无权图 | 状态数巨大且无剪枝 |

## 2. 如何识别 DP 问题

看到下面任意 3 个信号，就优先尝试 DP。

1. **求最优值**：最大金额、最小代价、最长长度、最少次数。
2. **求计数**：有多少种走法、多少种组合、多少个方案。
3. **求可行性**：能否凑出目标、能否分割、能否匹配。
4. **选择序列中若干元素**：选或不选、拿或不拿、买或卖。
5. **两个字符串/数组对齐**：相等、插入、删除、替换、公共子序列。
6. **网格路径**：只能从上/左/右下等固定方向转移。
7. **区间合并**：先算短区间，再算长区间。
8. **树上选择**：节点选不选影响子节点。
9. **子集状态**：访问过哪些点、使用过哪些人，用 bitmask 表示。
10. **数字逐位约束**：不超过 N，统计满足条件的整数。

### 快速判断流程

```text
题目要求是什么？
├─ 最优值 / 计数 / 可行性？不是 → 先考虑别的算法
├─ 能拆成前缀、容量、区间、节点、集合、数位？不能 → 先补状态信息
├─ 子问题会重复？是 → DP
└─ 每步局部最优可证明？是 → 也许贪心；否则 DP
```

## 3. 通用五步法：从题面到代码

原始提纲里的五步法是 DP 的主线，本章后面所有题型都按它展开。

1. **状态定义**：`dp[i]` / `dp[i][j]` 表示什么？必须能用自然语言精确描述。
2. **转移方程**：`dp[i]` 由哪些更小的状态推出？写成数学式。
3. **边界/初始化**：最小子问题的值，例如空字符串、容量 0、单个节点。
4. **计算顺序**：保证算 `dp[i]` 时依赖项已算好。
5. **答案位置**：最终答案在哪个状态，是 `dp[n]`、`max(dp)` 还是 `dp[0][n-1]`。

### 状态定义的 4 个常用维度

| 维度 | 例子 | 状态写法 |
| --- | --- | --- |
| 前缀长度 | 前 `i` 个物品、前 `i` 个字符 | `dp[i]`, `dp[i][j]` |
| 容量/金额 | 背包容量 `w`、金额 `a` | `dp[w]`, `dp[i][w]` |
| 区间 | 子数组 `[i, j]` | `dp[i][j]` |
| 选择状态 | 某节点选/不选、集合 mask | `dp[u][0/1]`, `dp[mask]` |

### 从暴力递归到 DP 的翻译法

先写一个「选择树」：每一步有哪些选择？递归参数是什么？如果同一组参数会重复出现，把这些参数变成 DP 状态。

```python
from functools import lru_cache

# 例：爬楼梯，每次走 1 或 2 阶，问到 n 阶有几种方法。
def climb_stairs(n: int) -> int:
    @lru_cache(None)
    def ways(i: int) -> int:
        if i == 0:
            return 1
        if i < 0:
            return 0
        return ways(i - 1) + ways(i - 2)

    return ways(n)

assert climb_stairs(2) == 2
assert climb_stairs(3) == 3
```

这里递归参数 `i` 就是状态：`dp[i]` 表示到达第 `i` 阶的方案数。

## 4. Top-down memoization vs Bottom-up tabulation

### Top-down：记忆化搜索

适合：

- 状态图不是简单线性顺序，例如树、区间、复杂约束。
- 只会访问部分状态。
- 面试时先保证正确，再优化成迭代。

```python
from functools import lru_cache


def min_cost_climb_top_down(cost: list[int]) -> int:
    n = len(cost)

    @lru_cache(None)
    def dp(i: int) -> int:
        if i <= 1:
            return 0
        return min(dp(i - 1) + cost[i - 1], dp(i - 2) + cost[i - 2])

    return dp(n)

assert min_cost_climb_top_down([10, 15, 20]) == 15
```

### Bottom-up：表格递推

适合：

- 依赖顺序明确。
- 需要严格控制空间。
- Python 递归深度可能爆掉。

```python

def min_cost_climb_bottom_up(cost: list[int]) -> int:
    n = len(cost)
    dp = [0] * (n + 1)
    for i in range(2, n + 1):
        dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])
    return dp[n]

assert min_cost_climb_bottom_up([10, 15, 20]) == 15
```

### 什么时候两者等价

记忆化 DFS 与自底向上 DP 是**等价**的：都在同一张状态图上求值。差异只在遍历顺序：

- top-down 从答案状态出发，递归访问依赖。
- bottom-up 从 base case 出发，按拓扑序填表。

## 5. 空间优化：只保留仍会被依赖的状态

空间优化不是机械删除数组，而是回答：计算当前状态时，还会用到哪些旧状态？

| 转移 | 原空间 | 可优化到 | 原因 |
| --- | --- | --- | --- |
| `dp[i]` 只依赖 `dp[i-1]`, `dp[i-2]` | $O(n)$ | $O(1)$ | 只需两个变量 |
| `dp[i][j]` 依赖上一行和当前行左侧 | $O(nm)$ | $O(m)$ | 滚动一行 |
| 0/1 背包 `dp[w]` 依赖上一轮 `dp[w-weight]` | $O(nW)$ | $O(W)$，容量倒序 | 防止同一物品重复使用 |
| 完全背包 `dp[w]` 依赖当前轮 `dp[w-weight]` | $O(nW)$ | $O(W)$，容量正序 | 允许同一物品重复使用 |

### 滚动变量示例：打家劫舍

$$
dp[i]=\max(dp[i-1],\ dp[i-2]+nums[i-1])
$$

只保留 `dp[i-2]` 与 `dp[i-1]`。

```python

def rob_linear(nums: list[int]) -> int:
    prev2 = 0
    prev1 = 0
    for x in nums:
        cur = max(prev1, prev2 + x)
        prev2, prev1 = prev1, cur
    return prev1

assert rob_linear([1, 2, 3, 1]) == 4
assert rob_linear([2, 7, 9, 3, 1]) == 12
```

## 6. 0/1 背包：每个物品最多选一次

### 问题模型

给定 `weights[i]`、`values[i]` 和容量 `capacity`，每个物品只能拿 0 或 1 次，求最大价值。

### 状态定义

`dp[i][w]` 表示只考虑前 `i` 个物品、容量不超过 `w` 时的最大价值。

### 转移方程

第 `i` 个物品下标为 `i-1`，重量 $wt_i$，价值 $val_i$。

$$
dp[i][w]=
\begin{cases}
dp[i-1][w], & w < wt_i \\
\max(dp[i-1][w],\ dp[i-1][w-wt_i]+val_i), & w \ge wt_i
\end{cases}
$$

边界：$dp[0][w]=0$，$dp[i][0]=0$。

### 可运行 Python：二维表

```python

def knapsack_01_2d(weights: list[int], values: list[int], capacity: int) -> int:
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        wt = weights[i - 1]
        val = values[i - 1]
        for w in range(capacity + 1):
            dp[i][w] = dp[i - 1][w]
            if w >= wt:
                dp[i][w] = max(dp[i][w], dp[i - 1][w - wt] + val)
    return dp[n][capacity]

assert knapsack_01_2d([2, 3, 4], [4, 5, 6], 5) == 9
```

### 空间优化：容量倒序

倒序是关键：`dp[w - wt]` 必须来自上一件物品处理完后的旧值，不能在同一轮重复使用当前物品。

```python

def knapsack_01(weights: list[int], values: list[int], capacity: int) -> int:
    dp = [0] * (capacity + 1)
    for wt, val in zip(weights, values):
        for w in range(capacity, wt - 1, -1):
            dp[w] = max(dp[w], dp[w - wt] + val)
    return dp[capacity]

assert knapsack_01([2, 3, 4], [4, 5, 6], 5) == 9
print("01 knapsack ok")
```

复杂度：二维版时间 $O(nW)$、空间 $O(nW)$；优化版时间 $O(nW)$、空间 $O(W)$。

## 7. Unbounded Knapsack：每个物品可选无限次

### 问题模型

每种物品可以重复选择。例如完全平方数、零钱兑换、切钢条。

### 状态定义

`dp[w]` 表示容量恰好或不超过 `w` 时能得到的最大价值。若题目要求「恰好装满」，不可达状态要初始化为 $-\infty$。

### 转移方程

$$
dp[w]=\max_{i: wt_i\le w}(dp[w-wt_i]+val_i)
$$

如果按物品枚举，完全背包的一维转移是：

$$
dp[w]=\max(dp[w],\ dp[w-wt_i]+val_i),\quad w=wt_i,wt_i+1,\dots,W
$$

容量正序表示当前物品可以被再次使用。

```python

def unbounded_knapsack(weights: list[int], values: list[int], capacity: int) -> int:
    dp = [0] * (capacity + 1)
    for wt, val in zip(weights, values):
        for w in range(wt, capacity + 1):
            dp[w] = max(dp[w], dp[w - wt] + val)
    return dp[capacity]

assert unbounded_knapsack([2, 3, 5], [4, 5, 10], 10) == 20
print("unbounded knapsack ok")
```

复杂度：时间 $O(nW)$，空间 $O(W)$。

## 8. Coin Change：零钱兑换与组合计数

零钱题有两类，状态相似但含义不同。

| 题型 | 目标 | 初始化 | 转移 |
| --- | --- | --- | --- |
| 最少硬币数（LC322） | min coins | `dp[0]=0`, 其余 $+\infty$ | `min(dp[a], dp[a-c]+1)` |
| 组合数（LC518） | count combinations | `dp[0]=1` | `dp[a] += dp[a-c]` |

### 8.1 最少硬币数

`dp[a]` 表示凑出金额 `a` 的最少硬币数。

$$
dp[a]=\min_{c\in coins,\ c\le a}(dp[a-c]+1),\quad dp[0]=0
$$

```python

def coin_change_min(coins: list[int], amount: int) -> int:
    inf = amount + 1
    dp = [inf] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return -1 if dp[amount] == inf else dp[amount]

assert coin_change_min([1, 2, 5], 11) == 3
assert coin_change_min([2], 3) == -1
```

复杂度：时间 $O(A\cdot C)$，空间 $O(A)$。

### 8.2 组合数：顺序不重要

`dp[a]` 表示使用已处理硬币凑出金额 `a` 的组合数。

$$
dp[a] \leftarrow dp[a] + dp[a-c]
$$

外层枚举硬币，内层金额正序，避免把 `[1,2]` 与 `[2,1]` 当成两种。

```python

def coin_change_combinations(coins: list[int], amount: int) -> int:
    dp = [0] * (amount + 1)
    dp[0] = 1
    for c in coins:
        for a in range(c, amount + 1):
            dp[a] += dp[a - c]
    return dp[amount]

assert coin_change_combinations([1, 2, 5], 5) == 4
print("coin change ok")
```

复杂度：时间 $O(A\cdot C)$，空间 $O(A)$。

## 9. LIS：最长递增子序列

最长递增子序列（LIS）有 DP 解和贪心 + 二分解。本章给 DP 版本，完整专题见下一章：[最长递增子序列专文](07-longest-increasing-subsequence.md)。

### 状态定义

`dp[i]` 表示**以 `nums[i]` 结尾**的最长严格递增子序列长度。

### 转移方程

$$
dp[i]=1+\max_{0\le j<i,\ nums[j]<nums[i]} dp[j]
$$

如果没有满足条件的 `j`，则 `dp[i]=1`。答案是 $\max_i dp[i]$，不是固定的 `dp[n-1]`。

```python

def length_of_lis_dp(nums: list[int]) -> int:
    if not nums:
        return 0
    n = len(nums)
    dp = [1] * n
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

assert length_of_lis_dp([10, 9, 2, 5, 3, 7, 101, 18]) == 4
assert length_of_lis_dp([7, 7, 7]) == 1
```

复杂度：时间 $O(n^2)$，空间 $O(n)$。二分优化版可到 $O(n\log n)$，见 [07-longest-increasing-subsequence.md](07-longest-increasing-subsequence.md)。

## 10. LCS：最长公共子序列

LCS（Longest Common Subsequence）用于两个字符串/数组的相似度、diff、版本对齐。注意：子序列不要求连续。

### 状态定义

`dp[i][j]` 表示 `text1` 的前 `i` 个字符与 `text2` 的前 `j` 个字符的 LCS 长度。

### 转移方程（显式 recurrence）

$$
dp[i][j]=
\begin{cases}
dp[i-1][j-1]+1, & text1[i-1]=text2[j-1] \\
\max(dp[i-1][j],\ dp[i][j-1]), & text1[i-1]\ne text2[j-1]
\end{cases}
$$

边界：$dp[0][j]=0$，$dp[i][0]=0$。一般 LCS 没有通用的 $O(n\log n)$ 解；只有当元素互异并可映射为排列时，才可能转成 LIS，参考 [最长递增子序列专文](07-longest-increasing-subsequence.md)。

```python

def longest_common_subsequence(text1: str, text2: str) -> int:
    n, m = len(text1), len(text2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[n][m]

assert longest_common_subsequence("abcde", "ace") == 3
assert longest_common_subsequence("abc", "def") == 0
```

### 空间优化版

`dp[i][j]` 依赖上一行同列、当前行左侧、上一行左上角。用一维数组时，`prev_diag` 保存上一行左上角。

```python

def lcs_optimized(text1: str, text2: str) -> int:
    if len(text2) > len(text1):
        text1, text2 = text2, text1
    m = len(text2)
    dp = [0] * (m + 1)
    for ch1 in text1:
        prev_diag = 0
        for j in range(1, m + 1):
            old = dp[j]
            if ch1 == text2[j - 1]:
                dp[j] = prev_diag + 1
            else:
                dp[j] = max(dp[j], dp[j - 1])
            prev_diag = old
    return dp[m]

assert lcs_optimized("abcde", "ace") == 3
print("lcs ok")
```

复杂度：二维版时间 $O(nm)$、空间 $O(nm)$；优化版时间 $O(nm)$、空间 $O(\min(n,m))$。

## 11. Edit Distance：编辑距离

编辑距离（Levenshtein Distance）衡量 `word1` 变成 `word2` 的最少插入、删除、替换次数。

### 状态定义

`dp[i][j]` 表示 `word1` 的前 `i` 个字符转换成 `word2` 的前 `j` 个字符的最少操作数。

### 转移方程（显式 recurrence）

如果最后一个字符相等，不需要新操作：

$$
dp[i][j]=dp[i-1][j-1],\quad word1[i-1]=word2[j-1]
$$

如果不相等，从三种操作取最小：

$$
dp[i][j]=1+\min\left(
\begin{array}{l}
dp[i-1][j] \quad \text{删除 word1[i-1]} \\
dp[i][j-1] \quad \text{插入 word2[j-1]} \\
dp[i-1][j-1] \quad \text{替换 word1[i-1]}
\end{array}
\right)
$$

边界：$dp[i][0]=i$，$dp[0][j]=j$。

```python

def min_distance(word1: str, word2: str) -> int:
    n, m = len(word1), len(word2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(n + 1):
        dp[i][0] = i
    for j in range(m + 1):
        dp[0][j] = j

    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(
                    dp[i - 1][j],      # delete
                    dp[i][j - 1],      # insert
                    dp[i - 1][j - 1],  # replace
                )
    return dp[n][m]

assert min_distance("horse", "ros") == 3
assert min_distance("intention", "execution") == 5
print("edit distance ok")
```

复杂度：时间 $O(nm)$，空间 $O(nm)$；可滚动数组到 $O(m)$。

## 12. Matrix Path：网格路径与最小路径和

### 问题模型

给定矩阵 `grid`，每次只能向右或向下走，从左上角到右下角，求路径最小和。

### 状态定义

`dp[i][j]` 表示从 `(0,0)` 走到 `(i,j)` 的最小路径和。

### 转移方程

$$
dp[i][j]=grid[i][j]+\min(dp[i-1][j],\ dp[i][j-1])
$$

边界：第一行只能从左侧来，第一列只能从上方来。

```python

def min_path_sum(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    dp = [[0] * cols for _ in range(rows)]
    dp[0][0] = grid[0][0]
    for i in range(1, rows):
        dp[i][0] = dp[i - 1][0] + grid[i][0]
    for j in range(1, cols):
        dp[0][j] = dp[0][j - 1] + grid[0][j]
    for i in range(1, rows):
        for j in range(1, cols):
            dp[i][j] = grid[i][j] + min(dp[i - 1][j], dp[i][j - 1])
    return dp[-1][-1]

assert min_path_sum([[1, 3, 1], [1, 5, 1], [4, 2, 1]]) == 7
```

### 原地优化

如果允许修改输入矩阵，可把 `grid` 当作 `dp` 表。

```python

def min_path_sum_inplace(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    for i in range(rows):
        for j in range(cols):
            if i == 0 and j == 0:
                continue
            up = grid[i - 1][j] if i > 0 else float("inf")
            left = grid[i][j - 1] if j > 0 else float("inf")
            grid[i][j] += min(up, left)
    return grid[-1][-1]

assert min_path_sum_inplace([[1, 3, 1], [1, 5, 1], [4, 2, 1]]) == 7
print("matrix path ok")
```

复杂度：时间 $O(RC)$；二维空间 $O(RC)$，原地版额外空间 $O(1)$。

## 13. House Robber：相邻不能同时选

### 状态定义

`dp[i]` 表示考虑前 `i` 间房时能偷到的最大金额。前 `i` 间对应数组下标 `0..i-1`。

### 转移方程

第 `i` 间房（下标 `i-1`）有两个选择：

$$
dp[i]=\max(dp[i-1],\ dp[i-2]+nums[i-1])
$$

边界：$dp[0]=0$，$dp[1]=nums[0]$。

```python

def house_robber(nums: list[int]) -> int:
    prev2 = 0
    prev1 = 0
    for money in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + money)
    return prev1

assert house_robber([2, 7, 9, 3, 1]) == 12
```

### 环形房屋

如果第一间与最后一间相邻，答案是「不偷第一间」与「不偷最后一间」两种线性问题的最大值。

```python

def house_robber_circle(nums: list[int]) -> int:
    def rob_line(arr: list[int]) -> int:
        prev2 = 0
        prev1 = 0
        for money in arr:
            prev2, prev1 = prev1, max(prev1, prev2 + money)
        return prev1

    if len(nums) == 1:
        return nums[0]
    return max(rob_line(nums[:-1]), rob_line(nums[1:]))

assert house_robber_circle([2, 3, 2]) == 3
assert house_robber_circle([1, 2, 3, 1]) == 4
print("house robber ok")
```

复杂度：时间 $O(n)$，空间 $O(1)$。

## 14. Partition / Subset Sum：子集和与等和分割

### 问题模型

给定正整数数组，判断能否选出若干数，使和等于 `target`。等和分割（LC416）就是判断总和是否为偶数，并把 `target` 设为 `sum(nums)//2`。

### 状态定义

`dp[s]` 表示使用已处理元素能否凑出和 `s`。

### 转移方程

每个数只能使用一次，因此容量倒序：

$$
dp[s]=dp[s]\lor dp[s-x],\quad s=target,target-1,\dots,x
$$

边界：$dp[0]=\text{true}$。

```python

def subset_sum(nums: list[int], target: int) -> bool:
    dp = [False] * (target + 1)
    dp[0] = True
    for x in nums:
        for s in range(target, x - 1, -1):
            dp[s] = dp[s] or dp[s - x]
    return dp[target]


def can_partition(nums: list[int]) -> bool:
    total = sum(nums)
    if total % 2 == 1:
        return False
    return subset_sum(nums, total // 2)

assert subset_sum([3, 34, 4, 12, 5, 2], 9) is True
assert can_partition([1, 5, 11, 5]) is True
assert can_partition([1, 2, 3, 5]) is False
print("subset sum ok")
```

复杂度：时间 $O(nS)$，空间 $O(S)$。

## 15. Interval DP：区间 DP

区间 DP 的特点：状态是一个闭区间或半开区间，转移枚举分割点或最后一个被选择的元素。计算顺序通常按区间长度从小到大。

### 例题：戳气球（LC312）

戳破气球 `k` 时得到 `nums[left] * nums[k] * nums[right]`。技巧是反过来想：在区间 `(left, right)` 内，枚举**最后一个**被戳破的气球 `k`。

### 状态定义

`dp[left][right]` 表示开区间 `(left, right)` 内所有气球被戳破能得到的最大硬币数，不包含 `left` 与 `right`。

### 转移方程

$$
dp[left][right]=\max_{left<k<right}\left(dp[left][k]+nums[left]\cdot nums[k]\cdot nums[right]+dp[k][right]\right)
$$

边界：当 `right-left<=1` 时区间为空，收益为 0。

```python

def max_coins(nums: list[int]) -> int:
    arr = [1] + nums + [1]
    n = len(arr)
    dp = [[0] * n for _ in range(n)]
    for length in range(2, n):
        for left in range(0, n - length):
            right = left + length
            best = 0
            for k in range(left + 1, right):
                coins = dp[left][k] + arr[left] * arr[k] * arr[right] + dp[k][right]
                best = max(best, coins)
            dp[left][right] = best
    return dp[0][n - 1]

assert max_coins([3, 1, 5, 8]) == 167
print("interval dp ok")
```

复杂度：时间 $O(n^3)$，空间 $O(n^2)$。

## 16. Tree DP：树上动态规划

树 DP 的状态通常绑定到节点，并且父节点状态会影响子节点选择。写法多为 DFS 后序遍历。

### 例题：树上打家劫舍（LC337）

每个节点有金额，不能同时偷父子节点。

### 状态定义

对每个节点 `u` 返回二元组：

- `take[u]`：偷 `u` 时，以 `u` 为根的子树最大金额。
- `skip[u]`：不偷 `u` 时，以 `u` 为根的子树最大金额。

### 转移方程

$$
take[u]=u.val+skip[left]+skip[right]
$$

$$
skip[u]=\max(take[left],skip[left])+\max(take[right],skip[right])
$$

```python
from dataclasses import dataclass

@dataclass
class TreeNode:
    val: int
    left: "TreeNode | None" = None
    right: "TreeNode | None" = None


def rob_tree(root: TreeNode | None) -> int:
    def dfs(node: TreeNode | None) -> tuple[int, int]:
        if node is None:
            return (0, 0)
        left_take, left_skip = dfs(node.left)
        right_take, right_skip = dfs(node.right)
        take = node.val + left_skip + right_skip
        skip = max(left_take, left_skip) + max(right_take, right_skip)
        return (take, skip)

    return max(dfs(root))

root = TreeNode(3, TreeNode(2, None, TreeNode(3)), TreeNode(3, None, TreeNode(1)))
assert rob_tree(root) == 7
print("tree dp ok")
```

复杂度：时间 $O(n)$，递归栈空间 $O(h)$。

## 17. Bitmask DP：用整数表示集合

当 `n <= 20` 且状态与「哪些元素已经使用」有关时，考虑 bitmask DP。第 `i` 位为 1 表示第 `i` 个元素已被选或访问。

### 例题：访问所有城市的最短 Hamilton 路径（小规模 TSP）

给定距离矩阵，从城市 0 出发访问所有城市一次，求最短路程。

### 状态定义

`dp[mask][i]` 表示已经访问集合 `mask`，并且当前停在城市 `i` 的最短距离。

### 转移方程

$$
dp[mask\cup\{j\}][j]=\min(dp[mask\cup\{j\}][j],\ dp[mask][i]+dist[i][j])
$$

边界：$dp[1][0]=0$。

```python

def tsp_shortest_path(dist: list[list[int]]) -> int:
    n = len(dist)
    inf = 10**18
    dp = [[inf] * n for _ in range(1 << n)]
    dp[1][0] = 0
    for mask in range(1 << n):
        for i in range(n):
            if dp[mask][i] == inf:
                continue
            for j in range(n):
                if mask & (1 << j):
                    continue
                new_mask = mask | (1 << j)
                dp[new_mask][j] = min(dp[new_mask][j], dp[mask][i] + dist[i][j])
    full = (1 << n) - 1
    return min(dp[full])

matrix = [
    [0, 10, 15, 20],
    [10, 0, 35, 25],
    [15, 35, 0, 30],
    [20, 25, 30, 0],
]
assert tsp_shortest_path(matrix) == 65
print("bitmask dp ok")
```

复杂度：时间 $O(2^n n^2)$，空间 $O(2^n n)$。

## 18. Digit DP：数位 DP（简版）

数位 DP 用来统计 `0..N` 中满足某种逐位约束的数字个数，例如「不含数字 4」「各位数字和等于 S」「数字不重复」。核心参数通常包含：当前位置、是否贴上界、前导零状态、已用数字集合、累计值。

### 例题：统计 `0..n` 中不含数字 4 的非负整数个数

### 状态定义

`dfs(pos, tight, started)` 表示正在处理第 `pos` 位：

- `tight=True`：当前前缀已经贴着上界，下一位不能超过 `n` 对应位。
- `started=False`：还在前导零阶段，此时可以继续跳过当前位。

### 转移方程

设当前可选上界为 $limit$，候选数字为 $d$：

$$
ans=\sum_{d=0}^{limit} dfs(pos+1,\ tight\land(d=limit),\ started\lor(d\ne0))
$$

但当 `started` 后选择 `d=4` 时，该分支非法。

```python
from functools import lru_cache


def count_without_digit_four(n: int) -> int:
    digits = list(map(int, str(n)))

    @lru_cache(None)
    def dfs(pos: int, tight: bool, started: bool) -> int:
        if pos == len(digits):
            return 1
        limit = digits[pos] if tight else 9
        total = 0
        for d in range(limit + 1):
            next_tight = tight and d == limit
            next_started = started or d != 0
            if next_started and d == 4:
                continue
            total += dfs(pos + 1, next_tight, next_started)
        return total

    return dfs(0, True, False)

assert count_without_digit_four(20) == 19  # 0..20 except 4 and 14
assert count_without_digit_four(4) == 4    # 0,1,2,3
print("digit dp ok")
```

复杂度：状态约 $O(位数\cdot2\cdot2)$，每个状态枚举 10 个数字。

## 19. 最小完整项目：一个 DP Playground

把下面代码保存为 `06-dp-playground.py`，运行后会验证本章核心函数。这个小项目的目的不是复用所有代码，而是建立「每写一个 DP 就配一组断言」的习惯。

```python
from functools import lru_cache


def coin_change_min(coins: list[int], amount: int) -> int:
    inf = amount + 1
    dp = [inf] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return -1 if dp[amount] == inf else dp[amount]


def lcs(a: str, b: str) -> int:
    dp = [[0] * (len(b) + 1) for _ in range(len(a) + 1)]
    for i in range(1, len(a) + 1):
        for j in range(1, len(b) + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[-1][-1]


def edit_distance(a: str, b: str) -> int:
    dp = [[0] * (len(b) + 1) for _ in range(len(a) + 1)]
    for i in range(len(a) + 1):
        dp[i][0] = i
    for j in range(len(b) + 1):
        dp[0][j] = j
    for i in range(1, len(a) + 1):
        for j in range(1, len(b) + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1])
    return dp[-1][-1]


def min_path_sum(grid: list[list[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    dp = [10**18] * cols
    dp[0] = 0
    for i in range(rows):
        for j in range(cols):
            if j == 0:
                dp[j] = dp[j] + grid[i][j]
            else:
                dp[j] = min(dp[j], dp[j - 1]) + grid[i][j]
    return dp[-1]


def run_tests() -> None:
    assert coin_change_min([1, 2, 5], 11) == 3
    assert lcs("abcde", "ace") == 3
    assert edit_distance("horse", "ros") == 3
    assert min_path_sum([[1, 3, 1], [1, 5, 1], [4, 2, 1]]) == 7
    print("all dp playground tests passed")


if __name__ == "__main__":
    run_tests()
```

预期输出：

```text
all dp playground tests passed
```

## 20. 常见坑与排查表

| 症状 | 高概率原因 | 解决办法 |
| --- | --- | --- |
| 0/1 背包结果偏大 | 容量正序导致同一物品被重复用 | 一维优化时从 `capacity` 倒序到 `weight` |
| 完全背包结果偏小 | 容量倒序阻止重复使用 | 完全背包用正序容量 |
| LCS 少 1 或越界 | `i/j` 与字符下标混用 | `dp` 用前缀长度，字符用 `i-1/j-1` |
| LIS 答案取了 `dp[-1]` | 最长递增子序列不一定以最后一个元素结尾 | 返回 `max(dp)` |
| 编辑距离插入/删除搞反 | 状态含义不清 | 固定解释：`dp[i][j]` 是 word1 前 i 到 word2 前 j |
| 区间 DP 依赖未算 | 枚举顺序错 | 外层按区间长度递增 |
| 树 DP 重复访问父节点 | 无向树没有传 parent | DFS 参数带 `parent` 或构建有根树 |
| 数位 DP 统计漏掉 0 | 前导零状态处理不一致 | 明确空数字是否代表 0，本章示例把 0 计入答案 |
| 递归爆栈 | Python 默认递归深度有限 | 改 bottom-up 或谨慎设置 `sys.setrecursionlimit` |

## 21. 面试写 DP 的 6 分钟模板

1. **第 0–1 分钟：复述题目目标**  
   “We need the minimum number of operations / maximum value / number of ways.”
2. **第 1–2 分钟：定义状态**  
   “Let `dp[i][j]` be ... for the first `i` elements and first `j` elements.”
3. **第 2–3 分钟：写 base case**  
   空数组、空字符串、容量 0、单节点先填。
4. **第 3–4 分钟：写 recurrence**  
   按「选/不选」「匹配/不匹配」「从上/左来」「枚举分割点」组织。
5. **第 4–5 分钟：确定循环顺序**  
   前缀 DP 从小到大，背包看是否允许重复，区间 DP 按长度。
6. **第 5–6 分钟：复杂度与优化**  
   先给正确表格，再说明是否能滚动数组。

可直接说的英文模板：

```text
I will define dp[i][j] as the answer for the prefix ending before i and j.
The transition depends on whether the current elements match.
The base cases are empty prefixes.
We fill the table row by row, so every dependency is already computed.
The time complexity is O(nm), and the space can be optimized to O(m).
```

## 22. Practice Problem Ladder：从易到难

### Easy：建立状态意识

- LC70 Climbing Stairs：`dp[i]=dp[i-1]+dp[i-2]`。
- LC198 House Robber：相邻不能同时选。
- LC746 Min Cost Climbing Stairs：从递归参数翻译状态。
- LC62 Unique Paths：二维路径计数。
- LC64 Minimum Path Sum：路径最优值。

### Medium：掌握主流模型

- LC322 Coin Change：完全背包最少硬币数。
- LC518 Coin Change II：组合数，外层硬币。
- LC416 Partition Equal Subset Sum：0/1 背包可行性。
- LC300 Longest Increasing Subsequence：先 $O(n^2)$ DP，再看 [07-longest-increasing-subsequence.md](07-longest-increasing-subsequence.md)。
- LC1143 Longest Common Subsequence：二维前缀 DP。
- LC72 Edit Distance：三操作转移。
- LC139 Word Break：前缀可行性。

### Hard：练区间、树、状态压缩

- LC312 Burst Balloons：区间 DP，枚举最后戳破。
- LC337 House Robber III：树 DP 二状态。
- LC10 Regular Expression Matching：二维匹配 DP。
- LC847 Shortest Path Visiting All Nodes：bitmask + BFS/DP。
- LC943 Find the Shortest Superstring：状压 DP。
- LC233 Number of Digit One：数位 DP。
- LC188 Best Time to Buy and Sell Stock IV：交易次数状态。

## 23. 14 天学习闭环

每天 60–90 分钟，按「写状态 → 写转移 → 跑测试 → 复盘」循环。

| 天数 | 任务 | 完成标准 |
| --- | --- | --- |
| Day 1 | 爬楼梯、最小花费爬楼梯 | 能写 top-down 与 bottom-up |
| Day 2 | 打家劫舍 I/II | 能解释滚动变量 |
| Day 3 | 最小路径和、不同路径 | 能处理第一行第一列边界 |
| Day 4 | Coin Change I/II | 能说明组合与排列循环顺序差异 |
| Day 5 | 0/1 背包、等和分割 | 能解释容量倒序 |
| Day 6 | LIS $O(n^2)$ | 能说清 `dp[i]` 为什么是「以 i 结尾」 |
| Day 7 | 复习 LIS 专章 | 跑通 [07-longest-increasing-subsequence.md](07-longest-increasing-subsequence.md) 的二分版本 |
| Day 8 | LCS | 能默写 recurrence |
| Day 9 | 编辑距离 | 能画出 3x3 表格并解释三操作 |
| Day 10 | Word Break | 能从前缀可行性建模 |
| Day 11 | 区间 DP：戳气球 | 能解释「最后一个」技巧 |
| Day 12 | 树 DP：House Robber III | 能返回 `(take, skip)` |
| Day 13 | Bitmask DP：TSP 小规模 | 能用位运算检查集合 |
| Day 14 | 模拟面试 2 题 | 每题 25 分钟内写出可运行代码 |

每题复盘记录 4 行即可：

```text
题目：LC72 Edit Distance
状态：dp[i][j] = word1[:i] -> word2[:j] 的最少操作
转移：match 走左上；不 match 取删/插/替最小 + 1
错误：初始化 dp[0][j] 漏了；下次先写空前缀边界
```

## 24. 掌握检查清单

- [ ] 我能用一句话解释 DP 与暴力递归的关系。
- [ ] 我能判断题目是在求最优值、计数还是可行性。
- [ ] 我能为前缀、容量、区间、树、集合分别定义状态。
- [ ] 我能写出 LCS 与编辑距离的完整 recurrence。
- [ ] 我知道 0/1 背包一维优化要倒序容量。
- [ ] 我知道完全背包一维优化要正序容量。
- [ ] 我能解释 LIS 的 `dp[i]` 为什么不是「前 i 个元素的答案」。
- [ ] 我能把 top-down DFS 改成 bottom-up 表格。
- [ ] 我能说明空间优化是否安全，而不是盲目滚动数组。
- [ ] 我能给每个 DP 函数写 2–3 个断言测试。

## 25. 延伸阅读

- LeetCode Dynamic Programming Explore Card：https://leetcode.com/explore/learn/card/dynamic-programming/
- CP-Algorithms: Introduction to Dynamic Programming：https://cp-algorithms.com/dynamic_programming/intro-to-dp.html
- CP-Algorithms: Knapsack Problem：https://cp-algorithms.com/dynamic_programming/knapsack.html
- MIT 6.006 Dynamic Programming Lectures：https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/
- CLRS Chapter 14 Dynamic Programming：https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/
- Topcoder DP Tutorial：https://www.topcoder.com/thrive/articles/dynamic-programming-from-novice-to-advanced
- 本手册下一章：[最长递增子序列专题](07-longest-increasing-subsequence.md)

`标签` `DP` `动态规划` `面试` `刷题`

---

[← 上一章](05-coding-interview-patterns.md) · [WP-02 目录](README.md) · [下一章 →](07-longest-increasing-subsequence.md)
