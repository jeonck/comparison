---
title: "CQRS vs CRUD: Splitting Reads and Writes vs One Unified Model"
date: 2026-08-04T05:09:25.591235+09:00
tags: ["cqrs", "crud", "architecture", "api-design"]
---
## Overview

CRUD (Create, Read, Update, Delete) models an application around a <strong class="kw">single data model</strong> that handles both reads and writes through uniform operations. CQRS (Command Query Responsibility Segregation) instead <strong class="kw">splits reads and writes</strong> into separate paths, often with different models and stores optimized for each. The choice matters most as a system's read/write patterns diverge or its business logic grows too complex for a single model to represent cleanly.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker><marker id="arrowS" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--secondary)"/></marker></defs><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="160" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">CRUD</text><text x="480" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">CQRS</text><rect x="110" y="55" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="80" text-anchor="middle" style="fill:var(--content)" font-size="12">Client</text><line x1="160" y1="95" x2="160" y2="140" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)" marker-start="url(#arrowA)"/><rect x="110" y="140" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="165" text-anchor="middle" style="fill:var(--content)" font-size="12">Model / API</text><line x1="160" y1="180" x2="160" y2="225" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)" marker-start="url(#arrowA)"/><rect x="110" y="225" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="250" text-anchor="middle" style="fill:var(--content)" font-size="12">Database</text><text x="160" y="320" text-anchor="middle" style="fill:var(--secondary)" font-size="11">one path, one model</text><rect x="425" y="45" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="68" text-anchor="middle" style="fill:var(--content)" font-size="12">Client</text><line x1="450" y1="81" x2="400" y2="115" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="510" y1="81" x2="560" y2="115" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="345" y="115" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="138" text-anchor="middle" style="fill:var(--content)" font-size="11">Command</text><rect x="505" y="115" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="138" text-anchor="middle" style="fill:var(--content)" font-size="11">Query</text><line x1="400" y1="151" x2="400" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="560" y1="151" x2="560" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="345" y="185" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="208" text-anchor="middle" style="fill:var(--content)" font-size="11">Write Model</text><rect x="505" y="185" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="208" text-anchor="middle" style="fill:var(--content)" font-size="11">Read Model</text><line x1="400" y1="221" x2="400" y2="255" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="560" y1="221" x2="560" y2="255" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="345" y="255" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="278" text-anchor="middle" style="fill:var(--content)" font-size="11">Write DB</text><rect x="505" y="255" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="278" text-anchor="middle" style="fill:var(--content)" font-size="11">Read DB</text><line x1="455" y1="273" x2="505" y2="273" style="stroke:var(--secondary)" stroke-width="1.5" stroke-dasharray="3,3" marker-end="url(#arrowS)"/><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">sync via events</text><text x="480" y="320" text-anchor="middle" style="fill:var(--secondary)" font-size="11">split paths, split models</text></svg>
</div>

## Comparison Table

| Aspect | CRUD | CQRS |
| --- | --- | --- |
| Request handling | Single endpoint set (GET/POST/PUT/DELETE) hits one code path for both reads and writes | Requests split into distinct Command (write) and Query (read) channels with separate handlers |
| Data model | One model represents the entity for both reading and writing | Separate write model (domain/aggregate) and read model (denormalized view) per side |
| Storage | Single database or table serves both reads and writes | Optional separate stores per side, e.g. relational for writes, cache or search index for reads |
| Consistency | Strongly consistent by default since reads hit the same store just written to | Read side is often eventually consistent, synced from the write side via events |
| Business logic placement | Validation and rules scattered across create/update handlers | Rules concentrated in command handlers that enforce invariants before state changes |
| Scaling | Read and write load scale together since they share the same path | Read and write sides can be scaled and optimized independently |
| Implementation overhead | Minimal; straightforward to build, test, and reason about | Higher; requires sync mechanism and handling of eventual consistency |

## Key Differences

- CRUD uses a <strong class="kw">unified model</strong> for reads and writes; CQRS <strong class="kw">separates commands and queries</strong> into distinct paths
- CRUD read-after-write is immediate since storage is shared; CQRS's read side is often <strong class="kw">eventually consistent</strong>
- CRUD business logic lives in generic handlers; CQRS pushes rules into explicit <strong class="kw">command handlers</strong>
- CQRS allows <strong class="kw">independent scaling</strong> of reads and writes; CRUD scales both together
- CRUD is simpler to build; CQRS trades that simplicity for flexibility at the cost of <strong class="kw">architectural complexity</strong>

## When to Use Each

**CRUD**

- **Simple domain or admin tools**: Internal tools and basic CRUD apps where reads and writes are equally simple don't need separate models.
- **Small team or MVP**: A single model is faster to build, test, and onboard new engineers to.
- **Uniform read/write load**: When traffic patterns for reads and writes are similar, there's no scaling benefit to splitting them.

**CQRS**

- **High read/write asymmetry**: Dashboards or reporting views with heavy reads but infrequent writes benefit from a read model optimized separately.
- **Complex domain logic**: Rich business rules and invariants are easier to enforce through explicit command handlers than generic update endpoints.
- **Event sourcing systems**: CQRS pairs naturally with event sourcing, giving a clear audit trail of every state change.
- **Independent scaling needs**: Read replicas or caches can be scaled and tuned separately from the write database.
