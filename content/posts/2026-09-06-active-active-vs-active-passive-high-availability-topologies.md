---
title: "Active-Active vs Active-Passive: High-Availability Topologies Compared"
date: 2026-09-06T10:14:52.978680+09:00
tags: ["high-availability", "disaster-recovery", "failover", "load-balancing"]
---
## Overview

Both patterns keep a system running when a node fails, but they differ in whether every node is doing useful work all the time. <strong class="kw">Active-Active</strong> runs multiple nodes concurrently serving live traffic, while <strong class="kw">Active-Passive</strong> keeps a standby node idle until the primary fails. The choice affects utilization, cost, data consistency, and how much downtime you accept during failover.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Active-Active</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Active-Passive</text><circle cx="160" cy="58" r="10" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="160" y="62" text-anchor="middle" style="fill:var(--content)" font-size="11">C</text><rect x="130" y="92" width="60" height="28" rx="4" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="160" y="110" text-anchor="middle" style="fill:var(--content)" font-size="10">LB</text><line x1="160" y1="68" x2="160" y2="92" style="stroke:var(--content)" stroke-width="1.5"/><line x1="140" y1="120" x2="110" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="180" y1="120" x2="210" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="70" y="160" width="80" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="189" text-anchor="middle" style="fill:var(--content)" font-size="12">Node 1</text><rect x="170" y="160" width="80" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="210" y="189" text-anchor="middle" style="fill:var(--content)" font-size="12">Node 2</text><text x="160" y="235" text-anchor="middle" style="fill:var(--secondary)" font-size="11">both nodes serve live traffic</text><text x="160" y="250" text-anchor="middle" style="fill:var(--secondary)" font-size="11">failure of one: LB reroutes instantly</text><circle cx="480" cy="58" r="10" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="480" y="62" text-anchor="middle" style="fill:var(--content)" font-size="11">C</text><line x1="480" y1="68" x2="480" y2="158" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="158" width="80" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="187" text-anchor="middle" style="fill:var(--content)" font-size="12">Active</text><rect x="440" y="250" width="80" height="50" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="5 4"/><text x="480" y="279" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Standby</text><line x1="480" y1="208" x2="480" y2="250" style="stroke:var(--secondary)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="525" y="232" text-anchor="middle" style="fill:var(--secondary)" font-size="10">replication</text><text x="480" y="322" text-anchor="middle" style="fill:var(--secondary)" font-size="11">on failure: promote standby</text><text x="480" y="336" text-anchor="middle" style="fill:var(--secondary)" font-size="11">brief failover delay</text></svg>
</div>

## Comparison Table

| Aspect | Active-Active | Active-Passive |
| --- | --- | --- |
| Topology | All nodes are equal peers running the same workload | One primary node plus one or more idle standby nodes |
| Traffic routing | Load balancer distributes requests across every node | All requests go to the single active node |
| Resource utilization | Full capacity of every node used continuously | Standby capacity sits reserved but unused until needed |
| Failure detection | Health checks pull the unhealthy node out of the LB pool | Heartbeat or monitor detects primary is down |
| Failover behavior | Near-instant; surviving nodes absorb load with no promotion step | Standby must be promoted to primary, causing a brief outage |
| Data consistency | Requires conflict resolution or coordination across writable nodes | Single writer at a time keeps consistency simple |
| Cost efficiency | No idle capacity; you pay for what's used | Pay for standby capacity that mostly sits idle |
| Operational complexity | Higher: multi-master sync, conflict handling, split-brain risk | Lower: simple primary/standby roles, single write path |

## Key Differences

- <strong class="kw">Active-Active</strong> serves traffic from every node simultaneously; <strong class="kw">Active-Passive</strong> serves it from only one at a time
- Failover in Active-Active is near-instant since surviving nodes are already live, while Active-Passive needs a <strong class="kw">promotion</strong> step
- Active-Active fully utilizes hardware; Active-Passive leaves <strong class="kw">standby capacity</strong> idle as insurance
- Multi-writer setups need <strong class="kw">conflict resolution</strong>, whereas a single active writer avoids that complexity entirely

## When to Use Each

**Active-Active**

- **Global Low-Latency Reads**: Serving users from the nearest of several live regions requires every node to actually handle traffic, not sit idle.
- **High Throughput Scaling**: When one node's capacity isn't enough, Active-Active lets you add nodes that all contribute to serving load.
- **Zero-Downtime Failover Requirement**: Removing a failed node from an already-live pool avoids any promotion delay.

**Active-Passive**

- **Simple Stateful Database Failover**: A single writer avoids the multi-master conflict resolution that complicates active-active replication.
- **Cost-Sensitive Disaster Recovery**: A standby-only DR site can run on cheaper, smaller infrastructure since it's not serving live traffic.
- **Legacy Systems Without Multi-Master Support**: Applications that assume a single source of truth fit naturally into an active-passive model.
