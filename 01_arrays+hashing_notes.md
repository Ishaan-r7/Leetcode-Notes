# Arrays & Hashing — Python Revision Notes

Core idea: trade space for speed. A hashset/hashmap turns O(n) lookups into O(1).

```text
Need to check "have I seen this before"?      -> set()
Need to count/group things?                   -> dict() (often defaultdict or Counter)
Need index of a value, fast?                   -> dict: value -> index
```

---

## Contains Duplicate
```python
def hasDuplicate(self, nums):
    return len(nums) != len(set(nums))
```
Recall: set dedupes automatically — compare lengths instead of looping.

## Two Sum
```python
def twoSum(self, nums, target):
    seen = {}   # val -> index
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i
```
Recall: check for the COMPLEMENT before inserting current — avoids using the same element twice.

## Valid Anagram / Group Anagrams
```python
def isAnagram(self, s, t):
    return Counter(s) == Counter(t)

def groupAnagrams(self, strs):
    groups = defaultdict(list)
    for s in strs:
        key = ''.join(sorted(s))     # or a 26-length char-count tuple (faster)
        groups[key].append(s)
    return list(groups.values())
```
Recall: sorted string (or char-count signature) is the same for every anagram — use it as the hashmap key to bucket them.

## Top K Frequent Elements
```python
def topKFrequent(self, nums, k):
    count = Counter(nums)
    return [val for val, _ in count.most_common(k)]
```
Recall: `Counter.most_common(k)` — no need to hand-roll a heap unless asked to avoid built-ins. (Bucket sort by frequency is the O(n) manual approach if needed.)

## Product of Array Except Self
```python
def productExceptSelf(self, nums):
    n = len(nums)
    res = [1] * n
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]
    postfix = 1
    for i in range(n - 1, -1, -1):
        res[i] *= postfix
        postfix *= nums[i]
    return res
```
Recall: two passes — prefix product left→right, then multiply in postfix product right→left. No division needed (handles zeros safely).

## Longest Consecutive Sequence
```python
def longestConsecutive(self, nums):
    numSet = set(nums)
    longest = 0
    for n in numSet:
        if n - 1 not in numSet:          # only start counting from sequence STARTS
            length = 1
            while n + length in numSet:
                length += 1
            longest = max(longest, length)
    return longest
```
Recall: the `n - 1 not in numSet` check is what keeps this O(n) instead of O(n²) — only starts a count from the beginning of each run, never the middle.

---

## One-Line Pattern Recall
```text
Duplicate check           -> set(), compare lengths
Two Sum (single pass)      -> hashmap value->index, check complement before inserting
Anagram grouping           -> sorted string / char-count tuple as hashmap key
Top K frequent              -> Counter.most_common(k), or bucket sort by frequency
Prefix/suffix product      -> two passes, multiply running product in from both sides
Longest run in unsorted    -> set() + only start counting from run STARTS (n-1 not in set)
```

---

# Python Syntax Cheatsheet

## Sorting a dict by its values
```python
d = {'a': 3, 'b': 1, 'c': 2}

sorted(d, key=d.get)                      # ['b', 'c', 'a']  -- sorted KEYS, ordered by their value
sorted(d.items(), key=lambda x: x[1])     # [('b',1), ('c',2), ('a',3)]  -- (key, value) pairs
sorted(d, key=d.get, reverse=True)        # descending by value
```
Recall: `key=d.get` means "for each key, sort using `d[key]` instead of the key itself." `d.items()` version if you need both key AND value in the result, not just the key.

## Sorting a list of tuples/lists by a specific field
```python
pairs = [(3,'c'), (1,'a'), (2,'b')]
sorted(pairs)                   # sorts by first element by default: [(1,'a'),(2,'b'),(3,'c')]
sorted(pairs, key=lambda x: x[1])   # sort by second element instead
sorted(pairs, key=lambda x: -x[0])   # descending, without reverse=True (useful with multi-key sorts)
```

## Sorting by multiple keys
```python
sorted(people, key=lambda p: (p.age, p.name))   # age first, name as tiebreaker
sorted(people, key=lambda p: (-p.age, p.name))  # age descending, name ascending -- mixed directions
```

## Counter — common operations
```python
c = Counter(['a','b','a','c','a'])
c['a']                 # 3
c['z']                 # 0  -- missing key doesn't KeyError, just returns 0 (unlike a plain dict)
c.most_common(2)        # [('a',3), ('b',1)]  -- top k by frequency, already sorted
c.most_common()          # ALL elements, sorted by frequency descending
c1 - c2                  # subtract counts (keeps only positive results)
c1 + c2                  # add counts together
```

## defaultdict — avoids manual key-existence checks
```python
d = defaultdict(list)
d['x'].append(1)        # no KeyError even though 'x' was never set -- auto-creates []

d = defaultdict(int)
d['x'] += 1               # auto-creates 0 first, so this works immediately
```

## enumerate — index + value together
```python
for i, val in enumerate(nums):      # i = index, val = nums[i]
    ...
for i, val in enumerate(nums, start=1):   # start counting from 1 instead of 0
    ...
```

## zip — pair up multiple lists
```python
for a, b in zip(list1, list2):      # stops at the SHORTER list's length
    ...
dict(zip(keys, values))              # build a dict directly from two parallel lists
```

## List/dict comprehension filters
```python
[x for x in nums if x > 0]                    # filter
[x*2 for x in nums]                           # transform
{k: v for k, v in d.items() if v > 1}          # filter a dict
{x for x in nums}                              # set comprehension
```

## `sorted()` vs `.sort()`
```python
sorted(nums)     # returns a NEW list, original untouched
nums.sort()        # sorts IN PLACE, returns None -- don't do `nums = nums.sort()`
```
