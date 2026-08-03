---
title: "Event Sourcing vs State-Based Persistence: Storing Changes vs Storing Current State"
date: 2026-08-04T05:10:16.507910+09:00
tags: ["event-sourcing", "state-persistence", "cqrs", "data-modeling"]
---
## Overview

Event sourcing persists every change to an entity as an immutable, ordered <strong class="kw">event log</strong>, deriving current state by replaying that history. State-based persistence instead stores and overwrites a single <strong class="kw">current record</strong>, discarding prior values on each update. The choice affects auditability, storage growth, and how much replay/projection machinery you need to build.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Event Sourcing</text><rect x="60" y="52" width="200" height="32" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="72" text-anchor="middle" style="fill:var(--content)" font-size="11">AccountOpened</text><rect x="60" y="90" width="200" height="32" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="110" text-anchor="middle" style="fill:var(--content)" font-size="11">Deposited $100</text><rect x="60" y="128" width="200" height="32" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="148" text-anchor="middle" style="fill:var(--content)" font-size="11">Withdrew $30</text><rect x="60" y="166" width="200" height="32" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="186" text-anchor="middle" style="fill:var(--content)" font-size="11">Deposited $50</text><line x1="160" y1="198" x2="160" y2="238" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="178" y="222" style="fill:var(--secondary)" font-size="10">replay</text><rect x="85" y="246" width="150" height="34" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><text x="160" y="268" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">Balance: $120</text><text x="160" y="304" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Full history preserved</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">State-Based Persistence</text><rect x="400" y="56" width="160" height="32" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="480" y="76" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Balance: $0</text><line x1="480" y1="88" x2="480" y2="104" style="stroke:var(--border)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="106" width="160" height="32" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="480" y="126" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Balance: $100</text><line x1="480" y1="138" x2="480" y2="154" style="stroke:var(--border)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="156" width="160" height="32" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="480" y="176" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Balance: $70</text><line x1="480" y1="188" x2="480" y2="210" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="500" y="202" style="fill:var(--secondary)" font-size="10">overwrite</text><rect x="405" y="216" width="150" height="34" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="480" y="238" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">Balance: $120</text><text x="480" y="304" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Only latest state kept</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--border)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Event Sourcing | State-Based Persistence |
| --- | --- | --- |
| Write operation | Appends a new immutable event to the log; nothing is ever modified in place | Updates or overwrites the existing row/record with new values |
| Data representation | Sequence of domain events describing what happened | Single current snapshot of the entity's fields |
| Reading current state | Replay events from the start (or from a snapshot) to derive current state | Direct read of the stored row, no reconstruction needed |
| Concurrency conflicts | Detected by checking the expected event stream version before appending | Detected via optimistic locking (row version/timestamp) or DB-level locks |
| Historical/audit trail | Native and complete — every past state is reconstructable | Not retained by default; requires a separate audit log or triggers |
| Schema evolution | Old event versions must be upcast or handled explicitly by consumers | Existing rows are migrated or altered directly to the new shape |
| Storage growth | Grows unbounded with every change; needs snapshotting/compaction to stay fast | Stays roughly proportional to the number of entities, not their history |
| Query complexity | Ad-hoc queries need dedicated projections/read models built from the events | Supports direct ad-hoc SQL queries against the current data |

## Key Differences

- Event sourcing stores an append-only <strong class="kw">event log</strong>; state-based persistence stores only the <strong class="kw">current row</strong>
- Rebuilding state requires <strong class="kw">replay</strong> in event sourcing, versus a simple read in state-based systems
- Event sourcing gives a native <strong class="kw">audit trail</strong>; state-based needs bolt-on logging to recover history
- Concurrency is resolved via <strong class="kw">event versioning</strong> instead of row-level locking
- Storage grows unbounded without <strong class="kw">snapshots</strong> in event sourcing, while state-based storage stays compact

## When to Use Each

**Event Sourcing**

- **Financial Ledgers**: Regulatory and compliance needs demand an immutable, replayable record of every transaction, not just the latest balance.
- **Debugging Production Issues**: Replaying events lets you reconstruct the exact state of an entity at any past point in time.
- **CQRS Architectures**: The event stream naturally feeds multiple independent read-model projections tailored to different queries.

**State-Based Persistence**

- **CRUD Admin Panels**: Simple direct reads and writes against current data are sufficient and avoid the overhead of replay logic.
- **High-Write Simple Entities**: Avoids unbounded log growth for entities with frequent, low-value updates like counters or status flags.
- **Ad-hoc Reporting**: Analysts can run SQL directly against current-state tables without building and maintaining projections.
