# 16 — Interview Playbook

This is your tactical guide for the 45 minutes that matter. A perfect algorithm means nothing if you can't communicate it clearly, handle curveballs, or manage your time. This chapter covers the execution framework that turns pattern knowledge into offers.

---

## Table of Contents
1. [The 5-Minute Framework](#1-the-5-minute-framework)
2. [Universal Traps (Every Interview)](#2-universal-traps-every-interview)
3. [Communication Playbook](#3-communication-playbook)
4. [Counter-Questions by Pattern](#4-counter-questions-by-pattern)
5. [Mock Interview Rubric](#5-mock-interview-rubric)
6. [Time Management](#6-time-management)

---

## 1. The 5-Minute Framework

Before writing a single line of code, spend 5 minutes on this framework. It's the difference between a structured interview and a panicked one.

### Step 1: Clarify (1 min)
Ask these EVERY time, even if the answer seems obvious:

```
1. Input constraints:
   - "What's the size range of the input?" (determines O(n²) vs O(n log n) vs O(n))
   - "Can there be duplicates?"
   - "Is the input sorted?"
   - "Can values be negative?"

2. Output format:
   - "Return the indices or the values?"
   - "What if multiple valid answers exist?"
   - "What should I return for empty input?"

3. Edge cases (state, don't code):
   - "Empty array → return []"
   - "Single element → return [element]"
   - "All same elements → ..."
```

**Why this matters:** 30% of coding interview failures come from solving the wrong problem. A 60-second clarification prevents a 30-minute dead end.

### Step 2: Identify Pattern (1 min)

Use the [Pattern Recognition Table](./README.md) to map problem keywords to patterns:

```
Keywords → Pattern → Template

"sorted array" + "target" → Binary Search → lo/hi template
"substring" + "condition" → Sliding Window → variable window template
"all combinations" → Backtracking → combinations template
"shortest path" + "unweighted" → BFS → queue-based template
"minimum cost" + "choices" → DP → define state, transition, base case
```

**Say it out loud:** "This looks like a sliding window problem because we need the minimum substring that contains all characters."

### Step 3: Brute Force → Optimize (1 min)

Always state the brute force first:

```
"The brute force is to check all pairs: O(n²) time."
"Can we do better? Yes — with a hash map we can do O(n)."

OR

"The brute force is O(2^n) backtracking."
"With memoization, overlapping subproblems reduce this to O(n * amount)."
```

**Why brute force first?** It shows you understand the problem, gives you a fallback if you get stuck on the optimal, and the optimization becomes a clear upgrade.

### Step 4: Code (15-25 min)

```
- Write the function signature first
- Implement the core algorithm
- DON'T handle edge cases while coding (add them after)
- Use meaningful variable names: `left`, `right`, `maxLen`, not `i`, `j`, `m`
- If stuck for >3 min on a part, say so and move on
```

### Step 5: Test (3-5 min)

```
1. Trace through your code with a small example
2. Check edge cases: empty, single element, all same
3. Verify off-by-one: loop bounds, array indices
4. Confirm return value matches expected output
```

**Trace like this:** Write the variables as columns, step through line by line:
```
nums = [2, 7, 11, 15], target = 9
i=0: map={}, check 9-2=7 not in map, add {2:0}
i=1: map={2:0}, check 9-7=2 in map → return [0, 1] ✓
```

---

## 2. Universal Traps (Every Interview)

These mistakes kill otherwise correct solutions. Memorize them.

### Java-Specific Traps

| Trap | Example | Fix |
|------|---------|-----|
| Integer comparison with `==` | `new Integer(127) == new Integer(127)` → true, `128` → false | Always use `.equals()` or unbox to `int` |
| Integer overflow in arithmetic | `lo + hi` overflows when both > 10^9 | `lo + (hi - lo) / 2` |
| Integer overflow in comparator | `(a, b) -> a - b` overflows | `Integer.compare(a, b)` |
| `Stack` class | `Stack<Integer>` is synchronized, legacy | Use `ArrayDeque<Integer>` |
| `Arrays.asList()` returns fixed-size | Can't `.add()` or `.remove()` | Use `new ArrayList<>(Arrays.asList(...))` |
| `String` is immutable | `s += char` creates new String every time → O(n²) | Use `StringBuilder` |
| `HashMap.get()` returns `null` for missing keys | NPE when auto-unboxing: `int x = map.get(key)` | Use `getOrDefault()` or check `containsKey()` |
| `PriorityQueue` is min-heap by default | `pq.poll()` gives minimum, not maximum | `new PriorityQueue<>(Collections.reverseOrder())` |
| `char` arithmetic | `'a' + 1` returns `int`, not `char` | Cast: `(char)('a' + 1)` |
| `int` division truncates | `7 / 2 = 3`, not `3.5` | Cast: `(double) 7 / 2` or `7.0 / 2` |

### Algorithmic Traps

| Trap | Example | Fix |
|------|---------|-----|
| Not copying list when adding to result | Backtracking: all results are empty | `result.add(new ArrayList<>(curr))` |
| Modifying collection while iterating | `ConcurrentModificationException` | Use index-based loop or copy |
| Off-by-one in binary search | Infinite loop with `lo < hi` and `mid = lo + (hi - lo) / 2` | Ensure search space shrinks every iteration |
| BFS: not marking visited before enqueueing | Same node enqueued multiple times → TLE | Mark visited WHEN ADDING to queue, not when polling |
| DFS: not checking bounds first | `ArrayIndexOutOfBoundsException` | Check bounds before accessing `grid[r][c]` |
| Graph: not handling disconnected components | Missing nodes in traversal | Loop over all nodes, skip visited |
| DP: wrong base case | Cascade of wrong values | Trace dp[0], dp[1] by hand first |
| Sorting for duplicate skip without actually sorting | Skip logic fails | `Arrays.sort()` BEFORE the backtracking call |

### Complexity Traps

| Claim | Reality | Why It's Wrong |
|-------|---------|---------------|
| "HashMap is O(1)" | Amortized O(1), worst case O(n) | Hash collisions; mention amortized |
| "Sorting is O(n log n)" | True for comparison-based | Counting sort / radix sort can be O(n) |
| "Space is O(1)" when using recursion | Stack space is O(depth) | Always count recursion stack |
| "BFS is O(V + E)" | True for adjacency list | Adjacency matrix: O(V²) |

---

## 3. Communication Playbook

### What to Say (and When)

| Phase | Say This | Don't Say This |
|-------|----------|---------------|
| Reading problem | "Let me make sure I understand..." | *silent reading for 2 min* |
| Clarifying | "Can I assume the input is always valid?" | "I'll just assume..." |
| Pattern recognition | "This reminds me of [pattern] because..." | "I've seen this before" |
| Getting stuck | "I'm thinking about two approaches..." | *silence for 3 min* |
| Coding | "I'm initializing the window..." (brief) | *narrating every line* |
| Bug found | "I see the issue — the loop bound should be..." | "Wait, why is this wrong?" |
| Done | "Let me trace through an example to verify" | "I think that's right?" |

### How to Handle "I Don't Know"

```
GOOD: "I haven't seen this exact problem before, but it has properties of 
       [pattern]. Let me start with that approach and adjust."

GOOD: "My first instinct is [brute force]. Let me think about whether
       there's a way to optimize using [technique]."

BAD:  "I don't know how to solve this."
BAD:  "Can you give me a hint?"  (OK to ask, but exhaust your approaches first)
```

### How to Handle Hints

```
GOOD: "Ah, so you're suggesting I look at this as a graph problem. 
       That makes sense because [reason]. Let me restructure my approach."

BAD:  "Oh yeah, right." *immediately codes without understanding*
BAD:  "I was about to do that." (interviewer knows you weren't)
```

---

## 4. Counter-Questions by Pattern

When the interviewer asks follow-ups, these are the expected responses that demonstrate mastery. Organized by pattern for quick review.

### Arrays & Two Pointers
| Question | Response |
|----------|----------|
| "What if the array is unsorted?" | "I'd sort it first (O(n log n)) or use a hash map (O(n))." |
| "What about duplicates?" | "Sort + skip: `while (i < n && nums[i] == nums[i-1]) i++`" |
| "Can you do it in-place?" | "Two pointers: read pointer + write pointer, O(1) extra space." |

### Binary Search
| Question | Response |
|----------|----------|
| "How do you handle duplicates?" | "Use lower/upper bound templates: `lo < hi` with `hi = mid` or `lo = mid + 1`." |
| "What if the input isn't sorted?" | "Binary search on the answer space — search monotonic condition, not the array." |

### Trees
| Question | Response |
|----------|----------|
| "Recursive or iterative?" | "Recursive for clarity, iterative (with explicit stack) if stack depth is a concern or if the interviewer prefers." |
| "What about an unbalanced tree?" | "Worst case O(n) depth. If we need guaranteed O(log n), use a self-balancing BST (AVL/Red-Black)." |
| "Can you do it without extra space?" | "Morris traversal: O(1) space by threading the tree." |

### Graphs
| Question | Response |
|----------|----------|
| "BFS or DFS?" | "BFS for shortest unweighted path. DFS for exploring all paths or when we need backtracking." |
| "What about cycles?" | "Track visited set. For directed graphs, also track 'in current path' for cycle detection." |
| "What about weighted edges?" | "Dijkstra for non-negative weights, Bellman-Ford for negative weights." |

### Dynamic Programming
| Question | Response |
|----------|----------|
| "Top-down or bottom-up?" | "Same complexity. Top-down is easier to think about; bottom-up avoids stack overflow and is usually faster." |
| "Can you optimize space?" | "If `dp[i]` only depends on `dp[i-1]`, use rolling variables. If 2D, use rolling array (two rows)." |
| "How do you know it's DP?" | "Optimal substructure + overlapping subproblems. If 'counting ways' or 'min/max', likely DP." |

### Heaps
| Question | Response |
|----------|----------|
| "Why not just sort?" | "Sorting is O(n log n). Heap gives O(n log k) when k << n. Also, heap works with streams." |
| "Min-heap or max-heap?" | "For K-th largest: min-heap of size K (smallest of the K largest = K-th largest)." |

---

## 5. Mock Interview Rubric

Use this rubric for self-assessment during practice. Score yourself honestly.

### Scoring (1-4 per category)

| Category | 1 (Fail) | 2 (Weak) | 3 (Good) | 4 (Strong) |
|----------|----------|----------|----------|-----------|
| **Problem Comprehension** | Misunderstood the problem | Needed hints to clarify | Asked good questions | Identified all constraints and edge cases proactively |
| **Approach** | No viable approach | Brute force only | Optimal approach with some guidance | Optimal approach independently, stated trade-offs |
| **Coding** | Code doesn't compile | Compiles but major bugs | Minor bugs, self-corrected | Clean, correct code on first pass |
| **Testing** | No testing | Tested happy path only | Tested edge cases | Comprehensive testing with traces |
| **Communication** | Silent or unclear | Explained only when asked | Steady narration of thought process | Clear, structured communication throughout |
| **Time Management** | Didn't finish | Rushed at the end | Finished with time for optimization | Finished with time to discuss follow-ups |

### Score Interpretation

| Total | Likely Outcome |
|-------|---------------|
| 20-24 | Strong Hire |
| 16-19 | Hire / Lean Hire |
| 12-15 | Borderline |
| 8-11 | No Hire |
| 6-7 | Strong No Hire |

### Self-Practice Protocol

```
1. Set a 45-minute timer
2. Pick a random Medium/Hard from your target company's list
3. Explain your approach OUT LOUD (or record yourself)
4. Code without looking at solutions
5. Score yourself using the rubric above
6. Review the optimal solution and identify what you missed
7. Re-solve the same problem from scratch in 20 minutes (next day)
```

---

## 6. Time Management

### The 45-Minute Budget

```
0:00 - 1:00  Read and understand the problem
1:00 - 2:00  Ask clarifying questions
2:00 - 3:00  Identify pattern and state brute force
3:00 - 5:00  Describe optimal approach and get buy-in
5:00 - 30:00 Code the solution (25 minutes)
30:00 - 35:00 Test with examples
35:00 - 40:00 Discuss complexity and optimizations
40:00 - 45:00 Follow-up questions
```

### When You're Stuck

| Time Stuck | Action |
|-----------|--------|
| < 2 min | Think silently, it's normal |
| 2-3 min | Think out loud: "I'm considering X but stuck on Y..." |
| 3-5 min | Try a different approach entirely |
| > 5 min | Ask for a hint: "Could you point me in the right direction?" |

**"Productive stuck" vs "silent stuck":** Interviewers expect you to get stuck. What matters is whether you're VISIBLY making progress (drawing examples, trying small cases, reasoning about properties) or just staring silently.

### Red Flags (What Interviewers Look For Negatively)

| Red Flag | What It Signals |
|----------|----------------|
| Jumping straight to code | Doesn't think before acting |
| Not asking any questions | Doesn't communicate, makes assumptions |
| Silent for > 2 minutes | Can't think under pressure |
| "I've seen this problem before" | Memorized, doesn't understand |
| Defensive about bugs | Fragile ego, hard to work with |
| Can't explain their code | Memorized a template, doesn't understand it |
| Ignoring interviewer hints | Not collaborative, poor listening |
| Not testing their code | Doesn't care about correctness |

### Green Flags (What Impresses Interviewers)

| Green Flag | What It Signals |
|-----------|----------------|
| Asks about edge cases before coding | Thorough, thinks before acting |
| States brute force, then optimizes | Strong problem-solving process |
| Catches own bug before running | Attention to detail |
| Discusses trade-offs unprompted | Engineering maturity |
| Handles follow-ups gracefully | Deep understanding, not memorized |
| Admits when unsure, then reasons through | Honest, strong thinker |
| Writes clean code with good names | Will write maintainable production code |

---

## Pre-Interview Checklist

### Night Before
- [ ] Review pattern templates (don't solve new problems)
- [ ] Review company-specific tips from [Chapter 15](./15-company-specific.md)
- [ ] Prepare 3 STAR stories for behavioral questions
- [ ] Set up your coding environment (IDE, font size, etc.)
- [ ] Get 7+ hours of sleep

### 30 Minutes Before
- [ ] Review the 5-Minute Framework
- [ ] Review Java-specific traps (Integer cache, overflow, comparator)
- [ ] Quick glance at the complexity cheat sheet
- [ ] Breathe. You've practiced. Trust your preparation.

### During the Interview
- [ ] Clarify the problem (Step 1)
- [ ] State the pattern and approach (Steps 2-3)
- [ ] Get interviewer buy-in before coding
- [ ] Code cleanly (Step 4)
- [ ] Test with an example (Step 5)
- [ ] Discuss complexity proactively
- [ ] Handle follow-ups with confidence

---

## Cross-References

- Pattern Recognition Table → [README](./README.md)
- All pattern templates → [Chapters 01-14](./01-arrays-and-hashing.md)
- Company-specific preparation → [Chapter 15](./15-company-specific.md)
- Java-specific gotchas → [README Java Gotchas section](./README.md)
