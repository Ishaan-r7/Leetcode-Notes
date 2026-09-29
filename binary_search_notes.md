# Binary Search — Python Revision Notes

Core idea: cut the search space in half each step. Needs a **sorted array**, OR a "monotonic" answer space (true/false flips exactly once as you scan possible answers) — that second case is "binary search on the answer."

```text
Sorted array, find a value        -> classic binary search
Rotated sorted array               -> find which half is sorted, decide which side to search
"Minimize/maximize X such that condition holds" -> binary search on the ANSWER range
```

---

## Binary Search (classic template)
```python
def search(self, nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = (l + r) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            l = mid + 1
        else:
            r = mid - 1
    return -1
```
Recall: `l <= r` (not `<`) since a single-element range is still valid to check. `mid = (l + r) // 2` — watch overflow in other languages, not an issue in Python.

## Search a 2D Matrix
```python
def searchMatrix(self, matrix, target):
    rows, cols = len(matrix), len(matrix[0])
    l, r = 0, rows * cols - 1
    while l <= r:
        mid = (l + r) // 2
        row, col = mid // cols, mid % cols
        val = matrix[row][col]
        if val == target:
            return True
        elif val < target:
            l = mid + 1
        else:
            r = mid - 1
    return False
```
Recall: treat the 2D grid as one flattened 1D array — `mid // cols` and `mid % cols` convert the flat index back to (row, col).

## Koko Eating Bananas (binary search on the answer)
```python
def minEatingSpeed(self, piles, h):
    l, r = 1, max(piles)
    res = r
    while l <= r:
        speed = (l + r) // 2
        hours = sum(ceil(pile / speed) for pile in piles)
        if hours <= h:
            res = speed
            r = speed - 1     # this speed works -> try slower (smaller answer)
        else:
            l = speed + 1     # too slow -> need to go faster
    return res
```
Recall: not searching a given array — searching the RANGE of possible speeds `[1, max(piles)]`. "Can she finish in time at this speed?" is a yes/no check that flips exactly once as speed increases — that monotonic flip is what makes binary search valid here.

## Find Minimum in Rotated Sorted Array
```python
def findMin(self, nums):
    l, r = 0, len(nums) - 1
    while l < r:
        mid = (l + r) // 2
        if nums[mid] > nums[r]:
            l = mid + 1        # min is in the right half
        else:
            r = mid            # min is mid or to its left
    return nums[l]
```
Recall: compare `nums[mid]` to `nums[r]` (not `nums[l]`) — tells you which half contains the "break point" (rotation point), which is where the minimum lives.

## Search in Rotated Sorted Array
```python
def search(self, nums, target):
    l, r = 0, len(nums) - 1
    while l <= r:
        mid = (l + r) // 2
        if nums[mid] == target:
            return mid
        if nums[l] <= nums[mid]:                       # left half is sorted
            if nums[l] <= target < nums[mid]:
                r = mid - 1
            else:
                l = mid + 1
        else:                                           # right half is sorted
            if nums[mid] < target <= nums[r]:
                l = mid + 1
            else:
                r = mid - 1
    return -1
```
Recall: one half is ALWAYS properly sorted (compare `nums[l]` vs `nums[mid]` to find which). Check if target falls inside that sorted half's range — if so search there, else search the other half.

## Time Based Key-Value Store
```python
class TimeMap:
    def __init__(self):
        self.store = defaultdict(list)   # key -> [(timestamp, value), ...] sorted by timestamp

    def set(self, key, value, timestamp):
        self.store[key].append((timestamp, value))

    def get(self, key, timestamp):
        arr = self.store[key]
        l, r = 0, len(arr) - 1
        res = ""
        while l <= r:
            mid = (l + r) // 2
            if arr[mid][0] <= timestamp:
                res = arr[mid][1]        # valid candidate -> keep looking for something later
                l = mid + 1
            else:
                r = mid - 1
        return res
```
Recall: timestamps are appended in increasing order, so the list is already sorted — binary search for the LARGEST timestamp `<= target`, not an exact match.

## Median of Two Sorted Arrays
```python
def findMedianSortedArrays(self, nums1, nums2):
    A, B = nums1, nums2
    if len(A) > len(B):
        A, B = B, A                      # binary search on the SHORTER array
    total = len(A) + len(B)
    half = total // 2
    l, r = 0, len(A) - 1
    while True:
        i = (l + r) // 2 if len(A) > 0 else -1        # partition index in A
        j = half - i - 2                               # partition index in B

        Aleft = A[i] if i >= 0 else float('-inf')
        Aright = A[i + 1] if i + 1 < len(A) else float('inf')
        Bleft = B[j] if j >= 0 else float('-inf')
        Bright = B[j + 1] if j + 1 < len(B) else float('inf')

        if Aleft <= Bright and Bleft <= Aright:          # correct partition found
            if total % 2:
                return min(Aright, Bright)
            return (max(Aleft, Bleft) + min(Aright, Bright)) / 2
        elif Aleft > Bright:
            r = i - 1
        else:
            l = i + 1
```
Recall: binary search for a PARTITION point (not a value) such that everything to the left of the combined partition is ≤ everything to the right. Always search the shorter array to keep it O(log(min(m,n))).

---

## One-Line Pattern Recall
```text
Classic search              -> l <= r, mid = (l+r)//2
2D matrix                    -> flatten index: row = mid//cols, col = mid%cols
Binary search on answer      -> search a RANGE of possible answers, not the array itself;
                                 works when feasibility flips monotonically (true...true|false...false)
Min in rotated array         -> compare nums[mid] vs nums[r] to find which half has the break
Search rotated array         -> find which half is sorted, check if target's in that range
Time-based lookup            -> find largest timestamp <= target, not exact match
Median of two arrays          -> binary search a PARTITION index, always on the shorter array
```
