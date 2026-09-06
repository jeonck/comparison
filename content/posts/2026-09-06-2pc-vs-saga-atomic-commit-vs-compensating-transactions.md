---
title: "2PC vs Saga: Atomic Commit vs Compensating Transactions"
date: 2026-09-06T09:57:51.068150+09:00
tags: ["distributed-transactions", "microservices", "saga-pattern", "two-phase-commit"]
---
## Overview

Both patterns coordinate a transaction that spans multiple services or databases, but they resolve the coordination problem in opposite ways. <strong class="kw">2PC</strong> locks every participant until a coordinator confirms all can commit, guaranteeing atomicity at the cost of blocking; <strong class="kw">Saga</strong> lets each step commit immediately and unwinds failures afterward with compensating actions, trading strict atomicity for availability and throughput.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0L10,5L0,10z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0L10,5L0,10z" style="fill:var(--compare-b)"/></marker></defs><text x="160" y="28" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">2PC</text><text x="480" y="28" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Saga</text><line x1="320" y1="40" x2="320" y2="340" stroke-width="1.5" stroke-dasharray="4,4" style="stroke:var(--border)"/><rect x="90" y="50" width="140" height="36" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="160" y="73" text-anchor="middle" font-size="13" style="fill:var(--content)">Coordinator</text><rect x="40" y="190" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="75" y="214" text-anchor="middle" font-size="12" style="fill:var(--content)">P1</text><rect x="130" y="190" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="165" y="214" text-anchor="middle" font-size="12" style="fill:var(--content)">P2</text><rect x="220" y="190" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="255" y="214" text-anchor="middle" font-size="12" style="fill:var(--content)">P3</text><line x1="140" y1="86" x2="80" y2="190" stroke-width="1.5" marker-start="url(#arrowA)" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><line x1="160" y1="86" x2="165" y2="190" stroke-width="1.5" marker-start="url(#arrowA)" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><line x1="180" y1="86" x2="250" y2="190" stroke-width="1.5" marker-start="url(#arrowA)" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="160" y="135" text-anchor="middle" font-size="11" style="fill:var(--secondary)">prepare / vote / commit</text><rect x="50" y="245" width="50" height="18" rx="3" stroke-width="1" style="fill:none;stroke:var(--border)"/><text x="75" y="258" text-anchor="middle" font-size="9" style="fill:var(--secondary)">locked</text><rect x="140" y="245" width="50" height="18" rx="3" stroke-width="1" style="fill:none;stroke:var(--border)"/><text x="165" y="258" text-anchor="middle" font-size="9" style="fill:var(--secondary)">locked</text><rect x="230" y="245" width="50" height="18" rx="3" stroke-width="1" style="fill:none;stroke:var(--border)"/><text x="255" y="258" text-anchor="middle" font-size="9" style="fill:var(--secondary)">locked</text><text x="160" y="300" text-anchor="middle" font-size="11" style="fill:var(--secondary)">all-or-nothing, blocking until ack</text><rect x="350" y="140" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="385" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">T1</text><rect x="440" y="140" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="475" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">T2</text><rect x="530" y="140" width="70" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="565" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">T3</text><line x1="420" y1="160" x2="440" y2="160" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><line x1="510" y1="160" x2="530" y2="160" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="480" y="130" text-anchor="middle" font-size="11" style="fill:var(--secondary)">local commit, local commit</text><line x1="530" y1="220" x2="440" y2="220" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><line x1="440" y1="240" x2="350" y2="240" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="480" y="212" text-anchor="middle" font-size="11" style="fill:var(--secondary)">compensate</text><text x="395" y="257" text-anchor="middle" font-size="11" style="fill:var(--secondary)">compensate</text><text x="480" y="300" text-anchor="middle" font-size="11" style="fill:var(--secondary)">independent commits, compensating rollbacks</text></svg>
</div>

## Comparison Table

| Aspect | 2PC | Saga |
| --- | --- | --- |
| Initiation | Coordinator sends a prepare request to all participants at once | First service runs its local transaction and triggers the next step |
| Commit decision | Coordinator waits for every vote, then issues a single atomic commit or abort | No central decision; each step commits independently as it finishes |
| Resource locking | Participants hold locks from prepare until the commit acknowledgment arrives | No cross-step locks; each local transaction commits and releases immediately |
| Failure handling | Coordinator aborts and tells all participants to roll back the uncommitted work | Already-committed steps are undone via explicit compensating transactions |
| Atomicity guarantee | True all-or-nothing atomicity across every participant | No real atomicity; intermediate states are visible until compensations finish |
| Coordination dependency | Single coordinator is a synchronous, blocking point of failure | Runs via choreography or a lightweight orchestrator that doesn't hold locks |
| Latency and throughput | Higher latency and lower throughput from synchronous cross-service locking | Lower per-step latency and higher throughput since nothing blocks across services |
| Implementation effort | Relies on XA-compliant resources and a transaction manager | Requires custom compensating logic and step/state tracking per operation |

## Key Differences

- 2PC provides true <strong class="kw">atomicity</strong> across services, while Saga only approximates it through compensations after the fact
- 2PC holds <strong class="kw">locks</strong> on every participant until the coordinator commits; Saga commits each step locally with no cross-service locking
- Saga failures require explicit <strong class="kw">compensating transactions</strong>; 2PC failures simply abort the still-uncommitted transaction
- 2PC depends on a synchronous <strong class="kw">coordinator</strong> that all participants must trust and wait on; Saga can run via choreography or a non-blocking orchestrator
- 2PC trades throughput for consistency, while Saga trades strict consistency for <strong class="kw">availability</strong> at scale

## When to Use Each

**2PC**

- **XA-compliant databases**: 2PC fits when all participants are traditional databases or resource managers that natively support the XA protocol and prepare/commit semantics
- **Short-lived, low-latency transactions**: Brief transactions among a small number of tightly coupled, co-located participants make the blocking window acceptable
- **Strict atomicity requirements**: Financial ledger or inventory-reservation systems where partial completion is unacceptable justify the cost of holding locks

**Saga**

- **Long-running microservice workflows**: Saga suits multi-step business processes like order fulfillment that span many independently deployed services over seconds or minutes
- **High availability and throughput needs**: Services can't afford to hold locks or block on a central coordinator under heavy concurrent load
- **Cross-organization or heterogeneous systems**: When participants don't share a transaction manager or XA support, compensating actions are the only practical rollback mechanism
