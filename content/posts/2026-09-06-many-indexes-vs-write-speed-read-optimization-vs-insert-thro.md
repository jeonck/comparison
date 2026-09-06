---
title: "Many Indexes vs Write Speed: Read Optimization vs Insert Throughput"
date: 2026-09-06T09:46:35.546284+09:00
tags: ["database-indexing", "write-performance", "query-optimization", "database-design"]
---
## Overview

Every <strong class="kw">index</strong> you add speeds up specific queries by letting the database jump straight to matching rows instead of scanning the whole table. But each one is a second (or third, or thirteenth) structure that must be updated on every insert, update, and delete, so piling on indexes steadily erodes <strong class="kw">write speed</strong>. The right balance depends on whether your workload is dominated by reads or writes.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="15" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="28" text-anchor="middle" font-size="16" style="fill:var(--primary)">Many Indexes</text><text x="480" y="28" text-anchor="middle" font-size="16" style="fill:var(--primary)">Few Indexes</text><rect x="125" y="48" width="70" height="34" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">WRITE</text><rect x="445" y="48" width="70" height="34" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">WRITE</text><line x1="160" y1="82" x2="43" y2="118" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="160" y1="82" x2="99" y2="118" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="160" y1="82" x2="155" y2="118" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="160" y1="82" x2="211" y2="118" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="160" y1="82" x2="267" y2="118" style="stroke:var(--compare-a)" stroke-width="1"/><rect x="19" y="118" width="48" height="26" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><rect x="75" y="118" width="48" height="26" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><rect x="131" y="118" width="48" height="26" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><rect x="187" y="118" width="48" height="26" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><rect x="243" y="118" width="48" height="26" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><text x="43" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="99" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="155" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="211" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="267" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="160" y="160" text-anchor="middle" font-size="10" style="fill:var(--secondary)">5 index updates per write</text><line x1="43" y1="144" x2="160" y2="180" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="99" y1="144" x2="160" y2="180" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="155" y1="144" x2="160" y2="180" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="211" y1="144" x2="160" y2="180" style="stroke:var(--compare-a)" stroke-width="1"/><line x1="267" y1="144" x2="160" y2="180" style="stroke:var(--compare-a)" stroke-width="1"/><rect x="90" y="180" width="140" height="32" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1.2"/><text x="160" y="200" text-anchor="middle" font-size="11" style="fill:var(--content)">DISK</text><text x="160" y="240" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Write latency</text><rect x="60" y="250" width="200" height="18" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.2"/><text x="160" y="290" text-anchor="middle" font-size="11" style="fill:var(--content)">Higher latency, faster reads</text><line x1="480" y1="82" x2="452" y2="118" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="480" y1="82" x2="508" y2="118" style="stroke:var(--compare-b)" stroke-width="1"/><rect x="428" y="118" width="48" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><rect x="484" y="118" width="48" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="452" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="508" y="135" text-anchor="middle" font-size="9" style="fill:var(--content)">IDX</text><text x="480" y="160" text-anchor="middle" font-size="10" style="fill:var(--secondary)">2 index updates per write</text><line x1="452" y1="144" x2="480" y2="180" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="508" y1="144" x2="480" y2="180" style="stroke:var(--compare-b)" stroke-width="1"/><rect x="410" y="180" width="140" height="32" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1.2"/><text x="480" y="200" text-anchor="middle" font-size="11" style="fill:var(--content)">DISK</text><text x="480" y="240" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Write latency</text><rect x="430" y="250" width="100" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="480" y="290" text-anchor="middle" font-size="11" style="fill:var(--content)">Lower latency, slower reads</text></svg>
</div>

## Comparison Table

| Aspect | Many Indexes | Few Indexes |
| --- | --- | --- |
| Write path | Every INSERT/UPDATE/DELETE also updates each index's structure | Writes mostly touch just the base table (or 1-2 indexes) |
| Structures updated per write | One update per index plus the table (N+1 operations) | Minimal: table plus a small, fixed set of index updates |
| Disk I/O per write | Extra page writes and WAL/journal entries for each index B-tree | Fewer page writes, smaller transaction log footprint |
| Write throughput & latency | Lower sustained throughput; each write costs more | Higher sustained throughput; commits return faster |
| Read/query performance | Fast lookups and filtering across many indexed columns | Slower queries on unindexed columns; more full scans |
| Storage footprint | Larger on-disk size from redundant index copies of data | Smaller footprint, closer to raw table size |
| Maintenance cost | Rebuilds, vacuums, and statistics updates scale with index count | Cheaper, faster maintenance windows |
| Best-fit workload | Read-heavy, query-diverse systems (reporting, OLAP) | Write-heavy, ingest-heavy systems (logging, OLTP, ETL) |

## Key Differences

- Every extra index adds a corresponding update on each <strong class="kw">write</strong> operation, not just at read time.
- Many indexes shrink <strong class="kw">query latency</strong> but inflate <strong class="kw">insert cost</strong> on the same table.
- Fewer indexes cut <strong class="kw">WAL volume</strong> and lock contention during heavy write bursts.
- Index count is a direct trade between <strong class="kw">read performance</strong> and <strong class="kw">write throughput</strong>, not a free win.
- <strong class="kw">Rebuild and vacuum</strong> overhead grows with every index the database has to maintain.

## When to Use Each

**Many Indexes**

- **Read-Heavy Reporting**: Dashboards and BI tools filter and sort on many different columns, so multiple indexes keep those queries fast.
- **Ad-hoc Analytics**: Analysts query unpredictable column combinations, and broad index coverage avoids costly full table scans.
- **OLAP Star Schemas**: Fact tables with many foreign-key joins rely on indexed keys to keep join performance acceptable.

**Few Indexes**

- **High-Volume Ingestion**: Event or log pipelines need to sustain thousands of inserts per second, and each index would slow every one down.
- **Bulk Load Windows**: ETL jobs often drop non-essential indexes before a load and rebuild them after to maximize insert speed.
- **Write-Heavy OLTP**: Transactional systems prioritize fast commit latency over supporting every possible query shape.
