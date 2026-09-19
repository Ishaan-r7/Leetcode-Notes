# Merge K Sorted Lists — Divide and Conquer

## Main Idea

We already know how to merge **two sorted linked lists**.

For `k` lists, instead of merging them one-by-one:

```text
L1 + L2
result + L3
result + L4
...
```

merge them in pairs:

```text
[L1, L2, L3, L4]

Round 1:
L1 + L2 -> A
L3 + L4 -> B

[A, B]

Round 2:
A + B -> Final
```

Pattern:

```text
Merge lists in pairs
-> replace old lists with merged results
-> repeat until only one list remains
```

---

## Why `lists = merged`?

`merged` contains the results of the **current round**.

Example:

```text
lists = [L1, L2, L3, L4]
```

After one round:

```text
merged = [merge(L1, L2), merge(L3, L4)]
```

So:

```python
lists = merged
```

makes the next round operate on:

```text
[A, B]
```

Then:

```text
merged = [merge(A, B)]
```

Eventually:

```text
lists = [Final]
```

so:

```python
return lists[0]
```

returns the fully merged list.

---

## Divide-and-Conquer Code

```python
def mergeKLists(self, lists):
    if not lists:
        return None

    while len(lists) > 1:
        merged = []

        for i in range(0, len(lists), 2):
            l1 = lists[i]

            if i + 1 < len(lists):
                l2 = lists[i + 1]
            else:
                l2 = None

            merged.append(self.merge(l1, l2))

        lists = merged

    return lists[0]
```

The helper is the same **Merge Two Sorted Lists** pattern:

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

---

## Sequential vs Divide and Conquer

### Sequential

```text
L1 + L2
result + L3
result + L4
...
```

The merged list keeps getting larger every time.

### Divide and Conquer

```text
Round 1: merge small lists
Round 2: merge medium lists
Round 3: merge larger lists
```

Number of rounds:

```text
log(k)
```

where `k` is the number of linked lists.

---

## Complexity

Let:

```text
N = total number of nodes
k = number of lists
```

Then:

```text
Time: O(N log k)
```

because every node participates in about `log(k)` merge rounds.

---

## Quick Recall

```text
Merge K Sorted Lists
-> Merge Two Sorted Lists repeatedly in pairs

Pair adjacent lists
-> store results in merged

lists = merged
-> use current round as input to next round

Stop when
-> len(lists) == 1

Return
-> lists[0]
```
