---
title: "BFS vs DFS: Traversal Order and Data Structure"
date: 2026-08-02T23:44:09.909395+09:00
tags: ["algorithms", "graph-theory", "data-structures", "tree-traversal"]
---
## Overview

Both are algorithms for visiting every node in a tree or graph, but they differ in which node they explore next. <strong class="kw">BFS</strong> spreads outward level by level using a queue, while <strong class="kw">DFS</strong> plunges down one path as far as possible before backtracking, using a stack.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="5,5"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">BFS</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">DFS</text><line x1="160" y1="50" x2="100" y2="120" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="50" x2="220" y2="120" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="100" y1="120" x2="70" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="100" y1="120" x2="130" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="220" y1="120" x2="190" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="220" y1="120" x2="250" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="160" cy="50" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="55" text-anchor="middle" style="fill:var(--content)" font-size="13">1</text><circle cx="100" cy="120" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="100" y="125" text-anchor="middle" style="fill:var(--content)" font-size="13">2</text><circle cx="220" cy="120" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="220" y="125" text-anchor="middle" style="fill:var(--content)" font-size="13">3</text><circle cx="70" cy="190" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">4</text><circle cx="130" cy="190" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">5</text><circle cx="190" cy="190" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="190" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">6</text><circle cx="250" cy="190" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">7</text><line x1="480" y1="50" x2="420" y2="120" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="480" y1="50" x2="540" y2="120" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="420" y1="120" x2="390" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="420" y1="120" x2="450" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="540" y1="120" x2="510" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="540" y1="120" x2="570" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="480" cy="50" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="55" text-anchor="middle" style="fill:var(--content)" font-size="13">1</text><circle cx="420" cy="120" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="420" y="125" text-anchor="middle" style="fill:var(--content)" font-size="13">2</text><circle cx="540" cy="120" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="540" y="125" text-anchor="middle" style="fill:var(--content)" font-size="13">5</text><circle cx="390" cy="190" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">3</text><circle cx="450" cy="190" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="450" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">4</text><circle cx="510" cy="190" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">6</text><circle cx="570" cy="190" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="570" y="195" text-anchor="middle" style="fill:var(--content)" font-size="13">7</text><rect x="60" y="260" width="28" height="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="92" y="260" width="28" height="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="124" y="260" width="28" height="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="156" y="260" width="28" height="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="188" y="260" width="28" height="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="20" y="277" style="fill:var(--secondary)" font-size="11">out</text><text x="232" y="277" style="fill:var(--secondary)" font-size="11">in</text><text x="140" y="305" text-anchor="middle" style="fill:var(--secondary)" font-size="13">Queue (FIFO)</text><rect x="466" y="228" width="56" height="22" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="466" y="252" width="56" height="22" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="466" y="276" width="56" height="22" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="466" y="300" width="56" height="22" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="535" y="243" style="fill:var(--secondary)" font-size="11">push/pop</text><text x="494" y="340" text-anchor="middle" style="fill:var(--secondary)" font-size="13">Stack (LIFO)</text></svg>
</div>

## Comparison Table

| Aspect | BFS | DFS |
| --- | --- | --- |
| Underlying structure | Queue (FIFO) | Stack (FILO), often via recursion |
| Traversal pattern | Explores all neighbors at current depth before going deeper | Follows one branch to its end before backtracking |
| Order nodes are visited | Level by level (breadth-first) | Branch by branch (depth-first) |
| Memory usage | O(width) — can be large for wide/bushy graphs | O(depth) — can be large for deep graphs |
| Shortest path guarantee | Yes, on unweighted graphs (first visit = shortest path) | No, may find a longer path first |
| Implementation style | Iterative with explicit queue | Recursive, or iterative with explicit stack |
| Risk of infinite loop | Low with visited-set on cyclic graphs | Higher on cyclic graphs without visited-set, or infinite depth |
| Typical use cases | Shortest path, level-order processing, web crawling by hops | Topological sort, cycle detection, maze/backtracking problems |

## Key Differences

- <strong class="kw">BFS</strong> uses a queue and expands outward level by level, while <strong class="kw">DFS</strong> uses a stack (or recursion) and dives deep before backtracking.
- BFS guarantees the <strong class="kw">shortest path</strong> on unweighted graphs; DFS does not.
- BFS memory cost scales with graph <strong class="kw">width</strong>, while DFS memory cost scales with graph <strong class="kw">depth</strong>.
- DFS naturally supports <strong class="kw">backtracking</strong> algorithms like maze solving and topological sort.
- Both require a <strong class="kw">visited set</strong> to avoid infinite loops on cyclic graphs.

## When to Use Each

**BFS**

- **Shortest Path in Unweighted Graphs**: BFS's level-by-level order guarantees that the first visit to a node comes via the shortest path.
- **Level-Order Tree Processing**: When output must be organized by distance from the root, BFS's queue naturally produces nodes in that order.
- **Web Crawling by Hop Count**: BFS suits exploring nearest neighbors first, such as crawling pages a fixed number of links away from a start page.

**DFS**

- **Memory-Constrained Deep Graphs**: DFS's O(depth) memory footprint, versus BFS's O(width), makes it preferable when a graph is narrow but deep.
- **Topological Sort and Cycle Detection**: These problems rely on DFS's ability to fully explore one branch and backtrack before moving to the next.
- **Backtracking Puzzles like Mazes**: DFS's stack-based dive-then-backtrack pattern matches problems that require exhausting one path before trying alternatives.
