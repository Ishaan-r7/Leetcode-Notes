# Backtracking — Python Revision Notes

Core idea: explore a decision tree of choices. At each step, make a choice, recurse, then undo the choice (backtrack) before trying the next option. The "undo" step is what makes this different from plain DFS — you're reusing the same `path` list across all branches instead of creating a new copy each time.

```python
def backtrack(state, path):
    if <base case>:
        res.append(path[:])     # COPY the path -- path keeps mutating after this
        return
    for choice in <options at this state>:
        path.append(choice)                 # make the choice
        backtrack(<next state>, path)        # explore
        path.pop()                            # undo -- try the next option
```

```text
"Include or exclude each element"      -> subset-style (binary choice per element)
"Build sequences using each index once"-> permutation-style (track used[] or swap)
"Pick combos that sum to a target"      -> combination-style (loop + skip used-up options)
"Has duplicates, want unique results"   -> SORT first, then skip adjacent duplicates
"2D grid exploration"                    -> DFS + mark visited + backtrack (unmark) on the grid itself
```

## How to reason about backtracking complexity (don't just memorize)

```text
General shape: O(branches^depth * work_per_leaf)
    branches = how many choices at each step
    depth    = how many decisions get made before hitting a base case
    work_per_leaf = cost to process/copy a valid result (e.g. path[:] copy is O(depth))

Subsets (n elements, include/exclude each) -> 2 choices x n depth -> O(2^n) total subsets,
    times O(n) to copy each one -> O(n * 2^n)

Permutations (n elements, all orderings)    -> n! total orderings,
    times O(n) to copy each -> O(n * n!)

Combination Sum (unbounded reuse, target T)  -> branches depend on remaining target / smallest value;
    bounded roughly by O(2^T) in the worst case (very loose bound, actual runtime
    usually far smaller once pruning -- i.e. "stop early if sum > target" -- kicks in)

Word Search (grid, word length L)             -> O(rows * cols * 4^L) -- 4 directions per step,
    L deep, times every possible starting cell

KEY EDGE CASE TO WATCH: pruning changes the PRACTICAL runtime a lot even when
the worst-case Big-O looks the same. "if remaining < 0: return" (Combination Sum)
or "if index >= len(s): return" cuts off whole branches early -- always add these
checks, since the theoretical bound is rarely what actually runs.
```

## Subsets (no duplicates)

```python
def subsets(self, nums):
    res = []
    path = []
    def backtrack(i):
        if i == len(nums):
            res.append(path[:])
            return
        path.append(nums[i])          # choice: INCLUDE nums[i]
        backtrack(i + 1)
        path.pop()                     # undo
        backtrack(i + 1)                # choice: EXCLUDE nums[i] (no append needed)
    backtrack(0)
    return res
```

Recall: two recursive calls per index = binary include/exclude tree. `path[:]` is a REQUIRED copy — without it, every entry in `res` would point to the same mutating list and end up empty at the end.

```text
Time:  O(n * 2^n)   Space: O(n) recursion depth (excluding output storage)
```

## Combination Sum (reuse allowed, no duplicates in input)

```python
def combinationSum(self, candidates, target):
    res = []
    path = []
    def backtrack(i, remain):
        if remain == 0:
            res.append(path[:])
            return
        if i >= len(candidates) or remain < 0:
            return
        path.append(candidates[i])
        backtrack(i, remain - candidates[i])   # stay at i -- can REUSE this number
        path.pop()
        backtrack(i + 1, remain)                 # move on -- skip this number entirely
    backtrack(0, target)
    return res
```

Recall: `backtrack(i, ...)` (not `i+1`) in the "use it" branch is what allows reuse of the same number multiple times. `remain < 0` is the pruning check that keeps this fast in practice.

## Permutations

```python
def permute(self, nums):
    res = []
    if len(nums) == 1:
        return [nums[:]]
    for i in range(len(nums)):
        n = nums.pop(i)
        perms = self.permute(nums)
        for p in perms:
            p.append(n)
        res.extend(perms)
        nums.insert(i, n)        # undo
    return res
```

Recall: pull each element out, recursively permute the rest, append the pulled element to every resulting sub-permutation, then put it back (undo) before trying the next one.

```text
Time:  O(n * n!)   Space: O(n) recursion depth
```

## Subsets II (input HAS duplicates, want unique subsets)

```python
def subsetsWithDup(self, nums):
    nums.sort()                     # REQUIRED -- dedup relies on duplicates being adjacent
    res = []
    path = []
    def backtrack(i):
        res.append(path[:])
        for j in range(i, len(nums)):
            if j > i and nums[j] == nums[j - 1]:
                continue               # skip duplicate at this SAME recursion depth
            path.append(nums[j])
            backtrack(j + 1)
            path.pop()
    backtrack(0)
    return res
```

Recall: `j > i` means "not the first choice at this level" — only skip a duplicate if it's not the very first option being tried at this depth (the first occurrence always gets explored; later identical ones at the same level are skipped to avoid duplicate subsets).

## Combination Sum II (dup input, each number used once)

```python
def combinationSum2(self, candidates, target):
    candidates.sort()
    res = []
    path = []
    def backtrack(i, remain):
        if remain == 0:
            res.append(path[:])
            return
        if remain < 0:
            return
        for j in range(i, len(candidates)):
            if j > i and candidates[j] == candidates[j - 1]:
                continue              # same dedup trick as Subsets II
            path.append(candidates[j])
            backtrack(j + 1, remain - candidates[j])   # j+1 -- no reuse this time
            path.pop()
    backtrack(0, target)
    return res
```

Recall: combines Combination Sum's pruning (`remain < 0`) with Subsets II's dedup (`j > i` skip) — and uses `j + 1` instead of `j` since each number can only be used once here.

## Word Search

```python
def exist(self, board, word):
    rows, cols = len(board), len(board[0])
    path = set()

    def dfs(r, c, i):
        if i == len(word):
            return True
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or word[i] != board[r][c] or (r, c) in path):
            return False
        path.add((r, c))
        res = (dfs(r+1, c, i+1) or dfs(r-1, c, i+1) or
               dfs(r, c+1, i+1) or dfs(r, c-1, i+1))
        path.remove((r, c))      # undo -- this cell can be reused on a DIFFERENT path attempt
        return res

    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True
    return False
```

Recall: `path` set tracks cells used in the CURRENT attempted path only — must be removed (`path.remove`) on backtrack, since a cell could still be valid for a different route. All 4 boundary/mismatch/revisit checks live in ONE `if` for brevity, with Python's short-circuit `or` skipping the rest once a letter matches.

```text
Time:  O(rows * cols * 4^L) where L = len(word)   Space: O(L) recursion depth + path set
```

## Palindrome Partitioning

```python
def partition(self, s):
    res = []
    path = []
    def isPali(sub):
        return sub == sub[::-1]

    def backtrack(i):
        if i == len(s):
            res.append(path[:])
            return
        for j in range(i, len(s)):
            if isPali(s[i:j+1]):
                path.append(s[i:j+1])
                backtrack(j + 1)
                path.pop()
    backtrack(0)
    return res
```

Recall: try every possible "next cut" (`j`), only recurse into it if the piece `s[i:j+1]` is itself a palindrome — this is pruning built directly into the branch condition.

## Letter Combinations of a Phone Number

```python
def letterCombinations(self, digits):
    if not digits:
        return []
    digitToChar = {"2":"abc","3":"def","4":"ghi","5":"jkl",
                    "6":"mno","7":"pqrs","8":"tuv","9":"wxyz"}
    res = []
    path = []
    def backtrack(i):
        if i == len(digits):
            res.append("".join(path))
            return
        for c in digitToChar[digits[i]]:
            path.append(c)
            backtrack(i + 1)
            path.pop()
    backtrack(0)
    return res
```

Recall: same subset-style template, just with a different "choices available at this step" source (the letters mapped to the current digit, instead of binary include/exclude).

## N-Queens

```python
def solveNQueens(self, n):
    col = set()
    posDiag = set()    # r + c is constant along a /  diagonal
    negDiag = set()    # r - c is constant along a \  diagonal
    res = []
    board = [["."] * n for _ in range(n)]

    def backtrack(r):
        if r == n:
            res.append(["".join(row) for row in board])
            return
        for c in range(n):
            if c in col or (r + c) in posDiag or (r - c) in negDiag:
                continue
            col.add(c); posDiag.add(r + c); negDiag.add(r - c)
            board[r][c] = "Q"
            backtrack(r + 1)
            col.remove(c); posDiag.remove(r + c); negDiag.remove(r - c)
            board[r][c] = "."
    backtrack(0)
    return res
```

Recall: `r + c` is constant along one diagonal direction, `r - c` constant along the other — this is what lets you check "is this diagonal already occupied" in O(1) instead of scanning the whole board. One row placed per recursion level, so depth = n automatically caps placement at one queen per row.

## One-Line Pattern Recall

```text
Subsets                 -> include/exclude binary choice per index
Combination Sum          -> stay at i to allow reuse, i+1 to move on; prune remain<0
Permutations             -> pop element out, recurse on rest, append back, undo
Subsets II / CombSum II  -> SORT first, skip j>i duplicates at the same recursion depth
Word Search               -> DFS + visited set, remove on backtrack (reusable in other paths)
Palindrome Partitioning   -> try every cut, only recurse if that piece is a palindrome
Letter Combinations        -> subset-style template, choices = digit's mapped letters
N-Queens                  -> r+c / r-c encode diagonals in O(1), one queen placed per row
```
