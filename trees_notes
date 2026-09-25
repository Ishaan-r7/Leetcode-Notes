# Binary Tree Patterns — Python Revision Notes

# 1. Core Basics

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

Two families of tree problems:

```text
DFS  -> recursion, go deep first, "return info from children"
BFS  -> queue, go level by level
```

Rule of thumb:

```text
Need depth/height/path/compare-structure? -> DFS
Need level-by-level output, or shortest path in unweighted tree? -> BFS
```

---

# 2. DFS — "Combine Children" Pattern

Used in: Invert Tree, Max Depth, Diameter, Balanced Tree, Same Tree, Subtree.

## Template

```python
def dfs(node):
    if not node:
        return <base_case>

    left = dfs(node.left)
    right = dfs(node.right)

    return <combine left, right, node.val>
```

Mental model:

```text
Trust the recursion to solve left and right subtrees.
Your only job at this node: combine their answers.
```

---

## Invert Binary Tree

```python
def invertTree(self, root):
    if not root:
        return None

    root.left, root.right = self.invertTree(root.right), self.invertTree(root.left)
    return root
```

Recall: swap children, but swap the *recursive results*, not the raw nodes — otherwise you invert only one level.

---

## Max Depth

```python
def maxDepth(self, root):
    if not root:
        return 0
    return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
```

---

## Diameter of Binary Tree

Diameter = longest path, which may NOT pass through root. Classic trick: track a running max in an outer variable while still returning height for recursion.

```python
def diameterOfBinaryTree(self, root):
    self.res = 0

    def dfs(node):
        if not node:
            return 0
        left = dfs(node.left)
        right = dfs(node.right)
        self.res = max(self.res, left + right)   # update global answer
        return 1 + max(left, right)               # return HEIGHT, not diameter

    dfs(root)
    return self.res
```

Recall: `self.res` because helper is nested but still needs to persist state across all calls — see nonlocal/self.X section below.

---

## Balanced Binary Tree

Return **two pieces of info** per node: (is it balanced, height).

```python
def isBalanced(self, root):
    def dfs(node):
        if not node:
            return [True, 0]
        left = dfs(node.left)
        right = dfs(node.right)
        balanced = left[0] and right[0] and abs(left[1] - right[1]) <= 1
        return [balanced, 1 + max(left[1], right[1])]

    return dfs(root)[0]   # don't forget the return!
```

---

## Same Tree

```python
def isSameTree(self, p, q):
    if not p and not q:
        return True
    if not p or not q or p.val != q.val:
        return False
    return self.isSameTree(p.left, q.left) and self.isSameTree(p.right, q.right)
```

(BFS version also works — compare nodes level by level with two queues.)

---

## Subtree of Another Tree

Two nested problems: "does tree match exactly" (= isSameTree logic) run at every node of the main tree.

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

Bug to remember: **define the helper BEFORE you call it.** Defining `sameTree` after `return` in the function body means it doesn't exist yet when called (top-to-bottom execution).

---

# 3. `self.x` vs `nonlocal` vs helper return value

You need *some* way to keep state alive across recursive calls when the answer isn't just "what this call returns."

```text
Option A — self.x
    Use inside a class method when the helper is nested inside the main method.
    self.res = 0  (set before recursion starts)
    Works because `self` is shared across all nested calls automatically —
    no extra keyword needed.

Option B — nonlocal
    Use when you're in a plain (non-class) function with a nested helper,
    and the helper needs to MODIFY a variable from the enclosing scope.
    def outer():
        res = 0
        def dfs(node):
            nonlocal res
            res = max(res, ...)
        dfs(root)
        return res
    Without `nonlocal`, `res = ...` inside dfs creates a NEW local variable
    instead of updating the outer one -> silent bug, outer `res` stays 0.

Option C — just return it
    If the answer can be fully derived from what children return
    (no "track the best-so-far across the whole tree" need), just return
    it normally. Simplest, prefer this when possible (see isBalanced above).
```

Recall:

```text
Class method, nested helper -> self.res
Plain function, nested helper -> nonlocal res
No global tracking needed -> just return values up the call stack
```

---

# 4. Lowest Common Ancestor

## BST version — use value ordering, no need to search both sides

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

Recall: needs `while True` loop — a single if/elif/else only checks one level and falls off. Return type is `TreeNode`, so return `curr`, never `curr.val`.

## General binary tree version (not BST — can't use value comparison)

```python
def lowestCommonAncestor(self, root, p, q):
    if not root or root == p or root == q:
        return root

    left = self.lowestCommonAncestor(root.left, p, q)
    right = self.lowestCommonAncestor(root.right, p, q)

    if left and right:
        return root      # p and q found on different sides -> this is the split point
    return left or right # both on one side (or not found) -> bubble up whichever is non-None
```

---

# 5. BFS — Level Order Traversal

Base queue template:

```python
from collections import deque

q = deque([root])
while q:
    node = q.popleft()
    if node.left:  q.append(node.left)
    if node.right: q.append(node.right)
```

**The inner `for` loop is what turns plain BFS into LEVEL-AWARE BFS.**

```text
No inner for loop -> you just visit nodes one at a time, no idea which level they're on
Inner for loop over range(len(q)) -> freezes "how many nodes are in the CURRENT level"
                                       before you start popping, so you can group them
```

## Level Order Traversal

```python
def levelOrder(self, root):
    if not root:
        return []

    res = []
    q = deque([root])

    while q:
        level = []
        for _ in range(len(q)):       # <-- KEY LINE: locks in this level's size
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        res.append(level)

    return res
```

Recall trigger — **when do I need the for-loop?**

```text
Need output GROUPED BY LEVEL (level order, zigzag, right side view,
level averages, max per level)?
    -> YES, use for _ in range(len(q))

Just need to visit every node / find one thing anywhere in tree
(e.g. plain BFS search, shortest path exists)?
    -> NO for-loop needed, plain while q: node = q.popleft() is enough
```

The trap: `len(q)` MUST be evaluated once and stored (`range(len(q))`) before the loop starts appending children — since `q` grows inside the loop, calling `len(q)` fresh each iteration would include next level's nodes and break the grouping.

---

# 6. One-Line Pattern Recall

```text
Invert            -> swap recursive results of left/right
Max depth         -> 1 + max(left, right)
Diameter          -> track self.res = max(res, left+right), RETURN height
Balanced          -> return [is_balanced, height] pair
Same tree         -> compare val + recurse both sides
Subtree           -> sameTree helper defined FIRST, then scan every node
LCA (BST)         -> while True + value comparison, return node not val
LCA (general)     -> left/right recursion, both non-None = split point
Level order       -> deque + for _ in range(len(q)) to freeze level size
Need running max across whole tree? -> self.res (class) or nonlocal (function)
```
