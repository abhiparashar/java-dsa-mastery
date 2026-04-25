# 09 — Heaps (Priority Queues)

Heaps give you O(log n) insert/remove and O(1) access to the min or max. In interviews, heaps appear in four patterns: Top-K selection, two-heap balancing, merging K sorted structures, and lazy deletion. Java's `PriorityQueue` is a min-heap by default — every max-heap needs an explicit comparator.

---

## Table of Contents
1. [Top-K Pattern](#1-top-k-pattern)
2. [Two Heaps](#2-two-heaps)
3. [Merge K Sorted](#3-merge-k-sorted)
4. [Lazy Deletion / Scheduled Removal](#4-lazy-deletion--scheduled-removal)

---

## Java PriorityQueue Essentials

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap — NEVER use (a, b) -> b - a (integer overflow risk)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Integer::compare).reversed();
// OR: Collections.reverseOrder()
PriorityQueue<Integer> maxHeap2 = new PriorityQueue<>(Collections.reverseOrder());

// Custom comparator for int[] tuples (sort by first element)
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));

// Key operations and complexities:
// offer(e)  — O(log n)
// poll()    — O(log n)
// peek()    — O(1)
// remove(e) — O(n) ← this is why lazy deletion exists
// size()    — O(1)
```

**Critical Java gotcha:** `PriorityQueue.remove(Object)` is O(n), not O(log n). This makes naive "remove then re-add" approaches TLE on large inputs. Use lazy deletion instead (Pattern 4).

---

## 1. Top-K Pattern

### When to Use
- "**K-th largest/smallest**" element
- "**Top K frequent**" elements
- "**K closest points**"
- Any time you need the K best/worst out of N elements

### When NOT to Use
- K is close to N — just sort in O(n log n)
- You need the exact median — use Two Heaps instead
- Single min/max — don't need a heap, just track with a variable

### Optimized Java Template

```java
// Find K-th largest: use a MIN-heap of size K
// Intuition: the min of the K largest elements IS the K-th largest
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();  // evict smallest — only K largest survive
        }
    }
    return minHeap.peek();  // smallest of K largest = K-th largest
}

// Top K Frequent: bucket sort approach — O(n) instead of O(n log k)
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    // Bucket sort: index = frequency, value = list of numbers with that frequency
    @SuppressWarnings("unchecked")
    List<Integer>[] buckets = new List[nums.length + 1];
    for (var entry : freq.entrySet()) {
        int f = entry.getValue();
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(entry.getKey());
    }

    int[] result = new int[k];
    int idx = 0;
    for (int i = buckets.length - 1; i >= 0 && idx < k; i--) {
        if (buckets[i] != null) {
            for (int num : buckets[i]) {
                result[idx++] = num;
                if (idx == k) break;
            }
        }
    }
    return result;
}
```

**Why min-heap for K-th largest?** Counter-intuitive but correct: maintaining a min-heap of size K means the smallest element in the heap IS the K-th largest overall. A max-heap would require storing all N elements.

### Fully Solved Problem: K Closest Points to Origin (LC 973)

**Problem:** Given an array of points, return the K closest to the origin.

**Thinking process:**
1. "K closest" → Top-K pattern
2. We want the K **smallest** distances → use a **max-heap** of size K
3. The max-heap evicts the farthest point, keeping only the K closest
4. No need for `Math.sqrt` — compare squared distances (monotonic transformation)

```java
public int[][] kClosest(int[][] points, int k) {
    // Max-heap by distance (largest distance at top → gets evicted)
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> Integer.compare(dist(b), dist(a))
    );

    for (int[] p : points) {
        maxHeap.offer(p);
        if (maxHeap.size() > k) {
            maxHeap.poll();  // evict farthest
        }
    }

    return maxHeap.toArray(new int[k][]);
}

private int dist(int[] p) {
    return p[0] * p[0] + p[1] * p[1];  // no sqrt needed
}
```

**Complexity:** O(n log k) time, O(k) space.

**Optimal alternative:** Quickselect gives O(n) average but O(n²) worst case. Interviewers may ask about this tradeoff — heap is safer for interviews.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Max-heap for K-th largest | Stores all N elements, O(n log n) | Use min-heap of size K: O(n log k) |
| `(a, b) -> a[0] - b[0]` comparator | Integer overflow when values are near `Integer.MAX_VALUE` | `Integer.compare(a[0], b[0])` |
| Using `Math.sqrt` for distances | Unnecessary floating-point, slower | Compare squared distances |
| Forgetting to handle K > N | `ArrayIndexOutOfBoundsException` | Check edge case or let heap handle naturally |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Can you do better than O(n log k)?" | "Quickselect gives O(n) average, O(n²) worst. For guaranteed bounds, use Introselect or the median-of-medians." |
| "What if N is huge and doesn't fit in memory?" | "Stream through data with a size-K heap — only O(k) memory at any time." |
| "Why not just sort?" | "Sorting is O(n log n). Heap approach is O(n log k). When k << n, the heap wins significantly." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Kth Largest Element in an Array | Medium | Min-heap of size K | 215 |
| 2 | Top K Frequent Elements | Medium | Bucket sort O(n) or heap O(n log k) | 347 |
| 3 | K Closest Points to Origin | Medium | Max-heap of size K, squared distances | 973 |
| 4 | Sort Characters By Frequency | Medium | Frequency map + bucket sort | 451 |
| 5 | Kth Largest Element in a Stream | Easy | Maintain min-heap of size K across calls | 703 |
| 6 | Reorganize String | Medium | Max-heap by frequency, alternate placement | 767 |

---

## 2. Two Heaps

### When to Use
- "**Find median**" from data stream
- "**Sliding window median**"
- Balance two halves: max-heap for lower half, min-heap for upper half

### When NOT to Use
- Static array median — just sort or use quickselect
- Only need min or max — single heap suffices

### Optimized Java Template

```java
// Two-heap median finder
class MedianFinder {
    private PriorityQueue<Integer> lo;  // max-heap: lower half
    private PriorityQueue<Integer> hi;  // min-heap: upper half

    public MedianFinder() {
        lo = new PriorityQueue<>(Collections.reverseOrder());  // max-heap
        hi = new PriorityQueue<>();  // min-heap
    }

    public void addNum(int num) {
        lo.offer(num);
        hi.offer(lo.poll());      // balance: push max of lower to upper

        if (hi.size() > lo.size()) {
            lo.offer(hi.poll());  // keep lo.size >= hi.size
        }
    }

    public double findMedian() {
        if (lo.size() > hi.size()) {
            return lo.peek();
        }
        return (lo.peek() + hi.peek()) / 2.0;
    }
}
```

**Invariant:** `lo.size()` is always equal to or one more than `hi.size()`. All elements in `lo` ≤ all elements in `hi`. The median is either `lo.peek()` (odd count) or the average of both peeks (even count).

### Fully Solved Problem: Find Median from Data Stream (LC 295)

**Problem:** Design a data structure that supports `addNum(int num)` and `findMedian()` efficiently.

**Thinking process:**
1. We need O(1) median access → need the middle element(s) readily available
2. Split into two halves: lower half (max-heap) and upper half (min-heap)
3. The tops of both heaps are the two middle elements
4. Keep sizes balanced: `|lo.size() - hi.size()| <= 1`

The template above IS the solution. Key insight for the `addNum` flow:
1. Always add to `lo` first
2. Move `lo`'s max to `hi` (ensures `lo.max <= hi.min`)
3. If `hi` grows larger, move `hi`'s min back to `lo`

This 3-step dance guarantees both the size invariant and the ordering invariant in every case.

```java
// Trace with [5, 2, 8, 1]:
// add(5): lo=[5], hi=[]       → median = 5
// add(2): lo=[2], hi=[5]      → median = (2+5)/2 = 3.5
// add(8): lo=[5,2], hi=[8]    → median = 5
// add(1): lo=[2,1], hi=[5,8]  → median = (2+5)/2 = 3.5
```

**Complexity:** `addNum`: O(log n), `findMedian`: O(1).

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Adding to wrong heap first | Ordering invariant breaks | Always: add to lo → move max to hi → rebalance sizes |
| Integer division for median | `(lo.peek() + hi.peek()) / 2` truncates | Use `/ 2.0` for double result |
| Overflow with large integers | `lo.peek() + hi.peek()` overflows | Cast: `((long)lo.peek() + hi.peek()) / 2.0` |
| Sliding window: not removing expired | Heaps don't support efficient removal | Use lazy deletion (Pattern 4) |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "What about sliding window median?" | "Same two-heap approach but with lazy deletion for expired elements. Track balance separately from actual sizes." |
| "Can you do O(1) addNum?" | "Not with heaps. A self-balancing BST (like `TreeMap` with duplicates) can give O(log n). True O(1) isn't possible for ordered insertion." |
| "What if there are many duplicates?" | "Heaps handle duplicates naturally. Alternatively, use a `TreeMap<Integer, Integer>` (value → count) to save space." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Find Median from Data Stream | Hard | Two heaps: max-heap lower + min-heap upper | 295 |
| 2 | Sliding Window Median | Hard | Two heaps + lazy deletion for expired elements | 480 |
| 3 | IPO | Hard | Max-heap by profit, min-heap by capital | 502 |

---

## 3. Merge K Sorted

### When to Use
- "**Merge K sorted** lists/arrays"
- "**Smallest range** covering elements from K lists"
- "Find K-th element across multiple sorted sources"
- Multiple sorted streams that need unified ordering

### When NOT to Use
- Only 2 sorted arrays — merge directly in O(n + m)
- Unsorted input — sort first, then merge is pointless

### Optimized Java Template

```java
// Merge K sorted lists using min-heap
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)
    );

    for (ListNode head : lists) {
        if (head != null) pq.offer(head);
    }

    ListNode dummy = new ListNode(0);
    ListNode tail = dummy;

    while (!pq.isEmpty()) {
        ListNode min = pq.poll();
        tail.next = min;
        tail = tail.next;
        if (min.next != null) {
            pq.offer(min.next);
        }
    }

    return dummy.next;
}

// Merge K sorted arrays (same idea with index tracking)
public List<Integer> mergeKArrays(int[][] arrays) {
    // pq stores: [value, arrayIndex, elementIndex]
    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );

    for (int i = 0; i < arrays.length; i++) {
        if (arrays[i].length > 0) {
            pq.offer(new int[]{arrays[i][0], i, 0});
        }
    }

    List<Integer> result = new ArrayList<>();
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        result.add(curr[0]);
        int arrIdx = curr[1], eleIdx = curr[2];
        if (eleIdx + 1 < arrays[arrIdx].length) {
            pq.offer(new int[]{arrays[arrIdx][eleIdx + 1], arrIdx, eleIdx + 1});
        }
    }
    return result;
}
```

### Fully Solved Problem: Smallest Range Covering Elements from K Lists (LC 632)

**Problem:** Given K sorted lists, find the smallest range [a, b] such that at least one number from each list is included.

**Thinking process:**
1. We need one element from each list — like a sliding window across K lists
2. Keep a min-heap with one element from each list
3. Track the current max across all elements in the heap
4. The range is [heap.min, currentMax]. Advance the min to try to shrink the range
5. If any list is exhausted, we can't cover all K lists → stop

```java
public int[] smallestRange(List<List<Integer>> nums) {
    // pq stores: [value, listIndex, elementIndex]
    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])
    );

    int curMax = Integer.MIN_VALUE;
    for (int i = 0; i < nums.size(); i++) {
        int val = nums.get(i).get(0);
        pq.offer(new int[]{val, i, 0});
        curMax = Math.max(curMax, val);
    }

    int[] best = {pq.peek()[0], curMax};

    while (true) {
        int[] min = pq.poll();
        int listIdx = min[1], eleIdx = min[2];

        if (eleIdx + 1 >= nums.get(listIdx).size()) break;  // list exhausted

        int nextVal = nums.get(listIdx).get(eleIdx + 1);
        pq.offer(new int[]{nextVal, listIdx, eleIdx + 1});
        curMax = Math.max(curMax, nextVal);

        if (curMax - pq.peek()[0] < best[1] - best[0]) {
            best = new int[]{pq.peek()[0], curMax};
        }
    }

    return best;
}
```

**Complexity:** O(n log k) where n = total elements across all lists, k = number of lists.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not handling null/empty lists | `NullPointerException` | Filter out null/empty before adding to PQ |
| Comparing ListNode directly | Java can't compare objects without Comparator | Always provide explicit comparator |
| Forgetting to advance pointer | Infinite loop — same element re-added | Always move to `.next` or `eleIdx + 1` |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Time complexity of merge K lists?" | "O(N log K) where N = total nodes. Each of the N nodes enters/exits the size-K heap once." |
| "Can you merge without extra space?" | "Divide and conquer: merge pairs recursively. O(N log K) time, O(log K) stack space. Same complexity, avoids the heap." |
| "What if lists are unsorted?" | "Sort each first in O(n log n), then merge. Or just concatenate and sort everything: O(N log N)." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Merge K Sorted Lists | Hard | Min-heap with one node per list | 23 |
| 2 | Smallest Range Covering Elements from K Lists | Hard | Min-heap + track max, advance min pointer | 632 |
| 3 | Kth Smallest Element in a Sorted Matrix | Medium | Min-heap or binary search on value range | 378 |
| 4 | Find K Pairs with Smallest Sums | Medium | Min-heap with (nums1[i] + nums2[j], i, j) | 373 |
| 5 | Ugly Number II | Medium | Min-heap with factors {2, 3, 5}, dedup | 264 |

---

## 4. Lazy Deletion / Scheduled Removal

### When to Use
- "**Sliding window** with heap" (elements expire but PQ has no efficient remove)
- Any heap problem where you need to remove specific elements
- `PriorityQueue.remove(Object)` is O(n) — too slow for large inputs

### When NOT to Use
- You can avoid removal entirely (e.g., by using a different data structure like `TreeMap`)
- Small input sizes where O(n) removal is acceptable

### Optimized Java Template

```java
// Lazy deletion pattern: don't physically remove, just skip stale entries
class LazyHeap {
    PriorityQueue<int[]> pq;  // [value, index]
    Map<Integer, Integer> removeCount;  // value -> pending removals

    public LazyHeap(Comparator<int[]> comp) {
        pq = new PriorityQueue<>(comp);
        removeCount = new HashMap<>();
    }

    public void add(int[] entry) {
        pq.offer(entry);
    }

    public void lazyRemove(int val) {
        removeCount.merge(val, 1, Integer::sum);
    }

    public int[] peek() {
        purge();
        return pq.peek();
    }

    public int[] poll() {
        purge();
        return pq.poll();
    }

    private void purge() {
        while (!pq.isEmpty()) {
            int val = pq.peek()[0];
            Integer cnt = removeCount.get(val);
            if (cnt != null && cnt > 0) {
                pq.poll();
                removeCount.put(val, cnt - 1);
            } else {
                break;
            }
        }
    }

    public int size() {
        // Note: this returns physical size; for logical size, track separately
        return pq.size();
    }
}
```

### Fully Solved Problem: Sliding Window Median (LC 480)

**Problem:** Given an array and window size k, return the median of each window.

**Thinking process:**
1. Median → Two Heaps (Pattern 2)
2. Sliding window → need to remove expired elements → Lazy Deletion (this pattern)
3. Combine: two heaps with lazy deletion, track logical balance separately

```java
public double[] medianSlidingWindow(int[] nums, int k) {
    // Max-heap for lower half, min-heap for upper half
    PriorityQueue<Long> lo = new PriorityQueue<>(Collections.reverseOrder());
    PriorityQueue<Long> hi = new PriorityQueue<>();
    Map<Long, Integer> removed = new HashMap<>();

    double[] result = new double[nums.length - k + 1];
    int balance = 0;  // lo.logicalSize - hi.logicalSize

    // Initialize first window
    for (int i = 0; i < k; i++) lo.offer((long) nums[i]);
    for (int i = 0; i < k / 2; i++) hi.offer(lo.poll());
    balance = k - k / 2 - k / 2;  // = 0 if even, 1 if odd

    for (int i = 0; i <= nums.length - k; i++) {
        // Record median
        if (k % 2 == 1) {
            result[i] = lo.peek();
        } else {
            result[i] = ((double) lo.peek() + hi.peek()) / 2.0;
        }

        if (i == nums.length - k) break;

        long outgoing = nums[i];          // element leaving window
        long incoming = nums[i + k];      // element entering window

        // Lazy-remove outgoing
        removed.merge(outgoing, 1, Integer::sum);
        if (outgoing <= lo.peek()) {
            balance--;  // logically removed from lo
        } else {
            balance++;  // logically removed from hi
        }

        // Add incoming
        if (!lo.isEmpty() && incoming <= lo.peek()) {
            lo.offer(incoming);
            balance++;
        } else {
            hi.offer(incoming);
            balance--;
        }

        // Rebalance: balance should be 0 (even k) or 1 (odd k)
        if (balance > 1) {
            hi.offer(lo.poll());
            balance -= 2;
        } else if (balance < 0) {  // for even: balance < 0; for odd: balance < 1 doesn't happen correctly
            lo.offer(hi.poll());
            balance += 2;
        }

        // Purge stale tops
        purge(lo, removed);
        purge(hi, removed);
    }

    return result;
}

private void purge(PriorityQueue<Long> pq, Map<Long, Integer> removed) {
    while (!pq.isEmpty()) {
        long top = pq.peek();
        Integer cnt = removed.get(top);
        if (cnt != null && cnt > 0) {
            pq.poll();
            removed.put(top, cnt - 1);
        } else {
            break;
        }
    }
}
```

**Why `long`?** When computing `lo.peek() + hi.peek()`, two large `int` values can overflow. Using `long` prevents this.

**Complexity:** O(n log n) time (each element added/removed from heap at most twice), O(n) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using `PriorityQueue.remove()` directly | O(n) per removal → O(nk) total, TLE | Lazy deletion: mark removed, skip on peek/poll |
| Not purging after rebalance | Stale elements at heap tops corrupt median | Always purge both heaps after each window slide |
| Integer overflow in median calculation | `(a + b) / 2` overflows | Use `long` or `a + (b - a) / 2.0` |
| Balance tracking wrong | Off-by-one errors in median | Draw out a 4-5 element example and trace manually |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Can we use TreeMap instead of lazy deletion?" | "Yes — `TreeMap` with counts gives O(log n) add/remove. Cleaner code but same complexity." |
| "Why not just rebuild the heap each window?" | "That's O(k) per window → O(nk) total. Lazy deletion amortizes to O(n log n)." |
| "What if k = 1?" | "Every element is its own median. Handle as edge case to avoid heap operations." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Sliding Window Median | Hard | Two heaps + lazy deletion | 480 |
| 2 | Sliding Window Maximum | Hard | Monotonic deque (see [Stacks](./03-stacks-and-queues.md#4-monotonic-queue)) | 239 |
| 3 | The Skyline Problem | Hard | Events + max-heap with lazy deletion | 218 |

---

## Heap vs Alternatives Decision Framework

| Scenario | Best Choice | Why |
|----------|-------------|-----|
| K-th largest/smallest | Min/Max heap of size K | O(n log k) time, O(k) space |
| Streaming median | Two heaps | O(log n) add, O(1) median |
| Merge K sorted | Min-heap of size K | O(N log K), minimal memory |
| Sliding window max/min | Monotonic deque | O(n) total — better than heap's O(n log n) |
| Need ordered iteration | `TreeMap` | Sorted iteration + O(log n) operations |
| Exact K-th in static array | Quickselect | O(n) average vs heap's O(n log k) |

---

## Cross-References

- **Sliding Window Maximum** uses monotonic deque, not heap — see [Stacks & Queues](./03-stacks-and-queues.md#4-monotonic-queue)
- **Dijkstra's algorithm** uses a min-heap — see [Graphs](./06-graphs.md)
- **Top-K Frequent** can use bucket sort — connects to [Arrays & Hashing](./01-arrays-and-hashing.md)
- **Merge K Sorted Lists** also solvable with divide-and-conquer — see [Linked Lists](./02-linked-lists.md#3-merge)
