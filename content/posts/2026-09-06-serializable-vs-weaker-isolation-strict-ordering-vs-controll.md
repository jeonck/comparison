---
title: "Serializable vs Weaker Isolation: Strict Ordering vs Controlled Anomalies"
date: 2026-09-06T09:47:51.540854+09:00
tags: ["databases", "transactions", "concurrency-control", "acid"]
---
## Overview

Serializable isolation guarantees that concurrent transactions produce a result equivalent to some <strong class="kw">serial order</strong>, eliminating every possible race condition. Weaker isolation levels like Read Committed or Snapshot trade that guarantee for higher throughput, deliberately allowing certain <strong class="kw">read/write anomalies</strong> that application code must tolerate or guard against.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="600">Serializable</text><text x="480" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="600">Weaker Isolation</text><line x1="40" y1="180" x2="280" y2="180" style="stroke:var(--border)" stroke-width="1"/><line x1="360" y1="180" x2="600" y2="180" style="stroke:var(--border)" stroke-width="1"/><rect x="45" y="110" width="100" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="133" text-anchor="middle" style="fill:var(--content)" font-size="13">T1</text><rect x="175" y="110" width="100" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="225" y="133" text-anchor="middle" style="fill:var(--content)" font-size="13">T2</text><path d="M40 200 L280 200" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><text x="160" y="225" text-anchor="middle" style="fill:var(--secondary)" font-size="11">time (T1 fully precedes T2)</text><text x="160" y="260" text-anchor="middle" style="fill:var(--content)" font-size="12">No overlap in effect =</text><text x="160" y="278" text-anchor="middle" style="fill:var(--content)" font-size="12">equivalent to a serial run</text><rect x="45" y="100" width="140" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5" transform="translate(340,0)"/><text x="455" y="123" text-anchor="middle" style="fill:var(--content)" font-size="13">T1</text><rect x="115" y="144" width="140" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5" transform="translate(340,0)"/><text x="525" y="167" text-anchor="middle" style="fill:var(--content)" font-size="13">T2</text><rect x="455" y="100" width="45" height="80" style="fill:none;stroke:var(--compare-b)" stroke-width="1" stroke-dasharray="3 3"/><path d="M360 200 L600 200" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="480" y="225" text-anchor="middle" style="fill:var(--secondary)" font-size="11">time (T1 and T2 overlap)</text><text x="480" y="260" text-anchor="middle" style="fill:var(--content)" font-size="12">Overlap window =</text><text x="480" y="278" text-anchor="middle" style="fill:var(--content)" font-size="12">dirty/non-repeatable reads,</text><text x="480" y="296" text-anchor="middle" style="fill:var(--content)" font-size="12">phantoms, or write skew possible</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0 0 L8 4 L0 8 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0 0 L8 4 L0 8 z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Serializable | Weaker Isolation |
| --- | --- | --- |
| Isolation guarantee | Result equivalent to some serial execution of all transactions | Allows specific interleavings; guarantee varies by level (Read Committed, Repeatable Read, Snapshot) |
| Concurrency control | Full conflict serializability via strict two-phase locking, serializable snapshot isolation (SSI), or predicate locks | Row-level locks or MVCC snapshots that only block on narrower conflict sets (e.g. write-write) |
| Anomalies prevented | Dirty reads, non-repeatable reads, phantom reads, and write skew all eliminated | Only a subset prevented; phantoms and write skew commonly remain possible |
| Contention behavior | Higher rate of lock waits, deadlocks, or serialization-failure aborts under concurrent access | Readers rarely block writers (MVCC) or lower lock scope, so contention is reduced |
| Throughput impact | Lower throughput and higher latency as concurrency increases, especially with hot rows | Higher throughput and better scalability under contention |
| Application responsibility | Application can assume correctness; only needs to retry on serialization-failure errors | Application must reason about anomalies and add explicit checks (e.g. version columns, SELECT FOR UPDATE) |
| Conflict detection timing | Detected either at lock-acquisition time or at commit time (optimistic serializable schemes) | Detected only for the narrower conflicts the level covers, often just at commit for MVCC writes |
| Default in major databases | Rarely the default; must be explicitly requested (e.g. SET TRANSACTION ISOLATION LEVEL SERIALIZABLE) | Default in most systems out of the box (PostgreSQL/Oracle default to Read Committed, MySQL InnoDB to Repeatable Read) |

## Key Differences

- Serializable enforces true <strong class="kw">serial equivalence</strong>; weaker levels only rule out a defined subset of anomalies.
- Weaker isolation relies on <strong class="kw">MVCC snapshots</strong> or narrow locks, cutting contention compared to serializable's broader locking or SSI conflict tracking.
- Under Serializable, correctness bugs shift into <strong class="kw">retry loops</strong> on abort; under weaker levels, they shift into missed application-level checks.
- Write skew is the classic anomaly Serializable closes that even <strong class="kw">Snapshot Isolation</strong> leaves open.
- Choosing a level is a runtime trade-off between guaranteed correctness and <strong class="kw">throughput</strong>, not a one-time schema decision.

## When to Use Each

**Serializable**

- **Financial ledger transfers**: Serializable prevents write skew where two concurrent transactions each check a constraint (like combined balance) that only holds if they run one at a time.
- **Inventory reservation systems**: Guarantees that concurrent stock-decrement transactions can never oversell, since any interleaving is forced to behave like a serial run.
- **Regulatory or audit-critical logic**: When correctness proofs must hold under any concurrent schedule, serializable removes the need to enumerate specific anomaly scenarios.

**Weaker Isolation**

- **High-throughput read-heavy APIs**: Read Committed or Snapshot Isolation lets readers avoid blocking on writers, maximizing concurrency for typical CRUD workloads.
- **Analytics and reporting queries**: Snapshot Isolation gives a consistent point-in-time view without the lock contention serializable would impose on long-running scans.
- **Independent, non-conflicting writes**: When transactions rarely touch overlapping rows, a weaker level delivers near-identical correctness with far less locking overhead.
