---
title: "Concurrency vs Parallelism: Interleaving vs Simultaneous Execution"
date: 2026-08-02T23:53:42.755088+09:00
tags: ["concurrency", "parallelism", "multithreading", "cpu-architecture"]
---
## Overview

Concurrency and parallelism are often used interchangeably, but they describe different things: concurrency is about <strong class="kw">interleaving</strong> multiple tasks so a program can make progress on all of them, while parallelism is about <strong class="kw">simultaneous execution</strong> of tasks on separate hardware. A single-core CPU can be concurrent but never truly parallel; a multi-core CPU can be both.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="50" x2="320" y2="330" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Concurrency</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Parallelism</text><rect x="110" y="55" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="80" text-anchor="middle" style="fill:var(--content)" font-size="12">1 CPU Core</text><rect x="40" y="140" width="44" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="62" y="163" text-anchor="middle" style="fill:var(--content)" font-size="12">A</text><rect x="88" y="140" width="44" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="163" text-anchor="middle" style="fill:var(--content)" font-size="12">B</text><rect x="136" y="140" width="36" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="154" y="163" text-anchor="middle" style="fill:var(--content)" font-size="12">A</text><rect x="176" y="140" width="52" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="202" y="163" text-anchor="middle" style="fill:var(--content)" font-size="12">C</text><rect x="232" y="140" width="48" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="256" y="163" text-anchor="middle" style="fill:var(--content)" font-size="12">B</text><line x1="40" y1="195" x2="290" y2="195" style="stroke:var(--border)" stroke-width="1"/><path d="M290,195 L283,191 L283,199 Z" style="fill:var(--border)"/><text x="165" y="212" text-anchor="middle" style="fill:var(--secondary)" font-size="11">time (single lane, tasks interleave)</text><text x="165" y="230" text-anchor="middle" style="fill:var(--secondary)" font-size="11">only one task runs at any instant</text><rect x="380" y="60" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="420" y="80" text-anchor="middle" style="fill:var(--content)" font-size="11">Core 1</text><rect x="380" y="105" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="420" y="125" text-anchor="middle" style="fill:var(--content)" font-size="11">Core 2</text><rect x="380" y="150" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="420" y="170" text-anchor="middle" style="fill:var(--content)" font-size="11">Core 3</text><rect x="480" y="60" width="100" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="80" text-anchor="middle" style="fill:var(--content)" font-size="12">A</text><rect x="480" y="105" width="100" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="125" text-anchor="middle" style="fill:var(--content)" font-size="12">B</text><rect x="480" y="150" width="100" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="170" text-anchor="middle" style="fill:var(--content)" font-size="12">C</text><line x1="480" y1="55" x2="480" y2="195" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><line x1="580" y1="55" x2="580" y2="195" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><line x1="480" y1="205" x2="580" y2="205" style="stroke:var(--border)" stroke-width="1"/><path d="M580,205 L573,201 L573,209 Z" style="fill:var(--border)"/><text x="530" y="222" text-anchor="middle" style="fill:var(--secondary)" font-size="11">time (all run at once)</text><text x="530" y="240" text-anchor="middle" style="fill:var(--secondary)" font-size="11">tasks execute simultaneously</text></svg>
</div>

## Comparison Table

| Aspect | Concurrency | Parallelism |
| --- | --- | --- |
| Core definition | Structuring a program to make progress on multiple tasks by interleaving them | Executing multiple tasks or subtasks at the literal same instant |
| Hardware requirement | Works on a single core via context switching | Requires multiple cores, processors, or SIMD units |
| Execution pattern | Tasks take turns; interleaved progress, order not guaranteed | Tasks run simultaneously on separate execution units |
| Task independence | Tasks often share state and need coordination to interleave safely | Tasks are usually split to run independently, minimizing interference |
| Primary goal | Improve responsiveness and structure for handling many things at once | Improve throughput by doing more work in the same time |
| Typical workload | I/O-bound: network calls, file access, user events | CPU-bound: numerical computation, data processing |
| Common pitfalls | Race conditions, deadlocks, callback/coordination complexity | Synchronization overhead, diminishing returns (Amdahl's law) |
| Language/runtime constructs | Event loops, coroutines, async/await, green threads | OS threads, multiprocessing, GPU kernels, SIMD |

## Key Differences

- Concurrency is a way of <strong class="kw">structuring</strong> code to deal with multiple tasks; it doesn't require them to run at the same instant.
- Parallelism requires <strong class="kw">multiple cores</strong> executing work simultaneously, while concurrency runs fine on a single core.
- Concurrent code can run without being parallel, and parallel code (like <strong class="kw">SIMD</strong> data processing) can run without concurrent structure.
- Concurrency's main hazard is a <strong class="kw">race condition</strong> from shared state; parallelism's main cost is synchronization overhead.
- Concurrency optimizes for <strong class="kw">responsiveness</strong>; parallelism optimizes for raw computational throughput.

## When to Use Each

**Concurrency**

- **I/O-Bound Workloads**: Concurrency keeps a program responsive while waiting on network, disk, or user input, since tasks only need to interleave, not run at the same instant.
- **Single-Core Environments**: Because concurrency works through context switching, it delivers responsiveness gains even when only one core is available.
- **Coordinating Many Independent Requests**: Event loops, coroutines, and async/await let one thread structure progress across many in-flight tasks, such as handling concurrent network calls.

**Parallelism**

- **CPU-Bound Computation**: When multiple cores are available, splitting numerical or data-processing work to run simultaneously genuinely increases throughput.
- **Maximizing Hardware Utilization**: Parallelism via OS threads, multiprocessing, or SIMD units exploits multiple cores or processors instead of a single one.
- **Batch Data Processing at Scale**: Independent subtasks that don't need to share state are natural candidates for parallel execution, minimizing synchronization overhead.
