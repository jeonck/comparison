---
title: "Push vs Pull: Who Initiates the Data Transfer"
date: 2026-09-06T10:02:10.385870+09:00
tags: ["distributed-systems", "messaging", "architecture", "networking"]
---
## Overview

Push and pull describe which side initiates a data transfer between two systems: in a <strong class="kw">push</strong> model the source sends data as soon as it's ready, while in a <strong class="kw">pull</strong> model the consumer requests data on its own schedule. The choice shapes latency, backpressure handling, and how tightly the two sides are coupled in time.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Push</text><text x="480" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Pull</text><rect x="60" y="70" width="110" height="56" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="115" y="103" text-anchor="middle" font-size="13" style="fill:var(--content)">Source</text><rect x="60" y="200" width="110" height="56" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="115" y="233" text-anchor="middle" font-size="13" style="fill:var(--content)">Consumer</text><line x1="115" y1="126" x2="115" y2="200" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><text x="140" y="165" font-size="11" style="fill:var(--secondary)">sends data</text><text x="140" y="178" font-size="11" style="fill:var(--secondary)">when ready</text><circle cx="115" cy="85" r="4" style="fill:var(--compare-a)"/><rect x="380" y="70" width="110" height="56" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="435" y="103" text-anchor="middle" font-size="13" style="fill:var(--content)">Source</text><rect x="380" y="200" width="110" height="56" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="435" y="233" text-anchor="middle" font-size="13" style="fill:var(--content)">Consumer</text><line x1="435" y1="200" x2="435" y2="126" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="460" y="165" font-size="11" style="fill:var(--secondary)">requests data</text><text x="460" y="178" font-size="11" style="fill:var(--secondary)">on its schedule</text><line x1="120" y1="126" x2="120" y2="126" style="stroke:var(--border)"/><text x="115" y="280" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Source controls timing</text><text x="435" y="280" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Consumer controls timing</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Push | Pull |
| --- | --- | --- |
| Initiator | Source system triggers the transfer | Consumer system triggers the transfer |
| Timing control | Source decides when data is sent | Consumer decides when to fetch |
| Latency to consumer | Near-immediate once source has data | Bounded by polling interval, not source readiness |
| Backpressure handling | Source must slow down or buffer if consumer is overwhelmed | Consumer naturally paces itself by requesting only when ready |
| Coupling | Source needs to know consumer's address/endpoint | Consumer needs to know source's address/endpoint |
| Resource cost when idle | No wasted work; nothing sent if no updates | Repeated requests even when nothing changed |
| Failure handling | Source retries or queues if delivery fails | Consumer retries the pull on its own next cycle |
| Typical mechanisms | Webhooks, pub/sub, server-sent events | Polling, cron jobs, request/response APIs |

## Key Differences

- <strong class="kw">Push</strong> minimizes latency by sending data the instant it's available, while <strong class="kw">pull</strong> bounds latency to the polling interval.
- Pull gives the consumer natural <strong class="kw">backpressure</strong> control since it only asks for data when ready to process it.
- Push requires the source to hold a reference to every consumer's endpoint, increasing <strong class="kw">fan-out coupling</strong>.
- Pull wastes resources on <strong class="kw">empty polls</strong> when there's nothing new to fetch.
- Push systems need retry or queueing logic on the sender side; pull systems just retry the request on the next cycle.

## When to Use Each

**Push**

- **Real-time notifications**: Push delivers events like alerts or chat messages the instant they occur, avoiding polling delay.
- **High-volume fan-out**: A single event can be pushed to many subscribers via pub/sub without each one having to ask.
- **Webhooks between services**: Push lets an external service notify your app of state changes without you polling its API.

**Pull**

- **Rate-limited or metered APIs**: Pull lets the consumer control request timing to stay within quota and avoid overload.
- **Consumer with variable capacity**: Pull lets a slow or resource-constrained consumer fetch only when it has capacity to process.
- **Simple batch synchronization**: Pull-based polling on a schedule is easier to reason about and debug for periodic sync jobs.
