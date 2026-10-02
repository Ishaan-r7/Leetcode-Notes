# Heap / Priority Queue — Python Revision Notes

Core idea: `heapq` is just a regular `list` you promise to only touch via `heapq.heappush`/`heappop` — Python only gives you a min-heap (smallest at `heap[0]`). To simulate a max-heap, negate values going in and negate back coming out.

```text
Need the SMALLEST available element repeatedly      -> plain min-heap
Need the LARGEST available element repeatedly        -> negate values (max-heap trick)
Need "k-th largest/smallest" without full sort        -> fixed-size heap of size k
Need elements to come back later, in order            -> heap + separate structure for "not ready yet" (Task Scheduler)
```

## How to reason about heap complexity (don't just memorize — derive it)

```text
heappush / heappop       -> O(log n)   -- heap height is log n (complete binary tree), each op bubbles up/down the height
heapify(existing list)    -> O(n)       -- NOT O(n log n); lower levels dominate and need few swaps each (memorize this one)
Building heap n items ONE AT A TIME (n x heappush) -> O(n log n) total
Building heap from ALL n items upfront (heapify once) -> O(n) total  <- faster when you have everything in hand already

Rule of thumb for TOTAL complexity: count how many push/pop operations the
algorithm does overall, then multiply by O(log k) where k = current heap size
at that point (often k stays capped, e.g. "keep only k items" problems ->
O(n log k) total instead of O(n log n), which matters a lot when k << n).
```

## Kth Largest Element in a Stream

```python
class KthLargest:
    def __init__(self, k, nums):
        self.k = k
        self.heap = nums
        heapq.heapify(self.heap)
        while len(self.heap) > k:
            heapq.heappop(self.heap)

    def add(self, val):
        heapq.heappush(self.heap, val)
        if len(self.heap) > self.k:
            heapq.heappop(self.heap)
        return self.heap[0]
```

Recall: maintain a MIN-heap of size k holding the k largest seen so far — the smallest of those k (`heap[0]`) IS the k-th largest.

```text
Time:  __init__ O(n log n) worst case (n pushes/pops down to k); add O(log k) per call
Space: O(k)
```

## Last Stone Weight

```python
def lastStoneWeight(self, stones):
    heap = [-s for s in stones]
    heapq.heapify(heap)
    while len(heap) > 1:
        x, y = -heapq.heappop(heap), -heapq.heappop(heap)
        if x != y:
            heapq.heappush(heap, -(x - y))
    return -heap[0] if heap else 0
```

Recall: negate on the way IN and on the way OUT (max-heap trick). `x`, `y` are the two heaviest.

```text
Time:  O(n log n) -- up to n pops/pushes, each O(log n)
Space: O(n) for the heap
```

## K Closest Points to Origin

```python
def kClosest(self, points, k):
    heap = []
    for x, y in points:
        dist = x*x + y*y
        heapq.heappush(heap, (-dist, x, y))   # max-heap of size k via negation
        if len(heap) > k:
            heapq.heappop(heap)                # evict the FARTHEST point
    return [[x, y] for _, x, y in heap]
```

Recall: same "fixed-size heap of size k" pattern as Kth Largest in a Stream, but here you maintain a MAX-heap (via negation) of size k so the FARTHEST of the k closest ends up at the top — easy to evict when a closer point arrives. Sort by negative distance since tuples compare element-by-element.

```text
Time:  O(n log k) -- n points, each push/pop is O(log k) since heap capped at size k (NOT O(n log n) -- this is the payoff of capping heap size)
Space: O(k)
```

## Kth Largest Element in an Array

```python
def findKthLargest(self, nums, k):
    heap = [-n for n in nums]
    heapq.heapify(heap)
    for _ in range(k - 1):
        heapq.heappop(heap)
    return -heap[0]
```

Recall: negate, heapify once (O(n)), pop k-1 times to discard the top k-1 largest, what's left on top is the k-th largest.

```text
Time:  O(n + k log n) -- heapify is O(n), then k-1 pops at O(log n) each
Space: O(n)
```

## Task Scheduler

```python
def leastInterval(self, tasks, n):
    count = Counter(tasks)
    maxHeap = [-c for c in count.values()]
    heapq.heapify(maxHeap)

    queue = deque()   # [count, time_it_can_return]
    time = 0
    while maxHeap or queue:
        time += 1
        if maxHeap:
            cnt = 1 + heapq.heappop(maxHeap)   # one use consumed (still negative or 0)
            if cnt != 0:
                queue.append([cnt, time + n])
        if queue and queue[0][1] == time:
            heapq.heappush(maxHeap, queue.popleft()[0])
    return time
```

Recall: max-heap always gives the most-remaining task to run next (greedy). `queue` holds tasks "cooling down" until `time + n` — re-add to heap once their cooldown expires.

```text
Time:  O(n_tasks) -- heap size capped at 26 (letters), so push/pop are effectively O(log 26) = O(1); dominant cost is iterating `time` steps
Space: O(1) -- at most 26 distinct task types
```

## Design Twitter

```python
class Twitter:
    def __init__(self):
        self.count = 0                          # global "timestamp", decreasing = more recent
        self.tweetMap = defaultdict(list)        # userId -> [(count, tweetId), ...]
        self.followMap = defaultdict(set)         # userId -> set(followeeId)

    def postTweet(self, userId, tweetId):
        self.tweetMap[userId].append((self.count, tweetId))
        self.count -= 1

    def getNewsFeed(self, userId):
        res = []
        heap = []
        self.followMap[userId].add(userId)        # user sees their own tweets too
        for followeeId in self.followMap[userId]:
            if followeeId in self.tweetMap:
                index = len(self.tweetMap[followeeId]) - 1
                count, tweetId = self.tweetMap[followeeId][index]
                heapq.heappush(heap, (count, tweetId, followeeId, index - 1))

        while heap and len(res) < 10:
            count, tweetId, followeeId, index = heapq.heappop(heap)
            res.append(tweetId)
            if index >= 0:
                nextCount, nextTweetId = self.tweetMap[followeeId][index]
                heapq.heappush(heap, (nextCount, nextTweetId, followeeId, index - 1))

        return res

    def follow(self, followerId, followeeId):
        self.followMap[followerId].add(followeeId)

    def unfollow(self, followerId, followeeId):
        self.followMap[followerId].discard(followeeId)
```

Recall: merge-k-sorted-lists pattern — one heap entry per followee pointing at their MOST RECENT unseen tweet; popping the global-most-recent and pushing that followee's next tweet (like the deque/heap index-advance trick) keeps it O(log(followees)) per pop instead of sorting everything.

```text
Time:  getNewsFeed: O(F log F) where F = number of followees (heap never exceeds F elements)
Space: O(F) for the heap, O(total tweets) for storage
```

## Find Median from Data Stream

```python
class MedianFinder:
    def __init__(self):
        self.maxHeap, self.minHeap = [], []    # maxHeap = smaller half (negated), minHeap = larger half

    def addNum(self, num):
        heapq.heappush(self.maxHeap, -num)
        if self.maxHeap and self.minHeap and -self.maxHeap[0] >= self.minHeap[0]:
            heapq.heappush(self.minHeap, -heapq.heappop(self.maxHeap))
        if len(self.maxHeap) > len(self.minHeap) + 1:
            heapq.heappush(self.minHeap, -heapq.heappop(self.maxHeap))
        if len(self.maxHeap) < len(self.minHeap):
            heapq.heappush(self.maxHeap, -heapq.heappop(self.minHeap))

    def findMedian(self):
        if len(self.maxHeap) > len(self.minHeap):
            return -self.maxHeap[0]
        if len(self.maxHeap) < len(self.minHeap):
            return self.minHeap[0]
        return (-self.maxHeap[0] + self.minHeap[0]) / 2
```

Recall: two heaps split the stream in half. `maxHeap` (smaller half, negated) can have at most 1 more element than `minHeap` (larger half). Median is either the bigger half's root, or the average of both roots when sizes match. Always negate back before comparing `maxHeap[0]` to `minHeap[0]` — they're on different scales otherwise.

```text
Time:  addNum O(log n) per call; findMedian O(1)
Space: O(n) across both heaps
```

## One-Line Pattern Recall

```text
Fixed-size heap of size k         -> push then pop-if-over-k; top of heap = kth extreme
Max-heap simulation                -> negate IN, negate OUT
Merge-k-sorted (Design Twitter)     -> one heap entry per source, push next element on pop
Two-heap median split               -> maxHeap (smaller half) size >= minHeap (larger half) size, diff <= 1
Greedy "most remaining" scheduling  -> max-heap + cooldown queue (Task Scheduler)
```
