---
title: "Replication vs Sharding: Copying Data vs Splitting Data"
date: 2026-09-06T09:39:10.599162+09:00
tags: ["database", "scalability", "distributed-systems"]
---
## Overview

Both are techniques for scaling a database beyond a single node, but they solve different problems: <strong class="kw">replication</strong> copies the entire dataset onto multiple nodes to boost availability and read capacity, while <strong class="kw">sharding</strong> splits the dataset into disjoint partitions across nodes to boost storage and write capacity. Large-scale systems typically use both together — sharding for horizontal scale, replication within each shard for durability.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/>
  <text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Replication</text>
  <text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Sharding</text>
  <rect x="100" y="50" width="120" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="160" y="72" text-anchor="middle" style="fill:var(--content)" font-size="12">Primary</text>
  <text x="160" y="88" text-anchor="middle" style="fill:var(--secondary)" font-size="10">A B C D</text>
  <line x1="140" y1="100" x2="105" y2="172" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="180" y1="100" x2="225" y2="172" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <rect x="50" y="175" width="110" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="105" y="197" text-anchor="middle" style="fill:var(--content)" font-size="11">Replica A</text>
  <text x="105" y="213" text-anchor="middle" style="fill:var(--secondary)" font-size="10">A B C D</text>
  <rect x="170" y="175" width="110" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="225" y="197" text-anchor="middle" style="fill:var(--content)" font-size="11">Replica B</text>
  <text x="225" y="213" text-anchor="middle" style="fill:var(--secondary)" font-size="10">A B C D</text>
  <text x="160" y="255" text-anchor="middle" style="fill:var(--secondary)" font-size="11">full dataset, copied</text>
  <text x="160" y="272" text-anchor="middle" style="fill:var(--secondary)" font-size="11">to every node</text>
  <rect x="420" y="50" width="120" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="72" text-anchor="middle" style="fill:var(--content)" font-size="12">Router</text>
  <text x="480" y="88" text-anchor="middle" style="fill:var(--secondary)" font-size="10">key lookup</text>
  <line x1="460" y1="100" x2="430" y2="172" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="500" y1="100" x2="550" y2="172" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <rect x="375" y="175" width="110" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="430" y="197" text-anchor="middle" style="fill:var(--content)" font-size="11">Shard 1</text>
  <text x="430" y="213" text-anchor="middle" style="fill:var(--secondary)" font-size="10">keys A-M</text>
  <rect x="495" y="175" width="110" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="550" y="197" text-anchor="middle" style="fill:var(--content)" font-size="11">Shard 2</text>
  <text x="550" y="213" text-anchor="middle" style="fill:var(--secondary)" font-size="10">keys N-Z</text>
  <text x="480" y="255" text-anchor="middle" style="fill:var(--secondary)" font-size="11">dataset split into</text>
  <text x="480" y="272" text-anchor="middle" style="fill:var(--secondary)" font-size="11">disjoint subsets</text>
</svg>
</div>

## Comparison Table

| Aspect | Replication | Sharding |
| --- | --- | --- |
| Primary goal | Increase availability and read capacity | Increase storage and write capacity |
| Data distribution | Full dataset copied to every node | Dataset split into disjoint partitions across nodes |
| Write path | Writes go to primary, then propagate to replicas | Writes routed to the single shard owning the key |
| Read path | Any replica (or primary) can serve any read | Read must be routed to the shard holding the key |
| Node failure impact | Data survives since other copies exist | That shard's data becomes unavailable unless also replicated |
| Consistency concern | Replication lag between primary and replicas | Cross-shard transactions and joins are hard to coordinate |
| Scaling ceiling | Bounded by primary's write throughput | Bounded by cross-shard coordination and key hotspots |
| Operational overhead | Failover and leader election | Shard key design, rebalancing, and resharding |

## Key Differences

- Replication duplicates the same data everywhere; sharding partitions it so each node holds only a slice
- Replication scales <strong class="kw">reads</strong> and durability; sharding scales <strong class="kw">writes</strong> and total storage
- Sharding introduces a routing layer that must know which shard owns a given key
- Losing a replica is harmless, but losing an unreplicated shard causes real <strong class="kw">data loss</strong>
- Production systems commonly combine both: shard for scale, replicate each shard for resilience

## When to Use Each

**Replication**

- **Read-heavy workloads**: Adding replicas lets you spread read traffic across many nodes without touching the data model.
- **High availability / failover**: A standby replica can be promoted immediately if the primary fails, with no data repartitioning needed.
- **Geographic read locality**: Placing replicas near users reduces read latency while all data stays consistent in structure.

**Sharding**

- **Dataset exceeds one node**: When data volume outgrows a single machine's disk or memory, sharding spreads it across many machines.
- **Write throughput bottleneck**: Partitioning writes across shards removes the single-primary write ceiling that replication can't fix.
- **Multi-tenant isolation**: Sharding by tenant or customer ID isolates noisy neighbors and simplifies per-tenant scaling or compliance.
