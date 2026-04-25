# 12 — Tries

A Trie (prefix tree) stores strings character-by-character in a tree structure where shared prefixes share nodes. Two variants matter for interviews: the standard prefix trie and the XOR trie (for bitwise problems). Tries eliminate the O(L) hash computation on every lookup that HashMaps have.

---

## Table of Contents
1. [Prefix Trie](#1-prefix-trie)
2. [XOR Trie](#2-xor-trie)

---

## 1. Prefix Trie

### When to Use
- "**Prefix matching**" / "**autocomplete**"
- "**Word search II**" — find multiple words on a board
- "**Longest common prefix**"
- "**Word dictionary** with wildcards (`.` matches any)"
- Any time you need to search/filter by prefix across many strings

### When NOT to Use
- Exact match only with no prefix queries — HashMap is simpler and faster
- Single string pattern matching — use KMP or Rabin-Karp
- Very long strings with no shared prefixes — Trie wastes space

### Optimized Java Template

```java
// Array-based Trie (26x faster than HashMap-based for lowercase letters)
class Trie {
    private TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = searchPrefix(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }

    private TrieNode searchPrefix(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}

class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd;
}
```

**Why `TrieNode[26]` instead of `HashMap<Character, TrieNode>`?**
- Array: O(1) index access, no autoboxing, no hash computation, cache-friendly
- HashMap: autoboxes `char` → `Character`, computes hash, resolves collisions
- For lowercase-only: array is 5-10x faster in practice

### Fully Solved Problem: Word Search II (LC 212)

**Problem:** Given a 2D board and a list of words, find all words that exist on the board. Each cell can be used once per word. Adjacent cells are horizontally/vertically neighboring.

**Thinking process:**
1. Brute force: for each word, run Word Search (LC 79) → O(words * m * n * 4^L), TLE
2. Optimization: build a Trie from all words, then DFS once on the board
3. At each cell, follow the Trie — if the current path isn't a prefix in the Trie, prune immediately
4. When we hit `isEnd`, we found a word — remove it from the Trie to avoid duplicates

```java
public List<String> findWords(char[][] board, String[] words) {
    TrieNode root = buildTrie(words);
    List<String> result = new ArrayList<>();

    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            dfs(board, i, j, root, result);
        }
    }

    return result;
}

private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return;

    char ch = board[r][c];
    if (ch == '#' || node.children[ch - 'a'] == null) return;

    node = node.children[ch - 'a'];

    if (node.word != null) {
        result.add(node.word);
        node.word = null;  // de-duplicate: remove word after finding
    }

    board[r][c] = '#';  // mark visited
    dfs(board, r + 1, c, node, result);
    dfs(board, r - 1, c, node, result);
    dfs(board, r, c + 1, node, result);
    dfs(board, r, c - 1, node, result);
    board[r][c] = ch;   // unmark

    // Optimization: prune empty branches
    if (isEmpty(node)) {
        // Could remove this child from parent, but simpler to just let it be
    }
}

private boolean isEmpty(TrieNode node) {
    for (TrieNode child : node.children) {
        if (child != null) return false;
    }
    return node.word == null;
}

private TrieNode buildTrie(String[] words) {
    TrieNode root = new TrieNode();
    for (String word : words) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.word = word;  // store the full word at the end node
    }
    return root;
}

// Modified TrieNode for Word Search II
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    String word;  // null unless this node ends a word
}
```

**Key optimizations:**
1. **Store word at end node** instead of boolean — avoids rebuilding the string during DFS
2. **Set `word = null` after finding** — prevents duplicates without a separate Set
3. **Prune empty branches** — as words are found and removed, dead branches can be pruned

**Complexity:** O(m * n * 4^L) worst case, but Trie pruning makes it much faster in practice. L = max word length.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| HashMap-based Trie for lowercase-only | 5-10x slower than array | Use `TrieNode[26]` for fixed alphabets |
| Not handling duplicates in Word Search II | Same word added to result multiple times | Set `node.word = null` after finding |
| Word Search II: separate DFS per word | O(words * m * n * 4^L), TLE | Single Trie, single DFS pass |
| Forgetting to restore board cell | Later DFS paths skip valid cells | `board[r][c] = ch` after all 4 DFS calls |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Space complexity of a Trie?" | "O(total characters * alphabet size). For N words of average length L with 26 chars: O(N * L * 26). Each node has a 26-pointer array." |
| "Trie vs HashMap for prefix queries?" | "Trie: O(L) per query, supports prefix search natively. HashMap: O(L) per exact lookup, prefix search requires checking all keys." |
| "How to support `.` wildcard?" | "On `.`, recurse into ALL non-null children. This is Add and Search Word (LC 211)." |
| "How to add delete to a Trie?" | "Track count of words passing through each node. On delete, decrement counts and remove nodes when count reaches 0." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Implement Trie | Medium | Array-based nodes, `isEnd` flag | 208 |
| 2 | Add and Search Word | Medium | DFS with wildcard `.` matching | 211 |
| 3 | Word Search II | Hard | Trie + board DFS, prune empty branches | 212 |
| 4 | Replace Words | Medium | Find shortest root/prefix in Trie | 648 |
| 5 | Design Search Autocomplete | Hard | Trie + priority queue for top-3 | 642 |
| 6 | Longest Word in Dictionary | Medium | BFS/DFS on Trie, check all prefixes exist | 720 |

---

## 2. XOR Trie

### When to Use
- "**Maximum XOR** of two numbers"
- "**XOR queries** on subarrays"
- Any problem requiring bitwise prefix matching
- Finding the number that maximizes XOR with a given value

### When NOT to Use
- Non-XOR bitwise problems (AND, OR) — different techniques needed
- Single pair XOR — just XOR directly, no Trie needed

### Optimized Java Template

```java
// XOR Trie: stores numbers bit-by-bit from MSB to LSB
class XORTrie {
    private int[][] children;  // children[node][0/1] = next node index
    private int nodeCount;

    public XORTrie(int maxNumbers) {
        int maxNodes = maxNumbers * 31 + 1;  // 31 bits per number
        children = new int[maxNodes][2];
        nodeCount = 1;  // root is node 0
        children[0][0] = children[0][1] = -1;
    }

    public void insert(int num) {
        int node = 0;
        for (int bit = 30; bit >= 0; bit--) {  // 31 bits (0 to 2^31-1)
            int b = (num >> bit) & 1;
            if (children[node][b] == -1) {
                children[nodeCount][0] = children[nodeCount][1] = -1;
                children[node][b] = nodeCount++;
            }
            node = children[node][b];
        }
    }

    // Find the number in the Trie that gives maximum XOR with num
    public int maxXor(int num) {
        int node = 0, result = 0;
        for (int bit = 30; bit >= 0; bit--) {
            int b = (num >> bit) & 1;
            int want = 1 - b;  // we want the opposite bit to maximize XOR

            if (children[node][want] != -1) {
                result |= (1 << bit);  // this bit contributes to XOR
                node = children[node][want];
            } else {
                node = children[node][b];  // forced to take same bit
            }
        }
        return result;
    }
}
```

**Why process from MSB?** XOR is maximized when higher-order bits differ. By going MSB-first, we greedily pick the opposite bit at each level, maximizing the result from the most significant position.

### Fully Solved Problem: Maximum XOR of Two Numbers in an Array (LC 421)

**Problem:** Given an integer array, find the maximum XOR of any two numbers.

**Thinking process:**
1. Brute force: try all pairs → O(n²)
2. For each number, we want to find another number that maximizes XOR
3. Build a Trie of all numbers, then for each number, greedily traverse the Trie choosing opposite bits
4. O(31n) = O(n)

```java
public int findMaximumXOR(int[] nums) {
    // Build Trie
    int[][] children = new int[nums.length * 31 + 1][2];
    int nodeCount = 1;
    children[0][0] = children[0][1] = -1;

    for (int num : nums) {
        int node = 0;
        for (int bit = 30; bit >= 0; bit--) {
            int b = (num >> bit) & 1;
            if (children[node][b] == -1) {
                children[nodeCount][0] = children[nodeCount][1] = -1;
                children[node][b] = nodeCount++;
            }
            node = children[node][b];
        }
    }

    // Query each number for max XOR
    int maxXor = 0;
    for (int num : nums) {
        int node = 0, xor = 0;
        for (int bit = 30; bit >= 0; bit--) {
            int b = (num >> bit) & 1;
            int want = 1 - b;
            if (children[node][want] != -1) {
                xor |= (1 << bit);
                node = children[node][want];
            } else {
                node = children[node][b];
            }
        }
        maxXor = Math.max(maxXor, xor);
    }

    return maxXor;
}
```

**Alternative: HashSet approach (also O(n) per bit but simpler):**
```java
public int findMaximumXOR(int[] nums) {
    int max = 0, mask = 0;
    for (int bit = 30; bit >= 0; bit--) {
        mask |= (1 << bit);
        Set<Integer> prefixes = new HashSet<>();
        for (int num : nums) prefixes.add(num & mask);

        int candidate = max | (1 << bit);  // try setting this bit
        for (int prefix : prefixes) {
            // If prefix ^ candidate exists, then two numbers XOR to candidate
            if (prefixes.contains(prefix ^ candidate)) {
                max = candidate;
                break;
            }
        }
    }
    return max;
}
```

**Complexity:** O(31n) = O(n) for both approaches. Trie uses more memory but is more flexible for follow-ups.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Using 32 bits for non-negative numbers | Wastes a level on sign bit | Use 30 bits (0 to 2^31-1) or 31 if needed |
| Array too small for Trie nodes | `ArrayIndexOutOfBoundsException` | Max nodes = n * 31 + 1 |
| Not initializing children to -1 | 0 is a valid node index, causes confusion | Initialize all children to -1 |
| Forgetting about negative numbers | Sign bit flips XOR behavior | Handle sign bit separately or use 32 bits |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Time complexity?" | "O(31n) = O(n). Each number: 31 bit operations for insert, 31 for query." |
| "Can you solve it without a Trie?" | "Yes — bit-by-bit with HashSet. For each bit from MSB, try to set it in the result using the XOR property: if a^b = candidate, then a = b^candidate." |
| "What about max XOR of a subarray?" | "Prefix XOR array + XOR Trie. For each prefix, query the Trie for max XOR." |
| "Maximum XOR with a range constraint?" | "Persistent Trie or offline processing with sorting." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Maximum XOR of Two Numbers in an Array | Medium | XOR Trie, greedily pick opposite bits | 421 |
| 2 | Maximum XOR With an Element From Array | Hard | Sort queries + Trie, process offline | 1707 |

---

## Trie Complexity Summary

| Operation | Time | Space |
|-----------|------|-------|
| Insert word of length L | O(L) | O(L * alphabet_size) worst case |
| Search word of length L | O(L) | O(1) |
| Prefix search | O(L) | O(1) |
| Total space for N words, avg length L | — | O(N * L * alphabet_size) worst, much less with shared prefixes |

---

## Cross-References

- **Word Search II** combines Trie with grid DFS — see [Backtracking](./08-backtracking.md#4-gridboard-exploration)
- **XOR properties** connect to [Bit Manipulation](./13-bit-manipulation.md#1-xor-properties)
- **Autocomplete** can use Trie + Heap — see [Heaps](./09-heaps.md#1-top-k-pattern)
