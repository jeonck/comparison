---
title: "Recursion vs Iteration: Two Ways to Repeat Work"
date: 2026-08-02T23:45:29.339507+09:00
tags: ["recursion", "iteration", "algorithms", "data-structures"]
---
## Overview

Both techniques repeat a computation until some condition is met, but they do it through fundamentally different mechanisms. <strong class="kw">Recursion</strong> repeats by having a function call itself on the call stack, while <strong class="kw">iteration</strong> repeats by looping over the same stack frame. The choice affects memory usage, readability, and how deep a computation can safely go.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-a)"/></marker><marker id="arrB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Recursion</text><text x="480" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Iteration</text><text x="160" y="58" text-anchor="middle" style="fill:var(--secondary)" font-size="11">call stack</text><rect x="60" y="66" width="200" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="87" text-anchor="middle" style="fill:var(--content)" font-size="13">f(n)</text><rect x="60" y="104" width="200" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="125" text-anchor="middle" style="fill:var(--content)" font-size="13">f(n-1)</text><rect x="60" y="142" width="200" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="163" text-anchor="middle" style="fill:var(--content)" font-size="13">f(n-2)</text><rect x="60" y="180" width="200" height="26" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="160" y="197" text-anchor="middle" style="fill:var(--secondary)" font-size="13">...</text><line x1="278" y1="70" x2="278" y2="200" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrA)"/><text x="160" y="230" text-anchor="middle" style="fill:var(--secondary)" font-size="12">new frame per call</text><text x="160" y="300" text-anchor="middle" style="fill:var(--content)" font-size="13">O(n) stack memory</text><text x="160" y="318" text-anchor="middle" style="fill:var(--secondary)" font-size="11">risk: stack overflow</text><text x="480" y="58" text-anchor="middle" style="fill:var(--secondary)" font-size="11">loop body (single frame)</text><rect x="390" y="140" width="180" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="165" text-anchor="middle" style="fill:var(--content)" font-size="13">while / for</text><path d="M 560 185 C 560 225 400 225 400 185" style="stroke:var(--compare-b);fill:none" stroke-width="1.5" marker-end="url(#arrB)"/><text x="480" y="245" text-anchor="middle" style="fill:var(--secondary)" font-size="12">i++, same frame reused</text><text x="480" y="300" text-anchor="middle" style="fill:var(--content)" font-size="13">O(1) stack memory</text><text x="480" y="318" text-anchor="middle" style="fill:var(--secondary)" font-size="11">no growth per cycle</text></svg>
</div>

## Comparison Table

| Aspect | Recursion | Iteration |
| --- | --- | --- |
| Core mechanism | Function calls itself with a smaller subproblem | Explicit loop (for/while) repeats a block of code |
| Termination condition | A base case stops further calls | A loop condition evaluates to false |
| State storage | Held implicitly in parameters and locals of each stack frame | Held explicitly in variables updated each pass |
| Memory usage | O(n) call stack space, one new frame per call | O(1) auxiliary space, the same frame is reused |
| Failure mode | Stack overflow on deep or unbounded recursion | Infinite loop if the condition never turns false |
| Performance overhead | Function-call overhead per level unless tail-call optimized | No call overhead, just a branch/jump back |
| Best-fit problems | Tree/graph traversal, divide-and-conquer, backtracking | Linear, bounded repetition like counting or array scans |

## Key Differences

- Recursion breaks a problem into <strong class="kw">self-similar subcalls</strong>; iteration repeats a block via an explicit loop.
- Recursion grows the <strong class="kw">call stack</strong> by one frame per call; iteration reuses a single frame.
- Deep recursion risks <strong class="kw">stack overflow</strong>, while iteration's main failure mode is an infinite loop.
- Some languages apply <strong class="kw">tail-call optimization</strong> to turn tail recursion into iteration under the hood.
- Recursion maps naturally onto <strong class="kw">trees and graphs</strong>; iteration suits flat, bounded repetition.

## When to Use Each

**Recursion**

- **Tree and graph traversal**: These structures are naturally self-similar, so recursion's stack-frame-per-call model maps directly onto walking children or neighbors without manual bookkeeping.
- **Divide-and-conquer algorithms**: Problems that split into smaller subproblems (like merge sort) express cleanly as recursive calls where each frame holds its own subproblem state.
- **Backtracking with bounded depth**: When depth stays bounded, recursion's implicit state storage in each frame's locals is simpler to reason about than manually tracking a stack.

**Iteration**

- **Large or unbounded repetition counts**: Since iteration reuses a single frame with O(1) auxiliary space, it avoids the stack overflow risk recursion faces on deep or unbounded loops.
- **Performance-sensitive tight loops**: Iteration has no per-level function-call overhead, just a branch back, making it faster than non-tail-call-optimized recursion for simple counting or scans.
- **Languages without tail-call optimization**: When the runtime won't collapse tail recursion into a loop automatically, writing the loop directly avoids the memory cost recursion would otherwise incur.
