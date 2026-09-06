---
title: "At-Least-Once vs Exactly-Once: Delivery Guarantees Compared"
date: 2026-09-06T09:59:45.980241+09:00
tags: ["messaging", "distributed-systems", "delivery-semantics", "idempotency"]
---
## Overview

Messaging and stream-processing systems must pick a delivery guarantee: does a message arrive at least one time (possibly more), or does its effect happen precisely once no matter how many retries occur? <strong class="kw">At-least-once</strong> favors simplicity and throughput by retrying until acknowledged, at the cost of possible duplicates; <strong class="kw">exactly-once</strong> layers on deduplication or transactional coordination so retries never produce a second effect. The choice matters because a duplicate side effect — a double charge, a double email, a double stock decrement — can be catastrophic or merely annoying depending on the domain.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="10" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="24" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">At-Least-Once</text><text x="480" y="24" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Exactly-Once</text><rect x="50" y="44" width="110" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="105" y="64" text-anchor="middle" style="fill:var(--content)" font-size="12">Producer</text><rect x="50" y="104" width="110" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="105" y="124" text-anchor="middle" style="fill:var(--content)" font-size="12">Broker</text><rect x="50" y="164" width="110" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="105" y="184" text-anchor="middle" style="fill:var(--content)" font-size="12">Consumer</text><rect x="30" y="246" width="90" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="75" y="267" text-anchor="middle" style="fill:var(--content)" font-size="10">Apply Effect</text><rect x="150" y="246" width="90" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="262" text-anchor="middle" style="fill:var(--content)" font-size="10">Apply Effect</text><text x="195" y="275" text-anchor="middle" style="fill:var(--secondary)" font-size="9">(duplicate!)</text><text x="112" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="10">processed twice</text><line x1="105" y1="76" x2="105" y2="102" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="105" y1="136" x2="105" y2="162" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="85" y1="196" x2="75" y2="244" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="125" y1="196" x2="190" y2="244" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><path d="M165,180 C 235,180 235,120 165,120" style="stroke:var(--compare-a);fill:none" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="230" y="145" text-anchor="middle" style="fill:var(--secondary)" font-size="9">retry</text><text x="230" y="156" text-anchor="middle" style="fill:var(--secondary)" font-size="9">(ack lost)</text><rect x="370" y="44" width="110" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="425" y="64" text-anchor="middle" style="fill:var(--content)" font-size="12">Producer</text><rect x="370" y="104" width="110" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="425" y="124" text-anchor="middle" style="fill:var(--content)" font-size="12">Broker</text><rect x="370" y="164" width="110" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="425" y="184" text-anchor="middle" style="fill:var(--content)" font-size="12">Consumer</text><rect x="370" y="224" width="110" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="425" y="242" text-anchor="middle" style="fill:var(--content)" font-size="10">Dedup Check (msg id)</text><rect x="385" y="280" width="80" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="425" y="302" text-anchor="middle" style="fill:var(--content)" font-size="10">Apply Effect</text><rect x="500" y="224" width="95" height="28" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="547" y="242" text-anchor="middle" style="fill:var(--secondary)" font-size="9">Discarded (dup)</text><text x="425" y="332" text-anchor="middle" style="fill:var(--secondary)" font-size="10">processed once</text><line x1="425" y1="76" x2="425" y2="102" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="425" y1="136" x2="425" y2="162" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="425" y1="196" x2="425" y2="222" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="425" y1="252" x2="425" y2="278" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="480" y1="238" x2="498" y2="238" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><path d="M485,180 C 555,180 555,120 485,120" style="stroke:var(--compare-b);fill:none" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="550" y="145" text-anchor="middle" style="fill:var(--secondary)" font-size="9">retry</text><text x="550" y="156" text-anchor="middle" style="fill:var(--secondary)" font-size="9">(transport)</text></svg>
</div>

## Comparison Table

| Aspect | At-Least-Once | Exactly-Once |
| --- | --- | --- |
| Delivery guarantee | Message is delivered one or more times | Message's effect is applied exactly one time |
| Acknowledgment & retry | Consumer acks after processing; missing ack triggers redelivery | Same retry mechanics underneath, but paired with a dedup/transaction layer |
| Crash/failure behavior | Redelivers unacknowledged messages, which can duplicate work | Recovers to a consistent state so re-processing never re-applies the effect |
| Duplicate occurrence | Duplicates are expected and can reach the application | Duplicates are detected and dropped before the effect is applied |
| Idempotency requirement | Consumer logic must be idempotent to be safe | Consumer can be non-idempotent; system guarantees single application |
| Implementation mechanism | Simple ack-and-retry loop, no extra state needed | Idempotency keys, dedup tables, or atomic offset+write transactions |
| Performance overhead | Low overhead, high throughput | Higher overhead from tracking IDs, transactions, or coordination |
| Typical use cases | Logging, metrics, notifications, telemetry | Payments, inventory decrements, order processing |

## Key Differences

- At-least-once guarantees delivery but allows <strong class="kw">duplicates</strong>; exactly-once guarantees a <strong class="kw">single effect</strong> even under retries.
- Exactly-once relies on <strong class="kw">idempotency keys</strong> or transactional dedup, not just retry logic.
- At-least-once pushes duplicate handling onto the <strong class="kw">consumer</strong>; exactly-once centralizes it in the <strong class="kw">broker</strong> or framework.
- True exactly-once across independent systems is rarely free without <strong class="kw">two-phase commit</strong> or a shared transaction log.
- At-least-once trades correctness for <strong class="kw">throughput</strong>; exactly-once trades throughput for correctness.

## When to Use Each

**At-Least-Once**

- **High-volume telemetry**: Occasional duplicate log lines or metrics are harmless and not worth the dedup overhead.
- **Notification delivery**: A rare duplicate email or push notification is a minor annoyance, not a business risk.
- **Simple streaming pipelines**: When downstream processing is naturally idempotent, retry-based delivery is simpler and faster to build.

**Exactly-Once**

- **Payment processing**: Charging a customer twice due to a retried message is a financial and trust failure that must be prevented.
- **Inventory adjustments**: Decrementing stock more than once from a duplicated order event corrupts inventory counts.
- **Stateful event-driven workflows**: Order or workflow state machines break if a duplicate event re-triggers a transition.
