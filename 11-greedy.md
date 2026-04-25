# 11 — Greedy Algorithms

Greedy works when making the locally optimal choice at each step leads to the globally optimal solution. The challenge is recognizing WHEN greedy works (and proving it). Three patterns cover most interview greedy problems: activity/scheduling selection, sorting-based greedy, and the exchange argument.

---

## Table of Contents
1. [Activity Selection / Scheduling](#1-activity-selection--scheduling)
2. [Sorting-Based Greedy](#2-sorting-based-greedy)
3. [Exchange Argument](#3-exchange-argument)

---

## When Greedy Works (and When It Doesn't)

**Greedy works when:**
- The problem has **optimal substructure** (optimal solution contains optimal sub-solutions)
- It has the **greedy choice property** (locally optimal choice is globally optimal)
- Each choice is irrevocable and doesn't need revision

**Greedy fails when:**
- Choices interact — picking one item affects future options in non-obvious ways (→ DP)
- The problem needs to consider all possibilities (→ Backtracking)
- You can construct a counterexample where the greedy misses the optimal

**Quick test:** Can you construct a small input where the greedy gives the wrong answer? If yes, it's not greedy.

---

## 1. Activity Selection / Scheduling

### When to Use
- "**Maximum number** of non-overlapping activities"
- "**Minimum platforms** / rooms"
- Interval-based optimization (already covered in [Intervals](./10-intervals.md#2-scheduling-greedy-by-end-time))

This pattern is fully covered in the Intervals chapter. Here we focus on the non-interval scheduling greedy problems.

### Optimized Java Template

```java
// Task Scheduler: arrange tasks with cooldown period
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char c : tasks) freq[c - 'A']++;

    int maxFreq = 0, maxCount = 0;
    for (int f : freq) {
        if (f > maxFreq) {
            maxFreq = f;
            maxCount = 1;
        } else if (f == maxFreq) {
            maxCount++;
        }
    }

    // (maxFreq - 1) groups of (n + 1) slots, plus maxCount for the last group
    int minLength = (maxFreq - 1) * (n + 1) + maxCount;
    return Math.max(minLength, tasks.length);  // can't be less than total tasks
}
```

**Intuition:** The most frequent task creates "frames" of size `n+1`. Other tasks fill the idle slots. If there are more tasks than idle slots, no idle time is needed.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Task Scheduler | Medium | Most frequent task determines frame size | 621 |
| 2 | Reorganize String | Medium | Max-heap by frequency, alternate placement | 767 |
| 3 | Non-overlapping Intervals | Medium | Sort by end, greedy keep (see [Intervals](./10-intervals.md)) | 435 |

---

## 2. Sorting-Based Greedy

### When to Use
- Sort by some criterion, then make greedy choices
- "**Assign cookies**", "**jump game**", "**gas station**"
- "**Minimum cost** to connect sticks" (always merge smallest)
- Problems where processing items in sorted order reveals the optimal strategy

### When NOT to Use
- Sorting doesn't capture the right ordering (e.g., 0/1 knapsack needs DP)
- Items have interdependencies that sorting can't resolve

### Optimized Java Template

```java
// Jump Game: can you reach the last index?
public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false;  // stranded
        maxReach = Math.max(maxReach, i + nums[i]);
        if (maxReach >= nums.length - 1) return true;
    }
    return true;
}

// Jump Game II: minimum jumps to reach end
public int jump(int[] nums) {
    int jumps = 0, currEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {
        farthest = Math.max(farthest, i + nums[i]);
        if (i == currEnd) {  // must jump now
            jumps++;
            currEnd = farthest;
            if (currEnd >= nums.length - 1) break;
        }
    }
    return jumps;
}

// Gas Station: find starting point for circular tour
public int canCompleteCircuit(int[] gas, int[] cost) {
    int totalSurplus = 0, currSurplus = 0, start = 0;
    for (int i = 0; i < gas.length; i++) {
        totalSurplus += gas[i] - cost[i];
        currSurplus += gas[i] - cost[i];
        if (currSurplus < 0) {
            start = i + 1;   // can't start from anything before i+1
            currSurplus = 0;
        }
    }
    return totalSurplus >= 0 ? start : -1;
}

// Assign Cookies: maximize satisfied children
public int findContentChildren(int[] children, int[] cookies) {
    Arrays.sort(children);
    Arrays.sort(cookies);
    int child = 0;
    for (int cookie = 0; cookie < cookies.length && child < children.length; cookie++) {
        if (cookies[cookie] >= children[child]) {
            child++;  // this child is satisfied
        }
    }
    return child;
}
```

### Fully Solved Problem: Jump Game II (LC 45)

**Problem:** Minimum number of jumps to reach the last index. Guaranteed reachable.

**Thinking process:**
1. At each position, we can jump up to `nums[i]` steps
2. BFS-like thinking: from current "level" (jump), what's the farthest we can reach?
3. When we've exhausted positions in the current jump range, we MUST jump
4. The farthest reachable position becomes the end of the next jump range

```java
public int jump(int[] nums) {
    int jumps = 0, currEnd = 0, farthest = 0;

    for (int i = 0; i < nums.length - 1; i++) {
        farthest = Math.max(farthest, i + nums[i]);

        if (i == currEnd) {
            jumps++;
            currEnd = farthest;
        }
    }

    return jumps;
}
```

**Trace with `[2, 3, 1, 1, 4]`:**
- i=0: farthest = max(0, 0+2) = 2. i==currEnd(0) → jump! jumps=1, currEnd=2
- i=1: farthest = max(2, 1+3) = 4. i!=currEnd
- i=2: farthest = max(4, 2+1) = 4. i==currEnd(2) → jump! jumps=2, currEnd=4
- currEnd(4) >= 4 → done. Answer: 2

**Complexity:** O(n) time, O(1) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Jump Game: iterating to `nums.length` instead of `nums.length-1` | Unnecessary check at the goal | Loop to `n-2` (we stop BEFORE the last index) |
| Gas Station: trying all starting points | O(n²) brute force, TLE | Single pass: reset on negative, check total at end |
| Not realizing Jump Game I is simpler than II | Overcomplicating with BFS/DP | Jump Game I: just track maxReach, return `maxReach >= n-1` |
| Greedy on unsorted when order matters | Wrong pairings | Sort both arrays first (cookies problem) |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Prove Jump Game II greedy is optimal" | "It's equivalent to BFS on an implicit graph. BFS always finds shortest path in unweighted graphs. Each 'level' is one jump." |
| "What if Gas Station has multiple valid starts?" | "The problem guarantees at most one valid start. If `totalSurplus >= 0`, there's exactly one." |
| "Jump Game with costs per jump?" | "Then it's Dijkstra or DP, not greedy. Greedy only works for min-hops." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Jump Game | Medium | Track maxReach, check if stranded | 55 |
| 2 | Jump Game II | Medium | BFS-like level expansion, O(n) | 45 |
| 3 | Gas Station | Medium | Single pass, reset on negative surplus | 134 |
| 4 | Assign Cookies | Easy | Sort both, two pointers | 455 |
| 5 | Boats to Save People | Medium | Sort, two pointers from both ends | 881 |
| 6 | Partition Labels | Medium | Last occurrence map, expand window | 763 |
| 7 | Minimum Number of Arrows to Burst Balloons | Medium | Same as non-overlapping intervals | 452 |

---

## 3. Exchange Argument

### When to Use
- You need to prove a sorting order is optimal
- "**Queue reconstruction**" by some attribute
- "**Largest number**" from array digits
- "**Hand of Straights**" / "**Divide array into groups**"

### When NOT to Use
- The optimal order isn't captured by a simple comparison function
- Items have complex interdependencies

### The Exchange Argument Technique

To prove a greedy ordering is optimal:
1. Assume the optimal solution has two adjacent elements in the "wrong" order
2. Show that swapping them doesn't make the solution worse
3. Therefore, sorting by the greedy criterion gives an optimal solution

### Optimized Java Template

```java
// Largest Number: arrange numbers to form the largest string
public String largestNumber(int[] nums) {
    String[] strs = new String[nums.length];
    for (int i = 0; i < nums.length; i++) strs[i] = String.valueOf(nums[i]);

    // Exchange argument: compare "ab" vs "ba"
    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b));

    if (strs[0].equals("0")) return "0";  // all zeros
    
    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s);
    return sb.toString();
}

// Queue Reconstruction by Height
public int[][] reconstructQueue(int[][] people) {
    // Sort: tallest first; if same height, fewer people-in-front first
    Arrays.sort(people, (a, b) -> a[0] != b[0]
        ? Integer.compare(b[0], a[0])
        : Integer.compare(a[1], b[1]));

    List<int[]> result = new ArrayList<>();
    for (int[] person : people) {
        result.add(person[1], person);  // insert at index = k value
    }

    return result.toArray(new int[people.length][]);
}

// Hand of Straights: divide into groups of consecutive cards
public boolean isNStraightHand(int[] hand, int groupSize) {
    if (hand.length % groupSize != 0) return false;

    TreeMap<Integer, Integer> counts = new TreeMap<>();
    for (int card : hand) counts.merge(card, 1, Integer::sum);

    while (!counts.isEmpty()) {
        int first = counts.firstKey();
        for (int i = first; i < first + groupSize; i++) {
            Integer cnt = counts.get(i);
            if (cnt == null) return false;
            if (cnt == 1) counts.remove(i);
            else counts.put(i, cnt - 1);
        }
    }

    return true;
}
```

### Fully Solved Problem: Largest Number (LC 179)

**Problem:** Given a list of non-negative integers, arrange them to form the largest number.

**Thinking process:**
1. This is an ordering problem — which number should come first?
2. Comparing `a` vs `b`: should we place `a` before `b` or `b` before `a`?
3. Compare by concatenation: if `"ab" > "ba"`, then `a` goes first
4. Exchange argument: if two adjacent numbers are in the wrong order, swapping improves the result

```java
public String largestNumber(int[] nums) {
    String[] strs = new String[nums.length];
    for (int i = 0; i < nums.length; i++) strs[i] = String.valueOf(nums[i]);

    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b));

    if (strs[0].equals("0")) return "0";

    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s);
    return sb.toString();
}
```

**Trace with `[3, 30, 34, 5, 9]`:**
- Compare pairs by concatenation: "330" vs "303" → 3 before 30
- Sort result: `["9", "5", "34", "3", "30"]`
- Answer: "9534330"

**Proof sketch (exchange argument):**
If optimal has `...b, a...` where our comparator says `a` should be before `b`, then `ab > ba` as strings. Swapping to `...a, b...` gives a larger number. Contradiction with optimality.

**Complexity:** O(n log n * k) where k = max digits per number (string comparison cost).

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Comparing numbers directly for Largest Number | `9 > 30` but `"309" > "930"` is wrong | Compare by concatenation: `(b+a).compareTo(a+b)` |
| Not handling all-zeros case | `[0, 0, 0]` → "000" instead of "0" | Check if first element is "0" after sorting |
| Queue reconstruction: wrong sort order | Insertions don't maintain k-value invariant | Tallest first, then by k ascending |
| Hand of Straights: using HashMap | No ordered access to smallest card | Use `TreeMap` for O(log n) first-key access |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Prove the comparator is transitive" | "If ab ≥ ba and bc ≥ cb, then ac ≥ ca. This can be shown by treating concatenation as a mathematical operation on numbers." |
| "Queue reconstruction time complexity?" | "O(n²) due to list insertions. With a BIT/segment tree, achievable in O(n log n)." |
| "What if Largest Number has leading zeros?" | "Only possible when all numbers are 0. We check `strs[0].equals('0')` after sorting." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Largest Number | Medium | Comparator: (b+a).compareTo(a+b) | 179 |
| 2 | Queue Reconstruction by Height | Medium | Sort tall→short, insert at k-index | 406 |
| 3 | Hand of Straights | Medium | TreeMap, greedily build groups from smallest | 846 |
| 4 | Divide Array in Sets of K Consecutive Numbers | Medium | Same as Hand of Straights | 1296 |
| 5 | Reduce Array Size to The Half | Medium | Sort by frequency, remove most frequent first | 1338 |
| 6 | Candy | Hard | Two-pass: left-to-right and right-to-left | 135 |

---

## Greedy vs DP Decision Framework

| Signal | Greedy | DP |
|--------|--------|-----|
| "Minimum/maximum **number of** (actions)" | Often greedy (Jump Game, coins in specific cases) | When choices interact (coin denominations) |
| "**All** possible ways" | Never greedy | Backtracking or DP |
| Can construct a counterexample? | Not greedy | Use DP |
| Sorting + single pass works? | Greedy | — |
| Overlapping subproblems? | Not greedy | DP |
| Items have weights AND values? | Usually not (except fractional knapsack) | 0/1 Knapsack DP |

**Classic trap:** Coin change with arbitrary denominations is DP, not greedy. Greedy (take largest coin) fails for `[1, 3, 4]` target 6: greedy gives `4+1+1`, optimal is `3+3`.

---

## Cross-References

- **Interval scheduling** greedy is in [Intervals](./10-intervals.md#2-scheduling-greedy-by-end-time)
- **Huffman coding** uses greedy with a min-heap — see [Heaps](./09-heaps.md)
- **Fractional Knapsack** is greedy; **0/1 Knapsack** is DP — see [DP](./07-dynamic-programming.md#4-01-knapsack)
- **Dijkstra's algorithm** is a greedy algorithm — see [Graphs](./06-graphs.md)
