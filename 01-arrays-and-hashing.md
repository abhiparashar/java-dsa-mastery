# 01 — Arrays & Hashing

The most frequently tested topic in MAANG interviews. Master these 7 patterns and you can solve ~40% of all interview problems.

---

## Table of Contents
1. [Two Pointers](#1-two-pointers)
2. [Sliding Window — Fixed Size](#2-sliding-window--fixed-size)
3. [Sliding Window — Variable Size](#3-sliding-window--variable-size)
4. [Prefix Sum](#4-prefix-sum)
5. [Kadane's Algorithm](#5-kadanes-algorithm)
6. [Hash Map / Set Operations](#6-hash-map--set-operations)
7. [Dutch National Flag / Cyclic Sort](#7-dutch-national-flag--cyclic-sort)

---

## 1. Two Pointers

### When to Use (Recognition Signals)
- Array is **sorted** (or you can sort it without violating the problem)
- You need to find **pairs/triplets** satisfying a sum condition
- You need to **partition** an array (move elements left/right of a pivot)
- Problem says "in-place" with O(1) extra space
- "Remove duplicates from sorted array"

### When NOT to Use
- Array is unsorted AND sorting would lose required index information (use HashMap instead — e.g., Two Sum LC 1 needs original indices)
- You need to find subarrays (contiguous), not pairs — use Sliding Window instead
- The condition between elements isn't monotonic — two pointers need a clear direction to move

### Optimized Java Template

```java
// Template A: Converging Pointers (sorted array, find pair with condition)
public int[] twoPointers(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int sum = nums[lo] + nums[hi];
        if (sum == target) {
            return new int[]{lo, hi};
        } else if (sum < target) {
            lo++;  // need larger sum → move left pointer right
        } else {
            hi--;  // need smaller sum → move right pointer left
        }
    }
    return new int[]{-1, -1};
}

// Template B: Same-Direction Pointers (partitioning, removing duplicates)
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int write = 1;  // slow pointer: next write position
    for (int read = 1; read < nums.length; read++) {  // fast pointer
        if (nums[read] != nums[read - 1]) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
```

**Why each line matters:**
- `lo < hi` (not `<=`): prevents using the same element twice
- `sum = nums[lo] + nums[hi]`: compute BEFORE branching to avoid redundant access
- Same-direction: `write` pointer only advances when we find a valid element — everything before `write` is the "clean" portion

### Fully Solved Problem: 3Sum (LC 15)

**Problem:** Given an array `nums`, return all unique triplets `[nums[i], nums[j], nums[k]]` such that `i != j != k` and `nums[i] + nums[j] + nums[k] == 0`.

**Thinking Process:**
1. Brute force: three nested loops → O(n^3). Too slow for n up to 3000.
2. Key insight: if we fix one element, we need two elements that sum to its negative → Two Sum on a sorted array → Two Pointers.
3. Sort first (O(n log n)), then for each element, run two pointers on the remaining array → O(n^2).
4. Tricky part: handling duplicates. We must skip duplicate values for ALL three positions.

**Solution:**
```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);

    for (int i = 0; i < nums.length - 2; i++) {
        // Skip duplicate values for first element
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        // Early termination: if smallest possible triplet > 0, no solution
        if (nums[i] > 0) break;

        int lo = i + 1, hi = nums.length - 1;
        int target = -nums[i];

        while (lo < hi) {
            int sum = nums[lo] + nums[hi];
            if (sum == target) {
                result.add(Arrays.asList(nums[i], nums[lo], nums[hi]));
                // Skip duplicates for both pointers
                while (lo < hi && nums[lo] == nums[lo + 1]) lo++;
                while (lo < hi && nums[hi] == nums[hi - 1]) hi--;
                lo++;
                hi--;
            } else if (sum < target) {
                lo++;
            } else {
                hi--;
            }
        }
    }
    return result;
}
```

**Walkthrough with `[-1, 0, 1, 2, -1, -4]`:**
1. Sort → `[-4, -1, -1, 0, 1, 2]`
2. `i=0, nums[i]=-4, target=4`: lo=1, hi=5. Max sum = -1+2 = 1 < 4. No triplet.
3. `i=1, nums[i]=-1, target=1`: lo=2, hi=5.
   - `nums[2]+nums[5] = -1+2 = 1` → Found `[-1,-1,2]`. Skip dups. lo=3, hi=4.
   - `nums[3]+nums[4] = 0+1 = 1` → Found `[-1,0,1]`. lo=4, hi=3. Stop.
4. `i=2, nums[i]=-1`: Same as `nums[1]`, skip.
5. `i=3, nums[i]=0, target=0`: lo=4, hi=5. `1+2=3 > 0`. hi--. Stop.

**Result:** `[[-1,-1,2], [-1,0,1]]`

### Complexity Analysis
- **Time:** O(n^2) — O(n log n) sort + O(n) outer loop × O(n) two pointers = O(n^2)
- **Space:** O(1) extra (ignoring output and sort space)
- Mathematical note: we can't do better than O(n^2) for 3Sum — there's a matching lower bound in the algebraic decision tree model

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not skipping duplicates | Output contains duplicate triplets like `[-1,-1,2]` twice | Skip with `if (i > 0 && nums[i] == nums[i-1]) continue` |
| Using `i >= 0` for skip condition | Skips the first element when it equals previous (index 0 has no previous) | Use `i > 0` as guard |
| Forgetting to move BOTH pointers after finding a match | Infinite loop: same pair keeps being found | Always do `lo++; hi--` after recording a match |
| Integer overflow in sum | `nums[lo] + nums[hi]` overflows if values near INT_MAX | For most LC problems, values fit in int. But in interviews, mention this! |
| Sorting when you need original indices | Two Sum (LC 1) needs indices — sorting destroys them | Use HashMap for unsorted + index problems |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you solve it without sorting?" | "Yes, using HashMap for two-sum lookup, but handling duplicates in the result becomes harder. Time is still O(n^2) but space increases to O(n). Sorting is generally preferred because duplicate handling is cleaner." |
| "Now do 4Sum" | "Same approach — fix two elements, two-pointer on remaining. O(n^3). Generalize to kSum with recursion." |
| "What if we want the closest sum to target instead of exact?" | "Track the minimum difference. Same structure, but update `bestSum` when `abs(sum - target) < abs(bestSum - target)`. LC 16." |
| "What if the array is too large to fit in memory?" | "External sort (merge sort on disk), then streaming two pointers. Or hash-partition into buckets." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Two Sum II (sorted array) | Easy | Direct converging two pointers | 167 |
| 2 | Valid Palindrome | Easy | Two pointers from both ends, skip non-alphanumeric | 125 |
| 3 | Remove Duplicates from Sorted Array | Easy | Same-direction read/write pointers | 26 |
| 4 | Move Zeroes | Easy | Same-direction, partition around zero | 283 |
| 5 | Squares of Sorted Array | Easy | Converging pointers, largest at ends | 977 |
| 6 | 3Sum | Medium | Fix one, two-pointer on rest, skip duplicates | 15 |
| 7 | Container With Most Water | Medium | Converging pointers, move the shorter side | 11 |
| 8 | Sort Colors | Medium | Three-way partition (see [Pattern 7](#7-dutch-national-flag--cyclic-sort)) | 75 |
| 9 | 4Sum | Medium | Generalize 3Sum with one more loop | 18 |
| 10 | Trapping Rain Water | Hard | Two pointers maintaining leftMax/rightMax | 42 |

---

## 2. Sliding Window — Fixed Size

### When to Use (Recognition Signals)
- "Maximum/minimum/average of **every subarray of size K**"
- "Find property X in **all windows of size K**"
- The window size is **given** or **constant**

### When NOT to Use
- Window size varies based on a condition → use Variable Sliding Window
- You need sum of arbitrary ranges (not just sliding) → use Prefix Sum
- Elements aren't contiguous → not a subarray problem

### Optimized Java Template

```java
// Fixed-size sliding window with running aggregate
public long maxSumSubarray(int[] nums, int k) {
    if (nums.length < k) return -1;  // edge case: array smaller than window

    // Build first window
    long windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    long maxSum = windowSum;

    // Slide: add right edge, remove left edge
    for (int i = k; i < nums.length; i++) {
        windowSum += nums[i];       // add new element entering window
        windowSum -= nums[i - k];   // remove element leaving window
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}
```

**Why each line matters:**
- `long windowSum`: prevents integer overflow — n up to 10^5, values up to 10^4 → sum up to 10^9, which fits int, but 10^5 × 10^9 = 10^14 overflows. Always use `long` for sums in interviews.
- Build first window separately: avoids boundary checks inside the main loop
- Single subtraction + addition per slide: O(1) per step, not O(k) recomputation

### Fully Solved Problem: Sliding Window Maximum (LC 239)

**Problem:** Given an array `nums` and a window size `k`, return the maximum value in each window as the window slides from left to right.

**Thinking Process:**
1. Naive: for each window, scan all k elements → O(nk). For n = 10^5 and k = 10^4, this is 10^9 operations. TLE.
2. Can we maintain the maximum efficiently? We need a data structure that supports: add element, remove element, get max — all in O(1) or O(log n).
3. **Key insight:** we don't need to track ALL elements in the window. If `nums[j] >= nums[i]` and `j > i`, then `nums[i]` will NEVER be the maximum for any future window (because `nums[j]` will always be in the same or later window). We can discard `nums[i]`.
4. This is a **monotonic decreasing deque** — front has the max, and we remove smaller elements from the back.

**Solution:**
```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();  // stores INDICES, not values

    for (int i = 0; i < n; i++) {
        // Remove elements outside the window from front
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
            deque.pollFirst();
        }

        // Remove smaller elements from back (they'll never be the max)
        while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }

        deque.offerLast(i);

        // Window is fully formed starting at index k-1
        if (i >= k - 1) {
            result[i - k + 1] = nums[deque.peekFirst()];
        }
    }

    return result;
}
```

**Walkthrough with `[1, 3, -1, -3, 5, 3, 6, 7]`, k=3:**
```
i=0: deque=[0]          (value 1)
i=1: 3>1, remove 0.     deque=[1]          (value 3)
i=2: -1<3, keep.        deque=[1,2]         result[0]=nums[1]=3
i=3: -3<-1, keep.       deque=[1,2,3]       result[1]=nums[1]=3
i=4: 5>-3,>-1,>3.       deque=[4]           result[2]=nums[4]=5
i=5: 3<5, keep.         deque=[4,5]         result[3]=nums[4]=5
i=6: 6>3,>5.            deque=[6]           result[4]=nums[6]=6
i=7: 7>6.               deque=[7]           result[5]=nums[7]=7
```
Result: `[3, 3, 5, 5, 6, 7]`

### Complexity Analysis
- **Time:** O(n) — each element is added to and removed from the deque at most once → 2n operations total
- **Space:** O(k) — deque holds at most k elements
- This is optimal: you must read all n elements at least once

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Storing values instead of indices in deque | Can't determine when an element leaves the window | Always store indices — get value via `nums[deque.peek()]` |
| Using `Stack` or `LinkedList` instead of `ArrayDeque` | Stack: wrong interface (LIFO only). LinkedList: slow (node allocation per element) | `ArrayDeque` supports both ends in O(1) with array backing |
| Off-by-one on window formation | Either missing the first window or outputting too many | Window is valid when `i >= k - 1` |
| Using `<` instead of `<=` when removing from back | Equal elements get removed, but sometimes you need to keep them | Depends on problem — `<=` removes equal (strict decrease), `<` keeps equal |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you do this with a heap?" | "Yes — max-heap of (value, index). O(n log n) because we lazily remove stale entries. Monotonic deque is better at O(n)." |
| "What about sliding window minimum?" | "Same structure, but maintain monotonic INCREASING deque instead of decreasing." |
| "What if k changes for each position?" | "Then it's not a standard sliding window — might need a segment tree or sparse table for range queries." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Maximum Average Subarray I | Easy | Fixed window sum / k | 643 |
| 2 | Contains Duplicate II | Easy | Fixed window with HashSet | 219 |
| 3 | Max Sum of Distinct Subarrays With Length K | Medium | Fixed window + uniqueness constraint | 2461 |
| 4 | Grumpy Bookstore Owner | Medium | Which k-length window maximizes gain | 1052 |
| 5 | Sliding Window Maximum | Hard | Monotonic deque (solved above) | 239 |

---

## 3. Sliding Window — Variable Size

### When to Use (Recognition Signals)
- "**Longest** subarray/substring satisfying condition X"
- "**Shortest** subarray/substring satisfying condition X"
- "Subarray with **at most** K distinct elements"
- The window grows and shrinks based on a constraint

### When NOT to Use
- You need **exactly K** distinct elements → decompose into "at most K" minus "at most K-1"
- The array has **negative numbers** and you're looking for a sum condition → Prefix Sum + HashMap (sliding window needs monotonicity)
- The problem asks about subsequences (non-contiguous) → DP or two-pointer on sorted

### Optimized Java Template

```java
// Template: Longest subarray/substring satisfying condition
// Works for: longest substring without repeating chars, with at most K distinct, etc.
public int longestWithCondition(String s) {
    int[] freq = new int[128];  // ASCII frequency map — 5-10x faster than HashMap
    int lo = 0, maxLen = 0;

    for (int hi = 0; hi < s.length(); hi++) {
        freq[s.charAt(hi)]++;

        // Shrink window while condition is VIOLATED
        while (conditionViolated(freq)) {
            freq[s.charAt(lo)]--;
            lo++;
        }

        maxLen = Math.max(maxLen, hi - lo + 1);
    }

    return maxLen;
}

// Template: Shortest subarray satisfying condition
// Works for: minimum window substring, minimum size subarray sum, etc.
public int shortestWithCondition(int[] nums, int target) {
    int lo = 0, minLen = Integer.MAX_VALUE;
    int windowSum = 0;

    for (int hi = 0; hi < nums.length; hi++) {
        windowSum += nums[hi];

        // Shrink window while condition is SATISFIED (we want minimum)
        while (windowSum >= target) {
            minLen = Math.min(minLen, hi - lo + 1);
            windowSum -= nums[lo];
            lo++;
        }
    }

    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
```

**Why each line matters:**
- `int[128]` instead of `HashMap<Character, Integer>`: eliminates autoboxing (each `put`/`get` boxes int → Integer, creating garbage). For ASCII problems, this is 5-10x faster in practice.
- **Shrink with `while`, never `if`**: the window might need to shrink by multiple elements. A common bug is using `if` which only shrinks by one.
- **Longest: update AFTER shrinking** (window is valid). **Shortest: update INSIDE the shrink loop** (window just became valid).

### Fully Solved Problem: Minimum Window Substring (LC 76)

**Problem:** Given strings `s` and `t`, find the minimum window in `s` that contains ALL characters of `t` (including duplicates).

**Thinking Process:**
1. We need a window that contains all characters of `t`. This is a "shortest subarray satisfying condition" problem.
2. Expand `hi` to include more characters. When the window contains all of `t`, try shrinking `lo` to find the minimum.
3. How to check "contains all of `t`"? Track frequencies. A character is "satisfied" when its count in the window >= its count in `t`.
4. Optimization: instead of checking all 128 chars each time, maintain a counter `formed` — number of unique characters whose frequency requirement is met.

**Solution:**
```java
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";

    int[] need = new int[128];   // frequency of each char in t
    int[] have = new int[128];   // frequency of each char in current window

    int required = 0;  // number of unique chars in t
    for (char c : t.toCharArray()) {
        if (need[c] == 0) required++;
        need[c]++;
    }

    int formed = 0;  // number of unique chars where have[c] >= need[c]
    int lo = 0;
    int minLen = Integer.MAX_VALUE;
    int minStart = 0;

    for (int hi = 0; hi < s.length(); hi++) {
        char c = s.charAt(hi);
        have[c]++;

        // Check if this char's requirement is now met
        if (need[c] > 0 && have[c] == need[c]) {
            formed++;
        }

        // Shrink window while all requirements are met
        while (formed == required) {
            int windowLen = hi - lo + 1;
            if (windowLen < minLen) {
                minLen = windowLen;
                minStart = lo;
            }

            char leftChar = s.charAt(lo);
            have[leftChar]--;

            if (need[leftChar] > 0 && have[leftChar] < need[leftChar]) {
                formed--;
            }
            lo++;
        }
    }

    return minLen == Integer.MAX_VALUE ? "" : s.substring(minStart, minStart + minLen);
}
```

**Walkthrough with s = "ADOBECODEBANC", t = "ABC":**
```
need: A=1, B=1, C=1, required=3

hi=0 (A): have[A]=1, formed=1
hi=1 (D): have[D]=1, formed=1
hi=2 (O): have[O]=1, formed=1
hi=3 (B): have[B]=1, formed=2
hi=4 (E): have[E]=1, formed=2
hi=5 (C): have[C]=1, formed=3 ← all met!
  Shrink: window="ADOBEC" len=6, minLen=6, minStart=0
  Remove A: have[A]=0, formed=2. lo=1. Stop.
hi=6 (O): formed=2
hi=7 (D): formed=2
hi=8 (E): formed=2
hi=9 (B): formed=2
hi=10 (A): have[A]=1, formed=3 ← all met!
  Shrink: window="DOBECODEBA" len=10. Not better.
  Remove D: still formed=3. window="OBECODEBA" len=9. Not better.
  Remove O: still formed=3. window="BECODEBA" len=8. Not better.
  Remove B: have[B]=1, still >=1. formed=3. window="ECODEBA" len=7. Not better.
  Remove E: formed=3. window="CODEBA" len=6. Equal. Not better.
  Remove C: have[C]=0, formed=2. lo=6. Stop.
hi=11 (N): formed=2
hi=12 (C): have[C]=1, formed=3 ← all met!
  Shrink: window="ODEBANC" len=7. Not better.
  Remove O: formed=3. window="DEBANC" len=6. Equal.
  Remove D: formed=3. window="EBANC" len=5. Better! minLen=5, minStart=8.
  Remove E: formed=3. window="BANC" len=4. Better! minLen=4, minStart=9.
  Remove B: have[B]=0, formed=2. lo=10. Stop.

Result: s[9..12] = "BANC"
```

### Complexity Analysis
- **Time:** O(|s| + |t|) — each character in `s` is visited at most twice (once by `hi`, once by `lo`). Building `need` takes O(|t|).
- **Space:** O(1) — two fixed-size arrays of 128 integers (constant regardless of input size)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using `HashMap` for character frequency | Autoboxing, GC pressure, 5-10x slower | Use `int[128]` for ASCII, `int[26]` for lowercase only |
| Using `if` instead of `while` for shrinking | Window only shrinks by 1, missing shorter valid windows | Always use `while` — the window may need to shrink by many elements |
| Checking `have[c] == need[c]` without `need[c] > 0` guard | Characters not in `t` can accidentally increment `formed` | Only track characters that appear in `t` |
| Not handling `t` with duplicate characters | "t = AAB" requires the window to have 2 A's and 1 B, not just 1 of each | `need[c]` tracks the required COUNT, not just presence |
| Confusing "longest" vs "shortest" template | Longest updates OUTSIDE the shrink loop, shortest updates INSIDE | Memorize: longest = "valid window, try to maximize", shortest = "just became valid, record and shrink" |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "What if the alphabet is Unicode (100K+ chars)?" | "Replace `int[128]` with `HashMap<Character, Integer>`. Time complexity unchanged, but constant factor increases." |
| "Can you do it in one pass?" | "This IS one pass — each character is processed at most twice (right pointer adds, left pointer removes)." |
| "What if t has characters not in s?" | "Return empty string — impossible to form the window. Our solution handles this: `formed` will never reach `required`." |
| "What about the longest substring with at most K distinct characters?" | "Same template but for 'longest'. Shrink when distinct count exceeds K. Use `int[128]` and a counter for distinct chars." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Max Consecutive Ones III | Medium | Variable window, flip at most K zeros | 1004 |
| 2 | Longest Substring Without Repeating Characters | Medium | Shrink when duplicate found (freq > 1) | 3 |
| 3 | Longest Repeating Character Replacement | Medium | Shrink when window - maxFreq > K | 424 |
| 4 | Fruit Into Baskets | Medium | Longest with at most 2 distinct | 904 |
| 5 | Longest Substring with At Most K Distinct Characters | Medium | Generalization of Fruit Into Baskets | 340 |
| 6 | Minimum Window Substring | Hard | Solved above | 76 |
| 7 | Subarrays with K Different Integers | Hard | "Exactly K" = "at most K" - "at most K-1" | 992 |
| 8 | Minimum Size Subarray Sum | Medium | Shortest window with sum >= target | 209 |

---

## 4. Prefix Sum

### When to Use (Recognition Signals)
- "**Sum of subarray** from index i to j"
- "**Number of subarrays** with sum equal to K"
- "**Subarray sum divisible** by K"
- Multiple range sum queries on a static array
- You see a cumulative/running total pattern

### When NOT to Use
- Array is modified between queries → use Segment Tree or BIT (see [Advanced Patterns](./14-advanced-patterns.md))
- You need max/min of a range (not sum) → use Sparse Table or Segment Tree
- The problem is about contiguous max sum → use Kadane's (simpler)

### Optimized Java Template

```java
// Template A: Prefix Sum Array (for range sum queries)
public class PrefixSum {
    private long[] prefix;

    public PrefixSum(int[] nums) {
        prefix = new long[nums.length + 1];  // prefix[0] = 0 as sentinel
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }

    // Sum from index lo to hi (inclusive), O(1)
    public long rangeSum(int lo, int hi) {
        return prefix[hi + 1] - prefix[lo];
    }
}

// Template B: HashMap Prefix Sum (for counting subarrays with sum = K)
public int subarraySum(int[] nums, int k) {
    Map<Long, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0L, 1);  // empty prefix has sum 0 — crucial initialization

    long sum = 0;
    int count = 0;

    for (int num : nums) {
        sum += num;
        // If (sum - k) was a previous prefix sum, then the subarray between them has sum k
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }

    return count;
}
```

**Why each line matters:**
- `prefix[0] = 0` sentinel: enables `rangeSum(0, j)` without special-casing
- `long[] prefix`: prevents overflow — sum of 10^5 elements each up to 10^9 = 10^14, needs long
- `prefixCount.put(0L, 1)`: this is the most commonly forgotten line. Without it, subarrays starting at index 0 are missed. WHY: if `sum == k`, then `sum - k == 0`, and we need to count that one prefix (the empty prefix) that has sum 0.
- `prefixCount.merge(sum, 1, Integer::sum)`: single hash lookup instead of `getOrDefault` + `put` (two lookups)

### Fully Solved Problem: Subarray Sum Equals K (LC 560)

**Problem:** Given an integer array `nums` and an integer `k`, return the total number of subarrays whose sum equals `k`. Array may contain negative numbers.

**Thinking Process:**
1. Brute force: check all O(n^2) subarrays → compute each sum → O(n^2) or O(n^3). TLE.
2. Can we use sliding window? NO — negative numbers break the monotonicity. When we expand the window, the sum can decrease. So we can't make shrink decisions.
3. Key insight: `sum(i..j) = prefix[j+1] - prefix[i]`. We want `prefix[j+1] - prefix[i] = k`, i.e., `prefix[i] = prefix[j+1] - k`. For each `j`, count how many previous prefix sums equal `sum - k`.
4. HashMap stores {prefix_sum → count of occurrences}. This is the same idea as Two Sum but applied to prefix sums.

**Solution:**
```java
public int subarraySum(int[] nums, int k) {
    Map<Long, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0L, 1);

    long sum = 0;
    int count = 0;

    for (int num : nums) {
        sum += num;
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }

    return count;
}
```

**Walkthrough with `nums = [1, 2, 3]`, k = 3:**
```
Initial: prefixCount = {0: 1}, sum = 0, count = 0

i=0, num=1: sum=1. sum-k=1-3=-2. prefixCount[-2]=0. count=0.
  prefixCount = {0:1, 1:1}

i=1, num=2: sum=3. sum-k=3-3=0. prefixCount[0]=1. count=1.
  prefixCount = {0:1, 1:1, 3:1}

i=2, num=3: sum=6. sum-k=6-3=3. prefixCount[3]=1. count=2.
  prefixCount = {0:1, 1:1, 3:1, 6:1}

Result: 2 (subarrays [1,2] and [3])
```

### Complexity Analysis
- **Time:** O(n) — single pass, HashMap operations are O(1) amortized
- **Space:** O(n) — HashMap stores up to n+1 prefix sums

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Forgetting `prefixCount.put(0, 1)` | Misses subarrays starting at index 0 | Always initialize with empty prefix sum |
| Trying sliding window with negative numbers | Window can't decide when to shrink (sum isn't monotonic) | Use prefix sum + HashMap for arrays with negatives |
| Using `int` for prefix sums | Overflow when n × max_value exceeds 2^31 | Use `long` |
| Confusing "number of subarrays" with "does a subarray exist" | Using boolean instead of count; missing subarrays when same prefix sum appears multiple times | Store COUNT of each prefix sum, not just presence |
| Adding to map BEFORE checking | Counting the empty subarray (from element to itself) incorrectly | Always CHECK first, then ADD to map |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you do this with sliding window since there are no negatives?" | "If all elements are positive, yes — sliding window works because the sum is monotonically increasing as we expand. But this problem allows negatives, so we must use prefix sum." |
| "What if the array is very long and we need multiple queries for different k values?" | "Precompute the prefix sum array. For each query, iterate once with the HashMap. Or, if the same array is used repeatedly, consider storing prefix sums and binary searching." |
| "What about 2D prefix sum?" | "Extend to 2D: `prefix[i][j] = sum of rectangle (0,0) to (i-1,j-1)`. Range sum uses inclusion-exclusion: `prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]`." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Range Sum Query - Immutable | Easy | Direct prefix sum array | 303 |
| 2 | Find Pivot Index | Easy | Left sum = total - left sum - nums[i] | 724 |
| 3 | Running Sum of 1d Array | Easy | prefix[i] = prefix[i-1] + nums[i] | 1480 |
| 4 | Subarray Sum Equals K | Medium | Solved above | 560 |
| 5 | Contiguous Array | Medium | Convert 0→-1, then subarray sum = 0 | 525 |
| 6 | Product of Array Except Self | Medium | Prefix product and suffix product | 238 |
| 7 | Subarray Sums Divisible by K | Medium | Prefix sum mod K, count same remainders | 974 |
| 8 | Continuous Subarray Sum | Medium | Prefix sum mod K, check if same remainder seen (with gap ≥ 2) | 523 |
| 9 | Count of Range Sum | Hard | Merge sort on prefix sums for counting inversions in range | 327 |

---

## 5. Kadane's Algorithm

### When to Use (Recognition Signals)
- "**Maximum sum** contiguous subarray"
- "**Maximum product** subarray"
- "Best time to **buy and sell** stock" (single transaction — this IS Kadane's)
- Any problem where you track "best ending here" to build "best overall"

### When NOT to Use
- You need the actual subarray (not just the sum) AND the array has specific index requirements → need to track indices separately
- Circular array → modified Kadane's (max of normal Kadane's vs total - min subarray sum)
- Multiple non-overlapping subarrays → DP, not Kadane's

### Optimized Java Template

```java
// Template A: Maximum Subarray Sum (with index tracking)
public int[] maxSubarrayWithIndices(int[] nums) {
    int maxSum = nums[0], currSum = nums[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < nums.length; i++) {
        if (currSum + nums[i] < nums[i]) {
            currSum = nums[i];
            tempStart = i;
        } else {
            currSum += nums[i];
        }

        if (currSum > maxSum) {
            maxSum = currSum;
            start = tempStart;
            end = i;
        }
    }

    return new int[]{maxSum, start, end};
}

// Template B: Maximum Product Subarray
public int maxProduct(int[] nums) {
    int maxProd = nums[0];
    int currMax = nums[0], currMin = nums[0];  // track BOTH min and max

    for (int i = 1; i < nums.length; i++) {
        if (nums[i] < 0) {
            // Negative number flips max and min
            int temp = currMax;
            currMax = currMin;
            currMin = temp;
        }

        currMax = Math.max(nums[i], currMax * nums[i]);
        currMin = Math.min(nums[i], currMin * nums[i]);
        maxProd = Math.max(maxProd, currMax);
    }

    return maxProd;
}
```

**Why each line matters:**
- `currSum + nums[i] < nums[i]` is equivalent to `currSum < 0` — if the previous sum is negative, it can only hurt us, so start fresh
- **Product variant needs both min and max**: a large negative × negative can become the maximum. Example: `[-2, 3, -4]` → min at index 1 is -6, then -6 × -4 = 24 is the answer.
- Swap on negative: when we multiply by a negative number, the maximum becomes the minimum and vice versa

### Fully Solved Problem: Maximum Subarray (LC 53) + Circular Variant (LC 918)

**Problem (LC 53):** Find the contiguous subarray with the largest sum.

**Solution:**
```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int currSum = 0;

    for (int num : nums) {
        currSum = Math.max(num, currSum + num);
        maxSum = Math.max(maxSum, currSum);
    }

    return maxSum;
}
```

**The insight that makes Kadane's work:**
- At each position, we make a greedy choice: either extend the previous subarray or start a new one here.
- If `currSum + num < num` (i.e., `currSum < 0`), the previous subarray is "toxic" — it can only reduce our sum. So we start fresh.
- This greedy choice is provably optimal because any maximum subarray either (a) extends a previous positive-sum subarray or (b) starts at the current element.

**Follow-up: Maximum Sum Circular Subarray (LC 918):**

The maximum circular subarray is either:
1. A normal (non-wrapping) subarray → standard Kadane's
2. A wrapping subarray → the "middle" part that we DON'T take is the minimum subarray. So the answer is `totalSum - minSubarraySum`.

```java
public int maxSubarraySumCircular(int[] nums) {
    int totalSum = 0;
    int maxSum = nums[0], currMax = 0;
    int minSum = nums[0], currMin = 0;

    for (int num : nums) {
        currMax = Math.max(num, currMax + num);
        maxSum = Math.max(maxSum, currMax);

        currMin = Math.min(num, currMin + num);
        minSum = Math.min(minSum, currMin);

        totalSum += num;
    }

    // Edge case: all elements are negative → maxSum < 0, and totalSum - minSum = 0 (empty subarray)
    // We can't take an empty subarray, so return maxSum
    if (maxSum < 0) return maxSum;

    return Math.max(maxSum, totalSum - minSum);
}
```

### Complexity Analysis
- **Time:** O(n) — single pass
- **Space:** O(1) — only tracking running variables
- This is optimal: any algorithm must read all n elements

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Initializing `maxSum = 0` instead of `nums[0]` | Fails when all elements are negative (returns 0 instead of least negative) | Always initialize with `nums[0]` |
| Not handling the "all negative" edge case in circular variant | `totalSum - minSum = 0`, which represents an empty subarray (not valid) | Check `if (maxSum < 0) return maxSum` |
| Product variant: ignoring zeros | Zero resets both currMax and currMin to 0, which is correct, but the max should be at least 0 | The template handles this naturally via `Math.max(nums[i], ...)` |
| Confusing Kadane's with prefix sum for "subarray sum = K" | Kadane's finds MAX sum, not a specific sum. For "sum = K", use prefix sum + HashMap | Different problems, different patterns |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Return the actual subarray, not just the sum" | "Track `start`, `end`, `tempStart`. Update `start = tempStart, end = i` whenever `maxSum` improves." |
| "What about maximum sum of K non-overlapping subarrays?" | "DP with states: dp[i][j] = max sum using first i elements with exactly j subarrays. O(n*K) time." |
| "Can you do it with divide and conquer?" | "Yes — max subarray is in left half, right half, or crossing the midpoint. O(n log n). But Kadane's is O(n) and simpler." |
| "Best Time to Buy and Sell Stock — is that Kadane's?" | "Yes! Convert prices to daily changes: `changes[i] = prices[i] - prices[i-1]`. Maximum subarray of changes = maximum profit." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Maximum Subarray | Easy | Standard Kadane's | 53 |
| 2 | Best Time to Buy and Sell Stock | Easy | Kadane's on price differences (or just track min price) | 121 |
| 3 | Maximum Product Subarray | Medium | Track both max and min (negative × negative = positive) | 152 |
| 4 | Maximum Sum Circular Subarray | Medium | max(Kadane's max, total - Kadane's min) | 918 |
| 5 | Longest Turbulent Subarray | Medium | Kadane's variant — reset when pattern breaks | 978 |

---

## 6. Hash Map / Set Operations

### When to Use (Recognition Signals)
- Need **O(1) lookup** — "does this element exist?"
- **Frequency counting** — "how many times does X appear?"
- **Grouping** — "group elements by some property"
- "Two Sum" style — "find complement in O(1)"
- **Deduplication** — "unique elements"

### When NOT to Use
- You need elements in **sorted order** → use TreeMap/TreeSet (O(log n) operations)
- You need the **k-th smallest/largest** → use heap or quickselect
- Simple boolean tracking for known small range → use boolean array (faster than HashSet)

### Optimized Java Template

```java
// Template A: Frequency Count (modern Java)
public Map<Integer, Integer> frequencyMap(int[] nums) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.merge(num, 1, Integer::sum);  // one lookup, not two
    }
    return freq;
}

// Template B: Group By Property
public Map<String, List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}

// Template C: Two Sum Pattern (find complement)
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```

**Why each line matters:**
- `freq.merge(num, 1, Integer::sum)`: single hash lookup. The common pattern `freq.put(num, freq.getOrDefault(num, 0) + 1)` does TWO lookups (get + put). `merge` does one.
- `groups.computeIfAbsent(key, k -> new ArrayList<>())`: creates the list only if absent, and returns it for chaining. Cleaner than `if (!map.containsKey) map.put(...)`.
- Two Sum checks BEFORE inserting: this ensures we don't match an element with itself.

### Fully Solved Problem: Group Anagrams (LC 49)

**Problem:** Group strings that are anagrams of each other.

**Thinking Process:**
1. Two strings are anagrams if they have the same characters in the same frequencies.
2. Key insight: sort each string → anagrams produce the same sorted string. Use sorted string as HashMap key.
3. Alternative: use character frequency as key (e.g., `"#1#0#0#..."` for 26 letters). This is O(n*m) vs O(n*m*log(m)) but more complex to implement.

**Solution:**
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();

    for (String s : strs) {
        // O(m log m) key generation via sorting
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);

        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }

    return new ArrayList<>(groups.values());
}
```

**Alternative with frequency key (O(n*m) — better for long strings):**
```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();

    for (String s : strs) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) {
            freq[c - 'a']++;
        }
        // Build key like "#1#0#3#..." — unique per anagram group
        StringBuilder sb = new StringBuilder();
        for (int f : freq) {
            sb.append('#').append(f);
        }
        String key = sb.toString();

        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }

    return new ArrayList<>(groups.values());
}
```

### Complexity Analysis
- **Sort-based:** Time O(n * m log m), Space O(n * m) — n strings of average length m
- **Frequency-based:** Time O(n * m), Space O(n * m) — strictly better for long strings
- In interviews, mention both and let the interviewer choose

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| `Integer` comparison with `==` | Works for -128 to 127 (Integer cache), SILENTLY FAILS for 128+ | Always use `.equals()` for `Integer` comparisons |
| Not pre-sizing HashMap | Rehashing during growth causes GC spikes | `new HashMap<>(n * 4 / 3 + 1)` for known size |
| Using mutable objects as HashMap keys | If the key object is modified after insertion, hash changes, entry becomes unfindable | Use immutable keys (String, Integer) or defensive copy |
| Forgetting that `HashMap` iteration order is undefined | Code depends on insertion order | Use `LinkedHashMap` if order matters |
| `HashSet.add()` returns boolean | Ignoring the return value means you don't know if element was already present | Use the return value: `if (!seen.add(num)) { /* duplicate */ }` |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "What's the time complexity of HashMap operations?" | "O(1) amortized for get/put. Worst case O(n) if all keys hash to the same bucket (pathological). Java 8+ uses balanced trees for long chains, making worst case O(log n)." |
| "What if we need the groups sorted?" | "Use `TreeMap` instead of `HashMap` for sorted keys, or sort the result after grouping." |
| "What's the load factor and why does it matter?" | "Default 0.75. When size > capacity × load_factor, HashMap doubles and rehashes everything — O(n) operation. Pre-size to avoid this." |
| "HashMap vs HashTable vs ConcurrentHashMap?" | "HashMap: not thread-safe, allows null keys. Hashtable: synchronized (legacy, don't use). ConcurrentHashMap: thread-safe with striped locking, no null keys." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Two Sum | Easy | Complement lookup in HashMap | 1 |
| 2 | Valid Anagram | Easy | Frequency count comparison | 242 |
| 3 | Contains Duplicate | Easy | HashSet.add() returns false on duplicate | 217 |
| 4 | Majority Element | Easy | Boyer-Moore voting OR HashMap frequency | 169 |
| 5 | Intersection of Two Arrays | Easy | HashSet intersection | 349 |
| 6 | Group Anagrams | Medium | Sorted string or frequency as key | 49 |
| 7 | Top K Frequent Elements | Medium | HashMap + Bucket Sort or Heap | 347 |
| 8 | Longest Consecutive Sequence | Medium | HashSet, expand left/right from each start | 128 |
| 9 | Valid Sudoku | Medium | HashSet per row, column, box | 36 |
| 10 | First Missing Positive | Hard | In-place hashing / cyclic sort | 41 |

---

## 7. Dutch National Flag / Cyclic Sort

### When to Use (Recognition Signals)

**Dutch National Flag (3-way partition):**
- "Sort an array of **0s, 1s, and 2s**"
- "Partition array into **three groups**"
- "Move all X to front, Y to middle, Z to end — **in-place, one pass**"

**Cyclic Sort:**
- Array contains numbers in range **[1, n]** (or [0, n])
- "Find the **missing number**"
- "Find the **duplicate number**"
- "Find **all missing/duplicate** numbers"

### When NOT to Use
- More than 3 categories for DNF → use counting sort or general partition
- Range is not bounded to [1, n] → cyclic sort doesn't apply, use HashSet
- Array is already sorted or nearly sorted → insertion sort or no sort needed

### Optimized Java Template

```java
// Template A: Dutch National Flag (3-way partition)
public void sortColors(int[] nums) {
    int lo = 0;                    // boundary: everything before lo is 0
    int hi = nums.length - 1;     // boundary: everything after hi is 2
    int mid = 0;                   // current scanner

    while (mid <= hi) {
        if (nums[mid] == 0) {
            swap(nums, lo, mid);
            lo++;
            mid++;                 // safe to advance: swapped element was already processed
        } else if (nums[mid] == 2) {
            swap(nums, mid, hi);
            hi--;
            // DON'T advance mid: swapped element from hi hasn't been examined yet
        } else {
            mid++;                 // 1 is in the right place
        }
    }
}

// Template B: Cyclic Sort (numbers in [1, n])
public void cyclicSort(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correctIdx = nums[i] - 1;  // where nums[i] should be
        if (nums[i] != nums[correctIdx]) {
            swap(nums, i, correctIdx);
            // DON'T advance i: new element at i needs to be checked
        } else {
            i++;  // nums[i] is in correct position (or duplicate)
        }
    }
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Why each line matters:**
- DNF: `mid++` after swapping with `lo` but NOT after swapping with `hi`. The element from `lo` was already scanned (it was in the region `[lo, mid)` which contains only 1s), but the element from `hi` is unknown.
- Cyclic Sort: `correctIdx = nums[i] - 1` — element with value `v` belongs at index `v-1`. We keep swapping until the current position has the right element.
- Cyclic Sort: `nums[i] != nums[correctIdx]` prevents infinite loop when there are duplicates.

### Fully Solved Problem: Find All Numbers Disappeared in an Array (LC 448)

**Problem:** Given an array of n integers where each integer is in the range [1, n], find all integers in [1, n] that don't appear.

**Thinking Process:**
1. Brute force: HashSet of all present numbers, check 1 to n → O(n) time, O(n) space.
2. Can we do O(1) extra space? The array itself has n slots for values [1, n]. If we place each value at its correct index, then missing values correspond to misplaced positions.
3. Cyclic sort: place `nums[i]` at index `nums[i] - 1`. After sorting, scan for positions where `nums[i] != i + 1`.

**Solution:**
```java
public List<Integer> findDisappearedNumbers(int[] nums) {
    // Cyclic sort: place each number at its correct index
    int i = 0;
    while (i < nums.length) {
        int correctIdx = nums[i] - 1;
        if (nums[i] != nums[correctIdx]) {
            swap(nums, i, correctIdx);
        } else {
            i++;
        }
    }

    // Numbers not at their correct position are missing
    List<Integer> result = new ArrayList<>();
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != j + 1) {
            result.add(j + 1);
        }
    }
    return result;
}
```

**Walkthrough with `[4, 3, 2, 7, 8, 2, 3, 1]`:**
```
Initial: [4, 3, 2, 7, 8, 2, 3, 1]

i=0: nums[0]=4, correctIdx=3. nums[0]≠nums[3]. Swap → [7, 3, 2, 4, 8, 2, 3, 1]
i=0: nums[0]=7, correctIdx=6. nums[0]≠nums[6]. Swap → [3, 3, 2, 4, 8, 2, 7, 1]
i=0: nums[0]=3, correctIdx=2. nums[0]≠nums[2]. Swap → [2, 3, 3, 4, 8, 2, 7, 1]
i=0: nums[0]=2, correctIdx=1. nums[0]≠nums[1]? 2≠3. Swap → [3, 2, 3, 4, 8, 2, 7, 1]
i=0: nums[0]=3, correctIdx=2. nums[0]==nums[2]? 3==3. Yes. i++.
i=1: nums[1]=2, correctIdx=1. nums[1]==nums[1]. i++.
... (continue until end)

Final: [1, 2, 3, 4, 8, 2, 7, 3] (approximately — 8 and extras remain displaced)

Scan: index 4 has 8 (not 5) → missing 5. Index 5 has 2 (not 6) → missing 6.
Result: [5, 6]
```

### Complexity Analysis
- **DNF Time:** O(n) — single pass, each element moved at most twice
- **Cyclic Sort Time:** O(n) — despite the while loop inside while loop, each element is swapped at most once to its correct position → at most n swaps total
- **Space:** O(1) — in-place for both

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| DNF: advancing `mid` after swapping with `hi` | Unexamined element gets skipped | Only advance `mid` when swapping with `lo` or when element is 1 |
| Cyclic Sort: `correctIdx = nums[i]` instead of `nums[i] - 1` | Off-by-one: values [1,n] go to indices [0,n-1] | Always subtract 1 for [1,n] range |
| Cyclic Sort: using `nums[i] != i + 1` as swap condition | Infinite loop when duplicates exist (swapping equal values forever) | Use `nums[i] != nums[correctIdx]` to handle duplicates |
| Modifying array when problem says "don't modify" | Some problems require O(1) space AND read-only array (e.g., LC 287) | Use Floyd's cycle detection instead of cyclic sort |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you find the duplicate number without modifying the array?" | "Floyd's cycle detection on the array treated as a linked list: `next(i) = nums[i]`. LC 287." |
| "What if there are more than 3 categories?" | "Use counting sort (O(n + k)) or a general partition algorithm." |
| "Why not just use counting sort for Sort Colors?" | "Counting sort works but requires two passes (count, then fill). DNF does it in one pass. Interviewer may specifically ask for one-pass." |
| "Can cyclic sort work for range [0, n] instead of [1, n]?" | "Yes — `correctIdx = nums[i]` (no minus 1). Skip elements equal to n (they have no correct position in an n-length array)." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Missing Number | Easy | XOR or cyclic sort for [0, n] range | 268 |
| 2 | Sort Colors | Medium | Dutch National Flag 3-way partition | 75 |
| 3 | Find All Numbers Disappeared in Array | Easy | Cyclic sort, then scan for mismatches | 448 |
| 4 | Find the Duplicate Number | Medium | Floyd's cycle detection (no modification) or cyclic sort | 287 |
| 5 | Find All Duplicates in an Array | Medium | Cyclic sort, then positions where nums[i] != i+1 | 442 |
| 6 | Set Mismatch | Easy | Cyclic sort, find where value != index+1 | 645 |
| 7 | First Missing Positive | Hard | Cyclic sort on positive numbers only; ignore negatives and numbers > n | 41 |

---

## Cross-References

- **Sliding Window Maximum** uses [Monotonic Deque](./03-stacks-and-queues.md#4-monotonic-queue-deque) for O(n) — covered in detail there
- **Two Sum** appears in [Hash Map](#6-hash-map--set-operations) but the sorted variant uses [Two Pointers](#1-two-pointers)
- **Prefix Sum with Binary Search** for range count queries — see [Binary Search](./04-binary-search.md)
- **Kadane's** on price changes is equivalent to [Stock Problems](./07-dynamic-programming.md#7-state-machine-dp-buysell-stock) state machine DP
- **First Missing Positive** combines [Cyclic Sort](#7-dutch-national-flag--cyclic-sort) with edge case handling
