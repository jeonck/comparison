---
title: "Horizontal Scaling vs Vertical Scaling: Growing Out vs Growing Up"
date: 2026-08-02T23:47:18.287826+09:00
tags: ["scalability", "infrastructure", "cloud", "system-design"]
---
## Overview

Horizontal and vertical scaling are the two fundamental strategies for adding capacity to a system: one adds <strong class="kw">more nodes</strong> working in parallel, the other adds <strong class="kw">more resources</strong> to a single existing node. The choice shapes cost, downtime, fault tolerance, and how much your application architecture has to change to support it.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="600">Horizontal Scaling</text><text x="480" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="600">Vertical Scaling</text><rect x="125" y="58" width="70" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="79" text-anchor="middle" style="fill:var(--content)" font-size="12">Load Balancer</text><line x1="160" y1="90" x2="70" y2="148" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="90" x2="145" y2="148" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="90" x2="220" y2="148" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="90" x2="295" y2="148" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="40" y="150" width="60" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="185" text-anchor="middle" style="fill:var(--content)" font-size="13">S1</text><rect x="115" y="150" width="60" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="145" y="185" text-anchor="middle" style="fill:var(--content)" font-size="13">S2</text><rect x="190" y="150" width="60" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="220" y="185" text-anchor="middle" style="fill:var(--content)" font-size="13">S3</text><rect x="265" y="150" width="60" height="60" rx="4" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="5,4"/><text x="295" y="185" text-anchor="middle" style="fill:var(--compare-a)" font-size="18">+</text><text x="160" y="245" text-anchor="middle" style="fill:var(--secondary)" font-size="12">scale out: add identical nodes</text><text x="160" y="262" text-anchor="middle" style="fill:var(--secondary)" font-size="12">no downtime, redundant</text><rect x="440" y="235" width="70" height="70" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="475" y="265" text-anchor="middle" style="fill:var(--content)" font-size="12">Server</text><text x="475" y="282" text-anchor="middle" style="fill:var(--content)" font-size="11">2 CPU / 4GB</text><line x1="475" y1="230" x2="475" y2="200" style="stroke:var(--compare-b)" stroke-width="2"/><polygon points="475,190 469,202 481,202" style="fill:var(--compare-b)"/><rect x="390" y="90" width="170" height="105" rx="4" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="5,4"/><text x="475" y="135" text-anchor="middle" style="fill:var(--content)" font-size="13">Same Server</text><text x="475" y="155" text-anchor="middle" style="fill:var(--content)" font-size="11">16 CPU / 64GB</text><text x="475" y="172" text-anchor="middle" style="fill:var(--secondary)" font-size="11">upgraded</text><text x="475" y="325" text-anchor="middle" style="fill:var(--secondary)" font-size="12">scale up: add CPU/RAM/disk</text><text x="475" y="342" text-anchor="middle" style="fill:var(--secondary)" font-size="12">often needs downtime</text></svg>
</div>

## Comparison Table

| Aspect | Horizontal Scaling | Vertical Scaling |
| --- | --- | --- |
| Mechanism | Add more machines/nodes to the pool | Add more CPU, RAM, or disk to an existing machine |
| Implementation | Requires a load balancer and clustering to distribute work | Swap hardware or resize the VM/instance in place |
| Downtime | Typically none; new nodes join the pool live | Usually requires a reboot or maintenance window |
| Application requirements | App must be stateless or handle distributed state | App can remain unaware, since it still runs on one node |
| Cost model | Roughly linear cost per added commodity node | Cost rises steeply at high-end hardware tiers |
| Fault tolerance | Redundant; a node failing doesn't take the system down | Single point of failure; that node failing is an outage |
| Capacity ceiling | Practically unbounded, add nodes as needed | Bounded by the largest machine/instance available |
| Typical use case | Web-scale services, microservices, cloud-native apps | Databases, legacy monoliths, short-term quick fixes |

## Key Differences

- Horizontal scaling adds <strong class="kw">more nodes</strong> in parallel, while vertical scaling adds <strong class="kw">more resources</strong> to one existing node.
- Horizontal scaling needs a <strong class="kw">load balancer</strong> and app-level statelessness; vertical scaling needs no architectural change.
- Vertical scaling eventually hits a <strong class="kw">hardware ceiling</strong>; horizontal scaling can grow near-limitlessly.
- Vertical scaling usually requires <strong class="kw">downtime</strong> to resize, while horizontal scaling can add capacity live.
- Horizontal scaling improves <strong class="kw">fault tolerance</strong> through redundancy; vertical scaling keeps a single point of failure.

## When to Use Each

**Horizontal Scaling** — Prefer horizontal scaling when you need <strong class="kw">high availability</strong> and elastic, near-unlimited growth for stateless or distributed workloads.

**Vertical Scaling** — Prefer vertical scaling for a <strong class="kw">quick capacity boost</strong> on a monolithic or stateful system, like a single database, where redesigning for distribution isn't worth it yet.
