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
