# 14 — Advanced Patterns

These patterns appear in hard-tier interviews (Google L5+, Meta E5+) and competitive programming. Segment trees and BITs handle range queries; KMP and Rabin-Karp handle string matching. Knowing these separates "strong" from "exceptional" candidates.

---

## Table of Contents
1. [Segment Tree (Iterative)](#1-segment-tree-iterative)
2. [Binary Indexed Tree (Fenwick Tree)](#2-binary-indexed-tree-fenwick-tree)
3. [KMP (Knuth-Morris-Pratt)](#3-kmp-knuth-morris-pratt)
4. [Rabin-Karp (Rolling Hash)](#4-rabin-karp-rolling-hash)

---

## 1. Segment Tree (Iterative)

### When to Use
- "**Range sum/min/max query**" with **point updates**
- "**Range update** with lazy propagation"
- "**Count of elements** in a range" (with coordinate compression)
- Any problem needing O(log n) queries and updates on array ranges

### When NOT to Use
- Static array, no updates — use prefix sums (O(1) query)
- Only point queries — plain array suffices
- Range updates but only final state needed — difference array is simpler

### Optimized Java Template

```java
// Iterative segment tree — 2n space (vs recursive 4n), simpler, faster
class SegTree {
    int[] tree;
    int n;

    public SegTree(int[] arr) {
        n = arr.length;
        tree = new int[2 * n];

        // Build: copy leaves, then build parents bottom-up
        System.arraycopy(arr, 0, tree, n, n);
        for (int i = n - 1; i > 0; i--) {
            tree[i] = tree[2 * i] + tree[2 * i + 1];  // sum — change for min/max
        }
    }

    // Point update: set arr[pos] = val
    public void update(int pos, int val) {
        pos += n;  // shift to leaf position
        tree[pos] = val;
        while (pos > 1) {
            pos >>= 1;
            tree[pos] = tree[2 * pos] + tree[2 * pos + 1];
        }
    }

    // Range query: sum of [left, right) — half-open interval
    public int query(int left, int right) {
        int sum = 0;
        left += n;
        right += n;
        while (left < right) {
            if ((left & 1) == 1) sum += tree[left++];   // left is right child
            if ((right & 1) == 1) sum += tree[--right];  // right is right child
            left >>= 1;
            right >>= 1;
        }
        return sum;
    }
}

// Min variant — just change the merge operation
class MinSegTree {
    int[] tree;
    int n;

    public MinSegTree(int[] arr) {
        n = arr.length;
        tree = new int[2 * n];
        Arrays.fill(tree, Integer.MAX_VALUE);
        System.arraycopy(arr, 0, tree, n, n);
        for (int i = n - 1; i > 0; i--) {
            tree[i] = Math.min(tree[2 * i], tree[2 * i + 1]);
        }
    }

    public void update(int pos, int val) {
        pos += n;
        tree[pos] = val;
        while (pos > 1) {
            pos >>= 1;
            tree[pos] = Math.min(tree[2 * pos], tree[2 * pos + 1]);
        }
    }

    public int query(int left, int right) {
        int min = Integer.MAX_VALUE;
        left += n; right += n;
        while (left < right) {
            if ((left & 1) == 1) min = Math.min(min, tree[left++]);
            if ((right & 1) == 1) min = Math.min(min, tree[--right]);
            left >>= 1; right >>= 1;
        }
        return min;
    }
}
```

**Why iterative over recursive?**
- **2n space** vs 4n for recursive (half the memory)
- No function call overhead (no stack frames)
- Simpler to implement correctly in an interview
- Same O(log n) complexity for both query and update

### Fully Solved Problem: Range Sum Query — Mutable (LC 307)

**Problem:** Given an array, support `update(index, val)` and `sumRange(left, right)`.

```java
class NumArray {
    int[] tree;
    int n;

    public NumArray(int[] nums) {
        n = nums.length;
        tree = new int[2 * n];
        System.arraycopy(nums, 0, tree, n, n);
        for (int i = n - 1; i > 0; i--) {
            tree[i] = tree[2 * i] + tree[2 * i + 1];
        }
    }

    public void update(int index, int val) {
        index += n;
        tree[index] = val;
        while (index > 1) {
            index >>= 1;
            tree[index] = tree[2 * index] + tree[2 * index + 1];
        }
    }

    public int sumRange(int left, int right) {
        int sum = 0;
        left += n;
        right += n + 1;  // convert to half-open [left, right+1)
        while (left < right) {
            if ((left & 1) == 1) sum += tree[left++];
            if ((right & 1) == 1) sum += tree[--right];
            left >>= 1;
            right >>= 1;
        }
        return sum;
    }
}
```

**Complexity:** Build O(n), Update O(log n), Query O(log n), Space O(n).

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Recursive segment tree in interview | 4x space, harder to debug | Use iterative (2n array) |
| Closed vs half-open intervals | Off-by-one in queries | Iterative tree uses half-open `[left, right)` |
| Not building bottom-up after leaves | Parent nodes uninitialized | Loop `for (i = n-1; i > 0; i--)` |
| Using segment tree when prefix sum works | Over-engineering, wastes interview time | Only use segment tree when there are UPDATE operations |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Range Sum Query - Mutable | Medium | Iterative segment tree | 307 |
| 2 | Count of Smaller Numbers After Self | Hard | BIT or segment tree with coordinate compression | 315 |
| 3 | Count of Range Sum | Hard | Merge sort or segment tree | 327 |
| 4 | Falling Squares | Hard | Segment tree with lazy propagation | 699 |

---

## 2. Binary Indexed Tree (Fenwick Tree)

### When to Use
- **Prefix sum queries** with **point updates** — simpler than segment tree
- "**Count inversions**", "**count smaller after self**"
- When you only need prefix operations (sum from 0 to i), not arbitrary ranges

### When NOT to Use
- Need range minimum/maximum — BIT can't do this (use segment tree)
- Need arbitrary range updates — BIT is for point updates + prefix queries
- Static array — prefix sum array is simpler

### Optimized Java Template

```java
// BIT: 1-indexed, supports point update and prefix sum
class BIT {
    int[] tree;
    int n;

    public BIT(int n) {
        this.n = n;
        tree = new int[n + 1];  // 1-indexed
    }

    // Add delta to position i (1-indexed)
    public void update(int i, int delta) {
        for (; i <= n; i += i & (-i)) {
            tree[i] += delta;
        }
    }

    // Prefix sum: sum of [1, i]
    public int query(int i) {
        int sum = 0;
        for (; i > 0; i -= i & (-i)) {
            sum += tree[i];
        }
        return sum;
    }

    // Range sum: [left, right] (1-indexed)
    public int rangeQuery(int left, int right) {
        return query(right) - query(left - 1);
    }
}
```

**How `i & (-i)` works:** In two's complement, `-i` flips all bits and adds 1. `i & (-i)` isolates the lowest set bit of `i`. This bit determines the range of responsibility for each BIT node.

**BIT vs Segment Tree:**
| Feature | BIT | Segment Tree |
|---------|-----|-------------|
| Space | n+1 | 2n |
| Code complexity | ~15 lines | ~40 lines |
| Point update + prefix query | O(log n) | O(log n) |
| Arbitrary range query | Via prefix difference | Direct |
| Range min/max | Not supported | Supported |
| Range update + point query | Supported (with trick) | Supported |
| Range update + range query | Not supported | With lazy propagation |

### Fully Solved Problem: Count of Smaller Numbers After Self (LC 315)

**Problem:** For each element, count how many elements to its right are smaller.

**Thinking process:**
1. Process right to left — for each element, count how many already-seen elements are smaller
2. "Count of elements < x" = prefix sum query on a frequency array
3. Use BIT indexed by value (with coordinate compression for large values)

```java
public List<Integer> countSmaller(int[] nums) {
    // Coordinate compression
    int[] sorted = nums.clone();
    Arrays.sort(sorted);
    Map<Integer, Integer> rank = new HashMap<>();
    int r = 1;
    for (int num : sorted) {
        if (!rank.containsKey(num)) rank.put(num, r++);
    }

    BIT bit = new BIT(rank.size());
    Integer[] result = new Integer[nums.length];

    for (int i = nums.length - 1; i >= 0; i--) {
        int rnk = rank.get(nums[i]);
        result[i] = bit.query(rnk - 1);  // count elements with rank < current
        bit.update(rnk, 1);               // add current element
    }

    return Arrays.asList(result);
}

class BIT {
    int[] tree;
    int n;

    BIT(int n) { this.n = n; tree = new int[n + 1]; }

    void update(int i, int delta) {
        for (; i <= n; i += i & (-i)) tree[i] += delta;
    }

    int query(int i) {
        int sum = 0;
        for (; i > 0; i -= i & (-i)) sum += tree[i];
        return sum;
    }
}
```

**Complexity:** O(n log n) time (coordinate compression + n BIT operations), O(n) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| 0-indexed BIT | `i & (-i)` when i=0 is 0 → infinite loop | BIT must be 1-indexed |
| Forgetting coordinate compression | Array too large for BIT (values up to 10^9) | Map values to ranks 1..n |
| Using BIT for range min/max | BIT only supports prefix sums | Use segment tree instead |
| Off-by-one in prefix difference | `rangeSum(l, r) = query(r) - query(l)` wrong | `query(r) - query(l-1)` |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Range Sum Query - Mutable | Medium | BIT or segment tree | 307 |
| 2 | Count of Smaller Numbers After Self | Hard | BIT with coordinate compression | 315 |
| 3 | Reverse Pairs | Hard | BIT or merge sort | 493 |
| 4 | Create Sorted Array through Instructions | Hard | BIT: count < and > current | 1649 |

---

## 3. KMP (Knuth-Morris-Pratt)

### When to Use
- "**Find pattern** in text" in O(n + m)
- "**Shortest palindrome**" (reverse + KMP failure function)
- "**Repeated substring**" detection
- Any exact string matching where brute force O(nm) is too slow

### When NOT to Use
- Multiple pattern matching — use Aho-Corasick or Trie
- Approximate matching — use edit distance DP
- Simple `contains` check — `String.indexOf()` uses a heuristic that's often fast enough

### Optimized Java Template

```java
// Build KMP failure function (also called "partial match" or "prefix" table)
// lps[i] = length of longest proper prefix of pattern[0..i] that is also a suffix
public int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0;  // length of previous longest prefix-suffix
    int i = 1;

    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1];  // don't increment i — retry with shorter prefix
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}

// KMP search: find all occurrences of pattern in text
public List<Integer> kmpSearch(String text, String pattern) {
    int[] lps = buildLPS(pattern);
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    int i = 0, j = 0;

    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++; j++;
            if (j == m) {
                matches.add(i - m);  // found match at index i-m
                j = lps[j - 1];     // continue searching
            }
        } else if (j > 0) {
            j = lps[j - 1];  // use failure function
        } else {
            i++;
        }
    }
    return matches;
}
```

**Why KMP over brute force?** When a mismatch occurs, brute force restarts from the next position. KMP uses the failure function to skip positions that are guaranteed to not match, achieving O(n + m) total.

### Fully Solved Problem: Shortest Palindrome (LC 214)

**Problem:** Add characters to the front of string `s` to make it a palindrome. Return the shortest such palindrome.

**Thinking process:**
1. Find the longest palindromic prefix of `s`
2. Reverse the remaining suffix and prepend it
3. How to find the longest palindromic prefix? Create `s + "#" + reverse(s)` and compute the KMP failure function
4. The last value of the failure function gives the length of the longest palindromic prefix

```java
public String shortestPalindrome(String s) {
    if (s.length() <= 1) return s;

    String rev = new StringBuilder(s).reverse().toString();
    String combined = s + "#" + rev;  // "#" prevents cross-boundary matches

    int[] lps = buildLPS(combined);

    int palindromeLen = lps[combined.length() - 1];

    return rev.substring(0, s.length() - palindromeLen) + s;
}

private int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0, i = 1;
    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1];
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

**Trace with `s = "aacecaaa"`:**
- rev = "aaacecaa"
- combined = "aacecaaa#aaacecaa"
- lps of combined: `[0,1,0,0,0,1,2,2,0,1,2,2,3,4,5,6,7]`
- lps[last] = 7 → longest palindromic prefix has length 7 ("aacecaa")
- Prepend rev[0..0] = "a" → "aaacecaaa"

**Complexity:** O(n) time, O(n) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Missing separator in combined string | Cross-boundary false matches | Use `"#"` (or any char not in alphabet) between s and rev(s) |
| Wrong LPS construction: incrementing i on mismatch with len>0 | LPS values wrong, search breaks | Only increment i when match or len==0 |
| Confusing "proper" prefix | Full string is not a proper prefix of itself | LPS of full string = 0 by convention |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Implement strStr() | Easy | KMP or built-in (but know KMP for follow-ups) | 28 |
| 2 | Shortest Palindrome | Hard | KMP failure function on s + "#" + rev(s) | 214 |
| 3 | Repeated Substring Pattern | Easy | KMP: if `n % (n - lps[n-1]) == 0`, repeated | 459 |

---

## 4. Rabin-Karp (Rolling Hash)

### When to Use
- "**Find duplicate substrings**" of given length
- "**Longest duplicate substring**" (binary search + rolling hash)
- Multiple pattern matching of same length
- Any problem where you need to compare many substrings efficiently

### When NOT to Use
- Single pattern search — KMP is deterministic and simpler
- Very short patterns — brute force with `indexOf` is fine
- Need guaranteed correctness — rolling hash has collision risk (use double hash)

### Optimized Java Template

```java
// Rolling hash with modular arithmetic
class RollingHash {
    static final long MOD = (1L << 61) - 1;  // Mersenne prime — fewer collisions
    static final long BASE = 131;  // prime base

    // Compute hash of s[0..len-1]
    static long hash(String s, int len) {
        long h = 0;
        for (int i = 0; i < len; i++) {
            h = mod(mul(h, BASE) + s.charAt(i));
        }
        return h;
    }

    // Rolling: remove s[i-1], add s[i+len-1]
    // newHash = (oldHash - s[i-1] * BASE^(len-1)) * BASE + s[i+len-1]
    static long roll(long hash, char remove, char add, long basePow) {
        hash = mod(hash - mul(remove, basePow) + MOD);
        hash = mod(mul(hash, BASE) + add);
        return hash;
    }

    static long power(long base, long exp) {
        long result = 1;
        base = mod(base);
        while (exp > 0) {
            if ((exp & 1) == 1) result = mul(result, base);
            base = mul(base, base);
            exp >>= 1;
        }
        return result;
    }

    // Mersenne prime modular arithmetic (avoids slow % operator)
    static long mod(long a) {
        a = (a >> 61) + (a & MOD);
        return a >= MOD ? a - MOD : a;
    }

    static long mul(long a, long b) {
        long hi = Math.multiplyHigh(a, b);
        long lo = a * b;
        return mod((hi << 3) | (lo >>> 61)) + mod(lo & MOD);
    }
}

// Simple version (for interviews — easier to write)
public boolean hasSubstringOfLength(String s, int len) {
    long MOD = 1_000_000_007L;
    long BASE = 26;
    long hash = 0, power = 1;

    // Precompute BASE^(len-1)
    for (int i = 0; i < len - 1; i++) power = power * BASE % MOD;

    // Initial hash
    for (int i = 0; i < len; i++) {
        hash = (hash * BASE + (s.charAt(i) - 'a')) % MOD;
    }

    Set<Long> seen = new HashSet<>();
    seen.add(hash);

    for (int i = len; i < s.length(); i++) {
        hash = (hash - (s.charAt(i - len) - 'a') * power % MOD + MOD) % MOD;
        hash = (hash * BASE + (s.charAt(i) - 'a')) % MOD;
        if (!seen.add(hash)) return true;  // duplicate hash found
    }

    return false;
}
```

### Fully Solved Problem: Longest Duplicate Substring (LC 1044)

**Problem:** Find the longest substring that appears at least twice.

**Thinking process:**
1. Binary search on the answer length — if a duplicate of length L exists, one of length L-1 also exists
2. For each candidate length, use rolling hash to check for duplicates
3. If hash collision, verify with actual string comparison (to handle false positives)

```java
public String longestDupSubstring(String s) {
    int lo = 1, hi = s.length() - 1;
    String result = "";

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        String dup = findDuplicate(s, mid);
        if (dup != null) {
            result = dup;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }

    return result;
}

private String findDuplicate(String s, int len) {
    long MOD = (1L << 61) - 1;
    long BASE = 131;
    long hash = 0, power = 1;

    for (int i = 0; i < len - 1; i++) power = modMul(power, BASE, MOD);
    for (int i = 0; i < len; i++) hash = modAdd(modMul(hash, BASE, MOD), s.charAt(i), MOD);

    Map<Long, List<Integer>> seen = new HashMap<>();
    seen.computeIfAbsent(hash, k -> new ArrayList<>()).add(0);

    for (int i = len; i < s.length(); i++) {
        hash = modAdd(modMul(
            modAdd(hash, MOD - modMul(s.charAt(i - len), power, MOD), MOD),
            BASE, MOD), s.charAt(i), MOD);
        int start = i - len + 1;

        List<Integer> positions = seen.get(hash);
        if (positions != null) {
            String candidate = s.substring(start, start + len);
            for (int pos : positions) {
                if (s.substring(pos, pos + len).equals(candidate)) {
                    return candidate;
                }
            }
        }
        seen.computeIfAbsent(hash, k -> new ArrayList<>()).add(start);
    }

    return null;
}

private long modMul(long a, long b, long mod) {
    return Math.floorMod(a * b, mod);  // safe for values < 2^62
}

private long modAdd(long a, long b, long mod) {
    return (a + b) % mod;
}
```

**Complexity:** O(n log n) average (binary search * rolling hash), O(n²) worst case with many collisions.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Single hash → false positives | Report non-existent duplicates | Verify with actual string comparison, or use double hashing |
| Small modulus (e.g., 10^7) | Too many collisions | Use 10^9+7 or Mersenne prime 2^61-1 |
| Negative hash after subtraction | Negative modulo in Java | Add MOD before taking %: `(hash - x + MOD) % MOD` |
| Integer overflow in hash computation | Wrong hash values | Use `long` everywhere, be careful with multiplication |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "How to handle hash collisions?" | "Verify with actual string comparison. Or use two different hash functions (double hashing) for near-zero collision probability." |
| "Time complexity of Rabin-Karp?" | "O(n + m) average, O(nm) worst case (many collisions). With good hash: practically O(n + m)." |
| "Why binary search works for longest duplicate substring?" | "Monotonicity: if duplicate of length L exists, duplicate of length L-1 must exist (any prefix of the duplicate works)." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Implement strStr() | Easy | Rabin-Karp or KMP | 28 |
| 2 | Longest Duplicate Substring | Hard | Binary search + rolling hash | 1044 |
| 3 | Repeated DNA Sequences | Medium | Rolling hash or bitmask (2 bits per char) | 187 |
| 4 | Longest Happy Prefix | Hard | KMP or rolling hash | 1392 |
| 5 | Distinct Echo Substrings | Hard | Rolling hash + HashSet | 1316 |

---

## Pattern Selection Guide

| Need | Data Structure | Time |
|------|---------------|------|
| Range sum + point update | BIT (simplest) or Segment Tree | O(log n) |
| Range min/max + point update | Segment Tree | O(log n) |
| Range update + range query | Segment Tree with lazy propagation | O(log n) |
| Exact string matching | KMP (deterministic) | O(n + m) |
| Multiple same-length substring search | Rabin-Karp (rolling hash) | O(n) avg |
| Longest duplicate substring | Binary search + Rabin-Karp | O(n log n) avg |

---

## Cross-References

- **Segment Tree for count inversions** connects to [Sorting](./01-arrays-and-hashing.md) (merge sort approach)
- **KMP failure function** for palindromes connects to [String Partitioning](./08-backtracking.md#5-string-partitioning)
- **Rolling hash** for duplicate detection connects to [Sliding Window](./01-arrays-and-hashing.md#3-variable-sliding-window)
- **BIT for range sums** is a lighter alternative when segment tree's min/max isn't needed
