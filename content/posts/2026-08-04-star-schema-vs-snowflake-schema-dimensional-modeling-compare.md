---
title: "Star Schema vs Snowflake Schema: Dimensional Modeling Compared"
date: 2026-08-04T05:19:48.369214+09:00
tags: ["data-warehousing", "dimensional-modeling", "database-design", "olap"]
---
## Overview

Star schema and snowflake schema are two ways to structure dimension tables around a fact table in a data warehouse. Star schema keeps dimensions flat and <strong class="kw">denormalized</strong> for fast, simple joins, while snowflake schema splits dimensions into related sub-tables that are <strong class="kw">normalized</strong> to reduce redundancy. The choice trades query simplicity against storage efficiency and data integrity.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/>
  <text x="160" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Star Schema</text>
  <text x="480" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Snowflake Schema</text>
  <rect x="135" y="172" width="50" height="36" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="160" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Fact</text>
  <rect x="130" y="86" width="60" height="28" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="160" y="104" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="130" y="276" width="60" height="28" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="160" y="294" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="30" y="176" width="60" height="28" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="60" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="230" y="176" width="60" height="28" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="260" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <line x1="160" y1="172" x2="160" y2="114" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="160" y1="208" x2="160" y2="276" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="135" y1="190" x2="90" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <line x1="185" y1="190" x2="230" y2="190" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="160" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Dimensions denormalized, one hop to fact</text>
  <rect x="455" y="172" width="50" height="36" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Fact</text>
  <rect x="365" y="177" width="50" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="390" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="545" y="177" width="50" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="570" y="194" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <line x1="455" y1="190" x2="415" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="505" y1="190" x2="545" y2="190" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <rect x="450" y="127" width="60" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="144" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="450" y="57" width="60" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="74" text-anchor="middle" style="fill:var(--content)" font-size="11">Sub</text>
  <line x1="480" y1="172" x2="480" y2="153" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="480" y1="127" x2="480" y2="83" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <rect x="450" y="227" width="60" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="244" text-anchor="middle" style="fill:var(--content)" font-size="11">Dim</text>
  <rect x="450" y="287" width="60" height="26" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="304" text-anchor="middle" style="fill:var(--content)" font-size="11">Sub</text>
  <line x1="480" y1="208" x2="480" y2="227" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <line x1="480" y1="253" x2="480" y2="287" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="480" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Dimensions normalized into sub-tables</text>
</svg>
</div>

## Comparison Table

| Aspect | Star Schema | Snowflake Schema |
| --- | --- | --- |
| Dimension structure | Each dimension is a single flat table with all descriptive attributes together | Dimensions are split into multiple related tables organized by hierarchy level |
| Data redundancy | Attributes like category or region repeat across many rows within a dimension | Redundant attributes are moved into separate sub-tables and referenced by key |
| Join complexity per query | Fact table joins directly to each dimension, one hop per dimension | Queries often need multi-level joins through sub-dimension chains to reach an attribute |
| Query performance | Fewer joins generally mean faster scans and simpler execution plans | Extra joins across normalized levels typically add query latency and planning overhead |
| Storage footprint | Larger on disk due to repeated attribute values across rows | Smaller footprint since each attribute value is stored once and referenced |
| Update and integrity handling | Updating a shared attribute means touching many rows, risking inconsistency | Updating a shared attribute means changing one row in a sub-table, preserving integrity |
| ETL and load complexity | Simpler load logic since each dimension maps to one target table | More complex load logic to populate and link multiple normalized tables correctly |
| BI tool and end-user friendliness | Flat structure maps naturally to how most BI tools expect dimensions | Nested hierarchies can confuse drag-and-drop BI tools and require extra modeling |

## Key Differences

- Star schema keeps each dimension as one <strong class="kw">flat table</strong>; snowflake schema breaks dimensions into <strong class="kw">normalized</strong> sub-tables.
- Star schema favors fewer joins and faster <strong class="kw">read performance</strong>; snowflake schema favors lower <strong class="kw">storage redundancy</strong>.
- Snowflake schema reduces update anomalies by centralizing shared attributes, improving <strong class="kw">data integrity</strong>.
- Star schema is generally the default recommendation in <strong class="kw">Kimball-style</strong> dimensional modeling for BI workloads.
- Snowflake schema's extra joins increase <strong class="kw">query complexity</strong> for both engines and end users.

## When to Use Each

**Star Schema**

- **Interactive BI dashboards**: Flat dimensions minimize joins so dashboard queries return quickly under interactive load.
- **Self-service reporting tools**: Business users and drag-and-drop BI tools work more intuitively with single-table dimensions.
- **Small to medium dimension tables**: When redundancy costs little in storage, denormalization is worth the query simplicity.

**Snowflake Schema**

- **Large, deeply hierarchical dimensions**: Normalizing geography, product, or org hierarchies avoids massive redundant storage at scale.
- **Storage-cost-sensitive warehouses**: Eliminating repeated attribute values matters more when storage or memory is constrained.
- **Strict referential integrity needs**: Centralized attribute tables prevent inconsistent updates across a shared dimension.
- **Shared sub-dimensions across facts**: Normalized sub-tables can be reused by multiple fact tables without duplicating hierarchy data.
