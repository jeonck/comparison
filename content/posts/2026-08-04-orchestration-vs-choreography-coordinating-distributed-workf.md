---
title: "Orchestration vs Choreography: Coordinating Distributed Workflows"
date: 2026-08-04T05:11:21.372823+09:00
tags: ["orchestration", "choreography", "microservices", "event-driven-architecture"]
---
## Overview

Orchestration and choreography are two ways to coordinate multiple services in a distributed system or workflow. Orchestration relies on a <strong class="kw">central controller</strong> that explicitly directs each step, while choreography lets services independently respond to <strong class="kw">event reactions</strong> with no single component in charge. The choice shapes coupling, visibility, and how easily the workflow evolves over time.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Orchestration</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Choreography</text><rect x="110" y="150" width="100" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="180" text-anchor="middle" style="fill:var(--content)" font-size="12">Orchestrator</text><rect x="20" y="55" width="80" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="60" y="80" text-anchor="middle" style="fill:var(--content)" font-size="11">Service A</text><rect x="220" y="55" width="80" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="260" y="80" text-anchor="middle" style="fill:var(--content)" font-size="11">Service B</text><rect x="120" y="265" width="80" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="290" text-anchor="middle" style="fill:var(--content)" font-size="11">Service C</text><line x1="130" y1="150" x2="75" y2="97" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="190" y1="150" x2="245" y2="97" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="160" y1="200" x2="160" y2="263" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="160" y="330" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Central controller directs each step</text><rect x="440" y="55" width="90" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="485" y="80" text-anchor="middle" style="fill:var(--content)" font-size="11">Service X</text><rect x="350" y="220" width="90" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="395" y="245" text-anchor="middle" style="fill:var(--content)" font-size="11">Service Y</text><rect x="520" y="220" width="90" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="565" y="245" text-anchor="middle" style="fill:var(--content)" font-size="11">Service Z</text><line x1="510" y1="92" x2="555" y2="222" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="520" y1="240" x2="440" y2="240" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="410" y1="222" x2="465" y2="92" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="545" y="155" text-anchor="middle" style="fill:var(--secondary)" font-size="9">event</text><text x="480" y="255" text-anchor="middle" style="fill:var(--secondary)" font-size="9">event</text><text x="420" y="155" text-anchor="middle" style="fill:var(--secondary)" font-size="9">event</text><text x="480" y="330" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Services react to each other's events</text></svg>
</div>

## Comparison Table

| Aspect | Orchestration | Choreography |
| --- | --- | --- |
| Control flow | A central orchestrator explicitly invokes and sequences each step | Each service reacts to events independently; no component sequences the whole flow |
| Coupling | Services couple to the orchestrator's contract, not to each other | Services couple to shared event schemas rather than a controller |
| Workflow knowledge | Orchestrator holds the full picture; services only know their own task | No single component knows the entire workflow, only its trigger and reaction |
| Failure handling | Retries and compensation logic live centrally in the orchestrator | Compensation is distributed, with services reacting to failure events themselves |
| Adding new steps | Requires modifying the orchestrator to include the new call | A new service just subscribes to existing events with no central change |
| Observability | Single place to trace and inspect current workflow state | Requires aggregating distributed logs and traces across services |
| Failure/scaling risk | Orchestrator can become a bottleneck or single point of failure | Overall flow becomes hard to see, risking uncoordinated 'event sprawl' |
| Typical tooling | BPMN engines, AWS Step Functions, Temporal, Camunda | Message brokers and event buses like Kafka, SNS/SQS, domain events |

## Key Differences

- Orchestration uses a <strong class="kw">central controller</strong>; choreography relies on services reacting to <strong class="kw">events</strong>
- Orchestration couples services to the controller; choreography couples them to <strong class="kw">event contracts</strong>
- Extending orchestration means <strong class="kw">updating the orchestrator</strong>; extending choreography just means subscribing to events
- Orchestration gives <strong class="kw">centralized visibility</strong>; choreography requires <strong class="kw">distributed tracing</strong> to see the full flow
- Orchestration risks a bottleneck at the controller; choreography risks <strong class="kw">workflow sprawl</strong> across services

## When to Use Each

**Orchestration**

- **Complex multi-step sagas**: Explicit sequencing, branching logic, and compensation are easier to manage from one orchestrator.
- **Centralized monitoring needs**: A single controller gives one place to inspect and debug the current state of a long-running workflow.
- **Strict ordering or human tasks**: Workflows with conditional approvals or fixed ordering benefit from an explicit controller enforcing the sequence.

**Choreography**

- **Loosely coupled microservices**: Teams want to avoid a shared controller becoming a coupling point or single point of failure.
- **Simple reactive pipelines**: Each step naturally responds to the prior event without complex branching, so no controller is needed.
- **Independent team deployability**: Services can evolve and deploy autonomously without coordinating changes through a shared orchestrator.
