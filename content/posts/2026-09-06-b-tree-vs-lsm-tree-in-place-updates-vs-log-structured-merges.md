---
title: "B-Tree vs LSM Tree: In-Place Updates vs Log-Structured Merges"
date: 2026-09-06T09:42:56.024175+09:00
tags: ["databases", "data-structures", "storage-engines", "indexing"]
---
## Overview

B-Trees and LSM Trees are the two dominant on-disk index structures used by databases, and they diverge on how they handle writes. A B-Tree performs <strong class="kw">in-place updates</strong> on a balanced page structure to keep reads fast, while an LSM Tree buffers writes in memory and reconciles them later through <strong class="kw">background compaction</strong>, trading read simplicity for write throughput.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="24" text-anchor="middle" font-size="16" style="fill:var(--primary)">B-Tree</text><text x="480" y="24" text-anchor="middle" font-size="16" style="fill:var(--primary)">LSM Tree</text><rect x="120" y="40" width="80" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="59" text-anchor="middle" font-size="11" style="fill:var(--content)">root</text><rect x="60" y="110" width="70" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="129" text-anchor="middle" font-size="11" style="fill:var(--content)">node</text><rect x="180" y="110" width="70" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="215" y="129" text-anchor="middle" font-size="11" style="fill:var(--content)">node</text><rect x="15" y="180" width="55" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="42" y="198" text-anchor="middle" font-size="10" style="fill:var(--content)">leaf</text><rect x="80" y="180" width="55" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="107" y="198" text-anchor="middle" font-size="10" style="fill:var(--content)">leaf</text><rect x="165" y="180" width="55" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="192" y="198" text-anchor="middle" font-size="10" style="fill:var(--content)">leaf</text><rect x="230" y="180" width="55" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="257" y="198" text-anchor="middle" font-size="10" style="fill:var(--content)">leaf</text><line x1="160" y1="70" x2="95" y2="110" style="stroke:var(--border)" stroke-width="1.5"/><line x1="160" y1="70" x2="215" y2="110" style="stroke:var(--border)" stroke-width="1.5"/><line x1="95" y1="140" x2="42" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><line x1="95" y1="140" x2="107" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><line x1="215" y1="140" x2="192" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><line x1="215" y1="140" x2="257" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><line x1="107" y1="194" x2="150" y2="194" style="stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="3,2"/><text x="152" y="197" font-size="9" style="fill:var(--secondary)">update overwrites here</text><text x="160" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">in-place updates, balanced traversal</text><line x1="320" y1="30" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><defs><marker id="arrow" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-b)"/></marker></defs><rect x="400" y="45" width="160" height="34" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="66" text-anchor="middle" font-size="11" style="fill:var(--content)">memtable (in-memory)</text><line x1="480" y1="79" x2="480" y2="98" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrow)"/><rect x="400" y="100" width="160" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="118" text-anchor="middle" font-size="11" style="fill:var(--content)">L0 SSTables</text><line x1="480" y1="128" x2="480" y2="147" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrow)"/><rect x="400" y="149" width="160" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="167" text-anchor="middle" font-size="11" style="fill:var(--content)">L1 SSTables</text><line x1="480" y1="177" x2="480" y2="196" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrow)"/><rect x="400" y="198" width="160" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="216" text-anchor="middle" font-size="11" style="fill:var(--content)">L2 SSTables</text><text x="480" y="248" text-anchor="middle" font-size="10" style="fill:var(--secondary)">compaction merges levels</text><text x="480" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">sequential writes, background merge</text></svg>
</div>

## Comparison Table

| Aspect | B-Tree | LSM Tree |
| --- | --- | --- |
| Write path | Traverses the tree to locate the target page and updates it in place, splitting nodes as needed | Appends the entry to an in-memory memtable plus a write-ahead log; no seek to the record's final location |
| Read path | Single root-to-leaf traversal, O(log n) page reads from one location | Checks the memtable then potentially multiple SSTables across levels, often aided by bloom filters |
| Update/Delete handling | Overwrites the existing value directly at its page | Writes a new version or a tombstone; the old entry is only removed later during compaction |
| On-disk structure | One mutable, balanced tree of fixed-size pages, always sorted | Immutable sorted SSTable files organized into levels of increasing size |
| Background maintenance | Node splits and merges happen incrementally as part of each write | Periodic compaction merges SSTables across levels and drops stale versions |
| Write amplification | Low to moderate; occasional page rewrites and splits | Higher; the same record can be rewritten multiple times as it moves through levels |
| Read amplification & space reclaim | Minimal read amplification; deleted space is reclaimed immediately | Higher read amplification from scanning multiple levels; space reclaimed only after compaction |
| Range scans | Efficient via sorted leaf pages linked in order | Efficient within a level but requires merging sorted runs across levels |

## Key Differences

- A B-Tree updates data <strong class="kw">in place</strong>, while an LSM Tree defers changes through <strong class="kw">append-only</strong> writes to a memtable
- LSM Trees gain higher <strong class="kw">write throughput</strong> by avoiding random disk seeks, at the cost of ongoing <strong class="kw">compaction</strong>
- B-Trees give more predictable <strong class="kw">read latency</strong> since each key lives in exactly one place
- LSM read and space overhead comes from having to consult <strong class="kw">multiple SSTable levels</strong>
- Deletes in an LSM Tree are recorded as <strong class="kw">tombstones</strong> rather than removed immediately

## When to Use Each

**B-Tree**

- **Read-heavy OLTP workloads**: A single predictable traversal path keeps lookup latency low and consistent when reads dominate.
- **Sorted range queries**: Linked leaf pages let the tree scan ordered ranges without merging multiple data sources.
- **Frequent small in-place updates**: Overwriting a value directly avoids the version bookkeeping and later cleanup that log-structured designs require.

**LSM Tree**

- **Write-heavy ingestion**: Buffering writes in memory and flushing sequentially avoids the random I/O cost that dominates B-Tree writes.
- **SSD-optimized storage engines**: Sequential SSTable writes reduce write amplification on flash compared to scattered in-place page updates.
- **Systems that can tolerate compaction pauses**: Workloads like time-series or log storage can trade occasional compaction overhead for sustained write speed, as in Cassandra or RocksDB.
