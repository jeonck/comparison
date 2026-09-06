---
title: "Single Model vs CQRS: One Data Model vs Split Read/Write Models"
date: 2026-09-06T09:53:08.881103+09:00
tags: ["cqrs", "architecture", "data-modeling", "system-design"]
---
## Overview

A <strong class="kw">single model</strong> uses one unified set of classes and one schema for both reading and writing data, while <strong class="kw">CQRS</strong> (Command Query Responsibility Segregation) splits the system into separate write models (commands) and read models (queries) that can use different schemas, storage, or even databases. The choice matters because it trades simplicity and consistency for scalability and query flexibility as read and write demands diverge.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="32" text-anchor="middle" font-size="16" style="fill:var(--primary)">Single Model</text><text x="480" y="32" text-anchor="middle" font-size="16" style="fill:var(--primary)">CQRS</text><rect x="60" y="60" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="105" y="85" text-anchor="middle" font-size="12" style="fill:var(--content)">Read Req</text><rect x="230" y="60" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="275" y="85" text-anchor="middle" font-size="12" style="fill:var(--content)">Write Req</text><path d="M150,80 L200,80" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA1)"/><path d="M230,80 L200,80" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA2)"/><rect x="140" y="140" width="140" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><text x="210" y="165" text-anchor="middle" font-size="13" style="fill:var(--primary)">Model</text><text x="210" y="183" text-anchor="middle" font-size="11" style="fill:var(--secondary)">reads + writes</text><path d="M200,100 L210,140" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="140" y="250" width="140" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="210" y="280" text-anchor="middle" font-size="12" style="fill:var(--content)">One DB / Schema</text><path d="M210,200 L210,250" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA3)"/><rect x="390" y="60" width="90" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="435" y="85" text-anchor="middle" font-size="12" style="fill:var(--content)">Query</text><rect x="520" y="60" width="90" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="565" y="85" text-anchor="middle" font-size="12" style="fill:var(--content)">Command</text><rect x="395" y="140" width="90" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="440" y="163" text-anchor="middle" font-size="12" style="fill:var(--primary)">Read Model</text><text x="440" y="180" text-anchor="middle" font-size="10" style="fill:var(--secondary)">optimized view</text><rect x="525" y="140" width="90" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="570" y="163" text-anchor="middle" font-size="12" style="fill:var(--primary)">Write Model</text><text x="570" y="180" text-anchor="middle" font-size="10" style="fill:var(--secondary)">domain logic</text><path d="M435,100 L440,140" style="stroke:var(--compare-b)" stroke-width="1.5"/><path d="M565,100 L570,140" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="395" y="260" width="90" height="45" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="440" y="287" text-anchor="middle" font-size="11" style="fill:var(--content)">Read Store</text><rect x="525" y="260" width="90" height="45" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="570" y="287" text-anchor="middle" font-size="11" style="fill:var(--content)">Write Store</text><path d="M440,195 L440,260" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB1)"/><path d="M570,195 L570,260" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB2)"/><path d="M525,282 C505,320 465,320 445,282" fill="none" style="stroke:var(--secondary)" stroke-width="1.2" stroke-dasharray="4,3" marker-end="url(#arrowB3)"/><text x="485" y="335" text-anchor="middle" font-size="10" style="fill:var(--secondary)">sync / events</text><defs><marker id="arrowA1" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowA2" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowA3" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB1" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-b)"/></marker><marker id="arrowB2" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-b)"/></marker><marker id="arrowB3" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--secondary)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Single Model | CQRS |
| --- | --- | --- |
| Request entry point | One API/service handles reads and writes through the same code path | Requests are split upfront into command handlers and query handlers |
| Data model shape | One set of classes/entities represents the domain for every operation | Separate write model (rich domain logic) and read model (denormalized, query-optimized) |
| Write path | Write validates and persists directly to the shared schema | Command handler validates, applies business rules, persists to the write store |
| Read path | Read queries the same schema writes use, often requiring joins | Query handler reads from a precomputed, often denormalized read store |
| Sync between models | Not applicable — there is only one model, so no sync is needed | Read store is updated via events or projections after each write, introducing lag |
| Consistency guarantee | Strong consistency by default since reads see writes immediately | Eventual consistency between write and read sides unless engineered otherwise |
| Scaling behavior | Read and write load scale together since they share infrastructure | Read and write sides scale independently to match different load profiles |
| Operational complexity | Low — one schema, one deployment, one mental model to maintain | Higher — multiple stores, projection/event pipelines, and eventual-consistency debugging |

## Key Differences

- Single model keeps one schema for everything; CQRS splits into a <strong class="kw">write model</strong> and a <strong class="kw">read model</strong>
- CQRS trades immediate consistency for <strong class="kw">eventual consistency</strong> via projections or events
- Single model is simpler to reason about; CQRS adds <strong class="kw">operational overhead</strong> from syncing multiple stores
- CQRS enables independent <strong class="kw">scaling</strong> of reads and writes; single model scales them together
- Query flexibility is higher in CQRS since read models can be shaped per <strong class="kw">use case</strong>

## When to Use Each

**Single Model**

- **CRUD-heavy applications**: Standard CRUD apps benefit from the simplicity of one model with no sync logic to maintain.
- **Small to mid-size teams**: A single model reduces cognitive load and infrastructure when a team lacks the bandwidth to manage eventual consistency.
- **Strong consistency requirements**: When reads must always reflect the latest write immediately, a single model avoids replication lag entirely.

**CQRS**

- **Read/write ratio imbalance**: When reads vastly outnumber writes (or vice versa), CQRS lets you scale and optimize each side independently.
- **Complex reporting needs**: Denormalized read models let you serve dashboards and search views without expensive joins on the write schema.
- **Event-sourced domains**: CQRS pairs naturally with event sourcing, where the write side appends events and read models are rebuilt as projections.
