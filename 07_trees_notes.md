# Binary Tree Patterns — Python Revision Notes

# 1. Core Basics

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

```text
DFS -> recursion, go deep, "combine children's answers"
BFS -> queue, go level by level

Need depth/height/path/compare-structure?      -> DFS
Need level-grouped output or shortest hops?    -> BFS
```

---

# 2. DFS — "Combine Children" Pattern

Template:
```python
def dfs(node):
    if not node:
        return <base_case>
    left = dfs(node.left)
    right = dfs(node.right)
    return <combine left, right, node.val>
```
Mental model: trust recursion to solve both subtrees — your job is just combining their answers.

## Invert Binary Tree
```python
def invertTree(self, root):
    if not root:
        return None
    root.left, root.right = self.invertTree(root.right), self.invertTree(root.left)
    return root
```
Recall: swap the *recursive results*, not raw nodes.

```text
Time:  O(n) -- every node visited exactly once, O(1) work per node (one swap)
Space: O(h) -- recursion stack depth equals tree height h (O(log n) balanced, O(n) worst-case skewed)
```

## Max Depth
```python
def maxDepth(self, root):
    if not root:
        return 0
    return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
```
```text
Time:  O(n) -- visits every node once to compute 1 + max(left, right)
Space: O(h) -- recursion stack depth equals height h, same balanced/skewed split as Invert
```

## Diameter of Binary Tree
```python
def diameterOfBinaryTree(self, root):
    self.res = 0
    def dfs(node):
        if not node:
            return 0
        left, right = dfs(node.left), dfs(node.right)
        self.res = max(self.res, left + right)   # update global answer
        return 1 + max(left, right)                # return HEIGHT, not diameter
    dfs(root)
    return self.res
```
Recall: `self.res` = running max across WHOLE tree; return value ≠ the answer itself.

```text
Time:  O(n) -- single DFS pass, each node does O(1) work combining its children's heights
Space: O(h) -- recursion depth tracks height, not diameter (a bushy tree can have h << n)
```

## Balanced Binary Tree
```python
def isBalanced(self, root):
    def dfs(node):
        if not node:
            return [True, 0]
        left, right = dfs(node.left), dfs(node.right)
        balanced = left[0] and right[0] and abs(left[1]-right[1]) <= 1
        return [balanced, 1 + max(left[1], right[1])]
    return dfs(root)[0]     # don't forget the return!
```
Recall: return TWO pieces of info per node (status + height) when parent needs both.

```text
Time:  O(n) -- single bottom-up pass; bundling [status, height] together avoids the naive
       O(n^2) version that recomputes height from scratch at every node
Space: O(h) -- recursion stack proportional to height
```

## Same Tree
```python
def isSameTree(self, p, q):
    if not p and not q:
        return True
    if not p or not q or p.val != q.val:
        return False
    return self.isSameTree(p.left, q.left) and self.isSameTree(p.right, q.right)
```
```text
Time:  O(min(n_p, n_q)) -- recursion short-circuits the instant a mismatch is found,
       otherwise visits every node of the smaller tree once
Space: O(h) -- recursion depth bounded by the shallower tree's height
```

## Subtree of Another Tree
```python
def isSubtree(self, root, subRoot):
    def sameTree(a, b):
        if not a and not b:
            return True
        if not a or not b or a.val != b.val:
            return False
        return sameTree(a.left, b.left) and sameTree(a.right, b.right)

    if not subRoot:
        return True
    if not root:
        return False
    if sameTree(root, subRoot):
        return True
    return self.isSubtree(root.left, subRoot) or self.isSubtree(root.right, subRoot)
```
Bug to remember: define the helper BEFORE calling it (top-to-bottom execution).

```text
Time:  O(n * m) -- worst case, sameTree costs O(m) (m = size of subRoot) and can run once
       at every one of root's n nodes, e.g. when many nodes happen to share subRoot's value
Space: O(h) -- recursion stack depth, h = height of root
```

## Count Good Nodes in Binary Tree
```python
def goodNodes(self, root):
    self.count = 0
    def dfs(node, maxVal):
        if not node:
            return
        if node.val >= maxVal:
            self.count += 1
        maxVal = max(maxVal, node.val)
        dfs(node.left, maxVal)
        dfs(node.right, maxVal)
    dfs(root, root.val)
    return self.count
```
Recall: "good along a PATH" -> pass running state DOWN as a parameter (each branch needs its own copy). Contrast with Diameter, where the state is shared globally (`self.res`) because it's one answer for the whole tree, not per-path.

```text
Time:  O(n) -- single DFS pass, each node does O(1) work (one comparison against maxVal)
Space: O(h) -- recursion depth equals height; maxVal is passed by value, not accumulated per branch
```

## Binary Tree Maximum Path Sum
```python
def maxPathSum(self, root):
    self.res = root.val
    def dfs(node):
        if not node:
            return 0
        leftGain = max(dfs(node.left), 0)    # clip negative contributions to 0
        rightGain = max(dfs(node.right), 0)
        self.res = max(self.res, node.val + leftGain + rightGain)   # "peak" using BOTH sides
        return node.val + max(leftGain, rightGain)                    # but only ONE side upward
    dfs(root)
    return self.res
```
Recall: same `self.res` trick as Diameter. You're allowed to use BOTH children to update the global max (a path can bend once, at the peak), but you can only RETURN one side to the parent — a path can't branch twice.

```text
Time:  O(n) -- single DFS pass, each node does O(1) work combining its two children's gains
Space: O(h) -- recursion stack depth equals height
```

## Distribute Coins in Binary Tree (extra, not in the 150)
```python
def distributeCoins(self, root):
    res = 0
    def dfs(node):
        nonlocal res
        if not node:
            return [0, 0]
        l_size, l_coins = dfs(node.left)
        r_size, r_coins = dfs(node.right)
        size = 1 + l_size + r_size
        coins = node.val + l_coins + r_coins
        res += abs(size - coins)     # this subtree's imbalance = moves crossing its parent edge
        return [size, coins]
    dfs(root)
    return res
```
Recall: return BOTH size and coin-count per subtree (like Balanced Tree's `[status, height]` pair). `abs(size - coins)` at a node = coins that must cross the edge to its parent. Root's own imbalance is always 0 (total coins = total nodes), so including it in the sum is harmless — no need to special-case it.

```text
Time:  O(n) -- single bottom-up pass, each node does O(1) work (one addition to res)
Space: O(h) -- recursion stack depth equals height; the [size, coins] pair per call is O(1)
```

---

# 3. `self.x` vs `nonlocal` vs return value

```text
self.x    -> class method, nested helper needs to persist/share state -> self.res = 0
nonlocal  -> plain function, nested helper needs to MODIFY an outer var
return    -> if answer is fully derivable from children's returns, just return it (simplest)

Per-PATH state (Good Nodes, BST bounds) -> pass DOWN as parameter, not shared
Per-TREE state (Diameter, Max Path Sum, node count) -> self.res / nonlocal, shared globally
```

---

# 4. BST-Specific Patterns

## Validate Binary Search Tree
```python
def isValidBST(self, root):
    def dfs(node, low, high):
        if not node:
            return True
        if not (low < node.val < high):
            return False
        return dfs(node.left, low, node.val) and dfs(node.right, node.val, high)
    return dfs(root, float('-inf'), float('inf'))
```
Recall: checking only vs immediate parent is NOT enough — a node must satisfy bounds set by EVERY ancestor. Thread `(low, high)` down, same "pass state as parameter" idea as Good Nodes.

```text
Time:  O(n) -- each node visited once, O(1) bound check per node
Space: O(h) -- recursion depth equals height; (low, high) are passed by value, not stored
```

## Lowest Common Ancestor (BST)
```python
def lowestCommonAncestor(self, root, p, q):
    curr = root
    while True:
        if p.val > curr.val and q.val > curr.val:
            curr = curr.right
        elif p.val < curr.val and q.val < curr.val:
            curr = curr.left
        else:
            return curr     # return the NODE, not curr.val
```
Recall: needs `while True` loop. Return type is `TreeNode` — never return `.val`.

```text
Time:  O(h) -- NOT O(n): each step moves one level down toward p or q, so the walk is
       bounded by height rather than total node count
Space: O(1) -- iterative while loop, no recursion stack
```

## Lowest Common Ancestor (general tree — extra, not in the 150)
```python
def lowestCommonAncestor(self, root, p, q):
    if not root or root == p or root == q:
        return root
    left = self.lowestCommonAncestor(root.left, p, q)
    right = self.lowestCommonAncestor(root.right, p, q)
    if left and right:
        return root          # p, q split across both sides -> this IS the LCA
    return left or right     # both on one side (or neither found) -> bubble up
```
Recall: can't use value comparison without BST ordering — must search both sides.

```text
Time:  O(n) -- without BST ordering to guide the search, every node must potentially be visited once
Space: O(h) -- recursion stack depth equals height
```

## Kth Smallest Element in a BST
```python
def kthSmallest(self, root, k):
    arr = []
    def dfs(node):
        if not node:
            return
        dfs(node.left)
        arr.append(node.val)
        dfs(node.right)
    dfs(root)
    return arr[k - 1]
```
Recall: **in-order (L, node, R) on a BST always visits values in SORTED order.** That's the whole trick.

```text
Time:  O(n) -- in-order DFS visits every node once to build arr; doesn't stop early at k
Space: O(n) -- arr stores every value; recursion stack adds O(h) on top
```

## Construct Binary Tree from Preorder and Inorder Traversal
```python
def buildTree(self, preorder, inorder):
    if not preorder or not inorder:
        return None
    root_val = preorder[0]
    root = TreeNode(root_val)
    mid = inorder.index(root_val)
    root.left = self.buildTree(preorder[1:mid+1], inorder[:mid])
    root.right = self.buildTree(preorder[mid+1:], inorder[mid+1:])
    return root
```
Recall:
```text
preorder = [root, ...left..., ...right...]   -> preorder[0] is always the root
inorder  = [...left..., root, ...right...]   -> find root's index -> splits left/right sizes
```
Use the split point (`mid`) from inorder to slice preorder into matching left/right chunks. (Slicing is O(n) per call -> O(n²) total; can optimize with a val→index hashmap + pointers if needed, but this is the interview-baseline version.)

```text
Time:  O(n^2) worst case -- inorder.index() is O(n) per call and runs once per node (n calls),
       and slicing preorder/inorder is also O(n) per call; a skewed tree makes both worse
Space: O(n^2) -- repeated slicing allocates new lists at every recursion level
       (the hashmap + pointer optimization brings both down to O(n))
```

---

# 5. BFS Patterns

Base template:
```python
from collections import deque
q = deque([root])
while q:
    node = q.popleft()
    if node.left:  q.append(node.left)
    if node.right: q.append(node.right)
```
The inner `for` loop is what makes BFS level-aware:
```text
Need output GROUPED BY LEVEL?  -> for _ in range(len(q)) before popping — freezes level size
Just need to visit every node? -> no for-loop needed
```
Trap: `len(q)` must be captured BEFORE the loop starts appending children, or the count includes next level's nodes.

## Binary Tree Level Order Traversal
```python
def levelOrder(self, root):
    if not root:
        return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result
```
Bug to remember: loop on `while queue:`, NOT `while root:` — `root` never changes, so looping on it never terminates.

```text
Time:  O(n) -- every node enqueued and dequeued exactly once, O(1) work per node
Space: O(n) -- queue holds up to the widest level's worth of nodes (up to ~n/2 for a
       complete tree); result also stores all n values
```

## Binary Tree Right Side View
```python
def rightSideView(self, root):
    if not root:
        return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level.pop())     # last node in the level = rightmost
    return result
```
Bug to remember: always append the CURRENT popped `node`'s children, never the outer `root`'s.

```text
Time:  O(n) -- same BFS traversal as level order, every node visited once
Space: O(n) -- queue width bounded by the widest level; result stores one value per level
```

## Serialize and Deserialize Binary Tree
```python
class Codec:
    def serialize(self, root):
        res = []
        def dfs(node):
            if not node:
                res.append('N')
                return
            res.append(str(node.val))
            dfs(node.left)
            dfs(node.right)
        dfs(root)
        return ','.join(res)

    def deserialize(self, data):
        vals = iter(data.split(','))
        def dfs():
            val = next(vals)
            if val == 'N':
                return None
            node = TreeNode(int(val))
            node.left = dfs()
            node.right = dfs()
            return node
        return dfs()
```
Recall: preorder (root, left, right) + explicit `'N'` markers for `None` = unambiguous to rebuild. `deserialize` walks the SAME order, consuming one token at a time via `iter`/`next`.

```text
Time:  O(n) -- serialize visits every node once; deserialize consumes each token once via next(vals)
Space: O(n) -- the serialized string holds one entry per node plus one 'N' per null child;
       recursion stack adds O(h) on top
```

---

# 6. One-Line Pattern Recall

```text
Invert              -> swap recursive results of left/right
Max depth           -> 1 + max(left, right)
Diameter            -> self.res = max(res, left+right); RETURN height
Balanced            -> return [is_balanced, height] pair
Same tree           -> compare val + recurse both sides
Subtree             -> sameTree helper defined FIRST, then scan every node
Good nodes          -> pass running max DOWN as parameter (per-path state)
Max path sum        -> self.res tracks peak (both sides); return only ONE side up
Validate BST        -> pass (low, high) bounds DOWN, not just compare to parent
LCA (BST)           -> while True + value comparison, return node not val
LCA (general)       -> left/right recursion, both non-None = split point
Kth smallest BST    -> in-order traversal = sorted order
Build tree pre+in   -> preorder[0] = root; inorder.index(root) = left/right split point
Level order         -> deque + for _ in range(len(q)) to freeze level size
Right side view     -> level order + take level.pop() (last = rightmost)
Serialize/Deserial. -> preorder + 'N' markers; rebuild with iter/next, same order

Per-PATH state (good nodes, BST bounds)      -> pass as parameter
Per-TREE running max (diameter, path sum)     -> self.res / nonlocal
```
