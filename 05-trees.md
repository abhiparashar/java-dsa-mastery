# 05 — Trees

Trees are the most common interview topic at Meta, Amazon, and Apple. The secret: every tree problem is either DFS (top-down or bottom-up) or BFS. Once you identify which, the template writes itself.

---

## Table of Contents
1. [DFS — Top-Down (Preorder)](#1-dfs--top-down-preorder)
2. [DFS — Bottom-Up (Postorder)](#2-dfs--bottom-up-postorder)
3. [BFS (Level Order)](#3-bfs-level-order)
4. [Binary Search Tree (BST)](#4-binary-search-tree-bst)
5. [Tree Construction](#5-tree-construction)
6. [Morris Traversal](#6-morris-traversal)

---

## 1. DFS — Top-Down (Preorder)

### When to Use (Recognition Signals)
- Information flows from **root to leaves** (pass down accumulated values)
- "**Path sum**" problems — accumulate sum along the path
- "**Same tree**" / "**symmetric tree**" — compare corresponding nodes
- You need to make decisions at each node based on parent information

### When NOT to Use
- You need information FROM children to compute at the parent → use Bottom-Up
- You need level-by-level processing → use BFS
- You need to find diameter/height → Bottom-Up is cleaner

### Optimized Java Template

```java
// Template: Top-Down DFS with accumulator
public void dfs(TreeNode node, int accumulatedValue) {
    if (node == null) return;

    // Process current node with accumulated info from parent
    int newValue = accumulatedValue + node.val;

    if (node.left == null && node.right == null) {
        // Leaf node: finalize result
    }

    dfs(node.left, newValue);
    dfs(node.right, newValue);
}

// Template: Top-Down with path tracking (for path problems)
public void dfsWithPath(TreeNode node, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;

    path.add(node.val);

    if (node.left == null && node.right == null) {
        result.add(new ArrayList<>(path));  // copy — path is shared
    }

    dfsWithPath(node.left, path, result);
    dfsWithPath(node.right, path, result);

    path.remove(path.size() - 1);  // backtrack
}
```

### Fully Solved Problem: Path Sum III (LC 437)

**Problem:** Count paths in a tree that sum to a target. Paths can start and end at any node but must go downward.

**Thinking Process:**
1. Brute force: from each node, try all paths going down → O(n²).
2. Key insight: this is prefix sum on tree paths! At each node, the running sum from root = `currSum`. A path from ancestor to current node has sum = `currSum - ancestorSum`. If `ancestorSum = currSum - target`, then the path sums to target.
3. Use a HashMap of prefix sums, just like [Subarray Sum Equals K](./01-arrays-and-hashing.md#4-prefix-sum).

**Solution:**
```java
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> prefixMap = new HashMap<>();
    prefixMap.put(0L, 1);  // empty path
    return dfs(root, 0L, targetSum, prefixMap);
}

private int dfs(TreeNode node, long currSum, int target, Map<Long, Integer> prefixMap) {
    if (node == null) return 0;

    currSum += node.val;
    int count = prefixMap.getOrDefault(currSum - target, 0);

    prefixMap.merge(currSum, 1, Integer::sum);

    count += dfs(node.left, currSum, target, prefixMap);
    count += dfs(node.right, currSum, target, prefixMap);

    prefixMap.merge(currSum, -1, Integer::sum);  // backtrack: remove current prefix sum

    return count;
}
```

### Complexity Analysis
- **Time:** O(n) — visit each node once, HashMap ops are O(1)
- **Space:** O(n) — HashMap + recursion stack (O(h) for balanced, O(n) for skewed)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not backtracking the prefix map | Prefix sums from one branch leak into another branch | Always undo: `prefixMap.merge(currSum, -1, Integer::sum)` after recursing |
| Using `int` for currSum | Overflow: node values up to 10^9, path length up to 1000 → 10^12 | Use `long` |
| Forgetting `prefixMap.put(0L, 1)` | Misses paths starting from root | Same issue as array prefix sum — always initialize with empty path |
| Confusing "root-to-leaf" with "any-to-any downward" | Root-to-leaf is simpler; any-to-any needs prefix sum technique | Read the problem carefully — LC 112 vs LC 437 |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Maximum Depth of Binary Tree | Easy | Top-down: pass depth down, or bottom-up: return height | 104 |
| 2 | Same Tree | Easy | Compare node-by-node in parallel | 100 |
| 3 | Invert Binary Tree | Easy | Swap children at each node | 226 |
| 4 | Path Sum | Easy | Accumulate sum, check at leaf | 112 |
| 5 | Path Sum II | Medium | Track path, copy at leaf | 113 |
| 6 | Path Sum III | Medium | Prefix sum on tree paths (solved above) | 437 |
| 7 | Sum Root to Leaf Numbers | Medium | Accumulate `num = num * 10 + node.val` | 129 |
| 8 | Flatten Binary Tree to Linked List | Medium | Preorder DFS, reconnect nodes | 114 |

---

## 2. DFS — Bottom-Up (Postorder)

### When to Use (Recognition Signals)
- Information flows from **leaves to root** (compute from children, report to parent)
- "**Height**" / "**depth**" / "**diameter**" of tree
- "**Balanced** binary tree"
- "**Lowest Common Ancestor**"
- You need each node to know something about its subtrees

### When NOT to Use
- You need to pass info FROM root TO leaves → use Top-Down
- You're processing level by level → use BFS

### Optimized Java Template

```java
// Template: Bottom-Up returning a value
public int dfs(TreeNode node) {
    if (node == null) return 0;  // base case

    int left = dfs(node.left);
    int right = dfs(node.right);

    // Combine children's results at current node
    // Update global answer if needed
    return processAndReturn(left, right, node);
}

// Template: Diameter / Longest Path (global variable pattern)
private int maxDiameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return maxDiameter;
}

private int height(TreeNode node) {
    if (node == null) return 0;

    int left = height(node.left);
    int right = height(node.right);

    maxDiameter = Math.max(maxDiameter, left + right);  // update global

    return Math.max(left, right) + 1;  // return height to parent
}
```

**Why the "global variable" pattern:**
- The answer (diameter, max path sum) is a property that spans ACROSS a node (left subtree + node + right subtree).
- But the RETURN value must be a single-direction path (for the parent to use).
- Solution: return the single-direction value, but update a global variable with the across-node value.

### Fully Solved Problem: Binary Tree Maximum Path Sum (LC 124)

**Problem:** Find the maximum path sum in a binary tree. A path can start and end at any node.

**Thinking Process:**
1. At each node, the max path THROUGH this node = `node.val + leftGain + rightGain` (where gains are ≥ 0).
2. But the max path contributed TO the parent = `node.val + max(leftGain, rightGain)` (can only extend one way).
3. Same "global variable" pattern as diameter: update global with through-node sum, return one-direction sum.

**Solution:**
```java
private int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    dfs(root);
    return maxSum;
}

private int dfs(TreeNode node) {
    if (node == null) return 0;

    int leftGain = Math.max(0, dfs(node.left));   // ignore negative paths
    int rightGain = Math.max(0, dfs(node.right));

    // Path through this node (potential global max)
    int pathThroughNode = node.val + leftGain + rightGain;
    maxSum = Math.max(maxSum, pathThroughNode);

    // Return max gain if continuing through parent
    return node.val + Math.max(leftGain, rightGain);
}
```

**Walkthrough with tree `[-10, 9, 20, null, null, 15, 7]`:**
```
     -10
     /  \
    9    20
        /  \
       15    7

dfs(9):  left=0, right=0. through=9. return 9.
dfs(15): left=0, right=0. through=15. return 15.
dfs(7):  left=0, right=0. through=7. return 7.
dfs(20): left=15, right=7. through=20+15+7=42. return 20+15=35.
dfs(-10): left=9, right=35. through=-10+9+35=34. return -10+35=25.

maxSum = 42 (path: 15 → 20 → 7)
```

### Complexity Analysis
- **Time:** O(n) — visit each node exactly once
- **Space:** O(h) — recursion stack (O(log n) balanced, O(n) skewed)

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not clamping negative gains to 0 | Including a negative subtree that reduces the total | `Math.max(0, dfs(node.left))` — we can always choose to NOT extend into a subtree |
| Returning the through-node sum instead of one-direction | Parent receives an invalid "split" path | Return `node.val + Math.max(leftGain, rightGain)`, not `+ leftGain + rightGain` |
| Initializing maxSum to 0 | Fails for all-negative trees (answer should be the least negative node) | Initialize to `Integer.MIN_VALUE` |
| Stack overflow on deep trees | Java default stack ~512KB, ~10K recursion depth | Mention iterative solution with explicit stack, or Morris traversal for O(1) space |

### Interviewer Counter-Questions

| Follow-up Question | Expected Response |
|--------------------|-------------------|
| "Can you do this iteratively?" | "Yes, using postorder with an explicit stack. Push node, process after both children are done. More complex but avoids stack overflow." |
| "What about Longest Univalue Path (LC 687)?" | "Same pattern: bottom-up return length of univalue path. If child's value == current, extend path; otherwise, reset to 0." |
| "Diameter of N-ary tree?" | "Same idea: track the two longest paths among all children. Return only the longest one. Update global with sum of two longest." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Balanced Binary Tree | Easy | Return height, -1 if unbalanced | 110 |
| 2 | Diameter of Binary Tree | Easy | Global max of left+right, return max(left,right)+1 | 543 |
| 3 | Subtree of Another Tree | Easy | Check isSubtree at each node + isSameTree helper | 572 |
| 4 | Lowest Common Ancestor | Medium | Return node if found, bubble up | 236 |
| 5 | Count Good Nodes | Medium | Pass max-so-far top-down (actually top-down, but often grouped here) | 1448 |
| 6 | House Robber III | Medium | Return (rob, notRob) pair from each subtree | 337 |
| 7 | Binary Tree Maximum Path Sum | Hard | Solved above | 124 |

---

## 3. BFS (Level Order)

### When to Use (Recognition Signals)
- "**Level order**" traversal
- "**Right side view**" / "leftmost/rightmost at each level"
- "**Zigzag**" level order
- "**Minimum depth**" (BFS finds it faster than DFS for wide trees)
- "**Connect next right pointers**"
- Any problem asking about levels, layers, or minimum steps in a tree

### When NOT to Use
- Path-based problems (root-to-leaf sums) → DFS is more natural
- Problems requiring backtracking → DFS with backtrack

### Optimized Java Template

```java
// BFS Level Order with level-size tracking
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Deque<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();  // capture size BEFORE processing
        List<Integer> level = new ArrayList<>(size);

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);

            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }

        result.add(level);
    }

    return result;
}
```

**Why `Deque<TreeNode>` instead of `Queue<TreeNode>`:**
- `Queue` is an interface. `LinkedList` implements it but allocates a node object per element.
- `ArrayDeque` is backed by a circular array — much better cache locality and no per-element allocation.

### Fully Solved Problem: Binary Tree Zigzag Level Order Traversal (LC 103)

**Problem:** Level-order traversal, but alternate between left-to-right and right-to-left per level.

**Solution:**
```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Deque<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    boolean leftToRight = true;

    while (!queue.isEmpty()) {
        int size = queue.size();
        LinkedList<Integer> level = new LinkedList<>();

        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();

            if (leftToRight) {
                level.addLast(node.val);
            } else {
                level.addFirst(node.val);  // reverse by adding to front
            }

            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }

        result.add(level);
        leftToRight = !leftToRight;
    }

    return result;
}
```

**Why `LinkedList` for zigzag:**
- `addFirst` is O(1) on `LinkedList` but O(n) on `ArrayList` (shifts all elements).
- We use `LinkedList` only for the level list (small), not for the BFS queue (`ArrayDeque` is still better there).

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Binary Tree Level Order Traversal | Medium | Standard BFS template | 102 |
| 2 | Average of Levels | Easy | Sum each level, divide by size | 637 |
| 3 | Zigzag Level Order | Medium | addFirst/addLast alternation | 103 |
| 4 | Binary Tree Right Side View | Medium | Last element of each level = right side | 199 |
| 5 | Populating Next Right Pointers | Medium | Connect nodes within each level | 116 |
| 6 | All Nodes Distance K | Medium | Build parent map, then BFS from target | 863 |
| 7 | Minimum Depth | Easy | BFS: first leaf found is minimum depth | 111 |

---

## 4. Binary Search Tree (BST)

### When to Use (Recognition Signals)
- Problem explicitly says "BST"
- "**Validate**" a BST
- "**Kth smallest/largest**" in BST
- "**Inorder successor/predecessor**"
- "**Range operations**" in BST

### The Key Property
**Inorder traversal of a BST gives sorted order.** This single fact solves most BST problems.

### Optimized Java Template

```java
// Template A: Validate BST (range-based)
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) &&
           validate(node.right, node.val, max);
}

// Template B: Kth Smallest using iterative inorder
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;

    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }

        curr = stack.pop();
        k--;
        if (k == 0) return curr.val;
        curr = curr.right;
    }

    return -1;  // k > number of nodes
}

// Template C: LCA in BST (O(h) using BST property)
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode curr = root;
    while (curr != null) {
        if (p.val < curr.val && q.val < curr.val) {
            curr = curr.left;
        } else if (p.val > curr.val && q.val > curr.val) {
            curr = curr.right;
        } else {
            return curr;  // split point = LCA
        }
    }
    return null;
}
```

**Why `long` for BST validation bounds:**
- Node values can be `Integer.MIN_VALUE` or `Integer.MAX_VALUE`. If bounds are `int`, we can't represent "less than Integer.MIN_VALUE".
- Using `Long.MIN_VALUE` and `Long.MAX_VALUE` as initial bounds handles all edge cases.

### Fully Solved Problem: Validate Binary Search Tree (LC 98)

**Solution:** Template A above.

**Common mistake — checking only immediate children:**
```java
// WRONG: only checks node vs parent
if (node.left != null && node.left.val >= node.val) return false;

// This misses: [5, 1, 6, null, null, 3, 7] where 3 < 5 (violates BST)
// Node 3 is in right subtree of 5, so it must be > 5, but checking only 3 < 6 passes
```

The range-based approach propagates constraints through the entire tree, catching this.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Search in BST | Easy | Go left if smaller, right if larger | 700 |
| 2 | Range Sum of BST | Easy | Prune branches outside range | 938 |
| 3 | Convert Sorted Array to BST | Easy | Mid element = root, recurse on halves | 108 |
| 4 | Validate BST | Medium | Pass (min, max) range down, use long | 98 |
| 5 | Kth Smallest in BST | Medium | Iterative inorder, stop at kth | 230 |
| 6 | LCA of BST | Medium | Iterative: go toward split point | 235 |
| 7 | Delete Node in BST | Medium | Replace with inorder successor, recurse | 450 |
| 8 | Recover BST | Hard | Inorder finds two swapped elements (first and last out-of-order) | 99 |

---

## 5. Tree Construction

### When to Use (Recognition Signals)
- "Construct tree from **preorder and inorder**"
- "Construct tree from **inorder and postorder**"
- "**Serialize and deserialize**" a tree
- Any problem that builds a tree from a description

### Optimized Java Template

```java
// Construct from Preorder + Inorder
public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> inMap = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) {
        inMap.put(inorder[i], i);  // O(1) index lookup instead of O(n) scan
    }
    return build(preorder, 0, preorder.length - 1,
                 0, inorder.length - 1, inMap);
}

private int preIdx = 0;  // global preorder index

private TreeNode build(int[] preorder, int preStart, int preEnd,
                       int inStart, int inEnd, Map<Integer, Integer> inMap) {
    if (preStart > preEnd || inStart > inEnd) return null;

    int rootVal = preorder[preStart];
    TreeNode root = new TreeNode(rootVal);
    int inRoot = inMap.get(rootVal);
    int leftSize = inRoot - inStart;

    root.left = build(preorder, preStart + 1, preStart + leftSize,
                      inStart, inRoot - 1, inMap);
    root.right = build(preorder, preStart + leftSize + 1, preEnd,
                       inRoot + 1, inEnd, inMap);

    return root;
}
```

**Why HashMap for inorder index lookup:**
- Without it: each recursive call scans inorder array to find root → O(n) per call → O(n²) total.
- With HashMap: O(1) lookup → O(n) total.

### Fully Solved Problem: Serialize and Deserialize Binary Tree (LC 297)

**Solution (Preorder with null markers):**
```java
public class Codec {
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeDFS(root, sb);
        return sb.toString();
    }

    private void serializeDFS(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append("#,");
            return;
        }
        sb.append(node.val).append(",");
        serializeDFS(node.left, sb);
        serializeDFS(node.right, sb);
    }

    public TreeNode deserialize(String data) {
        Deque<String> tokens = new ArrayDeque<>(Arrays.asList(data.split(",")));
        return deserializeDFS(tokens);
    }

    private TreeNode deserializeDFS(Deque<String> tokens) {
        String val = tokens.poll();
        if ("#".equals(val)) return null;

        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = deserializeDFS(tokens);
        node.right = deserializeDFS(tokens);
        return node;
    }
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Build Tree from Preorder and Inorder | Medium | HashMap for O(1) inorder index lookup | 105 |
| 2 | Build Tree from Inorder and Postorder | Medium | Same idea, postorder goes right-to-left | 106 |
| 3 | BST from Preorder | Medium | Use upper-bound to determine subtree boundaries | 1008 |
| 4 | Serialize and Deserialize Binary Tree | Hard | Preorder + null markers | 297 |

---

## 6. Morris Traversal

### When to Use (Recognition Signals)
- "Inorder traversal with **O(1) space**"
- "**Recover BST** without extra space"
- Interview explicitly asks for O(1) space tree traversal

### When NOT to Use
- Acceptable to use O(h) space → standard recursive or iterative with stack is simpler
- You need to modify the tree permanently → Morris temporarily modifies and restores
- The tree is not a binary tree → Morris only works for binary trees

### Optimized Java Template

```java
// Morris Inorder Traversal — O(n) time, O(1) space
public List<Integer> morrisInorder(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    TreeNode curr = root;

    while (curr != null) {
        if (curr.left == null) {
            result.add(curr.val);  // visit
            curr = curr.right;
        } else {
            // Find inorder predecessor (rightmost in left subtree)
            TreeNode pred = curr.left;
            while (pred.right != null && pred.right != curr) {
                pred = pred.right;
            }

            if (pred.right == null) {
                // Create thread: predecessor → current
                pred.right = curr;
                curr = curr.left;
            } else {
                // Thread exists: we've returned via thread. Remove it.
                pred.right = null;
                result.add(curr.val);  // visit
                curr = curr.right;
            }
        }
    }

    return result;
}
```

**How it works:**
- For each node with a left child, we find its **inorder predecessor** (rightmost node in the left subtree).
- We create a temporary "thread" from the predecessor back to the current node.
- When we follow the thread back, we visit the node and remove the thread (restoring the tree).
- Each edge is traversed at most twice (create thread, remove thread) → O(n) total.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Binary Tree Inorder Traversal | Easy | Morris for O(1) space | 94 |
| 2 | Recover BST | Hard | Morris inorder finds two swapped nodes in O(1) space | 99 |
| 3 | Kth Smallest in BST | Medium | Morris inorder, stop at kth element | 230 |

---

## Cross-References

- **Tree DFS** is the foundation for [Graph DFS](./06-graphs.md#2-dfs-connected-components) — graphs generalize trees
- **BFS on trees** uses the same pattern as [Graph BFS](./06-graphs.md#1-bfs-shortest-path-unweighted)
- **BST property** is leveraged in [Binary Search](./04-binary-search.md) — BST IS a binary search structure
- **Path Sum III** combines tree DFS with [Prefix Sum](./01-arrays-and-hashing.md#4-prefix-sum)
- **Bottom-up tree DP** extends to general [Tree DP](./07-dynamic-programming.md) problems like House Robber III
