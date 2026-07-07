[← 手册首页](../README.md) · [WP-02 职业发展](README.md)

---

# 编程面试的核心套路（模式化刷题）

> 结论先行：编程面试题目千变万化，**底层模式就十几个**。会识别“这题属于哪个模式”，比盲刷题量更重要。目标是看到题面能在 30 秒内报出候选模式和数据结构，并在 3 分钟内说清复杂度、边界条件和第一版可写方案。

相关阅读：[动态规划系统指南](06-dynamic-programming-guide.md)、[最长递增子序列](07-longest-increasing-subsequence.md)、[系统设计面试框架](03-system-design-interview.md)、[技术面试英语](../english/01-technical-interview-english.md)。

## 0. 本章怎么用

本章不是“题单”，而是一套可执行的面试训练系统：你会把题目先归类为模式，再套模板、处理边界、写出可运行代码。

**前置知识**：Python 基础、数组/链表/树/图/哈希表、时间复杂度；建议本地安装 Python 3.10+。

**你学完后要能做到**：

- 30 秒内从题面识别 1–3 个候选模式。
- 2 分钟内口述暴力解、优化方向、目标复杂度。
- 10–20 分钟内写出主要模式的 Python 模板。
- 每题提交前用 3 个边界样例自测：空/最小、重复/相等、极端规模。
- 复盘时把错题归入“模式 + 失误类型”，而不是只记题号。

**本地运行方法**：每个代码块都可单独保存为 `solution.py` 后运行：

```powershell
python .\solution.py
```

预期看到 `OK` 或示例输出。下面所有代码只使用 Python 标准库。

## 1. 面试模式心智模型

面试题通常由三层组成：

1. **输入结构**：数组、字符串、链表、树、图、矩阵、区间、单词集合。
2. **目标动词**：找最短/最长、计数、判断存在、列出所有、求第 K、处理依赖、维护在线状态。
3. **约束规模**：`n <= 20` 可以指数；`n <= 10^3` 常见 `O(n^2)`；`n >= 10^5` 通常要 `O(n log n)` 或 `O(n)`。

把这三层合起来，就能反推出模式。例如：“字符串 + 最长无重复 + n=10^5”几乎一定是滑动窗口；“课程依赖 + 能否完成”几乎一定是拓扑排序。

### 1.1 高频模式速查表

| 模式 | 识别信号 | 典型题 | 首选复杂度 |
| --- | --- | --- | --- |
| 双指针 | 有序数组、两端收缩、原地去重 | Two Sum II、3Sum | `O(n)`/`O(n^2)` |
| 滑动窗口 | 连续子数组/子串、最长/最短、固定长度 | Longest Substring Without Repeating Characters | `O(n)` |
| 快慢指针 | 链表环、找中点、原地链表技巧 | Linked List Cycle | `O(n)` |
| 二分查找 | 有序、单调谓词、“最小可行值” | Search in Rotated Sorted Array | `O(log n)` |
| BFS/DFS | 树/图/矩阵连通、层序、最短步数 | Number of Islands、Binary Tree Level Order | `O(V+E)` |
| 回溯 | 所有方案、排列组合、约束搜索 | Subsets、Permutations、N-Queens | 指数级 |
| 堆 / Top-K | 第 K 大/小、动态中位数、合并有序流 | Top K Frequent Elements | `O(n log k)` |
| 区间 | 合并、重叠、会议室、扫描线 | Merge Intervals | `O(n log n)` |
| 单调栈 | 下一个更大/更小、温度、柱状图 | Daily Temperatures | `O(n)` |
| 前缀和 | 子数组和、区间和、频次差 | Subarray Sum Equals K | `O(n)` |
| 并查集 | 连通性、分组、动态合并 | Number of Provinces | 近似 `O(α(n))` |
| Trie | 前缀、字典、自动补全、单词搜索剪枝 | Implement Trie | `O(L)` |
| 拓扑排序 | 依赖顺序、有向图是否有环 | Course Schedule | `O(V+E)` |
| 动态规划 | 最优子结构、重叠子问题、选择/不选择 | Coin Change、LIS | 视状态数而定 |
| 哈希表（辅助） | 查存在、计数、去重、映射索引 | Two Sum、Valid Anagram | `O(n)` |

### 1.2 30 秒识别流程

```text
读题 30 秒：
1. 圈输入结构：array/string/list/tree/graph/matrix/interval/word
2. 圈目标词：longest/shortest/count/all/kth/exist/minimize/order
3. 看限制：n=20? 10^3? 10^5?
4. 报候选模式：主模式 + 辅助结构
5. 先说暴力，再说为什么能优化
```

示例话术：

> The input is a string and asks for the longest contiguous substring under a constraint. That suggests a variable-size sliding window with a hash map/count map. I will maintain a valid window and move the left pointer only when the constraint is violated.

## 2. 模式一：双指针 Two Pointers

### 2.1 什么时候识别

- 数组/字符串已经排序，或排序后不影响答案。
- 题目要求找两个数、三个数、去重、原地压缩。
- 你能用“左端”和“右端”的移动排除一批候选。

### 2.2 模板

```python
def two_pointers(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        s = nums[left] + nums[right]
        if s == target:
            return [left, right]
        if s < target:
            left += 1
        else:
            right -= 1
    return [-1, -1]
```

### 2.3 例题：LeetCode 167 Two Sum II

思路：数组递增。若两端和太小，左指针右移才能变大；若太大，右指针左移才能变小。

```python
from typing import List

class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        left, right = 0, len(numbers) - 1
        while left < right:
            total = numbers[left] + numbers[right]
            if total == target:
                return [left + 1, right + 1]
            if total < target:
                left += 1
            else:
                right -= 1
        return []

if __name__ == "__main__":
    s = Solution()
    assert s.twoSum([2, 7, 11, 15], 9) == [1, 2]
    assert s.twoSum([2, 3, 4], 6) == [1, 3]
    print("OK")
```

复杂度：时间 `O(n)`，空间 `O(1)`。

练习：3Sum、Container With Most Water、Remove Duplicates from Sorted Array、Valid Palindrome。

## 3. 模式二：滑动窗口 Sliding Window

### 3.1 什么时候识别

- 题目包含“连续子数组/连续子串”。
- 要求最长、最短、固定长度最大值、窗口内满足某个计数约束。
- 右指针扩张，左指针收缩后，状态可以增量维护。

### 3.2 模板

```python
from collections import defaultdict

def variable_window(s):
    count = defaultdict(int)
    left = 0
    best = 0
    for right, ch in enumerate(s):
        count[ch] += 1
        while count[ch] > 1:
            count[s[left]] -= 1
            left += 1
        best = max(best, right - left + 1)
    return best
```

### 3.3 例题：LeetCode 3 Longest Substring Without Repeating Characters

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last_seen = {}
        left = 0
        ans = 0
        for right, ch in enumerate(s):
            if ch in last_seen and last_seen[ch] >= left:
                left = last_seen[ch] + 1
            last_seen[ch] = right
            ans = max(ans, right - left + 1)
        return ans

if __name__ == "__main__":
    sol = Solution()
    assert sol.lengthOfLongestSubstring("abcabcbb") == 3
    assert sol.lengthOfLongestSubstring("bbbbb") == 1
    assert sol.lengthOfLongestSubstring("") == 0
    print("OK")
```

复杂度：时间 `O(n)`，空间 `O(min(n, charset))`。

练习：Minimum Window Substring、Longest Repeating Character Replacement、Permutation in String、Maximum Average Subarray I。

## 4. 模式三：快慢指针 Fast / Slow Pointers

### 4.1 什么时候识别

- 链表是否有环、环入口、找中点、倒数第 N 个节点。
- 题目要求 `O(1)` 额外空间。
- 可用两个速度不同的指针制造相遇或固定间距。

### 4.2 模板

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

### 4.3 例题：LeetCode 141 Linked List Cycle

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def hasCycle(self, head) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False

def build_cycle(values, pos):
    nodes = [ListNode(v) for v in values]
    for i in range(len(nodes) - 1):
        nodes[i].next = nodes[i + 1]
    if pos != -1:
        nodes[-1].next = nodes[pos]
    return nodes[0] if nodes else None

if __name__ == "__main__":
    sol = Solution()
    assert sol.hasCycle(build_cycle([3, 2, 0, -4], 1)) is True
    assert sol.hasCycle(build_cycle([1, 2], -1)) is False
    print("OK")
```

复杂度：时间 `O(n)`，空间 `O(1)`。

练习：Linked List Cycle II、Middle of the Linked List、Remove Nth Node From End of List、Happy Number。

## 5. 模式四：二分查找 Binary Search

### 5.1 什么时候识别

- 输入有序，或答案空间存在单调性。
- 题目问“第一个/最后一个满足条件的位置”。
- 题目问“最小的最大值”“最小速度”“最大可行长度”。

### 5.2 模板：左闭右闭找第一个满足

```python
def first_true(lo, hi, ok):
    ans = hi + 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if ok(mid):
            ans = mid
            hi = mid - 1
        else:
            lo = mid + 1
    return ans
```

### 5.3 例题：LeetCode 704 Binary Search

```python
from typing import List

class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        return -1

if __name__ == "__main__":
    sol = Solution()
    assert sol.search([-1, 0, 3, 5, 9, 12], 9) == 4
    assert sol.search([-1, 0, 3, 5, 9, 12], 2) == -1
    print("OK")
```

复杂度：时间 `O(log n)`，空间 `O(1)`。

练习：Search in Rotated Sorted Array、Find Minimum in Rotated Sorted Array、Koko Eating Bananas、Find Peak Element。

## 6. 模式五：BFS / DFS 图与矩阵遍历

### 6.1 什么时候识别

- 树、图、网格、岛屿、连通块。
- BFS：最短步数、层序遍历、无权图最短路。
- DFS：遍历所有连通区域、递归搜索、树路径。

### 6.2 模板

```python
from collections import deque

def bfs(start, neighbors):
    q = deque([start])
    seen = {start}
    while q:
        node = q.popleft()
        for nxt in neighbors(node):
            if nxt not in seen:
                seen.add(nxt)
                q.append(nxt)
    return seen
```

### 6.3 例题：LeetCode 200 Number of Islands

```python
from typing import List

class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        if not grid or not grid[0]:
            return 0
        rows, cols = len(grid), len(grid[0])
        count = 0

        def dfs(r: int, c: int) -> None:
            if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != "1":
                return
            grid[r][c] = "0"
            dfs(r + 1, c)
            dfs(r - 1, c)
            dfs(r, c + 1)
            dfs(r, c - 1)

        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == "1":
                    count += 1
                    dfs(r, c)
        return count

if __name__ == "__main__":
    sol = Solution()
    grid = [list("11110"), list("11010"), list("11000"), list("00000")]
    assert sol.numIslands(grid) == 1
    grid2 = [list("11000"), list("11000"), list("00100"), list("00011")]
    assert sol.numIslands(grid2) == 3
    print("OK")
```

复杂度：时间 `O(mn)`，空间最坏 `O(mn)`（递归栈）。

练习：Rotting Oranges、Walls and Gates、Clone Graph、Binary Tree Level Order Traversal。

## 7. 模式六：回溯 Backtracking

### 7.1 什么时候识别

- 题目要求返回所有排列、组合、子集、路径。
- 需要“做选择 → 递归 → 撤销选择”。
- 输入规模通常较小，例如 `n <= 10` 或候选数量有限。

### 7.2 模板

```python
def backtrack(path, choices):
    if is_solution(path):
        ans.append(path[:])
        return
    for choice in choices:
        if not valid(choice, path):
            continue
        path.append(choice)
        backtrack(path, next_choices(choice))
        path.pop()
```

### 7.3 例题：LeetCode 78 Subsets

```python
from typing import List

class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        ans = []
        path = []

        def dfs(start: int) -> None:
            ans.append(path[:])
            for i in range(start, len(nums)):
                path.append(nums[i])
                dfs(i + 1)
                path.pop()

        dfs(0)
        return ans

if __name__ == "__main__":
    sol = Solution()
    out = sol.subsets([1, 2, 3])
    assert sorted(map(tuple, out)) == sorted(map(tuple, [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]))
    print("OK")
```

复杂度：时间 `O(n * 2^n)`，空间 `O(n)` 递归栈，不含输出。

练习：Permutations、Combination Sum、Generate Parentheses、N-Queens。

## 8. 模式七：堆 / Top-K

### 8.1 什么时候识别

- 第 K 大/小、前 K 个、高频 K 个。
- 数据流持续到来，需要维护局部最优。
- 多个有序列表合并。

### 8.2 模板

```python
import heapq

def top_k_largest(nums, k):
    heap = []
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:
            heapq.heappop(heap)
    return sorted(heap, reverse=True)
```

### 8.3 例题：LeetCode 347 Top K Frequent Elements

```python
from collections import Counter
import heapq
from typing import List

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        freq = Counter(nums)
        heap = []
        for num, count in freq.items():
            heapq.heappush(heap, (count, num))
            if len(heap) > k:
                heapq.heappop(heap)
        return [num for count, num in heap]

if __name__ == "__main__":
    sol = Solution()
    assert set(sol.topKFrequent([1, 1, 1, 2, 2, 3], 2)) == {1, 2}
    assert sol.topKFrequent([1], 1) == [1]
    print("OK")
```

复杂度：时间 `O(n log k)`，空间 `O(n)` 计数 + `O(k)` 堆。

练习：Kth Largest Element in an Array、Merge k Sorted Lists、Find Median from Data Stream、K Closest Points to Origin。

## 9. 模式八：区间 Intervals

### 9.1 什么时候识别

- 输入是 `[start, end]`，问重叠、合并、插入、最少会议室。
- 排序后只需比较当前区间与上一个区间。
- “同时在线人数”“最大重叠数”可转成扫描线。

### 9.2 模板

```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = []
    for start, end in intervals:
        if not merged or start > merged[-1][1]:
            merged.append([start, end])
        else:
            merged[-1][1] = max(merged[-1][1], end)
    return merged
```

### 9.3 例题：LeetCode 56 Merge Intervals

```python
from typing import List

class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        intervals.sort(key=lambda x: x[0])
        ans = []
        for start, end in intervals:
            if not ans or start > ans[-1][1]:
                ans.append([start, end])
            else:
                ans[-1][1] = max(ans[-1][1], end)
        return ans

if __name__ == "__main__":
    sol = Solution()
    assert sol.merge([[1,3],[2,6],[8,10],[15,18]]) == [[1,6],[8,10],[15,18]]
    assert sol.merge([[1,4],[4,5]]) == [[1,5]]
    print("OK")
```

复杂度：时间 `O(n log n)` 来自排序，空间 `O(n)` 输出。

练习：Insert Interval、Non-overlapping Intervals、Meeting Rooms II、Minimum Number of Arrows to Burst Balloons。

## 10. 模式九：单调栈 Monotonic Stack

### 10.1 什么时候识别

- 题目问“下一个更大/更小元素”。
- 当前元素会让之前一批元素得到答案。
- 需要在线性时间消除无用候选。

### 10.2 模板

```python
def next_greater(nums):
    ans = [-1] * len(nums)
    stack = []
    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            j = stack.pop()
            ans[j] = x
        stack.append(i)
    return ans
```

### 10.3 例题：LeetCode 739 Daily Temperatures

```python
from typing import List

class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        ans = [0] * len(temperatures)
        stack = []
        for i, temp in enumerate(temperatures):
            while stack and temperatures[stack[-1]] < temp:
                j = stack.pop()
                ans[j] = i - j
            stack.append(i)
        return ans

if __name__ == "__main__":
    sol = Solution()
    assert sol.dailyTemperatures([73,74,75,71,69,72,76,73]) == [1,1,4,2,1,1,0,0]
    assert sol.dailyTemperatures([30,40,50,60]) == [1,1,1,0]
    print("OK")
```

复杂度：时间 `O(n)`，每个元素最多入栈出栈一次；空间 `O(n)`。

练习：Next Greater Element I、Largest Rectangle in Histogram、Trapping Rain Water、Online Stock Span。

## 11. 模式十：前缀和 Prefix Sum

### 11.1 什么时候识别

- 子数组和、区间和、连续区间计数。
- 多次查询 `sum(i, j)`。
- 需要把“当前前缀 - 目标值”转成哈希查找。

### 11.2 模板

```python
from collections import defaultdict

def count_subarray_sum(nums, k):
    count = defaultdict(int)
    count[0] = 1
    prefix = 0
    ans = 0
    for x in nums:
        prefix += x
        ans += count[prefix - k]
        count[prefix] += 1
    return ans
```

### 11.3 例题：LeetCode 560 Subarray Sum Equals K

```python
from collections import defaultdict
from typing import List

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        seen = defaultdict(int)
        seen[0] = 1
        prefix = 0
        ans = 0
        for x in nums:
            prefix += x
            ans += seen[prefix - k]
            seen[prefix] += 1
        return ans

if __name__ == "__main__":
    sol = Solution()
    assert sol.subarraySum([1, 1, 1], 2) == 2
    assert sol.subarraySum([1, 2, 3], 3) == 2
    assert sol.subarraySum([0, 0, 0], 0) == 6
    print("OK")
```

复杂度：时间 `O(n)`，空间 `O(n)`。

练习：Range Sum Query Immutable、Contiguous Array、Product of Array Except Self、Subarray Sums Divisible by K。

## 12. 模式十一：并查集 Union-Find

### 12.1 什么时候识别

- 动态合并集合、判断两个节点是否连通。
- 无向图连通分量、朋友圈、省份数量、冗余边。
- 题目不要求路径本身，只要求分组或连通性。

### 12.2 模板

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1
        return True
```

### 12.3 例题：LeetCode 547 Number of Provinces

```python
from typing import List

class DSU:
    def __init__(self, n: int):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.count = n

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, a: int, b: int) -> None:
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return
        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1
        self.count -= 1

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        dsu = DSU(n)
        for i in range(n):
            for j in range(i + 1, n):
                if isConnected[i][j] == 1:
                    dsu.union(i, j)
        return dsu.count

if __name__ == "__main__":
    sol = Solution()
    assert sol.findCircleNum([[1,1,0],[1,1,0],[0,0,1]]) == 2
    assert sol.findCircleNum([[1,0,0],[0,1,0],[0,0,1]]) == 3
    print("OK")
```

复杂度：时间 `O(n^2 * α(n))`，空间 `O(n)`。

练习：Redundant Connection、Accounts Merge、Graph Valid Tree、Number of Connected Components in an Undirected Graph。

## 13. 模式十二：Trie 前缀树

### 13.1 什么时候识别

- 需要按前缀查找、自动补全、词典匹配。
- 多个单词共享前缀，逐个字符串暴力比较太慢。
- 单词搜索中需要按前缀剪枝。

### 13.2 模板

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False
```

### 13.3 例题：LeetCode 208 Implement Trie

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for ch in word:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True

    def search(self, word: str) -> bool:
        node = self._find(word)
        return bool(node and node.is_word)

    def startsWith(self, prefix: str) -> bool:
        return self._find(prefix) is not None

    def _find(self, text: str):
        node = self.root
        for ch in text:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node

if __name__ == "__main__":
    trie = Trie()
    trie.insert("apple")
    assert trie.search("apple") is True
    assert trie.search("app") is False
    assert trie.startsWith("app") is True
    trie.insert("app")
    assert trie.search("app") is True
    print("OK")
```

复杂度：插入/查找时间 `O(L)`，空间 `O(总字符数)`。

练习：Design Add and Search Words Data Structure、Word Search II、Replace Words、Maximum XOR of Two Numbers in an Array。

## 14. 模式十三：拓扑排序 Topological Sort

### 14.1 什么时候识别

- 有先修、依赖、构建顺序、任务调度。
- 有向图中需要判断是否有环。
- 输出一个合法顺序，或判断能否完成全部任务。

### 14.2 模板：Kahn 入度法

```python
from collections import deque

def topo(n, edges):
    graph = [[] for _ in range(n)]
    indeg = [0] * n
    for a, b in edges:
        graph[b].append(a)
        indeg[a] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        node = q.popleft()
        order.append(node)
        for nxt in graph[node]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                q.append(nxt)
    return order if len(order) == n else []
```

### 14.3 例题：LeetCode 207 Course Schedule

```python
from collections import deque
from typing import List

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = [[] for _ in range(numCourses)]
        indeg = [0] * numCourses
        for course, pre in prerequisites:
            graph[pre].append(course)
            indeg[course] += 1

        q = deque(i for i in range(numCourses) if indeg[i] == 0)
        visited = 0
        while q:
            node = q.popleft()
            visited += 1
            for nxt in graph[node]:
                indeg[nxt] -= 1
                if indeg[nxt] == 0:
                    q.append(nxt)
        return visited == numCourses

if __name__ == "__main__":
    sol = Solution()
    assert sol.canFinish(2, [[1, 0]]) is True
    assert sol.canFinish(2, [[1, 0], [0, 1]]) is False
    print("OK")
```

复杂度：时间 `O(V+E)`，空间 `O(V+E)`。

练习：Course Schedule II、Alien Dictionary、Minimum Height Trees、Parallel Courses。

## 15. 模式十四：动态规划 Dynamic Programming

### 15.1 什么时候识别

- 题目问最优值、方案数、可达性。
- 当前答案依赖更小规模子问题。
- 暴力递归会重复计算同一状态。

更多 DP 体系化训练见 [动态规划系统指南](06-dynamic-programming-guide.md) 和 [最长递增子序列](07-longest-increasing-subsequence.md)。

### 15.2 五步法模板

```text
1. 定义状态：dp[i] 表示什么？
2. 写转移：dp[i] 从哪些旧状态来？
3. 初始化：空集、第一行、第一列是什么？
4. 遍历顺序：从小到大、二维嵌套、倒序容量？
5. 返回答案：dp[n]、max(dp)、dp[m][n]？
```

### 15.3 例题：LeetCode 322 Coin Change

```python
from typing import List

class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        inf = amount + 1
        dp = [inf] * (amount + 1)
        dp[0] = 0
        for x in range(1, amount + 1):
            for coin in coins:
                if x >= coin:
                    dp[x] = min(dp[x], dp[x - coin] + 1)
        return -1 if dp[amount] == inf else dp[amount]

if __name__ == "__main__":
    sol = Solution()
    assert sol.coinChange([1, 2, 5], 11) == 3
    assert sol.coinChange([2], 3) == -1
    assert sol.coinChange([1], 0) == 0
    print("OK")
```

复杂度：时间 `O(amount * len(coins))`，空间 `O(amount)`。

练习：Climbing Stairs、House Robber、Longest Increasing Subsequence、Edit Distance。

## 16. 哈希表：最常见的辅助武器

哈希表本身常常不是主模式，但几乎所有模式都会用它维护状态：滑动窗口计数、前缀和频次、图访问集、去重、索引映射。

例题：LeetCode 1 Two Sum。

```python
from typing import List

class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}
        for i, x in enumerate(nums):
            need = target - x
            if need in seen:
                return [seen[need], i]
            seen[x] = i
        return []

if __name__ == "__main__":
    sol = Solution()
    assert sol.twoSum([2, 7, 11, 15], 9) == [0, 1]
    assert sol.twoSum([3, 2, 4], 6) == [1, 2]
    print("OK")
```

复杂度：时间 `O(n)`，空间 `O(n)`。

练习：Valid Anagram、Group Anagrams、Contains Duplicate、Longest Consecutive Sequence。

## 17. 从读题到 AC 的固定流程

### 17.1 现场流程

1. **复述题目**：确认输入、输出、是否允许修改输入、重复元素怎么处理。
2. **问约束**：`n` 多大、值域、是否有负数、是否排序。
3. **说暴力解**：让面试官知道你有 baseline。
4. **识别模式**：说明为什么选这个模式，不是背模板。
5. **走一个样例**：手动跑 3–5 步，验证状态变量含义。
6. **写代码**：先函数签名，再边界，再主体循环。
7. **自测**：正常样例 + 边界样例 + 反例。
8. **复杂度**：给时间、空间，并说明主导项。

### 17.2 约束反推表

| 约束 | 可接受复杂度 | 常见模式 |
| --- | --- | --- |
| `n <= 10` | `O(n!)` / `O(2^n)` | 回溯 |
| `n <= 20` | `O(2^n)` | 子集 DP、回溯 |
| `n <= 500` | `O(n^3)` 可能可过 | 区间 DP、Floyd |
| `n <= 2,000` | `O(n^2)` | DP、双层枚举 |
| `n <= 10^5` | `O(n log n)` / `O(n)` | 排序、堆、单调栈、滑窗 |
| `n >= 10^6` | `O(n)` 或更低常数 | 哈希、双指针、前缀和 |

## 18. 模式选择决策树

```text
是否要求所有方案？
├─ 是：回溯；若存在重复子问题，再考虑记忆化/DP
└─ 否：输入是否有依赖关系？
   ├─ 有向依赖：拓扑排序
   ├─ 无向连通：并查集 / DFS
   └─ 否：是否连续子数组/子串？
      ├─ 是：滑动窗口 / 前缀和
      └─ 否：是否有序或答案单调？
         ├─ 是：二分 / 双指针
         └─ 否：是否第 K / Top-K？
            ├─ 是：堆 / 快速选择
            └─ 否：是否下一个更大/更小？
               ├─ 是：单调栈
               └─ 否：考虑哈希表、排序、DP
```

## 19. 一题多模式对比：Subarray Sum Equals K

- 暴力：枚举所有子数组，时间 `O(n^2)`。
- 滑动窗口：只有当数组全非负时才稳定可用；有负数会失效。
- 前缀和 + 哈希：支持负数，时间 `O(n)`。

面试中要主动说出限制：“If all numbers are non-negative, a sliding window works. Since negative numbers are allowed, I will use prefix sum with a frequency map.”

## 20. 错题复盘模板

每做完一题，用 5 分钟写一条记录：

```text
题名：
主模式：
辅助结构：
我第一眼误判为：
卡住点：识别 / 边界 / 数据结构 / 复杂度 / 代码 bug
正确不变量：
下次看到什么信号要触发该模式：
复习日期：D+1 / D+3 / D+7 / D+14 / D+30
```

示例：

```text
题名：Subarray Sum Equals K
主模式：前缀和 + 哈希表
误判：滑动窗口
卡住点：忽略负数导致窗口不单调
正确不变量：seen[prefix] 表示某个前缀和出现次数
触发信号：连续子数组 + 和为 K + 有负数
```

## 21. 4 周训练计划与间隔重复

### 第 1 周：线性结构

- Day 1：双指针 3 题，手写 Two Sum II 与 3Sum。
- Day 2：滑动窗口 4 题，固定窗口和变长窗口各 2 题。
- Day 3：前缀和 3 题，重点练“prefix - k”。
- Day 4：单调栈 3 题，画栈变化表。
- Day 5：快慢指针 3 题，链表题必须手动画节点。
- Day 6：限时模拟 2 题，每题 35 分钟。
- Day 7：复盘错题，重写模板，不看答案。

### 第 2 周：树图与搜索

- Day 8：BFS 3 题，练层序与最短步数。
- Day 9：DFS 3 题，练网格连通块。
- Day 10：回溯 3 题，练选择/撤销。
- Day 11：拓扑排序 2 题，练建图和入度。
- Day 12：并查集 3 题，练路径压缩。
- Day 13：Trie 2 题，练前缀搜索。
- Day 14：模拟 2 题 + 复盘。

### 第 3 周：排序、堆、二分、区间

- Day 15：二分基础 3 题。
- Day 16：答案二分 3 题，用 `ok(mid)` 写法。
- Day 17：堆/Top-K 4 题。
- Day 18：区间 4 题，合并与扫描线。
- Day 19：混合题 2 题，强制先报模式。
- Day 20：模拟 2 题。
- Day 21：复盘 D+7 错题。

### 第 4 周：DP 与综合模拟

- Day 22：一维 DP 4 题。
- Day 23：二维 DP 3 题。
- Day 24：背包/选择型 DP 3 题。
- Day 25：随机 Easy/Medium 3 题。
- Day 26：随机 Medium 2 题，录屏复盘表达。
- Day 27：完整模拟 2 轮：45 分钟 coding + 10 分钟讲解。
- Day 28：整理最终模板卡片与错题清单。

### 间隔重复规则

- 第一次 AC 后：当天用英文讲一遍思路。
- D+1：不看代码重写。
- D+3：只看题名报模式与复杂度。
- D+7：换同模式新题验证迁移能力。
- D+14：复做错题。
- D+30：随机混合模拟。

## 22. 面试当天检查清单

- [ ] 先确认输入规模、值域、是否有负数/重复/排序。
- [ ] 先说暴力解，再说优化模式。
- [ ] 每个变量都有明确含义：`left/right/window/prefix/visited/dp`。
- [ ] 写循环前确定边界：空数组、单元素、全重复、无答案。
- [ ] 写完后手跑至少一个正常样例和一个边界样例。
- [ ] 主动报告时间复杂度和空间复杂度。
- [ ] 若卡住 5 分钟，改用暴力可运行版本争取 partial credit。
- [ ] 结束前说一个可优化方向或替代方案。

## 23. 常见坑与排查表

| 症状 | 常见原因 | 修复动作 |
| --- | --- | --- |
| 滑动窗口答案差 1 | 更新答案时机错 | 固定“加入右端 → 收缩到合法 → 更新答案” |
| 二分死循环 | `left/right` 更新不收缩 | 使用 `while left <= right` 或半开区间模板，别混用 |
| DFS 超时或爆栈 | 没有标记 visited | 入队/入栈时就标记，避免重复加入 |
| 回溯结果全一样 | append 了同一个 path 引用 | 记录时用 `path[:]` |
| Top-K 顺序不稳定 | 题目只要求集合，你却断言顺序 | 测试时用 `set()` 或排序后比较 |
| 前缀和漏计从 0 开始的子数组 | 没初始化 `seen[0] = 1` | 所有计数型前缀和先放 0 |
| 并查集退化 | 没路径压缩/按秩合并 | `find` 压缩，`union` 用 rank/size |
| DP 状态写不清 | 直接套公式 | 先用一句话定义 `dp[i]`，再写转移 |

## 24. 最小可执行训练项目：随机抽题器

把下面脚本保存为 `pattern_drill.py`，每天运行一次，按输出做 1–2 题。

```python
import random

BANK = {
    "two-pointers": ["Two Sum II", "3Sum", "Container With Most Water"],
    "sliding-window": ["Longest Substring Without Repeating Characters", "Minimum Window Substring"],
    "binary-search": ["Binary Search", "Koko Eating Bananas", "Find Peak Element"],
    "bfs-dfs": ["Number of Islands", "Rotting Oranges", "Clone Graph"],
    "backtracking": ["Subsets", "Permutations", "Combination Sum"],
    "heap": ["Top K Frequent Elements", "Kth Largest Element", "Merge k Sorted Lists"],
    "intervals": ["Merge Intervals", "Insert Interval", "Meeting Rooms II"],
    "monotonic-stack": ["Daily Temperatures", "Largest Rectangle in Histogram"],
    "prefix-sum": ["Subarray Sum Equals K", "Contiguous Array"],
    "union-find": ["Number of Provinces", "Redundant Connection"],
    "trie": ["Implement Trie", "Word Search II"],
    "topological-sort": ["Course Schedule", "Course Schedule II"],
    "dp": ["Coin Change", "House Robber", "Edit Distance"],
}

if __name__ == "__main__":
    pattern = random.choice(list(BANK))
    problem = random.choice(BANK[pattern])
    print(f"Today: {pattern} -> {problem}")
    print("Checklist: identify signal, write template, solve, test, complexity, review.")
```

运行：

```powershell
python .\pattern_drill.py
```

预期输出类似：

```text
Today: prefix-sum -> Subarray Sum Equals K
Checklist: identify signal, write template, solve, test, complexity, review.
```

## 25. 练到“面试可用”的完成标准

- [ ] 14 个核心模式都能不看资料写出模板。
- [ ] 每个模式至少完成 3 题，其中 1 题能用英文讲解。
- [ ] 错题本中每题都有“误判信号”和“正确不变量”。
- [ ] 随机 Medium 题 35 分钟内能完成 60% 以上。
- [ ] 任何提交都包含 3 个自测样例。
- [ ] 能解释为什么某题不能用更简单模式，例如负数数组不能直接滑动窗口。

## 26. 延伸阅读

- [LeetCode Explore](https://leetcode.com/explore/)：按数据结构与算法主题训练。
- [NeetCode Roadmap](https://neetcode.io/roadmap)：面试模式路线图。
- [CP-Algorithms](https://cp-algorithms.com/)：图、数据结构、DP 的严谨解释。
- [Python heapq 文档](https://docs.python.org/3/library/heapq.html)：堆 API 细节。
- [Python collections 文档](https://docs.python.org/3/library/collections.html)：`deque`、`Counter`、`defaultdict`。
- [VisuAlgo](https://visualgo.net/en)：可视化二分、图遍历、并查集等算法。

## 小结

把编程面试当作“模式识别 + 不变量维护 + 可运行代码”的训练。先判断输入结构和目标动词，再用约束规模筛掉不可能的复杂度。刷题数量会带来熟悉感，但真正迁移能力来自：每题都能说出模式、模板、不变量、边界和复杂度。


`标签` `算法` `面试` `刷题` `数据结构` `套路`

---

[← 上一章](04-behavioral-interview-star.md) · [WP-02 目录](README.md) · [下一章 →](06-dynamic-programming-guide.md)
