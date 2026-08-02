---
title: "Serverless vs Containers: Who Manages the Runtime"
date: 2026-08-03T06:20:35.450399+09:00
tags: ["serverless", "containers", "cloud-architecture"]
---
## Overview

Both let you deploy application code without owning physical servers, but they draw the abstraction line in different places. <strong class="kw">Serverless</strong> functions run only in response to events and scale to zero between invocations, while <strong class="kw">containers</strong> package your app with its dependencies into a persistent, always-addressable process you (or an orchestrator) keep running.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="32" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Serverless</text><text x="480" y="32" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Containers</text><line x1="320" y1="20" x2="320" y2="340" stroke-width="1" style="stroke:var(--border)"/><line x1="50" y1="230" x2="290" y2="230" stroke-width="2" style="stroke:var(--border)"/><circle cx="75" cy="230" r="7" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="75" y1="223" x2="75" y2="180" stroke-dasharray="3,3" stroke-width="1.5" style="stroke:var(--compare-a)"/><rect x="50" y="140" width="50" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="165" cy="230" r="7" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="165" y1="223" x2="165" y2="170" stroke-dasharray="3,3" stroke-width="1.5" style="stroke:var(--compare-a)"/><rect x="140" y="130" width="50" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="255" cy="230" r="7" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="255" y1="223" x2="255" y2="190" stroke-dasharray="3,3" stroke-width="1.5" style="stroke:var(--compare-a)"/><rect x="230" y="150" width="50" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="110" text-anchor="middle" font-size="11" style="fill:var(--content)">Instance per event</text><text x="170" y="260" text-anchor="middle" font-size="11" style="fill:var(--secondary)">requests</text><text x="170" y="300" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Idle gaps = zero cost, zero running process</text><rect x="360" y="60" width="240" height="230" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,3"/><text x="480" y="80" text-anchor="middle" font-size="11" style="fill:var(--content)">Cluster / host</text><rect x="385" y="100" width="190" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="122" text-anchor="middle" font-size="11" style="fill:var(--content)">Container A</text><rect x="385" y="150" width="190" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="172" text-anchor="middle" font-size="11" style="fill:var(--content)">Container B</text><rect x="385" y="200" width="190" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="222" text-anchor="middle" font-size="11" style="fill:var(--content)">Container C</text><text x="480" y="312" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Always running, billed continuously</text></svg>
</div>

## Comparison Table

| Aspect | Serverless | Containers |
| --- | --- | --- |
| Deployment unit | Single function handler plus its dependencies | Full image with OS layers, runtime, and app code |
| Startup trigger | Invoked per event (HTTP call, queue message, timer) | Started explicitly and left running by an orchestrator |
| Runtime lifetime | Ephemeral, seconds to minutes, then torn down | Long-lived, runs continuously until stopped or redeployed |
| State handling | Stateless between invocations; external store required | Can hold in-memory state across requests within its life |
| Scaling behavior | Platform scales instance count automatically, including to zero | You or an orchestrator (e.g. Kubernetes) define replica counts and rules |
| Resource control | No control over OS, runtime patching, or underlying host | Full control over base image, OS packages, and runtime version |
| Cost model | Pay per invocation and execution time, nothing when idle | Pay for allocated capacity whether or not it's handling traffic |
| Operational overhead | No servers, patching, or orchestration to manage | You own cluster upkeep, scaling policy, and image maintenance |

## Key Differences

- Serverless bills per <strong class="kw">invocation</strong>, containers bill for <strong class="kw">allocated capacity</strong> regardless of traffic
- Containers give you a fixed <strong class="kw">runtime environment</strong> you control; serverless abstracts the OS away entirely
- Cold starts and short execution limits shape serverless <strong class="kw">function design</strong>; containers have no such ceiling
- Serverless functions are inherently <strong class="kw">stateless</strong>, while containers can maintain in-process state across requests

## When to Use Each

**Serverless**

- **Spiky or unpredictable traffic**: Serverless scales to zero and back without you pre-provisioning capacity for rare bursts.
- **Event-driven glue code**: Short handlers reacting to queue messages, uploads, or webhooks fit the per-invocation execution model well.
- **Minimizing ops burden**: No cluster, patching, or scaling policy to maintain when the platform manages the runtime entirely.

**Containers**

- **Long-running or stateful services**: Containers can hold connections, caches, or in-memory state that would be lost between serverless invocations.
- **Consistent, predictable load**: Steady traffic makes fixed running capacity more cost-effective than per-invocation billing.
- **Custom runtime or OS needs**: Containers let you pin exact OS packages, binaries, or runtime versions that a managed function environment won't allow.
