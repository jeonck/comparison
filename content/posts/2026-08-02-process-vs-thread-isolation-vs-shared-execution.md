---
title: "Process vs Thread: Isolation vs Shared Execution"
date: 2026-08-02T05:43:29.187240+09:00
tags: ["operating-systems", "concurrency", "process-management", "multithreading"]
---
## Overview

A process is an independently executing program instance with its own private address space, while a thread is a lightweight unit of execution that runs inside a process and shares that process's memory with its sibling threads. The distinction matters because it determines how much isolation, communication overhead, and crash-safety you get versus how cheap context switching and data sharing are.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <text x="95" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Process</text>
  <text x="475" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Thread</text>
  <rect x="20" y="55" width="110" height="250" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="75" y="72" text-anchor="middle" style="fill:var(--primary)" font-size="11" font-weight="bold">Process A</text>
  <rect x="30" y="80" width="90" height="24" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="75" y="96" text-anchor="middle" style="fill:var(--content)" font-size="9">Code</text>
  <rect x="30" y="108" width="90" height="24" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="75" y="124" text-anchor="middle" style="fill:var(--content)" font-size="9">Data</text>
  <rect x="30" y="136" width="90" height="34" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="75" y="157" text-anchor="middle" style="fill:var(--content)" font-size="9">Heap</text>
  <rect x="30" y="174" width="90" height="48" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="75" y="200" text-anchor="middle" style="fill:var(--content)" font-size="9">Stack</text>
  <rect x="170" y="55" width="110" height="250" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="225" y="72" text-anchor="middle" style="fill:var(--primary)" font-size="11" font-weight="bold">Process B</text>
  <rect x="180" y="80" width="90" height="24" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="225" y="96" text-anchor="middle" style="fill:var(--content)" font-size="9">Code</text>
  <rect x="180" y="108" width="90" height="24" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="225" y="124" text-anchor="middle" style="fill:var(--content)" font-size="9">Data</text>
  <rect x="180" y="136" width="90" height="34" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="225" y="157" text-anchor="middle" style="fill:var(--content)" font-size="9">Heap</text>
  <rect x="180" y="174" width="90" height="48" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="225" y="200" text-anchor="middle" style="fill:var(--content)" font-size="9">Stack</text>
  <line x1="131" y1="165" x2="169" y2="165" style="stroke:var(--secondary)" stroke-width="1.5" stroke-dasharray="4,3"/>
  <text x="150" y="245" text-anchor="middle" style="fill:var(--secondary)" font-size="9">no shared memory</text>
  <text x="150" y="258" text-anchor="middle" style="fill:var(--secondary)" font-size="9">(IPC only)</text>
  <rect x="340" y="55" width="270" height="250" style="fill:none;stroke:var(--border)" stroke-width="1.5"/>
  <text x="475" y="70" text-anchor="middle" style="fill:var(--secondary)" font-size="9">single process address space</text>
  <rect x="355" y="82" width="240" height="50" style="fill:none;stroke:var(--border)" stroke-width="1.5"/>
  <text x="475" y="103" text-anchor="middle" style="fill:var(--content)" font-size="9">Shared Code / Data / Heap</text>
  <line x1="390" y1="132" x2="390" y2="160" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="475" y1="132" x2="475" y2="160" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="560" y1="132" x2="560" y2="160" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <rect x="355" y="160" width="70" height="130" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="390" y="176" text-anchor="middle" style="fill:var(--primary)" font-size="9" font-weight="bold">Thread 1</text>
  <rect x="362" y="184" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="390" y="209" text-anchor="middle" style="fill:var(--content)" font-size="8">Stack</text>
  <rect x="362" y="232" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="390" y="257" text-anchor="middle" style="fill:var(--content)" font-size="8">Registers</text>
  <rect x="440" y="160" width="70" height="130" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="475" y="176" text-anchor="middle" style="fill:var(--primary)" font-size="9" font-weight="bold">Thread 2</text>
  <rect x="447" y="184" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="475" y="209" text-anchor="middle" style="fill:var(--content)" font-size="8">Stack</text>
  <rect x="447" y="232" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="475" y="257" text-anchor="middle" style="fill:var(--content)" font-size="8">Registers</text>
  <rect x="525" y="160" width="70" height="130" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="560" y="176" text-anchor="middle" style="fill:var(--primary)" font-size="9" font-weight="bold">Thread 3</text>
  <rect x="532" y="184" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="560" y="209" text-anchor="middle" style="fill:var(--content)" font-size="8">Stack</text>
  <rect x="532" y="232" width="56" height="44" style="fill:none;stroke:var(--border)" stroke-width="1"/>
  <text x="560" y="257" text-anchor="middle" style="fill:var(--content)" font-size="8">Registers</text>
  <text x="475" y="312" text-anchor="middle" style="fill:var(--secondary)" font-size="9">shared heap/code, private stack per thread</text>
</svg>
</div>

## Comparison Table

| Aspect | Process | Thread |
| --- | --- | --- |
| Memory space | Own isolated virtual address space | Shares address space with sibling threads in the same process |
| Creation cost | Expensive (fork/CreateProcess, new page tables) | Cheap (allocate stack + TCB, reuse existing address space) |
| Context switch cost | Higher (flush TLB, swap page tables) | Lower (same address space, just swap registers/stack pointer) |
| Communication | Requires IPC: pipes, sockets, shared memory, message queues | Direct via shared variables/heap; needs locks/mutexes for safety |
| Fault isolation | A crash typically stays contained to that process | A crash (e.g. bad pointer, unhandled exception) can take down the whole process |
| Scheduling unit | OS schedules processes (which contain ≥ 1 thread) | OS (or runtime) schedules threads independently within a process |
| Concurrency primitives needed | Rarely needed within a single process | Mutexes, semaphores, atomics to guard shared state |
| Typical use case | Running separate, independently-failing programs (browser tabs as processes, microservices) | Parallelizing work within one program (web server handling many requests, UI thread + workers) |

## Key Differences

- A thread lives inside a process and shares its code, heap, and open file handles; a process owns its own private address space.
- Threads communicate by directly reading/writing shared memory (needing synchronization); processes must use explicit IPC mechanisms.
- Creating and context-switching a thread is much cheaper than doing the same for a process, since no new address space or page table is involved.
- A crashing thread can corrupt or kill its entire parent process; a crashing process is generally isolated from other processes by the OS.
- Multiple threads share one process's resource limits (file descriptors, memory quota); each process gets its own.

## When to Use Each

**Process**

- **Sandboxing untrusted code**: Since a crash typically stays contained to its own process, running a plugin or third-party code in a separate process keeps it from taking down the host program.
- **Isolating independently-failing services**: Microservices or browser tabs run as separate processes so one crashing process doesn't corrupt the memory or state of the others.
- **Enforcing independent resource limits**: When each workload needs its own file descriptor and memory quota rather than sharing one process's limits, separate processes give that isolation.

**Thread**

- **Handling many concurrent connections**: A web server juggling many simultaneous requests benefits from threads' cheap creation and low context-switch cost compared to spawning a process per request.
- **Parallelizing CPU work without duplicating memory**: Threads share the same code, data, and heap, so offloading work to worker threads avoids copying memory the way separate processes would require.
- **Tight in-program data sharing**: When multiple execution units need to read and write the same in-memory state directly (guarded by mutexes), threads avoid the IPC overhead processes would need.
