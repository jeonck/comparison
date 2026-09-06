---
title: "Normalization vs Denormalization: Split Tables vs Duplicated Data"
date: 2026-09-06T09:38:11.533551+09:00
tags: ["database-design", "normalization", "denormalization", "sql"]
---
## Overview

Normalization organizes data into separate, related tables to eliminate redundancy and protect <strong class="kw">integrity</strong>, while denormalization intentionally merges and duplicates data to boost <strong class="kw">read speed</strong>. The right choice depends on whether your workload is dominated by frequent writes or by heavy, complex reads.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Normalization</text><text x="480" y="36" text-anchor="middle" font-size="18" style="fill:var(--primary)">Denormalization</text><rect x="40" y="70" width="180" height="56" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="92" text-anchor="middle" font-size="12" style="fill:var(--content)">Users</text><text x="130" y="108" text-anchor="middle" font-size="10" style="fill:var(--secondary)">id, name</text><rect x="40" y="156" width="180" height="56" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="178" text-anchor="middle" font-size="12" style="fill:var(--content)">Orders</text><text x="130" y="194" text-anchor="middle" font-size="10" style="fill:var(--secondary)">id, user_id, product_id</text><rect x="40" y="242" width="180" height="56" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="264" text-anchor="middle" font-size="12" style="fill:var(--content)">Products</text><text x="130" y="280" text-anchor="middle" font-size="10" style="fill:var(--secondary)">id, name, price</text><line x1="130" y1="126" x2="130" y2="156" style="stroke:var(--compare-a)" stroke-width="1.5"/><path d="M 220 184 C 260 184 260 270 220 270" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="316" text-anchor="middle" font-size="11" style="fill:var(--secondary)">3 linked tables, zero duplication</text><rect x="360" y="70" width="240" height="228" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><line x1="360" y1="108" x2="600" y2="108" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="360" y1="146" x2="600" y2="146" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="360" y1="184" x2="600" y2="184" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="360" y1="222" x2="600" y2="222" style="stroke:var(--compare-b)" stroke-width="1"/><line x1="360" y1="260" x2="600" y2="260" style="stroke:var(--compare-b)" stroke-width="1"/><text x="375" y="92" font-size="11" style="fill:var(--primary)">order_id | customer | product</text><text x="375" y="130" font-size="11" style="fill:var(--content)">101 | Alice | Widget</text><text x="375" y="168" font-size="11" style="fill:var(--content)">102 | Alice | Gadget</text><text x="375" y="206" font-size="11" style="fill:var(--content)">103 | Bob | Widget</text><text x="375" y="244" font-size="11" style="fill:var(--content)">104 | Bob | Gizmo</text><text x="480" y="316" text-anchor="middle" font-size="11" style="fill:var(--secondary)">1 wide table, repeated values</text></svg>
</div>

## Comparison Table

| Aspect | Normalization | Denormalization |
| --- | --- | --- |
| Design goal | Eliminate redundancy by decomposing data into logical entities | Optimize for fast retrieval by pre-combining related data |
| Table structure | Many narrow, related tables linked by foreign keys | Fewer, wider tables that embed related data directly |
| Data redundancy | Minimal; each fact stored in exactly one place | Deliberate; the same fact may appear in many rows |
| Write operations | Single-row updates ripple correctly since data lives once | Updates must touch every duplicated copy or drift occurs |
| Read operations | Requires assembling data from multiple tables | Data is already co-located, so reads are direct |
| Joins needed | Frequent, often multi-table joins for common queries | Rare or none, since data is flattened in advance |
| Data integrity risk | Low; constraints enforce a single source of truth | Higher; duplicate copies can become inconsistent |
| Storage requirements | Compact, no duplicated values | Larger footprint due to stored redundancy |

## Key Differences

- Normalization removes <strong class="kw">redundancy</strong> by splitting data into related tables; denormalization reintroduces it deliberately for speed
- Normalized schemas need more <strong class="kw">joins</strong> at read time, while denormalized ones avoid them by pre-joining data
- Denormalization trades update simplicity for risk of <strong class="kw">anomalies</strong> when duplicated copies fall out of sync
- Normalization favors <strong class="kw">write-heavy</strong> transactional workloads; denormalization favors <strong class="kw">read-heavy</strong> analytical ones
- Storage cost is lower under normalization but query complexity is lower under denormalization

## When to Use Each

**Normalization**

- **Transactional (OLTP) systems**: Frequent inserts and updates benefit from data existing in exactly one place to keep it consistent
- **Financial or regulated data**: Strong integrity guarantees matter more than query speed when correctness has legal or monetary consequences
- **Evolving schemas**: Well-normalized tables are easier to extend without rewriting duplicated data across the database

**Denormalization**

- **Analytics and reporting (OLAP)**: Pre-joined, flattened tables let dashboards and reports scan data without expensive multi-table joins
- **Read-heavy public APIs**: Serving high-traffic reads directly from a wide table avoids repeated join overhead under load
- **Caching and materialized views**: Denormalized snapshots are a natural fit for precomputed results meant purely for fast lookup
