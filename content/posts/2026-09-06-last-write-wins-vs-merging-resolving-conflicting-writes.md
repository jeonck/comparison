---
title: "Last-Write-Wins vs Merging: Resolving Conflicting Writes"
date: 2026-09-06T09:50:29.314020+09:00
tags: ["distributed-systems", "conflict-resolution", "replication", "crdt"]
---
## Overview

When two replicas accept concurrent writes to the same key, a system must reconcile them. <strong class="kw">Last-Write-Wins</strong> picks a single winner by timestamp and discards the rest, while <strong class="kw">Merging</strong> combines both writes into a new value using domain-specific or CRDT logic.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Last-Write-Wins</text><text x="480" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Merging</text><rect x="40" y="55" width="110" height="45" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Write A</text><text x="95" y="91" text-anchor="middle" style="fill:var(--secondary)" font-size="10">t=10, val=1</text><rect x="170" y="55" width="110" height="45" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="225" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Write B</text><text x="225" y="91" text-anchor="middle" style="fill:var(--secondary)" font-size="10">t=12, val=2</text><line x1="95" y1="100" x2="160" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="225" y1="100" x2="160" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="160" cy="155" r="5" style="fill:var(--compare-a)"/><line x1="160" y1="160" x2="160" y2="200" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="90" y="205" width="140" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><text x="160" y="227" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">val = 2</text><text x="160" y="243" text-anchor="middle" style="fill:var(--secondary)" font-size="10">(highest timestamp wins)</text><text x="160" y="280" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Write A silently discarded</text><rect x="360" y="55" width="110" height="45" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="415" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Write A</text><text x="415" y="91" text-anchor="middle" style="fill:var(--secondary)" font-size="10">t=10, val=1</text><rect x="490" y="55" width="110" height="45" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="545" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Write B</text><text x="545" y="91" text-anchor="middle" style="fill:var(--secondary)" font-size="10">t=12, val=2</text><line x1="415" y1="100" x2="480" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="545" y1="100" x2="480" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="480" cy="155" r="5" style="fill:var(--compare-b)"/><line x1="480" y1="160" x2="480" y2="200" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="405" y="205" width="150" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="480" y="227" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">{A:1, B:2}</text><text x="480" y="243" text-anchor="middle" style="fill:var(--secondary)" font-size="10">(both values combined)</text><text x="480" y="280" text-anchor="middle" style="fill:var(--secondary)" font-size="10">No data lost, app may reconcile</text></svg>
</div>

## Comparison Table

| Aspect | Last-Write-Wins | Merging |
| --- | --- | --- |
| Conflict trigger | Fires when two writes to the same key arrive with overlapping validity, regardless of content | Fires the same way, but treats both writes as valid inputs rather than competitors |
| Resolution mechanism | Compares timestamps (or version numbers) and keeps the highest one | Applies a merge function, CRDT join, or three-way diff to combine both values |
| Data/metadata required | A reliable clock or monotonic counter per write | Version vectors, causal history, or a semantically defined merge operation |
| Application involvement | None — resolution is automatic and content-agnostic | Requires the app or data structure to define what 'combining' means |
| Outcome for the losing write | Discarded entirely, no trace remains | Incorporated into the final merged state, nothing is dropped |
| Consistency guarantee | Deterministic convergence, but the winner may be arbitrary relative to causality | Deterministic convergence that also respects the semantics of both updates |
| Performance overhead | Minimal — a single comparison per conflict | Higher — merge logic, extra metadata, and sometimes multi-way comparisons |
| Failure mode | Silent data loss under clock skew or concurrent writes at the same timestamp | Unresolvable merge conflicts that surface to the application or user |

## Key Differences

- LWW resolves conflicts purely by comparing <strong class="kw">timestamps</strong>, keeping only one write.
- Merging combines concurrent writes using a <strong class="kw">merge function</strong> or CRDT join instead of picking a single winner.
- LWW can cause <strong class="kw">silent data loss</strong> when clocks skew or writes race within the same tick.
- Merging needs <strong class="kw">semantic knowledge</strong> of the data type to combine values correctly.
- LWW adds negligible overhead per write; merging trades that simplicity for <strong class="kw">correctness</strong> under concurrency.

## When to Use Each

**Last-Write-Wins**

- **Cache and Session State**: Losing a stale write is harmless when the data is ephemeral and quickly overwritten again.
- **High-throughput Key-Value Stores**: The cost of per-write merge logic isn't worth it when most keys never actually conflict.
- **Simple Last-Update Semantics**: When the business rule genuinely is 'most recent value wins', LWW implements it directly with no extra logic.

**Merging**

- **Collaborative Document Editing**: Concurrent edits from multiple users must all be preserved, not overwritten by whichever arrives last.
- **Shopping Cart Synchronization**: Items added on different devices while offline need to be unioned together, not have one device's additions dropped.
- **Distributed Counters and Sets**: CRDT-based merges let increments or set additions from every replica accumulate correctly on reconciliation.
