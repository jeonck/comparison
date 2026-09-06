---
title: "Load Shedding vs Request Completion: Rejecting Early vs Finishing In-Flight Work"
date: 2026-09-06T10:14:11.496448+09:00
tags: ["load-shedding", "backpressure", "reliability", "distributed-systems"]
---
## Overview

When a service is overloaded, it must choose between two competing policies: <strong class="kw">load shedding</strong>, which rejects excess requests at the door before they consume resources, and <strong class="kw">request completion</strong>, which guarantees every admitted request runs to its natural end. The choice determines whether overload shows up as explicit client-visible rejections or as growing queues and degraded latency for everyone still being served.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="50" x2="320" y2="320" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Load Shedding</text><text x="480" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Request Completion</text><line x1="20" y1="80" x2="95" y2="80" style="stroke:var(--content)" stroke-width="1.5"/><line x1="20" y1="130" x2="95" y2="130" style="stroke:var(--content)" stroke-width="1.5"/><line x1="20" y1="180" x2="95" y2="180" style="stroke:var(--content)" stroke-width="1.5"/><line x1="20" y1="230" x2="95" y2="230" style="stroke:var(--content)" stroke-width="1.5"/><rect x="105" y="60" width="30" height="200" style="fill:none;stroke:var(--content)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="120" y="52" text-anchor="middle" style="fill:var(--content)" font-size="10">gate</text><line x1="135" y1="80" x2="200" y2="80" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="135" y1="130" x2="200" y2="130" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="200" y="65" width="90" height="90" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="245" y="115" text-anchor="middle" style="fill:var(--content)" font-size="12">Accepted</text><line x1="112" y1="173" x2="128" y2="187" style="stroke:var(--secondary)" stroke-width="2"/><line x1="112" y1="187" x2="128" y2="173" style="stroke:var(--secondary)" stroke-width="2"/><line x1="112" y1="223" x2="128" y2="237" style="stroke:var(--secondary)" stroke-width="2"/><line x1="112" y1="237" x2="128" y2="223" style="stroke:var(--secondary)" stroke-width="2"/><text x="150" y="210" style="fill:var(--secondary)" font-size="11">shed (503)</text><text x="160" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Rejects excess before admission</text><line x1="325" y1="100" x2="345" y2="100" style="stroke:var(--content)" stroke-width="1.5"/><line x1="325" y1="160" x2="345" y2="160" style="stroke:var(--content)" stroke-width="1.5"/><line x1="325" y1="220" x2="345" y2="220" style="stroke:var(--content)" stroke-width="1.5"/><rect x="345" y="70" width="70" height="180" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="380" y="62" text-anchor="middle" style="fill:var(--content)" font-size="10">admitted</text><line x1="415" y1="160" x2="440" y2="160" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="100" width="80" height="120" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="92" text-anchor="middle" style="fill:var(--content)" font-size="10">in-flight</text><line x1="520" y1="160" x2="540" y2="160" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="575" cy="160" r="28" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><polyline points="562,160 572,170 590,145" style="fill:none;stroke:var(--compare-b)" stroke-width="2.5"/><text x="575" y="122" text-anchor="middle" style="fill:var(--primary)" font-size="11">complete</text><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Every admitted request runs to completion</text></svg>
</div>

## Comparison Table

| Aspect | Load Shedding | Request Completion |
| --- | --- | --- |
| Lifecycle stage | Applied at admission, before a request enters processing | Applied after admission, to work already in flight |
| Trigger | Fires when queue depth, CPU, or latency crosses an overload threshold | Is the default behavior for any request that was accepted, regardless of load |
| Resource cost of the decision | Cheap — rejects with a fast, minimal-work response | Expensive — the request's resources are already committed and must be paid out |
| Client-facing outcome | Explicit rejection (e.g. HTTP 503), client must retry later | Eventual success or failure on the request's own merits, no artificial cutoff |
| Effect on accepted traffic | Protects tail latency for accepted requests by removing excess load | Risks rising tail latency and queueing as accepted work competes for the same resources |
| Prioritization | Can selectively drop low-priority or cheap-to-reject traffic first | Typically processes admitted work FIFO, with no mid-flight reordering |
| Failure mode if misapplied | Too aggressive shedding rejects healthy capacity and wastes headroom | No shedding at all leads to resource exhaustion and cascading failure |
| Implementation layer | Load balancer, API gateway, or admission-control middleware | Service handler or business logic that owns the request once accepted |

## Key Differences

- Load shedding acts at the front door, before <strong class="kw">admission</strong>; completion policy governs work already in-flight.
- Shedding trades a guaranteed rejection for protecting <strong class="kw">tail latency</strong> of everything else being served.
- Completing every accepted request avoids wasting <strong class="kw">sunk cost</strong> already spent, but risks resource exhaustion under sustained overload.
- Shedding can prioritize which traffic to drop, while completion is typically <strong class="kw">FIFO</strong> once work is admitted.
- Over-aggressive shedding causes false rejections; refusing to shed at all invites <strong class="kw">cascading failure</strong>.

## When to Use Each

**Load Shedding**

- **Sudden traffic spike**: Shedding protects core availability by rejecting excess requests during a flash crowd instead of letting everyone queue.
- **Multi-tenant API gateway**: Dropping or throttling low-priority tenants preserves SLA compliance for higher-priority ones.
- **Approaching resource exhaustion**: Shedding before CPU or memory saturates avoids a total outage caused by one more accepted request.

**Request Completion**

- **Financial or transactional operations**: Partial execution risks inconsistent state, so accepted transactions must run to completion once started.
- **Comfortable capacity headroom**: When demand is well within capacity there's no overload to shed against, so requests simply complete.
- **Already-admitted long-running jobs**: Abandoning work mid-way wastes resources already spent, so finishing is cheaper than shedding late.
