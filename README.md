# DSA Mastery: Zero to MAANG | Java | Staff-Engineer Quality

A comprehensive guide to mastering Data Structures and Algorithms for MAANG interviews. Every template is optimized, every pattern includes traps and interviewer counter-questions, and every solved problem walks through the thinking process — not just the code.

## Who This Is For

Anyone preparing for software engineering interviews at Google, Meta, Amazon, Apple, Microsoft, Netflix, or similar companies. Takes you from fundamentals through advanced patterns that separate senior from staff-level candidates.

## How This Guide Is Different

| Generic DSA Guide | This Repository |
|---|---|
| `Stack<Integer>` for stack problems | `ArrayDeque<Integer>` (2-3x faster, not synchronized) |
| `HashMap<Character, Integer>` for sliding window | `int[128]` array (zero autoboxing, zero GC pressure) |
| Recursive segment tree with `4*n` array | Iterative segment tree with `2*n` array (half the code, faster) |
| `(a, b) -> b - a` comparator | `Integer.compare(b, a)` (no integer overflow) |
| 2D `dp[n][W]` knapsack | 1D `dp[W]` with reverse iteration (O(W) space) |
| "Here's the solution" | "Here's how you'd ARRIVE at this solution in 5 minutes under pressure" |
| No traps or follow-ups | Every pattern has interviewer traps and counter-questions |

## Table of Contents

| # | File | Patterns | Difficulty |
|---|------|----------|-----------|
| 1 | [Arrays & Hashing](./01-arrays-and-hashing.md) | Two Pointers, Sliding Window (Fixed/Variable), Prefix Sum, Kadane's, Hash Map, Dutch National Flag, Cyclic Sort | Easy - Hard |
| 2 | [Linked Lists](./02-linked-lists.md) | Fast/Slow (Floyd's), Reversal, Merge, In-place Manipulation | Easy - Hard |
| 3 | [Stacks & Queues](./03-stacks-and-queues.md) | Monotonic Stack, Expression Parsing, Special Stacks, Monotonic Queue (Deque) | Easy - Hard |
| 4 | [Binary Search](./04-binary-search.md) | Standard, Search on Answer, Rotated Arrays, 2D Search, Median of Two Arrays | Easy - Hard |
| 5 | [Trees](./05-trees.md) | DFS Top-Down, DFS Bottom-Up, BFS Level Order, BST, Construction, Morris Traversal | Easy - Hard |
| 6 | [Graphs](./06-graphs.md) | BFS, DFS, Topo Sort, Union Find, Dijkstra, Bellman-Ford, Floyd-Warshall, Tarjan's, Bipartite, Multi-source BFS | Easy - Hard |
| 7 | [Dynamic Programming](./07-dynamic-programming.md) | 1D Linear, 2D Grid, Two Sequences, 0/1 Knapsack, Unbounded Knapsack, Palindrome, State Machine, Interval, Bitmask, Digit DP | Easy - Hard |
| 8 | [Backtracking](./08-backtracking.md) | Combinations, Permutations, Subsets, Grid/Board, String Partitioning | Medium - Hard |
| 9 | [Heaps](./09-heaps.md) | Top K, Two Heaps, Merge K Sorted, Lazy Deletion | Easy - Hard |
| 10 | [Intervals](./10-intervals.md) | Merge Intervals, Scheduling, Line Sweep | Easy - Hard |
| 11 | [Greedy](./11-greedy.md) | Activity Selection, Sorting-based, Exchange Argument | Easy - Hard |
| 12 | [Tries](./12-tries.md) | Prefix Trie, XOR Trie | Easy - Hard |
| 13 | [Bit Manipulation](./13-bit-manipulation.md) | XOR Properties, Bit Masking | Easy - Hard |
| 14 | [Advanced Patterns](./14-advanced-patterns.md) | Segment Tree, BIT/Fenwick, KMP, Rabin-Karp | Medium - Hard |
| 15 | [Company-Specific](./15-company-specific.md) | Google, Amazon, Meta, Apple, Netflix, Microsoft | Mixed |
| 16 | [Interview Playbook](./16-interview-playbook.md) | 5-min framework, traps, communication, counter-questions, mock rubric | - |

## Pattern Recognition Quick-Reference

When you see these keywords, reach for these patterns:

| Keyword / Signal | Pattern | File |
|---|---|---|
| "Sorted array", "pair with sum" | Two Pointers | [01](./01-arrays-and-hashing.md) |
| "Subarray of size K" | Fixed Sliding Window | [01](./01-arrays-and-hashing.md) |
| "Longest/shortest subarray with condition" | Variable Sliding Window | [01](./01-arrays-and-hashing.md) |
| "Subarray sum equals K", "range sum" | Prefix Sum | [01](./01-arrays-and-hashing.md) |
| "Maximum subarray sum" | Kadane's Algorithm | [01](./01-arrays-and-hashing.md) |
| "Frequency", "duplicates", "anagrams" | Hash Map | [01](./01-arrays-and-hashing.md) |
| "Sort 0s/1s/2s", "partition around pivot" | Dutch National Flag | [01](./01-arrays-and-hashing.md) |
| "Missing number in [1,n]" | Cyclic Sort | [01](./01-arrays-and-hashing.md) |
| "Cycle detection", "find middle" | Fast/Slow Pointers | [02](./02-linked-lists.md) |
| "Reverse linked list" | Reversal Pattern | [02](./02-linked-lists.md) |
| "Next greater/smaller element" | Monotonic Stack | [03](./03-stacks-and-queues.md) |
| "Parentheses", "calculator" | Expression Stack | [03](./03-stacks-and-queues.md) |
| "Maximum in sliding window" | Monotonic Deque | [03](./03-stacks-and-queues.md) |
| "Sorted array, find element" | Binary Search | [04](./04-binary-search.md) |
| "Minimize the maximum", "maximize the minimum" | Binary Search on Answer | [04](./04-binary-search.md) |
| "Rotated sorted array" | Modified Binary Search | [04](./04-binary-search.md) |
| "Path sum", "tree traversal" | DFS (Top-Down/Bottom-Up) | [05](./05-trees.md) |
| "Level order", "right side view" | BFS | [05](./05-trees.md) |
| "Validate BST", "kth smallest in BST" | BST Property | [05](./05-trees.md) |
| "Shortest path (unweighted)" | Graph BFS | [06](./06-graphs.md) |
| "Number of islands", "connected components" | Graph DFS | [06](./06-graphs.md) |
| "Prerequisites", "task ordering" | Topological Sort | [06](./06-graphs.md) |
| "Dynamic connectivity", "redundant edge" | Union Find | [06](./06-graphs.md) |
| "Shortest path (weighted, non-negative)" | Dijkstra | [06](./06-graphs.md) |
| "Shortest path (negative weights)" | Bellman-Ford | [06](./06-graphs.md) |
| "Critical connections", "bridges" | Tarjan's Algorithm | [06](./06-graphs.md) |
| "Climbing stairs", "house robber" | 1D DP | [07](./07-dynamic-programming.md) |
| "Grid paths", "minimum path sum" | 2D Grid DP | [07](./07-dynamic-programming.md) |
| "LCS", "edit distance" | Two Sequence DP | [07](./07-dynamic-programming.md) |
| "Subset sum", "partition equal" | 0/1 Knapsack | [07](./07-dynamic-programming.md) |
| "Coin change", "unlimited items" | Unbounded Knapsack | [07](./07-dynamic-programming.md) |
| "Buy/sell stock" (all variants) | State Machine DP | [07](./07-dynamic-programming.md) |
| "Burst balloons", "matrix chain" | Interval DP | [07](./07-dynamic-programming.md) |
| "TSP", "small n with subsets" | Bitmask DP | [07](./07-dynamic-programming.md) |
| "All combinations", "choose k from n" | Backtracking (Combinations) | [08](./08-backtracking.md) |
| "All permutations" | Backtracking (Permutations) | [08](./08-backtracking.md) |
| "Power set", "all subsets" | Backtracking (Subsets) | [08](./08-backtracking.md) |
| "N-Queens", "Sudoku" | Board Backtracking | [08](./08-backtracking.md) |
| "Top K", "Kth largest" | Heap | [09](./09-heaps.md) |
| "Find median in stream" | Two Heaps | [09](./09-heaps.md) |
| "Merge K sorted" | Heap Merge | [09](./09-heaps.md) |
| "Merge overlapping intervals" | Interval Merge | [10](./10-intervals.md) |
| "Meeting rooms" | Interval Scheduling | [10](./10-intervals.md) |
| "Skyline problem" | Line Sweep | [10](./10-intervals.md) |
| "Jump game", "gas station" | Greedy | [11](./11-greedy.md) |
| "Prefix matching", "autocomplete" | Trie | [12](./12-tries.md) |
| "Maximum XOR" | XOR Trie | [12](./12-tries.md) |
| "Single number", "missing number" | XOR / Bit Manipulation | [13](./13-bit-manipulation.md) |
| "Range query with updates" | Segment Tree / BIT | [14](./14-advanced-patterns.md) |
| "Pattern matching in string" | KMP / Rabin-Karp | [14](./14-advanced-patterns.md) |

## Complexity Quick-Reference

| Pattern | Time | Space | Notes |
|---------|------|-------|-------|
| Two Pointers | O(n) | O(1) | Requires sorted or specific structure |
| Sliding Window | O(n) | O(k) | k = window size or unique elements |
| Binary Search | O(log n) | O(1) | Requires monotonic property |
| Binary Search on Answer | O(n log M) | O(1) | M = answer range |
| DFS/BFS on Tree | O(n) | O(h) / O(w) | h = height, w = max width |
| DFS/BFS on Graph | O(V + E) | O(V) | Adjacency list representation |
| Topological Sort | O(V + E) | O(V) | DAG only |
| Dijkstra | O((V+E) log V) | O(V) | Non-negative weights only |
| Bellman-Ford | O(V * E) | O(V) | Handles negative weights |
| Floyd-Warshall | O(V^3) | O(V^2) | All-pairs shortest path |
| Union Find | O(alpha(n)) ~ O(1) | O(n) | With path compression + union by rank |
| DP (1D) | O(n) | O(1) optimized | Rolling variables when possible |
| DP (2D) | O(n*m) | O(min(n,m)) optimized | Rolling array |
| Knapsack | O(n*W) | O(W) optimized | W = capacity |
| Backtracking | O(2^n) or O(n!) | O(n) | Exponential by nature |
| Heap (Top K) | O(n log k) | O(k) | Min-heap of size k |
| Trie | O(m) per operation | O(SIGMA * m * n) | m = word length, SIGMA = alphabet |
| Segment Tree | O(log n) per op | O(n) | Iterative = 2n array |
| BIT / Fenwick | O(log n) per op | O(n) | Simpler than segment tree for prefix ops |

## Java-Specific Gotchas for DSA

These will bite you in interviews if you don't know them:

| Gotcha | What Happens | Fix |
|--------|-------------|-----|
| `Integer` comparison with `==` | Works for -128 to 127 (Integer cache), fails for 128+ | Always use `.equals()` or `.intValue()` |
| `(a + b) / 2` for mid in binary search | Integer overflow when a + b > 2^31 | Use `a + (b - a) / 2` |
| `Stack` class | Synchronized, extends `Vector`, legacy, slow | Use `ArrayDeque` as stack |
| `(a, b) -> b - a` in PQ comparator | Integer overflow when b - a exceeds int range | Use `Integer.compare(b, a)` |
| `Arrays.sort(int[])` | Dual-pivot quicksort, O(n^2) worst case | For objects, `Arrays.sort(Integer[])` uses TimSort O(n log n) guaranteed |
| String concatenation in loops | Creates new String object each time, O(n^2) total | Use `StringBuilder` |
| `HashMap` initial capacity | Rehashes at 75% load factor, triggers GC | Pre-size: `new HashMap<>(n * 4 / 3 + 1)` |
| `PriorityQueue` is min-heap | `poll()` returns smallest, not largest | For max-heap: `new PriorityQueue<>(Comparator.reverseOrder())` |
| `int` overflow in sum/product | Sum of 10^5 elements each up to 10^9 overflows int | Use `long` for accumulators |
| `char` arithmetic | `'a' - 'a'` = 0, but `'a' + 1` = 98 (int, not char) | Cast explicitly: `(char)('a' + 1)` |

## 16-Week Study Schedule

### Phase 1: Foundations (Weeks 1-4)

**Week 1: Arrays & Hashing Basics**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Two Pointers | 5 easy, 2 medium |
| 3-4 | Sliding Window (Fixed) | 3 easy, 2 medium |
| 5-6 | Sliding Window (Variable) | 1 easy, 4 medium |
| 7 | Review + mixed problems | Revisit any unsolved |

**Week 2: Arrays & Hashing Advanced**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Prefix Sum | 3 easy, 3 medium |
| 3-4 | Kadane's Algorithm | 2 easy, 3 medium |
| 5-6 | Hash Map + DNF + Cyclic Sort | 4 easy, 4 medium |
| 7 | Timed practice: 3 random array problems in 60 min | |

**Week 3: Linked Lists**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Fast/Slow Pointers | 4 easy, 3 medium |
| 3-4 | Reversal Patterns | 1 easy, 4 medium, 1 hard |
| 5-6 | Merge + In-place | 2 easy, 4 medium, 1 hard |
| 7 | Review all linked list patterns | |

**Week 4: Stacks & Queues**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-3 | Monotonic Stack | 2 easy, 4 medium, 2 hard |
| 4-5 | Expression Parsing | 1 easy, 3 medium, 1 hard |
| 6 | Special Stacks + Monotonic Queue | 3 easy, 1 medium, 1 hard |
| 7 | Timed practice: 2 medium + 1 hard in 75 min | |

### Phase 2: Core Algorithms (Weeks 5-9)

**Week 5: Binary Search**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Standard + Bisect Templates | 5 easy, 3 medium |
| 3-4 | Binary Search on Answer | 4 medium, 2 hard |
| 5-6 | Rotated + 2D + Median of Two Arrays | 4 medium, 2 hard |
| 7 | Review — focus on choosing the right template | |

**Week 6: Trees Part 1**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | DFS Top-Down (Preorder) | 4 easy, 3 medium |
| 3-4 | DFS Bottom-Up (Postorder) | 3 easy, 3 medium, 1 hard |
| 5-6 | BFS Level Order | 3 easy, 4 medium |
| 7 | Mixed tree problems — identify DFS vs BFS | |

**Week 7: Trees Part 2 + Graphs Intro**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | BST Patterns | 4 easy, 5 medium, 1 hard |
| 3-4 | Tree Construction + Morris Traversal | 3 medium, 2 hard |
| 5-6 | Graph BFS + Multi-source BFS | 1 easy, 4 medium |
| 7 | Review trees + intro graphs | |

**Week 8: Graphs — Traversals**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-3 | DFS Connected Components | 1 easy, 6 medium, 1 hard |
| 4-5 | Topological Sort | 4 medium, 2 hard |
| 6-7 | Bipartite + Review | 2 medium + mixed graph problems |

**Week 9: Graphs — Advanced**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Union Find | 1 easy, 5 medium, 2 hard |
| 3-4 | Dijkstra + Bellman-Ford | 4 medium, 2 hard |
| 5-6 | Floyd-Warshall + Tarjan's | 2 medium, 2 hard |
| 7 | Timed: 2 medium graph problems in 45 min | |

### Phase 3: Dynamic Programming (Weeks 10-13)

**Week 10: DP Basics**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | 1D Linear DP | 3 easy, 5 medium |
| 3-4 | 2D Grid DP | 5 medium, 2 hard |
| 5-6 | Practice space optimization on solved problems | |
| 7 | Review — draw state transition diagrams | |

**Week 11: DP Sequences**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-3 | Two Sequences DP (LCS, Edit Distance) | 4 medium, 3 hard |
| 4-5 | State Machine DP (Stock problems) | 1 easy, 3 medium, 2 hard |
| 6-7 | Review all DP patterns so far | |

**Week 12: DP Optimization**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | 0/1 Knapsack | 4 medium |
| 3-4 | Unbounded Knapsack | 4 medium |
| 5-6 | Palindrome DP | 4 medium, 1 hard |
| 7 | Mixed DP — identify which knapsack variant | |

**Week 13: DP Hard**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Interval DP | 4 hard |
| 3-4 | Bitmask DP | 3 hard |
| 5-6 | Digit DP | 2 hard |
| 7 | Timed: 1 medium + 1 hard DP in 60 min | |

### Phase 4: Advanced Patterns (Weeks 14-15)

**Week 14: Backtracking & Heaps**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Combinations + Subsets | 6 medium |
| 3 | Permutations | 3 medium, 1 hard |
| 4-5 | Board Backtracking (N-Queens, Sudoku) | 1 medium, 3 hard |
| 6-7 | All Heap Patterns | 2 easy, 5 medium, 3 hard |

**Week 15: Remaining Patterns**
| Day | Focus | Problems |
|-----|-------|----------|
| 1-2 | Intervals + Line Sweep | 1 easy, 5 medium, 2 hard |
| 3 | Greedy (all 3 sub-patterns) | 2 easy, 4 medium, 1 hard |
| 4 | Tries | 1 easy, 4 medium, 1 hard |
| 5 | Bit Manipulation | 5 easy, 3 medium, 1 hard |
| 6-7 | Advanced: Segment Tree, BIT, KMP | 5 hard |

### Phase 5: Interview Prep (Week 16)

| Day | Focus |
|-----|-------|
| 1 | Google-style: 5 problems with follow-ups |
| 2 | Meta-style: 5 problems (speed focus) |
| 3 | Amazon-style: 5 problems |
| 4 | Mock: 2 medium + 1 hard in 90 min |
| 5 | Mock: 2 medium + 1 hard in 90 min |
| 6 | Review weakest patterns |
| 7 | Final revision + review Interview Playbook |

## How to Use This Repository

1. **Read the pattern** — understand WHEN to use it, not just HOW
2. **Study the template** — understand WHY each line exists
3. **Walk through the solved problem** — trace through with pen and paper
4. **Attempt practice problems** — start with easy, work up to hard
5. **Review traps** — these are what separate "correct" from "hired"
6. **Time yourself** — after learning a pattern, solve new problems in 25 min
7. **Do mock interviews** — explain your thinking out loud

## Input Size to Complexity Mapping

Use this to quickly determine what algorithm class is expected:

| Input Size (n) | Expected Complexity | Patterns That Fit |
|---|---|---|
| n <= 10 | O(n!), O(2^n) | Backtracking, Brute Force |
| n <= 20 | O(2^n), O(n * 2^n) | Bitmask DP, Backtracking |
| n <= 100 | O(n^3) | Floyd-Warshall, Interval DP |
| n <= 1,000 | O(n^2) | DP (2D), Brute Force TLE boundary |
| n <= 10^5 | O(n log n) | Sorting, Binary Search, Heap |
| n <= 10^6 | O(n) | Two Pointers, Sliding Window, Hash Map |
| n <= 10^9 | O(log n), O(sqrt(n)) | Binary Search, Math |
| n <= 10^18 | O(log n) | Binary Exponentiation, Math |
