---
layout: default
title: LeetCode 高频100题总结
permalink: /notes/leetcode-top100/
---

# LeetCode 高频100题总结

> 按题型分类整理，每题含**完整题目描述**、核心思路、复杂度分析与 Python 代码模板。

---

## 目录

- [数组 / 双指针 / 滑动窗口](#一数组--双指针--滑动窗口)
- [链表](#二链表)
- [二叉树 / DFS / BFS](#三二叉树--dfs--bfs)
- [动态规划](#四动态规划)
- [栈 / 队列 / 单调栈](#五栈--队列--单调栈)
- [哈希表](#六哈希表)
- [二分查找](#七二分查找)
- [回溯](#八回溯)
- [贪心](#九贪心)
- [图 / 拓扑排序](#十图--拓扑排序)

---

## 一、数组 / 双指针 / 滑动窗口

### [1. 两数之和](https://leetcode.cn/problems/two-sum/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出**和为目标值** `target` 的那**两个**整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。你可以按任意顺序返回答案。

**示例：**
```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9，返回 [0, 1]。

输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**约束：**
- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- 只会存在一个有效答案

#### 核心思路
用哈希表存已遍历元素，对每个数查找 `target - num` 是否存在。一次遍历即可，避免暴力 O(n²)。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def twoSum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
```

---

### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个长度为 `n` 的整数数组 `height`。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])`。

找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。返回容器可以储存的最大水量。

**说明：** 你不能倾斜容器。

**示例：**
```
输入：height = [1,8,6,2,5,4,8,3,7]
输出：49
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。
在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

输入：height = [1,1]
输出：1
```

**约束：**
- `n == height.length`
- `2 <= n <= 10^5`
- `0 <= height[i] <= 10^4`

#### 核心思路
双指针从两端向中间收缩，每次移动较短的那端（移长端只会让面积变小）。贪心地保留可能更大的那端。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def maxArea(height):
    l, r = 0, len(height) - 1
    res = 0
    while l < r:
        res = max(res, min(height[l], height[r]) * (r - l))
        if height[l] < height[r]:
            l += 1
        else:
            r -= 1
    return res
```

---

### [15. 三数之和](https://leetcode.cn/problems/3sum/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k`，同时还满足 `nums[i] + nums[j] + nums[k] == 0`。请你返回所有和为 `0` 且不重复的三元组。

**注意：** 答案中不可以包含重复的三元组。

**示例：**
```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0
     nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0
     nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0

输入：nums = [0,1,1]
输出：[]

输入：nums = [0,0,0]
输出：[[0,0,0]]
```

**约束：**
- `3 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`

#### 核心思路
排序后固定一个数，对剩余部分用双指针求两数之和。注意跳过重复元素以避免重复三元组。

#### 复杂度
- 时间：O(n²)
- 空间：O(1)

#### 代码模板
```python
def threeSum(nums):
    nums.sort()
    res = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]:
            continue
        l, r = i + 1, len(nums) - 1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s == 0:
                res.append([nums[i], nums[l], nums[r]])
                while l < r and nums[l] == nums[l+1]: l += 1
                while l < r and nums[r] == nums[r-1]: r -= 1
                l += 1; r -= 1
            elif s < 0: l += 1
            else: r -= 1
    return res
```

---

### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个字符串 `s`，请你找出其中不含有重复字符的**最长子串**的长度。

**示例：**
```
输入：s = "abcabcbb"
输出：3
解释：因为无重复字符的最长子串是 "abc"，所以其长度为 3。

输入：s = "bbbbb"
输出：1
解释：因为无重复字符的最长子串是 "b"，所以其长度为 1。

输入：s = "pwwkew"
输出：3
解释：因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

**约束：**
- `0 <= s.length <= 5 * 10^4`
- `s` 由英文字母、数字、符号和空格组成

#### 核心思路
滑动窗口 + 哈希集合，右指针扩展窗口，遇到重复字符则左指针收缩直到窗口合法。

#### 复杂度
- 时间：O(n)
- 空间：O(min(m,n))

#### 代码模板
```python
def lengthOfLongestSubstring(s):
    seen = {}
    l = res = 0
    for r, c in enumerate(s):
        if c in seen and seen[c] >= l:
            l = seen[c] + 1
        seen[c] = r
        res = max(res, r - l + 1)
    return res
```

---

### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**示例：**
```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，
在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

输入：height = [4,2,0,3,2,5]
输出：9
```

**约束：**
- `n == height.length`
- `1 <= n <= 2 * 10^4`
- `0 <= height[i] <= 10^5`

#### 核心思路
双指针法，维护左右最大高度，较小一侧可以确定积水量（等于该侧最大值减去当前高度）。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def trap(height):
    l, r = 0, len(height) - 1
    left_max = right_max = res = 0
    while l < r:
        if height[l] < height[r]:
            if height[l] >= left_max: left_max = height[l]
            else: res += left_max - height[l]
            l += 1
        else:
            if height[r] >= right_max: right_max = height[r]
            else: res += right_max - height[r]
            r -= 1
    return res
```

---

### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组** 是数组中的一个连续部分。

**示例：**
```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6。

输入：nums = [1]
输出：1

输入：nums = [5,4,-1,7,8]
输出：23
```

**约束：**
- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

#### 核心思路
Kadane 算法，维护以当前元素结尾的最大子数组和。若前缀和为负则舍弃，从当前元素重新开始。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def maxSubArray(nums):
    cur = res = nums[0]
    for n in nums[1:]:
        cur = max(n, cur + n)
        res = max(res, cur)
    return res
```

---

### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个数组 `prices`，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择**某一天**买入这只股票，并选择在**未来的某一个不同的日子**卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0`。

**示例：**
```
输入：prices = [7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，
最大利润 = 6-1 = 5。注意利润不能是 7-1 = 6，因为卖出价格需要大于买入价格；
同时，你不能在买入前卖出股票。

输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下，没有交易完成，所以最大利润为 0。
```

**约束：**
- `1 <= prices.length <= 10^5`
- `0 <= prices[i] <= 10^4`

#### 核心思路
一次遍历，记录历史最低买入价，对每天计算当天卖出的利润，取最大值。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def maxProfit(prices):
    min_price, max_profit = float('inf'), 0
    for p in prices:
        min_price = min(min_price, p)
        max_profit = max(max_profit, p - min_price)
    return max_profit
```

---

### [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次**旋转**后，得到输入数组。例如，原数组 `nums = [0,1,2,4,5,6,7]` 在变化后可能得到：
- 若旋转 `4` 次，则可以得到 `[4,5,6,7,0,1,2]`
- 若旋转 `7` 次，则可以得到 `[0,1,2,4,5,6,7]`

注意，数组 `[a[0], a[1], a[2], ..., a[n-1]]` **旋转一次** 的结果为数组 `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]`。

给你一个元素值**互不相同**的数组 `nums`，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的**最小元素**。你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

**示例：**
```
输入：nums = [3,4,5,1,2]
输出：1
解释：原数组为 [1,2,3,4,5]，旋转 3 次得到输入数组。

输入：nums = [4,5,6,7,0,1,2]
输出：0
解释：原数组为 [0,1,2,4,5,6,7]，旋转 4 次得到输入数组。

输入：nums = [11,13,15,17]
输出：11
解释：原数组为 [11,13,15,17]，旋转 4 次得到输入数组。
```

**约束：**
- `n == nums.length`
- `1 <= n <= 5000`
- `-5000 <= nums[i] <= 5000`
- `nums` 中的所有整数**互不相同**
- `nums` 原来是一个升序排序的数组，并进行了 `1` 至 `n` 次旋转

#### 核心思路
二分比较 mid 与 right，若 `nums[mid] > nums[right]` 则最小值在右半边，否则在左半边（含 mid）。

#### 复杂度
- 时间：O(log n)
- 空间：O(1)

#### 代码模板
```python
def findMin(nums):
    l, r = 0, len(nums) - 1
    while l < r:
        mid = (l + r) // 2
        if nums[mid] > nums[r]: l = mid + 1
        else: r = mid
    return nums[l]
```

---

### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个含有 `n` 个正整数的数组和一个正整数 `target`。

找出该数组中满足其总和大于等于 `target` 的长度最小的**连续子数组** `[numsl, numsl+1, ..., numsr-1, numsr]`，并返回其长度。如果不存在符合条件的子数组，返回 `0`。

**示例：**
```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。

输入：target = 4, nums = [1,4,4]
输出：1

输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

**约束：**
- `1 <= target <= 10^9`
- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^4`

#### 核心思路
滑动窗口，右指针不断扩展，当窗口和 >= target 时，左指针收缩并更新最小长度，直到窗口和不满足条件。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def minSubArrayLen(target, nums):
    l = 0
    total = 0
    res = float('inf')
    for r in range(len(nums)):
        total += nums[r]
        while total >= target:
            res = min(res, r - l + 1)
            total -= nums[l]
            l += 1
    return res if res != float('inf') else 0
```

---

### [238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，返回 数组 `answer`，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积。

题目数据**保证**数组 `nums` 之中任意元素的全部前缀元素和后缀的乘积都在**32 位**整数范围内。

请**不要使用除法**，且在 `O(n)` 时间复杂度内完成此题。

**示例：**
```
输入：nums = [1,2,3,4]
输出：[24,12,8,6]

输入：nums = [-1,1,0,-3,3]
输出：[0,0,9,0,0]
```

**约束：**
- `2 <= nums.length <= 10^5`
- `-30 <= nums[i] <= 30`
- **保证** 数组 `nums` 之中任意元素的全部前缀元素和后缀的乘积都在  **32 位** 整数范围内

#### 核心思路
先从左到右计算前缀积，再从右到左乘以后缀积，两趟遍历不使用除法。

#### 复杂度
- 时间：O(n)
- 空间：O(1)（输出数组不计）

#### 代码模板
```python
def productExceptSelf(nums):
    n = len(nums)
    res = [1] * n
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]
    suffix = 1
    for i in range(n-1, -1, -1):
        res[i] *= suffix
        suffix *= nums[i]
    return res
```

---

## 二、链表

### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

将两个升序链表合并为一个新的**升序**链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

**示例：**
```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]

输入：l1 = [], l2 = []
输出：[]

输入：l1 = [], l2 = [0]
输出：[0]
```

**约束：**
- 两个链表的节点数目范围是 `[0, 50]`
- `-100 <= Node.val <= 100`
- `l1` 和 `l2` 均按**非递减顺序**排列

#### 核心思路
使用哑节点，逐一比较两链表头节点大小，将较小的接入结果链表，直到一方耗尽后追加剩余部分。

#### 复杂度
- 时间：O(m+n)
- 空间：O(1)

#### 代码模板
```python
def mergeTwoLists(l1, l2):
    dummy = cur = ListNode(0)
    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1; l1 = l1.next
        else:
            cur.next = l2; l2 = l2.next
        cur = cur.next
    cur.next = l1 or l2
    return dummy.next
```

---

### [23. 合并K个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

**示例：**
```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到：
1->1->2->3->4->4->5->6

输入：lists = []
输出：[]

输入：lists = [[]]
输出：[]
```

**约束：**
- `k == lists.length`
- `0 <= k <= 10^4`
- `0 <= lists[i].length <= 500`
- `-10^4 <= lists[i][j] <= 10^4`
- `lists[i]` 按**升序**排列
- `lists[i].length` 的总和不超过 `10^4`

#### 核心思路
使用最小堆（优先队列）存各链表当前头节点，每次弹出最小值并将其后继入堆。避免两两合并的重复遍历。

#### 复杂度
- 时间：O(N log k)
- 空间：O(k)

#### 代码模板
```python
import heapq
def mergeKLists(lists):
    dummy = cur = ListNode(0)
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    while heap:
        val, i, node = heapq.heappop(heap)
        cur.next = node; cur = cur.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

---

### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给你一个链表的头节点 `head`，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递**。仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 `true`。否则，返回 `false`。

**示例：**
```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。

输入：head = [1,2], pos = 0
输出：true
解释：链表中有一个环，其尾部连接到第一个节点。

输入：head = [1], pos = -1
输出：false
解释：链表中没有环。
```

**约束：**
- 链表中节点的数目范围是 `[0, 10^4]`
- `-10^5 <= Node.val <= 10^5`
- `pos` 为 `-1` 或者链表中的一个**有效索引**

#### 核心思路
Floyd 快慢指针，慢指针每次走一步，快指针走两步。若有环则两者必相遇，否则快指针先到达 null。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def hasCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

---

### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个链表的头节点 `head`，返回链表开始入环的第一个节点。如果链表无环，则返回 `null`。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改**链表。

**示例：**
```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。

输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。

输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```

**约束：**
- 链表中节点的数目范围在范围 `[0, 10^4]` 内
- `-10^5 <= Node.val <= 10^5`
- `pos` 的值为 `-1` 或者链表中的一个有效索引

#### 核心思路
快慢指针相遇后，将一个指针移回 head，两个指针同速前进，再次相遇处即为环入口（数学推导）。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def detectCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next; fast = fast.next.next
        if slow == fast:
            slow = head
            while slow != fast:
                slow = slow.next; fast = fast.next
            return slow
    return None
```

---

### [148. 排序链表](https://leetcode.cn/problems/sort-list/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你链表的头结点 `head`，请你对链表进行排序并返回排序后链表的头结点。

请你在 `O(n log n)` 时间复杂度和常数级空间复杂度下，对链表进行排序。

**示例：**
```
输入：head = [4,2,1,3]
输出：[1,2,3,4]

输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]

输入：head = []
输出：[]
```

**约束：**
- 链表中节点的数目在范围 `[0, 5 * 10^4]` 内
- `-10^5 <= Node.val <= 10^5`

#### 核心思路
归并排序：用快慢指针找中点，将链表分为两半，递归排序后合并。自底向上合并可实现 O(1) 空间。

#### 复杂度
- 时间：O(n log n)
- 空间：O(log n)（递归栈）

#### 代码模板
```python
def sortList(head):
    if not head or not head.next:
        return head
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next; fast = fast.next.next
    mid = slow.next; slow.next = None
    left = sortList(head)
    right = sortList(mid)
    dummy = cur = ListNode(0)
    while left and right:
        if left.val <= right.val:
            cur.next = left; left = left.next
        else:
            cur.next = right; right = right.next
        cur = cur.next
    cur.next = left or right
    return dummy.next
```

---

### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给你两个单链表的头节点 `headA` 和 `headB`，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null`。

题目数据**保证**整个链式结构中不存在环。

**注意**，函数返回结果后，链表必须**保持其原始结构**。

**示例：**
```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5], skipA = 2, skipB = 3
输出：Intersected at '8'
解释：相交节点的值为 8（注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,6,1,8,4,5]。
在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。

输入：intersectVal = 2, listA = [1,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Intersected at '2'

输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
```

**约束：**
- `listA` 中节点数目为 `m`
- `listB` 中节点数目为 `n`
- `1 <= m, n <= 3 * 10^4`
- `1 <= Node.val <= 10^5`
- `0 <= skipA <= m`
- `0 <= skipB <= n`
- 如果 `listA` 和 `listB` 没有交点，`intersectVal` 为 `0`
- 如果 `listA` 和 `listB` 有交点，`intersectVal == listA[skipA] == listB[skipB]`

#### 核心思路
两指针分别遍历 A+B 和 B+A，路径总长相同，若有交点必在同一位置相遇。

#### 复杂度
- 时间：O(m+n)
- 空间：O(1)

#### 代码模板
```python
def getIntersectionNode(headA, headB):
    a, b = headA, headB
    while a != b:
        a = a.next if a else headB
        b = b.next if b else headA
    return a
```

---

### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给你单链表的头节点 `head`，请你反转链表，并返回反转后的链表。

**示例：**
```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]

输入：head = [1,2]
输出：[2,1]

输入：head = []
输出：[]
```

**约束：**
- 链表中节点的数目范围是 `[0, 5000]`
- `-5000 <= Node.val <= 5000`

**进阶：** 链表可以选用迭代或递归方式完成反转。你能否用两种方法解决这道题？

#### 核心思路
迭代法，维护 prev 和 cur 两个指针，逐一将 cur.next 指向 prev，向前推进。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def reverseList(head):
    prev, cur = None, head
    while cur:
        nxt = cur.next
        cur.next = prev
        prev = cur; cur = nxt
    return prev
```

---

## 三、二叉树 / DFS / BFS

### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个二叉树的根节点 `root`，返回它的**中序**遍历。

**示例：**
```
输入：root = [1,null,2,3]
输出：[1,3,2]

输入：root = []
输出：[]

输入：root = [1]
输出：[1]
```

**约束：**
- 树中节点数目在范围 `[0, 100]` 内
- `-100 <= Node.val <= 100`

**进阶：** 递归算法很简单，你可以通过迭代算法完成吗？

#### 核心思路
递归或迭代（用栈模拟）：左→根→右。迭代时用栈存路径，先一路向左压栈，弹出时访问，再转向右子树。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def inorderTraversal(root):
    res, stack = [], []
    cur = root
    while cur or stack:
        while cur:
            stack.append(cur); cur = cur.left
        cur = stack.pop()
        res.append(cur.val)
        cur = cur.right
    return res
```

---

### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你二叉树的根节点 `root`，返回其节点值的**层序遍历**。（即逐层地，从左到右访问所有节点）。

**示例：**
```
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[9,20],[15,7]]

输入：root = [1]
输出：[[1]]

输入：root = []
输出：[]
```

**约束：**
- 树中节点数目在范围 `[0, 2000]` 内
- `-1000 <= Node.val <= 1000`

#### 核心思路
BFS，使用队列逐层处理，每层开始时记录当前队列长度，弹出该层所有节点并将子节点入队。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
from collections import deque
def levelOrder(root):
    if not root: return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        res.append(level)
    return res
```

---

### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个二叉树 `root`，返回其最大深度。

二叉树的**最大深度**是指从根节点到最远叶子节点的最长路径上的节点数。

**示例：**
```
输入：root = [3,9,20,null,null,15,7]
输出：3

输入：root = [1,null,2]
输出：2
```

**约束：**
- 树中节点的数量在 `[0, 10^4]` 区间内
- `-100 <= Node.val <= 100`

#### 核心思路
递归，树的最大深度等于左右子树最大深度的较大值加一。空节点深度为 0。

#### 复杂度
- 时间：O(n)
- 空间：O(h)

#### 代码模板
```python
def maxDepth(root):
    if not root: return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

---

### [105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定两个整数数组 `preorder` 和 `inorder`，其中 `preorder` 是二叉树的**先序遍历**，`inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

**示例：**
```
输入：preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出：[3,9,20,null,null,15,7]

输入：preorder = [-1], inorder = [-1]
输出：[-1]
```

**约束：**
- `1 <= preorder.length <= 3000`
- `inorder.length == preorder.length`
- `-3000 <= preorder[i], inorder[i] <= 3000`
- `preorder` 和 `inorder` 均**无重复**元素
- `inorder` 均出现在 `preorder`
- `preorder` **保证**为二叉树的前序遍历序列
- `inorder` **保证**为二叉树的中序遍历序列

#### 核心思路
前序第一个元素为根，在中序中找到根的位置，左侧为左子树，右侧为右子树，递归构建。用哈希表加速查找。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def buildTree(preorder, inorder):
    idx = {v: i for i, v in enumerate(inorder)}
    def dfs(pre_l, pre_r, in_l, in_r):
        if pre_l > pre_r: return None
        root = TreeNode(preorder[pre_l])
        mid = idx[preorder[pre_l]]
        left_size = mid - in_l
        root.left = dfs(pre_l+1, pre_l+left_size, in_l, mid-1)
        root.right = dfs(pre_l+left_size+1, pre_r, mid+1, in_r)
        return root
    return dfs(0, len(preorder)-1, 0, len(inorder)-1)
```

---

### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你二叉树的根结点 `root`，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode`，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null`。
- 展开后的单链表应该与二叉树**先序遍历**顺序相同。

**示例：**
```
输入：root = [1,2,5,3,4,null,6]
输出：[1,null,2,null,3,null,4,null,5,null,6]

输入：root = []
输出：[]

输入：root = [0]
输出：[0]
```

**约束：**
- 树中节点数在范围 `[0, 2000]` 内
- `-100 <= Node.val <= 100`

**进阶：** 你可以使用原地算法（`O(1)` 额外空间）展开这棵树吗？

#### 核心思路
找到左子树的最右节点（前驱节点），将右子树接到其后，再将左子树移到右侧，重复处理。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def flatten(root):
    cur = root
    while cur:
        if cur.left:
            pre = cur.left
            while pre.right:
                pre = pre.right
            pre.right = cur.right
            cur.right = cur.left
            cur.left = None
        cur = cur.right
```

---

### [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

二叉树中的**路径**被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中**至多出现一次**。该路径**至少包含一个**节点，且不一定经过根节点。

**路径和**是路径中各节点值的总和。

给你一个二叉树的根节点 `root`，返回其**最大路径和**。

**示例：**
```
输入：root = [1,2,3]
输出：6
解释：最优路径是 2 -> 1 -> 3，路径和为 2 + 1 + 3 = 6

输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 -> 20 -> 7，路径和为 15 + 20 + 7 = 42
```

**约束：**
- 树中节点数目范围是 `[1, 3 * 10^4]`
- `-1000 <= Node.val <= 1000`

#### 核心思路
DFS 返回"以当前节点为端点的最大单侧路径和"，同时更新全局答案（左贡献 + 根 + 右贡献）。负贡献舍弃为 0。

#### 复杂度
- 时间：O(n)
- 空间：O(h)

#### 代码模板
```python
def maxPathSum(root):
    res = [root.val]
    def dfs(node):
        if not node: return 0
        l = max(dfs(node.left), 0)
        r = max(dfs(node.right), 0)
        res[0] = max(res[0], node.val + l + r)
        return node.val + max(l, r)
    dfs(root)
    return res[0]
```

---

### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给你一棵二叉树的根节点 `root`，翻转这棵二叉树，并返回其根节点。

**示例：**
```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]

输入：root = [2,1,3]
输出：[2,3,1]

输入：root = []
输出：[]
```

**约束：**
- 树中节点数目范围在 `[0, 100]` 内
- `-100 <= Node.val <= 100`

#### 核心思路
递归交换每个节点的左右子树，自底向上处理，空节点直接返回。

#### 复杂度
- 时间：O(n)
- 空间：O(h)

#### 代码模板
```python
def invertTree(root):
    if not root: return None
    root.left, root.right = invertTree(root.right), invertTree(root.left)
    return root
```

---

### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个二叉树，找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834)中最近公共祖先的定义为："对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。"

**示例：**
```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出：3
解释：节点 5 和节点 1 的最近公共祖先是节点 3 。

输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出：5
解释：节点 5 和节点 4 的最近公共祖先是节点 5 。因为根据定义最近公共祖先节点可以为节点本身。

输入：root = [1,2], p = 1, q = 2
输出：1
```

**约束：**
- 树中节点数目在范围 `[2, 10^5]` 内
- `-10^9 <= Node.val <= 10^9`
- 所有 `Node.val` 互不相同
- `p != q`
- `p` 和 `q` 均存在于给定的二叉树中

#### 核心思路
若当前节点为 p 或 q 则返回自身；递归查左右子树，若两侧均非空则当前节点为 LCA；否则返回非空的那侧。

#### 复杂度
- 时间：O(n)
- 空间：O(h)

#### 代码模板
```python
def lowestCommonAncestor(root, p, q):
    if not root or root == p or root == q: return root
    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)
    if left and right: return root
    return left or right
```

---

## 四、动态规划

### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。

**示例：**
```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。

输入：s = "cbbd"
输出："bb"
```

**约束：**
- `1 <= s.length <= 1000`
- `s` 仅由数字和英文字母组成

#### 核心思路
中心扩展法，枚举每个位置作为回文中心（奇数长度和偶数长度分别处理），向两端扩展记录最长回文子串。

#### 复杂度
- 时间：O(n²)
- 空间：O(1)

#### 代码模板
```python
def longestPalindrome(s):
    res = ""
    def expand(l, r):
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1; r += 1
        return s[l+1:r]
    for i in range(len(s)):
        odd = expand(i, i)
        even = expand(i, i+1)
        if len(odd) > len(res): res = odd
        if len(even) > len(res): res = even
    return res
```

---

### [62. 不同路径](https://leetcode.cn/problems/unique-paths/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

一个机器人位于一个 `m x n` 网格的左上角（起始点在下图中标记为 "Start"）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 "Finish"）。

问总共有多少条不同的路径？

**示例：**
```
输入：m = 3, n = 7
输出：28

输入：m = 3, n = 2
输出：3
解释：从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右
3. 向下 -> 向右 -> 向下
```

**约束：**
- `1 <= m, n <= 100`

#### 核心思路
dp[i][j] = dp[i-1][j] + dp[i][j-1]，从左上到右下，每格等于上方加左方路径数之和。可压缩为一维。

#### 复杂度
- 时间：O(mn)
- 空间：O(n)

#### 代码模板
```python
def uniquePaths(m, n):
    dp = [1] * n
    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j-1]
    return dp[-1]
```

---

### [72. 编辑距离](https://leetcode.cn/problems/edit-distance/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

给你两个单词 `word1` 和 `word2`，请返回将 `word1` 转换成 `word2` 所使用的最少操作数。

你可以对一个单词进行如下三种操作：
- 插入一个字符
- 删除一个字符
- 替换一个字符

**示例：**
```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')

输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

**约束：**
- `0 <= word1.length, word2.length <= 500`
- `word1` 和 `word2` 由小写英文字母组成

#### 核心思路
dp[i][j] 表示将 word1[:i] 变为 word2[:j] 的最少操作数。字符相同则继承，否则取插入、删除、替换三种操作的最小值加一。

#### 复杂度
- 时间：O(mn)
- 空间：O(mn)

#### 代码模板
```python
def minDistance(word1, word2):
    m, n = len(word1), len(word2)
    dp = list(range(n + 1))
    for i in range(1, m + 1):
        prev = dp[0]; dp[0] = i
        for j in range(1, n + 1):
            temp = dp[j]
            if word1[i-1] == word2[j-1]: dp[j] = prev
            else: dp[j] = 1 + min(prev, dp[j], dp[j-1])
            prev = temp
    return dp[n]
```

---

### [96. 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数 `n`，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的**二叉搜索树**有多少种？返回满足题意的二叉搜索树的种数。

**示例：**
```
输入：n = 3
输出：5

输入：n = 1
输出：1
```

**约束：**
- `1 <= n <= 19`

#### 核心思路
卡特兰数。dp[i] 表示 i 个节点的 BST 种数。以每个节点 j 为根，左子树有 j-1 个节点，右子树有 i-j 个节点，累加所有情况。

#### 复杂度
- 时间：O(n²)
- 空间：O(n)

#### 代码模板
```python
def numTrees(n):
    dp = [0] * (n + 1)
    dp[0] = dp[1] = 1
    for i in range(2, n + 1):
        for j in range(1, i + 1):
            dp[i] += dp[j-1] * dp[i-j]
    return dp[n]
```

---

### [139. 单词拆分](https://leetcode.cn/problems/word-break/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：** 不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

**示例：**
```
输入：s = "leetcode", wordDict = ["leet", "code"]
输出：true
解释：返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。

输入：s = "applepenapple", wordDict = ["apple", "pen"]
输出：true
解释：返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
     注意，你可以重复使用字典中的单词。

输入：s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出：false
```

**约束：**
- `1 <= s.length <= 300`
- `1 <= wordDict.length <= 1000`
- `1 <= wordDict[i].length <= 20`
- `s` 和 `wordDict[i]` 仅由小写英文字母组成
- `wordDict` 中的所有字符串**互不相同**

#### 核心思路
dp[i] 表示 s[:i] 能否被字典中的单词拼接。对每个位置 j，若 dp[j] 为真且 s[j:i] 在字典中，则 dp[i] 为真。

#### 复杂度
- 时间：O(n²)
- 空间：O(n)

#### 代码模板
```python
def wordBreak(s, wordDict):
    ws = set(wordDict)
    dp = [False] * (len(s) + 1)
    dp[0] = True
    for i in range(1, len(s) + 1):
        for j in range(i):
            if dp[j] and s[j:i] in ws:
                dp[i] = True; break
    return dp[-1]
```

---

### [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个**32 位**整数。

**示例：**
```
输入：nums = [2,3,-2,4]
输出：6
解释：子数组 [2,3] 有最大乘积 6。

输入：nums = [-2,0,-1]
输出：0
解释：结果不能为 2，因为 [-2,-1] 不是子数组。
```

**约束：**
- `1 <= nums.length <= 2 * 10^4`
- `-10 <= nums[i] <= 10`
- `nums` 的任何前缀或后缀的乘积的绝对值不超过 `10^10` 。

#### 核心思路
同时维护以当前元素结尾的最大积和最小积（负数乘以最小值可变最大），每步更新全局最大值。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def maxProduct(nums):
    res = cur_max = cur_min = nums[0]
    for n in nums[1:]:
        candidates = (n, cur_max * n, cur_min * n)
        cur_max, cur_min = max(candidates), min(candidates)
        res = max(res, cur_max)
    return res
```

---

### [198. 打家劫舍](https://leetcode.cn/problems/house-robber/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你**不触动警报装置的情况下**，一夜之内能够偷窃到的最高金额。

**示例：**
```
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。

输入：[2,7,9,3,1]
输出：12
解释：偷窃 1 号房屋 (金额 = 2)、偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```

**约束：**
- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 400`

#### 核心思路
dp[i] = max(dp[i-1], dp[i-2] + nums[i])，只需维护前两个状态，滚动变量节省空间。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def rob(nums):
    prev2, prev1 = 0, 0
    for n in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + n)
    return prev1
```

---

### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

**示例：**
```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。

输入：nums = [0,1,0,2,3]
输出：4

输入：nums = [7,7,7,7,7,7,7]
输出：1
```

**约束：**
- `1 <= nums.length <= 2500`
- `-10^4 <= nums[i] <= 10^4`

**进阶：** 你能将算法的时间复杂度降低到 `O(n log(n))` 吗?

#### 核心思路
patience sorting / 贪心 + 二分，维护一个 tails 数组，每次用二分找当前数字的位置替换或追加，长度即为 LIS。

#### 复杂度
- 时间：O(n log n)
- 空间：O(n)

#### 代码模板
```python
import bisect
def lengthOfLIS(nums):
    tails = []
    for n in nums:
        pos = bisect.bisect_left(tails, n)
        if pos == len(tails): tails.append(n)
        else: tails[pos] = n
    return len(tails)
```

---

### [322. 零钱兑换](https://leetcode.cn/problems/coin-change/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `coins`，代表不同面额的硬币；以及一个整数 `amount`，代表总金额。

计算并返回可以凑成总金额所需的**最少的硬币个数**。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

你可以认为每种硬币的数量是**无限的**。

**示例：**
```
输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1

输入：coins = [2], amount = 3
输出：-1

输入：coins = [1], amount = 0
输出：0
```

**约束：**
- `1 <= coins.length <= 12`
- `1 <= coins[i] <= 2^31 - 1`
- `0 <= amount <= 10^4`

#### 核心思路
完全背包，dp[i] 表示凑成金额 i 所需最少硬币数。对每种硬币，若 i >= coin 则 `dp[i] = min(dp[i], dp[i-coin]+1)`。

#### 复杂度
- 时间：O(amount × n)
- 空间：O(amount)

#### 代码模板
```python
def coinChange(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] = min(dp[i], dp[i - coin] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

---

## 五、栈 / 队列 / 单调栈

### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s`，判断字符串是否有效。

有效字符串需满足：
1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

**示例：**
```
输入：s = "()"
输出：true

输入：s = "()[]{}"
输出：true

输入：s = "(]"
输出：false

输入：s = "([)]"
输出：false

输入：s = "{[]}"
输出：true
```

**约束：**
- `1 <= s.length <= 10^4`
- `s` 仅由括号 `'()[]{}'` 组成

#### 核心思路
用栈存左括号，遇到右括号检查栈顶是否为对应左括号。最终栈为空则合法。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def isValid(s):
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    for c in s:
        if c in mapping:
            if not stack or stack[-1] != mapping[c]: return False
            stack.pop()
        else:
            stack.append(c)
    return not stack
```

---

### [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

给定 `n` 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 `1`。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

**示例：**
```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：最大的矩形为图中红色区域，面积为 10

输入：heights = [2,4]
输出：4
```

**约束：**
- `1 <= heights.length <= 10^5`
- `0 <= heights[i] <= 10^4`

#### 核心思路
单调递增栈，当遇到比栈顶更矮的柱子时，弹出栈顶并计算以该柱子为高度的矩形面积，宽度由当前索引和新栈顶决定。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def largestRectangleArea(heights):
    stack = [-1]
    res = 0
    for i, h in enumerate(heights):
        while stack[-1] != -1 and heights[stack[-1]] >= h:
            height = heights[stack.pop()]
            width = i - stack[-1] - 1
            res = max(res, height * width)
        stack.append(i)
    while stack[-1] != -1:
        height = heights[stack.pop()]
        width = len(heights) - stack[-1] - 1
        res = max(res, height * width)
    return res
```

---

### [155. 最小栈](https://leetcode.cn/problems/min-stack/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

设计一个支持 `push`，`pop`，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类：
- `MinStack()` 初始化堆栈对象
- `void push(int val)` 将元素 `val` 推入堆栈
- `void pop()` 删除堆栈顶部的元素
- `int top()` 获取堆栈顶部的元素
- `int getMin()` 获取堆栈中的最小元素

**示例：**
```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); --> 返回 -3.
minStack.pop();
minStack.top();    --> 返回 0.
minStack.getMin(); --> 返回 -2.
```

**约束：**
- `-2^31 <= val <= 2^31 - 1`
- `pop`、`top` 和 `getMin` 操作总是在**非空栈**上调用
- `push`、`pop`、`top` 和 `getMin` 最多被调用 `3 * 10^4` 次

#### 核心思路
用辅助栈同步维护当前最小值，push 时将 `min(val, 辅助栈顶)` 压入辅助栈，pop 时两栈同步弹出。

#### 复杂度
- 时间：O(1)
- 空间：O(n)

#### 代码模板
```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []
    def push(self, val):
        self.stack.append(val)
        self.min_stack.append(min(val, self.min_stack[-1] if self.min_stack else val))
    def pop(self):
        self.stack.pop(); self.min_stack.pop()
    def top(self): return self.stack[-1]
    def getMin(self): return self.min_stack[-1]
```

---

### [232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

实现 `MyQueue` 类：
- `void push(int x)` 将元素 x 推到队列的末尾
- `int pop()` 从队列的开头移除并返回元素
- `int peek()` 返回队列开头的元素
- `boolean empty()` 如果队列为空，返回 `true`；否则，返回 `false`

**说明：** 你**只能**使用标准的栈操作 —— 也就是只有 `push to top`、`peek/pop from top`、`size` 和 `is empty` 操作是合法的。

**示例：**
```
输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]

解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.peek();  // return 1
myQueue.pop();   // return 1, queue is [2]
myQueue.empty(); // return false
```

**约束：**
- `1 <= x <= 9`
- 最多调用 `100` 次 `push`、`pop`、`peek` 和 `empty`
- 假设所有操作都是有效的

**进阶：** 你能否实现每个操作均摊时间复杂度为 `O(1)` 的队列？换句话说，执行 `n` 个操作的总时间复杂度为 `O(n)`，即使其中一个操作可能花费较长时间。

#### 核心思路
用两个栈（输入栈和输出栈），push 进输入栈，pop/peek 时若输出栈空则将输入栈全部倒入输出栈，均摊 O(1)。

#### 复杂度
- 时间：O(1) 均摊
- 空间：O(n)

#### 代码模板
```python
class MyQueue:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []
    def push(self, x):
        self.in_stack.append(x)
    def _transfer(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())
    def pop(self):
        self._transfer(); return self.out_stack.pop()
    def peek(self):
        self._transfer(); return self.out_stack[-1]
    def empty(self):
        return not self.in_stack and not self.out_stack
```

---

### [739. 每日温度](https://leetcode.cn/problems/daily-temperatures/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个整数数组 `temperatures`，表示每天的温度，返回一个数组 `answer`，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

**示例：**
```
输入：temperatures = [73,74,75,71,69,72,76,73]
输出：[1,1,4,2,1,1,0,0]

输入：temperatures = [30,40,50,60]
输出：[1,1,1,0]

输入：temperatures = [30,60,90]
输出：[1,1,0]
```

**约束：**
- `1 <= temperatures.length <= 10^5`
- `30 <= temperatures[i] <= 100`

#### 核心思路
单调递减栈（存索引），遍历时若当前温度大于栈顶索引对应温度，则弹出并计算天数差，否则入栈。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def dailyTemperatures(temperatures):
    n = len(temperatures)
    ans = [0] * n
    stack = []
    for i, t in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < t:
            j = stack.pop()
            ans[j] = i - j
        stack.append(i)
    return ans
```

---

## 六、哈希表

### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个字符串数组，请你将**字母异位词**组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

**示例：**
```
输入：strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出：[["bat"],["nat","tan"],["ate","eat","tea"]]

输入：strs = [""]
输出：[[""]]

输入：strs = ["a"]
输出：[["a"]]
```

**约束：**
- `1 <= strs.length <= 10^4`
- `0 <= strs[i].length <= 100`
- `strs[i]` 仅包含小写字母

#### 核心思路
对每个单词排序后的结果作为哈希 key，将同组异位词归并到同一列表。或用字母计数元组作 key。

#### 复杂度
- 时间：O(n·k log k)
- 空间：O(nk)

#### 代码模板
```python
from collections import defaultdict
def groupAnagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        groups[tuple(sorted(s))].append(s)
    return list(groups.values())
```

---

### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个未排序的整数数组 `nums`，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

**示例：**
```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。

输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

**约束：**
- `0 <= nums.length <= 10^5`
- `-10^9 <= nums[i] <= 10^9`

#### 核心思路
将所有数放入哈希集合，只从序列起点（num-1 不在集合中）开始向右扩展，统计连续长度，避免重复遍历。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
def longestConsecutive(nums):
    num_set = set(nums)
    res = 0
    for n in num_set:
        if n - 1 not in num_set:
            length = 1
            while n + length in num_set:
                length += 1
            res = max(res, length)
    return res
```

---

### [146. LRU缓存](https://leetcode.cn/problems/lru-cache/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

请你设计并实现一个满足  [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：
- `LRUCache(int capacity)` 以**正整数**作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1`
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value`；如果不存在，则向缓存中插入该组 `key-value`。如果插入操作导致关键字数量超过 `capacity`，则应该**逐出**最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

**示例：**
```
输入：
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出：
[null, null, null, 1, null, -1, null, -1, 3, 4]
```

**约束：**
- `1 <= capacity <= 3000`
- `0 <= key <= 10^4`
- `0 <= value <= 10^5`
- 最多调用 `2 * 10^5` 次 `get` 和 `put`

#### 核心思路
哈希表 + 双向链表。哈希表 O(1) 查找，双向链表维护访问顺序（最近使用移到头部，超容量删除尾部）。

#### 复杂度
- 时间：O(1)
- 空间：O(capacity)

#### 代码模板
```python
from collections import OrderedDict
class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = OrderedDict()
    def get(self, key):
        if key not in self.cache: return -1
        self.cache.move_to_end(key)
        return self.cache[key]
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)
```

---

### [560. 和为K的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums` 和一个整数 `k`，请你统计并返回该数组中和为 `k` 的子数组的个数。

子数组是数组中元素的连续非空序列。

**示例：**
```
输入：nums = [1,1,1], k = 2
输出：2

输入：nums = [1,2,3], k = 3
输出：2
```

**约束：**
- `1 <= nums.length <= 2 * 10^4`
- `-1000 <= nums[i] <= 1000`
- `-10^7 <= k <= 10^7`

#### 核心思路
前缀和 + 哈希表，记录每个前缀和出现的次数，对每个位置检查 `prefix - k` 是否存在于哈希表中。

#### 复杂度
- 时间：O(n)
- 空间：O(n)

#### 代码模板
```python
from collections import defaultdict
def subarraySum(nums, k):
    count = defaultdict(int)
    count[0] = 1
    prefix = res = 0
    for n in nums:
        prefix += n
        res += count[prefix - k]
        count[prefix] += 1
    return res
```

---

## 七、二分查找

### [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

整数数组 `nums` 按升序排列，数组中的值**互不相同**。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了**旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标**从 0 开始**计数）。例如，`[0,1,2,4,5,6,7]` 在下标 `3` 处经旋转后可能变为 `[4,5,6,7,0,1,2]`。

给你旋转后的数组 `nums` 和一个整数 `target`，如果 `nums` 中存在这个目标值 `target`，则返回它的下标，否则返回 `-1`。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

**示例：**
```
输入：nums = [4,5,6,7,0,1,2], target = 0
输出：4

输入：nums = [4,5,6,7,0,1,2], target = 3
输出：-1

输入：nums = [1], target = 0
输出：-1
```

**约束：**
- `1 <= nums.length <= 5000`
- `-10^4 <= nums[i] <= 10^4`
- `nums` 中的每个值都**独一无二**
- 题目数据保证 `nums` 在预先未知的某个下标上进行了旋转
- `-10^4 <= target <= 10^4`

#### 核心思路
二分时先判断哪半边是有序的，再判断目标是否在有序部分内，据此缩小查找范围。

#### 复杂度
- 时间：O(log n)
- 空间：O(1)

#### 代码模板
```python
def search(nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = (l + r) // 2
        if nums[mid] == target: return mid
        if nums[l] <= nums[mid]:  # 左边有序
            if nums[l] <= target < nums[mid]: r = mid - 1
            else: l = mid + 1
        else:  # 右边有序
            if nums[mid] < target <= nums[r]: l = mid + 1
            else: r = mid - 1
    return -1
```

---

### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

**示例：**
```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]

输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]

输入：nums = [], target = 0
输出：[-1,-1]
```

**约束：**
- `0 <= nums.length <= 10^5`
- `-10^9 <= nums[i] <= 10^9`
- `nums` 是一个非递减数组
- `-10^9 <= target <= 10^9`

#### 核心思路
两次二分，第一次找左边界（第一个 >= target 的位置），第二次找右边界（最后一个 <= target 的位置）。

#### 复杂度
- 时间：O(log n)
- 空间：O(1)

#### 代码模板
```python
import bisect
def searchRange(nums, target):
    l = bisect.bisect_left(nums, target)
    r = bisect.bisect_right(nums, target) - 1
    if l <= r and r < len(nums) and nums[l] == target:
        return [l, r]
    return [-1, -1]
```

---

### [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**示例：**
```
输入：nums = [1,3,5,6], target = 5
输出：2

输入：nums = [1,3,5,6], target = 2
输出：1

输入：nums = [1,3,5,6], target = 7
输出：4
```

**约束：**
- `1 <= nums.length <= 10^4`
- `-10^4 <= nums[i] <= 10^4`
- `nums` 为**无重复元素**的**升序**排列数组
- `-10^4 <= target <= 10^4`

#### 核心思路
标准二分查找，找第一个 >= target 的位置。若找到则返回该位置，否则返回应插入位置。

#### 复杂度
- 时间：O(log n)
- 空间：O(1)

#### 代码模板
```python
def searchInsert(nums, target):
    l, r = 0, len(nums)
    while l < r:
        mid = (l + r) // 2
        if nums[mid] < target:
            l = mid + 1
        else:
            r = mid
    return l
```

---

### [704. 二分查找](https://leetcode.cn/problems/binary-search/) ![Easy](https://img.shields.io/badge/-Easy-green)

#### 题目描述

给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target`，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。

**示例：**
```
输入：nums = [-1,0,3,5,9,12], target = 9
输出：4
解释：9 出现在 nums 中并且下标为 4

输入：nums = [-1,0,3,5,9,12], target = 2
输出：-1
解释：2 不存在 nums 中因此返回 -1
```

**约束：**
- 你可以假设 `nums` 中的所有元素是不重复的。
- `n` 将在 `[1, 10000]` 之间。
- `nums` 的每个元素都将在 `[-9999, 9999]` 之间。

#### 核心思路
经典二分，左闭右闭区间，mid 等于 target 则返回，否则根据大小缩小区间。

#### 复杂度
- 时间：O(log n)
- 空间：O(1)

#### 代码模板
```python
def search(nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = (l + r) // 2
        if nums[mid] == target: return mid
        elif nums[mid] < target: l = mid + 1
        else: r = mid - 1
    return -1
```

---

## 八、回溯

### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按**任意顺序**返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

```
2: abc   3: def   4: ghi   5: jkl
6: mno   7: pqrs  8: tuv   9: wxyz
```

**示例：**
```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]

输入：digits = ""
输出：[]

输入：digits = "2"
输出：["a","b","c"]
```

**约束：**
- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的一个数字

#### 核心思路
回溯，按位处理每个数字，对该数字对应的每个字母递归添加到路径，到达末尾时加入结果。

#### 复杂度
- 时间：O(4^n × n)
- 空间：O(n)

#### 代码模板
```python
def letterCombinations(digits):
    if not digits: return []
    phone = {'2':'abc','3':'def','4':'ghi','5':'jkl',
             '6':'mno','7':'pqrs','8':'tuv','9':'wxyz'}
    res = []
    def bt(i, path):
        if i == len(digits): res.append(''.join(path)); return
        for c in phone[digits[i]]:
            path.append(c)
            bt(i+1, path)
            path.pop()
    bt(0, [])
    return res
```

---

### [39. 组合总和](https://leetcode.cn/problems/combination-sum/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个**无重复元素**的整数数组 `candidates` 和一个目标整数 `target`，找出 `candidates` 中可以使数字和为目标数 `target` 的所有**不同组合**，并以列表形式返回。你可以按**任意顺序**返回这些组合。

`candidates` 中的**同一个**数字可以**无限制重复被选取**。如果至少一个数字的被选数量不同，则两种组合是不同的。

对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

**示例：**
```
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]
解释：
2 和 3 可以形成一组候选，2 + 2 + 3 = 7。注意 2 可以使用多次。
7 也是一个候选，7 = 7。
仅有这两种组合。

输入：candidates = [2,3,5], target = 8
输出：[[2,2,2,2],[2,3,3],[3,5]]
```

**约束：**
- `1 <= candidates.length <= 30`
- `2 <= candidates[i] <= 40`
- `candidates` 的所有元素**互不相同**
- `1 <= target <= 40`

#### 核心思路
回溯，可重复选同一元素。对候选数组排序后，若当前数已大于 remaining 则剪枝。每次递归从当前索引开始选。

#### 复杂度
- 时间：O(N^(target/min))
- 空间：O(target/min)

#### 代码模板
```python
def combinationSum(candidates, target):
    res = []
    def bt(start, path, remain):
        if remain == 0: res.append(path[:]); return
        for i in range(start, len(candidates)):
            if candidates[i] > remain: break
            path.append(candidates[i])
            bt(i, path, remain - candidates[i])
            path.pop()
    candidates.sort()
    bt(0, [], target)
    return res
```

---

### [46. 全排列](https://leetcode.cn/problems/permutations/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个不含重复数字的数组 `nums`，返回其**所有可能的全排列**。你可以**按任意顺序**返回答案。

**示例：**
```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

输入：nums = [0,1]
输出：[[0,1],[1,0]]

输入：nums = [1]
输出：[[1]]
```

**约束：**
- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有整数**互不相同**

#### 核心思路
回溯，维护已使用集合，对每个未使用元素加入路径并递归，返回时撤销选择。

#### 复杂度
- 时间：O(n·n!)
- 空间：O(n)

#### 代码模板
```python
def permute(nums):
    res = []
    def bt(path, used):
        if len(path) == len(nums): res.append(path[:]); return
        for n in nums:
            if n not in used:
                used.add(n); path.append(n)
                bt(path, used)
                path.pop(); used.remove(n)
    bt([], set())
    return res
```

---

### [51. N皇后](https://leetcode.cn/problems/n-queens/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题**研究的是如何将 `n` 个皇后放置在 `n × n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n`，返回所有不同的**n 皇后问题**的解决方案。

每一种解法包含一个不同的 **n 皇后问题**的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

**示例：**
```
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
解释：如上图所示，4 皇后问题存在两个不同的解法。

输入：n = 1
输出：[["Q"]]
```

**约束：**
- `1 <= n <= 9`

#### 核心思路
逐行放置皇后，用集合记录已占用的列、正对角线、反对角线，满足约束则进入下一行，回溯时移除标记。

#### 复杂度
- 时间：O(n!)
- 空间：O(n)

#### 代码模板
```python
def solveNQueens(n):
    res = []
    cols, diag1, diag2 = set(), set(), set()
    board = [['.' ] * n for _ in range(n)]
    def bt(row):
        if row == n:
            res.append([''.join(r) for r in board]); return
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2: continue
            cols.add(col); diag1.add(row-col); diag2.add(row+col)
            board[row][col] = 'Q'
            bt(row + 1)
            board[row][col] = '.'
            cols.remove(col); diag1.remove(row-col); diag2.remove(row+col)
    bt(0)
    return res
```

---

### [78. 子集](https://leetcode.cn/problems/subsets/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个整数数组 `nums`，数组中的元素**互不相同**。返回该数组所有可能的子集（幂集）。

解集**不能**包含重复的子集。你可以按**任意顺序**返回解集。

**示例：**
```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

输入：nums = [0]
输出：[[],[0]]
```

**约束：**
- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有元素**互不相同**

#### 核心思路
回溯，每进入一层就记录当前路径为一个子集。从 start 开始枚举，避免重复。

#### 复杂度
- 时间：O(n·2^n)
- 空间：O(n)

#### 代码模板
```python
def subsets(nums):
    res = []
    def bt(start, path):
        res.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            bt(i + 1, path)
            path.pop()
    bt(0, [])
    return res
```

---

## 九、贪心

### [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。

每个元素 `nums[i]` 表示从索引 `i` 向前跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处：
- `0 <= j <= nums[i]`
- `i + j < n`

返回到达 `nums[n - 1]` 的最小跳跃次数。生成的测试用例可以到达 `nums[n - 1]`。

**示例：**
```
输入：nums = [2,3,1,1,4]
输出：2
解释：跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。

输入：nums = [2,3,0,1,4]
输出：2
```

**约束：**
- `1 <= nums.length <= 10^4`
- `0 <= nums[i] <= 1000`
- 生成的测试用例可以到达 `nums[n - 1]`

#### 核心思路
贪心，维护当前跳跃能到达的最远边界和下次跳跃的最远位置。到达边界时步数加一，更新边界。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def jump(nums):
    jumps = cur_end = cur_far = 0
    for i in range(len(nums) - 1):
        cur_far = max(cur_far, i + nums[i])
        if i == cur_end:
            jumps += 1; cur_end = cur_far
    return jumps
```

---

### [55. 跳跃游戏](https://leetcode.cn/problems/jump-game/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个非负整数数组 `nums`，你最初位于数组的**第一个下标**。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true`；否则，返回 `false`。

**示例：**
```
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。

输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0，所以永远不能到达最后一个下标。
```

**约束：**
- `1 <= nums.length <= 3 * 10^4`
- `0 <= nums[i] <= 10^5`

#### 核心思路
维护当前能到达的最远位置 `max_reach`，遍历每个位置若当前位置可达则更新最远距离，最终判断能否到达末尾。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def canJump(nums):
    max_reach = 0
    for i, n in enumerate(nums):
        if i > max_reach: return False
        max_reach = max(max_reach, i + n)
    return True
```

---

### [56. 合并区间](https://leetcode.cn/problems/merge-intervals/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]`。请你合并所有重叠的区间，并返回一个**不重叠的区间数组**，该数组需恰好覆盖输入中的所有区间。

**示例：**
```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠，将它们合并为 [1,6]。

输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 是可以视为重叠区间。
```

**约束：**
- `1 <= intervals.length <= 10^4`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 10^4`

#### 核心思路
按起始点排序后，逐个检查当前区间是否与结果集最后一个区间重叠，重叠则合并（更新终点），否则直接加入。

#### 复杂度
- 时间：O(n log n)
- 空间：O(n)

#### 代码模板
```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])
    res = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= res[-1][1]:
            res[-1][1] = max(res[-1][1], end)
        else:
            res.append([start, end])
    return res
```

---

### [134. 加油站](https://leetcode.cn/problems/gas-station/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

给定两个整数数组 `gas` 和 `cost`，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 `-1`。如果存在解，则**保证**它是**唯一**的。

**示例：**
```
输入：gas = [1,2,3,4,5], cost = [3,4,5,1,2]
输出：3
解释：
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为出发点。

输入：gas = [2,3,4], cost = [3,4,3]
输出：-1
解释：你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油。
开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油。
开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油。
你无法返回 2 号加油站，因为返程需要消耗 4 > 3 升汽油。
```

**约束：**
- `n == gas.length == cost.length`
- `1 <= n <= 10^5`
- `0 <= gas[i] <= 10^4`
- `0 <= cost[i] <= 10^4`

#### 核心思路
总油量 >= 总消耗则一定有解。从头扫描，维护当前油箱余量，一旦为负则从下一站重新开始，最终起点即为答案。

#### 复杂度
- 时间：O(n)
- 空间：O(1)

#### 代码模板
```python
def canCompleteCircuit(gas, cost):
    if sum(gas) < sum(cost): return -1
    tank = start = 0
    for i in range(len(gas)):
        tank += gas[i] - cost[i]
        if tank < 0:
            tank = 0; start = i + 1
    return start
```

---

### [406. 根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi`，前面**正好**有 `ki` 个身高**大于或等于** `hi` 的人。

请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue`，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

**示例：**
```
输入：people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
输出：[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
解释：
编号为 0 的人身高为 5，没有身高更高或者相同的人排在他前面。
编号为 4 的人身高为 4，有 4 个身高更高或者相同的人排在他前面，即编号为 0、1、2、3 的人。

输入：people = [[6,0],[5,0],[4,0],[3,2],[2,2],[1,4]]
输出：[[4,0],[5,0],[2,2],[3,2],[1,4],[6,0]]
```

**约束：**
- `1 <= people.length <= 2000`
- `0 <= hi <= 10^6`
- `0 <= ki < people.length`
- 题目数据确保队列可以被重建

#### 核心思路
先按身高降序（身高相同按 k 升序）排列，然后依次将每个人按其 k 值插入结果数组的对应位置。

#### 复杂度
- 时间：O(n²)
- 空间：O(n)

#### 代码模板
```python
def reconstructQueue(people):
    people.sort(key=lambda x: (-x[0], x[1]))
    res = []
    for p in people:
        res.insert(p[1], p)
    return res
```

---

## 十、图 / 拓扑排序

### [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水拥围，并且每座岛屿只能由水平方向和/或垂直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

**示例：**
```
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1

输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```

**约束：**
- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 300`
- `grid[i][j]` 的值为 `'0'` 或 `'1'`

#### 核心思路
DFS/BFS，遍历网格，遇到 '1' 则计数加一，并将连通的所有 '1' 标记为已访问（改为 '0'）。

#### 复杂度
- 时间：O(mn)
- 空间：O(mn)

#### 代码模板
```python
def numIslands(grid):
    m, n = len(grid), len(grid[0])
    def dfs(i, j):
        if not (0 <= i < m and 0 <= j < n) or grid[i][j] != '1': return
        grid[i][j] = '0'
        for di, dj in [(0,1),(0,-1),(1,0),(-1,0)]:
            dfs(i+di, j+dj)
    count = 0
    for i in range(m):
        for j in range(n):
            if grid[i][j] == '1': dfs(i, j); count += 1
    return count
```

---

### [207. 课程表](https://leetcode.cn/problems/course-schedule/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1`。

在选修某些课程之前需要一些先修课程。先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]`，表示如果要学习课程 `ai` 则**必须**先学习课程 `bi`。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0`，你需要先完成课程 `1`。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true`；否则，返回 `false`。

**示例：**
```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0。这是可能的。

输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0；并且学习课程 0 之前，你还应先完成课程 1。这是不可能的。
```

**约束：**
- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= 5000`
- `prerequisites[i].length == 2`
- `0 <= ai, bi < numCourses`
- `prerequisites[i]` 中的所有课程对**互不相同**

#### 核心思路
拓扑排序（BFS/Kahn 算法），建图后计算入度，将入度为 0 的节点入队，逐步移除并减少邻居入度，判断最终是否所有课程都处理完。

#### 复杂度
- 时间：O(V+E)
- 空间：O(V+E)

#### 代码模板
```python
from collections import deque, defaultdict
def canFinish(numCourses, prerequisites):
    graph = defaultdict(list)
    indegree = [0] * numCourses
    for a, b in prerequisites:
        graph[b].append(a); indegree[a] += 1
    q = deque(i for i in range(numCourses) if indegree[i] == 0)
    count = 0
    while q:
        node = q.popleft(); count += 1
        for nei in graph[node]:
            indegree[nei] -= 1
            if indegree[nei] == 0: q.append(nei)
    return count == numCourses
```

---

### [210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

现在你总共有 `numCourses` 门课需要选，记为 `0` 到 `numCourses - 1`。给你一个数组 `prerequisites`，其中 `prerequisites[i] = [ai, bi]`，表示在选修课程 `ai` 前**必须**先选修 `bi`。

- 例如，想要学习课程 `0`，你需要先完成课程 `1`，我们用一个匹配来表示：`[0,1]`。

返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回**任意一种**就可以了。如果不可能完成所有课程，返回**一个空数组**。

**示例：**
```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：[0,1]
解释：总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1]。

输入：numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
输出：[0,2,1,3]
解释：总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
     因此，一个正确的课程顺序是 [0,1,2,3]。另一个正确的排序是 [0,2,1,3]。
```

**约束：**
- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= numCourses * (numCourses - 1)`
- `prerequisites[i].length == 2`
- `0 <= ai, bi < numCourses`
- `ai != bi`
- 所有`[ai, bi]` **互不相同**

#### 核心思路
同课程表 I，但需记录拓扑排序的顺序。BFS 按入度 0 依次处理，记录每个出队节点即为合法修课顺序。

#### 复杂度
- 时间：O(V+E)
- 空间：O(V+E)

#### 代码模板
```python
from collections import deque, defaultdict
def findOrder(numCourses, prerequisites):
    graph = defaultdict(list)
    indegree = [0] * numCourses
    for a, b in prerequisites:
        graph[b].append(a); indegree[a] += 1
    q = deque(i for i in range(numCourses) if indegree[i] == 0)
    order = []
    while q:
        node = q.popleft(); order.append(node)
        for nei in graph[node]:
            indegree[nei] -= 1
            if indegree[nei] == 0: q.append(nei)
    return order if len(order) == numCourses else []
```

---

### [301. 删除无效的括号](https://leetcode.cn/problems/remove-invalid-parentheses/) ![Hard](https://img.shields.io/badge/-Hard-red)

#### 题目描述

给你一个由若干括号和字母组成的字符串 `s`，删除最小数量的无效括号，使得输入的字符串有效。

返回所有可能的结果。答案可以按**任意顺序**返回。

**示例：**
```
输入：s = "()())()"
输出：["(())()", "()()()"]

输入：s = "(a)())()"
输出：["(a)()()", "(a())()"]

输入：s = ")("
输出：[""]
```

**约束：**
- `1 <= s.length <= 25`
- `s` 由小写英文字母以及括号 `'('` 和 `')'` 组成
- `s` 中至多含 `20` 个括号

#### 核心思路
BFS 按层搜索，每层尝试删除一个括号生成所有下一状态，用集合去重。若当前层找到有效字符串则不再继续往下层搜（已是最少删除）。

#### 复杂度
- 时间：O(n · 2^n)
- 空间：O(2^n)

#### 代码模板
```python
def removeInvalidParentheses(s):
    def is_valid(s):
        count = 0
        for c in s:
            if c == '(': count += 1
            elif c == ')':
                count -= 1
                if count < 0: return False
        return count == 0
    res = []
    visited = {s}
    queue = [s]
    found = False
    while queue:
        next_queue = []
        for curr in queue:
            if is_valid(curr):
                res.append(curr)
                found = True
            if not found:
                for i in range(len(curr)):
                    if curr[i] in '()':
                        nxt = curr[:i] + curr[i+1:]
                        if nxt not in visited:
                            visited.add(nxt)
                            next_queue.append(nxt)
        if found: break
        queue = next_queue
    return res if res else [""]
```

---

### [399. 除法求值](https://leetcode.cn/problems/evaluate-division/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

给你一个变量对数组 `equations` 和一个实数值数组 `values` 作为已知条件，其中 `equations[i] = [Ai, Bi]` 和 `values[i]` 共同表示等式 `Ai / Bi = values[i]`。每个 `Ai` 或 `Bi` 是一个表示单个变量的字符串。

另有一些以数组 `queries` 表示的问题，其中 `queries[j] = [Cj, Dj]` 表示第 `j` 个问题，请你根据已知条件找出 `Cj / Dj = ?` 的结果作为答案。

返回**所有问题的答案**。如果存在某个无法确定的答案，则用 `-1.0` 替代这个答案。如果问题中出现了给定的已知条件中没有出现的字符串，也需要用 `-1.0` 替代这个答案。

**注意：** 输入总是有效的。你可以假设除法运算中不会出现除数为 0 的情况，且不存在任何矛盾的结果。

**示例：**
```
输入：equations = [["a","b"],["b","c"]], values = [2.0,3.0],
queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
输出：[6.00000,0.50000,-1.00000,1.00000,-1.00000]
解释：
条件：a / b = 2.0, b / c = 3.0
问题：a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ?
结果：[6.0, 0.5, -1.0, 1.0, -1.0 ]
注意：x 是未定义的 => -1.0
```

**约束：**
- `1 <= equations.length <= 20`
- `equations[i].length == 2`
- `1 <= Ai.length, Bi.length <= 5`
- `values.length == equations.length`
- `0.0 < values[i] <= 20.0`
- `1 <= queries.length <= 20`
- `queries[i].length == 2`
- `1 <= Cj.length, Dj.length <= 5`
- `Ai, Bi, Cj, Dj` 由小写英文字母与数字组成

#### 核心思路
建立带权有向图（a→b 权为 values[i]，b→a 权为 1/values[i]），对每个查询 BFS/DFS 从源节点到目标节点，累乘路径上的权重。

#### 复杂度
- 时间：O((N+Q) × E)
- 空间：O(N)

#### 代码模板
```python
from collections import defaultdict, deque
def calcEquation(equations, values, queries):
    graph = defaultdict(dict)
    for (a, b), v in zip(equations, values):
        graph[a][b] = v
        graph[b][a] = 1 / v
    def bfs(src, dst):
        if src not in graph or dst not in graph: return -1.0
        if src == dst: return 1.0
        visited = {src}
        q = deque([(src, 1.0)])
        while q:
            node, prod = q.popleft()
            if node == dst: return prod
            for nei, w in graph[node].items():
                if nei not in visited:
                    visited.add(nei)
                    q.append((nei, prod * w))
        return -1.0
    return [bfs(c, d) for c, d in queries]
```

---

### [684. 冗余连接](https://leetcode.cn/problems/redundant-connection/) ![Medium](https://img.shields.io/badge/-Medium-orange)

#### 题目描述

树可以看成是一个连通且**无环**的**无向**图。

给定往一棵 `n` 个节点（节点值 `1～n`）的树中添加一条边后的图。添加的边的两个顶点包含在 `1` 到 `n` 中间，且这条附加的边不属于树中已存在的边。图的信息记录于长度为 `n` 的二维数组 `edges`，`edges[i] = [ai, bi]` 表示图中在 `ai` 和 `bi` 之间存在一条边。

请找出一条可以删去的边，删除后可使得剩余部分是一个有着 `n` 个节点的树。如果有多个答案，则返回数组 `edges` 中最后出现的那个。

**示例：**
```
输入：edges = [[1,2],[1,3],[2,3]]
输出：[2,3]

输入：edges = [[1,2],[2,3],[3,4],[1,4],[1,5]]
输出：[1,4]
```

**约束：**
- `n == edges.length`
- `3 <= n <= 1000`
- `edges[i].length == 2`
- `1 <= ai < bi <= edges.length`
- `ai != bi`
- `edges` 中无重复元素
- 给定的图是连通的

#### 核心思路
并查集，依次处理每条边，若边的两端已连通则该边为冗余边；否则合并两端。

#### 复杂度
- 时间：O(n·α(n))
- 空间：O(n)

#### 代码模板
```python
def findRedundantConnection(edges):
    parent = list(range(len(edges) + 1))
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]; x = parent[x]
        return x
    for u, v in edges:
        pu, pv = find(u), find(v)
        if pu == pv: return [u, v]
        parent[pu] = pv
```

---

*整理完成 · 共覆盖 LeetCode 高频 100 题核心题目（含完整题目描述）· 持续更新*
