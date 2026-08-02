---
title: "Spot Instances vs On-Demand Instances: When Cheap Compute Comes With Strings Attached"
date: 2026-08-03T06:25:19.298663+09:00
tags: ["cloud-computing", "aws", "cost-optimization"]
---
## Overview

Both are ways to rent compute capacity from a cloud provider, but they trade cost against reliability in opposite directions. <strong class="kw">Spot Instances</strong> tap into unused capacity at steep discounts but can be reclaimed with almost no notice, while <strong class="kw">On-Demand Instances</strong> cost more per hour in exchange for a guaranteed, uninterrupted slot.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="320" y="28" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Timeline of a running workload</text><text x="60" y="62" font-size="15" font-weight="600" style="fill:var(--compare-b)">On-Demand Instance</text><rect x="60" y="80" width="520" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="320" y="110" text-anchor="middle" font-size="13" style="fill:var(--content)">Reserved for you, no interruptions</text><text x="60" y="148" font-size="12" style="fill:var(--secondary)">Runs continuously until you stop it</text><line x1="60" y1="175" x2="580" y2="175" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,4"/><text x="60" y="212" font-size="15" font-weight="600" style="fill:var(--compare-a)">Spot Instance</text><rect x="60" y="230" width="180" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">Running</text><rect x="240" y="230" width="60" height="50" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,3"/><polygon points="270,203 280,220 260,220" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><text x="270" y="217" text-anchor="middle" font-size="9" style="fill:var(--compare-a)">!</text><text x="270" y="296" text-anchor="middle" font-size="11" style="fill:var(--secondary)">2-min warning</text><text x="270" y="310" text-anchor="middle" font-size="11" style="fill:var(--secondary)">then reclaimed</text><rect x="300" y="230" width="280" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="440" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">Resumes on new capacity</text><text x="60" y="334" font-size="12" style="fill:var(--secondary)">Up to 90% cheaper, but availability is never guaranteed</text></svg>
</div>

## Comparison Table

| Aspect | Spot Instances | On-Demand Instances |
| --- | --- | --- |
| Request & provisioning | Fulfilled only if provider has spare capacity at your bid price | Fulfilled immediately from reserved capacity pools |
| Capacity guarantee | None — provider can reclaim the instance at any time | Guaranteed for as long as you keep paying |
| Pricing model | Variable, set by real-time supply and demand for spare capacity | Fixed hourly rate published by the provider |
| Interruption behavior | Reclaimed with a short warning (e.g. ~2 minutes on AWS) | Never interrupted by the provider; you control shutdown |
| Cost predictability | Fluctuates; can spike or be revoked when demand rises | Stable and predictable, easy to forecast in a budget |
| Ideal workloads | Fault-tolerant, stateless, or checkpointable batch jobs | Stateful, latency-sensitive, or continuously running services |
| Termination control | Provider-initiated; your app must handle abrupt shutdown | User-initiated; you decide exactly when it stops |

## Key Differences

- Spot pricing floats with market demand and can be up to 90% cheaper than <strong class="kw">On-Demand</strong> rates
- Spot capacity is <strong class="kw">reclaimable</strong> at any time, typically with only a short warning window
- On-Demand gives a firm <strong class="kw">capacity guarantee</strong> that Spot never promises
- Workloads on Spot need to tolerate sudden termination or design for <strong class="kw">checkpointing</strong>
- On-Demand cost is fixed and predictable, while Spot cost is variable and market-driven

## When to Use Each

**Spot Instances**

- **Batch data processing**: Jobs like log aggregation or ETL can checkpoint progress and resume cheaply if a Spot instance is reclaimed.
- **CI/CD build fleets**: Short-lived, parallelizable build and test runners tolerate interruption without affecting production.
- **Large-scale simulations**: Distributed training or rendering jobs can lose individual nodes without losing the overall run.
- **Autoscaled stateless workers**: A fleet behind a load balancer can absorb individual node loss with no user-facing impact.

**On-Demand Instances**

- **Production databases**: Stateful services that can't tolerate abrupt shutdown need guaranteed, uninterrupted capacity.
- **Customer-facing APIs**: Latency-sensitive traffic requires instances that won't vanish mid-request.
- **Unpredictable spiky demand**: When you must scale up immediately regardless of spot market availability, On-Demand guarantees the capacity is there.
- **Short one-off tasks**: For a quick job where setup time to handle interruptions isn't worth it, On-Demand is simpler to reason about.
