---
layout: default
title: LeetCode 高频100题总结
permalink: /notes/leetcode-top100/
---

# LeetCode 高频100题总结

> 按题型分类整理，每题含核心思路、复杂度分析与 Python 代码模板。

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

### 1. 两数之和 (Two Sum) ★Easy
**核心思路**：用哈希表存已遍历元素，对每个数查找 `target - num` 是否存在。一次遍历即可，避免暴力 O(n²)。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 11. 盛最多水的容器 (Container With Most Water) ★Medium
**核心思路**：双指针从两端向中间收缩，每次移动较短的那端（移长端只会让面积变小）。贪心地保留可能更大的那端。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 15. 三数之和 (3Sum) ★Medium
**核心思路**：排序后固定一个数，对剩余部分用双指针求两数之和。注意跳过重复元素以避免重复三元组。  
**时间复杂度**：O(n²)　**空间复杂度**：O(1)
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

### 26. 删除有序数组中的重复项 (Remove Duplicates from Sorted Array) ★Easy
**核心思路**：慢快双指针，慢指针指向已处理区间的末尾，快指针向前扫描，遇到新元素就写入慢指针位置。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def removeDuplicates(nums):
    k = 1
    for i in range(1, len(nums)):
        if nums[i] != nums[i-1]:
            nums[k] = nums[i]
            k += 1
    return k
```

---

### 42. 接雨水 (Trapping Rain Water) ★Hard
**核心思路**：双指针法，维护左右最大高度，较小一侧可以确定积水量（等于该侧最大值减去当前高度）。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 53. 最大子数组和 (Maximum Subarray) ★Medium
**核心思路**：Kadane 算法，维护以当前元素结尾的最大子数组和。若前缀和为负则舍弃，从当前元素重新开始。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def maxSubArray(nums):
    cur = res = nums[0]
    for n in nums[1:]:
        cur = max(n, cur + n)
        res = max(res, cur)
    return res
```

---

### 121. 买卖股票的最佳时机 (Best Time to Buy and Sell Stock) ★Easy
**核心思路**：一次遍历，记录历史最低买入价，对每天计算当天卖出的利润，取最大值。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def maxProfit(prices):
    min_price, max_profit = float('inf'), 0
    for p in prices:
        min_price = min(min_price, p)
        max_profit = max(max_profit, p - min_price)
    return max_profit
```

---

### 238. 除自身以外数组的乘积 (Product of Array Except Self) ★Medium
**核心思路**：先从左到右计算前缀积，再从右到左乘以后缀积，两趟遍历不使用除法。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)（输出数组不计）
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

### 3. 无重复字符的最长子串 (Longest Substring Without Repeating Characters) ★Medium
**核心思路**：滑动窗口 + 哈希集合，右指针扩展窗口，遇到重复字符则左指针收缩直到窗口合法。  
**时间复杂度**：O(n)　**空间复杂度**：O(min(m,n))
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

### 76. 最小覆盖子串 (Minimum Window Substring) ★Hard
**核心思路**：滑动窗口，右指针扩展直到覆盖 t 的所有字符，然后左指针收缩尽量缩小窗口，记录最优解。  
**时间复杂度**：O(|s| + |t|)　**空间复杂度**：O(|s| + |t|)
```python
from collections import Counter
def minWindow(s, t):
    need = Counter(t)
    missing = len(t)
    l = start = end = 0
    for r, c in enumerate(s, 1):
        if need[c] > 0:
            missing -= 1
        need[c] -= 1
        if missing == 0:
            while need[s[l]] < 0:
                need[s[l]] += 1
                l += 1
            if end == 0 or r - l < end - start:
                start, end = l, r
            need[s[l]] += 1
            missing += 1
            l += 1
    return s[start:end]
```

---

## 二、链表

### 21. 合并两个有序链表 (Merge Two Sorted Lists) ★Easy
**核心思路**：使用哑节点，逐一比较两链表头节点大小，将较小的接入结果链表，直到一方耗尽后追加剩余部分。  
**时间复杂度**：O(m+n)　**空间复杂度**：O(1)
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

### 23. 合并K个升序链表 (Merge k Sorted Lists) ★Hard
**核心思路**：使用最小堆（优先队列）存各链表当前头节点，每次弹出最小值并将其后继入堆。避免两两合并的重复遍历。  
**时间复杂度**：O(N log k)　**空间复杂度**：O(k)
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

### 25. K个一组翻转链表 (Reverse Nodes in k-Group) ★Hard
**核心思路**：每次取 k 个节点进行原地翻转，翻转前先检查是否有足够节点，不足则保持原顺序。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def reverseKGroup(head, k):
    dummy = ListNode(0, head)
    group_prev = dummy
    while True:
        kth = group_prev
        for _ in range(k):
            kth = kth.next
            if not kth: return dummy.next
        group_next = kth.next
        prev, cur = group_next, group_prev.next
        while cur != group_next:
            nxt = cur.next
            cur.next = prev
            prev = cur; cur = nxt
        tmp = group_prev.next
        group_prev.next = kth
        group_prev = tmp
    return dummy.next
```

---

### 141. 环形链表 (Linked List Cycle) ★Easy
**核心思路**：Floyd 快慢指针，慢指针每次走一步，快指针走两步。若有环则两者必相遇，否则快指针先到达 null。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 142. 环形链表 II (Linked List Cycle II) ★Medium
**核心思路**：快慢指针相遇后，将一个指针移回 head，两个指针同速前进，再次相遇处即为环入口（数学推导）。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 160. 相交链表 (Intersection of Two Linked Lists) ★Easy
**核心思路**：两指针分别遍历 A+B 和 B+A，路径总长相同，若有交点必在同一位置相遇。  
**时间复杂度**：O(m+n)　**空间复杂度**：O(1)
```python
def getIntersectionNode(headA, headB):
    a, b = headA, headB
    while a != b:
        a = a.next if a else headB
        b = b.next if b else headA
    return a
```

---

### 206. 反转链表 (Reverse Linked List) ★Easy
**核心思路**：迭代法，维护 prev 和 cur 两个指针，逐一将 cur.next 指向 prev，向前推进。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 234. 回文链表 (Palindrome Linked List) ★Easy
**核心思路**：快慢指针找到链表中点，翻转后半段，然后双指针对比前后两段是否相等。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def isPalindrome(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next; fast = fast.next.next
    prev, cur = None, slow
    while cur:
        nxt = cur.next; cur.next = prev; prev = cur; cur = nxt
    l, r = head, prev
    while r:
        if l.val != r.val: return False
        l = l.next; r = r.next
    return True
```

---

## 三、二叉树 / DFS / BFS

### 94. 二叉树的中序遍历 (Binary Tree Inorder Traversal) ★Easy
**核心思路**：递归或迭代（用栈模拟）：左→根→右。迭代时用栈存路径，先一路向左压栈，弹出时访问，再转向右子树。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 102. 二叉树的层序遍历 (Binary Tree Level Order Traversal) ★Medium
**核心思路**：BFS，使用队列逐层处理，每层开始时记录当前队列长度，弹出该层所有节点并将子节点入队。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 104. 二叉树的最大深度 (Maximum Depth of Binary Tree) ★Easy
**核心思路**：递归，树的最大深度等于左右子树最大深度的较大值加一。空节点深度为 0。  
**时间复杂度**：O(n)　**空间复杂度**：O(h)
```python
def maxDepth(root):
    if not root: return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

---

### 105. 从前序与中序遍历序列构造二叉树 (Construct Binary Tree from Preorder and Inorder Traversal) ★Medium
**核心思路**：前序第一个元素为根，在中序中找到根的位置，左侧为左子树，右侧为右子树，递归构建。用哈希表加速查找。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 124. 二叉树中的最大路径和 (Binary Tree Maximum Path Sum) ★Hard
**核心思路**：DFS 返回"以当前节点为端点的最大单侧路径和"，同时更新全局答案（左贡献 + 根 + 右贡献）。负贡献舍弃为 0。  
**时间复杂度**：O(n)　**空间复杂度**：O(h)
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

### 226. 翻转二叉树 (Invert Binary Tree) ★Easy
**核心思路**：递归交换每个节点的左右子树，自底向上处理，空节点直接返回。  
**时间复杂度**：O(n)　**空间复杂度**：O(h)
```python
def invertTree(root):
    if not root: return None
    root.left, root.right = invertTree(root.right), invertTree(root.left)
    return root
```

---

### 236. 二叉树的最近公共祖先 (Lowest Common Ancestor of a Binary Tree) ★Medium
**核心思路**：若当前节点为 p 或 q 则返回自身；递归查左右子树，若两侧均非空则当前节点为 LCA；否则返回非空的那侧。  
**时间复杂度**：O(n)　**空间复杂度**：O(h)
```python
def lowestCommonAncestor(root, p, q):
    if not root or root == p or root == q: return root
    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)
    if left and right: return root
    return left or right
```

---

### 297. 二叉树的序列化与反序列化 (Serialize and Deserialize Binary Tree) ★Hard
**核心思路**：前序遍历序列化，用逗号分隔，空节点用 "#" 表示。反序列化时按前序递归消费字符串列表。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
```python
class Codec:
    def serialize(self, root):
        res = []
        def dfs(node):
            if not node: res.append('#'); return
            res.append(str(node.val))
            dfs(node.left); dfs(node.right)
        dfs(root)
        return ','.join(res)
    def deserialize(self, data):
        vals = iter(data.split(','))
        def dfs():
            v = next(vals)
            if v == '#': return None
            node = TreeNode(int(v))
            node.left = dfs(); node.right = dfs()
            return node
        return dfs()
```

---

### 543. 二叉树的直径 (Diameter of Binary Tree) ★Easy
**核心思路**：DFS 计算每个节点左右子树深度，直径 = 左深度 + 右深度，维护全局最大值（不一定过根节点）。  
**时间复杂度**：O(n)　**空间复杂度**：O(h)
```python
def diameterOfBinaryTree(root):
    res = [0]
    def dfs(node):
        if not node: return 0
        l, r = dfs(node.left), dfs(node.right)
        res[0] = max(res[0], l + r)
        return 1 + max(l, r)
    dfs(root)
    return res[0]
```

---

## 四、动态规划

### 70. 爬楼梯 (Climbing Stairs) ★Easy
**核心思路**：到第 n 阶可由第 n-1 或 n-2 阶到达，f(n) = f(n-1) + f(n-2)，即斐波那契数列。滚动变量节省空间。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def climbStairs(n):
    a, b = 1, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b
```

---

### 72. 编辑距离 (Edit Distance) ★Hard
**核心思路**：dp[i][j] 表示将 word1[:i] 变为 word2[:j] 的最少操作数。字符相同则继承，否则取插入、删除、替换三种操作的最小值加一。  
**时间复杂度**：O(mn)　**空间复杂度**：O(mn)
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

### 322. 零钱兑换 (Coin Change) ★Medium
**核心思路**：完全背包，dp[i] 表示凑成金额 i 所需最少硬币数。对每种硬币，若 i >= coin 则 dp[i] = min(dp[i], dp[i-coin]+1)。  
**时间复杂度**：O(amount × n)　**空间复杂度**：O(amount)
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

### 300. 最长递增子序列 (Longest Increasing Subsequence) ★Medium
**核心思路**：patience sorting / 贪心 + 二分，维护一个 tails 数组，每次用二分找当前数字的位置替换或追加，长度即为 LIS。  
**时间复杂度**：O(n log n)　**空间复杂度**：O(n)
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

### 416. 分割等和子集 (Partition Equal Subset Sum) ★Medium
**核心思路**：0-1 背包问题，判断能否从数组中选若干数使其和等于 total/2。dp[j] 表示能否凑出和为 j。  
**时间复杂度**：O(n × target)　**空间复杂度**：O(target)
```python
def canPartition(nums):
    total = sum(nums)
    if total % 2: return False
    target = total // 2
    dp = {0}
    for n in nums:
        dp |= {s + n for s in dp if s + n <= target}
    return target in dp
```

---

### 62. 不同路径 (Unique Paths) ★Medium
**核心思路**：dp[i][j] = dp[i-1][j] + dp[i][j-1]，从左上到右下，每格等于上方加左方路径数之和。可压缩为一维。  
**时间复杂度**：O(mn)　**空间复杂度**：O(n)
```python
def uniquePaths(m, n):
    dp = [1] * n
    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j-1]
    return dp[-1]
```

---

### 1143. 最长公共子序列 (Longest Common Subsequence) ★Medium
**核心思路**：经典 DP，dp[i][j] 表示 text1[:i] 和 text2[:j] 的 LCS 长度。字符相等则 +1，否则取上或左的最大值。  
**时间复杂度**：O(mn)　**空间复杂度**：O(mn)
```python
def longestCommonSubsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
            else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

---

### 139. 单词拆分 (Word Break) ★Medium
**核心思路**：dp[i] 表示 s[:i] 能否被字典中的单词拼接。对每个位置 j，若 dp[j] 为真且 s[j:i] 在字典中，则 dp[i] 为真。  
**时间复杂度**：O(n²)　**空间复杂度**：O(n)
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

### 152. 乘积最大子数组 (Maximum Product Subarray) ★Medium
**核心思路**：同时维护以当前元素结尾的最大积和最小积（负数乘以最小值可变最大），每步更新全局最大值。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

## 五、栈 / 队列 / 单调栈

### 20. 有效的括号 (Valid Parentheses) ★Easy
**核心思路**：用栈存左括号，遇到右括号检查栈顶是否为对应左括号。最终栈为空则合法。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 84. 柱状图中最大的矩形 (Largest Rectangle in Histogram) ★Hard
**核心思路**：单调递增栈，当遇到比栈顶更矮的柱子时，弹出栈顶并计算以该柱子为高度的矩形面积，宽度由当前索引和新栈顶决定。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 155. 最小栈 (Min Stack) ★Medium
**核心思路**：用辅助栈同步维护当前最小值，push 时将 min(val, 辅助栈顶) 压入辅助栈，pop 时两栈同步弹出。  
**时间复杂度**：O(1)　**空间复杂度**：O(n)
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

### 239. 滑动窗口最大值 (Sliding Window Maximum) ★Hard
**核心思路**：单调递减双端队列存索引，队首始终是窗口最大值。入队前弹出所有比当前值小的索引，超出窗口的队首也弹出。  
**时间复杂度**：O(n)　**空间复杂度**：O(k)
```python
from collections import deque
def maxSlidingWindow(nums, k):
    dq, res = deque(), []
    for i, n in enumerate(nums):
        while dq and nums[dq[-1]] <= n: dq.pop()
        dq.append(i)
        if dq[0] == i - k: dq.popleft()
        if i >= k - 1: res.append(nums[dq[0]])
    return res
```

---

### 394. 字符串解码 (Decode String) ★Medium
**核心思路**：用栈存当前字符串和倍数，遇到 `[` 压栈，遇到 `]` 弹出并将当前字符串重复指定次数后与弹出的字符串拼接。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
```python
def decodeString(s):
    stack = []
    cur_str, cur_num = '', 0
    for c in s:
        if c.isdigit():
            cur_num = cur_num * 10 + int(c)
        elif c == '[':
            stack.append((cur_str, cur_num))
            cur_str, cur_num = '', 0
        elif c == ']':
            prev_str, num = stack.pop()
            cur_str = prev_str + cur_str * num
        else:
            cur_str += c
    return cur_str
```

---

## 六、哈希表

### 49. 字母异位词分组 (Group Anagrams) ★Medium
**核心思路**：对每个单词排序后的结果作为哈希 key，将同组异位词归并到同一列表。或用字母计数元组作 key。  
**时间复杂度**：O(n·k log k)　**空间复杂度**：O(nk)
```python
from collections import defaultdict
def groupAnagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        groups[tuple(sorted(s))].append(s)
    return list(groups.values())
```

---

### 128. 最长连续序列 (Longest Consecutive Sequence) ★Medium
**核心思路**：将所有数放入哈希集合，只从序列起点（num-1 不在集合中）开始向右扩展，统计连续长度，避免重复遍历。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 560. 和为K的子数组 (Subarray Sum Equals K) ★Medium
**核心思路**：前缀和 + 哈希表，记录每个前缀和出现的次数，对每个位置检查 `prefix - k` 是否存在于哈希表中。  
**时间复杂度**：O(n)　**空间复杂度**：O(n)
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

### 380. O(1) 时间插入、删除和获取随机元素 (Insert Delete GetRandom O(1)) ★Medium
**核心思路**：数组存储元素（支持随机访问），哈希表存元素到索引的映射（支持 O(1) 查找/删除）。删除时用尾元素填补被删位置。  
**时间复杂度**：O(1)均摊　**空间复杂度**：O(n)
```python
import random
class RandomizedSet:
    def __init__(self):
        self.nums = []; self.idx = {}
    def insert(self, val):
        if val in self.idx: return False
        self.idx[val] = len(self.nums); self.nums.append(val); return True
    def remove(self, val):
        if val not in self.idx: return False
        i = self.idx[val]; last = self.nums[-1]
        self.nums[i] = last; self.idx[last] = i
        self.nums.pop(); del self.idx[val]; return True
    def getRandom(self): return random.choice(self.nums)
```

---

## 七、二分查找

### 33. 搜索旋转排序数组 (Search in Rotated Sorted Array) ★Medium
**核心思路**：二分时先判断哪半边是有序的，再判断目标是否在有序部分内，据此缩小查找范围。  
**时间复杂度**：O(log n)　**空间复杂度**：O(1)
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

### 153. 寻找旋转排序数组中的最小值 (Find Minimum in Rotated Sorted Array) ★Medium
**核心思路**：二分比较 mid 与 right，若 nums[mid] > nums[right] 则最小值在右半边，否则在左半边（含 mid）。  
**时间复杂度**：O(log n)　**空间复杂度**：O(1)
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

### 34. 在排序数组中查找元素的第一个和最后一个位置 ★Medium
**核心思路**：两次二分，第一次找左边界（第一个 >= target 的位置），第二次找右边界（最后一个 <= target 的位置）。  
**时间复杂度**：O(log n)　**空间复杂度**：O(1)
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

### 4. 寻找两个正序数组的中位数 (Median of Two Sorted Arrays) ★Hard
**核心思路**：对较短数组二分，找到一个切割点使两数组左半部分合并后恰好占一半元素，利用边界条件确定中位数。  
**时间复杂度**：O(log(min(m,n)))　**空间复杂度**：O(1)
```python
def findMedianSortedArrays(nums1, nums2):
    A, B = nums1, nums2
    if len(A) > len(B): A, B = B, A
    m, n = len(A), len(B)
    lo, hi = 0, m
    while lo <= hi:
        i = (lo + hi) // 2
        j = (m + n + 1) // 2 - i
        max_l_a = float('-inf') if i == 0 else A[i-1]
        min_r_a = float('inf') if i == m else A[i]
        max_l_b = float('-inf') if j == 0 else B[j-1]
        min_r_b = float('inf') if j == n else B[j]
        if max_l_a <= min_r_b and max_l_b <= min_r_a:
            if (m + n) % 2 == 1: return max(max_l_a, max_l_b)
            return (max(max_l_a, max_l_b) + min(min_r_a, min_r_b)) / 2
        elif max_l_a > min_r_b: hi = i - 1
        else: lo = i + 1
```

---

## 八、回溯

### 39. 组合总和 (Combination Sum) ★Medium
**核心思路**：回溯，可重复选同一元素。对候选数组排序后，若当前数已大于 remaining 则剪枝。每次递归从当前索引开始选。  
**时间复杂度**：O(N^(target/min))　**空间复杂度**：O(target/min)
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

### 46. 全排列 (Permutations) ★Medium
**核心思路**：回溯，维护已使用集合，对每个未使用元素加入路径并递归，返回时撤销选择。  
**时间复杂度**：O(n·n!)　**空间复杂度**：O(n)
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

### 78. 子集 (Subsets) ★Medium
**核心思路**：回溯，每进入一层就记录当前路径为一个子集。从 start 开始枚举，避免重复。  
**时间复杂度**：O(n·2^n)　**空间复杂度**：O(n)
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

### 79. 单词搜索 (Word Search) ★Medium
**核心思路**：DFS + 回溯，从每个格子出发，沿四个方向匹配字符，访问过的位置临时标记，回溯时恢复。  
**时间复杂度**：O(m·n·4^L)　**空间复杂度**：O(L)
```python
def exist(board, word):
    m, n = len(board), len(board[0])
    def dfs(i, j, k):
        if k == len(word): return True
        if not (0 <= i < m and 0 <= j < n) or board[i][j] != word[k]: return False
        tmp, board[i][j] = board[i][j], '#'
        found = any(dfs(i+di, j+dj, k+1) for di, dj in [(0,1),(0,-1),(1,0),(-1,0)])
        board[i][j] = tmp
        return found
    return any(dfs(i, j, 0) for i in range(m) for j in range(n))
```

---

### 51. N皇后 (N-Queens) ★Hard
**核心思路**：逐行放置皇后，用集合记录已占用的列、正对角线、反对角线，满足约束则进入下一行，回溯时移除标记。  
**时间复杂度**：O(n!)　**空间复杂度**：O(n)
```python
def solveNQueens(n):
    res = []
    cols = diag1 = diag2 = set()  # actually use separate sets
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

## 九、贪心

### 55. 跳跃游戏 (Jump Game) ★Medium
**核心思路**：维护当前能到达的最远位置 `max_reach`，遍历每个位置若当前位置可达则更新最远距离，最终判断能否到达末尾。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def canJump(nums):
    max_reach = 0
    for i, n in enumerate(nums):
        if i > max_reach: return False
        max_reach = max(max_reach, i + n)
    return True
```

---

### 45. 跳跃游戏 II (Jump Game II) ★Medium
**核心思路**：贪心，维护当前跳跃能到达的最远边界和下次跳跃的最远位置。到达边界时步数加一，更新边界。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

### 435. 无重叠区间 (Non-overlapping Intervals) ★Medium
**核心思路**：按结束时间排序，贪心保留结束最早的区间，遇到与当前区间重叠的则移除（计数），否则更新当前区间。  
**时间复杂度**：O(n log n)　**空间复杂度**：O(1)
```python
def eraseOverlapIntervals(intervals):
    intervals.sort(key=lambda x: x[1])
    res, end = 0, float('-inf')
    for s, e in intervals:
        if s >= end: end = e
        else: res += 1
    return res
```

---

### 763. 划分字母区间 (Partition Labels) ★Medium
**核心思路**：记录每个字母最后出现位置，贪心扩展当前片段的边界，到达边界时切分，开始新片段。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
```python
def partitionLabels(s):
    last = {c: i for i, c in enumerate(s)}
    res, start = [], 0
    end = 0
    for i, c in enumerate(s):
        end = max(end, last[c])
        if i == end:
            res.append(end - start + 1)
            start = i + 1
    return res
```

---

### 134. 加油站 (Gas Station) ★Medium
**核心思路**：总油量 >= 总消耗则一定有解。从头扫描，维护当前油箱余量，一旦为负则从下一站重新开始，最终起点即为答案。  
**时间复杂度**：O(n)　**空间复杂度**：O(1)
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

## 十、图 / 拓扑排序

### 200. 岛屿数量 (Number of Islands) ★Medium
**核心思路**：DFS/BFS，遍历网格，遇到 '1' 则计数加一，并将连通的所有 '1' 标记为已访问（改为 '0'）。  
**时间复杂度**：O(mn)　**空间复杂度**：O(mn)
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

### 207. 课程表 (Course Schedule) ★Medium
**核心思路**：拓扑排序（BFS/Kahn 算法），建图后计算入度，将入度为 0 的节点入队，逐步移除并减少邻居入度，判断最终是否所有课程都处理完。  
**时间复杂度**：O(V+E)　**空间复杂度**：O(V+E)
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

### 210. 课程表 II (Course Schedule II) ★Medium
**核心思路**：同课程表 I，但需记录拓扑排序的顺序。BFS 按入度 0 依次处理，记录每个出队节点即为合法修课顺序。  
**时间复杂度**：O(V+E)　**空间复杂度**：O(V+E)
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

### 994. 腐烂的橘子 (Rotting Oranges) ★Medium
**核心思路**：多源 BFS，将所有腐烂橘子同时入队为初始层，按分钟逐层扩散，最后检查是否有新鲜橘子剩余。  
**时间复杂度**：O(mn)　**空间复杂度**：O(mn)
```python
from collections import deque
def orangesRotting(grid):
    m, n = len(grid), len(grid[0])
    q = deque()
    fresh = 0
    for i in range(m):
        for j in range(n):
            if grid[i][j] == 2: q.append((i, j, 0))
            elif grid[i][j] == 1: fresh += 1
    minutes = 0
    while q:
        i, j, t = q.popleft()
        for di, dj in [(0,1),(0,-1),(1,0),(-1,0)]:
            ni, nj = i+di, j+dj
            if 0 <= ni < m and 0 <= nj < n and grid[ni][nj] == 1:
                grid[ni][nj] = 2; fresh -= 1
                q.append((ni, nj, t+1)); minutes = t+1
    return minutes if fresh == 0 else -1
```

---

### 547. 省份数量 (Number of Provinces) ★Medium
**核心思路**：并查集（Union-Find）或 DFS。并查集将每对直接相连的城市合并，最终统计根节点数量（连通分量数）。  
**时间复杂度**：O(n²·α(n))　**空间复杂度**：O(n)
```python
def findCircleNum(isConnected):
    n = len(isConnected)
    parent = list(range(n))
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]; x = parent[x]
        return x
    def union(a, b):
        pa, pb = find(a), find(b)
        if pa != pb: parent[pa] = pb; return True
        return False
    count = n
    for i in range(n):
        for j in range(i+1, n):
            if isConnected[i][j] and union(i, j): count -= 1
    return count
```

---

### 684. 冗余连接 (Redundant Connection) ★Medium
**核心思路**：并查集，依次处理每条边，若边的两端已连通则该边为冗余边；否则合并两端。  
**时间复杂度**：O(n·α(n))　**空间复杂度**：O(n)
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

*整理完成 · 共覆盖 LeetCode 高频 100 题核心题目 · 持续更新*
