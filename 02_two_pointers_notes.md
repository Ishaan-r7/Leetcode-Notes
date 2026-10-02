# Two Pointers — Python Revision Notes

Core idea: instead of nested loops (O(n²)), use two pointers moving toward each other (or together) to scan in O(n). Usually needs a **sorted array**, or you sort first.

```text
Opposite ends, converging inward    -> l, r = 0, len(arr)-1; move based on comparison
Same direction, one fast one slow    -> different pattern (see Sliding Window notes)
```

---

## Valid Palindrome
```python
def isPalindrome(self, s):
    s = [c.lower() for c in s if c.isalnum()]
    l, r = 0, len(s) - 1
    while l < r:
        if s[l] != s[r]:
            return False
        l += 1
        r -= 1
    return True
```
Recall: clean/normalize first, then converge pointers from both ends.

```text
Time:  O(n) -- building the cleaned list is O(n), then l and r converge toward each other
       covering at most n more steps combined
Space: O(n) -- the filtered/lowercased character list holds up to n elements
```

## Two Sum II (sorted input)
```python
def twoSum(self, numbers, target):
    l, r = 0, len(numbers) - 1
    while l < r:
        total = numbers[l] + numbers[r]
        if total == target:
            return [l + 1, r + 1]
        elif total < target:
            l += 1          # need a bigger sum -> move left pointer up
        else:
            r -= 1          # need a smaller sum -> move right pointer down
```
Recall: sorted array lets you REASON about direction — too small means increase `l`, too big means decrease `r`.

```text
Time:  O(n) -- l only moves right, r only moves left, so combined they take at most n steps total
Space: O(1) -- only two index variables are needed, no extra structure
```

## 3Sum
```python
def threeSum(self, nums):
    nums.sort()
    res = []
    for i in range(len(nums)):
        if i > 0 and nums[i] == nums[i - 1]:
            continue                              # skip duplicate anchors
        l, r = i + 1, len(nums) - 1
        while l < r:
            total = nums[i] + nums[l] + nums[r]
            if total < 0:
                l += 1
            elif total > 0:
                r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                while nums[l] == nums[l - 1] and l < r:   # skip duplicate l's
                    l += 1
    return res
```
Recall: fix one number (`i`), two-pointer the rest. Sort first — enables both the direction logic AND easy duplicate-skipping.

```text
Time:  O(n^2) -- the sort is O(n log n), dominated by the outer loop over i (O(n)) each running
       an O(n) two-pointer sweep on the remainder
Space: O(n) -- ignoring the output, this is sort's space (or recursion depth) overhead
```

## Container With Most Water
```python
def maxArea(self, height):
    l, r = 0, len(height) - 1
    res = 0
    while l < r:
        area = (r - l) * min(height[l], height[r])
        res = max(res, area)
        if height[l] < height[r]:
            l += 1          # the SHORTER wall is the bottleneck -> move it
        else:
            r -= 1
    return res
```
Recall: always move the pointer at the SHORTER wall — moving the taller one can only shrink the area (width shrinks, height still capped by the short side).

```text
Time:  O(n) -- l and r converge toward each other, combined covering at most n steps
Space: O(1) -- only the two pointers and a running max are tracked
```

## Trapping Rain Water
```python
def trap(self, height):
    l, r = 0, len(height) - 1
    leftMax, rightMax = height[l], height[r]
    res = 0
    while l < r:
        if leftMax < rightMax:
            l += 1
            leftMax = max(leftMax, height[l])
            res += leftMax - height[l]
        else:
            r -= 1
            rightMax = max(rightMax, height[r])
            res += rightMax - height[r]
    return res
```
Recall: water trapped at a point = `min(leftMax, rightMax) - height[point]`. Move the pointer on the SMALLER max side — that side's bound is already known/fixed, so it's safe to resolve now.

```text
Time:  O(n) -- l and r converge inward, each step resolves water at exactly one position
Space: O(1) -- leftMax/rightMax are tracked as running scalars instead of precomputed arrays
```

---

## One-Line Pattern Recall
```text
Palindrome check         -> l/r converge, compare, stop when mismatch
Sorted target sum        -> sum too small -> l+=1, too big -> r-=1
3Sum                     -> sort + fix one + two-pointer the rest, skip dupes
Max container area        -> move pointer at the SHORTER wall
Trapping rain water        -> move pointer on SMALLER max side, water = min(max)-height
```
