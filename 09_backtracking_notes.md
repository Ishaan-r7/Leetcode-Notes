# Backtracking — Python Revision Notes

Core idea: explore a decision tree of choices. At each step, **make a choice, recurse, then undo the choice** (backtrack) before trying the next option. The "undo" is what separates this from plain DFS: one shared `path` is reused across all branches instead of building a new one each time.

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
"Build sequences using each index once"-> permutation-style (track used[] or `in path`)
"Pick combos that sum to a target"      -> combination-style (loop + skip used-up options)
"Has duplicates, want unique results"   -> SORT first, then skip adjacent duplicates
"2D grid exploration"                    -> DFS + mark visited + backtrack (unmark) on the grid itself
```

---

# Concepts that are easy to forget

## 1. Two styles: shared object + undo, OR pass a new value (no undo)
```python
# Style 1: ONE shared mutable list. Must undo.
path = []
def dfs(i):
    if i == len(digits):
        res.append("".join(path))
        return
    for c in options[i]:
        path.append(c)       # mutate shared list
        dfs(i + 1)
        path.pop()           # undo, because the next iteration reuses the same list

# Style 2: pass a NEW value into each call. Nothing shared, nothing to undo.
def dfs(i, cur):
    if i == len(digits):
        res.append(cur)
        return
    for c in options[i]:
        dfs(i + 1, cur + c)  # cur + c is a brand-new string; parent's cur is untouched
```
```text
pop() exists ONLY to undo an append on a SHARED mutable object.
Pass a new value (str +, int +, tuple +) -> the "undo" happens for free, each call keeps its own copy.
Pick ONE style per function. Mixing them is the classic bug:
    path.append(c); dfs(i+1, path + [c])   -> adds c twice
    dfs(i+1, path + [c]); path.pop()       -> pops something never appended
```
Same function can use both: in Combination Sum, `sums + nums[i]` (int, passed by value, no undo) and `subset.append` (list, shared, needs pop).

## 2. When do I need `.copy()` / `[:]`?
```text
Saving a snapshot of a MUTABLE object (list/set/dict) that will keep changing -> copy it.
    res.append(path[:])   or   path.copy()   or   list(path)      (all equal, shallow copy)
Immutable things (str, int, tuple) -> never copy. Nothing can mutate them later.
Without the copy, every entry in res points at ONE list, which ends up empty ([[], [], []...]).
```

## 3. What is shared and what is independent between calls?
```text
i, sums, remain, cur  -> parameters. Every call gets its OWN copy. Independent.
path / subset / used  -> ONE shared object. Looks independent between sibling branches ONLY
                          because every append is matched by a pop before the next sibling runs.
                          Forget one pop and state leaks between branches.
```
Reading a trace:
```text
- Each dfs(...) call is its own frame. Calling it PAUSES the parent at that exact line.
- When the child returns, the parent RESUMES on the line right after the call.
- Two sibling calls like dfs(i+1) [include] and dfs(i+1) [exclude] get the same number only because
  they sit under the SAME parent doing the same +1. The 2nd does not resume the 1st, it is a fresh call.
- i=0 only happens once: the very first call from outside.
```

## 4. Parameter vs return value vs shared variable
```text
Parameter   -> state flowing DOWN one path (depth, running max, running sum, current string)
Return      -> totals flowing UP from children (size, area, height)
Shared var  -> ONE answer for the whole traversal (self.res / nonlocal / outer list)
```
A parameter can never carry a result back up (`count += 1` in a child is lost when it returns).

## 5. What should the base case return? (depends on what the PARENT does with it)
```text
Parent ignores the result (dfs only marks cells / fills a list)   -> bare `return`
Parent adds or compares numbers (1 + dfs() + dfs(), max(...))     -> return 0  (None + int crashes)
Parent branches on the result (if / or / and)                     -> return True / False

`or` chains between recursive calls: ONLY for "stop at first success" searches (Word Search).
If you need ALL branches to run (counting, flood fill, collecting), write separate plain calls.
A bare-return dfs inside an `or` chain "works" only because None is falsy. Don't rely on it.
```

## 6. Complexity: how to reason (don't just memorize)
```text
General shape: O(branches^depth * work_per_leaf)
    branches = choices at each step, depth = decisions before a base case,
    work_per_leaf = cost to copy/process a result (path[:] is O(depth))

Subsets (include/exclude each of n)  -> 2^n subsets x O(n) copy      -> O(n * 2^n)
Permutations (all orderings)         -> n! x O(n) copy               -> O(n * n!)
Combination Sum (target T)           -> loose O(2^T), pruning makes it far smaller in practice
Word Search (word length L)          -> O(rows * cols * 4^L)
Grid flood fill (islands)            -> O(rows * cols): each cell is visited once, then marked

PRUNING: "if remain < 0: return", "if i >= len: return" cut whole branches. The worst-case Big-O
rarely reflects what actually runs, so always add the checks.
```

---

# Problems

## Subsets (no duplicates)
```python
def subsets(self, nums):
    res, path = [], []
    def backtrack(i):
        if i == len(nums):
            res.append(path[:])
            return
        path.append(nums[i])          # INCLUDE nums[i]
        backtrack(i + 1)
        path.pop()                     # undo
        backtrack(i + 1)                # EXCLUDE nums[i]
    backtrack(0)
    return res
```
Recall: every call forks into exactly two: include (append happened just before) and exclude (pop just undid it). The `path[:]` copy is required.
```text
Time:  O(n * 2^n)   Space: O(n) recursion depth (excluding output)
```

## Combination Sum (reuse allowed, no duplicates in input)
```python
def combinationSum(self, candidates, target):
    res, path = [], []
    def backtrack(i, remain):
        if remain == 0:
            res.append(path[:])
            return                              # record AND return
        if i >= len(candidates) or remain < 0:
            return
        path.append(candidates[i])
        backtrack(i, remain - candidates[i])    # include, stay at i -> can REUSE this number
        path.pop()
        backtrack(i + 1, remain)                 # exclude: remain UNCHANGED
    backtrack(0, target)
    return res
```
Recall: `i` (not `i+1`) in the include branch is what allows reuse.
Bugs we hit:
```text
1. Exclude branch passed remain - candidates[i]. Whatever you append/pop on the path must be
   mirrored EXACTLY in the running total, or the two drift apart.
2. Forgot `return` after recording a match. The function falls through, keeps exploring, and
   dfs(i+1, same sums) records the same combo AGAIN (duplicates).
   Safe to return because all candidates are positive: going deeper only overshoots.
```
```text
Time:  loose O(2^T), far smaller with pruning.   Space: O(T / min(candidates)) recursion depth
```

## Permutations
```python
def permute(self, nums):
    res, perms = [], []
    def backtrack():
        if len(perms) == len(nums):
            res.append(perms.copy())
            return
        for i in range(len(nums)):
            if nums[i] not in perms:          # perms doubles as the "already used" check
                perms.append(nums[i])
                backtrack()
                perms.pop()
    backtrack()
    return res
```
Recall: no separate `used[]` needed. **Only valid with distinct values**, because `in` compares values, not positions. For duplicates use Permutations II.
```text
Time:  O(n * n!)   Space: O(n)
`not in perms` is an O(n) scan per check (vs O(1) for used[]): same Big-O, heavier constant.
```

## Permutations II (duplicates in input — extra, not in the 150)
```python
def permuteUnique(self, nums):
    nums.sort()                      # REQUIRED: duplicates must be adjacent
    res, perms = [], []
    used = [False] * len(nums)
    def backtrack():
        if len(perms) == len(nums):
            res.append(perms.copy())
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            if i > 0 and nums[i] == nums[i-1] and not used[i-1]:
                continue              # skip a duplicate unless its twin is in use higher up this branch
            used[i] = True
            perms.append(nums[i])
            backtrack()
            perms.pop()
            used[i] = False
    backtrack()
    return res
```
Recall: track by INDEX (`used[]`), not by value. Switching `in perms` to `used[]` fixes wrongly rejecting a repeated value; the sort + `not used[i-1]` skip fixes duplicate OUTPUT permutations. Same family as the Subsets II skip.
```text
Time:  O(n * n!) worst case   Space: O(n)
```

## Subsets II (duplicates in input, want unique subsets)
```python
def subsetsWithDup(self, nums):
    nums.sort()                     # REQUIRED: dedup relies on duplicates being adjacent
    res, path = [], []
    def backtrack(i):
        res.append(path[:])
        for j in range(i, len(nums)):
            if j > i and nums[j] == nums[j - 1]:
                continue               # skip a duplicate at this SAME depth
            path.append(nums[j])
            backtrack(j + 1)
            path.pop()
    backtrack(0)
    return res
```
Recall: `j > i` = "not the first choice at this level". The first occurrence is always explored; later identical ones at the same level are skipped.
```text
Time:  O(n * 2^n)   Space: O(n)
```

## Combination Sum II (duplicates in input, each number used once)
Loop style (same dedup trick as Subsets II):
```python
def combinationSum2(self, candidates, target):
    candidates.sort()
    res, path = [], []
    def backtrack(i, remain):
        if remain == 0:
            res.append(path[:])
            return
        if remain < 0:
            return
        for j in range(i, len(candidates)):
            if j > i and candidates[j] == candidates[j - 1]:
                continue
            path.append(candidates[j])
            backtrack(j + 1, remain - candidates[j])   # j+1: no reuse
            path.pop()
    backtrack(0, target)
    return res
```
Include/exclude style (works too):
```python
def combinationSum2(self, candidates, target):
    candidates.sort()                       # NOT `nums = candidates.sort()` (that is None)
    res, subset = [], []
    def dfs(i, sums):
        if sums == target:                   # check SUCCESS first
            res.append(subset.copy())
            return
        if sums > target or i >= len(candidates):
            return
        subset.append(candidates[i])
        dfs(i + 1, sums + candidates[i])     # include, move forward
        subset.pop()
        while i + 1 < len(candidates) and candidates[i] == candidates[i + 1]:
            i += 1                            # skip ALL copies of this value
        dfs(i + 1, sums)                      # exclude
    dfs(0, 0)
    return res
```
Bugs we hit:
```text
1. `nums = candidates.sort()` -> .sort() sorts in place and returns None.
2. Base-case ORDER: when the include branch moves to i+1, a valid combo can finish exactly as
   i == len. Check `sums == target` BEFORE `i >= len`, or you drop answers (e.g. [2,3] -> 5).
   (Combination Sum I never hit this because its include branch stays at i.)
3. Skip loop bound must be `i + 1 < len(candidates)`, not `<=`, or nums[i+1] is out of range.
4. Name mismatch (subsets vs subset) -> NameError. Pick one name.
```
```text
Time:  O(2^n) worst case, far less with pruning   Space: O(n)
```

## Word Search
```python
def exist(self, board, word):
    m, n = len(board), len(board[0])
    path = set()
    def dfs(r, c, i):
        if i == len(word):                       # 1. SUCCESS first (before anything reads word[i])
            return True
        if (r < 0 or c < 0 or r >= m or c >= n   # r vs m (rows), c vs n (cols)
                or word[i] != board[r][c] or (r, c) in path):
            return False                          # 2. failure returns False
        path.add((r, c))
        res = (dfs(r+1, c, i+1) or dfs(r-1, c, i+1) or
               dfs(r, c+1, i+1) or dfs(r, c-1, i+1))
        path.remove((r, c))                       # ALWAYS runs, win or lose
        return res
    for r in range(m):
        for c in range(n):
            if dfs(r, c, 0):                      # 3. USE the result
                return True
    return False                                  # only after EVERY start cell failed
```
Recall: `path.remove` removes ONE cell, the one this call added. A failed attempt unwinds layer by layer, so `path` is empty again before the outer loop moves to the next start cell. No manual reset needed.
Bugs we hit:
```text
- No success base case -> word[t] IndexError when t == len(word)
- Ignoring dfs's return in the outer loop (so it always returned False)
- `else: return False` inside the outer loop kills the search after the FIRST failed start cell
- Bounds: `r >= n` instead of `c >= n`
```
```text
Time:  O(rows * cols * 4^L), L = len(word)   Space: O(L) recursion + path set
```

## Palindrome Partitioning
```python
def partition(self, s):
    res, path = [], []
    def backtrack(i):
        if i == len(s):
            res.append(path[:])
            return
        for j in range(i, len(s)):
            piece = s[i:j+1]
            if piece == piece[::-1]:              # only cut here if the piece is a palindrome
                path.append(piece)
                backtrack(j + 1)
                path.pop()
    backtrack(0)
    return res
```
Recall: try every "next cut" j, and recurse only when the piece is a palindrome (pruning in the branch condition).
```text
Time:  O(n * 2^n) worst case (isPali is O(n))   Space: O(n)
```

## Letter Combinations of a Phone Number
```python
def letterCombinations(self, digits):
    if not digits:
        return []
    digitToChar = {"2": "abc", "3": "def", "4": "ghi", "5": "jkl",
                   "6": "mno", "7": "pqrs", "8": "tuv", "9": "wxyz"}
    res = []
    def dfs(i, cur):                              # Style 2: pass a new string, no undo
        if i == len(digits):
            res.append(cur)                       # str is immutable: no .copy() needed
            return
        for c in digitToChar[digits[i]]:
            dfs(i + 1, cur + c)
    dfs(0, "")
    return res
```
Style 1 version of the same thing (shared list + undo):
```python
    path = []
    def dfs(i):
        if i == len(digits):
            res.append("".join(path))
            return
        for c in digitToChar[digits[i]]:
            path.append(c)
            dfs(i + 1)
            path.pop()
```
Recall: the `for` loop advances through "def" by itself. No special code moves it from 'd' to 'e'. Watch the dict typo: "pqrs", not "qprs".
```text
Time:  O(4^n * n)   Space: O(n)
```

## N-Queens
```python
def solveNQueens(self, n):
    col, pos_diag, neg_diag = set(), set(), set()    # c, r+c, r-c
    grid = [["."] * n for _ in range(n)]
    res = []
    def dfs(r):
        if r == n:
            res.append(["".join(row) for row in grid])
            return
        for c in range(n):
            if c in col or r + c in pos_diag or r - c in neg_diag:
                continue                               # NOT return
            col.add(c); pos_diag.add(r + c); neg_diag.add(r - c)
            grid[r][c] = "Q"
            dfs(r + 1)
            col.remove(c); pos_diag.remove(r + c); neg_diag.remove(r - c)
            grid[r][c] = "."
    dfs(0)
    return res
```
Recall: `r+c` is constant along one diagonal direction, `r-c` along the other, so "is this diagonal taken" is O(1). One queen per row, so depth = n.
```text
Inside a for-loop that tries options:
    continue -> "this option is invalid, try the NEXT one"
    return   -> "this whole call is finished"
A pruning check on one choice is almost always `continue`. `return` kills every remaining column.
```
```text
Time:  O(n!)   Space: O(n^2) board + O(n) recursion
```

### 1D vs 2D comprehension, and the trap
```python
[-s for s in rows]                 # 1D: one loop, one value per iteration -> flat list
[['.'] * n for _ in range(n)]      # 2D: outer loop builds n FRESH rows

board = [['.'] * n] * n            # WRONG: one row object referenced n times.
                                   # board[0][0] = 'Q' also changes board[1][0], board[2][0]...
```
```text
board[r]        -> entire row r          len(board)     -> number of rows (m)
board[r][c]     -> cell at row r, col c  len(board[0])  -> number of columns (n)
```

---

## Grid DFS: Number of Islands / Max Area of Island
Same visit-and-mark idea as Word Search, but no undo: marking the cell as visited is permanent.
```python
# Number of Islands: grid holds STRINGS "0"/"1". dfs only MARKS; the outer loop counts.
def numIslands(self, grid):
    m, n = len(grid), len(grid[0])
    count = 0
    def dfs(i, j):
        if i < 0 or j < 0 or i >= m or j >= n or grid[i][j] == "0":
            return                              # bare return: caller ignores the result
        grid[i][j] = "0"
        dfs(i+1, j); dfs(i-1, j); dfs(i, j+1); dfs(i, j-1)    # all 4, plain calls (no `or`)
    for i in range(m):
        for j in range(n):
            if grid[i][j] == "1":
                dfs(i, j)
                count += 1                       # a fresh "1" must be a NEW island
    return count

# Max Area of Island: grid holds INTS 0/1. dfs RETURNS the size.
def maxAreaOfIsland(self, grid):
    m, n = len(grid), len(grid[0])
    best = 0
    def dfs(i, j):
        if i < 0 or j < 0 or i >= m or j >= n or grid[i][j] == 0:
            return 0                             # must be 0: parents add these up
        grid[i][j] = 0
        return 1 + dfs(i+1, j) + dfs(i-1, j) + dfs(i, j+1) + dfs(i, j-1)
    for i in range(m):
        for j in range(n):
            if grid[i][j] == 1:
                best = max(best, dfs(i, j))
    return best
```
Recall: `count += 1` needs no boolean. The outer loop only calls dfs on a still-unvisited land cell, and dfs flood-fills the whole island, so each call = one new island. Check the input type: strings in Number of Islands, ints in Max Area.
```text
Time:  O(rows * cols)   Space: O(rows * cols) recursion in the worst case
```

---

# Bug checklist (every mistake we hit while practicing)

```text
Copying
  [ ] Saving a list/set snapshot without .copy() / [:]  -> res full of empties

Returning
  [ ] Recorded an answer but forgot `return` -> falls through, duplicates
  [ ] Base case is a bare `return` but parents do 1 + dfs(...) -> None + int crash. Use return 0
  [ ] Boolean search: no SUCCESS base case, or the caller ignores dfs's result
  [ ] `else: return False` inside a "try every start" loop -> exits after the first failure
  [ ] `or` chain used where ALL branches must run (counting, flood fill)

Control flow
  [ ] return instead of continue when pruning ONE option inside a for-loop
  [ ] Success check placed AFTER the range check when the include branch moves to i+1
  [ ] Skip-duplicates while loop not bounded (i + 1 < len)

State
  [ ] Running total not mirrored with append/pop (exclude branch still adds the value)
  [ ] append + pop AND passing a new value at the same time (mixed styles)
  [ ] Forgot an undo (pop / remove / unmark) -> state leaks into the sibling branch
  [ ] Changed dfs from "mutate" to "return a value" but left old params/call sites/returns

Python
  [ ] x = lst.sort() -> None (sort is in place; sorted() returns a new list)
  [ ] Assigning to a name anywhere in a function makes it local everywhere in it (UnboundLocalError)
  [ ] [[x]*n]*n shares one row n times
  [ ] Bounds: r against m (rows), c against n (cols). Not the same dimension twice
  [ ] Wrong variable name (subset vs subsets) or typos (df vs dfs)
```

---

## One-Line Pattern Recall
```text
Subsets                 -> include/exclude binary choice per index
Combination Sum          -> stay at i to allow reuse, i+1 to move on; exclude passes remain unchanged
Permutations             -> shared perms list + "nums[i] not in perms" (distinct values only)
Permutations II          -> SORT + used[] by INDEX + skip nums[i]==nums[i-1] and not used[i-1]
Subsets II / CombSum II  -> SORT first, skip duplicates at the same depth (j>i, or while-skip)
Word Search               -> success base case first, visited set, remove on backtrack, USE the result
Palindrome Partitioning   -> try every cut, recurse only if the piece is a palindrome
Letter Combinations        -> pass cur + c (no undo) OR shared path + pop. Never mix
N-Queens                  -> r+c / r-c diagonals, `continue` to prune, one queen per row
Islands                   -> mark visited permanently; bare return if only marking, return 0 if summing

Base case returns: bare return (ignored) | 0 (summed) | True/False (branched on)
Parameter = down the path | return = up from children | shared var = one global answer
```
