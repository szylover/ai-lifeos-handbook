[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 最长递增子序列（LIS）

本章把最长递增子序列（Longest Increasing Subsequence, LIS）从面试题扩展成一套可直接运行、可复盘、可迁移到变体题的解题框架。

如果你还没有系统复习动态规划，先读上一章：[动态规划实战指南](06-dynamic-programming-guide.md)。本章默认你能读懂数组、二分查找、状态转移和 Python 基础语法。

学完本章，你应该能独立完成：

- 写出 $O(n^2)$ DP 解法，并解释状态定义、转移方程、初始化和答案提取。
- 写出 $O(n \log n)$ 的 patience sorting / binary search 解法，并解释 `tails` 数组为什么正确。
- 从长度扩展到「恢复一条实际 LIS」。
- 区分严格递增、非递减、递减、二维 LIS、Russian Doll Envelopes、LIS 计数。
- 精确回答「LCS 是否有 $O(\log n)$ 或更快通用解法」：一般没有；特殊情况下可转成 LIS。

---

## 1. 问题定义：子序列不是子数组

给定一个整数数组 `nums`，求最长严格递增子序列的长度。

严格递增的含义：

$$
i_1 < i_2 < \cdots < i_k
\quad\text{且}\quad
nums[i_1] < nums[i_2] < \cdots < nums[i_k]
$$

注意两个关键词：

| 概念 | 是否要求连续 | 示例数组 `[10, 9, 2, 5, 3, 7, 101, 18]` |
|---|---:|---|
| 子数组（subarray） | 是 | `[2, 5, 3, 7]` |
| 子序列（subsequence） | 否，只要求保留原相对顺序 | `[2, 3, 7, 18]` |

本题答案为 `4`，因为可以取 `[2, 3, 7, 18]`。

你在面试中要先确认：

1. 是否严格递增：`1, 1, 2` 的 LIS 是 `2`，不是 `3`。
2. 是否只要长度：只要长度可用更短代码；要实际序列需要额外记录父指针。
3. 输入规模：`n <= 2500` 时 $O(n^2)$ 可接受；`n >= 10^5` 时通常需要 $O(n \log n)$。
4. 数值范围：LIS 不依赖值域大小，负数、重复值、大整数都可以处理。

---

## 2. 解法总览

| 任务 | 推荐解法 | 时间复杂度 | 空间复杂度 | 是否容易恢复序列 |
|---|---|---:|---:|---|
| 只求长度，小规模 | DP | $O(n^2)$ | $O(n)$ | 容易 |
| 只求长度，大规模 | `tails` + 二分 | $O(n \log n)$ | $O(n)$ | 需要额外索引 |
| 求一条 LIS | DP 父指针或 `tails` 父指针 | $O(n^2)$ / $O(n \log n)$ | $O(n)$ | 可以 |
| 求 LIS 个数 | DP 长度 + 计数 | $O(n^2)$ | $O(n)$ | 不适合 `tails` 直接做 |
| 二维嵌套 | 排序 + 一维 LIS | $O(n \log n)$ | $O(n)$ | 可以扩展 |

面试答题建议：

1. 先给 DP：状态清晰，能证明你理解「子序列」。
2. 再优化到 `tails`：体现复杂度意识。
3. 如果追问「返回路径」或「变体」，用本章后半部分模板。

---

## 3. $O(n^2)$ 动态规划：最稳的基础解法

### 3.1 状态定义

定义：

$$
dp[i] = \text{以 } nums[i] \text{ 结尾的最长严格递增子序列长度}
$$

为什么必须强调「以 `i` 结尾」？

因为只有知道结尾元素，才能判断前一个元素 `nums[j]` 能不能接到 `nums[i]` 前面。

### 3.2 状态转移

对每个 `i`，枚举所有 `j < i`：

$$
dp[i] =
1 + \max_{\substack{0 \le j < i \\ nums[j] < nums[i]}} dp[j]
$$

如果不存在满足条件的 `j`，则：

$$
dp[i] = 1
$$

最终答案不是 `dp[n-1]`，而是：

$$
\max_{0 \le i < n} dp[i]
$$

因为最长递增子序列不一定以最后一个元素结尾。

### 3.3 手工 trace

输入：

```text
nums = [10, 9, 2, 5, 3, 7, 101, 18]
```

逐步计算：

| i | nums[i] | 可接在谁后面 | dp[i] | 当前 dp |
|---:|---:|---|---:|---|
| 0 | 10 | 无 | 1 | `[1]` |
| 1 | 9 | 无 | 1 | `[1, 1]` |
| 2 | 2 | 无 | 1 | `[1, 1, 1]` |
| 3 | 5 | 2 | 2 | `[1, 1, 1, 2]` |
| 4 | 3 | 2 | 2 | `[1, 1, 1, 2, 2]` |
| 5 | 7 | 2, 5, 3 | 3 | `[1, 1, 1, 2, 2, 3]` |
| 6 | 101 | 10, 9, 2, 5, 3, 7 | 4 | `[1, 1, 1, 2, 2, 3, 4]` |
| 7 | 18 | 10, 9, 2, 5, 3, 7 | 4 | `[1, 1, 1, 2, 2, 3, 4, 4]` |

答案是 `4`。

### 3.4 可运行 Python：只求长度

保存为 `lis_dp_length.py` 或直接复制到 Python REPL 运行。

```python
from typing import List


def length_of_lis_dp(nums: List[int]) -> int:
    if not nums:
        return 0

    n = len(nums)
    dp = [1] * n

    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)


if __name__ == "__main__":
    cases = [
        ([10, 9, 2, 5, 3, 7, 101, 18], 4),
        ([0, 1, 0, 3, 2, 3], 4),
        ([7, 7, 7, 7, 7], 1),
        ([], 0),
        ([-1, 3, 4, -2, 0, 6, 2, 3], 4),
    ]

    for nums, expected in cases:
        actual = length_of_lis_dp(nums)
        print(nums, "=>", actual)
        assert actual == expected

    print("all dp length tests passed")
```

预期输出：

```text
[10, 9, 2, 5, 3, 7, 101, 18] => 4
[0, 1, 0, 3, 2, 3] => 4
[7, 7, 7, 7, 7] => 1
[] => 0
[-1, 3, 4, -2, 0, 6, 2, 3] => 4
all dp length tests passed
```

### 3.5 DP 如何恢复一条实际 LIS

增加 `prev[i]`，表示 `nums[i]` 在当前最优子序列中的前驱下标。

当我们用 `j` 更新 `i`：

```text
dp[i] = dp[j] + 1
```

同时记录：

```text
prev[i] = j
```

最后找到 `dp` 最大的下标 `end`，沿着 `prev` 反向回溯，再反转。

```python
from typing import List


def reconstruct_lis_dp(nums: List[int]) -> List[int]:
    if not nums:
        return []

    n = len(nums)
    dp = [1] * n
    prev = [-1] * n

    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i] and dp[j] + 1 > dp[i]:
                dp[i] = dp[j] + 1
                prev[i] = j

    end = max(range(n), key=lambda i: dp[i])
    seq = []
    while end != -1:
        seq.append(nums[end])
        end = prev[end]

    return seq[::-1]


if __name__ == "__main__":
    nums = [10, 9, 2, 5, 3, 7, 101, 18]
    seq = reconstruct_lis_dp(nums)
    print(seq)
    assert len(seq) == 4
    assert all(seq[i] < seq[i + 1] for i in range(len(seq) - 1))
```

可能输出：

```text
[2, 5, 7, 101]
```

输出 `[2, 3, 7, 18]` 也正确。LIS 可能不唯一。

---

## 4. $O(n \log n)$：patience sorting / `tails` + 二分

### 4.1 心智模型：保留每种长度的最小尾巴

维护数组 `tails`：

$$
tails[k] = \text{长度为 } k+1 \text{ 的递增子序列的最小可能尾元素}
$$

例子：

```text
tails = [2, 3, 7]
```

含义不是「当前 LIS 是 `[2, 3, 7]`」。它只表示：

- 存在长度为 1 的递增子序列，最小尾元素是 `2`。
- 存在长度为 2 的递增子序列，最小尾元素是 `3`。
- 存在长度为 3 的递增子序列，最小尾元素是 `7`。

尾元素越小，未来越容易接上更大的数。例如 `[2, 3]` 比 `[2, 5]` 更有扩展潜力。

### 4.2 更新规则

遍历每个 `x`：

1. 在 `tails` 中二分查找第一个 `>= x` 的位置 `pos`。
2. 如果 `pos == len(tails)`，说明 `x` 比所有尾巴都大，追加。
3. 否则用 `x` 替换 `tails[pos]`。

严格递增使用 `bisect_left`，它对应 C++ 的 `lower_bound`。

```text
pos = first index where tails[pos] >= x
```

为什么替换不会破坏答案？

- 替换前：存在长度 `pos + 1` 的子序列，尾巴是旧值。
- 替换后：也存在长度 `pos + 1` 的子序列，尾巴是更小或相等的 `x`。
- 更小的尾巴不会减少未来扩展机会，只会增加或保持机会。
- `tails` 的长度只在能形成更长序列时增加，因此最终长度就是 LIS 长度。

### 4.3 为什么 `tails` 可以二分

`tails` 本身保持递增。

假设存在：

```text
tails[k - 1] >= tails[k]
```

长度为 `k + 1` 的递增子序列去掉最后一个元素后，得到长度为 `k` 的递增子序列，其尾元素一定小于 `tails[k]`。

那么长度为 `k` 的最小可能尾元素应该小于等于这个尾元素：

```text
tails[k - 1] < tails[k]
```

与假设矛盾。因此 `tails` 严格递增，可以二分。

### 4.4 完整 trace

输入：

```text
nums = [10, 9, 2, 5, 3, 7, 101, 18]
```

| x | 二分位置 | 操作 | tails |
|---:|---:|---|---|
| 10 | 0 | 追加 | `[10]` |
| 9 | 0 | 替换 10 | `[9]` |
| 2 | 0 | 替换 9 | `[2]` |
| 5 | 1 | 追加 | `[2, 5]` |
| 3 | 1 | 替换 5 | `[2, 3]` |
| 7 | 2 | 追加 | `[2, 3, 7]` |
| 101 | 3 | 追加 | `[2, 3, 7, 101]` |
| 18 | 3 | 替换 101 | `[2, 3, 7, 18]` |

答案：`len(tails) = 4`。

### 4.5 可运行 Python：`O(n log n)` 只求长度

```python
from bisect import bisect_left
from typing import List


def length_of_lis_binary(nums: List[int]) -> int:
    tails: List[int] = []

    for x in nums:
        pos = bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x

    return len(tails)


if __name__ == "__main__":
    cases = [
        ([10, 9, 2, 5, 3, 7, 101, 18], 4),
        ([0, 1, 0, 3, 2, 3], 4),
        ([7, 7, 7, 7, 7], 1),
        ([], 0),
        ([4, 10, 4, 3, 8, 9], 3),
    ]

    for nums, expected in cases:
        actual = length_of_lis_binary(nums)
        print(nums, "=>", actual)
        assert actual == expected

    print("all binary length tests passed")
```

预期输出：

```text
[10, 9, 2, 5, 3, 7, 101, 18] => 4
[0, 1, 0, 3, 2, 3] => 4
[7, 7, 7, 7, 7] => 1
[] => 0
[4, 10, 4, 3, 8, 9] => 3
all binary length tests passed
```

复杂度：

$$
T(n)=\sum_{i=1}^{n} O(\log i)=O(n\log n)
$$

空间复杂度：

$$
S(n)=O(n)
$$

---

## 5. 用二分解法恢复实际 LIS

只存 `tails` 的值不够，因为 `tails` 可能不是原数组中的同一条子序列。要恢复路径，需要存下标。

维护三个数组：

| 名称 | 含义 |
|---|---|
| `tails_values` | 每个长度的最小尾值，负责二分 |
| `tails_indices` | `tails_values[k]` 对应的原数组下标 |
| `prev_indices` | 每个元素在恢复路径中的前驱下标 |

处理 `nums[i] = x` 时：

1. `pos = bisect_left(tails_values, x)`。
2. 如果 `pos > 0`，则 `prev_indices[i] = tails_indices[pos - 1]`。
3. 更新 `tails_values[pos] = x` 和 `tails_indices[pos] = i`。
4. 最后从 `tails_indices[-1]` 开始沿 `prev_indices` 回溯。

```python
from bisect import bisect_left
from typing import List


def reconstruct_lis_binary(nums: List[int]) -> List[int]:
    if not nums:
        return []

    tails_values: List[int] = []
    tails_indices: List[int] = []
    prev_indices = [-1] * len(nums)

    for i, x in enumerate(nums):
        pos = bisect_left(tails_values, x)

        if pos > 0:
            prev_indices[i] = tails_indices[pos - 1]

        if pos == len(tails_values):
            tails_values.append(x)
            tails_indices.append(i)
        else:
            tails_values[pos] = x
            tails_indices[pos] = i

    result_indices = []
    k = tails_indices[-1]
    while k != -1:
        result_indices.append(k)
        k = prev_indices[k]

    result_indices.reverse()
    return [nums[i] for i in result_indices]


if __name__ == "__main__":
    tests = [
        [10, 9, 2, 5, 3, 7, 101, 18],
        [0, 1, 0, 3, 2, 3],
        [4, 10, 4, 3, 8, 9],
    ]

    for nums in tests:
        seq = reconstruct_lis_binary(nums)
        print(nums, "=>", seq)
        assert all(seq[i] < seq[i + 1] for i in range(len(seq) - 1))
```

示例输出：

```text
[10, 9, 2, 5, 3, 7, 101, 18] => [2, 3, 7, 18]
[0, 1, 0, 3, 2, 3] => [0, 1, 2, 3]
[4, 10, 4, 3, 8, 9] => [3, 8, 9]
```

路径恢复时最常见的错误是只保存 `tails_values`。值数组能给长度，但不能保证这些值来自合法的原始下标链。

---

## 6. 严格递增 vs 非递减：`bisect_left` 和 `bisect_right`

两个问题只差一个二分边界：

| 目标 | 条件 | Python 二分 | C++ 二分 |
|---|---|---|---|
| 最长严格递增 | `<` | `bisect_left(tails, x)` | `lower_bound` |
| 最长非递减 | `<=` | `bisect_right(tails, x)` | `upper_bound` |

为什么非递减要用 `bisect_right`？

数组 `[1, 1, 1]`：

- 严格递增：每个 `1` 都替换位置 0，长度为 1。
- 非递减：每个 `1` 都应该接在前一个 `1` 后面，长度为 3。

```python
from bisect import bisect_left, bisect_right
from typing import List


def lis_strict(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        pos = bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)


def longest_non_decreasing(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        pos = bisect_right(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)


if __name__ == "__main__":
    nums = [1, 1, 1]
    print("strict:", lis_strict(nums))
    print("non-decreasing:", longest_non_decreasing(nums))
    assert lis_strict(nums) == 1
    assert longest_non_decreasing(nums) == 3
```

预期输出：

```text
strict: 1
non-decreasing: 3
```

---

## 7. 常见变体

### 7.1 最长递减子序列（LDS）

把比较方向反过来即可。

做法一：对每个数取负数，然后求 LIS。

```python
from bisect import bisect_left
from typing import List


def longest_decreasing_subsequence(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        y = -x
        pos = bisect_left(tails, y)
        if pos == len(tails):
            tails.append(y)
        else:
            tails[pos] = y
    return len(tails)


if __name__ == "__main__":
    nums = [9, 4, 3, 2, 5, 4, 3, 2]
    print(longest_decreasing_subsequence(nums))
    assert longest_decreasing_subsequence(nums) == 5
```

一条递减子序列是 `[9, 5, 4, 3, 2]`。

### 7.2 LIS 的个数

题目：返回最长递增子序列的数量。

这类题不适合直接用 `tails`，因为 `tails` 会丢弃很多计数信息。用 DP：

- `lengths[i]`：以 `i` 结尾的 LIS 长度。
- `counts[i]`：以 `i` 结尾、且长度为 `lengths[i]` 的 LIS 数量。

转移：

```text
如果 nums[j] < nums[i]:
  如果 lengths[j] + 1 > lengths[i]:
      lengths[i] = lengths[j] + 1
      counts[i] = counts[j]
  如果 lengths[j] + 1 == lengths[i]:
      counts[i] += counts[j]
```

```python
from typing import List


def count_number_of_lis(nums: List[int]) -> int:
    if not nums:
        return 0

    n = len(nums)
    lengths = [1] * n
    counts = [1] * n

    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                candidate = lengths[j] + 1
                if candidate > lengths[i]:
                    lengths[i] = candidate
                    counts[i] = counts[j]
                elif candidate == lengths[i]:
                    counts[i] += counts[j]

    longest = max(lengths)
    return sum(counts[i] for i in range(n) if lengths[i] == longest)


if __name__ == "__main__":
    assert count_number_of_lis([1, 3, 5, 4, 7]) == 2
    assert count_number_of_lis([2, 2, 2, 2, 2]) == 5
    print("count tests passed")
```

`[1, 3, 5, 4, 7]` 的两条 LIS 是：

```text
[1, 3, 5, 7]
[1, 3, 4, 7]
```

### 7.3 Russian Doll Envelopes

题目：信封有 `(width, height)`，一个信封能套进另一个信封，当且仅当宽和高都严格更小。求最多能套几层。

关键排序：

```text
width 升序；width 相同时 height 降序
```

然后对 `height` 求严格 LIS。

为什么同宽要按高度降序？

因为同宽信封不能互相嵌套。如果同宽高度升序，LIS 会错误地把它们串起来。

```python
from bisect import bisect_left
from typing import List, Tuple


def max_envelopes(envelopes: List[Tuple[int, int]]) -> int:
    envelopes.sort(key=lambda item: (item[0], -item[1]))
    tails: List[int] = []

    for _, height in envelopes:
        pos = bisect_left(tails, height)
        if pos == len(tails):
            tails.append(height)
        else:
            tails[pos] = height

    return len(tails)


if __name__ == "__main__":
    envelopes = [(5, 4), (6, 4), (6, 7), (2, 3)]
    print(max_envelopes(envelopes))
    assert max_envelopes(envelopes) == 3
```

排序后：

```text
(2, 3), (5, 4), (6, 7), (6, 4)
```

高度序列：

```text
3, 4, 7, 4
```

LIS 长度是 `3`，对应 `(2, 3) -> (5, 4) -> (6, 7)`。

### 7.4 二维 LIS

二维 LIS 的通用套路：

1. 选一个维度排序。
2. 对另一个维度做一维 LIS。
3. 如果两个维度都要求严格递增，排序时相同第一维必须让第二维反向。

模板：

```python
from bisect import bisect_left
from typing import List, Tuple


def lis_2d(points: List[Tuple[int, int]]) -> int:
    points.sort(key=lambda p: (p[0], -p[1]))
    tails: List[int] = []

    for _, y in points:
        pos = bisect_left(tails, y)
        if pos == len(tails):
            tails.append(y)
        else:
            tails[pos] = y

    return len(tails)


if __name__ == "__main__":
    points = [(1, 1), (1, 2), (2, 2), (3, 3)]
    print(lis_2d(points))
    assert lis_2d(points) == 3
```

如果题目要求第二维非递减，需要重新分析排序和 `bisect_right`，不要机械套模板。

---

## 8. LIS 与 LCS 的关系：不要把两个问题混在一起

最长公共子序列（Longest Common Subsequence, LCS）给定两个序列 `a` 和 `b`，求二者共有的最长子序列长度。

经典 DP：

$$
dp[i][j] =
\begin{cases}
dp[i-1][j-1] + 1, & a[i-1] = b[j-1] \\
\max(dp[i-1][j], dp[i][j-1]), & a[i-1] \ne b[j-1]
\end{cases}
$$

时间复杂度：

$$
O(n \cdot m)
$$

空间可优化到：

$$
O(\min(n,m))
$$

但一般情况下，LCS 没有类似 LIS 的 $O(n \log n)$ 通用算法，更不存在只靠 $O(\log n)$ 就能解决任意输入的通用方法。直觉原因是：LCS 要同时处理两个序列之间的大量匹配与不匹配关系，状态天然是二维的；LIS 只在一个序列内部选择递增下标链。

精确回答用户常问的问题：

> LCS 有没有 $O(\log n)$ 或明显快于 $O(nm)$ 的通用解？

没有已知的通用 $O(\log n)$、$O(n \log n)$ 或强多项式突破算法能解决任意 LCS。实际工程中可以用位集优化、剪枝、启发式或针对小字母表/稀疏匹配的算法加速常数或特定输入，但经典一般模型下仍以 $O(nm)$ DP 为基准。

### 8.1 特殊情况：一个序列元素互不相同，可把 LCS 转成 LIS

如果 `a` 中每个元素都不重复，可以建立：

```text
value -> index in a
```

然后把 `b` 中也出现在 `a` 的元素映射成 `a` 的下标序列。此时问题变成：在这个下标序列中找最长递增子序列。

原因：

- LCS 要保持 `a` 和 `b` 中的相对顺序。
- 映射到 `a` 的下标后，在 `b` 中选择一个公共子序列，就等价于选择一串在 `a` 中递增的下标。

示例：

```text
a = [3, 4, 9, 1]
b = [5, 3, 8, 9, 10, 2, 1]

index in a:
3 -> 0
4 -> 1
9 -> 2
1 -> 3

b 映射后:
3, 9, 1 -> [0, 2, 3]

LIS([0, 2, 3]) = 3
所以 LCS 长度 = 3，对应 [3, 9, 1]
```

可运行代码：

```python
from bisect import bisect_left
from typing import Hashable, Iterable, List


def lcs_via_lis_when_first_distinct(a: Iterable[Hashable], b: Iterable[Hashable]) -> int:
    a_list = list(a)
    if len(set(a_list)) != len(a_list):
        raise ValueError("the first sequence must contain distinct elements")

    pos = {value: i for i, value in enumerate(a_list)}
    mapped: List[int] = [pos[value] for value in b if value in pos]

    tails: List[int] = []
    for x in mapped:
        idx = bisect_left(tails, x)
        if idx == len(tails):
            tails.append(x)
        else:
            tails[idx] = x

    return len(tails)


if __name__ == "__main__":
    a = [3, 4, 9, 1]
    b = [5, 3, 8, 9, 10, 2, 1]
    print(lcs_via_lis_when_first_distinct(a, b))
    assert lcs_via_lis_when_first_distinct(a, b) == 3
```

这个技巧常见于「两个排列的 LCS」：

- 两个数组都是同一批互不相同元素的排列。
- 先把第二个排列映射成第一个排列中的位置。
- 对位置数组求 LIS。
- 复杂度从 $O(n^2)$ 降到 $O(n \log n)$。

如果元素重复，映射会一对多，不能直接用这个简单技巧，需要更复杂的数据结构或回到 LCS DP。

---

## 9. 面试手写模板

### 9.1 只求长度，推荐最终版

```python
from bisect import bisect_left
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        tails: List[int] = []
        for x in nums:
            i = bisect_left(tails, x)
            if i == len(tails):
                tails.append(x)
            else:
                tails[i] = x
        return len(tails)
```

### 9.2 需要讲清楚的三句话

1. `tails[i]` 表示长度为 `i + 1` 的递增子序列的最小可能尾元素。
2. 对当前数 `x`，替换第一个 `>= x` 的尾元素，是为了让同长度子序列的尾巴尽量小。
3. `tails` 单调递增，所以可以二分；`tails` 的长度就是目前能构造出的最长长度。

### 9.3 白板推导顺序

```text
1. 先定义 dp[i] = 以 i 结尾的 LIS 长度。
2. 写出 nums[j] < nums[i] 时 dp[i] = max(dp[i], dp[j] + 1)。
3. 得出 O(n^2)。
4. 观察只关心「某个长度的最小尾巴」。
5. 定义 tails。
6. 用 lower_bound 找替换位置。
7. 得出 O(n log n)。
```

---

## 10. 一次性可运行综合脚本

下面脚本把本章核心函数放在一起，并做基本校验。

```python
from bisect import bisect_left, bisect_right
from typing import Hashable, Iterable, List, Tuple


def length_of_lis_dp(nums: List[int]) -> int:
    if not nums:
        return 0
    dp = [1] * len(nums)
    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)


def length_of_lis_binary(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        pos = bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)


def reconstruct_lis_binary(nums: List[int]) -> List[int]:
    if not nums:
        return []
    tails_values: List[int] = []
    tails_indices: List[int] = []
    prev_indices = [-1] * len(nums)
    for i, x in enumerate(nums):
        pos = bisect_left(tails_values, x)
        if pos > 0:
            prev_indices[i] = tails_indices[pos - 1]
        if pos == len(tails_values):
            tails_values.append(x)
            tails_indices.append(i)
        else:
            tails_values[pos] = x
            tails_indices[pos] = i
    indices = []
    k = tails_indices[-1]
    while k != -1:
        indices.append(k)
        k = prev_indices[k]
    indices.reverse()
    return [nums[i] for i in indices]


def longest_non_decreasing(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        pos = bisect_right(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)


def count_number_of_lis(nums: List[int]) -> int:
    if not nums:
        return 0
    n = len(nums)
    lengths = [1] * n
    counts = [1] * n
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                candidate = lengths[j] + 1
                if candidate > lengths[i]:
                    lengths[i] = candidate
                    counts[i] = counts[j]
                elif candidate == lengths[i]:
                    counts[i] += counts[j]
    longest = max(lengths)
    return sum(counts[i] for i in range(n) if lengths[i] == longest)


def max_envelopes(envelopes: List[Tuple[int, int]]) -> int:
    ordered = sorted(envelopes, key=lambda item: (item[0], -item[1]))
    tails: List[int] = []
    for _, height in ordered:
        pos = bisect_left(tails, height)
        if pos == len(tails):
            tails.append(height)
        else:
            tails[pos] = height
    return len(tails)


def lcs_via_lis_when_first_distinct(a: Iterable[Hashable], b: Iterable[Hashable]) -> int:
    a_list = list(a)
    if len(set(a_list)) != len(a_list):
        raise ValueError("the first sequence must contain distinct elements")
    pos = {value: i for i, value in enumerate(a_list)}
    mapped = [pos[value] for value in b if value in pos]
    return length_of_lis_binary(mapped)


def is_strictly_increasing(seq: List[int]) -> bool:
    return all(seq[i] < seq[i + 1] for i in range(len(seq) - 1))


if __name__ == "__main__":
    base = [10, 9, 2, 5, 3, 7, 101, 18]
    assert length_of_lis_dp(base) == 4
    assert length_of_lis_binary(base) == 4

    seq = reconstruct_lis_binary(base)
    print("one LIS:", seq)
    assert len(seq) == 4
    assert is_strictly_increasing(seq)

    assert longest_non_decreasing([1, 1, 1]) == 3
    assert count_number_of_lis([1, 3, 5, 4, 7]) == 2
    assert max_envelopes([(5, 4), (6, 4), (6, 7), (2, 3)]) == 3
    assert lcs_via_lis_when_first_distinct([3, 4, 9, 1], [5, 3, 8, 9, 10, 2, 1]) == 3

    print("all integrated LIS tests passed")
```

运行方式：

```bash
python lis_lab.py
```

预期输出：

```text
one LIS: [2, 3, 7, 18]
all integrated LIS tests passed
```

---

## 11. 常见坑与排查表

| 症状 | 常见原因 | 修复方法 |
|---|---|---|
| `[7,7,7]` 返回 3 | 把严格递增误写成非递减 | 严格递增用 `bisect_left`，DP 条件用 `<` |
| `tails` 输出看起来不是原数组子序列 | 误以为 `tails` 本身就是答案序列 | 只求长度可以用 `tails`；恢复序列必须记录下标和前驱 |
| DP 返回 `dp[-1]` | 忘记 LIS 不一定以最后元素结尾 | 返回 `max(dp)` |
| Russian Doll 同宽信封被串起来 | 同宽高度升序排序 | 改成 `(width, -height)` |
| LCS 想直接套 LIS | 两个序列元素重复或没有先映射 | 只有「一个序列元素互不相同」时才可用映射技巧 |
| 非递减结果偏小 | 用了 `bisect_left` | 非递减用 `bisect_right` |
| 计数题答案偏小 | 用 `tails` 丢掉了多条路径 | 用 `lengths + counts` 的 DP |

---

## 12. 检查清单

- [ ] 能用一句话区分子序列和子数组。
- [ ] 能写出 $dp[i]$ 的定义：以 `nums[i]` 结尾的 LIS 长度。
- [ ] 能写出转移方程并说明为什么答案是 `max(dp)`。
- [ ] 能手工 trace `[10,9,2,5,3,7,101,18]`。
- [ ] 能解释 `tails[i]` 是「长度 `i+1` 的最小尾元素」，不是实际 LIS。
- [ ] 能说明严格递增用 `bisect_left`，非递减用 `bisect_right`。
- [ ] 能恢复一条实际 LIS，而不只是长度。
- [ ] 能解决 Russian Doll Envelopes，并解释同宽高度降序排序。
- [ ] 能回答 LCS 一般不能用 LIS 的 $O(n \log n)$ 解法。
- [ ] 能说明「排列 LCS」为什么可以转成 LIS。

---

## 13. 练习题

### 入门

1. 实现 `length_of_lis_dp(nums)`，并打印每一步的 `dp` 数组。
2. 对 `[4, 10, 4, 3, 8, 9]` 手工写出 `tails` 变化过程。
3. 把严格递增版本改成非递减版本，验证 `[1, 1, 1]` 输出 `3`。

### 面试级

1. 实现 `reconstruct_lis_binary(nums)`，返回一条实际 LIS。
2. 实现 `count_number_of_lis(nums)`，验证 `[1,3,5,4,7]` 输出 `2`。
3. 实现 Russian Doll Envelopes，验证 `[(5,4),(6,4),(6,7),(2,3)]` 输出 `3`。

### 进阶

1. 给定两个排列，写 `lcs_of_permutations(a, b)`，内部转成 LIS。
2. 给定二维点 `(x, y)`，求同时满足 `x` 和 `y` 严格递增的最长链。
3. 研究带权 LIS：每个元素有权重，目标不是最长长度而是最大总权重。提示：可从 $O(n^2)$ DP 开始。

---

## 14. 延伸阅读

- CP-Algorithms: Longest increasing subsequence：<https://cp-algorithms.com/sequences/longest_increasing_subsequence.html>
- LeetCode 300. Longest Increasing Subsequence：<https://leetcode.com/problems/longest-increasing-subsequence/>
- LeetCode 673. Number of Longest Increasing Subsequence：<https://leetcode.com/problems/number-of-longest-increasing-subsequence/>
- LeetCode 354. Russian Doll Envelopes：<https://leetcode.com/problems/russian-doll-envelopes/>
- Python `bisect` 官方文档：<https://docs.python.org/3/library/bisect.html>
- 复习动态规划框架：[动态规划实战指南](06-dynamic-programming-guide.md)


`标签` `DP` `二分查找` `LIS`

---

[← 上一章](06-dynamic-programming-guide.md) · [WP-02 目录](README.md)
