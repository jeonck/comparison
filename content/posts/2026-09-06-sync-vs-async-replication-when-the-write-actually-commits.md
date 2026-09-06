---
title: "Sync vs Async Replication: When the Write Actually Commits"
date: 2026-09-06T09:39:49.229794+09:00
tags: ["database-replication", "distributed-systems", "consistency", "high-availability"]
---
## Overview

Synchronous and asynchronous replication differ in exactly one moment: when the primary tells the client a write succeeded. <strong class="kw">Sync replication</strong> waits for the replica to confirm before acknowledging, while <strong class="kw">async replication</strong> acknowledges immediately and copies the data afterward. That single timing difference cascades into everything else — latency, throughput, and how much data you can lose on failover.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-b)"/></marker></defs><text x="160" y="22" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Sync Replication</text><text x="480" y="22" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Async Replication</text><rect x="90" y="40" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="65" text-anchor="middle" style="fill:var(--content)" font-size="13">Client</text><rect x="90" y="150" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="175" text-anchor="middle" style="fill:var(--content)" font-size="13">Primary</text><rect x="90" y="270" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="295" text-anchor="middle" style="fill:var(--content)" font-size="13">Replica</text><line x1="150" y1="80" x2="150" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="60" y="115" style="fill:var(--secondary)" font-size="10">1. write</text><line x1="150" y1="190" x2="150" y2="270" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="60" y="235" style="fill:var(--secondary)" font-size="10">2. replicate</text><line x1="210" y1="270" x2="210" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="213" y="235" style="fill:var(--secondary)" font-size="10">3. ack</text><line x1="170" y1="150" x2="170" y2="80" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="173" y="115" style="fill:var(--secondary)" font-size="10">4. commit</text><text x="160" y="330" text-anchor="middle" style="fill:var(--content)" font-size="11">Client waits for step 3</text><rect x="410" y="40" width="140" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="65" text-anchor="middle" style="fill:var(--content)" font-size="13">Client</text><rect x="410" y="150" width="140" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="175" text-anchor="middle" style="fill:var(--content)" font-size="13">Primary</text><rect x="410" y="270" width="140" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="295" text-anchor="middle" style="fill:var(--content)" font-size="13">Replica</text><line x1="470" y1="80" x2="470" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="390" y="115" style="fill:var(--secondary)" font-size="10">1. write</text><line x1="500" y1="150" x2="500" y2="80" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="503" y="115" style="fill:var(--secondary)" font-size="10">2. commit</text><line x1="480" y1="190" x2="480" y2="270" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowB)"/><text x="390" y="235" style="fill:var(--secondary)" font-size="10">3. replicate later</text><text x="480" y="330" text-anchor="middle" style="fill:var(--content)" font-size="11">Client returns at step 2</text></svg>
</div>

## Comparison Table

| Aspect | Sync Replication | Async Replication |
| --- | --- | --- |
| Write acknowledgment | Waits for replica confirmation before committing | Commits on primary alone, replicates after |
| Commit latency | Includes network round-trip to replica | Bound only by primary's local write |
| Data consistency | Replica is always up to date at commit time | Replica can lag behind primary momentarily |
| Throughput under load | Degrades as replica distance or count grows | Unaffected by replica speed or distance |
| Replica or network failure | Writes block or fail until replica responds | Writes continue uninterrupted on primary |
| Failover data loss | None — replica always has the committed write | Possible — unreplicated writes are lost |
| Replication lag monitoring | Not applicable — lag is structurally zero | Critical — must track and alert on lag |

## Key Differences

- <strong class="kw">Commit timing</strong> is the root difference: sync waits, async doesn't
- Sync trades <strong class="kw">latency</strong> for a zero-data-loss guarantee on failover
- Async trades <strong class="kw">durability</strong> for consistently fast local commits
- Multi-region setups favor async since <strong class="kw">round-trip time</strong> would make sync commits too slow
- Async requires active <strong class="kw">lag monitoring</strong> that sync simply doesn't need

## When to Use Each

**Sync Replication**

- **Financial transactions**: Sync guarantees the replica has every committed write, so a primary failure never loses a confirmed balance change.
- **Same-datacenter replicas**: Low round-trip time keeps the sync latency penalty small enough to be worth the durability guarantee.
- **Regulatory durability requirements**: Compliance rules that mandate zero data loss on failover are only satisfiable with synchronous acknowledgment.

**Async Replication**

- **Cross-region disaster recovery**: Async avoids forcing every write to pay the cost of a continent-spanning round trip.
- **High-throughput write workloads**: Removing the replica wait lets the primary sustain far higher write rates.
- **Read replica scaling**: Read-only replicas serving cached or slightly-stale queries don't need every write instantly present.
