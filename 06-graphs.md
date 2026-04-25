# 06 — Graphs

Graphs are Google's favorite topic. Every graph problem reduces to: (1) how to represent the graph, (2) how to traverse it, (3) what to track during traversal. Master the 10 patterns below and you can solve any graph problem.

---

## Table of Contents
1. [BFS (Shortest Path Unweighted)](#1-bfs-shortest-path-unweighted)
2. [DFS (Connected Components)](#2-dfs-connected-components)
3. [Topological Sort](#3-topological-sort)
4. [Union Find (Disjoint Set)](#4-union-find-disjoint-set)
5. [Dijkstra (Weighted Shortest Path)](#5-dijkstra-weighted-shortest-path)
6. [Bellman-Ford](#6-bellman-ford)
7. [Floyd-Warshall](#7-floyd-warshall)
8. [Tarjan's (Bridges & Articulation Points)](#8-tarjans-bridges--articulation-points)
9. [Bipartite Check](#9-bipartite-check)
10. [Multi-source BFS](#10-multi-source-bfs)

---

## 1. BFS (Shortest Path Unweighted)

### When to Use
- "**Shortest path**" in unweighted graph or grid
- "**Minimum steps/moves**"
- "**Word ladder**" (each word is a node, edges connect words differing by one letter)

### When NOT to Use
- Graph has weighted edges → Dijkstra or Bellman-Ford
- You need ALL paths (not just shortest) → DFS or backtracking
- Graph is a tree and you need depth → DFS is simpler

### Optimized Java Template

```java
// BFS on grid
public int bfsGrid(int[][] grid, int[] start, int[] end) {
    int m = grid.length, n = grid[0].length;
    boolean[][] visited = new boolean[m][n];
    Deque<int[]> queue = new ArrayDeque<>();
    int[][] dirs = {{0,1},{1,0},{0,-1},{-1,0}};

    queue.offer(start);
    visited[start[0]][start[1]] = true;
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int[] curr = queue.poll();
            if (curr[0] == end[0] && curr[1] == end[1]) return steps;

            for (int[] d : dirs) {
                int nr = curr[0] + d[0], nc = curr[1] + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n
                    && !visited[nr][nc] && grid[nr][nc] == 0) {
                    visited[nr][nc] = true;  // mark BEFORE enqueueing
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
        steps++;
    }
    return -1;
}

// BFS on adjacency list
public int bfsGraph(List<List<Integer>> graph, int start, int end) {
    boolean[] visited = new boolean[graph.size()];
    Deque<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    visited[start] = true;
    int steps = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            int node = queue.poll();
            if (node == end) return steps;
            for (int neighbor : graph.get(node)) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    queue.offer(neighbor);
                }
            }
        }
        steps++;
    }
    return -1;
}
```

**Critical: mark visited BEFORE enqueueing, not after dequeueing.**
- If you mark after dequeueing, the same node can be enqueued multiple times by different neighbors → O(V²) instead of O(V+E).

### Fully Solved Problem: Word Ladder (LC 127)

**Problem:** Transform `beginWord` to `endWord`, changing one letter at a time. Each intermediate word must be in the word list. Find the shortest transformation length.

**Solution:**
```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;

    Deque<String> queue = new ArrayDeque<>();
    queue.offer(beginWord);
    Set<String> visited = new HashSet<>();
    visited.add(beginWord);
    int steps = 1;

    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            String word = queue.poll();
            if (word.equals(endWord)) return steps;

            char[] chars = word.toCharArray();
            for (int j = 0; j < chars.length; j++) {
                char original = chars[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == original) continue;
                    chars[j] = c;
                    String next = new String(chars);
                    if (wordSet.contains(next) && !visited.contains(next)) {
                        visited.add(next);
                        queue.offer(next);
                    }
                }
                chars[j] = original;
            }
        }
        steps++;
    }
    return 0;
}
```

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Marking visited after dequeue | Same node enqueued many times, TLE | Mark visited when you ADD to queue |
| Not tracking level (step count) | Can't report shortest distance | Use `size = queue.size()` to process one level at a time |
| Building adjacency list for word ladder | O(n² * m) for comparing all pairs | Instead, try all 26 mutations per position: O(26 * m * n) |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Flood Fill | Easy | BFS/DFS from start, spread to same-color neighbors | 733 |
| 2 | Rotting Oranges | Medium | Multi-source BFS (see [Pattern 10](#10-multi-source-bfs)) | 994 |
| 3 | Word Ladder | Hard | Each word = node, BFS for shortest path | 127 |
| 4 | Shortest Path in Binary Matrix | Medium | 8-directional BFS | 1091 |
| 5 | Open the Lock | Medium | BFS on state space (4-digit combinations) | 752 |

---

## 2. DFS (Connected Components)

### When to Use
- "**Number of islands**" / "connected components"
- "**Max area** of island"
- "**Surrounded regions**"
- "**Detect cycle**" in directed or undirected graph
- Exploring all reachable nodes from a source

### Optimized Java Template

```java
// DFS on grid (Number of Islands)
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                dfs(grid, i, j);
                count++;
            }
        }
    }
    return count;
}

private void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length
        || grid[r][c] != '1') return;

    grid[r][c] = '0';  // mark visited by modifying input (O(1) space)
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}

// DFS Cycle Detection in DIRECTED graph (using 3 colors)
public boolean hasCycleDirected(List<List<Integer>> graph, int n) {
    int[] color = new int[n];  // 0=white(unvisited), 1=gray(in-progress), 2=black(done)

    for (int i = 0; i < n; i++) {
        if (color[i] == 0 && dfsCycle(graph, i, color)) return true;
    }
    return false;
}

private boolean dfsCycle(List<List<Integer>> graph, int node, int[] color) {
    color[node] = 1;  // gray: currently being explored

    for (int neighbor : graph.get(node)) {
        if (color[neighbor] == 1) return true;   // back edge → cycle
        if (color[neighbor] == 0 && dfsCycle(graph, neighbor, color)) return true;
    }

    color[node] = 2;  // black: fully explored
    return false;
}
```

**Directed vs Undirected cycle detection:**
- **Directed**: 3-color DFS. A back edge (gray → gray) means cycle.
- **Undirected**: track parent. If neighbor is visited AND not parent → cycle. OR use Union Find.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Number of Islands | Medium | DFS/BFS, mark visited | 200 |
| 2 | Max Area of Island | Medium | DFS returns area count | 695 |
| 3 | Clone Graph | Medium | DFS + HashMap for old→new mapping | 133 |
| 4 | Number of Provinces | Medium | DFS or Union Find on adjacency matrix | 547 |
| 5 | Surrounded Regions | Medium | DFS from border O's (DON'T flip those), flip the rest | 130 |
| 6 | Pacific Atlantic Water Flow | Medium | DFS from each ocean, find intersection | 417 |
| 7 | Making A Large Island | Hard | Label islands + sizes, check each 0's neighbors | 827 |

---

## 3. Topological Sort

### When to Use
- "**Prerequisites**" / "dependencies" / "task ordering"
- "**Course schedule**"
- DAG (Directed Acyclic Graph) + ordering
- "Is there a valid ordering?" / "find any valid ordering"

### Optimized Java Template

```java
// Kahn's Algorithm (BFS-based) — preferred in interviews
public int[] topologicalSort(int n, int[][] edges) {
    List<List<Integer>> graph = new ArrayList<>();
    int[] indegree = new int[n];

    for (int i = 0; i < n; i++) graph.add(new ArrayList<>());

    for (int[] e : edges) {
        graph.get(e[0]).add(e[1]);
        indegree[e[1]]++;
    }

    Deque<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) queue.offer(i);
    }

    int[] order = new int[n];
    int idx = 0;

    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;

        for (int neighbor : graph.get(node)) {
            if (--indegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }

    return idx == n ? order : new int[0];  // empty = cycle exists
}
```

**Why Kahn's over DFS-based:**
- Kahn's naturally detects cycles (if `idx < n`, there's a cycle).
- Easier to implement correctly.
- DFS-based needs reverse postorder and separate cycle detection.

### Fully Solved Problem: Alien Dictionary (LC 269)

**Problem:** Given a sorted list of words in an alien language, determine the order of characters.

**Solution:**
```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> graph = new HashMap<>();
    Map<Character, Integer> indegree = new HashMap<>();

    // Initialize all characters
    for (String w : words) {
        for (char c : w.toCharArray()) {
            graph.putIfAbsent(c, new HashSet<>());
            indegree.putIfAbsent(c, 0);
        }
    }

    // Build graph from adjacent word pairs
    for (int i = 0; i < words.length - 1; i++) {
        String w1 = words[i], w2 = words[i + 1];
        int minLen = Math.min(w1.length(), w2.length());

        // Edge case: "abc" before "ab" is invalid
        if (w1.length() > w2.length() && w1.startsWith(w2)) return "";

        for (int j = 0; j < minLen; j++) {
            if (w1.charAt(j) != w2.charAt(j)) {
                if (graph.get(w1.charAt(j)).add(w2.charAt(j))) {
                    indegree.merge(w2.charAt(j), 1, Integer::sum);
                }
                break;  // only the FIRST difference matters
            }
        }
    }

    // Topological sort
    Deque<Character> queue = new ArrayDeque<>();
    for (var entry : indegree.entrySet()) {
        if (entry.getValue() == 0) queue.offer(entry.getKey());
    }

    StringBuilder sb = new StringBuilder();
    while (!queue.isEmpty()) {
        char c = queue.poll();
        sb.append(c);
        for (char neighbor : graph.get(c)) {
            indegree.merge(neighbor, -1, Integer::sum);
            if (indegree.get(neighbor) == 0) queue.offer(neighbor);
        }
    }

    return sb.length() == indegree.size() ? sb.toString() : "";
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Course Schedule | Medium | Can we complete all courses? (cycle detection) | 207 |
| 2 | Course Schedule II | Medium | Return a valid ordering | 210 |
| 3 | Minimum Height Trees | Medium | Peel leaves layer by layer (reverse topo) | 310 |
| 4 | Alien Dictionary | Hard | Build graph from word comparisons | 269 |

---

## 4. Union Find (Disjoint Set)

### When to Use
- "**Dynamic connectivity**" — elements connected/disconnected over time
- "**Redundant connection**" — find the edge that creates a cycle
- "**Accounts merge**" — group elements by equivalence
- Alternative to DFS for connected components, especially when edges arrive incrementally

### Optimized Java Template

```java
class UnionFind {
    private int[] parent;
    private int[] rank;
    private int components;

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);  // path compression
        }
        return parent[x];
    }

    public boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;  // already connected

        // Union by rank
        if (rank[rx] < rank[ry]) { int t = rx; rx = ry; ry = t; }
        parent[ry] = rx;
        if (rank[rx] == rank[ry]) rank[rx]++;
        components--;
        return true;
    }

    public boolean connected(int x, int y) { return find(x) == find(y); }
    public int getComponents() { return components; }
}
```

**Path compression**: after `find(x)`, every node on the path from `x` to root now points directly to root. This flattens the tree, making future `find` nearly O(1).

**Union by rank**: always attach the shorter tree under the taller tree's root. This keeps tree height logarithmic.

**Combined**: amortized O(α(n)) per operation, where α is the inverse Ackermann function — effectively O(1) for all practical inputs.

### Fully Solved Problem: Accounts Merge (LC 721)

**Problem:** Given accounts `[name, email1, email2, ...]`, merge accounts that share any email. Different accounts can have the same name.

**Solution:**
```java
public List<List<String>> accountsMerge(List<List<String>> accounts) {
    UnionFind uf = new UnionFind(accounts.size());
    Map<String, Integer> emailToAccount = new HashMap<>();

    // Union accounts that share an email
    for (int i = 0; i < accounts.size(); i++) {
        for (int j = 1; j < accounts.get(i).size(); j++) {
            String email = accounts.get(i).get(j);
            if (emailToAccount.containsKey(email)) {
                uf.union(i, emailToAccount.get(email));
            } else {
                emailToAccount.put(email, i);
            }
        }
    }

    // Group emails by root account
    Map<Integer, TreeSet<String>> groups = new HashMap<>();
    for (var entry : emailToAccount.entrySet()) {
        int root = uf.find(entry.getValue());
        groups.computeIfAbsent(root, k -> new TreeSet<>()).add(entry.getKey());
    }

    // Build result
    List<List<String>> result = new ArrayList<>();
    for (var entry : groups.entrySet()) {
        List<String> merged = new ArrayList<>();
        merged.add(accounts.get(entry.getKey()).get(0));  // name
        merged.addAll(entry.getValue());
        result.add(merged);
    }
    return result;
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Number of Connected Components | Medium | Union all edges, return component count | 323 |
| 2 | Redundant Connection | Medium | Union edges one by one; first that returns false = redundant | 684 |
| 3 | Accounts Merge | Medium | Solved above | 721 |
| 4 | Most Stones Removed | Medium | Union stones sharing row or column | 947 |
| 5 | Number of Islands II | Hard | Start with all water, add lands with union | 305 |

---

## 5. Dijkstra (Weighted Shortest Path)

### When to Use
- Weighted graph with **non-negative** edge weights
- "**Shortest path**" / "minimum cost" in weighted graph
- "**Network delay time**"

### When NOT to Use
- Negative edge weights → use Bellman-Ford
- Unweighted graph → BFS is simpler and faster
- All-pairs shortest path → Floyd-Warshall

### Optimized Java Template

```java
public int[] dijkstra(int n, List<List<int[]>> graph, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // PQ stores {node, distance} — use int[] to avoid object creation
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    pq.offer(new int[]{src, 0});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int node = curr[0], d = curr[1];

        if (d > dist[node]) continue;  // lazy deletion: stale entry

        for (int[] edge : graph.get(node)) {
            int next = edge[0], weight = edge[1];
            int newDist = d + weight;

            if (newDist < dist[next]) {
                dist[next] = newDist;
                pq.offer(new int[]{next, newDist});
            }
        }
    }

    return dist;
}
```

**Why `int[]` instead of a custom Node class:**
- Each `new Node(...)` creates an object on the heap → GC pressure.
- `int[]` is a single allocation for both values. For large graphs, this significantly reduces garbage collection pauses.

**Why lazy deletion instead of decrease-key:**
- Java's `PriorityQueue` doesn't support `decreaseKey`. We'd need to remove and re-add, which is O(n).
- Instead, we add a new entry and skip stale entries: `if (d > dist[node]) continue`. This is O(1) and works correctly.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Network Delay Time | Medium | Standard Dijkstra, return max of all distances | 743 |
| 2 | Path with Maximum Probability | Medium | Max-heap, multiply probabilities | 1514 |
| 3 | Path With Minimum Effort | Medium | Binary search on effort OR modified Dijkstra | 1631 |
| 4 | Cheapest Flights Within K Stops | Medium | Modified Dijkstra/BFS with stop count, or Bellman-Ford | 787 |
| 5 | Swim in Rising Water | Hard | Binary search + BFS, or modified Dijkstra on max elevation | 778 |

---

## 6. Bellman-Ford

### When to Use
- Graph has **negative edge weights**
- Need to detect **negative cycles**
- "Cheapest flights within **K stops**" (limited relaxation rounds)

### Optimized Java Template

```java
public int[] bellmanFord(int n, int[][] edges, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // Relax all edges (n-1) times
    for (int i = 0; i < n - 1; i++) {
        boolean updated = false;
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                updated = true;
            }
        }
        if (!updated) break;  // early termination
    }

    // Check for negative cycles (optional)
    for (int[] e : edges) {
        if (dist[e[0]] != Integer.MAX_VALUE && dist[e[0]] + e[2] < dist[e[1]]) {
            return null;  // negative cycle detected
        }
    }

    return dist;
}
```

**Cheapest Flights Within K Stops (LC 787):**
```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    for (int i = 0; i <= k; i++) {
        int[] temp = dist.clone();  // use previous round's distances
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < temp[v]) {
                temp[v] = dist[u] + w;
            }
        }
        dist = temp;
    }

    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

**Why clone dist[] in the K-stops variant:**
- Without cloning, updates within the same round cascade: node A relaxes node B, which then relaxes node C — all in one round. This is MORE than K stops.
- Cloning ensures each round only uses distances from the PREVIOUS round.

---

## 7. Floyd-Warshall

### When to Use
- "**All-pairs** shortest path"
- Small graph (V ≤ 400, since O(V³))
- "Find the city with the smallest number of reachable neighbors within distance threshold"

### Optimized Java Template

```java
public int[][] floydWarshall(int n, int[][] edges) {
    int[][] dist = new int[n][n];
    int INF = (int) 1e9;

    for (int[] row : dist) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) dist[i][i] = 0;
    for (int[] e : edges) dist[e[0]][e[1]] = e[2];

    // k MUST be the outermost loop
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }

    return dist;
}
```

**Why k must be outermost:** `dist[i][j]` after round `k` means "shortest path from `i` to `j` using only intermediate nodes `{0, 1, ..., k}`." If `k` isn't outermost, this invariant breaks and results are incorrect.

---

## 8. Tarjan's (Bridges & Articulation Points)

### When to Use
- "**Critical connections**" / "bridges" in a network
- "**Articulation points**" — nodes whose removal disconnects the graph
- Strongly connected components in directed graphs

### Optimized Java Template

```java
// Find all bridges in an undirected graph
private int timer = 0;

public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    List<List<Integer>> graph = new ArrayList<>();
    for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
    for (var conn : connections) {
        graph.get(conn.get(0)).add(conn.get(1));
        graph.get(conn.get(1)).add(conn.get(0));
    }

    int[] disc = new int[n];  // discovery time
    int[] low = new int[n];   // lowest discovery time reachable
    Arrays.fill(disc, -1);
    List<List<Integer>> bridges = new ArrayList<>();

    dfs(0, -1, graph, disc, low, bridges);
    return bridges;
}

private void dfs(int node, int parent, List<List<Integer>> graph,
                 int[] disc, int[] low, List<List<Integer>> bridges) {
    disc[node] = low[node] = timer++;

    for (int neighbor : graph.get(node)) {
        if (neighbor == parent) continue;

        if (disc[neighbor] == -1) {
            dfs(neighbor, node, graph, disc, low, bridges);
            low[node] = Math.min(low[node], low[neighbor]);

            if (low[neighbor] > disc[node]) {
                bridges.add(Arrays.asList(node, neighbor));
            }
        } else {
            low[node] = Math.min(low[node], disc[neighbor]);
        }
    }
}
```

**Bridge condition:** edge `(u, v)` is a bridge if `low[v] > disc[u]` — meaning `v` can't reach `u` or any ancestor of `u` without using the edge `(u, v)`.

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Critical Connections in a Network | Hard | Tarjan's bridge-finding algorithm | 1192 |

---

## 9. Bipartite Check

### When to Use
- "Is graph **bipartite**?"
- "**Two-color** the graph"
- "**Possible bipartition**" / "divide into two groups"

### Optimized Java Template

```java
public boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];  // 0=uncolored, 1=group A, -1=group B

    for (int i = 0; i < n; i++) {
        if (color[i] == 0 && !bfs(graph, i, color)) return false;
    }
    return true;
}

private boolean bfs(int[][] graph, int start, int[] color) {
    Deque<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    color[start] = 1;

    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int neighbor : graph[node]) {
            if (color[neighbor] == 0) {
                color[neighbor] = -color[node];
                queue.offer(neighbor);
            } else if (color[neighbor] == color[node]) {
                return false;  // same color on both ends = not bipartite
            }
        }
    }
    return true;
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Is Graph Bipartite? | Medium | BFS/DFS with 2-coloring | 785 |
| 2 | Possible Bipartition | Medium | Same as bipartite on dislike graph | 886 |

---

## 10. Multi-source BFS

### When to Use
- "**Rotting oranges**" — spread from ALL sources simultaneously
- "**01 Matrix**" — distance from nearest 0
- "**Walls and Gates**" — distance from nearest gate
- Any problem where you BFS from MULTIPLE starting points at once

### Why It's Different
- Single-source BFS: one start, explore outward.
- Multi-source BFS: add ALL sources to the queue initially, then BFS. This computes the shortest distance FROM any source — equivalent to adding a virtual super-source connected to all real sources.

### Optimized Java Template

```java
// Multi-source BFS on grid
public int[][] multiSourceBFS(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dist = new int[m][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);

    Deque<int[]> queue = new ArrayDeque<>();
    int[][] dirs = {{0,1},{1,0},{0,-1},{-1,0}};

    // Enqueue ALL sources
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 0) {  // source cells
                dist[i][j] = 0;
                queue.offer(new int[]{i, j});
            }
        }
    }

    // BFS from all sources simultaneously
    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        for (int[] d : dirs) {
            int nr = curr[0] + d[0], nc = curr[1] + d[1];
            if (nr >= 0 && nr < m && nc >= 0 && nc < n
                && dist[nr][nc] > dist[curr[0]][curr[1]] + 1) {
                dist[nr][nc] = dist[curr[0]][curr[1]] + 1;
                queue.offer(new int[]{nr, nc});
            }
        }
    }

    return dist;
}
```

### Fully Solved Problem: Rotting Oranges (LC 994)

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Deque<int[]> queue = new ArrayDeque<>();
    int fresh = 0;

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) queue.offer(new int[]{i, j});
            else if (grid[i][j] == 1) fresh++;
        }
    }

    if (fresh == 0) return 0;

    int[][] dirs = {{0,1},{1,0},{0,-1},{-1,0}};
    int minutes = 0;

    while (!queue.isEmpty()) {
        int size = queue.size();
        boolean rotted = false;

        for (int i = 0; i < size; i++) {
            int[] curr = queue.poll();
            for (int[] d : dirs) {
                int nr = curr[0] + d[0], nc = curr[1] + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    rotted = true;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }

        if (rotted) minutes++;
    }

    return fresh == 0 ? minutes : -1;
}
```

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | 01 Matrix | Medium | Multi-source BFS from all 0s | 542 |
| 2 | Rotting Oranges | Medium | Multi-source BFS from all rotten oranges | 994 |
| 3 | As Far from Land as Possible | Medium | Multi-source BFS from all land cells | 1162 |
| 4 | Shortest Bridge | Medium | DFS to find first island, BFS to reach second | 934 |

---

## Cross-References

- **BFS on graphs** generalizes [BFS on trees](./05-trees.md#3-bfs-level-order) — trees are just acyclic connected graphs
- **Topological Sort** is foundational for [DP on DAGs](./07-dynamic-programming.md) — process nodes in topo order
- **Union Find** is an alternative to DFS for [connected components](./06-graphs.md#2-dfs-connected-components) — preferred when edges arrive incrementally
- **Dijkstra** uses the same [Heap](./09-heaps.md) pattern as Top-K problems
- **Multi-source BFS** on a grid is used in [interval/scheduling](./10-intervals.md) analogies for 2D problems
