---
title: "Monitoring vs Observability: Knowing Something's Wrong vs Knowing Why"
date: 2026-08-02T08:34:10.522117+09:00
tags: ["monitoring", "observability", "sre", "distributed-systems"]
---
## Overview

Monitoring watches a predefined set of metrics, logs, and checks against known failure modes and alerts you when thresholds are breached. Observability is a property of a system built so that its internal state can be inferred from its external outputs, letting you investigate questions you didn't think to ask in advance. The distinction matters because monitoring answers 'is something wrong?' while observability answers 'why is it wrong?' for failures you've never seen before.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Monitoring</text><text x="480" y="36" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Observability</text><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><rect x="60" y="60" width="200" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="83" text-anchor="middle" font-size="13" style="fill:var(--content)">CPU %</text><rect x="60" y="106" width="200" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="129" text-anchor="middle" font-size="13" style="fill:var(--content)">Error rate</text><rect x="60" y="152" width="200" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="175" text-anchor="middle" font-size="13" style="fill:var(--content)">Latency p95</text><path d="M160,196 L160,225" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="85" y="228" width="150" height="34" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="250" text-anchor="middle" font-size="12" style="fill:var(--content)">Threshold check</text><path d="M160,262 L160,288" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="95" y="290" width="130" height="34" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="312" text-anchor="middle" font-size="12" font-weight="bold" style="fill:var(--content)">Alert fired</text><text x="160" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">known failure modes only</text><rect x="380" y="60" width="200" height="110" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="80" text-anchor="middle" font-size="13" font-weight="bold" style="fill:var(--content)">Traces + logs + metrics</text><line x1="395" y1="92" x2="565" y2="92" style="stroke:var(--border)" stroke-width="1"/><text x="480" y="110" text-anchor="middle" font-size="12" style="fill:var(--content)">high-cardinality events</text><text x="480" y="130" text-anchor="middle" font-size="12" style="fill:var(--content)">request_id, user_id,</text><text x="480" y="148" text-anchor="middle" font-size="12" style="fill:var(--content)">shard, build, region...</text><path d="M480,170 L480,196" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="198" width="160" height="34" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="220" text-anchor="middle" font-size="12" style="fill:var(--content)">Arbitrary query</text><path d="M480,232 L480,258" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="260" width="160" height="34" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="282" text-anchor="middle" font-size="12" font-weight="bold" style="fill:var(--content)">Root cause found</text><text x="480" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">answers novel questions</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Monitoring | Observability |
| --- | --- | --- |
| Instrumentation setup | Dashboards and checks built around metrics you predefine (CPU, memory, error rate, latency) | Structured, high-cardinality telemetry (traces, structured logs, events) emitted so any dimension can later be queried |
| Data collected | Aggregated time-series metrics sampled at intervals | Rich, granular events tagged with context like request ID, user ID, version, region |
| Question answered | 'Is the system healthy right now?' against known-good baselines | 'Why is this specific request/user/shard behaving this way?' for cases not anticipated in advance |
| Detection trigger | Threshold or anomaly crosses a predefined limit, firing an alert | No fixed trigger — engineer initiates an ad hoc query or trace when investigating a symptom |
| Investigation method | Look at the dashboard/panel already built for that known failure mode | Slice and drill into raw telemetry along arbitrary dimensions not decided ahead of time |
| Coverage of failure modes | Effective only for failure modes anticipated when dashboards/alerts were authored | Effective for novel, previously unseen failure modes because data grain supports post-hoc questions |
| Tooling examples | Prometheus, Grafana, Nagios, CloudWatch Alarms | Honeycomb, Jaeger/Tempo, OpenTelemetry, Datadog APM with distributed tracing |
| Cost/storage tradeoff | Cheap — aggregated numeric series compress well over time | Expensive — high-cardinality raw events require more storage and sampling strategy |

## Key Differences

- Monitoring is a practice (watch known metrics, alert on thresholds); observability is a system property (can internal state be inferred from outputs at all)
- Monitoring answers questions decided in advance; observability answers questions you didn't know to ask until an incident happens
- Observability requires richer instrumentation (high-cardinality, high-dimensionality data) than monitoring's aggregated metrics
- You can have monitoring without observability (dashboards for known issues, blind to novel ones), but not real observability without some monitoring layered on top for alerting

## When to Use Each

**Monitoring** — Use monitoring for well-understood, recurring failure modes where you know exactly what to watch — uptime checks, resource saturation, SLA breach alerts.

**Observability** — Invest in observability when your system is complex/distributed enough that failures are novel and multi-causal, and engineers need to ask ad hoc questions during incidents rather than relying on pre-built dashboards.
