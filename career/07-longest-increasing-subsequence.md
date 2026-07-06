[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 最长递增子序列（LIS）

给定数组，求最长严格递增子序列的长度。朴素 DP 是 $O(n^2)$，
但用「贪心 + 二分」可以做到 $O(n \log n)$。

## 核心思想

维护数组 `tails`，其中 `tails[i]` 表示长度为 `i+1` 的递增子序列的**最小可能尾元素**。
遍历每个 `x`，用二分找到第一个 $\ge x$ 的位置替换（严格递增用 `lower_bound`）：

- 若 `x` 比所有尾元素都大 → 追加，长度 +1；
- 否则 → 替换第一个 $\ge x$ 的尾元素，保持「尾元素尽量小」。

`tails` 的长度即为答案。注意 `tails` 本身不一定是真正的 LIS，但长度正确。

## 实现

```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return tails.size();
}
```

## 复杂度

$$
T(n) = \sum_{i=1}^{n} O(\log i) = O(n \log n)
$$

> 若要「非严格递增」（允许相等），把 `lower_bound` 换成 `upper_bound`。


`标签` `DP` `二分查找` `LIS`

---

[← 上一章](06-dynamic-programming-guide.md) · [WP-02 目录](README.md)
