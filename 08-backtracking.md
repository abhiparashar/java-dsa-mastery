# 08 — Backtracking

Backtracking = DFS + pruning. Every backtracking problem follows the same skeleton: make a choice, recurse, undo the choice. Master the three variants (combinations, permutations, subsets) and you can solve any backtracking problem.

---

## Table of Contents
1. [Combinations](#1-combinations)
2. [Permutations](#2-permutations)
3. [Subsets](#3-subsets)
4. [Grid/Board Exploration](#4-gridboard-exploration)
5. [String Partitioning](#5-string-partitioning)

---

## 1. Combinations

### When to Use
- "Choose **k elements** from n"
- "**Combination sum**" — find subsets that sum to target
- "**Generate parentheses**"
- Order doesn't matter: `{1,2}` == `{2,1}`

### Optimized Java Template

```java
// Template: Combinations with start index (prevents duplicates from reordering)
public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(result, new ArrayList<>(), 1, n, k);
    return result;
}

private void backtrack(List<List<Integer>> result, List<Integer> curr,
                       int start, int n, int k) {
    if (curr.size() == k) {
        result.add(new ArrayList<>(curr));  // COPY — curr is shared across calls
        return;
    }

    // Pruning: need (k - curr.size()) more elements, so i can go up to n - remaining + 1
    for (int i = start; i <= n - (k - curr.size()) + 1; i++) {
        curr.add(i);
        backtrack(result, curr, i + 1, n, k);
        curr.remove(curr.size() - 1);  // undo choice
    }
}

// Template: Combination Sum with duplicates in input (LC 40)
private void combSumWithDups(int[] candidates, int target, int start,
                              List<Integer> curr, List<List<Integer>> result) {
    if (target == 0) {
        result.add(new ArrayList<>(curr));
        return;
    }

    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > target) break;  // pruning (array must be sorted)
        if (i > start && candidates[i] == candidates[i - 1]) continue;  // skip duplicates

        curr.add(candidates[i]);
        combSumWithDups(candidates, target - candidates[i], i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

**Duplicate handling:** `if (i > start && candidates[i] == candidates[i-1]) continue` — this skips duplicate values AT THE SAME recursion level. At a deeper level (different `start`), duplicates are allowed. This is the most commonly asked clarification.

### Fully Solved Problem: Combination Sum II (LC 40)

**Problem:** Find all unique combinations in `candidates` that sum to `target`. Each number used at most once. Candidates may have duplicates.

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);  // MUST sort for duplicate skipping and pruning
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int remain, int start,
                       List<Integer> curr, List<List<Integer>> result) {
    if (remain == 0) {
        result.add(new ArrayList<>(curr));
        return;
    }

    for (int i = start; i < nums.length; i++) {
        if (nums[i] > remain) break;
        if (i > start && nums[i] == nums[i - 1]) continue;

        curr.add(nums[i]);
        backtrack(nums, remain - nums[i], i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not copying `curr` when adding to result | All entries in result point to the same list (which ends up empty) | `result.add(new ArrayList<>(curr))` |
| Duplicate combinations in output | `[1,2,1]` and `[1,1,2]` both appear | Sort input + skip duplicates with `i > start && nums[i] == nums[i-1]` |
| Not sorting before duplicate skip | Skip logic doesn't work on unsorted arrays | Always `Arrays.sort(candidates)` first |
| TLE on combination sum without pruning | Exploring impossible branches | `if (nums[i] > remain) break` after sorting |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "What's the time complexity?" | "O(2^n) in worst case — each element included or excluded. With pruning, much less in practice." |
| "Can each element be used unlimited times?" | "Then change `i + 1` to `i` in the recursive call. That's Combination Sum I (LC 39)." |
| "Generate Parentheses — is that combinations?" | "Yes — backtracking with constraint: at any point, open count ≥ close count, and both ≤ n." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Combinations | Medium | Start index to avoid reordering | 77 |
| 2 | Combination Sum | Medium | Unlimited use: recurse with same index | 39 |
| 3 | Combination Sum II | Medium | Solved above — sort + skip duplicates | 40 |
| 4 | Combination Sum III | Medium | Choose k from [1,9] summing to n | 216 |
| 5 | Letter Combinations of Phone Number | Medium | Map digits to letters, generate all combos | 17 |
| 6 | Generate Parentheses | Medium | Track open/close counts as constraints | 22 |

---

## 2. Permutations

### When to Use
- "All **orderings**" / "all arrangements"
- Order matters: `{1,2}` != `{2,1}`
- "**Next permutation**"

### Optimized Java Template

```java
// Template A: Permutations (no duplicates) — swap-based, no extra space
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    permuteHelper(nums, 0, result);
    return result;
}

private void permuteHelper(int[] nums, int start, List<List<Integer>> result) {
    if (start == nums.length) {
        List<Integer> perm = new ArrayList<>();
        for (int n : nums) perm.add(n);
        result.add(perm);
        return;
    }

    for (int i = start; i < nums.length; i++) {
        swap(nums, start, i);
        permuteHelper(nums, start + 1, result);
        swap(nums, start, i);  // backtrack
    }
}

// Template B: Permutations with duplicates (LC 47)
public List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> curr,
                       List<List<Integer>> result) {
    if (curr.size() == nums.length) {
        result.add(new ArrayList<>(curr));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // Skip duplicates: if same value and previous wasn't used, skip
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;

        used[i] = true;
        curr.add(nums[i]);
        backtrack(nums, used, curr, result);
        curr.remove(curr.size() - 1);
        used[i] = false;
    }
}
```

**Duplicate logic for permutations:** `!used[i-1]` ensures we only use the first occurrence of a duplicate value at each depth level. This is the opposite of combinations — here we track which specific elements are used.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Permutations | Medium | Swap-based or used[] array | 46 |
| 2 | Permutations II | Medium | Sort + skip duplicates with !used[i-1] | 47 |
| 3 | Next Permutation | Medium | Find rightmost ascending pair, swap, reverse suffix | 31 |
| 4 | Permutation Sequence | Hard | Math: factorial number system for kth permutation | 60 |

---

## 3. Subsets

### When to Use
- "**Power set**" / "all subsets"
- "Subsets with **duplicates**"
- Each element: include or exclude (binary choice)

### Optimized Java Template

```java
// Template: Subsets (include/exclude at each index)
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> curr,
                       List<List<Integer>> result) {
    result.add(new ArrayList<>(curr));  // every partial state is a valid subset

    for (int i = start; i < nums.length; i++) {
        curr.add(nums[i]);
        backtrack(nums, i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}

// Subsets II (with duplicates): sort + skip
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    backtrack2(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack2(int[] nums, int start, List<Integer> curr,
                        List<List<Integer>> result) {
    result.add(new ArrayList<>(curr));

    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i - 1]) continue;  // skip duplicates
        curr.add(nums[i]);
        backtrack2(nums, i + 1, curr, result);
        curr.remove(curr.size() - 1);
    }
}
```

**Alternative: bitmask enumeration for subsets**
```java
public List<List<Integer>> subsetsBitmask(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();
    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) subset.add(nums[i]);
        }
        result.add(subset);
    }
    return result;
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Subsets | Medium | Every node in recursion tree is a valid subset | 78 |
| 2 | Subsets II | Medium | Sort + skip duplicates at same level | 90 |

---

## 4. Grid/Board Exploration

### When to Use
- "**N-Queens**", "**Sudoku Solver**"
- "**Word Search**" on a board
- Place items on a grid with constraints

### Optimized Java Template

```java
// N-Queens with O(1) conflict detection
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    boolean[] cols = new boolean[n];
    boolean[] diag1 = new boolean[2 * n - 1];  // row + col
    boolean[] diag2 = new boolean[2 * n - 1];  // row - col + n - 1
    int[] queens = new int[n];  // queens[row] = col

    placeQueen(0, n, queens, cols, diag1, diag2, result);
    return result;
}

private void placeQueen(int row, int n, int[] queens, boolean[] cols,
                        boolean[] diag1, boolean[] diag2, List<List<String>> result) {
    if (row == n) {
        result.add(buildBoard(queens, n));
        return;
    }

    for (int col = 0; col < n; col++) {
        if (cols[col] || diag1[row + col] || diag2[row - col + n - 1]) continue;

        queens[row] = col;
        cols[col] = diag1[row + col] = diag2[row - col + n - 1] = true;

        placeQueen(row + 1, n, queens, cols, diag1, diag2, result);

        cols[col] = diag1[row + col] = diag2[row - col + n - 1] = false;
    }
}

private List<String> buildBoard(int[] queens, int n) {
    List<String> board = new ArrayList<>();
    for (int row = 0; row < n; row++) {
        char[] line = new char[n];
        Arrays.fill(line, '.');
        line[queens[row]] = 'Q';
        board.add(new String(line));
    }
    return board;
}
```

**O(1) conflict check** using boolean arrays:
- `cols[col]`: column conflict
- `diag1[row + col]`: main diagonal (top-left to bottom-right) — all cells on the same diagonal have the same `row + col`
- `diag2[row - col + n - 1]`: anti-diagonal — all cells have the same `row - col` (offset by `n-1` to avoid negative indices)

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Word Search | Medium | DFS on grid, mark visited, backtrack | 79 |
| 2 | N-Queens | Hard | O(1) conflict check with boolean arrays | 51 |
| 3 | N-Queens II | Hard | Same but just count solutions | 52 |
| 4 | Sudoku Solver | Hard | Row/col/box boolean arrays for O(1) validation | 37 |
| 5 | Word Search II | Hard | Trie + DFS (see [Tries](./12-tries.md)) | 212 |

---

## 5. String Partitioning

### When to Use
- "**Palindrome partitioning**"
- "**Restore IP addresses**"
- "**Word Break II**" — generate all valid splits
- Split a string into valid parts

### Optimized Java Template

```java
// Palindrome Partitioning with precomputed isPalin table
public List<List<String>> partition(String s) {
    int n = s.length();
    boolean[][] isPalin = new boolean[n][n];

    // Precompute palindrome table
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i; j < n; j++) {
            isPalin[i][j] = s.charAt(i) == s.charAt(j)
                && (j - i <= 2 || isPalin[i + 1][j - 1]);
        }
    }

    List<List<String>> result = new ArrayList<>();
    backtrack(s, 0, isPalin, new ArrayList<>(), result);
    return result;
}

private void backtrack(String s, int start, boolean[][] isPalin,
                       List<String> curr, List<List<String>> result) {
    if (start == s.length()) {
        result.add(new ArrayList<>(curr));
        return;
    }

    for (int end = start; end < s.length(); end++) {
        if (isPalin[start][end]) {
            curr.add(s.substring(start, end + 1));
            backtrack(s, end + 1, isPalin, curr, result);
            curr.remove(curr.size() - 1);
        }
    }
}
```

**Why precompute the palindrome table:** without it, each `isPalindrome` check is O(n), called O(2^n) times in worst case. Precomputing is O(n²) once, then O(1) per check.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Palindrome Partitioning | Medium | Precompute isPalin + backtrack | 131 |
| 2 | Restore IP Addresses | Medium | At each step, try 1-3 digits, validate 0-255 | 93 |
| 3 | Word Break II | Hard | Backtrack with memoization (or DP + reconstruction) | 140 |

---

## Backtracking vs DP Decision Framework

| Property | Backtracking | DP |
|----------|-------------|-----|
| Goal | Generate ALL solutions | Find OPTIMAL solution or COUNT |
| Overlapping subproblems | No (each branch is unique) | Yes (same subproblem solved multiple times) |
| Time complexity | Exponential (2^n, n!) | Polynomial (n², n*W, etc.) |
| When to use | "List all", "find all" | "Count all", "minimum/maximum" |

**Key insight:** if the problem asks "how many" instead of "list all", it's probably DP, not backtracking.

---

## Cross-References

- **Backtracking on grids** = DFS from [Graphs](./06-graphs.md#2-dfs-connected-components) with undo step
- **Subsets with bitmask** connects to [Bit Manipulation](./13-bit-manipulation.md#2-bit-masking)
- **Palindrome Partitioning DP** connects to [Palindrome DP](./07-dynamic-programming.md#6-palindrome-dp)
- **Word Search II** combines backtracking with [Tries](./12-tries.md)
