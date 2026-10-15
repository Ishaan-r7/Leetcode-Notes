# Graphs — Python Revision Notes

Core idea: a graph is nodes + edges. Almost every problem is "build neighbors, then DFS/BFS with a visited mark." Decide three things first: **what are the nodes, what are the neighbors, what counts as visited.**

```text
Grid of cells (islands, oranges, regions)   -> nodes = cells, neighbors = 4 directions, visited = mutate grid or set
Pairs like [a, b] (courses, edges)           -> build an adjacency list, then DFS/BFS
Count pieces / components                    -> outer loop over all nodes, one DFS per unseen node, count += 1
Shortest steps / spread over time            -> BFS (queue); multi-source BFS if many starting points
Is there a cycle?                            -> directed: 3-state DFS.  undirected: parent tracking / seen set
Ordering with "A before B" rules             -> topological sort (Kahn or DFS post-order)
```

---

# 1. Basics

## Adjacency list (the default way to store a graph)
```python
g = defaultdict(list)
for a, b in edges:
    g[a].append(b)            # directed: one way only
    g[b].append(a)            # undirected: ADD THIS LINE (both ways)
```
`defaultdict(list)` means `g[x]` for a node with no edges returns `[]` — the neighbor loop just doesn't run, no key-exists check needed.

Other forms: edge list (the raw input), adjacency matrix (`m[a][b] = 1`, O(N²) space, dense graphs only).

**Directed vs undirected is decided by the problem statement**, not the input format. The same input can give different answers (e.g. a diamond 0→1,0→2,1→3,2→3 has no directed cycle but has an undirected cycle).

## Grid template (DFS)
```python
def dfs(r, c):
    if r < 0 or r >= m or c < 0 or c >= n or grid[r][c] != 1:   # reject chain
        return
    grid[r][c] = 0               # mark visited by mutating the grid
    dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1)
```

## Bounds check: accept vs reject
```text
0 <= r < m   ==   r >= 0 and r < m           (chaining is just style)
Accept (in bounds):     0 <= r < m and 0 <= c < n                  -> AND
Reject (out of bounds): r < 0 or r >= m or c < 0 or c >= n         -> OR  (De Morgan)
Lower bound is 0 <= r, NEVER 0 < r.  Bounds go BEFORE grid[r][c].
grid[-1] does not error in Python — it wraps to the last row — so always check bounds.
```
Pick one style per function: accept style guards the recursive call, reject style guards the top.

## Where the starter loops live
A dfs/bfs helper only does the spreading. The loops that START it (over seeds, borders, all cells) go **outside** the helper. If they are indented inside the helper, nothing ever starts it.

## nonlocal vs mutation
```text
nonlocal is needed only to REBIND an outer name (=, +=, -=), usually an int counter.
Mutating in place (list.append, set.add, dict[k] = v, grid[r][c] = x) needs no nonlocal.
```
Without `nonlocal`, `fresh -= 1` inside a helper makes `fresh` a new local -> UnboundLocalError.

## Return value of the helper (same rule as backtracking)
```text
Caller ignores the result            -> bare return           (Number of Islands)
Caller sums the results              -> return 0 / 1 + ...    (Max Area of Island)
Caller branches on the result        -> return True / False   (cycle detection, redundant connection)
```
`count += 1` goes **directly** after the dfs call in Number of Islands: an unvisited 1 must be a new island, so no True/False is needed. Typo to avoid: `count =+ 1` silently assigns 1.

## BFS rules
```text
Mark visited on ENQUEUE (when appending), not on pop -> a node is never queued twice.
Level for-loop  for _ in range(len(queue))  ONLY if counting levels/time/distance.
Plain  while queue:  is enough if you only need to visit everything.
BFS vs DFS is queue vs stack/recursion, NOT where the helper is called.
```

---

# 2. Grid DFS — count and size

## Number of Islands
```python
def numIslands(self, grid):
    m, n = len(grid), len(grid[0])
    count = 0
    def dfs(r, c):
        if r < 0 or r >= m or c < 0 or c >= n or grid[r][c] != "1":
            return
        grid[r][c] = "0"
        dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1)

    for r in range(m):
        for c in range(n):
            if grid[r][c] == "1":
                dfs(r, c)
                count += 1
    return count
```
Recall: outer loop over every cell, dfs sinks the whole island, `count += 1` after the call.
```text
Time:  O(m*n) — each cell is visited once by the loop and sunk once by dfs
Space: O(m*n) worst case recursion depth (one huge island)
```

## Max Area of Island
```python
def maxAreaOfIsland(self, grid):
    m, n = len(grid), len(grid[0])
    def dfs(r, c):
        if r < 0 or r >= m or c < 0 or c >= n or grid[r][c] != 1:
            return 0                              # caller SUMS results -> return 0
        grid[r][c] = 0
        return 1 + dfs(r+1, c) + dfs(r-1, c) + dfs(r, c+1) + dfs(r, c-1)

    res = 0
    for r in range(m):
        for c in range(n):
            if grid[r][c] == 1:
                res = max(res, dfs(r, c))
    return res
```
Recall: the area is built from return values (1 + children), not from an outer counter. Same shape as Max Depth in trees.
```text
Time:  O(m*n)      Space: O(m*n) recursion depth worst case
```

---

# 3. Grid BFS — spreading and distance

## Rotting Oranges (multi-source BFS)
```python
def orangesRotting(self, grid):
    m, n = len(grid), len(grid[0])
    queue = deque()
    fresh = 0

    def rot(r, c):
        nonlocal fresh
        if r < 0 or r >= m or c < 0 or c >= n or grid[r][c] != 1:
            return                      # out of bounds, empty, or already rotten
        grid[r][c] = 2                  # rot it: this is also the visited mark
        queue.append((r, c))
        fresh -= 1

    for r in range(m):
        for c in range(n):
            if grid[r][c] == 2:
                queue.append((r, c))
            elif grid[r][c] == 1:
                fresh += 1

    time = 0
    while queue and fresh > 0:
        for _ in range(len(queue)):     # one full minute = one BFS level
            r, c = queue.popleft()
            rot(r + 1, c); rot(r - 1, c); rot(r, c + 1); rot(r, c - 1)
        time += 1
    return time if fresh == 0 else -1
```
Recall: put ALL rotten oranges in the queue first (multi-source), level loop because we count minutes, `fresh == 0` at the end or return -1. Use `r, c` from the pop (not stale loop vars), and `_` for unused loop counters.
```text
Time:  O(m*n) — each cell enters the queue at most once
Space: O(m*n) queue
```

## Walls and Gates (fill each room with distance to nearest gate)
```python
def wallsAndGates(self, rooms):
    m, n = len(rooms), len(rooms[0])
    def dfs(r, c, d):
        if r < 0 or r >= m or c < 0 or c >= n or rooms[r][c] < d:
            return                       # wall (-1), or already has a shorter distance
        rooms[r][c] = d
        dfs(r + 1, c, d + 1); dfs(r - 1, c, d + 1); dfs(r, c + 1, d + 1); dfs(r, c - 1, d + 1)

    for r in range(m):
        for c in range(n):
            if rooms[r][c] == 0:
                dfs(r, c, 0)             # start from every gate
```
Recall: `rooms[r][c] < d` is the whole trick — it rejects walls (-1 < d), and cells already closer to another gate; it keeps INF cells (INF > d). DFS from each gate overwrites with the smaller distance.
```text
Time:  worst case O(gates * m*n) for this DFS; the multi-source BFS version is O(m*n)
Space: O(m*n) recursion depth
```
BFS alternative: push all gates into the queue at once, pop and set `rooms[nr][nc] = rooms[r][c] + 1` for neighbors equal to INF (mark on enqueue). Each cell is set once, in order of distance.

## Pacific Atlantic Water Flow (reverse search from the oceans)
```python
def pacificAtlantic(self, heights):
    m, n = len(heights), len(heights[0])
    directions = [(1,0), (-1,0), (0,1), (0,-1)]

    def bfs(queue, visit):
        while queue:                                   # <- the loop, a single popleft is a bug
            r, c = queue.popleft()
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if (0 <= nr < m and 0 <= nc < n and (nr, nc) not in visit
                        and heights[nr][nc] >= heights[r][c]):   # uphill from the ocean
                    visit.add((nr, nc))
                    queue.append((nr, nc))

    p = {(0, j) for j in range(n)} | {(i, 0) for i in range(m)}
    a = {(m-1, j) for j in range(n)} | {(i, n-1) for i in range(m)}
    bfs(deque(p), p)
    bfs(deque(a), a)
    return [[r, c] for r, c in p & a]
```
Recall: don't start from every cell and ask "can water reach the ocean" (slow). Start from the oceans' border cells and move to neighbors with `>=` height (water flows downhill, so reverse = uphill). The cells in both visited sets are the answer.
```text
Time:  O(m*n)      Space: O(m*n)
```

## Surrounded Regions (reverse search from the border)
```python
def solve(self, board):
    m, n = len(board), len(board[0])
    def dfs(r, c):
        if r < 0 or r >= m or c < 0 or c >= n or board[r][c] != "O":
            return
        board[r][c] = "T"                # safe: connected to the border
        dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1)

    for r in range(m):                   # left and right columns  -> range(m)
        dfs(r, 0); dfs(r, n - 1)
    for c in range(n):                   # top and bottom rows      -> range(n)
        dfs(0, c); dfs(m - 1, c)

    for r in range(m):
        for c in range(n):
            if board[r][c] == "O":
                board[r][c] = "X"        # surrounded -> captured
            elif board[r][c] == "T":
                board[r][c] = "O"        # safe -> restore
```
Recall: 1) dfs from every border `O` -> mark `T`, 2) remaining `O` -> `X`, 3) `T` -> `O`. The border loops sit OUTSIDE dfs. Rows use `range(m)`, cols use `range(n)`. Two separate ifs (or two dfs calls) for opposite border cells, never `elif`. Marking `T` on the board replaces a visit set.
```text
Time:  O(m*n)      Space: O(m*n) recursion depth worst case
```

---

# 4. Graph from pairs — clone, components, tree

## Clone Graph
```python
def cloneGraph(self, node):
    old_new = {}                          # original node -> its clone

    def dfs(node):
        if node in old_new:               # already cloned -> reuse it
            return old_new[node]
        clone = Node(node.val)
        old_new[node] = clone             # register BEFORE recursing
        for nei in node.neighbors:
            clone.neighbors.append(dfs(nei))
        return clone

    return dfs(node) if node else None
```
Recall: dict old -> new is both the visited set and the memo. Create the clone ONCE, store it, register it before the neighbor loop (this is what stops cycles), return the stored clone, never a fresh `Node()`. Dict, not list.
```text
Time:  O(N + E)      Space: O(N)
```

## Number of Connected Components (undirected)
```python
def countComponents(self, n, edges):
    g = defaultdict(list)
    for a, b in edges:
        g[a].append(b)
        g[b].append(a)                    # undirected: both ways

    seen = set()
    def dfs(node):
        seen.add(node)
        for nei in g[node]:
            if nei not in seen:
                dfs(nei)

    count = 0
    for i in range(n):
        if i not in seen:                 # unseen node = start of a new component
            dfs(i)
            count += 1
    return count
```
Recall: outer loop over all n nodes (isolated nodes count as components), dfs marks the whole component, `count += 1` directly after the call.
```text
Time:  O(N + E)      Space: O(N + E)
```

## Valid Tree (undirected)
```python
def validTree(self, n, edges):
    if len(edges) != n - 1:               # too many -> cycle, too few -> disconnected
        return False
    g = defaultdict(list)
    for a, b in edges:
        g[a].append(b)
        g[b].append(a)

    seen = set()
    def dfs(node):
        seen.add(node)
        for nei in g[node]:
            if nei not in seen:
                dfs(nei)

    dfs(0)
    return len(seen) == n                 # connected: one dfs reached everything
```
Recall: tree = connected + no cycle = `n - 1` edges AND connected. With exactly `n - 1` edges, a cycle forces a disconnected piece, so the connectivity check alone catches both. Don't use the directed 3-state check here.
```text
Time:  O(N + E)      Space: O(N + E)
```

## Redundant Connection (tree + one extra edge)
```python
def findRedundantConnection(self, edges):
    g = defaultdict(list)

    def connected(src, target, seen):
        if src == target:
            return True
        seen.add(src)
        for nei in g[src]:
            if nei not in seen and connected(nei, target, seen):
                return True
        return False

    for a, b in edges:
        if connected(a, b, set()):        # a already reaches b -> this edge closes a cycle
            return [a, b]
        g[a].append(b)                    # otherwise add it, both ways
        g[b].append(a)
```
Recall: `connected(a, b)` runs on the graph BEFORE the edge is added, so it only sees earlier edges. First edge: graph empty, always False, gets added. `False` = a and b are in separate pieces, safe to join. `True` = already linked, this edge makes the cycle. The DFS searches for a PATH a -> b; the loop is that path plus the new edge. Fresh `seen` per edge. The first edge that closes a cycle is the last cycle edge in input order, which is what the problem wants.
```text
Time:  O(E * (V + E)) = O(V^2) since E ~ V      Space: O(V + E)
Faster: Union-Find (find(a) == find(b) means the edge closes a cycle), ~O(E * alpha)
```

---

# 5. Cycle detection

## Directed: 3-state DFS
```python
UNVISITED, VISITING, VISITED = 0, 1, 2
states = [UNVISITED] * n

def dfs(node):
    if states[node] == VISITING: return True      # back on the current path -> cycle
    if states[node] == VISITED:  return False     # proven safe, skip
    states[node] = VISITING
    for nei in g[node]:
        if dfs(nei):
            return True
    states[node] = VISITED
    return False

has_cycle = any(dfs(i) for i in range(n))         # outer loop covers all components
```
Recall: `VISITING` = on the CURRENT path (the 1s in `states` are always exactly the call stack). Seeing a `VISITING` node again = cycle. Seeing a `VISITED` node = fine (diamond shapes are not cycles). `VISITED` is never undone, so cost is O(N + E) (unlike Word Search, where the path is undone and the cost is exponential). Outer loop = driver across components; inner calls = follow edges. Leaf nodes (no neighbors) skip the loop and go straight to `VISITED`.
```text
Time:  O(N + E)      Space: O(N + E)
```
Note: Course Schedule's version returns `True` for "ok"; the above returns `True` for "cycle" — watch the meaning.

## Undirected: seen set + parent
```python
def dfs(node, parent):
    seen.add(node)
    for nei in g[node]:
        if nei == parent:             # the edge we just came through, not a cycle
            continue
        if nei in seen:               # reached an already-seen node another way -> cycle
            return True
        if dfs(nei, node):
            return True
    return False

for i in range(n):
    if i not in seen and dfs(i, -1):
        return True
```
Recall: both directions are stored, so the parent is always "seen" — skip it. Any other seen neighbor means a second route = cycle. Two states are enough. Assumes no duplicate edges between the same pair.

```text
Directed cycle   -> 3 states, True when you hit VISITING
Undirected cycle -> seen + parent, True when you hit a seen node that is not the parent
```

---

# 6. Course Schedule & Topological Sort

**Topological sort** = an order of the nodes so that for every edge `u -> v`, `u` comes before `v`. Only exists if there is no cycle. Not unique (any valid order is accepted).

```text
Course Schedule I  : does a valid order exist?  -> bool
Course Schedule II : give me one valid order     -> list ([] if cycle)
```

## Course Schedule I
```python
def canFinish(self, numCourses, prerequisites):
    g = defaultdict(list)
    for a, b in prerequisites:
        g[a].append(b)                    # edge a -> b  (a needs b)
    states = [0] * numCourses             # 0 UNVISITED, 1 VISITING, 2 VISITED

    def dfs(node):
        if states[node] == 2: return True
        if states[node] == 1: return False
        states[node] = 1
        for nei in g[node]:
            if not dfs(nei):
                return False
        states[node] = 2
        return True

    for i in range(numCourses):
        if not dfs(i):
            return False
    return True
```
```text
Time:  O(N + E)      Space: O(N + E)
```

## Course Schedule II — DFS version (post-order)
```python
def findOrder(self, numCourses, prerequisites):
    g = defaultdict(list)
    for c, p in prerequisites:
        g[c].append(p)                    # edge c -> p (c depends on p)
    states = [0] * numCourses
    order = []

    def dfs(node):
        if states[node] == 2: return True
        if states[node] == 1: return False        # cycle
        states[node] = 1
        for nei in g[node]:
            if not dfs(nei):
                return False
        states[node] = 2
        order.append(node)                # AFTER all its prerequisites are in the list
        return True

    for i in range(numCourses):
        if not dfs(i):
            return []
    return order
```
Recall: it's Course Schedule I plus one line, `order.append(node)` AFTER the neighbor loop. A node is appended only after everything it points to. Edges here point course -> prerequisite, so prerequisites land first: **no reverse needed**. If you build edges the other way (`g[p].append(c)`), return `order[::-1]`.

## Course Schedule II — Kahn's algorithm (BFS with in-degrees)
```python
def findOrder(self, numCourses, prerequisites):
    g = defaultdict(list)
    indeg = [0] * numCourses
    for c, p in prerequisites:            # "p before c"
        g[p].append(c)                    # arrow p -> c  (p unlocks c)
        indeg[c] += 1                     # c is the one waiting

    queue = deque(i for i in range(numCourses) if indeg[i] == 0)
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for nei in g[node]:
            indeg[nei] -= 1               # one prerequisite of nei is now done
            if indeg[nei] == 0:           # all prerequisites done -> ready
                queue.append(nei)

    return order if len(order) == numCourses else []
```
Recall: start with all in-degree-0 nodes, pop one, decrement its neighbors, enqueue a neighbor when its in-degree hits 0. Cycle iff `len(order) < n` (nodes in a cycle never reach in-degree 0). No level for-loop needed unless counting rounds/semesters.

## Reading the edge direction (the confusing part)
Don't memorize `(a, b)`. Read the sentence, draw the arrow from the one that goes FIRST to the one that goes LATER, and put the in-degree on the arrowhead (the later one).
```text
"to take a you must first take b" (Course Schedule, [a,b])  -> arrow b -> a ; indeg[a] += 1 ; g[b].append(a)
"a must come before b"                                       -> arrow a -> b ; indeg[b] += 1 ; g[a].append(b)
Everything else in Kahn (queue, loop, len(order) == n check) is identical in both.
```

## DFS vs Kahn
```text
DFS : reuses the 3-state cycle code; order comes out post-order (reverse if edges are first -> later); recursion depth
Kahn: no recursion, no reversing; cycle check is one comparison; easier to explain in an interview
```
```text
Time:  O(N + E)      Space: O(N + E)
```

---

# 7. Complexity — how to reason about graphs

```text
Graph in adjacency list form: every node is processed once, every edge looked at once (or twice if undirected)
  -> DFS/BFS with a visited mark = O(N + E)   (grid: N = m*n, E ~ 4*m*n, so O(m*n))
  -> Space: visited + recursion/queue = O(N), plus O(N + E) for the adjacency list itself
Visited is never undone  -> linear.   Visited IS undone (backtracking path) -> exponential.
Fresh DFS per edge (Redundant Connection) -> multiply: O(E * (V + E)).
Recursion depth can reach N (a long chain): mention it, or use BFS / an explicit stack.
```

---

# 8. One-Line Pattern Recall

```text
Count pieces (islands/components)   -> outer loop, dfs only if unseen, count += 1 right after the call
Size of a piece                     -> dfs returns 1 + sum of neighbors (return 0 on reject)
Spread over time / distance         -> multi-source BFS, ALL sources enqueued first, level loop for time
Reverse search                      -> start from the targets (oceans, border), move to >= neighbors
Surrounded Regions                  -> border O -> T, remaining O -> X, T -> O
Clone graph                         -> dict old -> new, register clone BEFORE the neighbor loop
Directed cycle                      -> 3 states, True on VISITING
Undirected cycle                    -> seen + parent
Valid tree                          -> len(edges) == n-1 AND one dfs reaches all n
Redundant connection                -> for each edge: does a already reach b? yes -> answer, no -> add both ways
Topological sort                    -> Kahn (indegree) or DFS post-order (append after the loop)
Edge direction                      -> arrow from FIRST to LATER; indegree++ on the later one
BFS                                 -> mark visited on enqueue; level loop only when counting levels
nonlocal                            -> only to rebind an int/str outside; mutation needs nothing
Starter loops                       -> OUTSIDE the dfs/bfs helper
```
