---
layout: note
title: 算法解题技巧总结
permalink: /notes/algo-tricks/
---

> 覆盖12大核心技巧，每项含适用场景、核心思想、Python模板、经典例题。

---

## 1. 双指针

### 1.1 对撞指针（两端向中间）

**适用场景**：有序数组求目标和、验证回文串、盛水容器等需要从两端夹逼的问题。

**核心思想**：左右指针分别指向两端，根据当前状态决定移动哪个指针，逐步缩小搜索空间，时间 O(n)。

> 💡 **图示**

```text
 nums = [1, 3, 5, 7, 9, 11]  target=12
  ↑                      ↑
  L                      R
  1+11=12 ✓ → 找到！

 若 L+R < target → L右移
 若 L+R > target → R左移
  ↑          ↑
  L          R
```

**解题模板**
```python
def two_pointer_collision(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        cur = arr[left] + arr[right]
        if cur == target:
            return [left, right]
        elif cur < target:
            left += 1   # 和太小，左指针右移
        else:
            right -= 1  # 和太大，右指针左移
    return []
```

**经典例题**
- **LC 167 两数之和 II（有序数组）**：直接套对撞模板，O(n)。
- **LC 15 三数之和**：排序后固定一个数，对剩余数组用对撞指针，注意去重。
- **LC 11 盛最多水的容器**：容量 = `min(h[l], h[r]) * (r-l)`，每次移动较矮一侧指针。
- **LC 125 验证回文串**：左右同时向中间走，忽略非字母数字字符。
- **LC 42 接雨水**：左右各维护最大高度 `maxL/maxR`，较小侧计算积水。

---

### 1.2 快慢指针（链表环检测 & 中点）

**适用场景**：链表环检测、找链表中点/倒数第k个节点、判断回文链表。

**核心思想**：慢指针每次走1步，快指针每次走2步。若有环，两者必在环内相遇；若无环，快指针先到 null。

> 💡 **图示**

```text
 1 → 2 → 3 → 4 → 5 → NULL
 S                          (slow每次走1步)
 F                          (fast每次走2步)

 Step1:  S=2, F=3
 Step2:  S=3, F=5
 Step3:  S=4, F=NULL → 中点=4（奇数链表）
```

**解题模板**
```python
# 环检测（Floyd判环）
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False

# 找链表中点
def find_middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow  # 奇数个节点返回中点，偶数返回中间偏右

# 找环入口（LC 142）
def detect_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            # 相遇后，一个从头走，一个从相遇点走，步长均为1
            ptr = head
            while ptr is not slow:
                ptr = ptr.next
                slow = slow.next
            return ptr
    return None
```

**经典例题**
- **LC 141 环形链表**：快慢指针是否相遇。
- **LC 142 环形链表 II**：Floyd算法找入口。
- **LC 876 链表的中间结点**：快慢指针找中点。
- **LC 234 回文链表**：找中点 → 反转后半段 → 比较。
- **LC 19 删除链表倒数第N个节点**：快指针先走N步，再同步前进。

---

### 1.3 滑动窗口（连续子数组/子串）

**适用场景**：求满足条件的最长/最短连续子数组、子串；窗口内元素的统计量随窗口扩缩单调变化。

**核心思想**：维护 `[left, right)` 窗口，right 不断右移扩大窗口，不满足条件时移动 left 收缩。

> 💡 **图示**

```text
 s = "a b c b a"
     [a b c]          窗口扩展
       [b c b]        左指针收缩（遇重复）
         [c b a]      继续扩展

 left↑         ↑right
```

**解题模板**
```python
from collections import defaultdict

def sliding_window(s, need_dict):
    """最小覆盖子串框架"""
    window = defaultdict(int)
    left = right = 0
    valid = 0           # 已满足条件的字符种数
    start, min_len = 0, float('inf')

    while right < len(s):
        c = s[right]
        right += 1
        # 扩大窗口
        if c in need_dict:
            window[c] += 1
            if window[c] == need_dict[c]:
                valid += 1

        # 收缩窗口
        while valid == len(need_dict):
            if right - left < min_len:
                start, min_len = left, right - left
            d = s[left]
            left += 1
            if d in need_dict:
                if window[d] == need_dict[d]:
                    valid -= 1
                window[d] -= 1

    return s[start:start + min_len] if min_len != float('inf') else ""
```

**经典例题**
- **LC 76 最小覆盖子串**：经典滑动窗口，上方模板直接适用。
- **LC 438 找到字符串中所有字母异位词**：固定窗口大小 = `len(p)`。
- **LC 3 无重复字符的最长子串**：窗口内字符不重复，`valid`改为set计数。
- **LC 209 长度最小的子数组**：窗口和 ≥ target 时尝试收缩。
- **LC 567 字符串的排列**：固定窗口，判断窗口是否是排列。

---

## 2. 前缀和 & 差分数组

### 2.1 前缀和

**适用场景**：频繁查询数组区间和、统计子数组和等于 k 的个数。

**核心思想**：`prefix[i] = arr[0] + ... + arr[i-1]`，区间和 `[l,r] = prefix[r+1] - prefix[l]`，O(1) 查询。

> 💡 **图示**

```text
 原数组:   [1,  2,  3,  4,  5]
 前缀和:   [0,  1,  3,  6, 10, 15]
            ↑
           pre[0]=0 (哨兵)

 区间[1,3]之和 = pre[4] - pre[1] = 10 - 1 = 9
```

**解题模板**
```python
# 一维前缀和
def build_prefix(arr):
    n = len(arr)
    prefix = [0] * (n + 1)
    for i in range(n):
        prefix[i + 1] = prefix[i] + arr[i]
    return prefix

def range_sum(prefix, l, r):
    """查询 arr[l..r] 的和（0-indexed，闭区间）"""
    return prefix[r + 1] - prefix[l]

# 子数组和等于 k（LC 560）
def subarray_sum(nums, k):
    from collections import defaultdict
    count = defaultdict(int)
    count[0] = 1
    prefix_sum = 0
    result = 0
    for num in nums:
        prefix_sum += num
        result += count[prefix_sum - k]  # 之前出现过 prefix_sum - k，说明中间这段 = k
        count[prefix_sum] += 1
    return result

# 二维前缀和
def build_2d_prefix(matrix):
    m, n = len(matrix), len(matrix[0])
    prefix = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            prefix[i][j] = (matrix[i-1][j-1]
                            + prefix[i-1][j]
                            + prefix[i][j-1]
                            - prefix[i-1][j-1])
    return prefix

def query_2d(prefix, r1, c1, r2, c2):
    """查询左上(r1,c1)到右下(r2,c2)的矩形和（0-indexed）"""
    return (prefix[r2+1][c2+1]
            - prefix[r1][c2+1]
            - prefix[r2+1][c1]
            + prefix[r1][c1])
```

**经典例题**
- **LC 303 区域和检索**：build_prefix + range_sum。
- **LC 560 和为K的子数组**：前缀和 + 哈希计数。
- **LC 304 二维区域和检索**：二维前缀和模板。
- **LC 525 连续数组**：0变-1，转化为子数组和为0。
- **LC 974 和可被 K 整除的子数组**：`(prefix % k + k) % k` 哈希计数。

---

### 2.2 差分数组

**适用场景**：对数组区间 `[l, r]` 统一加减某个值，操作结束后还原。

**核心思想**：差分数组 `diff[i] = arr[i] - arr[i-1]`，区间加法变成两点修改，最后对 diff 求前缀和得结果。

**解题模板**
```python
class DiffArray:
    def __init__(self, nums):
        n = len(nums)
        self.diff = [0] * (n + 1)
        # 初始化差分
        for i in range(n):
            self.diff[i] = nums[i] - (nums[i-1] if i > 0 else 0)

    def increment(self, l, r, val):
        """对 [l, r] 区间所有元素加 val（0-indexed）"""
        self.diff[l] += val
        if r + 1 < len(self.diff):
            self.diff[r + 1] -= val

    def result(self):
        res = []
        cur = 0
        for d in self.diff[:-1]:
            cur += d
            res.append(cur)
        return res
```

**经典例题**
- **LC 1109 航班预订统计**：区间 `[first, last]` 都加 seats，最后输出结果数组。
- **LC 1094 拼车**：差分数组统计每个站的乘客数，判断是否超容量。
- **LC 798 得分最高的最小轮调**：差分统计每种轮调得分变化。

---

## 3. 哈希表

**适用场景**：O(1) 查找、计数、去重、记录已访问状态。

**核心思想**：用 dict/set 换空间换时间，把 O(n) 遍历查找降到 O(1)。

**解题模板**
```python
# Two Sum（LC 1）
def two_sum(nums, target):
    seen = {}  # val -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# 字母异位词分组（LC 49）
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))   # 或者 frozenset(Counter(s).items())
        groups[key].append(s)
    return list(groups.values())

# 最长连续序列（LC 128）
def longest_consecutive(nums):
    num_set = set(nums)
    best = 0
    for n in num_set:
        if n - 1 not in num_set:   # 只从序列起点开始
            cur = n
            streak = 1
            while cur + 1 in num_set:
                cur += 1
                streak += 1
            best = max(best, streak)
    return best
```

**经典例题**
- **LC 1 两数之和**：单次遍历 + dict，O(n)。
- **LC 49 字母异位词分组**：排序后的字符串作 key。
- **LC 128 最长连续序列**：set + 从序列起点向右延伸。
- **LC 146 LRU 缓存**：OrderedDict 或 哈希表 + 双向链表。
- **LC 380 O(1) 时间插入、删除和获取随机元素**：dict(val→idx) + list 配合。

---

## 4. 单调栈 & 单调队列

### 4.1 单调栈

**适用场景**：求每个元素的下一个/上一个更大/更小元素；柱状图最大矩形；股票价格跨度。

**核心思想**：维护一个单调（递增或递减）的栈，当新元素破坏单调性时弹出并记录结果。

> 💡 **图示**

```text
 求下一个更大元素: [2, 1, 5, 3, 6]

 i=0: 栈=[2]
 i=1: 1<2, 栈=[2,1]
 i=2: 5>1, 弹1→res[1]=5; 5>2, 弹2→res[0]=5; 栈=[5]
 i=3: 3<5, 栈=[5,3]
 i=4: 6>3, 弹3→res[3]=6; 6>5, 弹5→res[2]=6; 栈=[6]

 结果: [5, 5, 6, 6, -1]
```

**解题模板**
```python
# 下一个更大元素（单调递减栈）
def next_greater_element(nums):
    n = len(nums)
    result = [-1] * n
    stack = []  # 存索引
    for i in range(n):
        # 当前元素比栈顶大，弹出并记录答案
        while stack and nums[i] > nums[stack[-1]]:
            idx = stack.pop()
            result[idx] = nums[i]
        stack.append(i)
    return result

# 柱状图中最大矩形（LC 84）
def largest_rectangle_in_histogram(heights):
    heights = [0] + heights + [0]  # 哨兵
    stack = [0]
    max_area = 0
    for i in range(1, len(heights)):
        while heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            w = i - stack[-1] - 1
            max_area = max(max_area, h * w)
        stack.append(i)
    return max_area
```

**经典例题**
- **LC 496 下一个更大元素 I**：单调递减栈 + 哈希映射。
- **LC 739 每日温度**：单调递减栈，弹出时记录等待天数。
- **LC 84 柱状图中最大的矩形**：哨兵 + 单调递增栈。
- **LC 85 最大矩形（矩阵）**：逐行转化为直方图，套LC 84。
- **LC 316 去除重复字母**：单调递增栈 + 剩余计数控制贪心。

---

### 4.2 单调队列

**适用场景**：固定窗口内的最大值/最小值，O(n) 解决滑动窗口极值。

**核心思想**：用双端队列维护候选索引，保证队列内对应值单调递减（求最大）；队头即为当前窗口最大值，过期则出队。

**解题模板**
```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()   # 存索引，对应值单调递减
    result = []
    for i, num in enumerate(nums):
        # 移除过期索引
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        # 维护单调性（移除比当前值小的尾部）
        while dq and nums[dq[-1]] < num:
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            result.append(nums[dq[0]])
    return result
```

**经典例题**
- **LC 239 滑动窗口最大值**：单调队列模板题。
- **LC 862 和至少为K的最短子数组**：前缀和 + 单调队列。
- **LC 1438 绝对差不超过限制的最长连续子数组**：两个单调队列分别维护最大最小值。

---

## 5. 二分查找

### 5.1 标准二分

**适用场景**：有序数组中查找特定值，O(log n)。

**核心思想**：每次排除一半搜索空间，不变量：目标在 `[left, right]` 内。

> 💡 **图示**

```text
 arr = [1, 3, 5, 7, 9, 11, 13]  target=7
        0  1  2  3  4   5   6

 lo=0, hi=6, mid=3 → arr[3]=7 ✓ 找到！

 若 arr[mid] < target → lo = mid+1  (搜右半)
 若 arr[mid] > target → hi = mid-1  (搜左半)
```

**解题模板**
```python
def binary_search(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

---

### 5.2 左/右边界二分

**解题模板**
```python
def left_bound(nums, target):
    """返回 target 第一次出现的索引（不存在返回-1）"""
    left, right = 0, len(nums)  # 左闭右开
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid     # 继续向左收缩
    return left if left < len(nums) and nums[left] == target else -1

def right_bound(nums, target):
    """返回 target 最后一次出现的索引"""
    left, right = 0, len(nums)
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] <= target:
            left = mid + 1
        else:
            right = mid
    pos = left - 1
    return pos if pos >= 0 and nums[pos] == target else -1
```

---

### 5.3 在答案空间二分（最小化最大值）

**适用场景**：答案本身是一个范围内的整数，且存在单调性（答案越大越容易满足条件）。

**解题模板**
```python
def minimize_max(data, limit):
    """在答案空间二分：最小化某个最大值"""
    def feasible(mid):
        # 判断答案为 mid 时是否可行（具体逻辑按题意填写）
        pass

    left, right = min_possible, max_possible
    while left < right:
        mid = left + (right - left) // 2
        if feasible(mid):
            right = mid
        else:
            left = mid + 1
    return left
```

**经典例题**
- **LC 704 二分查找**：标准模板题。
- **LC 34 在排序数组中查找元素的第一个和最后一个位置**：左右边界二分。
- **LC 875 爱吃香蕉的珂珂**：在答案空间二分速度。
- **LC 1011 在 D 天内送达包裹的能力**：二分最小载重能力。
- **LC 410 分割数组的最大值**：二分答案 + 贪心验证。

---

## 6. DFS & 回溯

**适用场景**：枚举所有可能的组合/排列/子集；路径搜索；需要"选择-探索-撤销"的问题。

**核心思想**：递归树枚举，进入时做选择，离开时撤销选择（回溯），通过剪枝减少无效分支。

> 💡 **图示**（全排列 `[1,2,3]` 的递归树）

<div class="mermaid">
graph TD
  A["[]"] --> B["[1]"]
  A --> C["[2]"]
  A --> D["[3]"]
  B --> E["[1,2]"]
  B --> F["[1,3]"]
  C --> G["[2,1]"]
  C --> H["[2,3]"]
  D --> I["[3,1]"]
  D --> J["[3,2]"]
  E --> K["[1,2,3]✓"]
  F --> L["[1,3,2]✓"]
  G --> M["[2,1,3]✓"]
  H --> N["[2,3,1]✓"]
  I --> O["[3,1,2]✓"]
  J --> P["[3,2,1]✓"]
  style K fill:#d4edda
  style L fill:#d4edda
  style M fill:#d4edda
  style N fill:#d4edda
  style O fill:#d4edda
  style P fill:#d4edda
</div>

**解题模板**
```python
# 通用回溯框架
def backtrack(path, choices):
    if 满足终止条件:
        result.append(path[:])  # 保存副本
        return
    for choice in choices:
        if 不合法:
            continue              # 剪枝
        path.append(choice)       # 做选择
        backtrack(path, 新的choices)
        path.pop()                # 撤销选择

# ---- 全排列（LC 46）----
def permute(nums):
    result = []
    def bt(path, used):
        if len(path) == len(nums):
            result.append(path[:])
            return
        for i, num in enumerate(nums):
            if used[i]: continue
            used[i] = True
            path.append(num)
            bt(path, used)
            path.pop()
            used[i] = False
    bt([], [False] * len(nums))
    return result

# ---- 组合（LC 77）----
def combine(n, k):
    result = []
    def bt(start, path):
        if len(path) == k:
            result.append(path[:])
            return
        # 剪枝：剩余元素不够凑 k 个时提前退出
        for i in range(start, n - (k - len(path)) + 2):
            path.append(i)
            bt(i + 1, path)
            path.pop()
    bt(1, [])
    return result

# ---- 子集（LC 78）----
def subsets(nums):
    result = []
    def bt(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            bt(i + 1, path)
            path.pop()
    bt(0, [])
    return result

# ---- N皇后（LC 51）----
def solve_n_queens(n):
    result = []
    cols = set(); diag1 = set(); diag2 = set()

    def bt(row, board):
        if row == n:
            result.append([''.join(r) for r in board])
            return
        for col in range(n):
            if col in cols or (row-col) in diag1 or (row+col) in diag2:
                continue
            cols.add(col); diag1.add(row-col); diag2.add(row+col)
            board[row][col] = 'Q'
            bt(row + 1, board)
            board[row][col] = '.'
            cols.remove(col); diag1.remove(row-col); diag2.remove(row+col)

    bt(0, [['.']*n for _ in range(n)])
    return result
```

**经典例题**
- **LC 46 全排列**：used数组去重。
- **LC 77 组合**：start参数控制不重复选取。
- **LC 78 子集**：每层都记录结果。
- **LC 39 组合总和**：可重复选，sum>target 时剪枝。
- **LC 51 N皇后**：三个 set 判断冲突，速度远快于矩阵遍历。

---

## 7. BFS

**适用场景**：无权图最短路径、层序遍历、多源扩散、拓扑排序。

**核心思想**：按层扩散，用队列保证先入先出，visited集合避免重复访问。

> 💡 **图示**（BFS 最短路径示例）

<div class="mermaid">
graph LR
  S((起点S)) -->|1| A((A))
  S -->|4| B((B))
  A -->|2| C((C))
  A -->|5| B
  B -->|1| E((终点E))
  C -->|1| E
  style S fill:#cce5ff
  style E fill:#d4edda
</div>

**解题模板**
```python
from collections import deque

# 单源 BFS 最短路
def bfs(graph, start, end):
    queue = deque([(start, 0)])  # (节点, 距离)
    visited = {start}
    while queue:
        node, dist = queue.popleft()
        if node == end:
            return dist
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    return -1

# 多源 BFS（同时从多个起点扩散）
def multi_source_bfs(grid, sources):
    queue = deque()
    visited = set()
    for s in sources:
        queue.append((s, 0))
        visited.add(s)
    while queue:
        pos, dist = queue.popleft()
        for nxt in get_neighbors(pos, grid):
            if nxt not in visited:
                visited.add(nxt)
                queue.append((nxt, dist + 1))
```

**拓扑排序（Kahn算法）**
```python
from collections import deque, defaultdict

def topological_sort(n, edges):
    """
    n: 节点数（0~n-1）
    edges: [(u, v)] 表示 u -> v
    返回拓扑序，若有环返回空列表
    """
    in_degree = [0] * n
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1

    queue = deque(i for i in range(n) if in_degree[i] == 0)
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nxt in graph[node]:
            in_degree[nxt] -= 1
            if in_degree[nxt] == 0:
                queue.append(nxt)
    return order if len(order) == n else []
```

**经典例题**
- **LC 127 单词接龙**：BFS 按层扩散，每次替换一个字符。
- **LC 994 腐烂的橘子**：多源 BFS，所有腐烂橘子同时扩散。
- **LC 207 课程表**：拓扑排序判断有无环。
- **LC 210 课程表 II**：返回拓扑序。
- **LC 1162 地图分析**：多源 BFS 求最远陆地。

---

## 8. 动态规划

### 8.1 线性 DP

**适用场景**：状态可以从前几个状态转移；最优子结构明显。

**最长递增子序列（LIS）**
```python
# O(n^2) DP
def length_of_lis(nums):
    n = len(nums)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

# O(n log n) 贪心 + 二分
import bisect
def length_of_lis_fast(nums):
    tails = []
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    return len(tails)

# 编辑距离（LC 72）
def min_distance(word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1): dp[i][0] = i
    for j in range(n + 1): dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j],    # 删除
                                   dp[i][j-1],    # 插入
                                   dp[i-1][j-1])  # 替换
    return dp[m][n]
```

**经典例题**
- **LC 300 最长递增子序列**：O(n²) DP 或 O(n log n) 贪心+二分。
- **LC 72 编辑距离**：二维 DP，三种操作转移。
- **LC 1143 最长公共子序列**：`dp[i][j]` 以两串各前i/j字符为结尾的LCS长度。
- **LC 5 最长回文子串**：区间DP或中心扩展O(n²)。

---

### 8.2 背包问题

> 💡 **图示**（0/1背包状态转移）

<div class="mermaid">
graph LR
  A["dp[i-1][j]<br/>不选第i件"] --> C["dp[i][j]<br/>= max(A, B)"]
  B["dp[i-1][j-w]+v<br/>选第i件"] --> C
  style C fill:#fff3cd
</div>

```text
       容量j: 0  1  2  3  4  5
 item0(w=2,v=3): 0  0  3  3  3  3
 item1(w=3,v=4): 0  0  3  4  4  7
 item2(w=4,v=5): 0  0  3  4  5  7
```

**解题模板**
```python
# 0/1 背包（每件物品最多选一次）
def knapsack_01(weights, values, W):
    n = len(weights)
    dp = [0] * (W + 1)  # 空间压缩为一维
    for i in range(n):
        for w in range(W, weights[i] - 1, -1):  # 逆序！防止重复选
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    return dp[W]

# 完全背包（每件物品可选无限次）
def knapsack_complete(weights, values, W):
    dp = [0] * (W + 1)
    for i in range(len(weights)):
        for w in range(weights[i], W + 1):  # 正序！允许重复选
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    return dp[W]
```

**经典例题**
- **LC 416 分割等和子集**：0/1背包，背包容量 = sum//2，物品重量=值。
- **LC 322 零钱兑换**：完全背包，求凑成 amount 的最少硬币数。
- **LC 518 零钱兑换 II**：完全背包，求凑成 amount 的方案数。
- **LC 474 一和零**：二维0/1背包（容量是 m 个0和 n 个1）。

---

### 8.3 区间 DP

**适用场景**：问题答案与子区间的合并方式有关（如矩阵链乘、气球爆破）。

**解题模板**
```python
def interval_dp(arr):
    n = len(arr)
    dp = [[0] * n for _ in range(n)]
    # 枚举区间长度
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = float('inf')
            for k in range(i, j):  # 分割点
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i, k, j))
    return dp[0][n-1]
```

**经典例题**
- **LC 312 戳气球**：区间DP，`dp[i][j]` = 戳完 `(i,j)` 内气球的最大硬币数。
- **LC 1039 多边形三角剖分的最低得分**：区间DP求最小权值三角剖分。
- **LC 516 最长回文子序列**：区间DP，`dp[i][j]` 为子串中最长回文子序列长度。

---

### 8.4 状态压缩 DP

**适用场景**：集合/路径选择问题，状态集合可用整数位掩码表示（n ≤ 20）。

**解题模板**
```python
# 旅行商问题（TSP）/ 最短哈密顿路径
def tsp(dist, n):
    INF = float('inf')
    # dp[mask][i] = 访问了 mask 中的城市，且最后停在城市 i 的最短路
    dp = [[INF] * n for _ in range(1 << n)]
    dp[1][0] = 0  # 从城市0出发
    for mask in range(1 << n):
        for u in range(n):
            if dp[mask][u] == INF: continue
            if not (mask >> u & 1): continue
            for v in range(n):
                if mask >> v & 1: continue
                new_mask = mask | (1 << v)
                dp[new_mask][v] = min(dp[new_mask][v], dp[mask][u] + dist[u][v])
    full = (1 << n) - 1
    return min(dp[full][i] + dist[i][0] for i in range(n))
```

**经典例题**
- **LC 847 访问所有节点的最短路径**：状压DP + BFS。
- **LC 691 贴纸拼词**：状压DP，用位掩码表示已覆盖的字符。
- **LC 1986 完成任务的最少工作时间段**：状压DP枚举子集划分。

---

### 8.5 DP 优化技巧

```python
# 滚动数组（二维DP压缩为一维或两行）
# 将 dp[i][j] 依赖关系分析清楚，只保留必要的行

# 空间压缩示例：LCS
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp = [0] * (n + 1)
    for i in range(1, m + 1):
        prev = 0
        for j in range(1, n + 1):
            temp = dp[j]
            if s1[i-1] == s2[j-1]:
                dp[j] = prev + 1
            else:
                dp[j] = max(dp[j], dp[j-1])
            prev = temp
    return dp[n]
```

---

## 9. 贪心

**适用场景**：局部最优能推出全局最优的问题；通常需要证明（反证法或交换论证）。

**核心思想**：每步都做当前看起来最好的选择，且该局部选择不影响后续的最优性。

> 💡 **图示**（区间调度贪心）

```text
 活动（按结束时间排序）:
 [1,3]  ████
 [2,4]   ████
 [3,5]    ████
 [4,6]     ████
 [1,6]  ██████████

 贪心选择:
 选[1,3] → 选[3,5] → 选[5,...]
 最多选不重叠区间数 = 3
```

**解题模板**
```python
# 区间调度（活动选择，LC 435 变体）
def erase_overlap_intervals(intervals):
    """最少删除多少个区间使剩余不重叠"""
    intervals.sort(key=lambda x: x[1])  # 按结束时间排序！
    end = float('-inf')
    count = 0
    for start, finish in intervals:
        if start >= end:
            end = finish
        else:
            count += 1  # 删除当前区间（贪心：保留结束早的）
    return count

# 跳跃游戏（LC 45）
def jump(nums):
    """最少跳跃次数到达末尾"""
    jumps = 0
    cur_end = 0   # 当前跳跃可达的最远位置
    farthest = 0  # 下一跳可达的最远位置
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        if i == cur_end:
            jumps += 1
            cur_end = farthest
    return jumps
```

**何时用贪心（证明思路）**
1. **贪心选择性质**：每步的贪心选择不会影响剩余子问题的最优解。
2. **最优子结构**：全局最优包含子问题最优。
3. **验证方法**：交换论证（假设最优解与贪心解在某步不同，交换后不变差）或数学归纳。

**经典例题**
- **LC 455 分发饼干**：小饼干优先满足小胃口，双指针贪心。
- **LC 45 跳跃游戏 II**：每次在当前可达范围内选最远跳。
- **LC 435 无重叠区间**：按结束时间排序，贪心保留结束早的区间。
- **LC 763 划分字母区间**：记录每字母最远出现位置，贪心确定区间端点。
- **LC 406 根据身高重建队列**：按身高降序+k升序排列后依次插入。

---

## 10. 分治

**适用场景**：问题可拆成规模更小的同类子问题，子问题独立，合并结果有规律。

**核心思想**：分（divide）→ 治（conquer）→ 合（merge）。

**解题模板**
```python
# 归并排序（统计逆序对，LC 315/493）
def merge_sort_count(nums):
    """返回 (排序后的数组, 逆序对数)"""
    if len(nums) <= 1:
        return nums, 0
    mid = len(nums) // 2
    left, l_count = merge_sort_count(nums[:mid])
    right, r_count = merge_sort_count(nums[mid:])
    merged = []
    count = l_count + r_count
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i]); i += 1
        else:
            # left[i..] 都大于 right[j]，各自构成逆序对
            count += len(left) - i
            merged.append(right[j]); j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged, count

# 快速选择（第K大/小元素，LC 215）
import random
def find_kth_largest(nums, k):
    """找第k大元素，平均 O(n)"""
    def partition(lo, hi):
        pivot_idx = random.randint(lo, hi)
        nums[pivot_idx], nums[hi] = nums[hi], nums[pivot_idx]
        pivot = nums[hi]
        p = lo
        for i in range(lo, hi):
            if nums[i] <= pivot:
                nums[p], nums[i] = nums[i], nums[p]
                p += 1
        nums[p], nums[hi] = nums[hi], nums[p]
        return p

    target = len(nums) - k  # 第k大 = 第(n-k)小
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        p = partition(lo, hi)
        if p == target: break
        elif p < target: lo = p + 1
        else: hi = p - 1
    return nums[target]
```

**经典例题**
- **LC 148 排序链表**：归并排序链表，O(n log n) 空间 O(log n)。
- **LC 315 计算右侧小于当前元素的个数**：归并排序统计逆序对变体。
- **LC 215 数组中的第K个最大元素**：快速选择平均 O(n)。
- **LC 4 寻找两个正序数组的中位数**：分治二分，O(log(m+n))。

---

## 11. 树的技巧

### 11.1 递归三要素

1. **函数定义**：明确函数做什么（接收节点，返回什么）。
2. **递归终止**：null 节点或叶节点的处理。
3. **单层逻辑**：当前节点做什么，如何利用子树结果。

> 💡 **图示**（二叉树递归结构）

<div class="mermaid">
graph TD
  R((根节点)) --> L((左子树))
  R --> Ri((右子树))
  L --> LL((左左))
  L --> LR((左右))
  Ri --> RL((右左))
  Ri --> RR((右右))
  style R fill:#cce5ff
  style L fill:#d4edda
  style Ri fill:#d4edda
</div>

**解题模板**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# 前序/中序/后序（迭代版本）
def inorder_iterative(root):
    stack, result = [], []
    cur = root
    while cur or stack:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        result.append(cur.val)
        cur = cur.right
    return result
```

---

### 11.2 最近公共祖先（LCA）

```python
def lowest_common_ancestor(root, p, q):
    if not root or root is p or root is q:
        return root
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    if left and right:
        return root   # p, q 分布在左右子树
    return left or right
```

---

### 11.3 树形 DP

**适用场景**：以每个节点为根的子树计算最优值，父节点结果依赖子树。

```python
# 二叉树最大路径和（LC 124）
def max_path_sum(root):
    max_sum = float('-inf')
    def dfs(node):
        nonlocal max_sum
        if not node: return 0
        left = max(dfs(node.left), 0)   # 负增益则不选
        right = max(dfs(node.right), 0)
        # 经过当前节点的路径
        max_sum = max(max_sum, node.val + left + right)
        # 向父节点只能返回一侧
        return node.val + max(left, right)
    dfs(root)
    return max_sum

# 打家劫舍 III（LC 337）- 树形DP
def rob(root):
    def dfs(node):
        if not node: return (0, 0)  # (不选node, 选node)
        l_no, l_yes = dfs(node.left)
        r_no, r_yes = dfs(node.right)
        no = max(l_no, l_yes) + max(r_no, r_yes)  # 不选当前，子节点随意
        yes = node.val + l_no + r_no               # 选当前，子节点不能选
        return (no, yes)
    return max(dfs(root))
```

**经典例题**
- **LC 104 二叉树的最大深度**：递归 `1 + max(left, right)`。
- **LC 236 二叉树的最近公共祖先**：后序DFS，左右同时找到则当前节点即为LCA。
- **LC 124 二叉树中的最大路径和**：树形DP，每个节点维护单侧最大贡献。
- **LC 337 打家劫舍 III**：树形DP，每节点返回选/不选两个状态。
- **LC 543 二叉树的直径**：`left_depth + right_depth` 的最大值。

---

## 12. 图论

### 12.1 并查集

**适用场景**：动态连通性、环检测、最小生成树（Kruskal）。

> 💡 **图示**（并查集合并与路径压缩）

```text
 初始: 1  2  3  4  5  (各自为根)

 union(1,2): 1←2    union(3,4): 3←4    union(1,3): 1←3
      1            3←4               1
      ↑            ↑                 ↑
      2            4               3   2
                                   ↑
                                   4
 find(4) → 4→3→1 (路径压缩后: 4→1)
```

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.count = n  # 连通分量数

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # 路径压缩
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False  # 已在同一集合（说明有环）
        # 按秩合并
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        self.count -= 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

**经典例题**
- **LC 200 岛屿数量**：DFS/BFS 或并查集均可。
- **LC 547 省份数量**：并查集统计连通分量数。
- **LC 684 冗余连接**：并查集检测环，添加边时若已连通则该边冗余。
- **LC 1584 连接所有点的最小费用**：Kruskal 最小生成树。

---

### 12.2 Dijkstra（单源最短路）

**适用场景**：带权有向/无向图，边权非负，求从源点到所有节点的最短距离。

```python
import heapq
from collections import defaultdict

def dijkstra(n, edges, src):
    """
    n: 节点数（0-indexed）
    edges: [(u, v, w)]
    返回 dist 数组
    """
    graph = defaultdict(list)
    for u, v, w in edges:
        graph[u].append((v, w))
        graph[v].append((u, w))  # 无向图

    dist = [float('inf')] * n
    dist[src] = 0
    heap = [(0, src)]  # (距离, 节点)

    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:
            continue  # 过期状态，跳过
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))
    return dist
```

**经典例题**
- **LC 743 网络延迟时间**：Dijkstra 单源最短路，返回 max(dist)。
- **LC 1631 最小体力消耗路径**：Dijkstra（最小化路径上最大差值）。
- **LC 787 K 站中转内最便宜的航班**：Bellman-Ford（限制边数）或 Dijkstra+层数。

---

### 12.3 拓扑排序（汇总）

拓扑排序详见 [§7 BFS 中的 Kahn 算法](#拓扑排序kahn算法)。

**DFS 版拓扑排序**：
```python
def topo_dfs(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)

    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n
    order = []
    has_cycle = [False]

    def dfs(u):
        if has_cycle[0]: return
        color[u] = GRAY
        for v in graph[u]:
            if color[v] == GRAY:
                has_cycle[0] = True; return
            if color[v] == WHITE:
                dfs(v)
        color[u] = BLACK
        order.append(u)

    for i in range(n):
        if color[i] == WHITE:
            dfs(i)

    return [] if has_cycle[0] else order[::-1]
```

**经典例题**
- **LC 207 课程表**：Kahn 算法检测有无环。
- **LC 210 课程表 II**：返回拓扑序。
- **LC 269 火星词典**：从字典构建偏序关系，拓扑排序。

---

## 总结速查表

| 技巧 | 时间复杂度 | 关键词 |
|------|-----------|--------|
| 对撞指针 | O(n) | 有序数组、两端夹逼 |
| 快慢指针 | O(n) | 链表环、中点 |
| 滑动窗口 | O(n) | 连续子数组、子串 |
| 前缀和 | O(1)查询 | 区间求和、子数组和为k |
| 差分数组 | O(1)更新 | 区间批量加减 |
| 哈希表 | O(1)均摊 | 计数、Two Sum |
| 单调栈 | O(n) | 下一个更大元素、矩形面积 |
| 单调队列 | O(n) | 滑动窗口最大值 |
| 二分查找 | O(log n) | 有序、答案空间单调 |
| 回溯 | O(n! ~ 2^n) | 排列/组合/子集 |
| BFS | O(V+E) | 最短路、层序、拓扑 |
| 线性DP | O(n²) | LIS、编辑距离 |
| 背包DP | O(nW) | 0/1背包、完全背包 |
| 贪心 | O(n log n) | 区间调度、跳跃游戏 |
| 快速选择 | O(n)均摊 | 第K大元素 |
| 并查集 | O(α(n))≈O(1) | 连通性、环检测 |
| Dijkstra | O((V+E)log V) | 非负权最短路 |
