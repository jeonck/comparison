---
title: "Sync vs Async APIs: Blocking Calls vs Non-Blocking Callbacks"
date: 2026-09-06T10:05:37.098530+09:00
tags: ["sync-vs-async", "api-design", "concurrency", "distributed-systems"]
---
## Overview

A <strong class="kw">synchronous call</strong> blocks the caller until the server returns a result, tying up a thread or connection for the full round trip. An <strong class="kw">asynchronous call</strong> returns immediately with an acknowledgment and delivers the actual result later via a callback, event, or poll, letting the caller do other work in the meantime.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="320" y="40" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Synchronous API</text><text x="60" y="99" font-size="13" style="fill:var(--content)">Client</text><text x="320" y="80" text-anchor="middle" font-size="11" style="fill:var(--secondary)">blocked - thread waits</text><line x1="100" y1="100" x2="580" y2="100" stroke-width="2" style="stroke:var(--compare-a)"/><polygon points="580,94 592,100 580,106" style="fill:var(--compare-a)"/><rect x="220" y="92" width="200" height="16" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="220" cy="100" r="5" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="420" cy="100" r="5" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="220" y="122" text-anchor="middle" font-size="11" style="fill:var(--content)">request sent</text><text x="420" y="122" text-anchor="middle" font-size="11" style="fill:var(--content)">response received</text><line x1="220" y1="106" x2="220" y2="135" stroke-width="1" stroke-dasharray="3 3" style="stroke:var(--border)"/><line x1="420" y1="106" x2="420" y2="135" stroke-width="1" stroke-dasharray="3 3" style="stroke:var(--border)"/><rect x="220" y="135" width="200" height="10" stroke-width="1" stroke-dasharray="3 3" style="fill:none;stroke:var(--border)"/><text x="320" y="161" text-anchor="middle" font-size="11" style="fill:var(--secondary)">server processing</text><line x1="40" y1="188" x2="600" y2="188" stroke-width="1" stroke-dasharray="4 4" style="stroke:var(--border)"/><text x="320" y="213" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Asynchronous API</text><text x="60" y="259" font-size="13" style="fill:var(--content)">Client</text><rect x="250" y="233" width="140" height="18" stroke-width="1" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="320" y="246" text-anchor="middle" font-size="10" style="fill:var(--content)">client free: other work</text><line x1="100" y1="260" x2="580" y2="260" stroke-width="2" style="stroke:var(--compare-b)"/><polygon points="580,254 592,260 580,266" style="fill:var(--compare-b)"/><circle cx="220" cy="260" r="5" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="420" cy="260" r="5" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="220" y="283" text-anchor="middle" font-size="11" style="fill:var(--content)">request sent</text><text x="420" y="283" text-anchor="middle" font-size="11" style="fill:var(--content)">callback received</text><line x1="220" y1="266" x2="220" y2="300" stroke-width="1" stroke-dasharray="3 3" style="stroke:var(--border)"/><line x1="420" y1="266" x2="420" y2="300" stroke-width="1" stroke-dasharray="3 3" style="stroke:var(--border)"/><rect x="220" y="300" width="200" height="10" stroke-width="1" stroke-dasharray="3 3" style="fill:none;stroke:var(--border)"/><text x="320" y="326" text-anchor="middle" font-size="11" style="fill:var(--secondary)">server working</text></svg>
</div>

## Comparison Table

| Aspect | Sync API | Async API |
| --- | --- | --- |
| Request initiation | Caller invokes and immediately awaits the result on the same call | Caller invokes and gets an immediate acknowledgment or handle (future, promise, message ID), not the result |
| Response delivery | Result returned in-line over the same connection/thread that made the call | Result delivered later via callback, event, webhook, or by polling |
| Caller behavior while waiting | Thread or connection is blocked and cannot do other work | Caller is free to continue other work or serve other requests |
| Concurrency model | Needs roughly one thread or connection per in-flight call | A single thread or event loop can multiplex many in-flight calls |
| Failure handling | Errors surface immediately as exceptions or status codes at the call site | Errors arrive out-of-band later and must be matched back to the original request |
| Ordering and sequencing | Strict: caller code executes in the exact order calls complete | Responses can arrive out of order, requiring correlation IDs to reassemble sequence |
| Latency impact on caller | Caller's total latency equals the full round trip | Caller's perceived latency is just the time to ack; real work overlaps with other tasks |
| Implementation complexity | Simpler code: straightforward call and return | More complex: needs callback/promise/event handling and explicit state tracking |

## Key Differences

- Sync calls <strong class="kw">block</strong> the caller until the response arrives, while async calls return control immediately.
- Async APIs scale better under load because they avoid <strong class="kw">thread-per-request</strong> limits inherent to blocking calls.
- Sync errors surface in-line at the call site; async errors require <strong class="kw">correlation</strong> back to the original request.
- Async responses can arrive out of order, adding <strong class="kw">sequencing</strong> complexity that sync calls never face.
- Sync code is easier to trace and debug since execution follows a single linear <strong class="kw">call stack</strong>.

## When to Use Each

**Sync API**

- **Simple request/response flows**: Use sync when the caller genuinely needs the result before it can proceed, like fetching a value to render a page.
- **Low-latency internal calls**: For fast in-process or same-datacenter calls, blocking briefly is cheap and the simpler code is worth it.
- **Straightforward debugging**: Stack traces and logs map directly to the call sequence, making failures easier to reason about.

**Async API**

- **Long-running operations**: Tasks like video encoding or batch jobs shouldn't force a caller to sit blocked for minutes.
- **High-concurrency services**: Gateways or brokers handling thousands of simultaneous connections need to avoid one thread per request.
- **Event-driven integrations**: Webhooks and message queues let systems react to events instead of polling or waiting inline.
