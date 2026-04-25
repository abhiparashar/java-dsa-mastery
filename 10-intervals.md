# 10 — Intervals

Interval problems reduce to sorting by start (or end) and then sweeping. Three patterns cover every interval problem: merge/insert, scheduling (greedy by end time), and line sweep (event-based). The key is converting the problem into an event timeline.

---

## Table of Contents
1. [Merge / Insert Intervals](#1-merge--insert-intervals)
2. [Scheduling (Greedy by End Time)](#2-scheduling-greedy-by-end-time)
3. [Line Sweep (Event-Based)](#3-line-sweep-event-based)

---

## 1. Merge / Insert Intervals

### When to Use
- "**Merge overlapping** intervals"
- "**Insert** a new interval and merge"
- "**Remove covered** intervals"
- Any time intervals need to be combined or simplified

### When NOT to Use
- Counting maximum overlaps at a point — use Line Sweep
- Selecting non-overlapping intervals — use Scheduling

### Optimized Java Template

```java
// Merge overlapping intervals
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

    List<int[]> merged = new ArrayList<>();
    for (int[] curr : intervals) {
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < curr[0]) {
            merged.add(curr);  // no overlap
        } else {
            merged.get(merged.size() - 1)[1] =
                Math.max(merged.get(merged.size() - 1)[1], curr[1]);  // extend
        }
    }
    return merged.toArray(new int[merged.size()][]);
}

// Insert interval (without sorting — input already sorted)
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Add all intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i++]);
    }

    // Merge all overlapping intervals with newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);

    // Add remaining intervals
    while (i < n) {
        result.add(intervals[i++]);
    }

    return result.toArray(new int[result.size()][]);
}
```

### Fully Solved Problem: Merge Intervals (LC 56)

**Problem:** Given an array of intervals, merge all overlapping intervals.

**Thinking process:**
1. If intervals are sorted by start, overlapping intervals are adjacent
2. Two intervals `[a, b]` and `[c, d]` overlap iff `b >= c` (when sorted by start)
3. When they overlap, merge into `[a, max(b, d)]`
4. When they don't overlap, start a new merged interval

The template above IS the solution. Trace with `[[1,3],[2,6],[8,10],[15,18]]`:
- `[1,3]` → merged = `[[1,3]]`
- `[2,6]`: 3 >= 2 → overlap → merged = `[[1,6]]`
- `[8,10]`: 6 < 8 → no overlap → merged = `[[1,6],[8,10]]`
- `[15,18]`: 10 < 15 → no overlap → merged = `[[1,6],[8,10],[15,18]]`

**Complexity:** O(n log n) for sorting, O(n) merge pass.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Not sorting first | Adjacent intervals might not overlap | `Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]))` |
| Using `curr[1] >= next[0]` instead of `<` | Off-by-one: touching intervals `[1,2],[2,3]` should merge | `merged.last[1] < curr[0]` means NO overlap |
| Forgetting `Math.max` on end | `[1,10],[2,5]` → `[1,5]` loses coverage | Always `Math.max(mergedEnd, currEnd)` |
| Modifying input array | Unexpected side effects | Use a new list for results |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "What if the input is sorted?" | "Skip the sort — O(n) total. Insert Interval (LC 57) is an example." |
| "What about interval intersections?" | "Two pointer: advance the pointer with the smaller end. Intersection exists when `max(start) <= min(end)`." |
| "How would you handle a stream of intervals?" | "Maintain a sorted set (`TreeMap`). On insert, merge with neighbors. O(log n) per insert." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Merge Intervals | Medium | Sort by start, extend overlapping | 56 |
| 2 | Insert Interval | Medium | Three-phase: before, overlap, after | 57 |
| 3 | Interval List Intersections | Medium | Two pointers, advance smaller end | 986 |
| 4 | Remove Covered Intervals | Medium | Sort by start asc, end desc; track maxEnd | 1288 |
| 5 | Summary Ranges | Easy | Walk array, extend while consecutive | 228 |

---

## 2. Scheduling (Greedy by End Time)

### When to Use
- "**Maximum non-overlapping** intervals"
- "**Minimum rooms** / resources needed"
- "**Erase overlapping** intervals"
- Selecting or counting non-conflicting intervals

### When NOT to Use
- Need to merge intervals — use Pattern 1
- Need to count events at a specific point — use Line Sweep

### Optimized Java Template

```java
// Maximum non-overlapping intervals (= N - min removals)
// Greedy: sort by END time, always pick the earliest-ending interval
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));  // sort by END

    int kept = 1;
    int lastEnd = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= lastEnd) {  // no overlap
            kept++;
            lastEnd = intervals[i][1];
        }
        // else: skip (this interval overlaps → remove it)
    }

    return intervals.length - kept;  // removals = total - kept
}

// Minimum meeting rooms (= maximum overlap at any point)
public int minMeetingRooms(int[][] intervals) {
    int n = intervals.length;
    int[] starts = new int[n], ends = new int[n];

    for (int i = 0; i < n; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }

    Arrays.sort(starts);
    Arrays.sort(ends);

    int rooms = 0, maxRooms = 0, endPtr = 0;
    for (int i = 0; i < n; i++) {
        if (starts[i] < ends[endPtr]) {
            rooms++;
        } else {
            endPtr++;  // a meeting ended before this one starts
        }
        maxRooms = Math.max(maxRooms, rooms);
    }
    return maxRooms;
}
```

**Why sort by end time for scheduling?** Picking the interval that ends earliest leaves the most room for future intervals. This is the classic activity selection proof by exchange argument.

### Fully Solved Problem: Non-overlapping Intervals (LC 435)

**Problem:** Given intervals, find the minimum number to remove to make the rest non-overlapping.

**Thinking process:**
1. "Min removals for non-overlapping" = "Max non-overlapping" and subtract from total
2. Greedy: sort by end time, greedily keep intervals that don't overlap
3. Why end time? If we sort by start, `[1,100],[2,3],[3,4]` we'd pick `[1,100]` and miss two smaller ones

```java
public int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length <= 1) return 0;

    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));

    int kept = 1;
    int prevEnd = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] >= prevEnd) {
            kept++;
            prevEnd = intervals[i][1];
        }
    }

    return intervals.length - kept;
}
```

**Trace with `[[1,2],[2,3],[3,4],[1,3]]`:**
- Sorted by end: `[[1,2],[1,3],[2,3],[3,4]]`
- Keep `[1,2]` (prevEnd=2), skip `[1,3]` (1<2), keep `[2,3]` (2>=2, prevEnd=3), keep `[3,4]` (3>=3)
- Kept=3, removals = 4-3 = 1

**Complexity:** O(n log n) time, O(1) extra space (in-place sort).

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Sorting by start for activity selection | Greedy picks long intervals, misses short ones | Sort by END time — earliest deadline first |
| `>` instead of `>=` for overlap check | `[1,2],[2,3]` treated as overlapping | `>=` means touching intervals are OK |
| Minimum rooms: using heap when not needed | Works but O(n log n) vs O(n log n) with simpler sort | Two-array sort approach is cleaner and faster |
| Confusing "min removals" with "max kept" | Returns wrong answer | min_removals = total - max_kept |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Prove the greedy is optimal" | "Exchange argument: if optimal skips an early-ending interval, we can swap it in without losing anything." |
| "What if intervals have weights?" | "Greedy fails — need DP. Sort by end, binary search for latest non-overlapping: O(n log n)." |
| "What about minimum platforms at a railway station?" | "Same as minimum meeting rooms — sort starts and ends, sweep through events." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | Non-overlapping Intervals | Medium | Sort by end, greedy keep | 435 |
| 2 | Meeting Rooms | Easy | Sort by start, check any overlap | 252 |
| 3 | Meeting Rooms II | Medium | Sort starts/ends or line sweep | 253 |
| 4 | Minimum Number of Arrows to Burst Balloons | Medium | Same as max non-overlapping | 452 |
| 5 | Task Scheduler | Medium | Greedy: most frequent task first, idle slots | 621 |

---

## 3. Line Sweep (Event-Based)

### When to Use
- "**Maximum overlap** at any point"
- "**Skyline problem**"
- "**Counting active** intervals at each time"
- "**My Calendar**" — dynamic interval booking
- Events happen at discrete times and you sweep left to right

### When NOT to Use
- Just merging intervals — Pattern 1 is simpler
- Selecting non-overlapping — Pattern 2 is direct

### Optimized Java Template

```java
// Line sweep for maximum overlap
public int maxOverlap(int[][] intervals) {
    // Create events: +1 at start, -1 at end
    TreeMap<Integer, Integer> events = new TreeMap<>();
    for (int[] interval : intervals) {
        events.merge(interval[0], 1, Integer::sum);   // +1 at start
        events.merge(interval[1], -1, Integer::sum);   // -1 at end
    }

    int active = 0, maxActive = 0;
    for (int delta : events.values()) {
        active += delta;
        maxActive = Math.max(maxActive, active);
    }
    return maxActive;
}

// Line sweep with sorted array (faster, O(n log n))
public int maxOverlapArray(int[][] intervals) {
    int n = intervals.length;
    int[][] events = new int[2 * n][2];
    for (int i = 0; i < n; i++) {
        events[2 * i] = new int[]{intervals[i][0], 1};      // start
        events[2 * i + 1] = new int[]{intervals[i][1], -1};  // end
    }

    // Sort by time; if tie, process ends (-1) before starts (+1)
    Arrays.sort(events, (a, b) -> a[0] != b[0]
        ? Integer.compare(a[0], b[0])
        : Integer.compare(a[1], b[1]));

    int active = 0, maxActive = 0;
    for (int[] event : events) {
        active += event[1];
        maxActive = Math.max(maxActive, active);
    }
    return maxActive;
}
```

**Tie-breaking matters:** When an end and a start happen at the same time, process the end first (`-1 < 1`). This means `[1,2]` and `[2,3]` are NOT considered overlapping at time 2. If you want them to overlap, reverse the tie-breaking.

### Fully Solved Problem: The Skyline Problem (LC 218)

**Problem:** Given buildings `[left, right, height]`, output the skyline as key points where the height changes.

**Thinking process:**
1. This is a line sweep where we track the maximum active height
2. Events: at `left`, a building "opens" with its height; at `right`, it "closes"
3. Use a max-heap (or `TreeMap`) to track all active heights
4. After processing each event, if the max height changed, record a key point

```java
public List<List<Integer>> getSkyline(int[][] buildings) {
    // Create events: [x, type] where type = -height for open, +height for close
    List<int[]> events = new ArrayList<>();
    for (int[] b : buildings) {
        events.add(new int[]{b[0], -b[2]});  // open: negative height (sorts first)
        events.add(new int[]{b[1], b[2]});   // close: positive height
    }

    // Sort: by x; if tie, smaller value first (opens before closes at same x,
    // taller opens before shorter opens, shorter closes before taller closes)
    events.sort((a, b) -> a[0] != b[0]
        ? Integer.compare(a[0], b[0])
        : Integer.compare(a[1], b[1]));

    // TreeMap as a max-heap with counts (handles duplicate heights)
    TreeMap<Integer, Integer> heights = new TreeMap<>(Collections.reverseOrder());
    heights.put(0, 1);  // ground level always present

    int prevMax = 0;
    List<List<Integer>> result = new ArrayList<>();

    for (int[] event : events) {
        int x = event[0], h = event[1];

        if (h < 0) {  // open
            heights.merge(-h, 1, Integer::sum);
        } else {       // close
            int count = heights.get(h);
            if (count == 1) heights.remove(h);
            else heights.put(h, count - 1);
        }

        int currMax = heights.firstKey();
        if (currMax != prevMax) {
            result.add(List.of(x, currMax));
            prevMax = currMax;
        }
    }

    return result;
}
```

**Why `TreeMap` over `PriorityQueue`?** We need O(log n) removal of specific heights when a building closes. PQ.remove() is O(n). TreeMap with counts gives O(log n) for all operations.

**Complexity:** O(n log n) time, O(n) space.

### Common Traps

| Trap | What Happens | How to Handle |
|------|-------------|---------------|
| Wrong tie-breaking in skyline | Duplicate points or missed transitions | Opens before closes at same x; taller opens first |
| PriorityQueue for skyline | O(n) removal when buildings close | Use `TreeMap` with counts instead |
| Not tracking ground level (height 0) | When all buildings close, max height becomes null | Initialize `TreeMap` with `{0: 1}` |
| Forgetting to handle duplicate heights | `TreeMap.remove` deletes all instances | Track counts, only remove when count reaches 0 |

### Interviewer Counter-Questions

| Follow-up | Expected Response |
|-----------|-------------------|
| "Time complexity of skyline?" | "O(n log n): sorting events + n iterations with O(log n) TreeMap operations." |
| "Can you solve it with divide and conquer?" | "Yes — recursively compute skyline of left and right halves, then merge like merge sort. Same O(n log n)." |
| "What about an online version (buildings arrive one at a time)?" | "Use a segment tree or balanced BST. Each building insertion updates O(log n) entries." |

### Practice Problems

| # | Problem | Difficulty | Key Insight | LC # |
|---|---------|------------|-------------|------|
| 1 | The Skyline Problem | Hard | Line sweep with TreeMap for max active height | 218 |
| 2 | Meeting Rooms II | Medium | Line sweep or sorted starts/ends | 253 |
| 3 | My Calendar I | Medium | TreeMap + overlap check with floorEntry/ceilingEntry | 729 |
| 4 | My Calendar II | Medium | Line sweep: event map, check if count ever > 2 | 731 |
| 5 | My Calendar III | Hard | Line sweep: return max overlap across all events | 732 |
| 6 | Car Pooling | Medium | Line sweep on trip events, check capacity | 1094 |
| 7 | Corporate Flight Bookings | Medium | Difference array (1D line sweep) | 1109 |

---

## Interval Pattern Decision Framework

| Question to Ask | Answer → Pattern |
|----------------|------------------|
| Need to simplify/combine intervals? | **Merge** (sort by start) |
| Need max non-overlapping / min removals? | **Scheduling** (sort by end) |
| Need max overlap count at any point? | **Line Sweep** (events ±1) |
| Need to find free time / gaps? | **Merge** then scan gaps |
| Heights/values associated with intervals? | **Line Sweep** with max-heap or TreeMap |

---

## Cross-References

- **Meeting Rooms II** also solvable via min-heap — see [Heaps](./09-heaps.md)
- **Difference array** is a discrete line sweep — see [Arrays & Hashing](./01-arrays-and-hashing.md#4-prefix-sum)
- **Weighted interval scheduling** needs DP — see [Dynamic Programming](./07-dynamic-programming.md)
- **Task Scheduler** uses greedy frequency counting — see [Greedy](./11-greedy.md)
