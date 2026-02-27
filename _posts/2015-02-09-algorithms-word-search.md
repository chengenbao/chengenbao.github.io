---
layout: note
title: "Word Search"
categories: Algorithm
tags: [DFS, Backtracking, Matrix]
---

## Problem

> **LeetCode 79 · Word Search**  
> Given an `m × n` character grid `board` and a string `word`, return `true` if `word` exists in the grid.  
> The word must be constructed from letters of **sequentially adjacent cells** (horizontally or vertically neighboring), and the **same cell may not be used more than once**.

**Example:**

```
board =
  A B C E
  S F C S
  A D E E

"ABCCED" → true   (A→B→C→C→E→D)
"SEE"    → true   (S→E→E)
"ABCB"   → false  (B cannot be reused)
```

---

## Analysis

Standard **DFS + Backtracking** on a 2-D grid.

| Step | Action |
|------|--------|
| 1 | Iterate every cell as a potential starting point |
| 2 | If `board[r][c] == word[0]`, launch DFS |
| 3 | DFS: mark cell visited → recurse on 4 neighbours → unmark (backtrack) |
| 4 | Return `true` as soon as the full word is matched |

**Complexity:** Time O(m·n·4^L), Space O(L) — where L = `len(word)`.

**Optimization:** Check character frequency up front; if the board lacks enough of any character in `word`, short-circuit immediately.

---

## Solution (Python)

```python
class Solution:
    def exist(self, board: list[list[str]], word: str) -> bool:
        from collections import Counter

        # Early exit: board must contain at least as many of each char as word needs
        board_count = Counter(c for row in board for c in row)
        for c, need in Counter(word).items():
            if board_count[c] < need:
                return False

        rows, cols = len(board), len(board[0])
        DIRS = ((0, 1), (0, -1), (1, 0), (-1, 0))

        def dfs(r: int, c: int, idx: int) -> bool:
            if idx == len(word):          # all characters matched
                return True
            if not (0 <= r < rows and 0 <= c < cols):
                return False
            if board[r][c] != word[idx]:  # character mismatch
                return False

            tmp, board[r][c] = board[r][c], "#"   # mark as visited
            found = any(dfs(r + dr, c + dc, idx + 1) for dr, dc in DIRS)
            board[r][c] = tmp                       # backtrack

            return found

        return any(
            dfs(r, c, 0)
            for r in range(rows)
            for c in range(cols)
        )
```

---

## Solution (C++)

```cpp
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        int m = board.size(), n = board[0].size();

        // Frequency check
        int boardFreq[128]{}, wordFreq[128]{};
        for (auto& row : board)
            for (char ch : row) ++boardFreq[(int)ch];
        for (char ch : word) ++wordFreq[(int)ch];
        for (int i = 0; i < 128; ++i)
            if (wordFreq[i] > boardFreq[i]) return false;

        int DIRS[4][2];
        DIRS[0][0]=0;  DIRS[0][1]=1;
        DIRS[1][0]=0;  DIRS[1][1]=-1;
        DIRS[2][0]=1;  DIRS[2][1]=0;
        DIRS[3][0]=-1; DIRS[3][1]=0;

        function<bool(int,int,int)> dfs = [&](int r, int c, int idx) -> bool {
            if (idx == (int)word.size()) return true;
            if (r < 0 || r >= m || c < 0 || c >= n) return false;
            if (board[r][c] != word[idx]) return false;

            char tmp = board[r][c];
            board[r][c] = '#';  // mark visited
            for (auto& d : DIRS)
                if (dfs(r + d[0], c + d[1], idx + 1)) {
                    board[r][c] = tmp;
                    return true;
                }
            board[r][c] = tmp;  // backtrack
            return false;
        };

        for (int r = 0; r < m; ++r)
            for (int c = 0; c < n; ++c)
                if (dfs(r, c, 0)) return true;

        return false;
    }
};
```

---

## Why the Original Solution Had Issues

The 2015 iterative version simulated a stack manually with a `direction` counter per frame.  
The bug: when a cell matched but **all 4 neighbours were exhausted or invalid**, it popped the stack but incremented the *parent's* direction counter — effectively skipping valid sibling paths that hadn't been tried yet from the grandparent.  
Recursive backtracking avoids this entirely: the call stack itself handles the frame management correctly.
