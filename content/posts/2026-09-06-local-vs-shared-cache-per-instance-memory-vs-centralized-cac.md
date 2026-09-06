---
title: "Local vs Shared Cache: Per-Instance Memory vs Centralized Cache Service"
date: 2026-09-06T09:43:40.862591+09:00
tags: ["caching", "distributed-systems", "performance", "architecture"]
---
## Overview

A <strong class="kw">local cache</strong> stores data in the memory of a single application process, giving the fastest possible reads but no visibility into what other instances hold. A <strong class="kw">shared cache</strong> lives in a separate service that every instance queries over the network, trading a bit of latency for one consistent view of cached data across the whole fleet. The choice shapes how you handle invalidation, scaling, and failure in a multi-instance deployment.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="24" text-anchor="middle" font-size="16" style="fill:var(--primary)">Local Cache</text><text x="480" y="24" text-anchor="middle" font-size="16" style="fill:var(--primary)">Shared Cache</text><line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><rect x="40" y="50" width="140" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="80" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 1</text><rect x="210" y="50" width="70" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="245" y="80" text-anchor="middle" font-size="12" style="fill:var(--content)">cache</text><line x1="180" y1="75" x2="210" y2="75" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="40" y="140" width="140" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="170" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 2</text><rect x="210" y="140" width="70" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="245" y="170" text-anchor="middle" font-size="12" style="fill:var(--content)">cache</text><line x1="180" y1="165" x2="210" y2="165" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="40" y="230" width="140" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 3</text><rect x="210" y="230" width="70" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="245" y="260" text-anchor="middle" font-size="12" style="fill:var(--content)">cache</text><line x1="180" y1="255" x2="210" y2="255" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="310" text-anchor="middle" font-size="12" style="fill:var(--secondary)">three separate copies, no sync</text><rect x="360" y="50" width="140" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="430" y="80" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 1</text><rect x="360" y="140" width="140" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="430" y="170" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 2</text><rect x="360" y="230" width="140" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="430" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">App instance 3</text><rect x="545" y="130" width="75" height="70" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="582" y="170" text-anchor="middle" font-size="12" style="fill:var(--content)">shared cache</text><line x1="500" y1="75" x2="545" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="500" y1="165" x2="545" y2="165" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="500" y1="255" x2="545" y2="180" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="320" text-anchor="middle" font-size="12" style="fill:var(--secondary)">one source, network hop</text></svg>
</div>

## Comparison Table

| Aspect | Local Cache | Shared Cache |
| --- | --- | --- |
| Storage location | In-process heap memory of the running application | External service (e.g. Redis, Memcached) reachable over the network |
| Read/write path | Direct memory access within the process, no serialization | Network round trip plus serialization/deserialization per call |
| Latency | Nanoseconds to low microseconds | Sub-millisecond to a few milliseconds depending on network |
| Consistency across instances | Each instance holds its own copy, values can diverge | Single copy visible identically to every instance |
| Invalidation | Must be broadcast to every instance individually (e.g. pub/sub) | Delete or update once, immediately visible to all readers |
| Capacity | Bounded by each process's own memory, duplicated per instance | Centralized pool sized independently of app instance count |
| Scaling and failure | Scales automatically with app instances but restarts always cold; no external dependency | Scales as its own tier; an outage or restart affects every consuming instance at once |

## Key Differences

- Local caches keep data in the same <strong class="kw">process memory</strong> as the app, so reads never leave the machine.
- Shared caches route every access through a <strong class="kw">network hop</strong> to a separate service.
- Only shared caches guarantee <strong class="kw">single source</strong> consistency across all instances.
- Local caches need explicit fan-out for <strong class="kw">invalidation</strong>; shared caches update once for everyone.
- A shared cache outage is a <strong class="kw">shared failure</strong>, while a local cache failure is isolated to one instance.

## When to Use Each

**Local Cache**

- **Hot, tiny lookup tables**: Reference data like enum labels or feature flags fits comfortably in memory and benefits from nanosecond access.
- **Per-request memoization**: Caching a value only needed for the lifetime of a single request avoids any network overhead entirely.
- **Minimizing infrastructure**: No extra service to deploy, monitor, or pay for when the working set is small and per-instance duplication is acceptable.

**Shared Cache**

- **Session or user state**: Data must be visible identically no matter which app instance handles the next request from that user.
- **Expensive shared computations**: Caching a costly query or API result once avoids redundant work across every instance in the fleet.
- **Large or fast-growing datasets**: A centralized service can be sized and scaled independently instead of being duplicated in every process's memory.
