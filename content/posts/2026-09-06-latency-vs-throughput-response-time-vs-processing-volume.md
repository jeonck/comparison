---
title: "Latency vs Throughput: Response Time vs Processing Volume"
date: 2026-09-06T09:34:39.801674+09:00
tags: ["latency", "throughput", "performance", "networking"]
---
## Overview

Latency and throughput are two orthogonal measures of system performance: <strong class="kw">latency</strong> is the time a single request takes to complete, while <strong class="kw">throughput</strong> is the volume of work a system finishes per unit of time. The distinction matters because architectures optimized for one can quietly degrade the other.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <text x="320" y="28" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Latency</text>
  <rect x="40" y="50" width="60" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="70" y="75" text-anchor="middle" font-size="12" style="fill:var(--content)">Client</text>
  <rect x="540" y="50" width="60" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="570" y="75" text-anchor="middle" font-size="12" style="fill:var(--content)">Server</text>
  <line x1="100" y1="70" x2="540" y2="70" style="stroke:var(--border)" stroke-width="2" stroke-dasharray="4 4"/>
  <circle cx="320" cy="70" r="9" style="fill:var(--compare-a);stroke:var(--compare-a)"/>
  <path d="M330,64 L342,70 L330,76 Z" style="fill:var(--compare-a)"/>
  <line x1="100" y1="120" x2="540" y2="120" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="100" y1="113" x2="100" y2="127" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="540" y1="113" x2="540" y2="127" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <path d="M100,120 L108,116 L108,124 Z" style="fill:var(--compare-a)"/>
  <path d="M540,120 L532,116 L532,124 Z" style="fill:var(--compare-a)"/>
  <text x="320" y="145" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Time for ONE request to complete</text>
  <line x1="20" y1="180" x2="620" y2="180" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="2 4"/>
  <text x="320" y="208" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Throughput</text>
  <rect x="40" y="225" width="60" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="70" y="250" text-anchor="middle" font-size="12" style="fill:var(--content)">Client</text>
  <rect x="540" y="225" width="60" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="570" y="250" text-anchor="middle" font-size="12" style="fill:var(--content)">Server</text>
  <rect x="100" y="235" width="440" height="20" rx="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <circle cx="140" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="200" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="260" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="320" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="380" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="440" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <circle cx="500" cy="245" r="6" style="fill:var(--compare-b);stroke:var(--compare-b)"/>
  <path d="M545,245 L557,239 L557,251 Z" style="fill:var(--compare-b)"/>
  <text x="320" y="300" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Total requests completed per second</text>
</svg>
</div>

## Comparison Table

| Aspect | Latency | Throughput |
| --- | --- | --- |
| Definition | Time elapsed for one request to travel and complete | Amount of work completed across all requests per unit time |
| What is measured | A single request's round trip or processing delay | Aggregate output of the system over an observation window |
| Unit of measurement | Milliseconds, microseconds, or seconds | Requests/sec, transactions/sec, or Mbps |
| Primary driver | Network round-trip time, serialization, and processing delay | Available bandwidth, parallel capacity, and resource pool size |
| Effect of concurrency | Individual request latency can rise as queueing builds up | Throughput rises with more parallel workers, up to a capacity limit |
| Behavior under overload | Tail latency spikes as queues grow (p95/p99 degrade) | Throughput plateaus or drops once the system saturates |
| Typical optimization | Reduce round trips, cache results, shorten the critical path | Batch requests, add parallel workers, scale out capacity |
| Measurement method | Ping, request timers, percentile latency (p50/p95/p99) | Requests-per-second counters, load testing, capacity benchmarks |

## Key Differences

- Latency measures the <strong class="kw">time</strong> for one request; throughput measures the <strong class="kw">volume</strong> processed per unit time.
- Batching to raise throughput can increase <strong class="kw">tail latency</strong> for individual requests.
- Latency is bounded by physical <strong class="kw">round-trip time</strong>; throughput is bounded by system <strong class="kw">capacity</strong>.
- Under heavy load, latency <strong class="kw">spikes</strong> from queueing while throughput <strong class="kw">plateaus</strong> at a ceiling.
- <strong class="kw">Little's Law</strong> links the two: average latency times concurrency roughly equals throughput.

## When to Use Each

**Latency**

- **Real-time interactive systems**: Gaming and video calls need low per-action latency for the interaction to feel responsive.
- **High-frequency trading**: Microsecond-level latency directly determines execution price and profitability.
- **User-facing API responses**: Perceived application responsiveness is driven by how fast a single request returns.

**Throughput**

- **Batch ETL pipelines**: Success is measured by total records processed per hour, not any single record's speed.
- **Bulk data transfer**: Large backups or file syncs care about total bytes moved per second, not per-packet delay.
- **Log and event ingestion**: Analytics pipelines need to sustain high sustained volume across many producers.
