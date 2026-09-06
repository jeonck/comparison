---
title: "Range vs Hash Partitioning: Ordered Splits vs Scattered Buckets"
date: 2026-09-06T09:41:37.071094+09:00
tags: ["database-partitioning", "sharding", "distributed-systems", "data-architecture"]
---
## Overview

Range and hash partitioning are two strategies for splitting a table's rows across multiple partitions or nodes based on a partition key. Range partitioning assigns rows to <strong class="kw">contiguous key intervals</strong> (like date ranges), preserving order for efficient range scans but risking uneven load. Hash partitioning runs the key through a <strong class="kw">hash function</strong> to scatter rows evenly, trading away ordering for balanced, predictable distribution.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">RANGE PARTITIONING</text><text x="480" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">HASH PARTITIONING</text><line x1="320" y1="10" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1"/><text x="160" y="45" text-anchor="middle" font-size="11" style="fill:var(--secondary)">incoming keys</text><text x="480" y="45" text-anchor="middle" font-size="11" style="fill:var(--secondary)">incoming keys</text><rect x="40" y="50" width="80" height="30" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="120" y="50" width="80" height="30" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="200" y="50" width="80" height="30" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="80" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">1-33</text><text x="160" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">34-66</text><text x="240" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">67-100</text><rect x="360" y="50" width="80" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="50" width="80" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="520" y="50" width="80" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">1-33</text><text x="480" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">34-66</text><text x="560" y="70" text-anchor="middle" font-size="11" style="fill:var(--content)">67-100</text><line x1="80" y1="80" x2="80" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="80" x2="160" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="240" y1="80" x2="240" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="400" y1="80" x2="560" y2="168" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="480" y1="80" x2="400" y2="168" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="560" y1="80" x2="480" y2="168" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="128" text-anchor="middle" font-size="10" style="fill:var(--secondary)">hash(key)</text><rect x="50" y="170" width="60" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="130" y="170" width="60" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="210" y="170" width="60" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="80" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P1</text><text x="160" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P2</text><text x="240" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P3</text><rect x="370" y="170" width="60" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="450" y="170" width="60" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="530" y="170" width="60" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P1</text><text x="480" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P2</text><text x="560" y="194" text-anchor="middle" font-size="12" style="fill:var(--content)">P3</text><text x="160" y="240" text-anchor="middle" font-size="12" style="fill:var(--content)">Ordered, contiguous ranges</text><text x="480" y="240" text-anchor="middle" font-size="12" style="fill:var(--content)">Scattered, uniform spread</text><text x="160" y="264" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Easy to extend: add a boundary</text><text x="480" y="264" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Costly to resize: rehash keys</text><text x="160" y="288" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Risk: skew on hot ranges</text><text x="480" y="288" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Risk: no range pruning</text></svg>
</div>

## Comparison Table

| Aspect | Range Partitioning | Hash Partitioning |
| --- | --- | --- |
| Partition key requirement | Needs an orderable key with defined boundaries (dates, IDs) | Any key works; only needs to be hashable |
| Row-to-partition mapping | Explicit boundary rules assign rows to intervals | Hash function output (often mod N) selects the bucket |
| Data distribution | Can be skewed if key values aren't uniformly spread | Near-uniform if the hash function distributes well |
| Range/scan queries | Prunes to only the partitions covering the range | Must fan out and scan every partition |
| Point/equality lookups | Requires a boundary search to find the right partition | Direct O(1) computation locates the partition |
| Adding or removing partitions | Cheap: append or split a boundary at the edge | Expensive: reshuffles most existing keys unless using consistent hashing |
| Hotspot behavior | Sequential writes (recent dates, auto-increment IDs) pile onto one partition | Spreads writes evenly but destroys any physical data locality |

## Key Differences

- Range partitioning preserves <strong class="kw">order</strong>, letting the query planner prune partitions; hash partitioning optimizes purely for <strong class="kw">even distribution</strong>
- Growing the partition count is a cheap boundary edit in range partitioning but forces a <strong class="kw">rehash</strong> of most keys in hash partitioning
- Range schemes are exposed to <strong class="kw">skew</strong> when writes cluster in a narrow key window; hash schemes avoid this at the cost of locality
- Point lookups under hashing are a direct <strong class="kw">hash computation</strong>, while range lookups need a <strong class="kw">boundary search</strong> through ordered intervals

## When to Use Each

**Range Partitioning**

- **Time-series or log data**: Partitioning by date range lets old partitions be dropped or archived wholesale and keeps recent-data queries fast.
- **Range-predicate heavy queries**: Queries like "orders between March and June" prune directly to the relevant partitions instead of scanning everything.
- **Ordered retention or purging**: Deleting expired data is a cheap drop-partition operation when partitions align with time or ID ranges.

**Hash Partitioning**

- **High-throughput writes**: Spreading inserts evenly across nodes avoids the single hot partition that sequential keys create under range partitioning.
- **No natural ordering key**: When rows have no meaningful sortable key (e.g., UUIDs), hashing still guarantees balanced placement.
- **Distributed key-value storage**: Systems needing predictable, uniform shard sizing favor hashing over range boundaries that require ongoing rebalancing.
