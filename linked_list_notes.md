# Linked List Patterns — Python Revision Notes

## 1. Core Node Idea

A normal singly linked-list node has:

```python
node.val
node.next
```

For random-pointer problems:

```python
node.val
node.next
node.random
```

Important:

```text
.next points to the next NODE, not just the next value.
```

Example:

```text
1 -> 2 -> 3
```

`head.next` points to node `2`, and from node `2` you can still access:

```text
2 -> 3
```

---

# 2. Basic Traversal

```python
cur = head

while cur:
    # process cur
    cur = cur.next
```

Mental model:

```text
cur = node currently being processed
```

Usually keep `head` unchanged and traverse with `cur`.

---

# 3. Dummy Node Pattern

Dummy nodes have two main uses.

## Case A — Building a New List

Example: Merge Two Sorted Lists

```python
dummy = ListNode()
tail = dummy
```

Initially:

```text
dummy
  |
  v
0 -> None
```

The dummy is not connected to an existing list yet.

As the result is built:

```text
dummy -> 1 -> 2 -> 3
                    ^
                   tail
```

Pattern:

```python
tail.next = some_node
tail = tail.next
```

Return:

```python
return dummy.next
```

because the dummy node itself is fake.

Use this for:

```text
Building a result list
Merging lists
Creating a list incrementally
```

---

## Case B — Fake Node Before Existing Head

Example: Remove Nth Node From End

```python
dummy = ListNode(0, head)
```

This means:

```python
dummy.val = 0
dummy.next = head
```

So:

```text
dummy -> 1 -> 2 -> 3
```

This is useful when the original head might be deleted.

Return:

```python
return dummy.next
```

because the head may have changed.

### Quick Recall

```text
ListNode()          -> dummy starts disconnected

ListNode(0, head)   -> dummy is placed before head
```

---

# 4. Dummy + Tail Pattern

Used when building a list.

```python
dummy = ListNode()
tail = dummy

while ...:
    tail.next = node
    tail = tail.next

return dummy.next
```

Roles:

```text
dummy = permanently remembers the start
tail  = moves as the result list grows
```
## Dummy + Tail: Existing Node vs New Node

This distinction is important.

### Case 1 — We already have a node

Example: **Merge Two Sorted Lists**

```python
tail.next = list1
tail = tail.next
```

Here, `list1` is already a `ListNode`.

Example:

```text
list1
  |
  v
Node(2) -> Node(4) -> Node(6)
```

So:

```python
tail.next = list1
```

means:

```text
Connect tail to this already-existing node.
```

No new node needs to be created.

---

### Case 2 — We only have a value

Example: **Add Two Numbers**

Suppose:

```python
digit = 7
```

`digit` is just an integer.

This is wrong:

```python
tail.next = digit
```

because `.next` must point to a `ListNode`, not an integer.

So we create a new node:

```python
tail.next = ListNode(digit)
tail = tail.next
```

`ListNode(digit)` creates:

```text
Node:
val = digit
next = None
```

Example:

```python
digit = 7
tail.next = ListNode(7)
```

creates:

```text
tail -> Node(7) -> None
```

---

### Quick Rule

```text
Already have a node?
-> tail.next = existing_node

Only have a value?
-> tail.next = ListNode(value)
```

Examples:

```python
# Merge Two Sorted Lists
tail.next = list1
```

because `list1` is already a node.

```python
# Add Two Numbers
tail.next = ListNode(digit)
```

because `digit` is only an integer.

---

## Add Two Numbers — Pattern

Main idea:

```text
Add both digits
+ carry
↓
Store result digit
↓
Update carry
↓
Create new result node
```

Core formulas:

```python
summ = val1 + val2 + carry

digit = summ % 10
carry = summ // 10
```

Then create the result node:

```python
tail.next = ListNode(digit)
tail = tail.next
```

Full iterative pattern:

```python
dummy = ListNode()
tail = dummy
carry = 0

while l1 or l2 or carry:

    val1 = l1.val if l1 else 0
    val2 = l2.val if l2 else 0

    summ = val1 + val2 + carry

    digit = summ % 10
    carry = summ // 10

    tail.next = ListNode(digit)
    tail = tail.next

    if l1:
        l1 = l1.next

    if l2:
        l2 = l2.next

return dummy.next
```

Pattern recall:

```text
Add Two Numbers
-> Dummy + Tail + Carry
```

---

# 5. Reverse Linked List — Iterative

Very important template:

```python
prev = None
cur = head

while cur:
    nxt = cur.next
    cur.next = prev
    prev = cur
    cur = nxt

return prev
```

Memorize as:

```text
save next
reverse link
move prev
move cur
```

Example:

```text
1 -> 2 -> 3

becomes

3 -> 2 -> 1
```

Important:

```python
nxt = cur.next
```

must happen before:

```python
cur.next = prev
```

otherwise you lose the remaining list.

---

# 6. Reverse Linked List — Recursive

Core pattern:

```python
newHead = self.reverseList(head.next)

head.next.next = head
head.next = None

return newHead
```

Mental model:

```text
Reverse everything after me first.
Then attach me at the end.
```

Example:

```text
2 -> 3
```

This:

```python
head.next.next = head
```

creates:

```text
3 -> 2
```

Then:

```python
head.next = None
```

breaks the original forward connection.

---

# 7. Fast + Slow Pointer Pattern

```python
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

Meaning:

```text
slow moves 1 step
fast moves 2 steps
```

Common uses:

```text
Find middle
Split list
Detect cycle
Reorder list
Palindrome list
```

---

# 8. Find Middle / Split List

Example:

```python
slow, fast = head, head.next

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

After this:

```text
slow = end of first half
slow.next = start of second half
```

To split:

```python
second = slow.next
slow.next = None
```

Now the two halves are disconnected.

---

# 9. Reorder List

Pattern:

```text
1. Find middle
2. Reverse second half
3. Merge both halves alternately
```

```python
# Find middle
slow, fast = head, head.next

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next

# Reverse second half
second = slow.next
slow.next = None

prev = None

while second:
    nxt = second.next
    second.next = prev
    prev = second
    second = nxt

# Merge both halves
first, second = head, prev

while second:
    nxt1 = first.next
    nxt2 = second.next

    first.next = second
    second.next = nxt1

    first = nxt1
    second = nxt2
```

Recall:

```text
Middle + Reverse + Merge
```

---

# 10. Remove Nth Node From End

Pattern:

```text
Dummy + fixed gap between fast and slow
```

```python
dummy = ListNode(0, head)

slow = dummy
fast = head

while n > 0:
    fast = fast.next
    n -= 1

while fast:
    slow = slow.next
    fast = fast.next

slow.next = slow.next.next

return dummy.next
```

Mental model:

```text
Move fast n steps ahead
Move fast and slow together
When fast ends, slow is before target
Delete slow.next
```

Deletion:

```python
slow.next = slow.next.next
```

---

# 11. Remove Nth Node From Beginning

If `n` is 1-based:

```python
if n == 1:
    return head.next

cur = head
count = 1

while cur and count < n - 1:
    cur = cur.next
    count += 1

cur.next = cur.next.next

return head
```

General rule:

```text
To delete a node, stand at the node BEFORE it.
```

Then:

```python
prev.next = prev.next.next
```

---

# 12. Merge Two Sorted Lists

```python
dummy = ListNode()
tail = dummy

while list1 and list2:
    if list1.val < list2.val:
        tail.next = list1
        list1 = list1.next
    else:
        tail.next = list2
        list2 = list2.next

    tail = tail.next

if list1:
    tail.next = list1
elif list2:
    tail.next = list2

return dummy.next
```

Mental model:

```text
Compare heads
Attach smaller node
Move that source pointer
Move tail
```

---

# 13. Copy List With Random Pointer

Main pattern:

```text
Pass 1 = create copied nodes
Pass 2 = connect copied nodes
```

Initialize:

```python
hash_map = {None: None}
```

### First Pass

```python
cur = head

while cur:
    hash_map[cur] = Node(cur.val)
    cur = cur.next
```

Mapping becomes:

```text
old A -> new A
old B -> new B
old C -> new C
```

### Second Pass

```python
cur = head

while cur:
    hash_map[cur].next = hash_map[cur.next]
    hash_map[cur].random = hash_map[cur.random]
    cur = cur.next
```

Return:

```python
return hash_map[head]
```

---

# 14. ListNode() vs Node()

Normal linked list:

```python
ListNode(val, next)
```

Example:

```python
ListNode(0, head)
```

creates:

```text
new node:
val = 0
next = head
```

Random-pointer problem:

```python
Node(val, next, random)
```

Example:

```python
Node(cur.val)
```

creates a new independent node with:

```text
same value
next = None
random = None
```

---

# 15. Pointer Assignment vs Rewiring

This:

```python
cur = cur.next
```

only moves the variable `cur`.

It does NOT modify the list.

This:

```python
cur.next = prev
```

changes the actual linked-list connection.

So remember:

```text
cur = ...        -> move pointer variable

cur.next = ...   -> modify linked list
```

---

# 16. Basic Node Deletion

Suppose:

```text
prev -> curr -> next
```

Delete `curr`:

```python
prev.next = curr.next
```

The chain becomes:

```text
prev -> next
```

---

# 17. Why We Save `next`

Wrong:

```python
cur.next = prev
cur = cur.next
```

After reversing:

```python
cur.next = prev
```

`cur.next` no longer points forward.

Correct:

```python
nxt = cur.next
cur.next = prev
prev = cur
cur = nxt
```

---

# 18. Common Dummy Mistake

This:

```python
dummy = ListNode(0, None)
```

creates:

```text
dummy -> None
```

It is NOT connected to the existing list.

For deletion problems where dummy should sit before head:

```python
dummy = ListNode(0, head)
```

creates:

```text
dummy -> head -> ...
```

---

# 19. Why `dummy.next` Returns the Whole List

Suppose:

```text
dummy -> 1 -> 2 -> 3
```

Then:

```python
dummy.next
```

points to node `1`.

Node `1` itself points to node `2`, which points to node `3`.

Therefore returning:

```python
return dummy.next
```

returns the entry point to:

```text
1 -> 2 -> 3
```

`.next` gives the next node, and that node gives access to the entire remaining chain.

---

# 20. Problems Covered So Far

## Reverse Linked List

```text
Pattern:
Reverse pointers

Main variables:
prev, cur, nxt
```

## Merge Two Sorted Lists

```text
Pattern:
Dummy + tail
```

## Reorder List

```text
Pattern:
Middle + reverse second half + merge
```

## Remove Nth Node From End

```text
Pattern:
Dummy + fixed fast/slow gap
```

## Copy List With Random Pointer

```text
Pattern:
HashMap old node -> copied node
```

## Basic Node Deletion

```text
Pattern:
Move to node before target
Rewire .next
```

---

# 21. One-Line Pattern Recall

```text
Traverse list
-> cur = cur.next

Build new list
-> dummy + tail

Head may change/delete
-> dummy = ListNode(0, head)

Reverse list
-> prev, cur, nxt

Find middle
-> slow + fast

Reorder list
-> middle + reverse second half + merge

Nth from end
-> fixed fast/slow gap

Delete node
-> prev.next = prev.next.next

Copy random list
-> hashmap old node -> copied node

Return real head after dummy
-> dummy.next
```

---

# 22. Core Python Templates To Memorize

## Traversal

```python
cur = head

while cur:
    cur = cur.next
```

## Reverse

```python
prev = None
cur = head

while cur:
    nxt = cur.next
    cur.next = prev
    prev = cur
    cur = nxt
```

## Fast / Slow

```python
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

## Dummy + Tail

```python
dummy = ListNode()
tail = dummy

while ...:
    tail.next = node
    tail = tail.next

return dummy.next
```

## Dummy Before Head

```python
dummy = ListNode(0, head)
```

## Delete Next Node

```python
cur.next = cur.next.next
```

## Fixed Gap

```python
slow = dummy
fast = head

for _ in range(n):
    fast = fast.next

while fast:
    slow = slow.next
    fast = fast.next
```

---

# Final Mental Map

```text
HEAD MAY CHANGE?
    |
   Yes
    |
  Dummy

BUILDING A RESULT LIST?
    |
   Yes
    |
Dummy + Tail

NEED MIDDLE?
    |
   Yes
    |
Slow + Fast

NEED REVERSE?
    |
   Yes
    |
prev + cur + nxt

NTH FROM END?
    |
   Yes
    |
Fast/Slow Fixed Gap

COPY RANDOM POINTERS?
    |
   Yes
    |
HashMap:
Old Node -> New Node

REORDER?
    |
   Yes
    |
Middle
  ↓
Reverse Second Half
  ↓
Merge
```
