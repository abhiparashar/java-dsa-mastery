# 13 — Bit Manipulation

Bit manipulation gives O(1) operations that replace loops and hash lookups. Two patterns dominate interviews: XOR tricks (find missing/duplicate, cancel pairs) and bitmask state encoding (subset enumeration, DP state compression). Master the 10 essential operations below and you can solve any bit problem.

---

## Table of Contents
1. [XOR Properties](#1-xor-properties)
2. [Bit Masking](#2-bit-masking)
3. [Essential Bit Operations](#3-essential-bit-operations)

---

## 3. Essential Bit Operations

Master these before anything else — they're building blocks for all bit problems.

```java
// 1. Check if i-th bit is set
boolean isSet = (n & (1 << i)) != 0;

// 2. Set the i-th bit
n = n | (1 << i);

// 3. Clear the i-th bit
n = n & ~(1 << i);

// 4. Toggle the i-th bit
n = n ^ (1 << i);

// 5. Clear the lowest set bit (used in counting bits)
n = n & (n - 1);

// 6. Isolate the lowest set bit
int lowest = n & (-n);

// 7. Check if power of 2
boolean isPow2 = n > 0 && (n & (n - 1)) == 0;

// 8. Count set bits (Brian Kernighan's)
int count = 0;
while (n != 0) { n &= (n - 1); count++; }
// Or: Integer.bitCount(n)

// 9. Get all 1s mask of length k
int mask = (1 << k) - 1;

// 10. Swap without temp
a ^= b; b ^= a; a ^= b;
```

**Java-specific:** `int` is 32 bits, `long` is 64 bits. Use `1L << i` for bit positions 32-63. `>>>` is unsigned right shift (fills with 0), `>>` is signed (fills with sign bit).

---

## 1. XOR Properties

### When to Use
- "Find the **single/missing** number"
- "Find **two unique** numbers" in array where others appear twice
- "**Complement** of a number"
- Pair cancellation: XOR of identical values = 0

### When NOT to Use
- Elements appear more than twice — XOR alone isn't enough (use modular bit counting)
- Need to find the actual duplicates, not just the missing one

### Key XOR Properties
```
a ^ a = 0          (self-cancel)
a ^ 0 = a          (identity)
a ^ b = b ^ a      (commutative)
(a ^ b) ^ c = a ^ (b ^ c)  (associative)
```

### Optimized Java Template

```java
// Single Number: every element appears twice except one
public int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) result ^= num;
    return result;
}

// Single Number III: two unique numbers, rest appear twice
public int[] singleNumberIII(int[] nums) {
    int xorAll = 0;
    for (int n : nums) xorAll ^= n;  // xorAll = a ^ b

    // Find any bit where a and b differ
    int diffBit = xorAll & (-xorAll);  // isolate lowest set bit

    int a = 0, b = 0;
    for (int n : nums) {
        if ((n & diffBit) == 0) a ^= n;  // group 1: bit not set
        else b ^= n;                       // group 2: bit set
    }

    return new int[]{a, b};
}

// Single Number II: every element appears THREE times except one
public int singleNumberII(int[] nums) {
    int ones = 0, twos = 0;
    for (int n : nums) {
        ones = (ones ^ n) & ~twos;
        twos = (twos ^ n) & ~ones;
    }
    return ones;
}

// Missing Number: [0, n] with one missing
public int missingNumber(int[] nums) {
    int xor = nums.length;
    for (int i = 0; i < nums.length; i++) {
        xor ^= i ^ nums[i];
    }
    return xor;
}
```

### Fully Solved Problem: Single Number III (LC 260)

**Problem:** Array where every element appears exactly twice except for two elements. Find the two unique elements.

**Thinking process:**
1. XOR all elements → gives `a ^ b` (all pairs cancel)
2. Since `a != b`, at least one bit in `a ^ b` is 1 — this bit differs between a and b
3. Use that bit to partition all numbers into two groups
4. XOR each group separately → each group contains exactly one unique number

```java
public int[] singleNumber(int[] nums) {
    int xorAll = 0;
    for (int n : nums) xorAll ^= n;

    int diffBit = xorAll & (-xorAll);

    int a = 0, b = 0;
    for (int n : nums) {
        if ((n & diffBit) == 0) a ^= n;
        else b ^= n;
    }
    return new int[]{a, b};
}
```

**Trace with `[1, 2, 1, 3, 2, 5]`:**
- xorAll = 1^2^1^3^2^5 = 3^5 = 6 (binary: 110)
- diffBit = 6 & -6 = 2 (binary: 010)
- Group 1 (bit 1 = 0): {1, 1, 5} → XOR = 5
- Group 2 (bit 1 = 1): {2, 3, 2} → XOR = 3
- Answer: [5, 3]

**Complexity:** O(n) time, O(1) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using XOR for three-occurrence problem | Wrong answer — XOR only cancels pairs | Use `ones`/`twos` state machine (Single Number II) |
| `xorAll & (xorAll - 1)` instead of `& (-xorAll)` | Gets all bits except lowest — wrong operation | `n & (-n)` isolates lowest set bit |
| Missing Number: forgetting to XOR with `n` | Off by one — `n` itself might be missing | Initialize `xor = nums.length` |
| Integer overflow with large XOR results | Not an issue for XOR | XOR doesn't overflow — result is always within int range |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "What if elements appear k times with one appearing once?" | "For k=3: state machine with `ones`/`twos`. General k: count each bit position mod k." |
| "Can XOR find the duplicate in [1,n] with one dup?" | "Yes — XOR all elements and XOR with 1..n. Same as missing number but finds the extra." |
| "How does `n & (-n)` work?" | "In two's complement, `-n = ~n + 1`. This flips all bits up to the lowest set bit, so AND isolates just that bit." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Single Number | Easy | XOR all elements | 136 |
| 2 | Single Number II | Medium | Bit counting mod 3 or state machine | 137 |
| 3 | Single Number III | Medium | XOR all → split by differing bit | 260 |
| 4 | Missing Number | Easy | XOR with indices and n | 268 |
| 5 | Complement of Base 10 Integer | Easy | XOR with all-1s mask of same length | 1009 |

---

## 2. Bit Masking

### When to Use
- "**Subsets** enumeration" (alternative to backtracking)
- "**DP with bitmask** state" (visited set, chosen items)
- "**Counting bits**" / "**Hamming distance**"
- When state can be encoded as a set of yes/no decisions

### When NOT to Use
- More than ~20 elements — `2^n` states won't fit in memory/time
- State requires more than presence/absence (e.g., counts)

### Optimized Java Template

```java
// Enumerate all subsets of a set of size n
public List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();

    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}

// Enumerate all submasks of a given mask
// Total iterations across ALL masks: 3^n (each element: in mask+in submask, in mask+not in submask, not in mask)
void enumerateSubmasks(int mask) {
    for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
        // process submask 'sub'
    }
    // don't forget sub = 0 (empty subset) if needed
}

// Counting Bits: dp[i] = number of 1s in binary representation of i
public int[] countBits(int n) {
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
        // OR: dp[i] = dp[i & (i-1)] + 1  (clear lowest bit)
    }
    return dp;
}

// Hamming Distance between two integers
public int hammingDistance(int x, int y) {
    return Integer.bitCount(x ^ y);
}

// Total Hamming Distance across all pairs
public int totalHammingDistance(int[] nums) {
    int total = 0, n = nums.length;
    for (int bit = 0; bit < 32; bit++) {
        int ones = 0;
        for (int num : nums) {
            ones += (num >> bit) & 1;
        }
        total += ones * (n - ones);  // pairs with different bit at this position
    }
    return total;
}
```

### Fully Solved Problem: Partition to K Equal Sum Subsets (LC 698)

**Problem:** Given an array, partition it into K subsets of equal sum.

**Thinking process:**
1. Total sum must be divisible by K, each subset has target = total/K
2. State = which elements have been used → bitmask
3. For each state, try adding each unused element to the current bucket
4. When a bucket reaches target, start a new bucket

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int sum = 0;
    for (int n : nums) sum += n;
    if (sum % k != 0) return false;

    int target = sum / k;
    int n = nums.length;
    boolean[] dp = new boolean[1 << n];
    int[] currentSum = new int[1 << n];
    dp[0] = true;

    Arrays.sort(nums);  // optimization: try large elements first (fail fast)

    for (int mask = 0; mask < (1 << n); mask++) {
        if (!dp[mask]) continue;

        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) continue;  // already used

            int nextMask = mask | (1 << i);
            if (dp[nextMask]) continue;  // already reached this state

            int bucketSum = currentSum[mask] % target;  // current bucket's sum
            if (bucketSum + nums[i] <= target) {
                dp[nextMask] = true;
                currentSum[nextMask] = currentSum[mask] + nums[i];
            }
        }
    }

    return dp[(1 << n) - 1];
}
```

**Why `currentSum[mask] % target`?** When a bucket fills to exactly `target`, the modulo resets to 0 and we start filling the next bucket. This elegantly handles the transition between buckets.

**Complexity:** O(n * 2^n) time, O(2^n) space. Works for n ≤ 20.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using `1 << i` with i >= 31 | Undefined behavior for `int` | Use `1L << i` for long, or ensure i < 31 |
| Bitmask DP with n > 20 | 2^20 = 1M states OK, 2^25 = 33M → MLE | Only use bitmask when n ≤ 20 |
| Not sorting for early pruning | Explores hopeless branches | Sort descending; if largest element > target, return false |
| Submask enumeration: missing empty set | Off-by-one | The loop `sub = (sub-1) & mask` stops before 0 |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Why bitmask instead of backtracking?" | "Bitmask DP with memoization avoids revisiting states. Backtracking can explore the same combination of used elements via different orderings." |
| "What's the limit on n for bitmask?" | "n ≤ 20 is safe (2^20 ≈ 1M). n = 25 might work with optimizations. n > 25 is too much." |
| "How to enumerate submasks efficiently?" | "`for (sub = mask; sub > 0; sub = (sub-1) & mask)`. Total across all masks of an n-element set: 3^n." |
| "Counting bits: explain the DP?" | "`dp[i] = dp[i>>1] + (i&1)`. Right-shifting removes the last bit (already counted), then add back if the last bit was 1." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Counting Bits | Easy | DP: `dp[i] = dp[i>>1] + (i&1)` | 338 |
| 2 | Hamming Distance | Easy | XOR + bitCount | 461 |
| 3 | Total Hamming Distance | Medium | Count 1s per bit position: `ones * zeros` | 477 |
| 4 | Subsets (bitmask approach) | Medium | Enumerate 0 to 2^n-1 | 78 |
| 5 | Partition to K Equal Sum Subsets | Medium | Bitmask DP with bucket sum tracking | 698 |
| 6 | Shortest Path Visiting All Nodes | Hard | BFS with bitmask state (node, visited_mask) | 847 |
| 7 | Maximum Students Taking Exam | Hard | Bitmask DP on rows with seat constraints | 1349 |

---

## Bit Manipulation Cheat Sheet

| Operation | Code | Use Case |
|-----------|------|----------|
| Check bit i | `(n >> i) & 1` | Query state |
| Set bit i | `n \| (1 << i)` | Add to set |
| Clear bit i | `n & ~(1 << i)` | Remove from set |
| Toggle bit i | `n ^ (1 << i)` | Flip state |
| Lowest set bit | `n & (-n)` | Find minimum element |
| Clear lowest bit | `n & (n-1)` | Count bits, power-of-2 check |
| All bits set (k bits) | `(1 << k) - 1` | Full mask |
| Population count | `Integer.bitCount(n)` | Subset size |
| Is power of 2? | `n > 0 && (n & (n-1)) == 0` | Special structure check |
| Iterate submasks | `for(s=m; s>0; s=(s-1)&m)` | Subset enumeration |

---

## Cross-References

- **Bitmask subsets** alternative to backtracking — see [Backtracking](./08-backtracking.md#3-subsets)
- **Bitmask DP** states — see [Dynamic Programming](./07-dynamic-programming.md#9-bitmask-dp)
- **XOR Trie** for max XOR queries — see [Tries](./12-tries.md#2-xor-trie)
- **Shortest Path Visiting All Nodes** combines BFS + bitmask — see [Graphs](./06-graphs.md)
