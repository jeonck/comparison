---
title: "Batch vs Stream Processing: Bounded Data Dumps vs Continuous Event Flow"
date: 2026-09-06T10:02:40.608236+09:00
tags: ["data-engineering", "batch-processing", "stream-processing", "system-design"]
---
## Overview

Batch processing collects data over a period and runs computation on the whole bounded set at once, while stream processing handles each event as it arrives, continuously. The choice determines whether your system optimizes for <strong class="kw">throughput and simplicity</strong> or <strong class="kw">low latency</strong> on fresh results.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Batch</text><text x="480" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Stream</text><rect x="40" y="60" width="240" height="90" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="95" text-anchor="middle" font-size="13" style="fill:var(--content)">Data accumulates</text><text x="160" y="115" text-anchor="middle" font-size="13" style="fill:var(--content)">into a bounded set</text><circle cx="70" cy="130" r="5" style="fill:var(--compare-a)"/><circle cx="110" cy="130" r="5" style="fill:var(--compare-a)"/><circle cx="150" cy="130" r="5" style="fill:var(--compare-a)"/><circle cx="190" cy="130" r="5" style="fill:var(--compare-a)"/><circle cx="230" cy="130" r="5" style="fill:var(--compare-a)"/><path d="M160 150 L160 190" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><rect x="80" y="195" width="160" height="55" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="217" text-anchor="middle" font-size="13" style="fill:var(--content)">Scheduled job</text><text x="160" y="235" text-anchor="middle" font-size="13" style="fill:var(--content)">processes all at once</text><path d="M160 250 L160 285" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><rect x="70" y="290" width="180" height="45" rx="4" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="160" y="317" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Result: high latency</text><rect x="400" y="60" width="200" height="280" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="85" text-anchor="middle" font-size="13" style="fill:var(--content)">Events flow continuously</text><circle cx="430" cy="115" r="5" style="fill:var(--compare-b)"/><path d="M430 115 L490 140" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="460" y="130" width="80" height="24" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><text x="500" y="146" text-anchor="middle" font-size="10" style="fill:var(--content)">process</text><circle cx="430" cy="175" r="5" style="fill:var(--compare-b)"/><path d="M430 175 L490 190" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="460" y="180" width="80" height="24" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><text x="500" y="196" text-anchor="middle" font-size="10" style="fill:var(--content)">process</text><circle cx="430" cy="235" r="5" style="fill:var(--compare-b)"/><path d="M430 235 L490 240" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="460" y="230" width="80" height="24" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><text x="500" y="246" text-anchor="middle" font-size="10" style="fill:var(--content)">process</text><circle cx="430" cy="290" r="5" style="fill:var(--compare-b)"/><path d="M430 290 L490 285" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="460" y="278" width="80" height="24" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><text x="500" y="294" text-anchor="middle" font-size="10" style="fill:var(--content)">process</text><text x="500" y="325" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Result: low latency</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Batch Processing | Stream Processing |
| --- | --- | --- |
| Data ingestion | Data collected and stored until job triggers | Events consumed individually as they arrive |
| Data scope per run | Bounded, finite dataset (a file, a partition, a day's data) | Unbounded, continuous sequence of events |
| Processing trigger | Scheduled interval or manual kickoff (hourly, nightly) | Continuous, triggered by each event or micro-window |
| Latency to result | Minutes to hours, depending on schedule | Milliseconds to seconds |
| State management | Recomputed fresh from full dataset each run | Maintained incrementally across the event stream |
| Fault recovery | Rerun the failed job against the same input | Checkpointing and replay from an offset in the log |
| Ordering guarantees | Whole dataset available, so ordering enforced within the job | Ordering must be explicitly handled (per-key, watermarks) |
| Resource usage pattern | Spiky: idle, then a burst of compute at run time | Steady, sustained compute and memory footprint |

## Key Differences

- Batch operates on a <strong class="kw">bounded dataset</strong>, stream operates on an <strong class="kw">unbounded sequence</strong> of events
- Batch trades latency for <strong class="kw">simplicity and throughput</strong>; stream trades complexity for <strong class="kw">freshness</strong>
- Stream systems need <strong class="kw">watermarks</strong> to handle late or out-of-order events, which batch avoids entirely
- Failure recovery in batch means <strong class="kw">rerunning the job</strong>; stream relies on <strong class="kw">checkpoint and replay</strong> semantics
- Batch pipelines are easier to reason about and test since input is <strong class="kw">fixed and reproducible</strong>

## When to Use Each

**Batch Processing**

- **Nightly reporting**: Aggregating a full day's transactions into summary tables is naturally bounded and doesn't need sub-second results.
- **Large-scale ETL**: Reprocessing historical data for schema migrations or backfills benefits from batch's simplicity and reproducibility.
- **Cost-sensitive workloads**: Running compute in scheduled bursts on cheap spot capacity is cheaper than keeping a stream cluster always on.

**Stream Processing**

- **Fraud detection**: Flagging a suspicious transaction needs a decision within seconds, not after the next batch window.
- **Real-time dashboards**: Live metrics like active users or system health require continuously updated aggregates.
- **Event-driven microservices**: Reacting to user actions or IoT sensor data as it happens keeps downstream systems in sync with minimal delay.
