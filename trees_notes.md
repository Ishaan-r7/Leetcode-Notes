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

## Max Depth
```python
def maxDepth(self, root):
    if not root:
        return 0
    return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
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

## Same Tree
```python
def isSameTree(self, p, q):
    if not p and not q:
        return True
    if not p or not q or p.val != q.val:
        return False
    return self.isSameTree(p.left, q.left) and self.isSameTree(p.right, q.right)
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
