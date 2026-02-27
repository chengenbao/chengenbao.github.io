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

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 240" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <defs>
    <marker id="arrowB" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#1565c0" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="arrowR" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#c62828" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
  <!-- Title -->
  <text x="260" y="20" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">对撞指针示例：nums=[1,3,5,7,9,11], target=12</text>
  <!-- Array boxes: y=30, h=36 → bottom=66 -->
  <rect x="20" y="30" width="60" height="36" rx="6" fill="#e8f4f8" stroke="#90caf9" stroke-width="1.5"/>
  <text x="50" y="53" text-anchor="middle" fill="#1a237e" font-weight="bold">1</text>
  <rect x="100" y="30" width="60" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="130" y="53" text-anchor="middle" fill="#333">3</text>
  <rect x="180" y="30" width="60" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="210" y="53" text-anchor="middle" fill="#333">5</text>
  <rect x="260" y="30" width="60" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="290" y="53" text-anchor="middle" fill="#333">7</text>
  <rect x="340" y="30" width="60" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="370" y="53" text-anchor="middle" fill="#333">9</text>
  <rect x="420" y="30" width="60" height="36" rx="6" fill="#e8f5e9" stroke="#a5d6a7" stroke-width="1.5"/>
  <text x="450" y="53" text-anchor="middle" fill="#1b5e20" font-weight="bold">11</text>
  <!-- Index labels: y=80 (14px below bottom of boxes) -->
  <text x="50"  y="80" text-anchor="middle" fill="#888" font-size="11">0</text>
  <text x="130" y="80" text-anchor="middle" fill="#888" font-size="11">1</text>
  <text x="210" y="80" text-anchor="middle" fill="#888" font-size="11">2</text>
  <text x="290" y="80" text-anchor="middle" fill="#888" font-size="11">3</text>
  <text x="370" y="80" text-anchor="middle" fill="#888" font-size="11">4</text>
  <text x="450" y="80" text-anchor="middle" fill="#888" font-size="11">5</text>
  <!-- Pointer arrows: start y=88, end y=100 -->
  <line x1="50"  y1="88" x2="50"  y2="100" stroke="#1565c0" stroke-width="2" marker-end="url(#arrowB)"/>
  <line x1="450" y1="88" x2="450" y2="100" stroke="#c62828" stroke-width="2" marker-end="url(#arrowR)"/>
  <!-- Pointer labels: y=116 -->
  <text x="50"  y="116" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="13">L</text>
  <text x="450" y="116" text-anchor="middle" fill="#c62828" font-weight="bold" font-size="13">R</text>
  <!-- Sum annotation: y=138 -->
  <text x="260" y="138" text-anchor="middle" fill="#2e7d32" font-weight="bold" font-size="13">L + R = 1 + 11 = 12 = target ✓ 找到！</text>
  <!-- Rule boxes: y=150, h=44 → bottom=194 -->
  <rect x="20"  y="150" width="220" height="44" rx="6" fill="#fff8e1" stroke="#ffe082" stroke-width="1.5"/>
  <text x="130" y="169" text-anchor="middle" fill="#e65100" font-size="12">L+R &lt; target</text>
  <text x="130" y="187" text-anchor="middle" fill="#e65100" font-size="12">→ L 右移（增大 sum）</text>
  <rect x="280" y="150" width="220" height="44" rx="6" fill="#fce4ec" stroke="#f48fb1" stroke-width="1.5"/>
  <text x="390" y="169" text-anchor="middle" fill="#880e4f" font-size="12">L+R &gt; target</text>
  <text x="390" y="187" text-anchor="middle" fill="#880e4f" font-size="12">→ R 左移（减小 sum）</text>
</svg>
</div>

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
- [LC 167 两数之和 II（有序数组）](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)：直接套对撞模板，O(n)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
left, right = 0, len(nums) - 1
while left < right:
    s = nums[left] + nums[right]
    if s == target:
        return [left + 1, right + 1]
    elif s < target:
        left += 1
    else:
        right -= 1
```

</details>

- [LC 15 三数之和](https://leetcode.cn/problems/3sum/)：排序后固定一个数，对剩余数组用对撞指针，注意去重。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
nums.sort()
res = []
for i in range(len(nums) - 2):
    if i > 0 and nums[i] == nums[i-1]:
        continue
    left, right = i + 1, len(nums) - 1
    while left < right:
        s = nums[i] + nums[left] + nums[right]
        if s == 0:
            res.append([nums[i], nums[left], nums[right]])
            while left < right and nums[left] == nums[left+1]: left += 1
            while left < right and nums[right] == nums[right-1]: right -= 1
            left += 1; right -= 1
        elif s < 0:
            left += 1
        else:
            right -= 1
return res
```

</details>

- [LC 11 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)：容量 = `min(h[l], h[r]) * (r-l)`，每次移动较矮一侧指针。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
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

</details>

- [LC 125 验证回文串](https://leetcode.cn/problems/valid-palindrome/)：左右同时向中间走，忽略非字母数字字符。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
l, r = 0, len(s) - 1
while l < r:
    while l < r and not s[l].isalnum(): l += 1
    while l < r and not s[r].isalnum(): r -= 1
    if s[l].lower() != s[r].lower():
        return False
    l += 1; r -= 1
return True
```

</details>

- [LC 42 接雨水](https://leetcode.cn/problems/trapping-rain-water/)：左右各维护最大高度 `maxL/maxR`，较小侧计算积水。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
l, r = 0, len(height) - 1
lmax = rmax = ans = 0
while l < r:
    if height[l] < height[r]:
        lmax = max(lmax, height[l])
        ans += lmax - height[l]
        l += 1
    else:
        rmax = max(rmax, height[r])
        ans += rmax - height[r]
        r -= 1
return ans
```

</details>


---

### 1.2 快慢指针（链表环检测 & 中点）

**适用场景**：链表环检测、找链表中点/倒数第k个节点、判断回文链表。

**核心思想**：慢指针每次走1步，快指针每次走2步。若有环，两者必在环内相遇；若无环，快指针先到 null。

> 💡 **图示**

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 210" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <defs>
    <marker id="arr-link" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#90a4ae" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="arr-slow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#1565c0" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="arr-fast" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#c62828" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
  <text x="260" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">快慢指针：找链表中点</text>
  <!-- 节点方块 y=28 h=36 bottom=64 -->
  <rect x="20"  y="28" width="56" height="36" rx="6" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="48"  y="51" text-anchor="middle" font-weight="bold" fill="#1a237e">1</text>
  <rect x="105" y="28" width="56" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="133" y="51" text-anchor="middle" fill="#333">2</text>
  <rect x="190" y="28" width="56" height="36" rx="6" fill="#e8f5e9" stroke="#81c784" stroke-width="1.5"/>
  <text x="218" y="51" text-anchor="middle" font-weight="bold" fill="#2e7d32">3</text>
  <rect x="275" y="28" width="56" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="303" y="51" text-anchor="middle" fill="#333">4</text>
  <rect x="360" y="28" width="56" height="36" rx="6" fill="#fce4ec" stroke="#f48fb1" stroke-width="1.5"/>
  <text x="388" y="51" text-anchor="middle" font-weight="bold" fill="#880e4f">5</text>
  <text x="428" y="51" text-anchor="start" fill="#999" font-size="12">NULL</text>
  <!-- 链表箭头 -->
  <line x1="77"  y1="46" x2="104" y2="46" stroke="#90a4ae" stroke-width="1.5" marker-end="url(#arr-link)"/>
  <line x1="162" y1="46" x2="189" y2="46" stroke="#90a4ae" stroke-width="1.5" marker-end="url(#arr-link)"/>
  <line x1="247" y1="46" x2="274" y2="46" stroke="#90a4ae" stroke-width="1.5" marker-end="url(#arr-link)"/>
  <line x1="332" y1="46" x2="359" y2="46" stroke="#90a4ae" stroke-width="1.5" marker-end="url(#arr-link)"/>
  <line x1="417" y1="46" x2="424" y2="46" stroke="#90a4ae" stroke-width="1.5" marker-end="url(#arr-link)"/>
  <!-- 初始位置标注: y=72 (8px below boxes) -->
  <text x="48"  y="74" text-anchor="middle" fill="#1565c0" font-size="11">S₀</text>
  <text x="48"  y="87" text-anchor="middle" fill="#c62828" font-size="11">F₀</text>
  <!-- 终止位置指针箭头: 从 y=72 开始 -->
  <line x1="218" y1="70" x2="218" y2="80" stroke="#1565c0" stroke-width="2" marker-end="url(#arr-slow)"/>
  <text x="218" y="94" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="12">S（中点）</text>
  <line x1="388" y1="70" x2="388" y2="80" stroke="#c62828" stroke-width="2" marker-end="url(#arr-fast)"/>
  <text x="388" y="94" text-anchor="middle" fill="#c62828" font-weight="bold" font-size="12">F（末尾）</text>
  <!-- 说明框: y=108 h=42 -->
  <rect x="20"  y="108" width="230" height="42" rx="6" fill="#e3f2fd" stroke="#90caf9" stroke-width="1"/>
  <text x="135" y="125" text-anchor="middle" fill="#1565c0" font-size="12">slow 每次走 1 步</text>
  <text x="135" y="142" text-anchor="middle" fill="#1565c0" font-size="12">fast 到末尾时 slow 在中点</text>
  <rect x="265" y="108" width="230" height="42" rx="6" fill="#fce4ec" stroke="#f48fb1" stroke-width="1"/>
  <text x="380" y="125" text-anchor="middle" fill="#c62828" font-size="12">fast 每次走 2 步</text>
  <text x="380" y="142" text-anchor="middle" fill="#c62828" font-size="12">节点数=5，中点=节点 3 ✓</text>
  <!-- 步骤说明 -->
  <text x="260" y="168" text-anchor="middle" fill="#555" font-size="11">初始 S₀=F₀=节点1 → 走3步后 S=节点3，F=节点5</text>
  <rect x="80" y="178" width="360" height="24" rx="5" fill="#e8f5e9" stroke="#81c784" stroke-width="1.5"/>
  <text x="260" y="194" text-anchor="middle" fill="#2e7d32" font-weight="bold" font-size="12">fast 抵达末尾 → slow 指向中点 节点3 ✓</text>
</svg>
</div>

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
- [LC 141 环形链表](https://leetcode.cn/problems/linked-list-cycle/)：快慢指针是否相遇。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow is fast:
        return True
return False
```

</details>

- [LC 142 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)：Floyd算法找入口。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow is fast:
        ptr = head
        while ptr is not slow:
            ptr = ptr.next
            slow = slow.next
        return ptr
return None
```

</details>

- [LC 876 链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)：快慢指针找中点。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
return slow
```

</details>

- [LC 234 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)：找中点 → 反转后半段 → 比较。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
# 找中点
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
# 反转后半
prev, cur = None, slow
while cur:
    nxt = cur.next
    cur.next = prev
    prev, cur = cur, nxt
# 比较
left, right = head, prev
while right:
    if left.val != right.val:
        return False
    left = left.next
    right = right.next
return True
```

</details>

- [LC 19 删除链表倒数第N个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)：快指针先走N步，再同步前进。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
dummy = ListNode(0, head)
fast = slow = dummy
for _ in range(n):
    fast = fast.next
while fast.next:
    fast = fast.next
    slow = slow.next
slow.next = slow.next.next
return dummy.next
```

</details>


---

### 1.3 滑动窗口（连续子数组/子串）

**适用场景**：求满足条件的最长/最短连续子数组、子串；窗口内元素的统计量随窗口扩缩单调变化。

**核心思想**：维护 `[left, right)` 窗口，right 不断右移扩大窗口，不满足条件时移动 left 收缩。

> 💡 **图示**

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 230" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <text x="260" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">滑动窗口：无重复字符最长子串</text>
  <!-- 字符数组 y=26 h=34 bottom=60 -->
  <text x="20" y="50" fill="#666" font-size="12">数组:</text>
  <rect x="62"  y="26" width="52" height="34" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="88"  y="48" text-anchor="middle" fill="#333" font-weight="bold">a</text>
  <rect x="122" y="26" width="52" height="34" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="148" y="48" text-anchor="middle" fill="#333" font-weight="bold">b</text>
  <rect x="182" y="26" width="52" height="34" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="208" y="48" text-anchor="middle" fill="#333" font-weight="bold">c</text>
  <rect x="242" y="26" width="52" height="34" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="268" y="48" text-anchor="middle" fill="#333" font-weight="bold">b</text>
  <rect x="302" y="26" width="52" height="34" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="328" y="48" text-anchor="middle" fill="#333" font-weight="bold">a</text>
  <!-- 索引 y=74 -->
  <text x="88"  y="74" text-anchor="middle" fill="#aaa" font-size="11">0</text>
  <text x="148" y="74" text-anchor="middle" fill="#aaa" font-size="11">1</text>
  <text x="208" y="74" text-anchor="middle" fill="#aaa" font-size="11">2</text>
  <text x="268" y="74" text-anchor="middle" fill="#aaa" font-size="11">3</text>
  <text x="328" y="74" text-anchor="middle" fill="#aaa" font-size="11">4</text>
  <!-- 步骤①: 窗口 [0,2] y=82 h=30 -->
  <text x="16" y="102" fill="#1565c0" font-size="11" font-weight="bold">①</text>
  <rect x="57" y="82" width="182" height="30" rx="6" fill="rgba(33,150,243,0.12)" stroke="#42a5f5" stroke-width="2" stroke-dasharray="4,2"/>
  <text x="67"  y="102" fill="#1565c0" font-size="11" font-weight="bold">L</text>
  <text x="227" y="102" fill="#1565c0" font-size="11" font-weight="bold">R</text>
  <text x="390" y="101" fill="#2e7d32" font-size="12" font-weight="bold">扩展 len=3</text>
  <!-- 步骤②: 窗口 [1,3] y=124 h=30 -->
  <text x="16" y="144" fill="#e65100" font-size="11" font-weight="bold">②</text>
  <rect x="117" y="124" width="182" height="30" rx="6" fill="rgba(255,152,0,0.12)" stroke="#ffa726" stroke-width="2" stroke-dasharray="4,2"/>
  <text x="127" y="144" fill="#e65100" font-size="11" font-weight="bold">L</text>
  <text x="287" y="144" fill="#e65100" font-size="11" font-weight="bold">R</text>
  <text x="390" y="138" fill="#e65100" font-size="12" font-weight="bold">收缩 len=3</text>
  <text x="390" y="152" fill="#e65100" font-size="11">（遇重复 'b'）</text>
  <!-- 步骤③: 窗口 [2,4] y=166 h=30 -->
  <text x="16" y="186" fill="#2e7d32" font-size="11" font-weight="bold">③</text>
  <rect x="177" y="166" width="182" height="30" rx="6" fill="rgba(76,175,80,0.12)" stroke="#66bb6a" stroke-width="2" stroke-dasharray="4,2"/>
  <text x="187" y="186" fill="#2e7d32" font-size="11" font-weight="bold">L</text>
  <text x="347" y="186" fill="#2e7d32" font-size="11" font-weight="bold">R</text>
  <text x="390" y="186" fill="#2e7d32" font-size="12" font-weight="bold">扩展 len=3</text>
  <!-- 结论 -->
  <text x="260" y="220" text-anchor="middle" fill="#555" font-size="11">最大窗口长度 = 3（"abc" 或 "cba"）</text>
</svg>
</div>

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
- [LC 76 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)：经典滑动窗口，上方模板直接适用。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict
need = defaultdict(int)
for c in t: need[c] += 1
window = defaultdict(int)
left = right = valid = 0
start, min_len = 0, float('inf')
while right < len(s):
    c = s[right]; right += 1
    if c in need:
        window[c] += 1
        if window[c] == need[c]: valid += 1
    while valid == len(need):
        if right - left < min_len:
            start, min_len = left, right - left
        d = s[left]; left += 1
        if d in need:
            if window[d] == need[d]: valid -= 1
            window[d] -= 1
return s[start:start+min_len] if min_len != float('inf') else ""
```

</details>

- [LC 438 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)：固定窗口大小 = `len(p)`。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import Counter
p_cnt = Counter(p)
window = Counter()
res = []
for i, c in enumerate(s):
    window[c] += 1
    if i >= len(p):
        left_c = s[i - len(p)]
        window[left_c] -= 1
        if window[left_c] == 0:
            del window[left_c]
    if window == p_cnt:
        res.append(i - len(p) + 1)
return res
```

</details>

- [LC 3 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)：窗口内字符不重复，`valid`改为set计数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
seen = set()
left = res = 0
for right, c in enumerate(s):
    while c in seen:
        seen.remove(s[left])
        left += 1
    seen.add(c)
    res = max(res, right - left + 1)
return res
```

</details>

- [LC 209 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)：窗口和 ≥ target 时尝试收缩。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
left = total = 0
res = float('inf')
for right, num in enumerate(nums):
    total += num
    while total >= target:
        res = min(res, right - left + 1)
        total -= nums[left]
        left += 1
return res if res != float('inf') else 0
```

</details>

- [LC 567 字符串的排列](https://leetcode.cn/problems/permutation-in-string/)：固定窗口，判断窗口是否是排列。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import Counter
p_cnt = Counter(p)
window = Counter(s[:len(p)])
if window == p_cnt: return True
for i in range(len(p), len(s)):
    window[s[i]] += 1
    left_c = s[i - len(p)]
    window[left_c] -= 1
    if window[left_c] == 0: del window[left_c]
    if window == p_cnt: return True
return False
```

</details>


---

## 2. 前缀和 & 差分数组

### 2.1 前缀和

**适用场景**：频繁查询数组区间和、统计子数组和等于 k 的个数。

**核心思想**：`prefix[i] = arr[0] + ... + arr[i-1]`，区间和 `[l,r] = prefix[r+1] - prefix[l]`，O(1) 查询。

> 💡 **图示**

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 190" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <text x="260" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">前缀和：O(1) 区间查询</text>
  <text x="8" y="49" fill="#666" font-size="11">原数组</text>
  <rect x="122" y="26" width="160" height="36" rx="4" fill="rgba(33,150,243,0.15)" stroke="#90caf9" stroke-width="1.5" stroke-dasharray="4,2"/>
  <rect x="68" y="28" width="52" height="32" rx="6" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="94" y="49" text-anchor="middle" fill="#1a237e">1</text>
  <rect x="124" y="28" width="52" height="32" rx="6" fill="#bbdefb" stroke="#64b5f6" stroke-width="2"/>
  <text x="150" y="49" text-anchor="middle" fill="#1a237e" font-weight="bold">2</text>
  <rect x="180" y="28" width="52" height="32" rx="6" fill="#bbdefb" stroke="#64b5f6" stroke-width="2"/>
  <text x="206" y="49" text-anchor="middle" fill="#1a237e" font-weight="bold">3</text>
  <rect x="236" y="28" width="52" height="32" rx="6" fill="#bbdefb" stroke="#64b5f6" stroke-width="2"/>
  <text x="262" y="49" text-anchor="middle" fill="#1a237e" font-weight="bold">4</text>
  <rect x="292" y="28" width="52" height="32" rx="6" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="318" y="49" text-anchor="middle" fill="#1a237e">5</text>
  <text x="94" y="72" text-anchor="middle" fill="#aaa" font-size="11">0</text>
  <text x="150" y="72" text-anchor="middle" fill="#aaa" font-size="11">1</text>
  <text x="206" y="72" text-anchor="middle" fill="#aaa" font-size="11">2</text>
  <text x="262" y="72" text-anchor="middle" fill="#aaa" font-size="11">3</text>
  <text x="318" y="72" text-anchor="middle" fill="#aaa" font-size="11">4</text>
  <text x="150" y="84" text-anchor="middle" fill="#1565c0" font-size="11">l=1</text>
  <text x="262" y="84" text-anchor="middle" fill="#1565c0" font-size="11">r=3</text>
  <text x="8" y="115" fill="#666" font-size="11">前缀和</text>
  <rect x="68" y="94" width="52" height="32" rx="6" fill="#fff9c4" stroke="#ffee58" stroke-width="2"/>
  <text x="94" y="115" text-anchor="middle" fill="#f57f17" font-weight="bold">0</text>
  <text x="94" y="130" text-anchor="middle" fill="#f9a825" font-size="10">哨兵</text>
  <rect x="124" y="94" width="52" height="32" rx="6" fill="#c8e6c9" stroke="#81c784" stroke-width="2"/>
  <text x="150" y="115" text-anchor="middle" fill="#2e7d32" font-weight="bold">1</text>
  <rect x="180" y="94" width="52" height="32" rx="6" fill="#e8f5e9" stroke="#a5d6a7" stroke-width="1.5"/>
  <text x="206" y="115" text-anchor="middle" fill="#388e3c">3</text>
  <rect x="236" y="94" width="52" height="32" rx="6" fill="#e8f5e9" stroke="#a5d6a7" stroke-width="1.5"/>
  <text x="262" y="115" text-anchor="middle" fill="#388e3c">6</text>
  <rect x="292" y="94" width="52" height="32" rx="6" fill="#c8e6c9" stroke="#81c784" stroke-width="2"/>
  <text x="318" y="115" text-anchor="middle" fill="#2e7d32" font-weight="bold">10</text>
  <rect x="348" y="94" width="52" height="32" rx="6" fill="#e8f5e9" stroke="#a5d6a7" stroke-width="1.5"/>
  <text x="374" y="115" text-anchor="middle" fill="#388e3c">15</text>
  <text x="94" y="138" text-anchor="middle" fill="#aaa" font-size="11">0</text>
  <text x="150" y="138" text-anchor="middle" fill="#aaa" font-size="11">1</text>
  <text x="206" y="138" text-anchor="middle" fill="#aaa" font-size="11">2</text>
  <text x="262" y="138" text-anchor="middle" fill="#aaa" font-size="11">3</text>
  <text x="318" y="138" text-anchor="middle" fill="#aaa" font-size="11">4</text>
  <text x="374" y="138" text-anchor="middle" fill="#aaa" font-size="11">5</text>
  <rect x="68" y="150" width="420" height="34" rx="6" fill="#fff8e1" stroke="#ffe082" stroke-width="1.5"/>
  <text x="278" y="163" text-anchor="middle" fill="#e65100" font-size="12" font-weight="bold">区间[1,3]之和 = pre[r+1] &#x2212; pre[l] = pre[4] &#x2212; pre[1] = 10 &#x2212; 1 = 9</text>
  <text x="278" y="178" text-anchor="middle" fill="#9e9e9e" font-size="11">（闭区间 [l, r]，查询时间 O(1)）</text>
</svg>
</div>

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
- [LC 303 区域和检索](https://leetcode.cn/problems/range-sum-query-immutable/)：build_prefix + range_sum。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
class NumArray:
    def __init__(self, nums):
        self.prefix = [0] * (len(nums) + 1)
        for i, v in enumerate(nums):
            self.prefix[i+1] = self.prefix[i] + v

    def sumRange(self, left, right):
        return self.prefix[right+1] - self.prefix[left]
```

</details>

- [LC 560 和为K的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)：前缀和 + 哈希计数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict
count = defaultdict(int)
count[0] = 1
prefix_sum = res = 0
for num in nums:
    prefix_sum += num
    res += count[prefix_sum - k]
    count[prefix_sum] += 1
return res
```

</details>

- [LC 304 二维区域和检索](https://leetcode.cn/problems/range-sum-query-2d-immutable/)：二维前缀和模板。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
class NumMatrix:
    def __init__(self, matrix):
        m, n = len(matrix), len(matrix[0])
        self.pre = [[0]*(n+1) for _ in range(m+1)]
        for i in range(1, m+1):
            for j in range(1, n+1):
                self.pre[i][j] = (matrix[i-1][j-1]
                    + self.pre[i-1][j] + self.pre[i][j-1]
                    - self.pre[i-1][j-1])

    def sumRegion(self, r1, c1, r2, c2):
        return (self.pre[r2+1][c2+1] - self.pre[r1][c2+1]
                - self.pre[r2+1][c1] + self.pre[r1][c1])
```

</details>

- [LC 525 连续数组](https://leetcode.cn/problems/contiguous-array/)：0变-1，转化为子数组和为0。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict
index = defaultdict(int)
index[0] = -1
prefix = res = 0
for i, v in enumerate(nums):
    prefix += 1 if v == 1 else -1
    if prefix in index:
        res = max(res, i - index[prefix])
    else:
        index[prefix] = i
return res
```

</details>

- [LC 974 和可被 K 整除的子数组](https://leetcode.cn/problems/subarray-sums-divisible-by-k/)：`(prefix % k + k) % k` 哈希计数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict
count = defaultdict(int)
count[0] = 1
prefix = res = 0
for num in nums:
    prefix = (prefix + num) % k
    res += count[prefix]
    count[prefix] += 1
return res
```

</details>


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
- [LC 1109 航班预订统计](https://leetcode.cn/problems/corporate-flight-bookings/)：区间 `[first, last]` 都加 seats，最后输出结果数组。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = max(e for _, e, _ in bookings)
diff = [0] * (n + 2)
for first, last, seats in bookings:
    diff[first] += seats
    diff[last + 1] -= seats
res, cur = [], 0
for i in range(1, n + 1):
    cur += diff[i]
    res.append(cur)
return res
```

</details>

- [LC 1094 拼车](https://leetcode.cn/problems/car-pooling/)：差分数组统计每个站的乘客数，判断是否超容量。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
diff = [0] * 1001
for start, end, num in trips:
    diff[start] += num
    diff[end] -= num
cur = 0
for i in range(1001):
    cur += diff[i]
    if cur > capacity:
        return False
return True
```

</details>

- [LC 798 得分最高的最小轮调](https://leetcode.cn/problems/smallest-rotation-with-highest-score/)：差分统计每种轮调得分变化。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(nums)
diff = [0] * n
for i, v in enumerate(nums):
    # 原始得分：nums[i] > 0 时，轮调 i 次得分
    # 差分统计每个轮调次数的得分变化
    lo = (i - v + 1 + n) % n
    hi = i
    if lo <= hi:
        diff[lo] += 1
        if hi + 1 < n: diff[hi + 1] -= 1
    else:
        diff[lo] += 1
        diff[0] += 1
        if hi + 1 < n: diff[hi + 1] -= 1
score = [0] * n
cur = 0
for i in range(n):
    cur += diff[i]
    score[i] = cur
return score.index(max(score))
```

</details>


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
- [LC 1 两数之和](https://leetcode.cn/problems/two-sum/)：单次遍历 + dict，O(n)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
seen = {}
for i, num in enumerate(nums):
    if target - num in seen:
        return [seen[target - num], i]
    seen[num] = i
```

</details>

- [LC 49 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)：排序后的字符串作 key。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict
groups = defaultdict(list)
for word in strs:
    groups[tuple(sorted(word))].append(word)
return list(groups.values())
```

</details>

- [LC 128 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)：set + 从序列起点向右延伸。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
num_set = set(nums)
best = 0
for n in num_set:
    if n - 1 not in num_set:
        cur, streak = n, 1
        while cur + 1 in num_set:
            cur += 1; streak += 1
        best = max(best, streak)
return best
```

</details>

- [LC 146 LRU 缓存](https://leetcode.cn/problems/lru-cache/)：OrderedDict 或 哈希表 + 双向链表。

<details markdown="1"><summary>▶ 参考解法</summary>

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

</details>

- [LC 380 O(1) 时间插入、删除和获取随机元素](https://leetcode.cn/problems/insert-delete-getrandom-o1/)：dict(val→idx) + list 配合。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
class RandomizedSet:
    def __init__(self):
        self.lst, self.idx = [], {}

    def insert(self, val):
        if val in self.idx: return False
        self.idx[val] = len(self.lst)
        self.lst.append(val)
        return True

    def remove(self, val):
        if val not in self.idx: return False
        i = self.idx[val]
        last = self.lst[-1]
        self.lst[i], self.idx[last] = last, i
        self.lst.pop(); del self.idx[val]
        return True

    def getRandom(self):
        import random
        return random.choice(self.lst)
```

</details>


---

## 4. 单调栈 & 单调队列

### 4.1 单调栈

**适用场景**：求每个元素的下一个/上一个更大/更小元素；柱状图最大矩形；股票价格跨度。

**核心思想**：维护一个单调（递增或递减）的栈，当新元素破坏单调性时弹出并记录结果。

> 💡 **图示**

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 400" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <defs>
    <marker id="ms-arr" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#e65100" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- Title -->
  <text x="260" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">单调栈：下一个更大元素 [2,1,5,3,6]</text>

  <!-- ── 顶部：原数组 ── y=28 -->
  <text x="12" y="44" fill="#666" font-size="11">原数组:</text>
  <rect x="70"  y="28" width="44" height="28" rx="5" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="92"  y="47" text-anchor="middle" fill="#333" font-weight="bold">2</text>
  <text x="92"  y="66" text-anchor="middle" fill="#aaa" font-size="10">[0]</text>
  <rect x="120" y="28" width="44" height="28" rx="5" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="142" y="47" text-anchor="middle" fill="#333" font-weight="bold">1</text>
  <text x="142" y="66" text-anchor="middle" fill="#aaa" font-size="10">[1]</text>
  <rect x="170" y="28" width="44" height="28" rx="5" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="192" y="47" text-anchor="middle" fill="#333" font-weight="bold">5</text>
  <text x="192" y="66" text-anchor="middle" fill="#aaa" font-size="10">[2]</text>
  <rect x="220" y="28" width="44" height="28" rx="5" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="242" y="47" text-anchor="middle" fill="#333" font-weight="bold">3</text>
  <text x="242" y="66" text-anchor="middle" fill="#aaa" font-size="10">[3]</text>
  <rect x="270" y="28" width="44" height="28" rx="5" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="292" y="47" text-anchor="middle" fill="#333" font-weight="bold">6</text>
  <text x="292" y="66" text-anchor="middle" fill="#aaa" font-size="10">[4]</text>

  <!-- 分隔线 -->
  <line x1="12" y1="74" x2="508" y2="74" stroke="#e0e0e0" stroke-width="1"/>

  <!-- ── 步骤① i=0 ── y_base=82 -->
  <rect x="8" y="80" width="120" height="50" rx="6" fill="#f8f8f8" stroke="#e0e0e0" stroke-width="1"/>
  <text x="68" y="98"  text-anchor="middle" fill="#1565c0" font-size="12" font-weight="bold">① i=0, val=2</text>
  <text x="68" y="114" text-anchor="middle" fill="#555" font-size="11">入栈 2</text>
  <!-- 栈内容 -->
  <text x="140" y="95" fill="#888" font-size="10">栈 (底→顶):</text>
  <rect x="140" y="100" width="36" height="24" rx="4" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="158" y="116" text-anchor="middle" fill="#f57f17" font-weight="bold">2</text>
  <text x="180" y="116" fill="#aaa" font-size="10">← 栈顶</text>
  <!-- 结果 -->
  <text x="340" y="112" fill="#9e9e9e" font-size="11">result: [?, ?, ?, ?, ?]</text>

  <!-- 分隔线 -->
  <line x1="12" y1="138" x2="508" y2="138" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- ── 步骤② i=1 ── y_base=146 -->
  <rect x="8" y="144" width="120" height="50" rx="6" fill="#f8f8f8" stroke="#e0e0e0" stroke-width="1"/>
  <text x="68" y="162" text-anchor="middle" fill="#1565c0" font-size="12" font-weight="bold">② i=1, val=1</text>
  <text x="68" y="178" text-anchor="middle" fill="#555" font-size="11">1&lt;2，直接入栈</text>
  <!-- 栈内容 -->
  <text x="140" y="159" fill="#888" font-size="10">栈 (底→顶):</text>
  <rect x="140" y="164" width="36" height="24" rx="4" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="158" y="180" text-anchor="middle" fill="#f57f17" font-weight="bold">2</text>
  <rect x="182" y="164" width="36" height="24" rx="4" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="200" y="180" text-anchor="middle" fill="#f57f17" font-weight="bold">1</text>
  <text x="222" y="180" fill="#aaa" font-size="10">← 栈顶</text>
  <!-- 结果 -->
  <text x="340" y="176" fill="#9e9e9e" font-size="11">result: [?, ?, ?, ?, ?]</text>

  <!-- 分隔线 -->
  <line x1="12" y1="202" x2="508" y2="202" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- ── 步骤③ i=2 ── y_base=210 -->
  <rect x="8" y="208" width="120" height="64" rx="6" fill="#fff3e0" stroke="#ffcc02" stroke-width="1.5"/>
  <text x="68" y="226" text-anchor="middle" fill="#e65100" font-size="12" font-weight="bold">③ i=2, val=5</text>
  <text x="68" y="242" text-anchor="middle" fill="#e65100" font-size="11">5&gt;1 弹出1→res[1]=5</text>
  <text x="68" y="258" text-anchor="middle" fill="#e65100" font-size="11">5&gt;2 弹出2→res[0]=5</text>
  <!-- 被弹出的栈元素（高亮橙色） -->
  <text x="140" y="223" fill="#888" font-size="10">弹出:</text>
  <rect x="140" y="228" width="36" height="24" rx="4" fill="#ffe0b2" stroke="#ff9800" stroke-width="2"/>
  <text x="158" y="244" text-anchor="middle" fill="#e65100" font-weight="bold">1</text>
  <line x1="178" y1="240" x2="218" y2="240" stroke="#e65100" stroke-width="1.5" marker-end="url(#ms-arr)"/>
  <text x="222" y="244" fill="#e65100" font-size="11">res[1]=5</text>
  <rect x="140" y="256" width="36" height="24" rx="4" fill="#ffe0b2" stroke="#ff9800" stroke-width="2"/>
  <text x="158" y="272" text-anchor="middle" fill="#e65100" font-weight="bold">2</text>
  <line x1="178" y1="268" x2="218" y2="268" stroke="#e65100" stroke-width="1.5" marker-end="url(#ms-arr)"/>
  <text x="222" y="272" fill="#e65100" font-size="11">res[0]=5</text>
  <!-- 剩余栈 -->
  <text x="330" y="223" fill="#888" font-size="10">入栈后:</text>
  <rect x="330" y="228" width="36" height="24" rx="4" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="348" y="244" text-anchor="middle" fill="#f57f17" font-weight="bold">5</text>
  <text x="370" y="244" fill="#aaa" font-size="10">← 栈顶</text>

  <!-- 分隔线 -->
  <line x1="12" y1="290" x2="508" y2="290" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- ── 步骤④ i=3,4 ── y_base=298 -->
  <rect x="8" y="296" width="120" height="64" rx="6" fill="#fff3e0" stroke="#ffcc02" stroke-width="1.5"/>
  <text x="68" y="314" text-anchor="middle" fill="#e65100" font-size="11" font-weight="bold">④ i=3 入栈3</text>
  <text x="68" y="330" text-anchor="middle" fill="#e65100" font-size="11">i=4, val=6</text>
  <text x="68" y="346" text-anchor="middle" fill="#e65100" font-size="11">弹3和5</text>
  <!-- 弹出元素 -->
  <text x="140" y="311" fill="#888" font-size="10">弹出:</text>
  <rect x="140" y="316" width="36" height="24" rx="4" fill="#ffe0b2" stroke="#ff9800" stroke-width="2"/>
  <text x="158" y="332" text-anchor="middle" fill="#e65100" font-weight="bold">3</text>
  <line x1="178" y1="328" x2="218" y2="328" stroke="#e65100" stroke-width="1.5" marker-end="url(#ms-arr)"/>
  <text x="222" y="332" fill="#e65100" font-size="11">res[3]=6</text>
  <rect x="140" y="344" width="36" height="24" rx="4" fill="#ffe0b2" stroke="#ff9800" stroke-width="2"/>
  <text x="158" y="360" text-anchor="middle" fill="#e65100" font-weight="bold">5</text>
  <line x1="178" y1="356" x2="218" y2="356" stroke="#e65100" stroke-width="1.5" marker-end="url(#ms-arr)"/>
  <text x="222" y="360" fill="#e65100" font-size="11">res[2]=6</text>
  <!-- 剩余栈 -->
  <text x="330" y="311" fill="#888" font-size="10">入栈后:</text>
  <rect x="330" y="316" width="36" height="24" rx="4" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="348" y="332" text-anchor="middle" fill="#f57f17" font-weight="bold">6</text>
  <text x="370" y="332" fill="#aaa" font-size="10">← 栈顶</text>
  <text x="330" y="356" fill="#9e9e9e" font-size="10">res[4]=-1（无更大元素）</text>

  <!-- 结果汇总 -->
  <rect x="12" y="372" width="496" height="24" rx="6" fill="#e8f5e9" stroke="#a5d6a7" stroke-width="1.5"/>
  <text x="260" y="388" text-anchor="middle" fill="#2e7d32" font-weight="bold" font-size="12">最终结果 result = [5, 5, 6, 6, −1]</text>
</svg>
</div>

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
- [LC 496 下一个更大元素 I](https://leetcode.cn/problems/next-greater-element-i/)：单调递减栈 + 哈希映射。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
stack, mapping = [], {}
for num in nums2:
    while stack and stack[-1] < num:
        mapping[stack.pop()] = num
    stack.append(num)
return [mapping.get(n, -1) for n in nums1]
```

</details>

- [LC 739 每日温度](https://leetcode.cn/problems/daily-temperatures/)：单调递减栈，弹出时记录等待天数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(temperatures)
res = [0] * n
stack = []
for i, t in enumerate(temperatures):
    while stack and temperatures[stack[-1]] < t:
        j = stack.pop()
        res[j] = i - j
    stack.append(i)
return res
```

</details>

- [LC 84 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)：哨兵 + 单调递增栈。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
heights = [0] + heights + [0]
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

</details>

- [LC 85 最大矩形（矩阵）](https://leetcode.cn/problems/maximal-rectangle/)：逐行转化为直方图，套LC 84。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
# 逐行转为直方图，套 LC 84 模板
m, n = len(matrix), len(matrix[0])
heights = [0] * n
res = 0
for row in matrix:
    for j in range(n):
        heights[j] = heights[j] + 1 if row[j] == '1' else 0
    # 柱状图最大矩形
    h = [0] + heights + [0]
    stk = [0]
    for i in range(1, len(h)):
        while h[i] < h[stk[-1]]:
            height = h[stk.pop()]
            width = i - stk[-1] - 1
            res = max(res, height * width)
        stk.append(i)
return res
```

</details>

- [LC 316 去除重复字母](https://leetcode.cn/problems/remove-duplicate-letters/)：单调递增栈 + 剩余计数控制贪心。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
stack, last_occ, visited = [], {}, set()
for i, c in enumerate(s):
    last_occ[c] = i
for i, c in enumerate(s):
    if c not in visited:
        while stack and c < stack[-1] and last_occ[stack[-1]] > i:
            visited.discard(stack.pop())
        stack.append(c)
        visited.add(c)
return ''.join(stack)
```

</details>


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
- [LC 239 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)：单调队列模板题。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
dq, res = deque(), []
for i, num in enumerate(nums):
    while dq and dq[0] < i - k + 1: dq.popleft()
    while dq and nums[dq[-1]] < num: dq.pop()
    dq.append(i)
    if i >= k - 1: res.append(nums[dq[0]])
return res
```

</details>

- [LC 862 和至少为K的最短子数组](https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/)：前缀和 + 单调队列。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
prefix = [0] * (len(A) + 1)
for i, v in enumerate(A): prefix[i+1] = prefix[i] + v
dq = deque()
res = float('inf')
for i, p in enumerate(prefix):
    while dq and p - prefix[dq[0]] >= K:
        res = min(res, i - dq.popleft())
    while dq and prefix[dq[-1]] >= p:
        dq.pop()
    dq.append(i)
return res if res != float('inf') else -1
```

</details>

- [LC 1438 绝对差不超过限制的最长连续子数组](https://leetcode.cn/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)：两个单调队列分别维护最大最小值。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
max_dq, min_dq = deque(), deque()
left = res = 0
for right, num in enumerate(nums):
    while max_dq and nums[max_dq[-1]] <= num: max_dq.pop()
    while min_dq and nums[min_dq[-1]] >= num: min_dq.pop()
    max_dq.append(right); min_dq.append(right)
    while nums[max_dq[0]] - nums[min_dq[0]] > limit:
        left += 1
        if max_dq[0] < left: max_dq.popleft()
        if min_dq[0] < left: min_dq.popleft()
    res = max(res, right - left + 1)
return res
```

</details>


---

## 5. 二分查找

### 5.1 标准二分

**适用场景**：有序数组中查找特定值，O(log n)。

**核心思想**：每次排除一半搜索空间，不变量：目标在 `[left, right]` 内。

> 💡 **图示**

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 215" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <defs>
    <marker id="bs-arrB" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#1565c0" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="bs-arrR" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#c62828" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="bs-arrO" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#e65100" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
  <text x="260" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">二分查找：target = 7</text>
  <!-- mid 标签在上方，先画：y=26 -->
  <text x="233" y="30" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">mid=3</text>
  <line x1="233" y1="33" x2="233" y2="43" stroke="#e65100" stroke-width="2" marker-end="url(#bs-arrO)"/>
  <!-- Array boxes: y=46 h=36 bottom=82 -->
  <rect x="12"  y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="41"  y="69" text-anchor="middle" fill="#333">1</text>
  <rect x="76"  y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="105" y="69" text-anchor="middle" fill="#333">3</text>
  <rect x="140" y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="169" y="69" text-anchor="middle" fill="#333">5</text>
  <rect x="204" y="46" width="58" height="36" rx="6" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2.5"/>
  <text x="233" y="69" text-anchor="middle" fill="#1b5e20" font-weight="bold">7</text>
  <rect x="268" y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="297" y="69" text-anchor="middle" fill="#333">9</text>
  <rect x="332" y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="361" y="69" text-anchor="middle" fill="#333">11</text>
  <rect x="396" y="46" width="58" height="36" rx="6" fill="#f5f5f5" stroke="#bdbdbd" stroke-width="1.5"/>
  <text x="425" y="69" text-anchor="middle" fill="#333">13</text>
  <!-- Index labels: y=96 -->
  <text x="41"  y="96" text-anchor="middle" fill="#aaa" font-size="11">0</text>
  <text x="105" y="96" text-anchor="middle" fill="#aaa" font-size="11">1</text>
  <text x="169" y="96" text-anchor="middle" fill="#aaa" font-size="11">2</text>
  <text x="233" y="96" text-anchor="middle" fill="#aaa" font-size="11">3</text>
  <text x="297" y="96" text-anchor="middle" fill="#aaa" font-size="11">4</text>
  <text x="361" y="96" text-anchor="middle" fill="#aaa" font-size="11">5</text>
  <text x="425" y="96" text-anchor="middle" fill="#aaa" font-size="11">6</text>
  <!-- lo/hi arrows: start y=100 -->
  <line x1="41"  y1="100" x2="41"  y2="110" stroke="#1565c0" stroke-width="2" marker-end="url(#bs-arrB)"/>
  <text x="41"  y="124" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="12">lo=0</text>
  <line x1="425" y1="100" x2="425" y2="110" stroke="#c62828" stroke-width="2" marker-end="url(#bs-arrR)"/>
  <text x="425" y="124" text-anchor="middle" fill="#c62828" font-weight="bold" font-size="12">hi=6</text>
  <!-- 找到提示 -->
  <text x="233" y="138" text-anchor="middle" fill="#2e7d32" font-weight="bold" font-size="13">= target ✓ 找到！</text>
  <!-- Rule boxes: y=148 h=40 -->
  <rect x="12"  y="148" width="240" height="40" rx="6" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="132" y="164" text-anchor="middle" fill="#1565c0" font-size="12">arr[mid] &lt; target</text>
  <text x="132" y="181" text-anchor="middle" fill="#1565c0" font-size="12">→ lo = mid+1（搜右半）</text>
  <rect x="268" y="148" width="240" height="40" rx="6" fill="#fce4ec" stroke="#f48fb1" stroke-width="1.5"/>
  <text x="388" y="164" text-anchor="middle" fill="#c62828" font-size="12">arr[mid] &gt; target</text>
  <text x="388" y="181" text-anchor="middle" fill="#c62828" font-size="12">→ hi = mid−1（搜左半）</text>
</svg>
</div>

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
- [LC 704 二分查找](https://leetcode.cn/problems/binary-search/)：标准模板题。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
l, r = 0, len(nums) - 1
while l <= r:
    mid = (l + r) // 2
    if nums[mid] == target: return mid
    elif nums[mid] < target: l = mid + 1
    else: r = mid - 1
return -1
```

</details>

- [LC 34 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)：左右边界二分。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def left_bound(nums, t):
    l, r = 0, len(nums)
    while l < r:
        mid = (l + r) // 2
        if nums[mid] < t: l = mid + 1
        else: r = mid
    return l

l = left_bound(nums, target)
r = left_bound(nums, target + 1) - 1
if l <= r < len(nums) and nums[l] == target:
    return [l, r]
return [-1, -1]
```

</details>

- [LC 875 爱吃香蕉的珂珂](https://leetcode.cn/problems/koko-eating-bananas/)：在答案空间二分速度。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import math
def feasible(k):
    return sum(math.ceil(p / k) for p in piles) <= h

l, r = 1, max(piles)
while l < r:
    mid = (l + r) // 2
    if feasible(mid): r = mid
    else: l = mid + 1
return l
```

</details>

- [LC 1011 在 D 天内送达包裹的能力](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)：二分最小载重能力。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def feasible(cap):
    days, cur = 1, 0
    for w in weights:
        if cur + w > cap: days += 1; cur = 0
        cur += w
    return days <= D

l, r = max(weights), sum(weights)
while l < r:
    mid = (l + r) // 2
    if feasible(mid): r = mid
    else: l = mid + 1
return l
```

</details>

- [LC 410 分割数组的最大值](https://leetcode.cn/problems/split-array-largest-sum/)：二分答案 + 贪心验证。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def feasible(mid):
    cnt, cur = 1, 0
    for num in nums:
        if cur + num > mid: cnt += 1; cur = 0
        cur += num
    return cnt <= m

l, r = max(nums), sum(nums)
while l < r:
    mid = (l + r) // 2
    if feasible(mid): r = mid
    else: l = mid + 1
return l
```

</details>


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
- [LC 46 全排列](https://leetcode.cn/problems/permutations/)：used数组去重。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
res = []
def bt(path, used):
    if len(path) == len(nums):
        res.append(path[:]); return
    for i, n in enumerate(nums):
        if used[i]: continue
        used[i] = True; path.append(n)
        bt(path, used)
        path.pop(); used[i] = False
bt([], [False]*len(nums))
return res
```

</details>

- [LC 77 组合](https://leetcode.cn/problems/combinations/)：start参数控制不重复选取。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
res = []
def bt(start, path):
    if len(path) == k:
        res.append(path[:]); return
    for i in range(start, n - (k - len(path)) + 2):
        path.append(i)
        bt(i + 1, path)
        path.pop()
bt(1, [])
return res
```

</details>

- [LC 78 子集](https://leetcode.cn/problems/subsets/)：每层都记录结果。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
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

</details>

- [LC 39 组合总和](https://leetcode.cn/problems/combination-sum/)：可重复选，sum>target 时剪枝。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
res = []
def bt(start, path, remain):
    if remain == 0:
        res.append(path[:]); return
    for i in range(start, len(candidates)):
        c = candidates[i]
        if c > remain: break
        path.append(c)
        bt(i, path, remain - c)
        path.pop()
candidates.sort()
bt(0, [], target)
return res
```

</details>

- [LC 51 N皇后](https://leetcode.cn/problems/n-queens/)：三个 set 判断冲突，速度远快于矩阵遍历。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
res = []
cols, diag1, diag2 = set(), set(), set()
def bt(row, board):
    if row == n:
        res.append([''.join(r) for r in board]); return
    for col in range(n):
        if col in cols or (row-col) in diag1 or (row+col) in diag2:
            continue
        cols.add(col); diag1.add(row-col); diag2.add(row+col)
        board[row][col] = 'Q'
        bt(row+1, board)
        board[row][col] = '.'
        cols.discard(col); diag1.discard(row-col); diag2.discard(row+col)
bt(0, [['.']*n for _ in range(n)])
return res
```

</details>


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
- [LC 127 单词接龙](https://leetcode.cn/problems/word-ladder/)：BFS 按层扩散，每次替换一个字符。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
word_set = set(wordList)
if endWord not in word_set: return 0
queue = deque([(beginWord, 1)])
while queue:
    word, steps = queue.popleft()
    for i in range(len(word)):
        for c in 'abcdefghijklmnopqrstuvwxyz':
            nw = word[:i] + c + word[i+1:]
            if nw == endWord: return steps + 1
            if nw in word_set:
                word_set.remove(nw)
                queue.append((nw, steps + 1))
return 0
```

</details>

- [LC 994 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)：多源 BFS，所有腐烂橘子同时扩散。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
m, n = len(grid), len(grid[0])
q = deque()
fresh = 0
for i in range(m):
    for j in range(n):
        if grid[i][j] == 2: q.append((i,j,0))
        elif grid[i][j] == 1: fresh += 1
time = 0
while q:
    r, c, t = q.popleft()
    for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
        nr, nc = r+dr, c+dc
        if 0<=nr<m and 0<=nc<n and grid[nr][nc]==1:
            grid[nr][nc] = 2; fresh -= 1
            time = max(time, t+1)
            q.append((nr,nc,t+1))
return time if fresh == 0 else -1
```

</details>

- [LC 207 课程表](https://leetcode.cn/problems/course-schedule/)：拓扑排序判断有无环。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque, defaultdict
indegree = [0] * numCourses
graph = defaultdict(list)
for a, b in prerequisites:
    graph[b].append(a); indegree[a] += 1
q = deque(i for i in range(numCourses) if indegree[i] == 0)
count = 0
while q:
    node = q.popleft(); count += 1
    for nxt in graph[node]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0: q.append(nxt)
return count == numCourses
```

</details>

- [LC 210 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)：返回拓扑序。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque, defaultdict
indegree = [0] * numCourses
graph = defaultdict(list)
for a, b in prerequisites:
    graph[b].append(a); indegree[a] += 1
q = deque(i for i in range(numCourses) if indegree[i] == 0)
order = []
while q:
    node = q.popleft(); order.append(node)
    for nxt in graph[node]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0: q.append(nxt)
return order if len(order) == numCourses else []
```

</details>

- [LC 1162 地图分析](https://leetcode.cn/problems/as-far-from-land-as-possible/)：多源 BFS 求最远陆地。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
m, n = len(grid), len(grid[0])
q = deque()
for i in range(m):
    for j in range(n):
        if grid[i][j] == 1: q.append((i,j,0))
dist = -1
while q:
    r, c, d = q.popleft()
    for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
        nr, nc = r+dr, c+dc
        if 0<=nr<m and 0<=nc<n and grid[nr][nc]==0:
            grid[nr][nc] = -1; dist = d+1
            q.append((nr,nc,d+1))
return dist
```

</details>


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
- [LC 300 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)：O(n²) DP 或 O(n log n) 贪心+二分。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
# O(n log n)
import bisect
tails = []
for num in nums:
    pos = bisect.bisect_left(tails, num)
    if pos == len(tails): tails.append(num)
    else: tails[pos] = num
return len(tails)
```

</details>

- [LC 72 编辑距离](https://leetcode.cn/problems/edit-distance/)：二维 DP，三种操作转移。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
m, n = len(word1), len(word2)
dp = [[0]*(n+1) for _ in range(m+1)]
for i in range(m+1): dp[i][0] = i
for j in range(n+1): dp[0][j] = j
for i in range(1, m+1):
    for j in range(1, n+1):
        if word1[i-1] == word2[j-1]:
            dp[i][j] = dp[i-1][j-1]
        else:
            dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
return dp[m][n]
```

</details>

- [LC 1143 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)：`dp[i][j]` 以两串各前i/j字符为结尾的LCS长度。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
m, n = len(text1), len(text2)
dp = [[0]*(n+1) for _ in range(m+1)]
for i in range(1, m+1):
    for j in range(1, n+1):
        if text1[i-1] == text2[j-1]:
            dp[i][j] = dp[i-1][j-1] + 1
        else:
            dp[i][j] = max(dp[i-1][j], dp[i][j-1])
return dp[m][n]
```

</details>

- [LC 5 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)：区间DP或中心扩展O(n²)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(s)
res = s[0]
for i in range(n):
    for odd, even in [(i,i), (i,i+1)]:
        l, r = odd, even
        while l >= 0 and r < n and s[l] == s[r]:
            l -= 1; r += 1
        if r - l - 1 > len(res):
            res = s[l+1:r]
return res
```

</details>


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
- [LC 416 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)：0/1背包，背包容量 = sum//2，物品重量=值。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
total = sum(nums)
if total % 2: return False
target = total // 2
dp = [False] * (target + 1)
dp[0] = True
for num in nums:
    for j in range(target, num - 1, -1):
        dp[j] = dp[j] or dp[j - num]
return dp[target]
```

</details>

- [LC 322 零钱兑换](https://leetcode.cn/problems/coin-change/)：完全背包，求凑成 amount 的最少硬币数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
dp = [float('inf')] * (amount + 1)
dp[0] = 0
for coin in coins:
    for i in range(coin, amount + 1):
        dp[i] = min(dp[i], dp[i - coin] + 1)
return dp[amount] if dp[amount] != float('inf') else -1
```

</details>

- [LC 518 零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)：完全背包，求凑成 amount 的方案数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
dp = [0] * (amount + 1)
dp[0] = 1
for coin in coins:
    for i in range(coin, amount + 1):
        dp[i] += dp[i - coin]
return dp[amount]
```

</details>

- [LC 474 一和零](https://leetcode.cn/problems/ones-and-zeroes/)：二维0/1背包（容量是 m 个0和 n 个1）。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
dp = [[0]*(n+1) for _ in range(m+1)]
for s in strs:
    zeros = s.count('0')
    ones = s.count('1')
    for i in range(m, zeros-1, -1):
        for j in range(n, ones-1, -1):
            dp[i][j] = max(dp[i][j], dp[i-zeros][j-ones] + 1)
return dp[m][n]
```

</details>


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
- [LC 312 戳气球](https://leetcode.cn/problems/burst-balloons/)：区间DP，`dp[i][j]` = 戳完 `(i,j)` 内气球的最大硬币数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
nums = [1] + list(nums) + [1]
n = len(nums)
dp = [[0]*n for _ in range(n)]
for length in range(2, n):
    for i in range(0, n-length):
        j = i + length
        for k in range(i+1, j):
            dp[i][j] = max(dp[i][j],
                dp[i][k] + dp[k][j] + nums[i]*nums[k]*nums[j])
return dp[0][n-1]
```

</details>

- [LC 1039 多边形三角剖分的最低得分](https://leetcode.cn/problems/minimum-score-triangulation-of-polygon/)：区间DP求最小权值三角剖分。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(values)
dp = [[0]*n for _ in range(n)]
for length in range(2, n):
    for i in range(n-length):
        j = i + length
        for k in range(i+1, j):
            dp[i][j] = min(dp[i][j] if dp[i][j] else float('inf'),
                dp[i][k] + dp[k][j] + values[i]*values[k]*values[j])
return dp[0][n-1]
```

</details>

- [LC 516 最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence/)：区间DP，`dp[i][j]` 为子串中最长回文子序列长度。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(s)
dp = [[0]*n for _ in range(n)]
for i in range(n): dp[i][i] = 1
for length in range(2, n+1):
    for i in range(n-length+1):
        j = i + length - 1
        if s[i] == s[j]:
            dp[i][j] = dp[i+1][j-1] + 2 if length > 2 else 2
        else:
            dp[i][j] = max(dp[i+1][j], dp[i][j-1])
return dp[0][n-1]
```

</details>


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
- [LC 847 访问所有节点的最短路径](https://leetcode.cn/problems/shortest-path-visiting-all-nodes/)：状压DP + BFS。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque
n = len(graph)
dist = [[float('inf')]*n for _ in range(n)]
for u in range(n):
    dist[u][u] = 0
    for v in graph[u]: dist[u][v] = 1
# Floyd
for k in range(n):
    for i in range(n):
        for j in range(n):
            dist[i][j] = min(dist[i][j], dist[i][k]+dist[k][j])
# BFS 状压
INF = float('inf')
dp = [[INF]*n for _ in range(1<<n)]
q = deque()
for i in range(n):
    dp[1<<i][i] = 0
    q.append((1<<i, i))
while q:
    mask, u = q.popleft()
    for v in range(n):
        new_mask = mask | (1<<v)
        if dp[new_mask][v] > dp[mask][u] + dist[u][v]:
            dp[new_mask][v] = dp[mask][u] + dist[u][v]
            q.append((new_mask, v))
full = (1<<n)-1
return min(dp[full])
```

</details>

- [LC 691 贴纸拼词](https://leetcode.cn/problems/stickers-to-spell-word/)：状压DP，用位掩码表示已覆盖的字符。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
target_letters = list(set(target))
t = len(target_letters)
letter_idx = {c: i for i, c in enumerate(target_letters)}
sticker_masks = []
for s in stickers:
    mask = 0
    for c in s:
        if c in letter_idx:
            mask |= (1 << letter_idx[c])
    sticker_masks.append(mask)
full = (1 << t) - 1
dp = [float('inf')] * (full + 1)
dp[0] = 0
for mask in range(full + 1):
    if dp[mask] == float('inf'): continue
    for sm in sticker_masks:
        new_mask = mask | sm
        dp[new_mask] = min(dp[new_mask], dp[mask] + 1)
return dp[full] if dp[full] != float('inf') else -1
```

</details>

- [LC 1986 完成任务的最少工作时间段](https://leetcode.cn/problems/minimum-number-of-work-sessions-to-finish-the-tasks/)：状压DP枚举子集划分。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
n = len(tasks)
total = (1 << n) - 1
dp = [float('inf')] * (total + 1)
dp[0] = 0
sub_time = [0] * (total + 1)
for mask in range(1, total + 1):
    for i in range(n):
        if mask & (1 << i):
            sub_time[mask] = sub_time[mask ^ (1<<i)] + tasks[i]
for mask in range(total + 1):
    if dp[mask] == float('inf'): continue
    sub = (total ^ mask)
    nxt = sub
    while nxt:
        if sub_time[nxt] <= sessionTime:
            dp[mask | nxt] = min(dp[mask | nxt], dp[mask] + 1)
        nxt = (nxt - 1) & sub
return dp[total]
```

</details>


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

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 520 210" xmlns="http://www.w3.org/2000/svg" style="max-width:520px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:13px">
  <text x="260" y="16" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">贪心区间调度（按结束时间排序）</text>
  <text x="110" y="30" text-anchor="middle" fill="#999" font-size="11">0</text>
  <text x="166" y="30" text-anchor="middle" fill="#999" font-size="11">1</text>
  <text x="222" y="30" text-anchor="middle" fill="#999" font-size="11">2</text>
  <text x="278" y="30" text-anchor="middle" fill="#999" font-size="11">3</text>
  <text x="334" y="30" text-anchor="middle" fill="#999" font-size="11">4</text>
  <text x="390" y="30" text-anchor="middle" fill="#999" font-size="11">5</text>
  <text x="446" y="30" text-anchor="middle" fill="#999" font-size="11">6</text>
  <text x="502" y="30" text-anchor="middle" fill="#999" font-size="11">7</text>
  <line x1="110" y1="34" x2="110" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="166" y1="34" x2="166" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="222" y1="34" x2="222" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="278" y1="34" x2="278" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="334" y1="34" x2="334" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="390" y1="34" x2="390" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="446" y1="34" x2="446" y2="190" stroke="#eee" stroke-width="1"/>
  <line x1="502" y1="34" x2="502" y2="190" stroke="#eee" stroke-width="1"/>
  <text x="8" y="58" fill="#555" font-size="12">[1, 3]</text>
  <rect x="166" y="44" width="112" height="22" rx="4" fill="#1976d2" stroke="#1565c0" stroke-width="1.5"/>
  <text x="222" y="59" text-anchor="middle" fill="white" font-size="12" font-weight="bold">&#x2713; 选中</text>
  <text x="8" y="90" fill="#555" font-size="12">[2, 4]</text>
  <rect x="222" y="76" width="112" height="22" rx="4" fill="#b0bec5" stroke="#90a4ae" stroke-width="1" stroke-dasharray="4,2"/>
  <text x="278" y="91" text-anchor="middle" fill="#607d8b" font-size="12">跳过</text>
  <text x="8" y="122" fill="#555" font-size="12">[3, 5]</text>
  <rect x="278" y="108" width="112" height="22" rx="4" fill="#388e3c" stroke="#2e7d32" stroke-width="1.5"/>
  <text x="334" y="123" text-anchor="middle" fill="white" font-size="12" font-weight="bold">&#x2713; 选中</text>
  <text x="8" y="154" fill="#555" font-size="12">[4, 6]</text>
  <rect x="334" y="140" width="112" height="22" rx="4" fill="#b0bec5" stroke="#90a4ae" stroke-width="1" stroke-dasharray="4,2"/>
  <text x="390" y="155" text-anchor="middle" fill="#607d8b" font-size="12">跳过</text>
  <text x="8" y="186" fill="#555" font-size="12">[1, 6]</text>
  <rect x="166" y="172" width="280" height="22" rx="4" fill="#b0bec5" stroke="#90a4ae" stroke-width="1" stroke-dasharray="4,2"/>
  <text x="306" y="187" text-anchor="middle" fill="#607d8b" font-size="12">跳过（结束时间晚）</text>
  <rect x="390" y="150" width="118" height="36" rx="6" fill="#e8f5e9" stroke="#81c784" stroke-width="1.5"/>
  <text x="449" y="165" text-anchor="middle" fill="#2e7d32" font-size="11" font-weight="bold">贪心选中 2 个</text>
  <text x="449" y="179" text-anchor="middle" fill="#388e3c" font-size="11">最多不重叠区间</text>
</svg>
</div>

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
- [LC 455 分发饼干](https://leetcode.cn/problems/assign-cookies/)：小饼干优先满足小胃口，双指针贪心。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
g.sort(); s.sort()
child = cookie = 0
while child < len(g) and cookie < len(s):
    if s[cookie] >= g[child]: child += 1
    cookie += 1
return child
```

</details>

- [LC 45 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)：每次在当前可达范围内选最远跳。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
jumps = cur_end = farthest = 0
for i in range(len(nums) - 1):
    farthest = max(farthest, i + nums[i])
    if i == cur_end:
        jumps += 1
        cur_end = farthest
return jumps
```

</details>

- [LC 435 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)：按结束时间排序，贪心保留结束早的区间。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
intervals.sort(key=lambda x: x[1])
end = float('-inf')
count = 0
for s, e in intervals:
    if s >= end:
        end = e
    else:
        count += 1
return count
```

</details>

- [LC 763 划分字母区间](https://leetcode.cn/problems/partition-labels/)：记录每字母最远出现位置，贪心确定区间端点。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
last = {c: i for i, c in enumerate(s)}
start = end = 0
res = []
for i, c in enumerate(s):
    end = max(end, last[c])
    if i == end:
        res.append(end - start + 1)
        start = i + 1
return res
```

</details>

- [LC 406 根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/)：按身高降序+k升序排列后依次插入。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
people.sort(key=lambda x: (-x[0], x[1]))
res = []
for p in people:
    res.insert(p[1], p)
return res
```

</details>


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
- [LC 148 排序链表](https://leetcode.cn/problems/sort-list/)：归并排序链表，O(n log n) 空间 O(log n)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def merge(l1, l2):
    dummy = cur = ListNode()
    while l1 and l2:
        if l1.val <= l2.val: cur.next = l1; l1 = l1.next
        else: cur.next = l2; l2 = l2.next
        cur = cur.next
    cur.next = l1 or l2
    return dummy.next

def sort(head):
    if not head or not head.next: return head
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next; fast = fast.next.next
    mid = slow.next; slow.next = None
    return merge(sort(head), sort(mid))

return sort(head)
```

</details>

- [LC 315 计算右侧小于当前元素的个数](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/)：归并排序统计逆序对变体。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
res = [0] * len(nums)
def merge_sort(arr):
    if len(arr) <= 1: return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    merged = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i][0] <= right[j][0]:
            res[left[i][1]] += j
            merged.append(left[i]); i += 1
        else:
            merged.append(right[j]); j += 1
    for x in left[i:]:
        res[x[1]] += j
        merged.append(x)
    merged.extend(right[j:])
    return merged
merge_sort(list(enumerate(nums)))
return res
```

</details>

- [LC 215 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)：快速选择平均 O(n)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import random
def partition(lo, hi):
    pivot = nums[hi]
    p = lo
    for i in range(lo, hi):
        if nums[i] <= pivot:
            nums[p], nums[i] = nums[i], nums[p]; p += 1
    nums[p], nums[hi] = nums[hi], nums[p]
    return p

target = len(nums) - k
lo, hi = 0, len(nums) - 1
while lo < hi:
    p = partition(lo, hi)
    if p == target: break
    elif p < target: lo = p + 1
    else: hi = p - 1
return nums[target]
```

</details>

- [LC 4 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)：分治二分，O(log(m+n))。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def find_kth(A, B, k):
    if not A: return B[k-1]
    if not B: return A[k-1]
    if k == 1: return min(A[0], B[0])
    half = k // 2
    ia = min(half, len(A)); ib = min(half, len(B))
    if A[ia-1] <= B[ib-1]:
        return find_kth(A[ia:], B, k-ia)
    else:
        return find_kth(A, B[ib:], k-ib)

m, n = len(nums1), len(nums2)
total = m + n
if total % 2:
    return find_kth(nums1, nums2, total//2+1)
else:
    return (find_kth(nums1, nums2, total//2) +
            find_kth(nums1, nums2, total//2+1)) / 2
```

</details>


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
- [LC 104 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)：递归 `1 + max(left, right)`。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def depth(node):
    if not node: return 0
    return max(depth(node.left), depth(node.right)) + 1
return depth(root)
```

</details>

- [LC 236 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)：后序DFS，左右同时找到则当前节点即为LCA。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def lca(node):
    if not node or node is p or node is q: return node
    left = lca(node.left)
    right = lca(node.right)
    if left and right: return node
    return left or right
return lca(root)
```

</details>

- [LC 124 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)：树形DP，每个节点维护单侧最大贡献。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
max_sum = float('-inf')
def dfs(node):
    nonlocal max_sum
    if not node: return 0
    left = max(dfs(node.left), 0)
    right = max(dfs(node.right), 0)
    max_sum = max(max_sum, node.val + left + right)
    return node.val + max(left, right)
dfs(root)
return max_sum
```

</details>

- [LC 337 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/)：树形DP，每节点返回选/不选两个状态。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
def dfs(node):
    if not node: return (0, 0)  # (不抢, 抢)
    l_no, l_yes = dfs(node.left)
    r_no, r_yes = dfs(node.right)
    no = max(l_no, l_yes) + max(r_no, r_yes)
    yes = node.val + l_no + r_no
    return (no, yes)
return max(dfs(root))
```

</details>

- [LC 543 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)：`left_depth + right_depth` 的最大值。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
diameter = 0
def depth(node):
    nonlocal diameter
    if not node: return 0
    left = depth(node.left)
    right = depth(node.right)
    diameter = max(diameter, left + right)
    return max(left, right) + 1
depth(root)
return diameter
```

</details>


---

## 12. 图论

### 12.1 并查集

**适用场景**：动态连通性、环检测、最小生成树（Kruskal）。

> 💡 **图示**（并查集合并与路径压缩）

<div style="overflow-x:auto;margin:1rem 0">
<svg viewBox="0 0 580 640" xmlns="http://www.w3.org/2000/svg" style="max-width:580px;width:100%;font-family:'Noto Sans SC',sans-serif;font-size:12px">
  <defs>
    <marker id="uf-arr" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#1565c0" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
    <marker id="uf-arrO" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M1,2 L8,5 L1,8" fill="none" stroke="#e65100" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- ══ 标题 ══ -->
  <text x="290" y="18" text-anchor="middle" font-weight="bold" fill="#2c3e50" font-size="14">并查集：Union 过程 &amp; 路径压缩</text>
  <text x="290" y="30" text-anchor="middle" fill="#888" font-size="10">箭头方向：孩子 → 父亲（parent[child] = parent）</text>

  <!-- 左右分隔线 -->
  <line x1="400" y1="36" x2="400" y2="635" stroke="#e0e0e0" stroke-width="1" stroke-dasharray="4,3"/>

  <!-- 列标题 -->
  <text x="55"  y="44" text-anchor="middle" fill="#888" font-size="10">操作</text>
  <text x="200" y="44" text-anchor="middle" fill="#888" font-size="10">当前树形状态</text>
  <text x="328" y="44" text-anchor="middle" fill="#888" font-size="10">parent 数组</text>
  <line x1="10" y1="48" x2="395" y2="48" stroke="#e0e0e0" stroke-width="1"/>

  <!-- ━━━━ Step 1: union(1,2)  根1(200,75) 子2(200,135) ━━━━ -->
  <rect x="10" y="56" width="90" height="24" rx="5" fill="#e3f2fd" stroke="#90caf9" stroke-width="1"/>
  <text x="55" y="72" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="11">union(1,2)</text>
  <circle cx="200" cy="75" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="200" y="80" text-anchor="middle" fill="#1b5e20" font-weight="bold" font-size="12">1</text>
  <!-- 2→1: from(200,122) to(200,89) -->
  <line x1="200" y1="122" x2="200" y2="90" stroke="#e65100" stroke-width="2" marker-end="url(#uf-arrO)"/>
  <circle cx="200" cy="135" r="13" fill="#fff9c4" stroke="#ffd54f" stroke-width="2"/>
  <text x="200" y="140" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">2</text>
  <text x="310" y="72" fill="#555" font-size="11">parent:</text>
  <text x="310" y="88" fill="#888" font-size="10">[_, 1, 1, 3, 4, 5]</text>
  <text x="310" y="103" fill="#e65100" font-size="10">↑ [2]=1 新增</text>
  <line x1="10" y1="160" x2="395" y2="160" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="3,3"/>

  <!-- ━━━━ Step 2: union(1,3)  根1(200,180) 子2(160,240) 子3(240,240) ━━━━ -->
  <rect x="10" y="168" width="90" height="24" rx="5" fill="#e3f2fd" stroke="#90caf9" stroke-width="1"/>
  <text x="55" y="184" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="11">union(1,3)</text>
  <circle cx="200" cy="188" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="200" y="193" text-anchor="middle" fill="#1b5e20" font-weight="bold" font-size="12">1</text>
  <!-- 2→1: from(164,227) to(188,201) -->
  <line x1="164" y1="227" x2="188" y2="201" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="160" cy="240" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="160" y="245" text-anchor="middle" fill="#333" font-size="12">2</text>
  <!-- 3→1: from(236,227) to(212,201) -->
  <line x1="236" y1="227" x2="212" y2="201" stroke="#e65100" stroke-width="2" marker-end="url(#uf-arrO)"/>
  <circle cx="240" cy="240" r="13" fill="#fff9c4" stroke="#ffd54f" stroke-width="2"/>
  <text x="240" y="245" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">3</text>
  <text x="310" y="185" fill="#555" font-size="11">parent:</text>
  <text x="310" y="201" fill="#888" font-size="10">[_, 1, 1, 1, 4, 5]</text>
  <text x="310" y="216" fill="#e65100" font-size="10">↑ [3]=1 新增</text>
  <line x1="10" y1="268" x2="395" y2="268" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="3,3"/>

  <!-- ━━━━ Step 3: union(3,4)  根1(195,285) 子2(145,345) 子3(245,345) 孙4(245,405) ━━━━ -->
  <rect x="10" y="276" width="90" height="24" rx="5" fill="#e3f2fd" stroke="#90caf9" stroke-width="1"/>
  <text x="55" y="292" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="11">union(3,4)</text>
  <circle cx="195" cy="293" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="195" y="298" text-anchor="middle" fill="#1b5e20" font-weight="bold" font-size="12">1</text>
  <!-- 2→1 -->
  <line x1="149" y1="332" x2="182" y2="306" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="145" cy="345" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="145" y="350" text-anchor="middle" fill="#333" font-size="12">2</text>
  <!-- 3→1 -->
  <line x1="241" y1="332" x2="208" y2="306" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="245" cy="345" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="245" y="350" text-anchor="middle" fill="#333" font-size="12">3</text>
  <!-- 4→3 -->
  <line x1="245" y1="392" x2="245" y2="360" stroke="#e65100" stroke-width="2" marker-end="url(#uf-arrO)"/>
  <circle cx="245" cy="405" r="13" fill="#fff9c4" stroke="#ffd54f" stroke-width="2"/>
  <text x="245" y="410" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">4</text>
  <text x="310" y="290" fill="#555" font-size="11">parent:</text>
  <text x="310" y="306" fill="#888" font-size="10">[_, 1, 1, 1, 3, 5]</text>
  <text x="310" y="321" fill="#e65100" font-size="10">↑ [4]=3 新增</text>
  <text x="310" y="336" fill="#888" font-size="10">find(4)→3→1</text>
  <line x1="10" y1="430" x2="395" y2="430" stroke="#f0f0f0" stroke-width="1" stroke-dasharray="3,3"/>

  <!-- ━━━━ Step 4: union(1,5)  根1(190,448) 子2(115,508) 子3(190,508) 子5(265,508) 孙4(190,568) ━━━━ -->
  <rect x="10" y="438" width="90" height="24" rx="5" fill="#e3f2fd" stroke="#90caf9" stroke-width="1"/>
  <text x="55" y="454" text-anchor="middle" fill="#1565c0" font-weight="bold" font-size="11">union(1,5)</text>
  <circle cx="190" cy="456" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="190" y="461" text-anchor="middle" fill="#1b5e20" font-weight="bold" font-size="12">1</text>
  <!-- 2→1 -->
  <line x1="119" y1="495" x2="178" y2="469" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="115" cy="508" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="115" y="513" text-anchor="middle" fill="#333" font-size="12">2</text>
  <!-- 3→1 -->
  <line x1="190" y1="495" x2="190" y2="470" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="190" cy="508" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="190" y="513" text-anchor="middle" fill="#333" font-size="12">3</text>
  <!-- 5→1 (新，橙色) -->
  <line x1="261" y1="495" x2="202" y2="469" stroke="#e65100" stroke-width="2" marker-end="url(#uf-arrO)"/>
  <circle cx="265" cy="508" r="13" fill="#fff9c4" stroke="#ffd54f" stroke-width="2"/>
  <text x="265" y="513" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">5</text>
  <!-- 4→3 -->
  <line x1="190" y1="555" x2="190" y2="523" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="190" cy="568" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="190" y="573" text-anchor="middle" fill="#333" font-size="12">4</text>
  <text x="310" y="453" fill="#555" font-size="11">parent:</text>
  <text x="310" y="469" fill="#888" font-size="10">[_, 1, 1, 1, 3, 1]</text>
  <text x="310" y="484" fill="#e65100" font-size="10">↑ [5]=1 新增</text>
  <line x1="10" y1="595" x2="395" y2="595" stroke="#e0e0e0" stroke-width="1"/>

  <!-- 图例 -->
  <line x1="18" y1="610" x2="42" y2="610" stroke="#e65100" stroke-width="2" marker-end="url(#uf-arrO)"/>
  <text x="46" y="614" fill="#e65100" font-size="10">本步新增</text>
  <line x1="118" y1="610" x2="142" y2="610" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <text x="146" y="614" fill="#1565c0" font-size="10">已有边</text>
  <circle cx="220" cy="610" r="8" fill="#fff9c4" stroke="#ffd54f" stroke-width="1.5"/>
  <text x="232" y="614" fill="#e65100" font-size="10">新节点</text>

  <!-- ══ 右侧：路径压缩 ══ -->
  <text x="490" y="44" text-anchor="middle" fill="#e65100" font-weight="bold" font-size="12">路径压缩 find(4)</text>
  <line x1="408" y1="48" x2="572" y2="48" stroke="#e0e0e0" stroke-width="1"/>

  <!-- 压缩前 -->
  <text x="490" y="68" text-anchor="middle" fill="#666" font-size="11">压缩前：4→3→1</text>
  <circle cx="490" cy="90" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="490" y="95" text-anchor="middle" fill="#1b5e20" font-weight="bold">1</text>
  <!-- 3→1 -->
  <line x1="490" y1="133" x2="490" y2="104" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="490" cy="147" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="490" y="152" text-anchor="middle" fill="#333">3</text>
  <!-- 4→3 -->
  <line x1="490" y1="190" x2="490" y2="161" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="490" cy="204" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="490" y="209" text-anchor="middle" fill="#333">4</text>
  <text x="490" y="228" text-anchor="middle" fill="#888" font-size="10">深度=2，需走2步</text>

  <text x="490" y="256" text-anchor="middle" fill="#e65100" font-size="22">↓</text>
  <text x="490" y="276" text-anchor="middle" fill="#e65100" font-size="11">路径压缩后</text>

  <!-- 压缩后 -->
  <text x="490" y="298" text-anchor="middle" fill="#666" font-size="11">压缩后：4直连根1</text>
  <circle cx="490" cy="318" r="13" fill="#c8e6c9" stroke="#66bb6a" stroke-width="2"/>
  <text x="490" y="323" text-anchor="middle" fill="#1b5e20" font-weight="bold">1</text>
  <!-- 3→1 -->
  <line x1="460" y1="355" x2="478" y2="332" stroke="#1565c0" stroke-width="1.5" marker-end="url(#uf-arr)"/>
  <circle cx="454" cy="368" r="13" fill="#e3f2fd" stroke="#90caf9" stroke-width="1.5"/>
  <text x="454" y="373" text-anchor="middle" fill="#333">3</text>
  <!-- 4→1 压缩新边 -->
  <line x1="520" y1="355" x2="502" y2="332" stroke="#e65100" stroke-width="2" stroke-dasharray="5,2" marker-end="url(#uf-arrO)"/>
  <circle cx="526" cy="368" r="13" fill="#ffe0b2" stroke="#ff9800" stroke-width="2"/>
  <text x="526" y="373" text-anchor="middle" fill="#e65100" font-weight="bold">4</text>
  <text x="490" y="396" text-anchor="middle" fill="#888" font-size="10">深度=1，只需1步 ✓</text>

  <!-- 右侧图例 -->
  <line x1="412" y1="420" x2="436" y2="420" stroke="#e65100" stroke-width="2" stroke-dasharray="5,2" marker-end="url(#uf-arrO)"/>
  <text x="440" y="424" fill="#e65100" font-size="10">压缩新边</text>
</svg>
</div>

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
- [LC 200 岛屿数量](https://leetcode.cn/problems/number-of-islands/)：DFS/BFS 或并查集均可。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
m, n = len(grid), len(grid[0])
def dfs(i, j):
    if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != '1': return
    grid[i][j] = '0'
    dfs(i+1,j); dfs(i-1,j); dfs(i,j+1); dfs(i,j-1)
count = 0
for i in range(m):
    for j in range(n):
        if grid[i][j] == '1':
            dfs(i, j); count += 1
return count
```

</details>

- [LC 547 省份数量](https://leetcode.cn/problems/number-of-provinces/)：并查集统计连通分量数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
parent = list(range(n))
def find(x):
    if parent[x] != x: parent[x] = find(parent[x])
    return parent[x]
def union(x, y):
    px, py = find(x), find(y)
    if px != py: parent[px] = py
count = n
for i in range(n):
    for j in range(i+1, n):
        if isConnected[i][j]:
            if find(i) != find(j): union(i,j); count -= 1
return count
```

</details>

- [LC 684 冗余连接](https://leetcode.cn/problems/redundant-connection/)：并查集检测环，添加边时若已连通则该边冗余。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
parent = list(range(len(edges) + 1))
def find(x):
    if parent[x] != x: parent[x] = find(parent[x])
    return parent[x]
for u, v in edges:
    pu, pv = find(u), find(v)
    if pu == pv: return [u, v]
    parent[pu] = pv
return []
```

</details>

- [LC 1584 连接所有点的最小费用](https://leetcode.cn/problems/min-cost-to-connect-all-points/)：Kruskal 最小生成树。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import heapq
n = len(points)
dist = [float('inf')] * n
dist[0] = 0
visited = [False] * n
heap = [(0, 0)]
total = 0
while heap:
    d, u = heapq.heappop(heap)
    if visited[u]: continue
    visited[u] = True; total += d
    for v in range(n):
        if not visited[v]:
            w = abs(points[u][0]-points[v][0]) + abs(points[u][1]-points[v][1])
            if w < dist[v]:
                dist[v] = w; heapq.heappush(heap, (w, v))
return total
```

</details>


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
- [LC 743 网络延迟时间](https://leetcode.cn/problems/network-delay-time/)：Dijkstra 单源最短路，返回 max(dist)。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import heapq
from collections import defaultdict
graph = defaultdict(list)
for u, v, w in times:
    graph[u].append((v, w))
dist = {k: float('inf') for k in range(1, n+1)}
dist[k] = 0
heap = [(0, k)]
while heap:
    d, u = heapq.heappop(heap)
    if d > dist[u]: continue
    for v, w in graph[u]:
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w; heapq.heappush(heap, (dist[v], v))
return max(dist.values()) if max(dist.values()) < float('inf') else -1
```

</details>

- [LC 1631 最小体力消耗路径](https://leetcode.cn/problems/path-with-minimum-effort/)：Dijkstra（最小化路径上最大差值）。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import heapq
m, n = len(heights), len(heights[0])
dist = [[float('inf')]*n for _ in range(m)]
dist[0][0] = 0
heap = [(0, 0, 0)]
while heap:
    d, r, c = heapq.heappop(heap)
    if d > dist[r][c]: continue
    for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
        nr, nc = r+dr, c+dc
        if 0<=nr<m and 0<=nc<n:
            nd = max(d, abs(heights[nr][nc]-heights[r][c]))
            if nd < dist[nr][nc]:
                dist[nr][nc] = nd; heapq.heappush(heap, (nd,nr,nc))
return dist[m-1][n-1]
```

</details>

- [LC 787 K 站中转内最便宜的航班](https://leetcode.cn/problems/cheapest-flights-within-k-stops/)：Bellman-Ford（限制边数）或 Dijkstra+层数。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
import heapq
from collections import defaultdict
graph = defaultdict(list)
for u, v, w in flights:
    graph[u].append((v, w))
# Dijkstra + 状态 (cost, node, stops)
dist = defaultdict(lambda: (float('inf'), float('inf')))
dist[src] = (0, 0)
heap = [(0, src, 0)]
while heap:
    cost, u, stops = heapq.heappop(heap)
    if u == dst: return cost
    if stops > k: continue
    for v, w in graph[u]:
        nc = cost + w
        if (nc, stops+1) < dist[v]:
            dist[v] = (nc, stops+1)
            heapq.heappush(heap, (nc, v, stops+1))
return -1
```

</details>


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
- [LC 207 课程表](https://leetcode.cn/problems/course-schedule/)：Kahn 算法检测有无环。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque, defaultdict
indegree = [0] * numCourses
graph = defaultdict(list)
for a, b in prerequisites:
    graph[b].append(a); indegree[a] += 1
q = deque(i for i in range(numCourses) if indegree[i] == 0)
count = 0
while q:
    node = q.popleft(); count += 1
    for nxt in graph[node]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0: q.append(nxt)
return count == numCourses
```

</details>

- [LC 210 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)：返回拓扑序。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import deque, defaultdict
indegree = [0] * numCourses
graph = defaultdict(list)
for a, b in prerequisites:
    graph[b].append(a); indegree[a] += 1
q = deque(i for i in range(numCourses) if indegree[i] == 0)
order = []
while q:
    node = q.popleft(); order.append(node)
    for nxt in graph[node]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0: q.append(nxt)
return order if len(order) == numCourses else []
```

</details>

- [LC 269 火星词典](https://leetcode.cn/problems/alien-dictionary/)：从字典构建偏序关系，拓扑排序。

<details markdown="1"><summary>▶ 参考解法</summary>

```python
from collections import defaultdict, deque
graph = defaultdict(set)
indegree = {c: 0 for word in words for c in word}
for i in range(len(words)-1):
    w1, w2 = words[i], words[i+1]
    for c1, c2 in zip(w1, w2):
        if c1 != c2:
            if c2 not in graph[c1]:
                graph[c1].add(c2); indegree[c2] += 1
            break
    else:
        if len(w1) > len(w2): return ""
q = deque(c for c in indegree if indegree[c] == 0)
res = []
while q:
    c = q.popleft(); res.append(c)
    for nxt in graph[c]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0: q.append(nxt)
return ''.join(res) if len(res) == len(indegree) else ""
```

</details>


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

