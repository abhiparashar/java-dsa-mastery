# 03 — Stacks & Queues

Stack and queue problems are among the most commonly asked in phone screens. The monotonic stack pattern alone covers ~15 frequently asked problems. Always use `ArrayDeque` in Java — never `Stack`.

---

## Table of Contents
1. [Monotonic Stack](#1-monotonic-stack)
2. [Expression Parsing](#2-expression-parsing)
3. [Special Stack Designs](#3-special-stack-designs)
4. [Monotonic Queue (Deque)](#4-monotonic-queue-deque)

---

## 1. Monotonic Stack

### When to Use (Recognition Signals)
- "**Next greater element**" / "next smaller element"
- "**Previous greater element**" / "previous smaller element"
- "**Daily temperatures**" (how many days until warmer)
- "**Largest rectangle** in histogram"
- Any problem where you need to find the nearest element satisfying a comparison

### When NOT to Use
- You need the Kth greater element (not just the nearest) → use heap or sorted structure
- The comparison isn't based on value ordering → monotonic stack needs a clear ordering

### Optimized Java Template

```java
// UNIFIED Template: Next Greater / Next Smaller / Previous Greater / Previous Smaller
// Configure by changing: (1) scan direction, (2) comparison operator, (3) when to record result

// Next Greater Element (scan left → right, decreasing stack)
public int[] nextGreater(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();  // stores INDICES

    for (int i = 0; i < n; i++) {
        // Pop elements that are SMALLER than current → current is their "next greater"
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}

// Next Smaller Element (scan left → right, increasing stack)
public int[] nextSmaller(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] > nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}

// Previous Greater Element (scan left → right, decreasing stack, record on push)
public int[] previousGreater(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] <= nums[i]) {
            stack.pop();
        }
        if (!stack.isEmpty()) {
            result[i] = nums[stack.peek()];  // top of stack is previous greater
        }
        stack.push(i);
    }
    return result;
}
```

**Why each line matters:**
- `Deque<Integer>` not `Stack<Integer>`: `Stack` extends `Vector` which is synchronized — unnecessary overhead. `ArrayDeque` is 2-3x faster.
- **Store indices, not values**: indices give you both the value (`nums[idx]`) AND the position. You'll need position for problems like "how many days until warmer."
- **Amortized O(n)**: each element is pushed once and popped at most once → total operations = 2n → O(n).

### Fully Solved Problem: Largest Rectangle in Histogram (LC 84)

**Problem:** Given an array of bar heights, find the largest rectangle that can be formed.

**Thinking Process:**
1. Brute force: for each bar, expand left and right to find the largest rectangle with that bar's height → O(n²).
2. Key insight: for each bar `i`, the rectangle's width is determined by the **nearest shorter bar on the left** and the **nearest shorter bar on the right**. Width = `rightSmaller[i] - leftSmaller[i] - 1`.
3. We can compute both "previous smaller" and "next smaller" using a monotonic increasing stack in a single pass.

**Solution:**
```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;

    for (int i = 0; i <= n; i++) {
        int currHeight = (i == n) ? 0 : heights[i];  // sentinel: height 0 forces all remaining bars to pop

        while (!stack.isEmpty() && heights[stack.peek()] > currHeight) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }

    return maxArea;
}
```

**Why this works:**
- We maintain an **increasing** stack of heights.
- When we encounter a bar shorter than the top, the top bar can no longer extend right → we calculate its area.
- **Width calculation**: `i - stack.peek() - 1`. The bar at `stack.peek()` is the previous smaller bar (left boundary). `i` is the next smaller bar (right boundary). Width = right - left - 1.
- If stack is empty after popping, the popped bar was the smallest so far → width extends to the beginning = `i`.
- **Sentinel trick**: appending a virtual bar of height 0 at position `n` forces all remaining bars in the stack to be processed.

**Walkthrough with `[2, 1, 5, 6, 2, 3]`:**
```
i=0, h=2: stack=[], push 0. stack=[0]
i=1, h=1: h[0]=2 > 1. Pop 0. height=2, stack empty → width=1. area=2. Push 1. stack=[1]
i=2, h=5: push 2. stack=[1,2]
i=3, h=6: push 3. stack=[1,2,3]
i=4, h=2: h[3]=6 > 2. Pop 3. height=6, width=4-2-1=1. area=6.
          h[2]=5 > 2. Pop 2. height=5, width=4-1-1=2. area=10. ← max so far
          Push 4. stack=[1,4]
i=5, h=3: push 5. stack=[1,4,5]
i=6, h=0 (sentinel): h[5]=3 > 0. Pop 5. height=3, width=6-4-1=1. area=3.
                      h[4]=2 > 0. Pop 4. height=2, width=6-1-1=4. area=8.
                      h[1]=1 > 0. Pop 1. height=1, stack empty → width=6. area=6.

maxArea = 10
```

### Complexity Analysis
- **Time:** O(n) — each element pushed and popped exactly once
- **Space:** O(n) — stack holds at most n elements

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Forgetting the sentinel at the end | Elements remaining in stack don't get processed | Use `i <= n` with `currHeight = (i == n) ? 0 : heights[i]` |
| Width calculation when stack is empty | `stack.peek()` throws exception | Check `stack.isEmpty()` → width = `i` |
| Strict vs non-strict comparison (`<` vs `<=`) | For equal heights, `<` keeps equal elements (correct for "next greater"), `<=` removes them | Think about what you're computing: "strictly greater" or "greater or equal" |
| Using monotonic stack for max sliding window | Works but O(n) deque approach exists (see [Pattern 4](#4-monotonic-queue-deque)) | Monotonic deque is the right tool for sliding window |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Now solve Maximal Rectangle (LC 85)" | "For each row, compute histogram heights (cumulative from top). Run Largest Rectangle in Histogram on each row's histogram. O(m*n)." |
| "What about Trapping Rain Water?" | "Two approaches: (1) Two pointers maintaining leftMax/rightMax — O(1) space. (2) Monotonic stack — pop when current > top, compute trapped water. Both O(n)." |
| "Can you prove the amortized O(n)?" | "Each of n elements is pushed exactly once and popped at most once. Total push operations = n, total pop operations ≤ n. So total = 2n = O(n)." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Next Greater Element I | Easy | HashMap for mapping + monotonic stack | 496 |
| 2 | Next Greater Element II | Medium | Circular array → iterate 2n with modulo | 503 |
| 3 | Daily Temperatures | Medium | Next greater element = first warmer day | 739 |
| 4 | Online Stock Span | Medium | Previous greater element (count consecutive smaller/equal) | 901 |
| 5 | Remove K Digits | Medium | Monotonic increasing stack, remove larger digits | 402 |
| 6 | Remove Duplicate Letters | Medium | Monotonic increasing stack + last occurrence tracking | 316 |
| 7 | Largest Rectangle in Histogram | Hard | Solved above | 84 |
| 8 | Maximal Rectangle | Hard | Row-by-row histogram + LC 84 | 85 |
| 9 | Trapping Rain Water | Hard | Stack-based or two-pointer approach | 42 |

---

## 2. Expression Parsing

### When to Use (Recognition Signals)
- "**Evaluate** an expression"
- "**Valid parentheses**" / matching brackets
- "**Decode string**" (nested encoding)
- "**Calculator**" problems
- Anything with **nested** or **recursive** structure in a string

### When NOT to Use
- Simple string matching without nesting → regular expressions or two pointers
- The expression has no operator precedence → might not need a stack

### Optimized Java Template

```java
// Template A: Valid Parentheses (matching brackets)
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();

    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');      // push expected closing
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }

    return stack.isEmpty();
}

// Template B: Basic Calculator II (handles +, -, *, /)
// No parentheses version — handles precedence with delayed evaluation
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int num = 0;
    char prevOp = '+';

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);

        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        }

        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            switch (prevOp) {
                case '+': stack.push(num); break;
                case '-': stack.push(-num); break;
                case '*': stack.push(stack.pop() * num); break;
                case '/': stack.push(stack.pop() / num); break;
            }
            prevOp = c;
            num = 0;
        }
    }

    int result = 0;
    for (int val : stack) result += val;
    return result;
}

// Template C: Decode String — "3[a2[c]]" → "accaccacc"
public String decodeString(String s) {
    Deque<StringBuilder> strStack = new ArrayDeque<>();
    Deque<Integer> countStack = new ArrayDeque<>();
    StringBuilder curr = new StringBuilder();
    int num = 0;

    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '[') {
            countStack.push(num);
            strStack.push(curr);
            curr = new StringBuilder();
            num = 0;
        } else if (c == ']') {
            int count = countStack.pop();
            StringBuilder prev = strStack.pop();
            String repeated = curr.toString().repeat(count);
            curr = prev.append(repeated);
        } else {
            curr.append(c);
        }
    }

    return curr.toString();
}
```

**Why each line matters:**
- **Valid Parentheses trick**: push the EXPECTED closing bracket instead of the opening bracket. Then comparison is just `stack.pop() != c` — no mapping needed.
- **Calculator II**: the key insight is `prevOp` — we process each number with the PREVIOUS operator. This naturally handles precedence: `*` and `/` are applied immediately (pop and push result), while `+` and `-` just push (deferred to the final sum).
- **Decode String**: two stacks — one for counts, one for partially built strings. On `[`, save current state and start fresh. On `]`, combine.

### Fully Solved Problem: Basic Calculator (LC 224) — with parentheses, +, -

**Problem:** Evaluate expression with `+`, `-`, `(`, `)`, spaces, and non-negative integers.

**Thinking Process:**
1. Without parentheses: just track sign and accumulate.
2. With parentheses: when we hit `(`, save current result and sign to stack. Start fresh inside parens. When we hit `)`, combine with saved state.

**Solution:**
```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int result = 0;
    int num = 0;
    int sign = 1;

    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num;
            num = 0;
            sign = 1;
        } else if (c == '-') {
            result += sign * num;
            num = 0;
            sign = -1;
        } else if (c == '(') {
            stack.push(result);  // save current result
            stack.push(sign);    // save current sign
            result = 0;
            sign = 1;
        } else if (c == ')') {
            result += sign * num;
            num = 0;
            result *= stack.pop();  // saved sign
            result += stack.pop();  // saved result
        }
    }

    return result + sign * num;  // handle last number
}
```

### Complexity Analysis
- **Valid Parentheses:** O(n) time, O(n) space
- **Calculator II:** O(n) time, O(n) space
- **Calculator I (with parens):** O(n) time, O(n) space (depth of nesting)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Forgetting the last number | Expression `"1+2"` — the `2` is never processed because no operator follows it | Add `i == s.length() - 1` check, or process after loop |
| Integer overflow in multi-digit parsing | `num = num * 10 + (c - '0')` can overflow | Use `long` for intermediate parsing if values can be large |
| Spaces in the expression | `' '` is neither digit nor operator, causes bugs | Explicitly skip spaces or handle in the `else` branch |
| Calculator with parentheses: sign before `(` | `-(3+2)` — the sign applies to the entire sub-expression | Save sign to stack, multiply when closing paren |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Now handle `*` and `/` too (Basic Calculator III, LC 772)" | "Combine Calculator I and II. Use recursive descent: treat `(` as starting a sub-expression. Or use two stacks (operators + operands) with precedence." |
| "Can you do it without a stack?" | "For Calculator I with only `+` and `-`: yes, use a variable for the running sign multiplier. But parentheses with `*`/`/` needs a stack (or recursion, which is implicit stack)." |
| "What about unary minus like `-3+2`?" | "Handle by checking if `-` appears at the start or after `(`. In that case, prepend a `0`: treat as `0-3+2`." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Valid Parentheses | Easy | Push expected closing bracket | 20 |
| 2 | Remove Outermost Parentheses | Easy | Track depth, skip depth-0 parens | 1021 |
| 3 | Decode String | Medium | Two stacks: counts + strings | 394 |
| 4 | Evaluate Reverse Polish Notation | Medium | Stack: pop two operands, push result | 150 |
| 5 | Min Remove to Make Valid Parentheses | Medium | Track unmatched indices with stack | 1249 |
| 6 | Simplify Path | Medium | Stack of directory names, handle `.` and `..` | 71 |
| 7 | Basic Calculator | Hard | Sign tracking + stack for parentheses | 224 |
| 8 | Basic Calculator II | Medium | Delayed evaluation with prevOp | 227 |

---

## 3. Special Stack Designs

### When to Use (Recognition Signals)
- "Design a stack that supports **getMin** in O(1)"
- "**Implement queue using stacks**"
- "**Implement stack using queues**"
- Any custom data structure combining stack/queue with additional O(1) operations

### When NOT to Use
- You need getMin AND arbitrary deletion → use TreeMap or balanced BST
- You need median → use two heaps (see [Heaps](./09-heaps.md))

### Optimized Java Template

```java
// Template A: MinStack — Single stack approach (space-optimized)
class MinStack {
    private Deque<Long> stack;  // stores difference from current min
    private long min;

    public MinStack() {
        stack = new ArrayDeque<>();
    }

    public void push(int val) {
        if (stack.isEmpty()) {
            stack.push(0L);
            min = val;
        } else {
            stack.push((long) val - min);  // store difference
            if (val < min) min = val;
        }
    }

    public void pop() {
        long diff = stack.pop();
        if (diff < 0) {
            min = min - diff;  // restore previous min
        }
    }

    public int top() {
        long diff = stack.peek();
        return (int) (diff < 0 ? min : min + diff);
    }

    public int getMin() {
        return (int) min;
    }
}

// Template B: MinStack — Two stack approach (simpler, recommended in interviews)
class MinStackSimple {
    private Deque<Integer> stack;
    private Deque<Integer> minStack;

    public MinStackSimple() {
        stack = new ArrayDeque<>();
        minStack = new ArrayDeque<>();
    }

    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }

    public void pop() {
        if (stack.pop().equals(minStack.peek())) {
            minStack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return minStack.peek();
    }
}

// Template C: Queue using Two Stacks (amortized O(1) per operation)
class MyQueue {
    private Deque<Integer> pushStack;
    private Deque<Integer> popStack;

    public MyQueue() {
        pushStack = new ArrayDeque<>();
        popStack = new ArrayDeque<>();
    }

    public void push(int x) {
        pushStack.push(x);
    }

    public int pop() {
        transferIfNeeded();
        return popStack.pop();
    }

    public int peek() {
        transferIfNeeded();
        return popStack.peek();
    }

    public boolean empty() {
        return pushStack.isEmpty() && popStack.isEmpty();
    }

    private void transferIfNeeded() {
        if (popStack.isEmpty()) {
            while (!pushStack.isEmpty()) {
                popStack.push(pushStack.pop());
            }
        }
    }
}
```

**Why each line matters:**
- **MinStack single-stack**: stores `val - min` as the difference. If difference is negative, val became the new min. On pop, if difference was negative, we can recover the previous min as `min - diff`. This uses O(1) extra space per element (just one stack). But it requires `long` to avoid overflow.
- **MinStack two-stack**: simpler — just maintain a parallel stack of minimums. `<=` (not `<`) is critical: if we push the same minimum twice, we need to track both occurrences.
- **Queue from two stacks**: the key insight is that reversing a stack reverses the order. Push stack has newest on top; transfer to pop stack puts oldest on top (queue order). Transfer is lazy — only when pop stack is empty.
- **Amortized O(1)**: each element is transferred at most once. Over n operations, total transfers ≤ n → amortized O(1).

### Fully Solved Problem: Min Stack (LC 155)

**Problem:** Design a stack supporting push, pop, top, and getMin — all in O(1).

**Solution:** Template B above (two-stack approach — recommended for interviews because it's simpler to explain).

**Critical detail in pop():**
```java
// WRONG: stack.pop() == minStack.peek()
// Integer comparison with == fails for values outside [-128, 127]

// CORRECT: stack.pop().equals(minStack.peek())
```

This is the #1 Java trap in this problem. `Integer` objects are cached only for -128 to 127. Beyond that range, `==` compares references, not values.

### Complexity Analysis
- **MinStack:** O(1) time for all operations, O(n) space
- **Queue from Stacks:** O(1) amortized for all operations, O(n) space
- **Stack from Queue:** O(n) for push (or pop, depending on design), O(1) for the other

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| `Integer` comparison with `==` in MinStack pop | Works for small values, silently fails for large values | Always use `.equals()` for `Integer` comparison |
| MinStack: using `<` instead of `<=` for pushing to min stack | Duplicate min values aren't tracked; popping one loses the min | Use `<=` to push equal values too |
| Queue from stacks: transferring on every pop | O(n) per pop instead of amortized O(1) | Only transfer when pop stack is empty |
| Single-stack MinStack: integer overflow | `val - min` can overflow int range | Use `long` for the stack values |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you do MinStack with O(1) space?" | "The single-stack approach stores differences, using O(1) extra beyond the stack itself. But the stack is still O(n). True O(1) extra is impossible if you need to store n elements." |
| "What about MaxStack with popMax?" | "Two stacks won't work because popMax needs to remove from the middle. Use a doubly-linked list + TreeMap for O(log n) operations. LC 716." |
| "Queue from stacks: what's the WORST case for a single pop?" | "O(n) — when pop stack is empty and all n elements must transfer. But amortized over n operations, total work is O(n) → O(1) per operation." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Min Stack | Easy | Two-stack or difference-based approach | 155 |
| 2 | Implement Queue using Stacks | Easy | Two stacks with lazy transfer | 232 |
| 3 | Implement Stack using Queues | Easy | Push to front by rotating after each push | 225 |
| 4 | Max Stack | Hard | Doubly-linked list + TreeMap for O(log n) popMax | 716 |

---

## 4. Monotonic Queue (Deque)

### When to Use (Recognition Signals)
- "**Maximum/minimum in sliding window**"
- "**Shortest subarray with sum ≥ K**" (with negative numbers)
- Any problem where you need the extremum of a moving range AND the range shrinks/grows from one or both ends

### When NOT to Use
- Fixed window with only sum/average needed → simple sliding window suffices
- You need Kth largest in window (not just max/min) → use augmented BST or two heaps
- No sliding window structure → probably a different pattern

### Optimized Java Template

```java
// Monotonic Decreasing Deque for Sliding Window Maximum
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();  // stores indices

    for (int i = 0; i < n; i++) {
        // Remove indices outside window from front
        if (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
            deque.pollFirst();
        }

        // Remove smaller elements from back (they're useless)
        while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }

        deque.offerLast(i);

        if (i >= k - 1) {
            result[i - k + 1] = nums[deque.peekFirst()];
        }
    }

    return result;
}

// Monotonic Increasing Deque for Sliding Window Minimum
// Same structure, flip the comparison:
// while (!deque.isEmpty() && nums[deque.peekLast()] >= nums[i])
```

**Why this is a separate pattern from Monotonic Stack:**
- Monotonic Stack: elements enter and leave from the **same end** (LIFO). Good for "next greater/smaller" problems.
- Monotonic Deque: elements enter from **one end** and can leave from **both ends** (front when they leave the window, back when they're dominated). Good for "extremum in sliding window" problems.

### Fully Solved Problem: Shortest Subarray with Sum at Least K (LC 862)

**Problem:** Given an integer array (may have negatives), find the shortest subarray with sum >= K.

**Thinking Process:**
1. If all positive: sliding window works (sum is monotonically increasing as we expand).
2. With negatives: sliding window breaks. We need prefix sums.
3. For each `j`, we want the largest `i < j` such that `prefix[j] - prefix[i] >= K`, i.e., `prefix[i] <= prefix[j] - K`.
4. Key insight: maintain a monotonic increasing deque of prefix sum indices. For each `j`, poll from front while `prefix[front] <= prefix[j] - K` (they give valid subarrays — record length). Also, poll from back while `prefix[back] >= prefix[j]` (if a later position has a smaller prefix sum, earlier larger ones will never be optimal).

**Solution:**
```java
public int shortestSubarray(int[] nums, int k) {
    int n = nums.length;
    long[] prefix = new long[n + 1];
    for (int i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }

    Deque<Integer> deque = new ArrayDeque<>();  // stores indices into prefix[]
    int minLen = Integer.MAX_VALUE;

    for (int i = 0; i <= n; i++) {
        // Front: poll indices where prefix[i] - prefix[front] >= k
        while (!deque.isEmpty() && prefix[i] - prefix[deque.peekFirst()] >= k) {
            minLen = Math.min(minLen, i - deque.pollFirst());
        }

        // Back: maintain increasing prefix sums
        while (!deque.isEmpty() && prefix[deque.peekLast()] >= prefix[i]) {
            deque.pollLast();
        }

        deque.offerLast(i);
    }

    return minLen == Integer.MAX_VALUE ? -1 : minLen;
}
```

### Complexity Analysis
- **Sliding Window Maximum:** O(n) time, O(k) space
- **Shortest Subarray with Sum ≥ K:** O(n) time, O(n) space

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using `LinkedList` as deque | Node allocation per element, poor cache locality | `ArrayDeque` is backed by a circular array — much faster |
| Not using `long` for prefix sums | Overflow with large values and negative numbers | Always `long[]` for prefix sums in competitive programming |
| Removing from front AFTER pushing | Can remove the element we just added, causing incorrect results | Remove from front first, then from back, then push |
| Confusing monotonic deque with monotonic stack | Stack only uses one end; deque uses both | Stack = "next greater/smaller". Deque = "window max/min" |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Why not use a heap for sliding window max?" | "Heap gives O(n log k) — worse than deque's O(n). Heap can't efficiently remove elements that leave the window (needs lazy deletion). Deque removes from front in O(1)." |
| "What if we need both max AND min of each window?" | "Use two deques: one monotonic decreasing (for max), one monotonic increasing (for min). Process them independently." |
| "Shortest Subarray Sum ≥ K: why can't we use sliding window?" | "Negative numbers break the monotonicity of the running sum. Expanding the window might decrease the sum, so we can't make shrink decisions. Prefix sum + monotonic deque handles this." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Sliding Window Maximum | Hard | Monotonic decreasing deque | 239 |
| 2 | Shortest Subarray with Sum at Least K | Hard | Prefix sum + monotonic increasing deque | 862 |
| 3 | Constrained Subsequence Sum | Hard | DP + monotonic deque for max in window | 1425 |
| 4 | Jump Game VI | Medium | DP + sliding window max (monotonic deque) | 1696 |
| 5 | Longest Continuous Subarray with Abs Diff ≤ Limit | Medium | Two deques: one for max, one for min | 1438 |

---

## Cross-References

- **Sliding Window Maximum** is also introduced in [Arrays — Fixed Sliding Window](./01-arrays-and-hashing.md#2-sliding-window--fixed-size) with simpler context
- **Monotonic Stack for histograms** is foundational for [Matrix DP](./07-dynamic-programming.md#2-2d-dp-gridmatrix-paths) — Maximal Rectangle uses it
- **Expression Parsing** uses the same recursive structure as [Backtracking](./08-backtracking.md) — both process nested structures
- **Queue from Stacks** concept appears in [BFS](./06-graphs.md) — BFS uses a queue
