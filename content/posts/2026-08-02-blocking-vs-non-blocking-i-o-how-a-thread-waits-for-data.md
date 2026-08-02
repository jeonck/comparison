---
title: "Blocking vs Non-blocking I/O: How a Thread Waits for Data"
date: 2026-08-02T09:10:28.123100+09:00
tags: ["io", "concurrency", "networking", "async"]
---
## Overview

Blocking and non-blocking I/O differ in what happens to the calling thread when a read or write can't complete immediately. Blocking I/O suspends the thread until data is ready; non-blocking I/O returns at once and lets the caller check back later. The choice shapes how a system scales to many simultaneous connections.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="45" x2="320" y2="315" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="28" text-anchor="middle" font-size="16" font-weight="600" style="fill:var(--primary)">Blocking I/O</text><text x="480" y="28" text-anchor="middle" font-size="16" font-weight="600" style="fill:var(--primary)">Non-blocking I/O</text><rect x="100" y="50" width="120" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">Thread</text><line x1="160" y1="80" x2="160" y2="102" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="160,105 156,98 164,98" style="fill:var(--secondary)"/><rect x="70" y="105" width="180" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="125" text-anchor="middle" font-size="12" style="fill:var(--content)">call read()</text><line x1="160" y1="135" x2="160" y2="157" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="160,160 156,153 164,153" style="fill:var(--secondary)"/><rect x="70" y="160" width="180" height="85" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="5,3"/><text x="160" y="198" text-anchor="middle" font-size="12" style="fill:var(--content)">thread blocked</text><text x="160" y="216" text-anchor="middle" font-size="11" style="fill:var(--secondary)">no other work possible</text><text x="160" y="234" text-anchor="middle" font-size="11" style="fill:var(--secondary)">until data arrives</text><line x1="160" y1="245" x2="160" y2="267" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="160,270 156,263 164,263" style="fill:var(--secondary)"/><rect x="70" y="270" width="180" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="290" text-anchor="middle" font-size="12" style="fill:var(--content)">data ready, resumes</text><text x="160" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">1 thread ≈ 1 in-flight call</text><rect x="390" y="50" width="180" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">thread / event loop</text><line x1="480" y1="80" x2="480" y2="102" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,105 476,98 484,98" style="fill:var(--secondary)"/><rect x="390" y="105" width="180" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="125" text-anchor="middle" font-size="12" style="fill:var(--content)">read() returns at once</text><line x1="480" y1="135" x2="480" y2="157" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,160 476,153 484,153" style="fill:var(--secondary)"/><rect x="390" y="160" width="180" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="182" text-anchor="middle" font-size="12" style="fill:var(--content)">poll / epoll check</text><path d="M 570 168 C 595 168 595 190 572 190" style="stroke:var(--compare-b);fill:none" stroke-width="1.3"/><polygon points="572,190 579,187 578,194" style="fill:var(--compare-b)"/><text x="596" y="182" font-size="9" style="fill:var(--secondary)">retry</text><line x1="480" y1="195" x2="480" y2="207" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,210 476,203 484,203" style="fill:var(--secondary)"/><rect x="390" y="210" width="180" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="228" text-anchor="middle" font-size="11" style="fill:var(--content)">meanwhile: serves other</text><text x="480" y="241" text-anchor="middle" font-size="11" style="fill:var(--content)">connections</text><line x1="480" y1="245" x2="480" y2="267" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,270 476,263 484,263" style="fill:var(--secondary)"/><rect x="390" y="270" width="180" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="290" text-anchor="middle" font-size="12" style="fill:var(--content)">ready → callback fires</text><text x="480" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">1 thread ≈ many in-flight calls</text></svg>
</div>

## Comparison Table

| Aspect | Blocking I/O | Non-blocking I/O |
| --- | --- | --- |
| Call behavior | Call halts the calling thread until the operation completes | Call returns immediately, with data or an EWOULDBLOCK/EAGAIN error |
| Thread state while I/O is pending | Thread is suspended off the run queue — no CPU used, but unavailable for other work | Thread stays runnable and can be reused to serve other requests |
| How readiness is discovered | OS wakes the thread automatically once data is ready | Caller polls or registers with select/poll/epoll/kqueue |
| Concurrency model | One thread (or process) per concurrent connection | Single or few threads multiplex many connections via an event loop |
| Resource overhead at scale | Grows linearly with connections — thread stacks, context switches | Stays flat, bounded by CPU cores rather than connection count |
| Code / control-flow complexity | Simple, sequential, top-to-bottom logic | Callback, promise, or async/await structure; state tracked across suspensions |
| Failure / edge-case handling | A slow or hung peer blocks the thread indefinitely without a timeout | A slow peer only delays its own event; a stalled callback can starve the whole loop |

## Key Differences

- Blocking I/O ties up a <strong class="kw">thread</strong> for the full call; non-blocking frees it immediately.
- Non-blocking servers rely on an <strong class="kw">event loop</strong> to learn when data is finally ready.
- Blocking scales concurrency with more <strong class="kw">threads</strong>; non-blocking scales with more <strong class="kw">callbacks</strong> on fewer threads.
- Blocking code reads <strong class="kw">sequentially</strong>; non-blocking code needs explicit <strong class="kw">state management</strong> across suspensions.
- At high connection counts, blocking hits a <strong class="kw">C10K</strong> wall that non-blocking avoids.

## When to Use Each

**Blocking I/O**

- **Simple Scripts and CLI Tools**: Sequential, top-to-bottom blocking calls keep code easy to read and debug when high concurrency isn't a concern.
- **Low-Concurrency Services**: With only a handful of connections active, the one-thread-per-connection model never reaches the resource overhead that scales linearly with connections.
- **Straightforward Failure Handling**: Without callback or promise state to track, errors surface in a linear call stack that's simpler to trace than a suspended async chain.

**Non-blocking I/O**

- **High-Concurrency Servers**: Proxies, chat backends, and API gateways handling thousands of simultaneous connections need an event loop to avoid the C10K wall blocking I/O hits.
- **Resource-Bounded Scaling**: Since overhead stays flat and bounded by CPU cores rather than connection count, non-blocking I/O suits systems where thread-per-connection costs would be prohibitive.
- **Multiplexing Many Slow Clients**: A single event loop can serve many peers at once, so one slow connection only delays its own event instead of tying up an entire thread.
