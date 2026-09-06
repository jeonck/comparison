---
title: "Consistency vs Availability: The CAP Theorem Tradeoff"
date: 2026-09-06T09:35:36.942956+09:00
tags: ["cap-theorem", "distributed-systems", "consistency", "availability"]
---
## Overview

When a distributed system suffers a network partition, it must choose between <strong class="kw">consistency</strong> (every node sees the same data, even if that means rejecting requests) and <strong class="kw">availability</strong> (every request gets a response, even if the data might be stale). This CAP theorem tradeoff shapes how databases behave under failure and directly affects correctness guarantees versus uptime.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="6,5"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="17" font-weight="bold">Consistency (CP)</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="17" font-weight="bold">Availability (AP)</text><rect x="100" y="44" width="120" height="36" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="67" text-anchor="middle" style="fill:var(--content)" font-size="13">Client</text><rect x="420" y="44" width="120" height="36" rx="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="67" text-anchor="middle" style="fill:var(--content)" font-size="13">Client</text><line x1="140" y1="80" x2="105" y2="140" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="105,140 112,131 118,138" style="fill:var(--compare-a)"/><line x1="175" y1="140" x2="185" y2="80" style="stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="4,3"/><polygon points="185,80 178,86 182,90" style="fill:var(--compare-a)"/><text x="210" y="105" text-anchor="middle" style="fill:var(--compare-a)" font-size="11">503 blocked</text><line x1="460" y1="80" x2="425" y2="140" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="425,140 432,131 438,138" style="fill:var(--compare-b)"/><line x1="495" y1="140" x2="505" y2="80" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="505,80 498,86 502,90" style="fill:var(--compare-b)"/><text x="530" y="105" text-anchor="middle" style="fill:var(--compare-b)" font-size="11">200 OK (stale)</text><rect x="60" y="140" width="100" height="50" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="170" text-anchor="middle" style="fill:var(--content)" font-size="13">Node A</text><rect x="200" y="140" width="100" height="50" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="170" text-anchor="middle" style="fill:var(--content)" font-size="13">Node B</text><path d="M180 135 L172 150 L188 165 L180 180 L172 195" style="stroke:var(--border);fill:none" stroke-width="2"/><text x="180" y="122" text-anchor="middle" style="fill:var(--secondary)" font-size="10">partition</text><rect x="380" y="140" width="100" height="50" rx="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="430" y="170" text-anchor="middle" style="fill:var(--content)" font-size="13">Node A</text><rect x="520" y="140" width="100" height="50" rx="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="570" y="170" text-anchor="middle" style="fill:var(--content)" font-size="13">Node B</text><path d="M500 135 L492 150 L508 165 L500 180 L492 195" style="stroke:var(--border);fill:none" stroke-width="2"/><text x="500" y="122" text-anchor="middle" style="fill:var(--secondary)" font-size="10">partition</text><text x="160" y="235" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Waits for quorum, rejects request</text><text x="160" y="260" text-anchor="middle" style="fill:var(--content)" font-size="12">Guarantee: no stale reads</text><text x="480" y="235" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Answers immediately from local data</text><text x="480" y="260" text-anchor="middle" style="fill:var(--content)" font-size="12">Guarantee: no downtime</text></svg>
</div>

## Comparison Table

| Aspect | Consistency (CP) | Availability (AP) |
| --- | --- | --- |
| Normal operation (no partition) | Behaves identically to any healthy cluster; all replicas agree | Behaves identically to any healthy cluster; all replicas agree |
| Behavior when a partition occurs | Nodes that cannot confirm quorum stop responding | All nodes keep responding regardless of quorum status |
| Write handling during partition | Writes are rejected or queued until enough replicas are reachable | Writes are accepted locally and replicated once the partition heals |
| Read handling during partition | Reads are blocked or errored if the latest value can't be confirmed | Reads are served from whatever local replica is reachable, even if stale |
| Client-facing failure mode | Client sees a timeout or explicit error (e.g. 503) | Client sees a successful response that may contain outdated data |
| Data guarantee provided | Linearizability - no two nodes ever disagree on current state | Liveness - the system always answers, correctness may lag |
| Recovery after partition heals | Resumes cleanly; no conflicting writes existed since they were blocked | Must reconcile diverging writes via vector clocks, LWW, or CRDTs |
| Representative systems | HBase, Zookeeper, MongoDB (default majority writes) | Cassandra, DynamoDB, Riak |

## Key Differences

- The tradeoff only bites during an actual <strong class="kw">network partition</strong> - outside of that, both behave the same.
- Consistency requires a <strong class="kw">quorum</strong> agreement before answering, which can mean refusing requests.
- Availability guarantees a response but risks returning <strong class="kw">stale data</strong> to the client.
- The choice determines whether you need a <strong class="kw">conflict resolution</strong> strategy for divergent writes after recovery.
- Many production databases offer <strong class="kw">tunable consistency</strong>, letting you pick per-operation rather than a single global stance.

## When to Use Each

**Consistency (CP)**

- **Financial transactions**: Double-spending or incorrect balances are worse than a temporarily rejected request.
- **Inventory and stock counts**: Overselling the same unit to two customers is a costlier error than brief unavailability.
- **Distributed locks and leader election**: Coordination services like Zookeeper must never let two nodes believe they hold the same lock.

**Availability (AP)**

- **Social media feeds**: Users tolerate seeing a slightly outdated post far more than they tolerate a broken page.
- **Shopping cart services**: Accepting an add-to-cart write even during a partition avoids losing a sale, as Amazon's Dynamo design chose.
- **IoT and sensor ingestion**: Continuous data collection matters more than perfect ordering across replicas.
- **Global CDN and edge caching**: Serving cached content during an outage keeps the site up even if it's briefly stale.
