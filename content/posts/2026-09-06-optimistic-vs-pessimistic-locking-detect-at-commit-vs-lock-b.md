---
title: "Optimistic vs Pessimistic Locking: Detect-at-Commit vs Lock-Before-Access"
date: 2026-09-06T09:49:36.207771+09:00
tags: ["concurrency-control", "database", "transactions", "locking"]
---
## Overview

Both are concurrency-control strategies for preventing lost updates when multiple transactions touch the same data, but they differ in when they deal with conflict. Optimistic locking assumes collisions are rare and only performs a <strong class="kw">version check</strong> at commit time, while pessimistic locking assumes collisions are likely and takes an <strong class="kw">exclusive lock</strong> before any read or write proceeds.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <text x="160" y="28" font-size="18" font-weight="bold" text-anchor="middle" style="fill:var(--primary)">Optimistic</text>
  <text x="480" y="28" font-size="18" font-weight="bold" text-anchor="middle" style="fill:var(--primary)">Pessimistic</text>
  <line x1="320" y1="45" x2="320" y2="335" stroke-width="1" stroke-dasharray="4,4" style="stroke:var(--border)"/>
  <rect x="115" y="55" width="90" height="38" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/>
  <text x="160" y="79" font-size="13" text-anchor="middle" style="fill:var(--content)">Record (v1)</text>
  <text x="160" y="105" font-size="10" text-anchor="middle" style="fill:var(--secondary)">no lock taken</text>
  <line x1="140" y1="93" x2="75" y2="140" stroke-width="1.5" style="stroke:var(--compare-a)"/>
  <line x1="180" y1="93" x2="245" y2="140" stroke-width="1.5" style="stroke:var(--compare-a)"/>
  <rect x="30" y="140" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/>
  <text x="75" y="165" font-size="13" text-anchor="middle" style="fill:var(--content)">Txn A</text>
  <rect x="200" y="140" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/>
  <text x="245" y="165" font-size="13" text-anchor="middle" style="fill:var(--content)">Txn B</text>
  <line x1="75" y1="180" x2="75" y2="225" stroke-width="1.5" style="stroke:var(--compare-a)"/>
  <line x1="245" y1="180" x2="245" y2="225" stroke-width="1.5" stroke-dasharray="3,3" style="stroke:var(--border)"/>
  <rect x="30" y="225" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/>
  <text x="75" y="245" font-size="12" text-anchor="middle" style="fill:var(--content)">v1 &#8594; v2</text>
  <text x="75" y="259" font-size="10" text-anchor="middle" style="fill:var(--secondary)">committed</text>
  <rect x="200" y="225" width="90" height="40" rx="4" stroke-width="1.5" stroke-dasharray="3,3" style="fill:none;stroke:var(--border)"/>
  <text x="245" y="245" font-size="12" text-anchor="middle" style="fill:var(--content)">v1 &#8800; v2</text>
  <text x="245" y="259" font-size="10" text-anchor="middle" style="fill:var(--secondary)">conflict, retry</text>
  <text x="160" y="310" font-size="11" text-anchor="middle" style="fill:var(--secondary)">conflict caught at commit time</text>
  <rect x="470" y="34" width="20" height="14" rx="2" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/>
  <path d="M474,34 v-5 a6,6 0 0 1 12,0 v5" stroke-width="1.5" style="fill:none;stroke:var(--compare-b)"/>
  <rect x="435" y="55" width="90" height="38" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/>
  <text x="480" y="79" font-size="13" text-anchor="middle" style="fill:var(--content)">Record</text>
  <line x1="460" y1="93" x2="395" y2="140" stroke-width="1.5" style="stroke:var(--compare-b)"/>
  <line x1="500" y1="93" x2="565" y2="140" stroke-width="1.5" stroke-dasharray="3,3" style="stroke:var(--border)"/>
  <text x="545" y="120" font-size="10" text-anchor="middle" style="fill:var(--secondary)">blocked</text>
  <rect x="350" y="140" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/>
  <text x="395" y="157" font-size="13" text-anchor="middle" style="fill:var(--content)">Txn A</text>
  <text x="395" y="171" font-size="10" text-anchor="middle" style="fill:var(--secondary)">holds lock</text>
  <rect x="520" y="140" width="90" height="40" rx="4" stroke-width="1.5" stroke-dasharray="3,3" style="fill:none;stroke:var(--border)"/>
  <text x="565" y="157" font-size="13" text-anchor="middle" style="fill:var(--content)">Txn B</text>
  <text x="565" y="171" font-size="10" text-anchor="middle" style="fill:var(--secondary)">waiting</text>
  <line x1="395" y1="180" x2="395" y2="225" stroke-width="1.5" style="stroke:var(--compare-b)"/>
  <line x1="565" y1="180" x2="565" y2="225" stroke-width="1.5" style="stroke:var(--compare-b)"/>
  <rect x="350" y="225" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/>
  <text x="395" y="245" font-size="11" text-anchor="middle" style="fill:var(--content)">commits &amp;</text>
  <text x="395" y="259" font-size="10" text-anchor="middle" style="fill:var(--secondary)">releases lock</text>
  <rect x="520" y="225" width="90" height="40" rx="4" stroke-width="1.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/>
  <text x="565" y="245" font-size="11" text-anchor="middle" style="fill:var(--content)">acquires lock</text>
  <text x="565" y="259" font-size="10" text-anchor="middle" style="fill:var(--secondary)">then runs</text>
  <text x="480" y="310" font-size="11" text-anchor="middle" style="fill:var(--secondary)">conflict prevented up front</text>
</svg>
</div>

## Comparison Table

| Aspect | Optimistic Locking | Pessimistic Locking |
| --- | --- | --- |
| Access phase | No lock taken; any transaction can read or begin writing the row immediately | Lock acquired (e.g. SELECT FOR UPDATE) before the transaction reads or writes the row |
| Concurrent access | Other transactions freely read and write the same row in parallel | Other transactions attempting the same row must wait for the lock holder to finish |
| Conflict detection timing | Deferred until commit, via a version number, timestamp, or hash comparison | Not needed as a separate step; the lock physically prevents overlapping access |
| On conflict | Commit is rejected; the transaction is rolled back and typically retried | No conflict occurs; the waiting transaction simply blocks until the lock is released |
| Implementation mechanism | Application-level version column checked in the UPDATE's WHERE clause | Database-level row or table locks managed by the lock manager |
| Throughput under low contention | High; no blocking overhead when collisions are rare | Lower; locking overhead is paid even when no real conflict would occur |
| Behavior under high contention | Retry storms and wasted work as many transactions repeatedly fail and re-run | Orderly queueing keeps correctness but serializes work and limits parallelism |
| Deadlock risk | None, since no locks are ever held | Possible when transactions acquire multiple locks in inconsistent order |

## Key Differences

- Optimistic locking detects conflicts at <strong class="kw">commit time</strong>; pessimistic locking prevents them via <strong class="kw">upfront locking</strong>.
- Optimistic locking never blocks other transactions, it only forces a <strong class="kw">retry</strong> on collision.
- Pessimistic locking holds an <strong class="kw">exclusive lock</strong> for the transaction's full duration, serializing access to the row.
- Optimistic locking carries no <strong class="kw">deadlock</strong> risk because it never holds locks.
- Pessimistic locking trades raw throughput for <strong class="kw">predictability</strong> when contention is heavy.

## When to Use Each

**Optimistic Locking**

- **Read-heavy workloads**: Most operations only read data and true write collisions are rare, so avoiding lock overhead maximizes throughput.
- **Disconnected or long-lived clients**: Web/REST clients may take seconds between fetching and submitting an edit, making it impractical to hold a database lock that whole time.
- **Low contention on shared rows**: Independent edits, like separate users updating their own profile fields, rarely collide, so version-check retries stay rare.

**Pessimistic Locking**

- **High-contention hotspots**: Rows like inventory counters or account balances are hit by many concurrent transactions, where retries under optimistic locking would be constant.
- **Expensive-to-redo transactions**: When a rollback and retry would waste significant computation or external calls, blocking upfront is cheaper than repeating the work.
- **Multi-step critical sections**: When several dependent reads and writes must appear atomic to everyone else, holding a lock for the whole sequence avoids partial-state visibility.
