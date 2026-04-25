# 04 — Binary Search

Binary search is deceptively simple — the concept takes 5 minutes to learn but the off-by-one bugs take a career to master. This guide gives you **three battle-tested templates** that cover every variant. Pick the right one and you'll never have an off-by-one again.

---

## Table of Contents
1. [Standard Binary Search + Bisect Templates](#1-standard-binary-search--bisect-templates)
2. [Binary Search on Answer (Parametric Search)](#2-binary-search-on-answer-parametric-search)
3. [Rotated / Modified Sorted Array](#3-rotated--modified-sorted-array)
4. [2D Binary Search](#4-2d-binary-search)
5. [Median of Two Sorted Arrays](#5-median-of-two-sorted-arrays)

---

## 1. Standard Binary Search + Bisect Templates

### When to Use (Recognition Signals)
- Array is **sorted** (or has a monotonic property)
- "Find element", "find position", "search insert position"
- Need O(log n) time
- "First occurrence" or "last occurrence" of a value

### When NOT to Use
- Array is unsorted and can't be sorted → linear scan
- You need to find ALL occurrences → binary search finds one, then expand
- The search space isn't monotonic → binary search needs a clear partition

### The Three Templates

**The fundamental insight:** every binary search problem boils down to: "find the boundary where a predicate changes from false to true (or vice versa)."

```java
// Template 1: Exact Match
// Use when: find a specific target value
// Loop invariant: target, if present, is in [lo, hi]
public int binarySearch(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;  // prevents overflow

        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }

    return -1;  // not found
}

// Template 2: Lower Bound (first index where nums[i] >= target)
// Use when: first occurrence, search insert position, leftmost satisfying element
// Loop invariant: answer is in [lo, hi)
public int lowerBound(int[] nums, int target) {
    int lo = 0, hi = nums.length;  // hi = length, NOT length - 1

    while (lo < hi) {  // NOT lo <= hi
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] < target) {
            lo = mid + 1;     // mid is too small, exclude it
        } else {
            hi = mid;          // mid might be the answer, keep it
        }
    }

    return lo;  // first index where nums[lo] >= target
}

// Template 3: Upper Bound (first index where nums[i] > target)
// Use when: last occurrence (upper_bound - 1), count of target values
public int upperBound(int[] nums, int target) {
    int lo = 0, hi = nums.length;

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] <= target) {
            lo = mid + 1;     // mid is <= target, we want strictly greater
        } else {
            hi = mid;
        }
    }

    return lo;  // first index where nums[lo] > target
}

// Derived: count of target = upperBound(target) - lowerBound(target)
// Derived: last occurrence of target = upperBound(target) - 1 (check if valid)
```

**Why `lo + (hi - lo) / 2` instead of `(lo + hi) / 2`:**
- `lo + hi` can overflow when both are large positive integers (e.g., both near `Integer.MAX_VALUE / 2`)
- `lo + (hi - lo) / 2` computes the difference first (which is always non-negative and small), then adds. Safe.

**When to use `lo <= hi` vs `lo < hi`:**
- Template 1 (`lo <= hi`): searching for an exact value. The loop needs to check the single remaining element when `lo == hi`.
- Templates 2 & 3 (`lo < hi`): searching for a boundary. When `lo == hi`, we've found the boundary — no more checking needed.

### Fully Solved Problem: Find First and Last Position in Sorted Array (LC 34)

**Problem:** Given a sorted array and a target, find the first and last index of the target. Return `[-1, -1]` if not found.

**Thinking Process:**
1. First position = lower_bound(target)
2. Last position = upper_bound(target) - 1
3. Validate that the positions actually contain the target

**Solution:**
```java
public int[] searchRange(int[] nums, int target) {
    int first = lowerBound(nums, target);
    int last = upperBound(nums, target) - 1;

    if (first < nums.length && nums[first] == target) {
        return new int[]{first, last};
    }
    return new int[]{-1, -1};
}
```

### Complexity Analysis
- **Time:** O(log n) for each search
- **Space:** O(1)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| `(lo + hi) / 2` overflow | mid becomes negative for large arrays | Use `lo + (hi - lo) / 2` |
| Infinite loop when `lo + 1 == hi` | With `lo <= hi` template, if you set `hi = mid` (not `mid - 1`), mid = lo forever | Template 1: always use `mid ± 1`. Templates 2/3: use `lo < hi` which terminates correctly |
| Wrong initial `hi` value | Template 1: `hi = length - 1`. Templates 2/3: `hi = length` (answer could be past the array) | Match `hi` to the template you're using |
| Off-by-one for "last occurrence" | `upperBound - 1` might be -1 if target doesn't exist | Always validate: check `nums[result] == target` |
| Searching for a value not in the array | Lower bound returns the insertion position, not -1 | Check the returned index: `if (lo < n && nums[lo] == target)` |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Which template would you use for X?" | "I choose based on what I'm looking for: exact match → Template 1, first >= → Template 2, first > → Template 3. Never mix templates mid-problem." |
| "Can you prove there's no off-by-one?" | "State the loop invariant: 'the answer is in [lo, hi)'. Each branch maintains it. When lo == hi, the range has one point — that's the answer." |
| "Why not just use `Arrays.binarySearch()`?" | "It finds ANY occurrence, not first/last. For first/last position, I need lower/upper bound. Also, `Arrays.binarySearch()` returns `-(insertion point) - 1` for missing values, which is error-prone." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Binary Search | Easy | Template 1: exact match | 704 |
| 2 | Search Insert Position | Easy | Template 2: lower bound | 35 |
| 3 | First Bad Version | Easy | Template 2: first version where isBadVersion is true | 278 |
| 4 | Sqrt(x) | Easy | Template 2: first n where n*n > x, return n-1 | 69 |
| 5 | Find First and Last Position | Medium | Lower bound + upper bound | 34 |
| 6 | Find Peak Element | Medium | Binary search on non-sorted: move toward increasing side | 162 |
| 7 | Single Element in Sorted Array | Medium | Binary search on parity of index pairs | 540 |

---

## 2. Binary Search on Answer (Parametric Search)

### When to Use (Recognition Signals)
- "**Minimize the maximum**" or "**maximize the minimum**"
- "Can you do X within Y?" — and the feasibility is monotonic
- The answer is a number in a range, and you can CHECK if a specific answer is achievable
- Keywords: "minimum capacity", "maximum distance", "minimum time", "split into K parts"

### When NOT to Use
- The feasibility function isn't monotonic (both small and large values can be feasible/infeasible)
- You need the exact configuration, not just the optimal value → may need DP

### Optimized Java Template

```java
// Binary Search on Answer Space
// predicate(x) returns true if x is feasible
// We want the MINIMUM feasible x (or MAXIMUM — adjust accordingly)
public int binarySearchOnAnswer(int[] nums, int constraint) {
    int lo = minPossibleAnswer(nums);
    int hi = maxPossibleAnswer(nums);

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;

        if (isFeasible(nums, mid, constraint)) {
            hi = mid;     // mid works, try smaller
        } else {
            lo = mid + 1; // mid doesn't work, need larger
        }
    }

    return lo;
}

// The isFeasible function is problem-specific
// Example: "Can we split nums into ≤ K groups, each with sum ≤ maxSum?"
private boolean isFeasible(int[] nums, int maxSum, int k) {
    int groups = 1, currSum = 0;
    for (int num : nums) {
        if (currSum + num > maxSum) {
            groups++;
            currSum = num;
            if (groups > k) return false;
        } else {
            currSum += num;
        }
    }
    return true;
}
```

**The mental model:**
- Imagine the answer space as a number line from `lo` to `hi`.
- There's a boundary: all values below it are infeasible (`false`), all values at or above are feasible (`true`). `FFFFFFTTTTTT`
- We binary search for the first `T` — that's the minimum feasible answer.
- For "maximize the minimum", the boundary is reversed: `TTTTTTFFFFFF` — we search for the last `T`.

### Fully Solved Problem: Koko Eating Bananas (LC 875)

**Problem:** Koko has `n` piles of bananas. She can eat at speed `k` bananas/hour (one pile per hour, even if she finishes early). Find the minimum `k` such that she finishes all piles in `h` hours.

**Thinking Process:**
1. Answer space: `k` ranges from 1 (minimum speed) to `max(piles)` (eat the biggest pile in one hour).
2. Feasibility: for a given `k`, can she finish in `h` hours? Hours needed = `sum(ceil(pile / k))` for each pile. Check if total ≤ h.
3. Monotonicity: higher `k` → fewer hours needed. The predicate "can finish in h hours" goes from `false` to `true` as `k` increases.
4. Binary search for the first `k` where it's feasible.

**Solution:**
```java
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1;
    int hi = 0;
    for (int p : piles) hi = Math.max(hi, p);

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, mid, h)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }

    return lo;
}

private boolean canFinish(int[] piles, int speed, int h) {
    long hours = 0;
    for (int pile : piles) {
        hours += (pile + speed - 1) / speed;  // ceil division without floating point
    }
    return hours <= h;
}
```

**Why `(pile + speed - 1) / speed` for ceiling division:**
- `Math.ceil((double) pile / speed)` works but uses floating point — slower and can have precision issues.
- Integer ceiling: `(a + b - 1) / b` for positive a and b. Faster and exact.

### Complexity Analysis
- **Time:** O(n log M) where M = max answer range, n = array size for feasibility check
- **Space:** O(1)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Wrong bounds for lo/hi | Missing the answer or infinite loop | lo = minimum possible answer, hi = maximum possible answer. Think carefully about both. |
| Floating point in ceiling division | Precision errors for large values | Use `(a + b - 1) / b` for integer ceiling |
| Integer overflow in feasibility check | Sum of hours can exceed int range | Use `long` for accumulator |
| Confusing "minimize maximum" vs "maximize minimum" | Wrong direction for binary search | Minimize → search for first feasible (hi = mid). Maximize → search for last feasible (lo = mid + 1 on failure). |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "How do you know the predicate is monotonic?" | "I think about it physically: more capacity → fewer trips. More speed → less time. If increasing the answer makes it 'easier', the predicate is monotonic." |
| "What if the answer is a real number, not integer?" | "Use `while (hi - lo > 1e-7)` with `double`. Or multiply by 10^k and work in integers." |
| "Can you solve Split Array Largest Sum (LC 410) with DP instead?" | "Yes, DP in O(n^2 * k) time. But binary search on answer is O(n log S) where S = sum — much faster. The DP approach is correct but slower." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Koko Eating Bananas | Medium | Solved above — minimize speed | 875 |
| 2 | Capacity to Ship Packages in D Days | Medium | Minimize max load per day | 1011 |
| 3 | Min Days to Make m Bouquets | Medium | Search on day count, check bloom feasibility | 1482 |
| 4 | Find Smallest Divisor Given a Threshold | Medium | Minimize divisor, check ceil sum ≤ threshold | 1283 |
| 5 | Split Array Largest Sum | Hard | Minimize max subarray sum for K splits | 410 |
| 6 | Minimum Time to Complete Trips | Hard | Search on time, check total trips ≥ target | 2187 |
| 7 | Magnetic Force Between Two Balls | Medium | Maximize minimum distance between balls | 1552 |

---

## 3. Rotated / Modified Sorted Array

### When to Use (Recognition Signals)
- "**Rotated sorted array**"
- "**Find minimum** in rotated array"
- "**Search** in rotated array"
- Array was sorted, then some modification was applied

### When NOT to Use
- Array isn't rotated (just sorted) → standard binary search
- Array is randomly ordered → can't use binary search

### Optimized Java Template

```java
// Template: Search in Rotated Sorted Array (no duplicates)
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) return mid;

        // Determine which half is sorted
        if (nums[lo] <= nums[mid]) {
            // Left half [lo..mid] is sorted
            if (nums[lo] <= target && target < nums[mid]) {
                hi = mid - 1;  // target is in sorted left half
            } else {
                lo = mid + 1;  // target is in right half
            }
        } else {
            // Right half [mid..hi] is sorted
            if (nums[mid] < target && target <= nums[hi]) {
                lo = mid + 1;  // target is in sorted right half
            } else {
                hi = mid - 1;  // target is in left half
            }
        }
    }

    return -1;
}

// Template: Find Minimum in Rotated Sorted Array (no duplicates)
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] > nums[hi]) {
            lo = mid + 1;  // minimum is in right half
        } else {
            hi = mid;       // mid could be the minimum
        }
    }

    return nums[lo];
}

// Template: Search in Rotated Array WITH DUPLICATES
public boolean search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) return true;

        // Handle duplicates: when nums[lo] == nums[mid] == nums[hi]
        // we can't determine which half is sorted
        if (nums[lo] == nums[mid] && nums[mid] == nums[hi]) {
            lo++;
            hi--;
            continue;
        }

        if (nums[lo] <= nums[mid]) {
            if (nums[lo] <= target && target < nums[mid]) {
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            if (nums[mid] < target && target <= nums[hi]) {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
    }

    return false;
}
```

**The key insight for rotated arrays:**
- At any `mid`, at least ONE half `[lo..mid]` or `[mid..hi]` is sorted (this is always true for a single rotation).
- Check which half is sorted using `nums[lo] <= nums[mid]`.
- If the target falls in the sorted half, narrow to it. Otherwise, go to the other half.
- **With duplicates**: when `nums[lo] == nums[mid] == nums[hi]`, we can't tell which half is sorted. Worst case: shrink both ends by 1, degrading to O(n).

### Fully Solved Problem: Find Minimum in Rotated Sorted Array (LC 153)

**Solution and walkthrough:**
```java
public int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;  // rotation point (minimum) is in right half
        } else {
            hi = mid;       // mid could be the minimum, don't exclude it
        }
    }
    return nums[lo];
}
```

**Walkthrough with `[3, 4, 5, 1, 2]`:**
```
lo=0, hi=4, mid=2: nums[2]=5 > nums[4]=2 → lo=3
lo=3, hi=4, mid=3: nums[3]=1 < nums[4]=2 → hi=3
lo=3, hi=3: exit. Return nums[3]=1. ✓
```

### Complexity Analysis
- **No duplicates:** O(log n) time, O(1) space
- **With duplicates:** O(n) worst case (e.g., `[1,1,1,1,0,1,1]`), O(log n) average

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Comparing `nums[mid]` with `nums[lo]` for findMin | When `lo == mid`, `nums[lo] == nums[mid]` is always true, giving no information | Compare with `nums[hi]` instead — more reliable |
| Duplicates causing ambiguity | `[1,3,1,1,1]`: can't tell which half contains the target | Fall back to `lo++; hi--` when `nums[lo] == nums[mid] == nums[hi]` |
| Not handling the unrotated case | Array `[1,2,3,4,5]` (rotation = 0) should still work | Both templates handle this — the "left half is sorted" branch covers it |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "What's the worst case with duplicates?" | "O(n) when all elements are the same except one. Example: `[1,1,1,0,1,1,1]`. We shrink by 1 each time." |
| "How many times was the array rotated?" | "The index of the minimum element. Use the findMin template but return the index, not the value." |
| "Can you find the rotation count without binary search?" | "If we know the original first element, yes — just find it. But typically we don't, so binary search on the structure." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Search in Rotated Sorted Array | Medium | Determine which half is sorted | 33 |
| 2 | Search in Rotated Sorted Array II | Medium | Handle duplicates with lo++/hi-- fallback | 81 |
| 3 | Find Minimum in Rotated Sorted Array | Medium | Compare mid with hi | 153 |
| 4 | Find Minimum in Rotated Sorted Array II | Hard | Duplicates: O(n) worst case | 154 |

---

## 4. 2D Binary Search

### When to Use (Recognition Signals)
- "Search a **2D matrix**" (sorted row-by-row or row-and-column sorted)
- "Kth smallest in **sorted matrix**"
- Matrix where each row is sorted and first element > last element of previous row

### When NOT to Use
- Matrix is unsorted → linear scan or hash
- You need the actual element AND its position in a row-column sorted matrix → staircase search (not binary search)

### Optimized Java Template

```java
// Template A: 2D → 1D Binary Search (each row sorted, row[i][-1] < row[i+1][0])
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int lo = 0, hi = m * n - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];  // convert 1D index to 2D

        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}

// Template B: Staircase Search (rows sorted, columns sorted independently)
// Start from top-right or bottom-left corner
public boolean searchMatrixII(int[][] matrix, int target) {
    int row = 0, col = matrix[0].length - 1;

    while (row < matrix.length && col >= 0) {
        if (matrix[row][col] == target) return true;
        else if (matrix[row][col] < target) row++;   // eliminate this row
        else col--;                                     // eliminate this column
    }
    return false;
}
```

**When to use which:**
- Template A: matrix is "fully sorted" — row-by-row concatenation is sorted. O(log(m*n)).
- Template B: matrix rows are sorted AND columns are sorted, but NOT necessarily fully sorted (e.g., `matrix[0][n-1]` could be > `matrix[1][0]`). O(m + n).

### Fully Solved Problem: Kth Smallest Element in a Sorted Matrix (LC 378)

**Problem:** Given an n×n matrix where each row and column is sorted, find the kth smallest element.

**Thinking Process:**
1. We could flatten and sort → O(n² log n). Too slow.
2. Use a min-heap: start with first column, extract-min and push next element in that row. O(k log n). Good if k is small.
3. Binary search on value: lo = matrix[0][0], hi = matrix[n-1][n-1]. For each mid, count how many elements ≤ mid using staircase search. O(n log(max-min)).

**Solution (Binary Search on Value):**
```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    int lo = matrix[0][0], hi = matrix[n - 1][n - 1];

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessOrEqual(matrix, mid);
        if (count < k) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

private int countLessOrEqual(int[][] matrix, int target) {
    int n = matrix.length;
    int count = 0;
    int row = n - 1, col = 0;  // start from bottom-left

    while (row >= 0 && col < n) {
        if (matrix[row][col] <= target) {
            count += row + 1;  // all elements in this column up to this row
            col++;
        } else {
            row--;
        }
    }
    return count;
}
```

### Complexity Analysis
- **2D → 1D search:** O(log(m*n)) time, O(1) space
- **Staircase search:** O(m + n) time, O(1) space
- **Kth smallest (BS on value):** O(n * log(max - min)) time, O(1) space

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Search a 2D Matrix | Medium | Treat as 1D sorted array: `row = mid/n, col = mid%n` | 74 |
| 2 | Search a 2D Matrix II | Medium | Staircase search from top-right | 240 |
| 3 | Kth Smallest in Sorted Matrix | Medium | Binary search on value + staircase count | 378 |
| 4 | Kth Smallest Pair Distance | Hard | Binary search on distance + counting pairs ≤ mid | 719 |

---

## 5. Median of Two Sorted Arrays

### When to Use (Recognition Signals)
- "**Median of two sorted arrays**" — this is THE classic hard binary search problem
- Any problem requiring a partition of two sorted arrays into "left" and "right" halves

### Optimized Solution

This problem deserves its own section because it's the hardest binary search problem and a Google/Meta favorite.

**Problem (LC 4):** Find the median of two sorted arrays in O(log(min(m, n))) time.

**Key Insight:** We're looking for a partition that splits the combined sorted array into two equal halves. Binary search on the partition position in the shorter array.

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    // Ensure nums1 is the shorter array
    if (nums1.length > nums2.length) {
        return findMedianSortedArrays(nums2, nums1);
    }

    int m = nums1.length, n = nums2.length;
    int lo = 0, hi = m;

    while (lo <= hi) {
        int i = lo + (hi - lo) / 2;       // partition index in nums1
        int j = (m + n + 1) / 2 - i;      // partition index in nums2

        int left1 = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
        int right1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
        int left2 = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
        int right2 = (j == n) ? Integer.MAX_VALUE : nums2[j];

        if (left1 <= right2 && left2 <= right1) {
            // Found the correct partition
            if ((m + n) % 2 == 0) {
                return (Math.max(left1, left2) + Math.min(right1, right2)) / 2.0;
            } else {
                return Math.max(left1, left2);
            }
        } else if (left1 > right2) {
            hi = i - 1;  // too many elements from nums1 on left
        } else {
            lo = i + 1;  // too few elements from nums1 on left
        }
    }

    return -1;  // unreachable if inputs are valid
}
```

**How it works:**
- We partition nums1 at index `i` and nums2 at index `j`, where `i + j = (m + n + 1) / 2` (so the left half has `ceil((m+n)/2)` elements).
- The partition is correct when: `nums1[i-1] <= nums2[j]` AND `nums2[j-1] <= nums1[i]` (left elements ≤ right elements).
- `Integer.MIN_VALUE` / `MAX_VALUE` handle boundary cases where `i` or `j` is 0 or at the array end.

### Complexity Analysis
- **Time:** O(log(min(m, n))) — binary search on the shorter array
- **Space:** O(1)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not ensuring nums1 is shorter | If m > n, `j` can become negative | Always swap so nums1 is shorter |
| Boundary conditions (i=0, i=m, j=0, j=n) | Index out of bounds | Use MIN_VALUE/MAX_VALUE sentinels |
| Integer division for even-length result | `(a + b) / 2` gives int, not double | Use `/ 2.0` for the average |
| Off-by-one in partition index j | Wrong relationship between i and j | `j = (m + n + 1) / 2 - i` — the `+1` handles odd total correctly |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Median of Two Sorted Arrays | Hard | Binary search on partition position | 4 |

---

## Cross-References

- **Binary Search on Answer** is used extensively in [Graph problems](./06-graphs.md) — e.g., "minimum effort path" (LC 1631) binary searches on effort threshold
- **Lower/Upper Bound** is fundamental for [Interval problems](./10-intervals.md) — finding overlapping intervals
- **2D search** concepts extend to [Segment Trees](./14-advanced-patterns.md) for range queries
- **Prefix Sum + Binary Search** combines [Prefix Sum](./01-arrays-and-hashing.md#4-prefix-sum) with lower bound for range counting
