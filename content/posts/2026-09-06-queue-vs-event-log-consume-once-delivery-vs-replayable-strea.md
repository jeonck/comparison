---
title: "Queue vs Event Log: Consume-Once Delivery vs Replayable Stream"
date: 2026-09-06T09:55:24.948320+09:00
tags: ["messaging", "event-driven", "architecture", "distributed-systems"]
---
## Overview

A message queue and an event log both move data from producers to consumers, but they differ in what happens after a message is read. A queue treats delivery as a one-time handoff where each message is <strong class="kw">consumed once</strong> and then removed, while an event log keeps every event in an ordered, <strong class="kw">replayable</strong> sequence that multiple independent readers can consume at their own pace. This distinction drives how each handles multiple consumers, failure recovery, and historical reprocessing.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5"/><text x="160" y="35" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Queue</text><text x="485" y="35" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Event Log</text><rect x="110" y="50" width="100" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">Producer</text><line x1="160" y1="80" x2="160" y2="98" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="155,98 165,98 160,106" style="fill:var(--content)"/><rect x="90" y="106" width="140" height="130" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="122" text-anchor="middle" font-size="10" style="fill:var(--secondary)">FIFO buffer</text><rect x="110" y="132" width="100" height="22" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="110" y="160" width="100" height="22" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="110" y="188" width="100" height="22" rx="3" stroke-dasharray="4,3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="236" x2="160" y2="254" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="155,254 165,254 160,262" style="fill:var(--content)"/><rect x="110" y="256" width="100" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="276" text-anchor="middle" font-size="12" style="fill:var(--content)">Consumer</text><text x="160" y="302" text-anchor="middle" font-size="10" style="fill:var(--secondary)">message deleted after ack</text><rect x="530" y="50" width="100" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="580" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">Producer</text><line x1="580" y1="80" x2="580" y2="98" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="575,98 585,98 580,106" style="fill:var(--content)"/><text x="485" y="98" text-anchor="middle" font-size="10" style="fill:var(--secondary)">append-only log (retained)</text><rect x="350" y="106" width="54" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="404" y="106" width="54" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="458" y="106" width="54" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="512" y="106" width="54" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="566" y="106" width="54" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="377" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">0</text><text x="431" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">1</text><text x="485" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">2</text><text x="539" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">3</text><text x="593" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">4</text><line x1="431" y1="198" x2="431" y2="150" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="426,150 436,150 431,142" style="fill:var(--content)"/><rect x="386" y="200" width="90" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="431" y="213" text-anchor="middle" font-size="10" style="fill:var(--content)">Consumer A</text><text x="431" y="225" text-anchor="middle" font-size="10" style="fill:var(--content)">(offset 1)</text><line x1="539" y1="198" x2="539" y2="150" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="534,150 544,150 539,142" style="fill:var(--content)"/><rect x="494" y="200" width="90" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="539" y="213" text-anchor="middle" font-size="10" style="fill:var(--content)">Consumer B</text><text x="539" y="225" text-anchor="middle" font-size="10" style="fill:var(--content)">(offset 3)</text><text x="485" y="252" text-anchor="middle" font-size="10" style="fill:var(--secondary)">each consumer tracks its own offset</text></svg>
</div>

## Comparison Table

| Aspect | Queue | Event Log |
| --- | --- | --- |
| Write path | Producer sends the message directly to the queue | Producer appends the event to the end of the log |
| Storage structure | Transient buffer of discrete messages | Append-only, ordered, persistent sequence |
| Consumption mechanic | Consumer pops/dequeues a message and acknowledges it | Consumer reads sequentially and tracks its own offset |
| Multiple consumers | Competing consumers — each message goes to only one consumer | Broadcast — every consumer group gets its own full copy of the stream |
| Retention after read | Message is deleted once acknowledged | Event stays retained per policy regardless of who has read it |
| Replay capability | Not possible without re-publishing the message | Trivial — reset the offset and re-read from any point |
| Ordering guarantee | FIFO within the queue, but no guaranteed order across consumers | Strict order guaranteed within a partition |
| Failure recovery | Failed messages route to a dead-letter queue for retry | Consumer resumes by rewinding to its last committed offset |

## Key Differences

- A queue's message disappears after <strong class="kw">acknowledgment</strong>; a log's event stays put
- Queues split work across <strong class="kw">competing consumers</strong>; logs broadcast to every consumer group
- Logs support full <strong class="kw">replay</strong> from any offset; queues generally don't
- Log ordering is <strong class="kw">partition</strong>-scoped and strict, while queue ordering is best-effort across consumers
- Queues excel at <strong class="kw">task distribution</strong>; logs excel at event sourcing and stream processing

## When to Use Each

**Queue**

- **Task distribution**: Spread discrete units of work across a pool of workers where each task should be handled exactly once.
- **Request buffering**: Smooth out bursts between a fast producer and a slower downstream service without needing history.
- **Simple decoupling**: Decouple two services with minimal operational overhead when neither replay nor multi-consumer fan-out is required.

**Event Log**

- **Event sourcing**: Rebuild application state by replaying the full history of events from the beginning of the log.
- **Multiple independent consumers**: Let several services — analytics, search indexing, notifications — each read the same stream at their own pace.
- **Stream processing pipelines**: Support windowed aggregations and joins that need to reprocess historical events.
