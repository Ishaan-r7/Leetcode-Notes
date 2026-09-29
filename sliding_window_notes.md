# Sliding Window — Python Revision Notes

Core idea: maintain a window `[l, r]` over the array/string. Expand `r` to grow it, shrink `l` when a constraint is violated. Avoids recomputing from scratch — O(n) instead of O(n²).

```text
Fixed-size window (size given)     -> r - l + 1 == k, slide both together
Variable-size window (condition)    -> expand r; while condition broken: shrink l
```

---

## Best Time to Buy and Sell Stock
```python
def maxProfit(self, prices):
    l, r = 0, 1
    maxP = 0
    while r < len(prices):
        if prices[l] < prices[r]:
            maxP = max(maxP, prices[r] - prices[l])
        else:
            l = r                 # found a new lower buy point -> reset window start
        r += 1
    return maxP
```
Recall: `l` = best buy day so far. If price drops below it, jump `l` to that new low — no need to track it separately.

## Longest Substring Without Repeating Characters
```python
def lengthOfLongestSubstring(self, s):
    seen = set()
    l = 0
    res = 0
    for r in range(len(s)):
        while s[r] in seen:
            seen.remove(s[l])
            l += 1
        seen.add(s[r])
        res = max(res, r - l + 1)
    return res
```
Recall: `while` (not `if`) to shrink — duplicate might require removing several chars from the left before it's gone.

## Longest Repeating Character Replacement
```python
def characterReplacement(self, s, k):
    count = {}
    l = 0
    maxFreq = 0
    res = 0
    for r in range(len(s)):
        count[s[r]] = count.get(s[r], 0) + 1
        maxFreq = max(maxFreq, count[s[r]])
        if (r - l + 1) - maxFreq > k:      # window - most frequent char > k replacements
            count[s[l]] -= 1
            l += 1
        res = max(res, r - l + 1)
    return res
```
Recall: window is valid if `(window size - count of most frequent char) <= k` — that gap is exactly how many chars you'd need to replace. `maxFreq` doesn't need correcting on shrink; it never causes a wrong answer here, only a delayed one.

## Permutation in String
```python
def checkInclusion(self, s1, s2):
    need = Counter(s1)
    window = Counter()
    l = 0
    for r in range(len(s2)):
        window[s2[r]] += 1
        if r - l + 1 > len(s1):
            window[s2[l]] -= 1
            if window[s2[l]] == 0:
                del window[s2[l]]
            l += 1
        if window == need:
            return True
    return False
```
Recall: FIXED window size = `len(s1)`. Slide it across `s2`, compare frequency maps.

## Minimum Window Substring
```python
def minWindow(self, s, t):
    if not t:
        return ""
    need = Counter(t)
    have, needCount = 0, len(need)
    window = {}
    res, resLen = [-1, -1], float('inf')
    l = 0
    for r in range(len(s)):
        c = s[r]
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            have += 1
        while have == needCount:                   # valid window -> try to shrink
            if (r - l + 1) < resLen:
                res = [l, r]
                resLen = r - l + 1
            window[s[l]] -= 1
            if s[l] in need and window[s[l]] < need[s[l]]:
                have -= 1
            l += 1
    l, r = res
    return s[l:r+1] if resLen != float('inf') else ""
```
Recall: `have == needCount` means the window currently satisfies every required character count — shrink greedily from the left while it stays valid, recording the smallest valid window found.

## Sliding Window Maximum
```python
def maxSlidingWindow(self, nums, k):
    dq = deque()   # stores INDICES, values kept decreasing left to right
    res = []
    l = 0
    for r in range(len(nums)):
        while dq and nums[dq[-1]] < nums[r]:
            dq.pop()                     # remove smaller values -- they can never be the max again
        dq.append(r)
        if dq[0] < l:                    # index fell out of window
            dq.popleft()
        if r - l + 1 >= k:
            res.append(nums[dq[0]])
            l += 1
    return res
```
Recall: monotonic decreasing deque — the max of the window is always at the front (`dq[0]`). Anything smaller than the newest value entering is useless and gets popped, since the new value will outlast it and is bigger.

---

## One-Line Pattern Recall
```text
Fixed-size window       -> slide l and r together, compare when r-l+1 == k
Variable window          -> expand r; while condition broken: shrink l
Buy/sell stock            -> l = lowest price seen so far; reset on new low
Longest no-repeat         -> set + while-shrink until duplicate gone
Char replacement window   -> valid if (window - maxFreq) <= k
Anagram/permutation match -> fixed window + Counter comparison
Min window w/ all chars   -> have == needCount -> shrink greedily, track min length
Window max                -> monotonic decreasing deque of INDICES
```
