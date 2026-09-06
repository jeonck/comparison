---
title: "Choreography vs Orchestration: Who Drives the Workflow"
date: 2026-09-06T09:56:16.218477+09:00
tags: ["microservices", "distributed-systems", "architecture", "event-driven"]
---
## Overview

Both patterns coordinate a multi-step business process across independent services, but they differ in where the coordination logic lives. In <strong class="kw">choreography</strong>, each service reacts to events and decides its own next move with no central brain; in <strong class="kw">orchestration</strong>, a dedicated controller tells every service what to do and in what order.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="150" y="30" text-anchor="middle" font-size="16" style="fill:var(--primary)">Choreography</text><text x="480" y="30" text-anchor="middle" font-size="16" style="fill:var(--primary)">Orchestration</text><rect x="50" y="140" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="164" text-anchor="middle" font-size="11" style="fill:var(--content)">Order Svc</text><rect x="150" y="60" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="84" text-anchor="middle" font-size="11" style="fill:var(--content)">Payment Svc</text><rect x="150" y="220" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="244" text-anchor="middle" font-size="11" style="fill:var(--content)">Shipping Svc</text><path d="M108,140 C125,112 138,100 150,90" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="140" y="105" font-size="9" style="fill:var(--secondary)">event</text><path d="M235,100 C262,150 262,180 235,222" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="248" y="165" font-size="9" style="fill:var(--secondary)">event</text><path d="M150,238 C110,220 90,200 95,180" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="90" y="215" font-size="9" style="fill:var(--secondary)">event</text><rect x="400" y="150" width="120" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="460" y="174" text-anchor="middle" font-size="11" style="fill:var(--primary)">Orchestrator</text><rect x="400" y="50" width="120" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="460" y="74" text-anchor="middle" font-size="11" style="fill:var(--content)">Payment Svc</text><rect x="340" y="230" width="100" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="254" text-anchor="middle" font-size="11" style="fill:var(--content)">Order Svc</text><rect x="480" y="230" width="100" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="254" text-anchor="middle" font-size="11" style="fill:var(--content)">Shipping Svc</text><line x1="450" y1="150" x2="450" y2="92" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="472" y1="92" x2="472" y2="150" style="stroke:var(--compare-b)" stroke-width="1" stroke-dasharray="3,3"/><line x1="420" y1="190" x2="400" y2="228" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="438" y1="228" x2="445" y2="192" style="stroke:var(--compare-b)" stroke-width="1" stroke-dasharray="3,3"/><line x1="500" y1="190" x2="520" y2="228" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="540" y1="228" x2="525" y2="192" style="stroke:var(--compare-b)" stroke-width="1" stroke-dasharray="3,3"/><text x="150" y="340" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no central controller</text><text x="480" y="340" text-anchor="middle" font-size="10" style="fill:var(--secondary)">commands out, responses back</text></svg>
</div>

## Comparison Table

| Aspect | Choreography | Orchestration |
| --- | --- | --- |
| Trigger | Any service publishes an event when something happens | A client or event calls the orchestrator to start the process |
| Coordination logic | Distributed across each service's event handlers | Centralized in one orchestrator component |
| Communication style | Asynchronous events broadcast to whoever is listening | Explicit commands and replies directed at specific services |
| Step sequencing | Emergent from chained event subscriptions | Explicitly defined as a workflow or state machine |
| Failure handling | Each service listens for failure events and compensates locally | Orchestrator detects failure and drives compensating transactions |
| Adding a new step | Add a listener; no existing service needs to change | Update the orchestrator's workflow definition |
| Observability | Hard to see the full process; requires distributed tracing | Process state is visible in one place, easy to audit |
| Coupling | Low coupling between services, higher coupling to event schema | Services decoupled from each other, but coupled to the orchestrator |

## Key Differences

- Choreography spreads decision-making across services via <strong class="kw">events</strong>; orchestration centralizes it in a single <strong class="kw">controller</strong>.
- Choreography scales extensibility easily but makes the overall process hard to <strong class="kw">trace</strong>.
- Orchestration makes the workflow explicit and easy to <strong class="kw">audit</strong>, at the cost of a single point of coordination.
- Compensation logic lives in each service under choreography, but is driven centrally under <strong class="kw">orchestration</strong>.
- Orchestration introduces a dependency on the <strong class="kw">orchestrator</strong> itself as new coupling, even as it decouples the services from each other.

## When to Use Each

**Choreography**

- **Simple event chains**: When steps are few and loosely related, letting services react to events avoids building extra infrastructure.
- **Independent team ownership**: Teams can evolve their service's event handling without coordinating changes to a shared workflow definition.
- **High scalability needs**: Removing a central coordinator avoids a bottleneck or single point of failure in high-throughput event pipelines.

**Orchestration**

- **Complex multi-step transactions**: A saga with many conditional branches and compensations is easier to reason about as an explicit workflow.
- **Need for visibility and auditing**: Regulated processes benefit from a single place showing the current state and history of each transaction.
- **Coordinated rollback logic**: When failures require carefully ordered compensating actions across services, a controller can sequence them reliably.
