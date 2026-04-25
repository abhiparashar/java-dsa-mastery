# 02 — Linked Lists

Linked list problems test pointer manipulation under pressure. The key to mastery: always use a **dummy/sentinel node** to eliminate head-modification edge cases, and draw the pointers on paper before coding.

---

## Table of Contents
1. [Fast/Slow Pointers (Floyd's)](#1-fastslow-pointers-floyds)
2. [Reversal Patterns](#2-reversal-patterns)
3. [Merge Patterns](#3-merge-patterns)
4. [In-place Manipulation](#4-in-place-manipulation)

---

## 1. Fast/Slow Pointers (Floyd's)

### When to Use (Recognition Signals)
- "**Detect a cycle**" in linked list
- "Find the **start of the cycle**"
- "Find the **middle** of a linked list"
- "Find the **kth node from the end**"
- "Check if a linked list is a **palindrome**"

### When NOT to Use
- You need to detect a cycle in a directed graph with multiple outgoing edges → use DFS coloring
- The list is doubly-linked → you can just traverse backward, no need for fast/slow

### Optimized Java Template

```java
// Template A: Find Middle (for even-length, returns first middle)
public ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;  // first middle for even-length lists
}

// Template B: Cycle Detection + Find Cycle Start (Floyd's Algorithm)
public ListNode detectCycleStart(ListNode head) {
    ListNode slow = head, fast = head;

    // Phase 1: detect cycle
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }

    // No cycle
    if (fast == null || fast.next == null) return null;

    // Phase 2: find cycle start
    // Mathematical proof: distance from head to cycle start = distance from meeting point to cycle start
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;  // both move at speed 1 now
    }
    return slow;
}

// Template C: Kth from End
public ListNode kthFromEnd(ListNode head, int k) {
    ListNode fast = head;
    for (int i = 0; i < k; i++) {
        fast = fast.next;  // advance fast by k steps
    }

    ListNode slow = head;
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow;  // slow is now at (n-k)th node = kth from end
}
```

**Why each line matters:**
- **Middle node**: `fast.next != null && fast.next.next != null` (not `fast != null && fast.next != null`) — this returns the FIRST middle for even-length lists. The other condition returns the second middle. Interview tip: clarify which one they want.
- **Floyd's Phase 2**: when slow and fast meet, the distance from head to cycle start equals the distance from meeting point to cycle start. This is because: let `a` = distance from head to cycle start, `b` = distance from cycle start to meeting point, `c` = cycle length. Fast travels `a + b + nc` (some full cycles), slow travels `a + b`. Since fast = 2 × slow: `a + b + nc = 2(a + b)` → `a = nc - b` → `a = (n-1)c + (c - b)`, meaning walking `a` steps from meeting point lands exactly at cycle start.

### Fully Solved Problem: Palindrome Linked List (LC 234)

**Problem:** Check if a singly linked list is a palindrome in O(1) extra space.

**Thinking Process:**
1. With O(n) space: copy to array, check palindrome. Easy but not O(1) space.
2. Key insight: find middle → reverse second half → compare with first half → (optionally) restore.
3. This combines TWO patterns: fast/slow to find middle, and reversal on the second half.

**Solution:**
```java
public boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;

    // Step 1: Find middle (first middle for even-length)
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: Reverse second half (starting from slow.next)
    ListNode secondHalf = reverse(slow.next);

    // Step 3: Compare first half with reversed second half
    ListNode p1 = head, p2 = secondHalf;
    boolean result = true;
    while (p2 != null) {
        if (p1.val != p2.val) {
            result = false;
            break;
        }
        p1 = p1.next;
        p2 = p2.next;
    }

    // Step 4 (optional): Restore the list
    slow.next = reverse(secondHalf);

    return result;
}

private ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

### Complexity Analysis
- **Time:** O(n) — find middle O(n/2) + reverse O(n/2) + compare O(n/2) = O(n)
- **Space:** O(1) — only pointer variables
- Floyd's cycle detection: O(n) time (fast pointer traverses at most 2n nodes), O(1) space

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Wrong middle for even-length list | Different behavior for `[1,2,2,1]` vs `[1,2,3,2,1]` | Clarify: `fast.next != null && fast.next.next != null` gives first middle |
| Not handling single-node list | Null pointer when accessing `head.next` | Add `if (head == null || head.next == null)` guard |
| Forgetting to restore the list | Original list is modified — some interviewers care | Reverse the second half back after comparison |
| Floyd's: checking `fast == slow` before first move | Both start at head, so they "meet" immediately | Always move FIRST, then check. Use do-while or check after the move. |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you find the cycle length?" | "After detection, keep one pointer at meeting point, advance the other until they meet again. Count the steps." |
| "What if we can't modify the list?" | "Use O(n) space: copy values to array, check palindrome. Or recursive approach with O(n) stack space." |
| "Is this thread-safe?" | "No — we're modifying the list structure. In a concurrent system, we'd need synchronization or the O(n) space approach." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Linked List Cycle | Easy | Fast meets slow → cycle exists | 141 |
| 2 | Middle of Linked List | Easy | Fast goes 2x, slow stops at middle | 876 |
| 3 | Palindrome Linked List | Easy | Find middle + reverse + compare | 234 |
| 4 | Remove Linked List Elements | Easy | Sentinel node + skip matching values | 203 |
| 5 | Linked List Cycle II | Medium | Floyd's Phase 2: head and meeting point converge at cycle start | 142 |
| 6 | Remove Nth Node From End | Medium | Fast leads by N, then both advance | 19 |
| 7 | Reorder List | Medium | Find middle + reverse second half + merge | 143 |
| 8 | Happy Number | Medium | Same concept: detect cycle in number sequence | 202 |
| 9 | Find the Duplicate Number | Medium | Floyd's on array: `next(i) = nums[i]` | 287 |

---

## 2. Reversal Patterns

### When to Use (Recognition Signals)
- "**Reverse** a linked list" (full or partial)
- "Reverse **between positions m and n**"
- "**Swap** nodes in pairs"
- "Reverse in **groups of k**"

### When NOT to Use
- You need to access elements by index repeatedly → array is better
- The problem is about reordering, not reversing → see In-place Manipulation

### Optimized Java Template

```java
// Template A: Full Reversal (iterative — O(1) space)
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;  // save next before overwriting
        curr.next = prev;           // reverse the link
        prev = curr;                // advance prev
        curr = next;                // advance curr
    }
    return prev;  // prev is the new head
}

// Template B: Reverse Between Positions m and n (1-indexed)
// Uses a sentinel node to handle m=1 edge case
public ListNode reverseBetween(ListNode head, int m, int n) {
    ListNode sentinel = new ListNode(0, head);
    ListNode prev = sentinel;

    // Move prev to the node BEFORE the reversal starts
    for (int i = 1; i < m; i++) {
        prev = prev.next;
    }

    // Reverse (n - m + 1) nodes starting from prev.next
    ListNode curr = prev.next;
    for (int i = 0; i < n - m; i++) {
        ListNode next = curr.next;
        curr.next = next.next;   // skip over 'next'
        next.next = prev.next;   // 'next' points to start of reversed portion
        prev.next = next;         // 'next' becomes new start of reversed portion
    }

    return sentinel.next;
}

// Template C: Reverse in K-Groups
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode sentinel = new ListNode(0, head);
    ListNode groupPrev = sentinel;

    while (true) {
        // Check if k nodes remain
        ListNode kth = getKth(groupPrev, k);
        if (kth == null) break;

        ListNode groupNext = kth.next;

        // Reverse the group
        ListNode prev = groupNext;
        ListNode curr = groupPrev.next;
        while (curr != groupNext) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        // Connect: groupPrev → kth (new first), old first → groupNext
        ListNode tmp = groupPrev.next;  // old first (now last in group)
        groupPrev.next = kth;           // point to new first (was kth)
        groupPrev = tmp;                // advance to end of reversed group
    }

    return sentinel.next;
}

private ListNode getKth(ListNode node, int k) {
    while (node != null && k > 0) {
        node = node.next;
        k--;
    }
    return node;
}
```

**Why each line matters:**
- **Sentinel node**: the single most important linked list technique. It eliminates special cases when the head changes. Without it, you'd need `if (m == 1)` branches everywhere.
- **Reverse Between** uses a "front insertion" technique: repeatedly take the next node and insert it at the front of the reversed portion. This avoids disconnecting and reconnecting pointers.
- **K-Group**: the key insight is reversing each group as a standalone list, where `prev` starts at `groupNext` (the node after the group). This automatically connects the reversed group to the rest.

### Fully Solved Problem: Reverse Nodes in k-Group (LC 25)

**Problem:** Reverse nodes in groups of k. If remaining nodes < k, leave them as-is.

**Thinking Process:**
1. For each group: count k nodes. If < k, stop.
2. Reverse the k nodes using standard reversal.
3. The tricky part: reconnecting groups. The last node of each reversed group must point to the first node of the next group.
4. Use a sentinel to handle the first group uniformly.

**Solution:** (Template C above)

**Walkthrough with `[1, 2, 3, 4, 5]`, k = 2:**
```
sentinel → 1 → 2 → 3 → 4 → 5

Group 1: groupPrev = sentinel, kth = 2, groupNext = 3
  Reverse [1, 2] with prev = 3:
    curr=1: 1→3, prev=1, curr=2
    curr=2: 2→1, prev=2, curr=3 (==groupNext, stop)
  sentinel → 2 → 1 → 3 → 4 → 5
  groupPrev = 1 (old first, now last)

Group 2: groupPrev = 1, kth = 4, groupNext = 5
  Reverse [3, 4] with prev = 5:
    curr=3: 3→5, prev=3, curr=4
    curr=4: 4→3, prev=4, curr=5 (==groupNext, stop)
  sentinel → 2 → 1 → 4 → 3 → 5
  groupPrev = 3

Group 3: groupPrev = 3, getKth(3, 2) needs 2 more nodes. Only 5 remains. Return null. Break.

Result: 2 → 1 → 4 → 3 → 5
```

### Complexity Analysis
- **Full reversal:** O(n) time, O(1) space
- **Reverse between:** O(n) time (traverse to position m + reverse n-m nodes), O(1) space
- **K-Group:** O(n) time (each node visited twice: once to count, once to reverse), O(1) space

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not saving `curr.next` before overwriting `curr.next = prev` | Lose the rest of the list | Always save: `ListNode next = curr.next` first |
| Forgetting sentinel node | Head modification requires special case code | Always use `new ListNode(0, head)` and return `sentinel.next` |
| Off-by-one in "reverse between m and n" | Reversing wrong number of nodes or wrong position | Use 1-indexed counting: loop `(n - m)` times, not `(n - m + 1)` |
| Recursive reversal: stack overflow for long lists | Java default stack size ~512KB, each frame ~50 bytes → ~10K max depth | Use iterative for production. Mention stack limit in interviews. |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Do it recursively" | "Recursive reversal: `reverse(node.next)` returns new head. Then `node.next.next = node; node.next = null`. O(n) space for stack." |
| "What if k > n?" | "Our solution handles this: `getKth` returns null, we break and don't reverse." |
| "Reverse alternating k-groups" | "Add a boolean flag. On every other group, skip the reversal and just advance `groupPrev` by k nodes." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Reverse Linked List | Easy | Three-pointer iterative reversal | 206 |
| 2 | Reverse Linked List II | Medium | Sentinel + front-insertion for partial reversal | 92 |
| 3 | Swap Nodes in Pairs | Medium | k-group reversal with k=2, or iterative pair swap | 24 |
| 4 | Odd Even Linked List | Medium | Separate odd/even lists, then connect | 328 |
| 5 | Rotate List | Medium | Form cycle, then break at (n - k%n)th position | 61 |
| 6 | Reverse Nodes in k-Group | Hard | Solved above | 25 |

---

## 3. Merge Patterns

### When to Use (Recognition Signals)
- "**Merge two sorted** lists"
- "**Sort** a linked list" (merge sort is optimal for linked lists)
- "**Merge k sorted** lists"
- "**Add two numbers** represented as linked lists"

### When NOT to Use
- Lists aren't sorted and you don't need to sort → probably a manipulation problem
- You need random access during merge → convert to array first

### Optimized Java Template

```java
// Template A: Merge Two Sorted Lists (iterative with sentinel)
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode sentinel = new ListNode(0);
    ListNode tail = sentinel;

    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            tail.next = l1;
            l1 = l1.next;
        } else {
            tail.next = l2;
            l2 = l2.next;
        }
        tail = tail.next;
    }

    tail.next = (l1 != null) ? l1 : l2;  // append remaining
    return sentinel.next;
}

// Template B: Merge Sort for Linked List (O(n log n) time, O(1) extra space)
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    // Split into two halves
    ListNode mid = findMiddle(head);
    ListNode rightHead = mid.next;
    mid.next = null;  // cut the link

    // Recursively sort each half
    ListNode left = sortList(head);
    ListNode right = sortList(rightHead);

    // Merge sorted halves
    return mergeTwoLists(left, right);
}

private ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

// Template C: Merge K Sorted Lists (Divide and Conquer — O(n log k))
public ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    return mergeRange(lists, 0, lists.length - 1);
}

private ListNode mergeRange(ListNode[] lists, int lo, int hi) {
    if (lo == hi) return lists[lo];
    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeRange(lists, lo, mid);
    ListNode right = mergeRange(lists, mid + 1, hi);
    return mergeTwoLists(left, right);
}
```

**Why each line matters:**
- **Merge K (D&C vs Heap)**: Divide and conquer merges pairs bottom-up in log(k) rounds. Each round processes all N elements → O(N log k). Heap approach: maintain a min-heap of k list heads, extract-min and add next → also O(N log k) but with higher constant factor (heap operations vs simple pointer comparison). D&C is simpler and often faster in practice.
- **Merge Sort on linked list** is special: unlike arrays, splitting a linked list in half takes O(n) (find middle), but merge is in-place (no extra array needed). So merge sort on linked lists is O(n log n) time with O(log n) recursion space — compared to O(n) extra space for array merge sort.

### Fully Solved Problem: Merge k Sorted Lists (LC 23)

**Problem:** Merge k sorted linked lists into one sorted list.

**Thinking Process:**
1. Naive: merge lists one by one → first two, then result with third, etc. Time: O(kN) where N = total nodes. Because each merge is O(n1 + n2), and we do k-1 merges, early lists get re-scanned repeatedly.
2. Better: pair-wise merge (divide and conquer). Merge in pairs: k lists → k/2 → k/4 → ... → 1. Each round processes N nodes → O(N log k).
3. Alternative: Min-heap. Always pick the smallest head among k lists. O(N log k). 

**Heap-based Solution (alternative perspective):**
```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)  // safe comparator, no overflow
    );

    for (ListNode head : lists) {
        if (head != null) pq.offer(head);
    }

    ListNode sentinel = new ListNode(0);
    ListNode tail = sentinel;

    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        tail.next = node;
        tail = tail.next;
        if (node.next != null) {
            pq.offer(node.next);
        }
    }

    return sentinel.next;
}
```

### Complexity Analysis
- **Merge two lists:** O(n + m) time, O(1) space
- **Merge K (D&C):** O(N log k) time, O(log k) recursion space
- **Merge K (Heap):** O(N log k) time, O(k) heap space
- **Sort List:** O(n log n) time, O(log n) recursion space

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| `(a, b) -> a.val - b.val` comparator | Integer overflow if values are near Integer.MIN_VALUE or MAX_VALUE | Use `Integer.compare(a.val, b.val)` |
| Not handling null lists in the input array | NullPointerException when adding to heap | Guard: `if (head != null) pq.offer(head)` |
| Merge sort: wrong middle for 2-node list | Infinite recursion if `findMiddle` returns the last node instead of first | Use `fast.next != null && fast.next.next != null` to get first middle |
| Not cutting the link in merge sort | Both halves share the same tail, merge produces a cycle | `mid.next = null` — essential! |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "D&C vs Heap — which is better?" | "Same asymptotic complexity. D&C has lower constant factor (simple comparisons vs heap operations). Heap is better when lists arrive incrementally (streaming). D&C needs all lists upfront." |
| "Can you sort a linked list in O(1) extra space?" | "Bottom-up merge sort avoids recursion stack. Merge sublists of size 1, then 2, then 4, etc. O(1) space but more complex to implement." |
| "Add Two Numbers: what if numbers are stored in normal order (not reversed)?" | "LC 445. Either reverse both lists first, or use a stack to process digits from least significant. Recursion also works." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Merge Two Sorted Lists | Easy | Sentinel + compare heads | 21 |
| 2 | Add Two Numbers | Medium | Digit-by-digit addition with carry | 2 |
| 3 | Add Two Numbers II | Medium | Reverse first, or use stack | 445 |
| 4 | Sort List | Medium | Merge sort on linked list | 148 |
| 5 | Merge k Sorted Lists | Hard | D&C or min-heap | 23 |

---

## 4. In-place Manipulation

### When to Use (Recognition Signals)
- "**Reorder** list" (e.g., L0→Ln→L1→Ln-1→...)
- "**Partition** list around a value"
- "**Remove duplicates**" from sorted/unsorted list
- "**Copy** list with random pointer"
- Problem requires O(1) extra space with structural changes

### When NOT to Use
- You need to maintain the original list intact → create a copy first
- Random access is needed repeatedly → convert to array

### Optimized Java Template

```java
// Template: Split, Transform, Reconnect
// Most in-place linked list problems follow this meta-pattern:
// 1. Split the list into parts (using fast/slow or counting)
// 2. Transform one or more parts (reverse, etc.)
// 3. Reconnect the parts (interleave, append, etc.)

// Example: Reorder List (L0→Ln→L1→Ln-1→...)
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // Step 1: Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: Reverse second half
    ListNode second = reverse(slow.next);
    slow.next = null;  // cut first half

    // Step 3: Interleave (merge alternating)
    ListNode first = head;
    while (second != null) {
        ListNode tmp1 = first.next;
        ListNode tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}

// Copy List with Random Pointer — O(1) space trick
public Node copyRandomList(Node head) {
    if (head == null) return null;

    // Step 1: Interleave copy nodes — A → A' → B → B' → C → C'
    Node curr = head;
    while (curr != null) {
        Node copy = new Node(curr.val);
        copy.next = curr.next;
        curr.next = copy;
        curr = copy.next;
    }

    // Step 2: Set random pointers for copies
    curr = head;
    while (curr != null) {
        if (curr.random != null) {
            curr.next.random = curr.random.next;  // copy's random = original's random's copy
        }
        curr = curr.next.next;
    }

    // Step 3: Separate original and copy lists
    Node sentinel = new Node(0);
    Node copyTail = sentinel;
    curr = head;
    while (curr != null) {
        copyTail.next = curr.next;
        copyTail = copyTail.next;
        curr.next = curr.next.next;
        curr = curr.next;
    }

    return sentinel.next;
}
```

### Fully Solved Problem: Copy List with Random Pointer (LC 138)

**Problem:** Deep copy a linked list where each node has a `next` pointer and a `random` pointer (which can point to any node or null).

**Thinking Process:**
1. With HashMap: O(n) space. Map original → copy. Two passes: create copies, then set pointers. Simple.
2. O(1) space trick: interleave copies into the original list. Each copy is right next to its original, so `original.random.next` gives us the copy of the random target.

**Solution:** (Template above — Copy List with Random Pointer)

**Walkthrough with `A(random→C) → B(random→A) → C(random→B)`:**
```
Step 1 - Interleave: A → A' → B → B' → C → C'
Step 2 - Random pointers:
  A'.random = A.random.next = C.next = C'    ✓
  B'.random = B.random.next = A.next = A'    ✓
  C'.random = C.random.next = B.next = B'    ✓
Step 3 - Separate:
  Original: A → B → C
  Copy: A' → B' → C'  (with correct random pointers)
```

### Complexity Analysis
- **Reorder List:** O(n) time, O(1) space
- **Copy Random List (HashMap):** O(n) time, O(n) space
- **Copy Random List (Interleave):** O(n) time, O(1) extra space (excluding the output)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not cutting the link between halves | After reversal, the two halves form a cycle | Always `slow.next = null` after finding middle |
| Interleave: off-by-one when list lengths differ by 1 | Null pointer on the last merge step | Check `while (second != null)` — first half can be 1 longer |
| Copy Random List: forgetting to restore original list | Modified the interviewer's input — they might check | Step 3 restores original as a side effect |
| Partition List: losing the tail | Last node of "less" partition still points into "greater" partition | Always terminate with `tail.next = null` |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you do Copy Random List with HashMap?" | "Two-pass: first pass creates copies in a map; second pass sets next and random. O(n) space." |
| "What about a doubly linked list with random pointers?" | "Same interleave technique works. Just handle `prev` pointers in the separation step." |
| "Reorder List but for arrays?" | "Trivial with indices. Linked list version is harder because we can't access elements by index." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Remove Duplicates from Sorted List | Easy | Skip nodes with same value | 83 |
| 2 | Intersection of Two Linked Lists | Easy | Align lengths, then walk together | 160 |
| 3 | Remove Duplicates from Sorted List II | Medium | Sentinel + skip ALL duplicates (not just extras) | 82 |
| 4 | Partition List | Medium | Two separate lists (< pivot, >= pivot), then connect | 86 |
| 5 | Copy List with Random Pointer | Medium | Solved above | 138 |
| 6 | Reorder List | Medium | Find middle + reverse + interleave | 143 |
| 7 | Odd Even Linked List | Medium | Separate odd-indexed and even-indexed, then connect | 328 |

---

## Cross-References

- **Find Middle** is used in [Merge Sort](./02-linked-lists.md#3-merge-patterns) and [Palindrome check](./02-linked-lists.md#1-fastslow-pointers-floyds)
- **Floyd's Cycle Detection** concept also applies to [Find Duplicate Number](./01-arrays-and-hashing.md#7-dutch-national-flag--cyclic-sort) on arrays
- **Merge K Sorted Lists** uses [Heap](./09-heaps.md#3-merge-k-sorted) for the priority queue approach
- **Sentinel node technique** is universally applicable — use it for all linked list problems
