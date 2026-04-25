# 07 — Dynamic Programming

DP is the most feared topic in interviews. The secret: DP is just recursion + memoization. Every DP problem has TWO properties: **optimal substructure** (the optimal solution uses optimal solutions of subproblems) and **overlapping subproblems** (same subproblem is solved multiple times). If both exist, use DP.

---

## Table of Contents
1. [1D Linear DP](#1-1d-linear-dp)
2. [2D Grid DP](#2-2d-grid-dp)
3. [Two Sequences DP (LCS Family)](#3-two-sequences-dp-lcs-family)
4. [0/1 Knapsack](#4-01-knapsack)
5. [Unbounded Knapsack](#5-unbounded-knapsack)
6. [Palindrome DP](#6-palindrome-dp)
7. [State Machine DP (Buy/Sell Stock)](#7-state-machine-dp-buysell-stock)
8. [Interval DP](#8-interval-dp)
9. [Bitmask DP](#9-bitmask-dp)
10. [Digit DP](#10-digit-dp)

---

## 1. 1D Linear DP

### When to Use
- Decision at position `i` depends on previous positions
- "**Climbing stairs**", "**house robber**", "**decode ways**"
- Single sequence, counting ways, or optimizing a value

### Optimized Java Template

```java
// Template: Space-optimized 1D DP (rolling variables)
// When dp[i] depends only on dp[i-1] and dp[i-2]
public int solve(int[] nums) {
    int prev2 = /* base case dp[0] */;
    int prev1 = /* base case dp[1] */;

    for (int i = 2; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);  // recurrence
        prev2 = prev1;
        prev1 = curr;
    }

    return prev1;
}
```

### Fully Solved Problem: House Robber (LC 198)

**Problem:** Rob houses along a street. Can't rob two adjacent houses. Maximize total.

**Thinking Process:**
1. At each house `i`, two choices: rob it (add `nums[i]` to what we had at `i-2`) or skip it (keep what we had at `i-1`).
2. `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`
3. Only depends on two previous values → optimize to O(1) space.

**Solution:**
```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];

    int prev2 = 0, prev1 = 0;

    for (int num : nums) {
        int curr = Math.max(prev1, prev2 + num);
        prev2 = prev1;
        prev1 = curr;
    }

    return prev1;
}
```

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Wrong base cases | Everything cascades from bad initial values | Think carefully: what does dp[0] and dp[1] mean? Verify with small examples. |
| Not considering dp[i] depending on dp[i-1] ONLY vs dp[i-1] AND dp[i-2] | Using rolling variables when you need the full array | Only optimize space when you've confirmed the dependency window |
| House Robber II (circular) | First and last houses are adjacent | Run twice: houses [0, n-2] and [1, n-1], take max |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Climbing Stairs | Easy | dp[i] = dp[i-1] + dp[i-2] (Fibonacci) | 70 |
| 2 | Min Cost Climbing Stairs | Easy | dp[i] = min(dp[i-1], dp[i-2]) + cost[i] | 746 |
| 3 | House Robber | Medium | Solved above | 198 |
| 4 | House Robber II | Medium | Circular: max(rob[0..n-2], rob[1..n-1]) | 213 |
| 5 | Decode Ways | Medium | dp[i] depends on single digit and two-digit validity | 91 |
| 6 | Longest Increasing Subsequence | Medium | O(n²) DP or O(n log n) with patience sorting | 300 |
| 7 | Word Break | Medium | dp[i] = any dp[j] && s[j..i] in dict | 139 |
| 8 | Jump Game | Medium | Greedy: track maxReach | 55 |

---

## 2. 2D Grid DP

### When to Use
- "**Unique paths**" in a grid
- "**Minimum path sum**"
- Movement is restricted (only right/down, no backtracking)

### When NOT to Use
- Can move in all 4 directions → BFS/DFS, not DP (DP needs acyclic dependencies)

### Optimized Java Template

```java
// Template: Rolling array optimization (2D → 1D space)
public int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[] dp = new int[n];

    dp[0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[j] = dp[j - 1] + grid[0][j];  // first row

    for (int i = 1; i < m; i++) {
        dp[0] += grid[i][0];  // first column
        for (int j = 1; j < n; j++) {
            dp[j] = Math.min(dp[j], dp[j - 1]) + grid[i][j];
            // dp[j] (before update) = value from row above
            // dp[j-1] (after update) = value from left in current row
        }
    }

    return dp[n - 1];
}
```

**Why rolling array works:** `dp[i][j]` only depends on `dp[i-1][j]` (above) and `dp[i][j-1]` (left). We reuse a single row: `dp[j]` from the previous iteration IS `dp[i-1][j]`.

### Fully Solved Problem: Maximal Square (LC 221)

**Problem:** Find the largest square containing only 1's in a binary matrix.

**Key insight:** `dp[i][j]` = side length of largest square with bottom-right corner at `(i, j)`.

**Recurrence:** `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` (if `matrix[i][j] == '1'`).

```java
public int maximalSquare(char[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[] dp = new int[n + 1];
    int maxSide = 0, prev = 0;  // prev = dp[i-1][j-1]

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            int temp = dp[j];
            if (matrix[i - 1][j - 1] == '1') {
                dp[j] = Math.min(Math.min(dp[j], dp[j - 1]), prev) + 1;
                maxSide = Math.max(maxSide, dp[j]);
            } else {
                dp[j] = 0;
            }
            prev = temp;
        }
        prev = 0;
    }

    return maxSide * maxSide;
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Unique Paths | Medium | dp[i][j] = dp[i-1][j] + dp[i][j-1] | 62 |
| 2 | Unique Paths II | Medium | Set dp = 0 for obstacles | 63 |
| 3 | Minimum Path Sum | Medium | dp[i][j] = min(up, left) + grid[i][j] | 64 |
| 4 | Triangle | Medium | Bottom-up: dp[j] = min(dp[j], dp[j+1]) + tri[i][j] | 120 |
| 5 | Maximal Square | Medium | Solved above | 221 |
| 6 | Dungeon Game | Hard | DP from bottom-right to top-left | 174 |

---

## 3. Two Sequences DP (LCS Family)

### When to Use
- Two strings or arrays, find optimal alignment
- "**Longest Common Subsequence**"
- "**Edit Distance**"
- "**Interleaving String**"

### Optimized Java Template

```java
// Template: Two Sequences with rolling array
public int lcs(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[] dp = new int[n + 1];

    for (int i = 1; i <= m; i++) {
        int prev = 0;  // dp[i-1][j-1]
        for (int j = 1; j <= n; j++) {
            int temp = dp[j];
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[j] = prev + 1;
            } else {
                dp[j] = Math.max(dp[j], dp[j - 1]);
            }
            prev = temp;
        }
    }

    return dp[n];
}
```

### Fully Solved Problem: Edit Distance (LC 72)

**Problem:** Find the minimum number of operations (insert, delete, replace) to convert `word1` to `word2`.

**Recurrence:**
- If `word1[i] == word2[j]`: `dp[i][j] = dp[i-1][j-1]` (no operation needed)
- Else: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])` (delete, insert, replace)

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[] dp = new int[n + 1];

    for (int j = 0; j <= n; j++) dp[j] = j;  // base: converting "" to word2[0..j]

    for (int i = 1; i <= m; i++) {
        int prev = dp[0];
        dp[0] = i;  // base: converting word1[0..i] to ""
        for (int j = 1; j <= n; j++) {
            int temp = dp[j];
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[j] = prev;
            } else {
                dp[j] = 1 + Math.min(Math.min(dp[j], dp[j - 1]), prev);
            }
            prev = temp;
        }
    }

    return dp[n];
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Longest Common Subsequence | Medium | Classic 2D DP | 1143 |
| 2 | Delete Operation for Two Strings | Medium | LCS variant: answer = m + n - 2*LCS | 583 |
| 3 | Edit Distance | Hard | Solved above | 72 |
| 4 | Distinct Subsequences | Hard | Count ways s contains t as subsequence | 115 |
| 5 | Regular Expression Matching | Hard | dp[i][j] with '*' handling two cases | 10 |
| 6 | Wildcard Matching | Hard | Similar to regex but simpler '*' semantics | 44 |

---

## 4. 0/1 Knapsack

### When to Use
- "**Partition** equal subset sum"
- "**Target sum**" with +/- choices
- Each item can be **used at most once**
- "Can we select a subset summing to X?"

### Optimized Java Template

```java
// 0/1 Knapsack with 1D array — REVERSE iteration is critical
public boolean canPartition(int[] nums) {
    int total = 0;
    for (int n : nums) total += n;
    if (total % 2 != 0) return false;

    int target = total / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;

    for (int num : nums) {
        // REVERSE: each item used at most once
        for (int j = target; j >= num; j--) {
            dp[j] = dp[j] || dp[j - num];
        }
    }

    return dp[target];
}
```

**Why reverse iteration:**
- Forward iteration: `dp[j]` uses `dp[j - num]` which was ALREADY UPDATED in this round → effectively using the item multiple times (unbounded knapsack behavior).
- Reverse iteration: `dp[j]` uses `dp[j - num]` from the PREVIOUS round → each item used at most once.

**This is the single most important DP optimization to understand for interviews.**

### Fully Solved Problem: Target Sum (LC 494)

**Problem:** Given nums and a target, assign + or - to each number to reach the target. Count the number of ways.

**Key insight:** Let P = sum of positive subset, N = sum of negative subset. `P - N = target` and `P + N = total`. So `P = (target + total) / 2`. Reduce to: count subsets summing to P.

```java
public int findTargetSumWays(int[] nums, int target) {
    int total = 0;
    for (int n : nums) total += n;
    if ((total + target) % 2 != 0 || Math.abs(target) > total) return 0;

    int subsetSum = (total + target) / 2;
    int[] dp = new int[subsetSum + 1];
    dp[0] = 1;

    for (int num : nums) {
        for (int j = subsetSum; j >= num; j--) {
            dp[j] += dp[j - num];
        }
    }

    return dp[subsetSum];
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Partition Equal Subset Sum | Medium | Can we find subset summing to total/2? | 416 |
| 2 | Target Sum | Medium | Reduce to subset sum counting | 494 |
| 3 | Ones and Zeroes | Medium | 2D knapsack: capacity = (m zeros, n ones) | 474 |
| 4 | Last Stone Weight II | Medium | Minimize abs(S1 - S2) = partition into closest halves | 1049 |

---

## 5. Unbounded Knapsack

### When to Use
- Items can be **used unlimited times**
- "**Coin change**" — minimum coins / number of ways
- "**Perfect squares**"
- "**Cutting rod**"

### Optimized Java Template

```java
// Coin Change: minimum coins (FORWARD iteration — unlimited use)
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // "infinity" but avoids overflow on +1
    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}

// Coin Change II: count combinations (ORDER MATTERS for loop nesting)
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;

    // Outer loop on coins: each combination counted once
    for (int coin : coins) {
        for (int j = coin; j <= amount; j++) {
            dp[j] += dp[j - coin];
        }
    }

    return dp[amount];
}
```

**Critical distinction — loop nesting for counting:**
- **Outer loop on coins, inner on amount**: counts COMBINATIONS (order doesn't matter). `{1,2}` and `{2,1}` counted as same.
- **Outer loop on amount, inner on coins**: counts PERMUTATIONS (order matters). `{1,2}` and `{2,1}` counted separately.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Coin Change | Medium | Minimum coins (forward iteration) | 322 |
| 2 | Coin Change II | Medium | Count combinations (coins as outer loop) | 518 |
| 3 | Perfect Squares | Medium | Coin change where "coins" are 1,4,9,16,... | 279 |
| 4 | Minimum Cost For Tickets | Medium | Coins = [1, 7, 30] day tickets | 983 |

---

## 6. Palindrome DP

### When to Use
- "**Longest palindromic** substring/subsequence"
- "**Palindrome partitioning**"
- "Minimum cuts to partition into palindromes"

### Optimized Java Template

```java
// Expand Around Center — O(n²) time, O(1) space for substring
public String longestPalindrome(String s) {
    int start = 0, maxLen = 1;

    for (int i = 0; i < s.length(); i++) {
        // Odd length palindromes
        int len1 = expandAroundCenter(s, i, i);
        // Even length palindromes
        int len2 = expandAroundCenter(s, i, i + 1);

        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = i - (len - 1) / 2;
        }
    }

    return s.substring(start, start + maxLen);
}

private int expandAroundCenter(String s, int lo, int hi) {
    while (lo >= 0 && hi < s.length() && s.charAt(lo) == s.charAt(hi)) {
        lo--;
        hi++;
    }
    return hi - lo - 1;
}
```

**Why expand-around-center over DP table:**
- DP table: O(n²) time, O(n²) space.
- Expand around center: O(n²) time, O(1) space. Same time, much less space.
- For subsequence (not substring), DP table is necessary.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Longest Palindromic Substring | Medium | Expand around center, O(1) space | 5 |
| 2 | Palindromic Substrings | Medium | Count all palindromes using expand technique | 647 |
| 3 | Longest Palindromic Subsequence | Medium | 2D DP: dp[i][j] = LPS of s[i..j] | 516 |
| 4 | Palindrome Partitioning II | Hard | Precompute isPalin[][], then 1D DP for min cuts | 132 |

---

## 7. State Machine DP (Buy/Sell Stock)

### When to Use
- "**Best time to buy and sell stock**" (all variants)
- Problems with discrete states and transitions
- States: held/not-held/cooldown etc.

### Optimized Java Template

```java
// UNIFIED Stock Template: handles all variants
// States: held (holding stock), sold (just sold), reset (cooldown/ready)

// With Cooldown (LC 309)
public int maxProfit(int[] prices) {
    int held = Integer.MIN_VALUE;  // max profit while holding
    int sold = 0;                   // max profit just after selling
    int reset = 0;                  // max profit in cooldown/ready state

    for (int price : prices) {
        int prevSold = sold;
        sold = held + price;                      // sell today
        held = Math.max(held, reset - price);     // buy today or keep holding
        reset = Math.max(reset, prevSold);        // cooldown
    }

    return Math.max(sold, reset);
}

// With Transaction Fee (LC 714)
public int maxProfitWithFee(int[] prices, int fee) {
    int held = -prices[0];  // bought on day 0
    int cash = 0;           // not holding

    for (int i = 1; i < prices.length; i++) {
        held = Math.max(held, cash - prices[i]);
        cash = Math.max(cash, held + prices[i] - fee);
    }

    return cash;
}

// At Most K Transactions (LC 188) — generalized
public int maxProfitKTransactions(int k, int[] prices) {
    if (k >= prices.length / 2) {
        // Unlimited transactions — greedy
        int profit = 0;
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
        }
        return profit;
    }

    int[][] dp = new int[k + 1][2];
    for (int j = 0; j <= k; j++) dp[j][1] = Integer.MIN_VALUE;

    for (int price : prices) {
        for (int j = k; j >= 1; j--) {
            dp[j][0] = Math.max(dp[j][0], dp[j][1] + price);      // sell
            dp[j][1] = Math.max(dp[j][1], dp[j - 1][0] - price);  // buy
        }
    }

    return dp[k][0];
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Best Time to Buy and Sell Stock | Easy | Single transaction: track min price | 121 |
| 2 | Best Time to Buy and Sell Stock II | Easy | Unlimited: sum all positive diffs | 122 |
| 3 | With Cooldown | Medium | 3 states: held, sold, reset | 309 |
| 4 | With Transaction Fee | Medium | 2 states: held, cash | 714 |
| 5 | Best Time III (at most 2) | Hard | K=2 transactions DP | 123 |
| 6 | Best Time IV (at most K) | Hard | General K transactions | 188 |

---

## 8. Interval DP

### When to Use
- "**Burst balloons**"
- "**Matrix chain multiplication**"
- "Minimum cost to merge stones"
- Problems where you split/merge an interval and optimize across all split points

### Optimized Java Template

```java
// Template: Interval DP — iterate by length
// dp[i][j] = optimal value for interval [i, j]
public int intervalDP(int[] arr) {
    int n = arr.length;
    int[][] dp = new int[n][n];

    // Base case: intervals of length 1
    for (int i = 0; i < n; i++) dp[i][i] = /* base */;

    // Fill by increasing length
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;  // or MIN_VALUE for maximization

            for (int k = i; k < j; k++) {  // split point
                dp[i][j] = Math.min(dp[i][j],
                    dp[i][k] + dp[k + 1][j] + cost(i, j, k));
            }
        }
    }

    return dp[0][n - 1];
}
```

### Fully Solved Problem: Burst Balloons (LC 312)

**Key trick:** instead of thinking "which balloon to burst FIRST", think "which balloon to burst LAST" in interval `[i, j]`. The last balloon to burst in `[i, j]` is surrounded by `arr[i-1]` and `arr[j+1]` (boundaries).

```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];

    int[][] dp = new int[n + 2][n + 2];

    for (int len = 1; len <= n; len++) {
        for (int left = 1; left <= n - len + 1; left++) {
            int right = left + len - 1;
            for (int last = left; last <= right; last++) {
                int coins = arr[left - 1] * arr[last] * arr[right + 1]
                          + dp[left][last - 1] + dp[last + 1][right];
                dp[left][right] = Math.max(dp[left][right], coins);
            }
        }
    }

    return dp[1][n];
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Burst Balloons | Hard | "Last to burst" formulation | 312 |
| 2 | Minimum Cost Tree from Leaf Values | Medium | Greedy or interval DP | 1130 |
| 3 | Strange Printer | Hard | dp[i][j] = min turns to print s[i..j] | 664 |

---

## 9. Bitmask DP

### When to Use
- **n ≤ 20** (2^20 = ~10^6 states, manageable)
- "**Traveling Salesman**" / visit all nodes
- "**Shortest superstring**"
- Subset enumeration where order matters

### Optimized Java Template

```java
// Template: dp[mask][i] = optimal value for visiting set 'mask', currently at node 'i'
public int bitmaskDP(int[][] dist) {
    int n = dist.length;
    int FULL = (1 << n) - 1;
    int[][] dp = new int[1 << n][n];
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE / 2);

    // Base: start at each node
    for (int i = 0; i < n; i++) dp[1 << i][i] = 0;

    for (int mask = 1; mask <= FULL; mask++) {
        for (int u = 0; u < n; u++) {
            if ((mask & (1 << u)) == 0) continue;  // u not in mask

            for (int v = 0; v < n; v++) {
                if ((mask & (1 << v)) != 0) continue;  // v already visited

                int newMask = mask | (1 << v);
                dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }

    int ans = Integer.MAX_VALUE;
    for (int i = 0; i < n; i++) {
        ans = Math.min(ans, dp[FULL][i]);
    }
    return ans;
}
```

**Iterating over submasks of a mask:**
```java
// Enumerate all submasks of 'mask' (excluding empty set)
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process submask 'sub'
}
// Total iterations across all masks: O(3^n) — not O(4^n)
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Shortest Path Visiting All Nodes | Hard | BFS + bitmask for visited state | 847 |
| 2 | Partition to K Equal Sum Subsets | Medium | Bitmask DP or backtracking | 698 |
| 3 | Smallest Sufficient Team | Hard | Bitmask for skill coverage | 1125 |

---

## 10. Digit DP

### When to Use
- "Count numbers in **[L, R]** with property X"
- "Numbers **at most N** with given digit set"
- Properties based on digits: digit sum, no repeated digits, etc.
- **n can be huge** (10^18) but number of DIGITS is small (≤ 18)

### Optimized Java Template

```java
// Template: Digit DP with memoization
// Count numbers in [0, num] with some property
public int digitDP(String num) {
    int n = num.length();
    Integer[][][] memo = new Integer[n][/* state size */][2];
    return solve(num, 0, 0, true, memo);
}

private int solve(String num, int pos, int state, boolean tight, Integer[][][] memo) {
    if (pos == num.length()) {
        return /* check if state represents valid number */ ? 1 : 0;
    }

    int tightIdx = tight ? 1 : 0;
    if (memo[pos][state][tightIdx] != null) return memo[pos][state][tightIdx];

    int limit = tight ? (num.charAt(pos) - '0') : 9;
    int result = 0;

    for (int digit = 0; digit <= limit; digit++) {
        int newState = /* update state with digit */;
        boolean newTight = tight && (digit == limit);
        result += solve(num, pos + 1, newState, newTight, memo);
    }

    memo[pos][state][tightIdx] = result;
    return result;
}
```

**The "tight" constraint:** when `tight = true`, the digit at this position can't exceed the corresponding digit of the upper bound. Once we place a digit strictly less than the bound, all future positions are "free" (tight = false).

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Numbers At Most N Given Digit Set | Hard | Digit DP with tight constraint | 902 |
| 2 | Count Numbers with Unique Digits | Medium | Bitmask of used digits | 357 |
| 3 | Non-negative Integers without Consecutive Ones | Hard | Digit DP on binary representation | 600 |

---

## DP Problem-Solving Framework

### Step 1: Define State
- What information do I need to make a decision at each step?
- What's the minimum set of variables that capture the problem state?

### Step 2: Define Recurrence
- How does `dp[current]` relate to `dp[previous states]`?
- What are the CHOICES at each step?

### Step 3: Define Base Cases
- What are the trivial subproblems?
- Verify with the smallest possible inputs.

### Step 4: Determine Iteration Order
- Bottom-up: ensure all dependencies are computed before use.
- Top-down (memoization): recursion handles order automatically.

### Step 5: Optimize Space
- Does `dp[i]` depend only on `dp[i-1]`? → rolling variables.
- Does `dp[i][j]` depend only on the previous row? → rolling 1D array.

### When to Use Top-Down vs Bottom-Up
| Aspect | Top-Down (Memo) | Bottom-Up (Tabulation) |
|--------|-----------------|------------------------|
| Ease of coding | Easier (just add memo to recursion) | Harder (determine iteration order) |
| States computed | Only reachable states | All states |
| Stack overflow risk | Yes (deep recursion) | No |
| Space optimization | Harder | Easier (rolling array) |
| Interview recommendation | Start with top-down, optimize to bottom-up if asked |

---

## Cross-References

- **Kadane's** is a special case of [1D DP](./07-dynamic-programming.md#1-1d-linear-dp) — see [Arrays](./01-arrays-and-hashing.md#5-kadanes-algorithm)
- **Tree DP** (House Robber III) is bottom-up DFS on trees — see [Trees](./05-trees.md#2-dfs--bottom-up-postorder)
- **0/1 Knapsack** forward vs reverse connects to [Unbounded Knapsack](./07-dynamic-programming.md#5-unbounded-knapsack)
- **Bitmask DP** relies on [Bit Manipulation](./13-bit-manipulation.md) fundamentals
- **Interval DP** connects to [Greedy](./11-greedy.md) — sometimes greedy replaces DP (prove it with exchange argument)
