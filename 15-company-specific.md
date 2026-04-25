# 15 — Company-Specific Patterns

Every MAANG company has favorite problem types, constraints, and evaluation styles. This chapter maps patterns to companies, highlights company-specific gotchas, and provides the most frequently asked problems per company. Knowing the meta-game gives you a 20-30% edge over someone who "just practices LeetCode."

---

## Table of Contents
1. [Google](#1-google)
2. [Amazon](#2-amazon)
3. [Meta (Facebook)](#3-meta-facebook)
4. [Apple](#4-apple)
5. [Microsoft](#5-microsoft)
6. [Netflix](#6-netflix)
7. [Tier 1.5 Companies](#7-tier-15-companies)

---

## 1. Google

### Interview Style
- **2 coding rounds** (45 min each), 1 Googleyness/Leadership, 1 System Design (L4+)
- Interviewers expect **clean, compilable code** on a whiteboard/doc
- **Follow-up heavy** — solve the base case, then expect 2-3 extensions
- Values **mathematical thinking** and optimization — "Can you do better?" is guaranteed
- Interviewers write detailed feedback — every sentence you say matters

### Favorite Patterns
| Priority | Pattern | Why Google Loves It |
|----------|---------|-------------------|
| High | Graph algorithms (BFS, DFS, Topo Sort) | Infrastructure/distributed systems DNA |
| High | Dynamic Programming (all variants) | Tests algorithmic maturity |
| High | Binary Search on answer | Tests problem transformation skills |
| Medium | Segment Tree / BIT | Google asks harder than other companies |
| Medium | String algorithms (KMP, rolling hash) | Search engine roots |
| Medium | Greedy with proof | "Prove your solution is optimal" |
| Lower | Standard LinkedList / Stack | Rarely asked (too common) |

### Top 20 Google Problems

| # | Problem | Pattern | Key Insight | LC # |
|---|---------|---------|-------------|------|
| 1 | Median of Two Sorted Arrays | Binary Search | Partition-based, O(log min(m,n)) | 4 |
| 2 | Longest Substring Without Repeating | Sliding Window | Variable window with `int[128]` | 3 |
| 3 | Regular Expression Matching | DP | 2D DP with `.*` and `.` handling | 10 |
| 4 | Merge Intervals | Intervals | Sort by start, extend overlapping | 56 |
| 5 | Word Ladder | Graph BFS | Bidirectional BFS for optimization | 127 |
| 6 | Alien Dictionary | Topo Sort | Build graph from word order | 269 |
| 7 | Sliding Window Maximum | Mono Deque | `ArrayDeque` maintaining decreasing order | 239 |
| 8 | Course Schedule II | Topo Sort | Kahn's algorithm, detect cycles | 210 |
| 9 | Word Search II | Trie + DFS | Build Trie, single DFS pass | 212 |
| 10 | Trapping Rain Water | Two Pointer | `leftMax` and `rightMax` converging | 42 |
| 11 | Longest Increasing Subsequence | DP + Binary Search | Patience sorting O(n log n) | 300 |
| 12 | Range Sum Query - Mutable | Segment Tree / BIT | Iterative segment tree | 307 |
| 13 | Serialize and Deserialize Binary Tree | Tree + Design | Preorder with null markers | 297 |
| 14 | Split Array Largest Sum | Binary Search on Answer | Min the max subarray sum | 410 |
| 15 | Decode Ways | 1D DP | Handle '0' carefully, check 10-26 | 91 |
| 16 | Maximum Product Subarray | DP | Track both max and min (negatives flip) | 152 |
| 17 | Find Median from Data Stream | Two Heaps | Max-heap lower + min-heap upper | 295 |
| 18 | LRU Cache | Design + LinkedHashMap | `LinkedHashMap` or custom DLL + HashMap | 146 |
| 19 | Robot Room Cleaner | Backtracking + DFS | Relative direction tracking | 489 |
| 20 | Shortest Path Visiting All Nodes | BFS + Bitmask | State = (node, visited_mask) | 847 |

### Google-Specific Tips
- **Always ask clarifying questions** — Google interviewers grade this explicitly
- **State your approach BEFORE coding** — "I'll use BFS because the graph is unweighted"
- **Discuss time/space complexity proactively** — don't wait to be asked
- **Handle edge cases verbally** — "If the input is empty, I'd return..." even if you don't code it
- **Expect: "Can you optimize?"** — always have a brute force AND an optimized approach ready

---

## 2. Amazon

### Interview Style
- **Amazon focuses on Leadership Principles as much as coding**
- OA (Online Assessment) + 4-5 virtual loops (2 coding, 2 LP, 1 system design for SDE2+)
- Coding problems tend to be **LC Medium** — rarely Hard
- Values **practical problem solving** and **working code** over mathematical elegance
- **BQ (Behavioral Questions)** are CRITICAL — prepare STAR stories for all 16 LPs

### Favorite Patterns
| Priority | Pattern | Why Amazon Loves It |
|----------|---------|-------------------|
| High | BFS/DFS on grids | Warehouse/logistics optimization |
| High | Heap / Priority Queue | Scheduling, Top-K for recommendations |
| High | Two Pointers / Sliding Window | String/array processing for retail |
| High | Hash Map | Frequency counting, dedup |
| Medium | Union Find | Connected components in networks |
| Medium | Tree traversals | Organizational hierarchy problems |
| Lower | Advanced (Segment Tree, KMP) | Rarely asked |

### Top 20 Amazon Problems

| # | Problem | Pattern | Key Insight | LC # |
|---|---------|---------|-------------|------|
| 1 | Two Sum | Hash Map | One-pass with complement lookup | 1 |
| 2 | Number of Islands | BFS/DFS on Grid | Mark visited, count components | 200 |
| 3 | LRU Cache | Design | DLL + HashMap, O(1) operations | 146 |
| 4 | Merge Intervals | Intervals | Sort by start, merge overlapping | 56 |
| 5 | K Closest Points to Origin | Heap / Quickselect | Max-heap of size K | 973 |
| 6 | Reorder Data in Log Files | Custom Sort | Letters before digits, stable sort | 937 |
| 7 | Word Ladder | BFS | Level-order BFS for shortest path | 127 |
| 8 | Rotting Oranges | Multi-source BFS | All rotten oranges in queue at once | 994 |
| 9 | Meeting Rooms II | Intervals / Heap | Sort starts/ends or min-heap | 253 |
| 10 | Top K Frequent Words | Heap | Min-heap of size K with custom comparator | 692 |
| 11 | Min Cost to Connect Sticks | Heap | Always merge two smallest (Huffman) | 1167 |
| 12 | Subtree of Another Tree | Tree | Compare subtrees recursively or serialize | 572 |
| 13 | Copy List with Random Pointer | Linked List | Interweave or HashMap clone | 138 |
| 14 | Search in Rotated Sorted Array | Binary Search | Determine sorted half, narrow search | 33 |
| 15 | Group Anagrams | Hash Map | Sort chars as key, or frequency key | 49 |
| 16 | Accounts Merge | Union Find | Email → account ID mapping | 721 |
| 17 | Maximum Subarray | Kadane's | Track running sum, reset when negative | 53 |
| 18 | Min Stack | Stack Design | Pair: (value, current_min) | 155 |
| 19 | Partition Labels | Greedy | Last occurrence map, expand window | 763 |
| 20 | Word Break | DP | `dp[i]` = can s[0..i) be segmented | 139 |

### Amazon-Specific Tips
- **Tie every answer to an LP**: "Customer Obsession" → handle edge cases, "Bias for Action" → start with brute force, optimize
- **OA matters** — poor OA can disqualify you even before interviews
- **Code must RUN** — Amazon interviewers often paste your code and test it
- **Explain trade-offs** — "I chose HashMap for O(1) lookup at the cost of O(n) space"
- **Think about scale** — "This would work for 10M items because..."

---

## 3. Meta (Facebook)

### Interview Style
- **2 coding rounds** (40 min each, one problem per round — must finish fast)
- 1 System Design (E4+), 1 Behavioral
- **Speed is paramount** — you have ~35 min to code, test, and discuss
- Problems are **LC Medium** (occasionally Hard for E5+)
- Interviewers follow a rubric with specific checkpoints

### Favorite Patterns
| Priority | Pattern | Why Meta Loves It |
|----------|---------|------------------|
| High | Arrays and Strings | Quick to assess, many variants |
| High | Trees and Graphs | Social graph, organizational structure |
| High | Binary Search | Clean algorithmic thinking |
| High | Dynamic Programming (1D/2D) | Scalable problem solving |
| Medium | Sliding Window | News feed, content window |
| Medium | Intervals | Event scheduling |
| Lower | Advanced data structures | Rarely asked (time too short) |

### Top 20 Meta Problems

| # | Problem | Pattern | Key Insight | LC # |
|---|---------|---------|-------------|------|
| 1 | Valid Palindrome II | Two Pointers | Delete at most one char, check remaining | 680 |
| 2 | Minimum Remove to Make Valid Parentheses | Stack | Track unmatched indices | 1249 |
| 3 | Binary Tree Vertical Order Traversal | BFS + Map | Column index → BFS for top-to-bottom order | 314 |
| 4 | Random Pick with Weight | Prefix Sum + Binary Search | Weighted random = binary search on cumulative | 528 |
| 5 | Subarray Sum Equals K | Prefix Sum + Map | Count prefix sums that differ by K | 560 |
| 6 | Lowest Common Ancestor of BT | Tree DFS | Post-order: if both sides found, current is LCA | 236 |
| 7 | Add Strings | Math | Digit-by-digit addition, carry | 415 |
| 8 | Merge Intervals | Intervals | Sort + merge overlapping | 56 |
| 9 | Dot Product of Two Sparse Vectors | Design | Store non-zero indices, two-pointer multiply | 1570 |
| 10 | Buildings With an Ocean View | Stack / Reverse Sweep | Monotonic stack or sweep right-to-left | 1762 |
| 11 | Nested List Weight Sum | DFS | Multiply by depth level | 339 |
| 12 | Clone Graph | BFS/DFS + Map | Visited map: original → clone | 133 |
| 13 | Diameter of Binary Tree | Tree Bottom-Up | Max of (leftHeight + rightHeight) across all nodes | 543 |
| 14 | Range Sum of BST | BST Traversal | Prune branches outside [low, high] | 938 |
| 15 | Move Zeroes | Two Pointers | Swap non-zeros forward, fill zeros | 283 |
| 16 | Product of Array Except Self | Prefix/Suffix | Left pass * right pass | 238 |
| 17 | 3Sum | Two Pointers | Sort + fix one, two-pointer for rest | 15 |
| 18 | Word Break | DP | `dp[i]` with dictionary lookup | 139 |
| 19 | Accounts Merge | Union Find | Group by email, map email → name | 721 |
| 20 | Basic Calculator II | Stack | Handle precedence with immediate compute | 227 |

### Meta-Specific Tips
- **Speed > perfection** — a working solution in 25 min beats an optimal one in 40
- **Practice 35-minute drills** — simulate real time pressure
- **Test your code out loud** — walk through an example before running
- **Meta loves practical problems** — "design a function that..." not "find the optimal..."
- **Clarify constraints early** — input size, character set, edge cases

---

## 4. Apple

### Interview Style
- Emphasis on **system design** and **domain knowledge** (UIKit/SwiftUI for iOS roles)
- Coding rounds use **Xcode or shared editor** — expect to run your code
- Apple values **clean, readable code** and **attention to edge cases**
- Questions tend to be **classic patterns** with real-world framing

### Favorite Patterns & Top Problems

| # | Problem | Pattern | LC # |
|---|---------|---------|------|
| 1 | Two Sum | Hash Map | 1 |
| 2 | Merge Two Sorted Lists | Linked List | 21 |
| 3 | Valid Parentheses | Stack | 20 |
| 4 | Maximum Subarray | Kadane's | 53 |
| 5 | Binary Tree Level Order | BFS | 102 |
| 6 | LRU Cache | Design | 146 |
| 7 | Word Search | Backtracking | 79 |
| 8 | Merge Intervals | Intervals | 56 |
| 9 | Valid Palindrome | Two Pointers | 125 |
| 10 | Spiral Matrix | Simulation | 54 |

### Apple-Specific Tips
- **Code quality matters** — variable names, structure, no hacks
- **Discuss testing** — "I'd write unit tests for: empty input, single element, large input"
- **Expect domain questions** — concurrency, memory management, OS fundamentals
- **Collaborative style** — interviewers work WITH you, not against you

---

## 5. Microsoft

### Interview Style
- **3-4 coding rounds** + 1 "As Appropriate" (hiring manager)
- Problems range from **Easy to Hard** within the same loop
- Strong focus on **data structures** and **clean OOP**
- Microsoft interviewers often ask **system design elements** within coding rounds

### Favorite Patterns & Top Problems

| # | Problem | Pattern | LC # |
|---|---------|---------|------|
| 1 | Two Sum | Hash Map | 1 |
| 2 | Add Two Numbers | Linked List | 2 |
| 3 | Longest Substring Without Repeating | Sliding Window | 3 |
| 4 | Reverse Linked List | Linked List | 206 |
| 5 | Number of Islands | BFS/DFS | 200 |
| 6 | Validate BST | Tree | 98 |
| 7 | LRU Cache | Design | 146 |
| 8 | Serialize and Deserialize BT | Tree | 297 |
| 9 | Word Ladder | BFS | 127 |
| 10 | Coin Change | DP | 322 |

### Microsoft-Specific Tips
- **OOP matters** — clean class design, proper encapsulation
- **Ask about constraints** — Microsoft problems sometimes have unusual constraints
- **Edge cases are explicitly tested** — null inputs, empty collections, Integer.MAX_VALUE
- **Communication style** — structured, clear, like a design review

---

## 6. Netflix

### Interview Style
- **Senior-heavy hiring** — mostly L5+ equivalent
- Deep focus on **system design** (streaming, caching, recommendations)
- Coding is **practical** — "implement this component" more than "solve this puzzle"
- Values **culture fit** extremely highly

### Favorite Patterns & Top Problems

| # | Problem | Pattern | LC # |
|---|---------|---------|------|
| 1 | LRU Cache | Design | 146 |
| 2 | Top K Frequent Elements | Heap | 347 |
| 3 | Merge K Sorted Lists | Heap | 23 |
| 4 | Graph traversals | BFS/DFS | various |
| 5 | Rate limiter design | Sliding Window | custom |

### Netflix-Specific Tips
- **System design is 50%+ of the evaluation** for senior roles
- **Demonstrate depth** — Netflix wants experts, not generalists
- **"Freedom and Responsibility"** — show independent thinking
- **Know streaming concepts** — CDN, buffering, adaptive bitrate

---

## 7. Tier 1.5 Companies

Companies like **Uber, Airbnb, Stripe, LinkedIn, Twitter/X, Snap, Pinterest** — slightly different focus areas.

### Uber
- Heavy on **graphs** (routing, matching) and **geospatial** (quad trees)
- "Design a ride matching system" is a classic
- Key problems: Meeting Rooms, Course Schedule, Cheapest Flights Within K Stops

### Airbnb
- Custom problems: **calendar booking**, **pagination**, **search ranking**
- Collaborative coding — almost pair programming
- Key problems: My Calendar, Meeting Rooms II, Word Search

### Stripe
- **Practical coding** — "parse this log", "implement this API"
- Focus on correctness, error handling, and testing
- Less algorithmic puzzles, more engineering problems

### LinkedIn
- **Graph problems** (social network, connections)
- Nested List Iterator, All O'one Data Structure
- Key problems: Nested List Weight Sum, Max Stack, Shortest Word Distance

### Twitter/X
- **Real-time processing**, streaming algorithms
- Key problems: Design Twitter, Merge K Sorted Lists, Top K Frequent Words

---

## Universal Preparation Strategy

### By Time Available

| Weeks | Focus |
|-------|-------|
| 1-2 | Top 50 from Blind 75 — covers all basic patterns |
| 3-4 | Company-specific top 20 from above lists |
| 5-8 | Full Blind 75 + Neetcode 150 |
| 8-16 | LC premium company-tagged problems + contests |

### By Difficulty Level

| Your Level | Target |
|-----------|--------|
| Beginner | Easy + straightforward Mediums (Two Sum, Valid Parentheses, Merge Intervals) |
| Intermediate | All Mediums fluently, start touching Hards |
| Advanced | Hard problems in 30-40 min, multiple approaches for each |
| Expert | Solve Hards in 25 min, discuss trade-offs at system level |

### Pattern Priority (Universal)

| Rank | Pattern | Frequency |
|------|---------|-----------|
| 1 | Arrays + Two Pointers | ~25% of all problems |
| 2 | Trees + Graphs | ~20% |
| 3 | Dynamic Programming | ~15% |
| 4 | Binary Search | ~10% |
| 5 | Sliding Window + Hash Map | ~10% |
| 6 | Stack + Queue | ~8% |
| 7 | Heap | ~5% |
| 8 | Intervals + Greedy | ~4% |
| 9 | Backtracking | ~3% |

---

## Cross-References

Every problem in this chapter maps to a pattern chapter:
- Arrays/Hash Map → [01](./01-arrays-and-hashing.md)
- Linked Lists → [02](./02-linked-lists.md)
- Stacks/Queues → [03](./03-stacks-and-queues.md)
- Binary Search → [04](./04-binary-search.md)
- Trees → [05](./05-trees.md)
- Graphs → [06](./06-graphs.md)
- DP → [07](./07-dynamic-programming.md)
- Backtracking → [08](./08-backtracking.md)
- Heaps → [09](./09-heaps.md)
- Intervals → [10](./10-intervals.md)
- Greedy → [11](./11-greedy.md)
- Tries → [12](./12-tries.md)
- Bit Manipulation → [13](./13-bit-manipulation.md)
- Advanced → [14](./14-advanced-patterns.md)
