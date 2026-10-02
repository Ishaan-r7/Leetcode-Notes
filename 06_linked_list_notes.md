# Linked List Patterns — Python Revision Notes

# 1. Core Linked List Basics

A singly linked-list node:

```python
node.val
node.next
```

Random-pointer node:

```python
node.val
node.next
node.random
```

Important:

```text
.next points to the next NODE, not just the next value.
```

Traversal:

```python
cur = head

while cur:
    # process cur
    cur = cur.next
```

Rule:

```text
cur = cur.next      -> move pointer
cur.next = something -> modify the list
```

---

# 2. Dummy Node Patterns

## A. Dummy Before Existing Head

Use when the head may be deleted or changed.

```python
dummy = ListNode(0, head)
```

Structure:

```text
dummy -> head -> ...
```

Return:

```python
return dummy.next
```

Used in:

```text
Remove Nth Node From End
Deletion problems
```

---

## B. Dummy + Tail — Build a Result List

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
dummy = remembers start
tail  = grows the result
```

### Existing Node vs New Node

Already have a node:

```python
tail.next = list1
```

Only have a value:

```python
tail.next = ListNode(value)
```

Rule:

```text
Existing node -> attach it
Raw value     -> create ListNode(value)
```

---

# 3. Reverse Linked List Pattern

## Iterative

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

Memorize:

```text
save next
reverse link
move prev
move cur
```

Key bug:

```python
nxt = cur.next
```

must happen before changing:

```python
cur.next
```

---

## Recursive

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

Complexity (applies to both iterative and recursive):

```text
Time:  O(n) -- single pass over all n nodes either way, each node's pointer flipped once
Space: O(1) iterative -- only prev/cur/nxt pointers needed
       O(n) recursive -- call stack depth equals list length, since each call waits for the
       full chain to unwind before relinking head.next.next
```

---

# 4. Fast / Slow Pointer Patterns

Basic template:

```python
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

Use for:

```text
Find middle
Split list
Cycle detection
Reorder list
Palindrome
```

---

## Fixed Gap — Nth From End

```python
dummy = ListNode(0, head)

slow = dummy
fast = head

for _ in range(n):
    fast = fast.next

while fast:
    slow = slow.next
    fast = fast.next

slow.next = slow.next.next

return dummy.next
```

Mental model:

```text
fast stays n nodes ahead
slow stops before target
```

Complexity:

```text
Time:  O(n) -- fast advances n steps up front, then slow and fast move together through
       the remaining (length - n) nodes; total steps bounded by the list length
Space: O(1) -- only dummy, slow, fast pointers needed
```

---

# 5. Merge Two Sorted Lists

Pattern:

```text
Dummy + Tail
```

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

Recall:

```text
Compare heads
Attach smaller
Move source pointer
Move tail
```

Complexity:

```text
Time:  O(n + m) -- every node from both list1 (n nodes) and list2 (m nodes) is visited
       and attached exactly once
Space: O(1) extra -- reuses existing nodes instead of allocating new ones, only dummy/tail needed
```

---

# 6. Add Two Numbers

Pattern:

```text
Dummy + Tail + Carry
```

Core:

```python
val1 = l1.val if l1 else 0
val2 = l2.val if l2 else 0

summ = val1 + val2 + carry

digit = summ % 10
carry = summ // 10
```

Build result:

```python
tail.next = ListNode(digit)
tail = tail.next
```

Full pattern:

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

Complexity:

```text
Time:  O(max(n, m)) -- the loop runs once per digit position across both lists, extended
       by at most one extra iteration if a final carry remains
Space: O(max(n, m)) -- for the output list; O(1) extra beyond that since carry is a scalar
```

---

# 7. Reorder List

Pattern:

```text
Find Middle
-> Split
-> Reverse Second Half
-> Merge Alternately
```

```python
slow, fast = head, head.next

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next

second = slow.next
slow.next = None

prev = None

while second:
    nxt = second.next
    second.next = prev
    prev = second
    second = nxt

first, second = head, prev

while second:
    nxt1 = first.next
    nxt2 = second.next

    first.next = second
    second.next = nxt1

    first = nxt1
    second = nxt2
```

Important:

```python
slow.next = None
```

This separates both halves and prevents cycles.

Complexity:

```text
Time:  O(n) -- three linear passes (find middle, reverse second half, merge alternately),
       each bounded by n
Space: O(1) -- reversal and merge happen in place on existing nodes, no extra data structure
```

---

# 8. Copy List With Random Pointer

Pattern:

```text
HashMap:
Old Node -> Copied Node
```

Initialize:

```python
hash_map = {None: None}
```

## Pass 1 — Create Copies

```python
cur = head

while cur:
    hash_map[cur] = Node(cur.val)
    cur = cur.next
```

Mapping:

```text
old A -> new A
old B -> new B
old C -> new C
```

## Pass 2 — Connect Copies

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

Recall:

```text
Pass 1 = create nodes
Pass 2 = recreate relationships
```

Complexity:

```text
Time:  O(n) -- two separate linear passes over the list: one to create all copies,
       one to wire up next/random pointers via the hash_map lookups (O(1) each)
Space: O(n) -- hash_map stores one entry per original node mapping it to its copy
```

---

# 9. LRU Cache

Pattern:

```text
HashMap + Doubly Linked List
```

Goal:

```text
get -> O(1)
put -> O(1)
```

Structure:

```text
head <-> LRU ... MRU <-> tail
```

So:

```text
head.next = LRU
tail.prev = MRU
```

Node:

```python
class ListNode:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None
```

Initialize:

```python
self.cache = {}

self.head = ListNode(0, 0)
self.tail = ListNode(0, 0)

self.head.next = self.tail
self.tail.prev = self.head
```

HashMap:

```text
key -> node
```

---

## Remove Node

```python
def remove(self, node):
    node.prev.next = node.next
    node.next.prev = node.prev
```

---

## Add as MRU

```python
def add(self, node):
    prev_node = self.tail.prev

    prev_node.next = node
    node.prev = prev_node

    node.next = self.tail
    self.tail.prev = node
```

---

## get()

```python
def get(self, key):
    if key not in self.cache:
        return -1

    node = self.cache[key]

    self.remove(node)
    self.add(node)

    return node.value
```

---

## put()

```python
def put(self, key, value):
    if key in self.cache:
        self.remove(self.cache[key])

    node = ListNode(key, value)
    self.cache[key] = node
    self.add(node)

    if len(self.cache) > self.cap:
        lru = self.head.next

        self.remove(lru)
        del self.cache[lru.key]
```

Recall:

```text
get
-> lookup
-> move to MRU

put
-> remove old if needed
-> add as MRU
-> evict head.next if over capacity
```

Complexity:

```text
Time:  O(1) per call -- get/put each do one dict lookup plus a fixed number of pointer
       relinks (remove + add), never scanning the list
Space: O(capacity) -- cache dict and doubly linked list both bounded by the cache's max size
```

`self.` rule:

```text
self.cache / self.head / self.tail
-> belongs to LRUCache object

node / prev_node / lru
-> temporary local variables
```

---

# 10. Merge K Sorted Lists

Reuse the normal two-list merge helper.

## Sequential Idea

```text
L1 + L2
result + L3
result + L4
...
```

Works, but not optimal.

---

## Divide and Conquer

Merge pairs:

```text
[L1, L2, L3, L4]

Round 1:
L1 + L2 -> A
L3 + L4 -> B

Round 2:
A + B -> Final
```

Code:

```python
def mergeKLists(self, lists):
    if not lists:
        return None

    while len(lists) > 1:
        merged = []

        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i + 1] if i + 1 < len(lists) else None

            merged.append(self.merge(l1, l2))

        lists = merged

    return lists[0]
```

Helper:

```python
def merge(self, l1, l2):
    dummy = ListNode()
    tail = dummy

    while l1 and l2:
        if l1.val < l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next

        tail = tail.next

    if l1:
        tail.next = l1
    elif l2:
        tail.next = l2

    return dummy.next
```

Why:

```python
lists = merged
```

Because the output of the current round becomes the input to the next round.

Complexity:

```text
N = total nodes
k = number of lists

Time = O(N log k)
```

How this bound is derived:

```text
Each round pairs up lists and merges them -> number of rounds = log2(k), since the
list count halves each round (4 lists -> 2 -> 1)
Each round does O(N) total work across all its pairwise merges, since every node
gets touched exactly once per round regardless of how many pairs it's split across
Total: O(N) work per round x O(log k) rounds = O(N log k)

Space: O(1) extra beyond input/output -- merging relinks existing nodes rather than
allocating new ones; the `merged` list itself holds O(k) pointers per round
```

---

# 11. Basic Deletion / Pointer Rules

Delete next node:

```python
cur.next = cur.next.next
```

General:

```text
Stand before the target node.
```

Example:

```text
prev -> curr -> next
```

Delete `curr`:

```python
prev.next = curr.next
```

---

# 12. Common Linked List Bugs

## Losing the Rest of the List

Wrong:

```python
cur.next = prev
cur = cur.next
```

Correct:

```python
nxt = cur.next
cur.next = prev
prev = cur
cur = nxt
```

---

## Dummy Not Connected

```python
dummy = ListNode(0, None)
```

means:

```text
dummy -> None
```

For dummy-before-head:

```python
dummy = ListNode(0, head)
```

---

## Why Return `dummy.next`?

If:

```text
dummy -> 1 -> 2 -> 3
```

then:

```python
dummy.next
```

points to node `1`, which gives access to:

```text
1 -> 2 -> 3
```

So:

```python
return dummy.next
```

returns the real list without the fake dummy node.

---

# 13. One-Line Pattern Recall

```text
Traverse
-> cur = cur.next

Build result list
-> dummy + tail

Head may change
-> dummy = ListNode(0, head)

Existing node
-> tail.next = node

Only value
-> tail.next = ListNode(value)

Reverse
-> prev, cur, nxt

Middle / split
-> slow + fast

Nth from end
-> fixed fast/slow gap

Merge Two Lists
-> dummy + tail

Add Two Numbers
-> dummy + tail + carry

Reorder
-> middle + reverse + merge

Copy Random List
-> hashmap old node -> copied node

LRU
-> hashmap + doubly linked list

Merge K Lists
-> pairwise merge + divide and conquer
```

---

# 14. Core Templates To Memorize

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

tail.next = node
tail = tail.next
```

## Dummy Before Head

```python
dummy = ListNode(0, head)
```

## Fixed Gap

```python
for _ in range(n):
    fast = fast.next

while fast:
    slow = slow.next
    fast = fast.next
```

## DLL Remove

```python
node.prev.next = node.next
node.next.prev = node.prev
```

## Add Before Tail

```python
prev_node = tail.prev

prev_node.next = node
node.prev = prev_node

node.next = tail
tail.prev = node
```

---

# Final Mental Map

```text
HEAD MAY CHANGE?
-> Dummy before head

BUILDING RESULT?
-> Dummy + Tail

NEED REVERSE?
-> prev + cur + nxt

NEED MIDDLE?
-> Slow + Fast

NTH FROM END?
-> Fixed Gap

COPY RANDOM POINTERS?
-> HashMap Old -> New

REORDER?
-> Middle + Reverse + Merge

LRU?
-> HashMap + Doubly Linked List

MERGE K LISTS?
-> Pairwise Merge + Divide and Conquer
```
