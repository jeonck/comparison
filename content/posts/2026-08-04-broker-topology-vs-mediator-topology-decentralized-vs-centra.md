---
title: "Broker Topology vs Mediator Topology: Decentralized vs Centralized Event Flow"
date: 2026-08-04T05:22:46.418775+09:00
tags: ["event-driven-architecture", "software-architecture", "messaging-patterns", "enterprise-integration"]
---
## Overview

Broker and mediator topology are the two core styles for structuring event-driven architectures, differing in who controls the flow of events between components. In a <strong class="kw">broker topology</strong>, events flow directly between producers and consumers through a distributed broker with no central authority, while a <strong class="kw">mediator topology</strong> routes every event through a central orchestrator that dictates the sequence of steps. The choice determines how easily the system scales versus how easily complex, stateful workflows can be managed and debugged.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="40" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="155" y="22" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="600">Broker Topology</text><rect x="105" y="32" width="100" height="26" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="49" text-anchor="middle" style="fill:var(--content)" font-size="10">Initiating Event</text><line x1="155" y1="58" x2="155" y2="88" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><circle cx="155" cy="112" r="24" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="116" text-anchor="middle" style="fill:var(--primary)" font-size="10">Broker</text><line x1="140" y1="132" x2="70" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="155" y1="136" x2="155" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="170" y1="132" x2="240" y2="168" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="30" y="170" width="80" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="188" text-anchor="middle" style="fill:var(--content)" font-size="9">Consumer A</text><rect x="115" y="170" width="80" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="188" text-anchor="middle" style="fill:var(--content)" font-size="9">Consumer B</text><rect x="200" y="170" width="80" height="28" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="240" y="188" text-anchor="middle" style="fill:var(--content)" font-size="9">Consumer C</text><line x1="240" y1="198" x2="240" y2="224" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><circle cx="240" cy="238" r="14" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="240" y="241" text-anchor="middle" style="fill:var(--primary)" font-size="8">Topic</text><line x1="240" y1="252" x2="240" y2="270" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="195" y="272" width="90" height="26" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="240" y="289" text-anchor="middle" style="fill:var(--content)" font-size="9">Consumer D</text><text x="155" y="320" text-anchor="middle" style="fill:var(--secondary)" font-size="9">No central control -</text><text x="155" y="333" text-anchor="middle" style="fill:var(--secondary)" font-size="9">events propagate independently</text><text x="485" y="22" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="600">Mediator Topology</text><rect x="440" y="32" width="100" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="490" y="49" text-anchor="middle" style="fill:var(--content)" font-size="10">Initiating Event</text><line x1="490" y1="58" x2="490" y2="90" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="92" width="170" height="46" rx="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="485" y="119" text-anchor="middle" style="fill:var(--primary)" font-size="11" font-weight="600">Mediator</text><line x1="440" y1="138" x2="390" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5" marker-start="url(#arrowB)" marker-end="url(#arrowB)"/><line x1="485" y1="138" x2="485" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5" marker-start="url(#arrowB)" marker-end="url(#arrowB)"/><line x1="530" y1="138" x2="580" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5" marker-start="url(#arrowB)" marker-end="url(#arrowB)"/><rect x="350" y="172" width="80" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="190" text-anchor="middle" style="fill:var(--content)" font-size="9">Step 1</text><rect x="445" y="172" width="80" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="485" y="190" text-anchor="middle" style="fill:var(--content)" font-size="9">Step 2</text><rect x="540" y="172" width="80" height="28" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="580" y="190" text-anchor="middle" style="fill:var(--content)" font-size="9">Step 3</text><text x="485" y="320" text-anchor="middle" style="fill:var(--secondary)" font-size="9">Mediator directs sequence</text><text x="485" y="333" text-anchor="middle" style="fill:var(--secondary)" font-size="9">and receives every result back</text></svg>
</div>

## Comparison Table

| Aspect | Broker Topology | Mediator Topology |
| --- | --- | --- |
| Event entry point | Producer publishes to a broker/topic; consumers subscribe independently | Producer sends the event directly to a central mediator component |
| Control flow ownership | No single owner - flow emerges from a chain of independent subscriptions | Mediator explicitly owns and dictates the order of processing steps |
| Component coupling | Producers and consumers are decoupled from each other and from any workflow logic | Processors are decoupled from each other but coupled to the mediator's contract |
| Workflow complexity support | Best suited to simple, single-step or loosely chained reactions | Built for complex, multi-step, conditional, or branching workflows |
| State and error management | Distributed across consumers, making end-to-end tracing harder | Centralized in the mediator, giving one place to track state and errors |
| Scalability | Scales horizontally with ease - add brokers or consumers freely | Mediator throughput can become a bottleneck as load grows |
| Failure impact | Broker or consumer outage only degrades that specific processing path | Mediator outage halts the entire orchestrated workflow |
| Adding or changing steps | Add a new consumer subscribing to an existing topic - no other code changes | Requires modifying the mediator's orchestration logic directly |

## Key Differences

- <strong class="kw">Control</strong> is implicit and distributed in broker topology versus explicit and centralized in mediator topology
- Broker topology favors <strong class="kw">scalability</strong>, while mediator topology favors <strong class="kw">workflow visibility</strong>
- Error handling is scattered across consumers in broker topology but consolidated in the <strong class="kw">mediator</strong>
- The mediator itself is a potential <strong class="kw">single point of failure</strong>, a risk broker topology avoids by design
- Changing a workflow means adding a subscriber in broker topology, but editing <strong class="kw">orchestration logic</strong> in mediator topology

## When to Use Each

**Broker Topology**

- **High-Volume Pub/Sub**: Broker topology scales horizontally with minimal coordination overhead, ideal for high-throughput event streams.
- **Independent Microservices**: Services that react to events without needing awareness of a larger workflow fit naturally into a decentralized broker model.
- **Simple Reactive Chains**: When one event triggers a small number of loosely related follow-up actions, a broker avoids unnecessary orchestration overhead.

**Mediator Topology**

- **Multi-Step Business Processes**: Mediator topology suits workflows with strict ordering, branching, or compensation logic, like order fulfillment or approval chains.
- **Centralized Error Recovery**: When failures need coordinated retries or rollback across multiple steps, a mediator provides the single control point to manage that.
- **Auditable Orchestration**: Regulated processes that require a clear, traceable record of execution order benefit from the mediator's explicit control flow.
