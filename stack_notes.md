# Stack — Python Revision Notes

Core idea: LIFO — last in, first out. Use a plain Python `list` as a stack (`.append()` = push, `.pop()` = pop). Reach for a stack whenever a problem involves **matching/undoing the most recent thing**, or you need to compare each element against the "nearest usable" one before/after it (monotonic stack).

```text
Matching pairs / nesting / undo-last     -> plain stack
"Next/previous greater or smaller element"-> monotonic stack (keep it sorted as you go)
```

---

## Valid Parentheses
```python
def isValid(self, s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for c in s:
        if c in pairs:
            if not stack or stack[-1] != pairs[c]:
                return False
            stack.pop()
        else:
            stack.append(c)
    return not stack
```
Recall: closing bracket -> must match the TOP of the stack (the most recent open one). Empty stack at the end = everything matched.

## Min Stack
```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.minStack = []       # minStack[i] = min of stack[0..i]

    def push(self, val):
        self.stack.append(val)
        val = min(val, self.minStack[-1] if self.minStack else val)
        self.minStack.append(val)

    def pop(self):
        self.stack.pop()
        self.minStack.pop()

    def top(self):
        return self.stack[-1]

    def getMin(self):
        return self.minStack[-1]
```
Recall: second stack tracks the running min AT EACH DEPTH, so popping the main stack automatically "restores" the correct previous min — no recomputation needed.

## Evaluate Reverse Polish Notation
```python
def evalRPN(self, tokens):
    stack = []
    for t in tokens:
        if t in '+-*/':
            b, a = stack.pop(), stack.pop()   # note order: b popped first (top of stack)
            if t == '+': stack.append(a + b)
            elif t == '-': stack.append(a - b)
            elif t == '*': stack.append(a * b)
            else: stack.append(int(a / b))    # truncate toward zero
        else:
            stack.append(int(t))
    return stack[0]
```
Recall: `b` is popped BEFORE `a` — top of stack is the more recent/right operand, so `a - b` (not `b - a`) preserves correct order.

## Generate Parentheses
```python
def generateParenthesis(self, n):
    res = []
    def backtrack(openN, closeN, path):
        if openN == closeN == n:
            res.append(''.join(path))
            return
        if openN < n:
            path.append('(')
            backtrack(openN + 1, closeN, path)
            path.pop()                          # undo -- try the other branch
        if closeN < openN:
            path.append(')')
            backtrack(openN, closeN + 1, path)
            path.pop()
    backtrack(0, 0, [])
    return res
```
Recall: `path` acts as an implicit stack for backtracking — push a choice, recurse, pop it back off to try the next option. Close only allowed when `closeN < openN` (never more closes than opens so far).

## Daily Temperatures
```python
def dailyTemperatures(self, temperatures):
    res = [0] * len(temperatures)
    stack = []    # stores INDICES, temps kept decreasing bottom to top
    for i, t in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < t:
            prevIdx = stack.pop()
            res[prevIdx] = i - prevIdx
        stack.append(i)
    return res
```
Recall: monotonic DEcreasing stack of indices. A warmer day pops everything colder than it off the stack — that's the "next greater element" pattern.

## Car Fleet
```python
def carFleet(self, target, position, speed):
    pairs = sorted(zip(position, speed), reverse=True)   # closest to target first
    stack = []
    for pos, spd in pairs:
        time = (target - pos) / spd
        stack.append(time)
        if len(stack) >= 2 and stack[-1] <= stack[-2]:
            stack.pop()          # this car catches up -> merges into fleet ahead, doesn't add a new fleet
    return len(stack)
```
Recall: sort by position descending (nearest to target processed first), stack holds arrival times of DISTINCT fleets — a car that arrives sooner-or-equal to the fleet ahead just merges in.

## Largest Rectangle in Histogram
```python
def largestRectangleArea(self, heights):
    stack = []    # (start_index, height), heights kept increasing
    maxArea = 0
    for i, h in enumerate(heights):
        start = i
        while stack and stack[-1][1] > h:
            idx, height = stack.pop()
            maxArea = max(maxArea, height * (i - idx))
            start = idx                    # this bar's rectangle can extend back to idx
        stack.append((start, h))
    for idx, height in stack:
        maxArea = max(maxArea, height * (len(heights) - idx))
    return maxArea
```
Recall: monotonic INcreasing stack of `(start, height)`. When a shorter bar comes, every taller bar behind it has found its right boundary — close out its rectangle now.

---

## One-Line Pattern Recall
```text
Matching brackets           -> stack, closing must match top
Min stack                   -> parallel stack tracking running min at each depth
RPN eval                    -> pop b then a (b = top = right operand)
Backtracking w/ path         -> path list = manual stack, append then pop to undo
Next greater element         -> monotonic DEcreasing stack of indices
Car fleet                   -> sort by position, stack of arrival times, merge if <= previous
Largest rectangle            -> monotonic INcreasing stack of (start_idx, height)
```
