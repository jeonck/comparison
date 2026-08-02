---
title: "Liveness Probe vs Readiness Probe: Kubernetes Health Checks Compared"
date: 2026-08-03T05:22:16.438326+09:00
tags: ["kubernetes", "health-checks", "devops", "reliability"]
---
## Overview

Kubernetes uses liveness and readiness probes to answer two different questions about a running container: is it alive, and is it ready to serve requests. A failed liveness probe triggers a container <strong class="kw">restart</strong>, while a failed readiness probe only affects <strong class="kw">traffic routing</strong> by pulling the pod out of Service endpoints without killing it.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="26" text-anchor="middle" style="fill:var(--primary)" font-size="15" font-weight="bold">Liveness Probe</text><text x="480" y="26" text-anchor="middle" style="fill:var(--primary)" font-size="15" font-weight="bold">Readiness Probe</text><rect x="90" y="42" width="140" height="34" rx="4" style="fill:none;stroke:var(--content)"/><text x="160" y="63" text-anchor="middle" style="fill:var(--content)" font-size="12">Container</text><line x1="160" y1="76" x2="160" y2="96" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="60" y="96" width="200" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="119" text-anchor="middle" style="fill:var(--content)" font-size="11">kubelet: is it alive?</text><line x1="160" y1="134" x2="160" y2="152" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,152 198,180 160,208 122,180" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="184" text-anchor="middle" style="fill:var(--content)" font-size="10">Fails?</text><line x1="160" y1="208" x2="160" y2="240" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="172" y="226" style="fill:var(--secondary)" font-size="10">yes</text><rect x="60" y="240" width="200" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="263" text-anchor="middle" style="fill:var(--content)" font-size="11">Kill &amp; Restart Container</text><path d="M60,259 C15,259 15,59 88,59" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="88,59 78,54 78,64" style="fill:var(--compare-a)"/><line x1="122" y1="180" x2="90" y2="180" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="105" y="192" text-anchor="end" style="fill:var(--secondary)" font-size="10">no</text><text x="30" y="184" text-anchor="middle" style="fill:var(--secondary)" font-size="10">stays</text><text x="30" y="196" text-anchor="middle" style="fill:var(--secondary)" font-size="10">running</text><text x="160" y="305" text-anchor="middle" style="fill:var(--secondary)" font-size="11">No effect on Service traffic</text><rect x="410" y="42" width="140" height="34" rx="4" style="fill:none;stroke:var(--content)"/><text x="480" y="63" text-anchor="middle" style="fill:var(--content)" font-size="12">Container</text><line x1="480" y1="76" x2="480" y2="96" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="380" y="96" width="200" height="38" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="119" text-anchor="middle" style="fill:var(--content)" font-size="11">kubelet: is it ready?</text><line x1="480" y1="134" x2="480" y2="152" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,152 518,180 480,208 442,180" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="184" text-anchor="middle" style="fill:var(--content)" font-size="10">Ready?</text><line x1="480" y1="208" x2="480" y2="240" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="492" y="226" style="fill:var(--secondary)" font-size="10">yes</text><rect x="380" y="240" width="200" height="38" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="263" text-anchor="middle" style="fill:var(--content)" font-size="11">Added to Service Endpoints</text><line x1="518" y1="180" x2="530" y2="180" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="535" y="175" style="fill:var(--secondary)" font-size="10">no</text><rect x="530" y="186" width="80" height="46" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3 3"/><text x="570" y="204" text-anchor="middle" style="fill:var(--content)" font-size="9">Removed</text><text x="570" y="216" text-anchor="middle" style="fill:var(--content)" font-size="9">(not killed)</text><line x1="480" y1="278" x2="480" y2="300" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="390" y="300" width="180" height="34" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="321" text-anchor="middle" style="fill:var(--content)" font-size="11">Service / Load Balancer</text></svg>
</div>

## Comparison Table

| Aspect | Liveness Probe | Readiness Probe |
| --- | --- | --- |
| Core question | Is the process still functioning? | Is the process ready to accept traffic? |
| Action on failure | kubelet kills and restarts the container | Container is left running, no restart |
| Effect on Service endpoints | None directly; pod may keep receiving traffic until restart completes | Pod is removed from Service endpoints, stops receiving traffic |
| Effect on rolling deployments | Not consulted for rollout progress | Must pass before the pod counts as available and rollout proceeds |
| Restart count impact | Increments the container restart count on each failure | Never causes a restart |
| Typical checks used | Lightweight self-check for hangs or deadlocks | Checks dependency health: DB connections, cache warm-up, config load |
| Misconfiguration risk | Too-aggressive thresholds cause restart loops (CrashLoopBackOff) | Too-aggressive thresholds pull healthy pods out of rotation, cutting capacity |

## Key Differences

- Liveness failure causes a container <strong class="kw">restart</strong>; readiness failure only removes the pod from <strong class="kw">Service endpoints</strong>.
- Liveness answers "is it alive," readiness answers "is it <strong class="kw">ready for traffic</strong>."
- Readiness gates <strong class="kw">rolling deployments</strong>; liveness has no say in rollout progress.
- Using a dependency check as a liveness probe risks a <strong class="kw">restart loop</strong> when the real problem is an external service, not the process.

## When to Use Each

**Liveness Probe**

- **Detecting Deadlocks**: A liveness probe catches a process that is technically running but internally stuck, and forces a restart to recover it.
- **Self-Healing Long-Running Services**: For processes prone to gradual state corruption (memory leaks, stuck threads), a periodic liveness check keeps the workload self-repairing without manual intervention.
- **Guarding Against Silent Hangs**: A minimal endpoint that just confirms the event loop or main thread still responds is enough to catch hangs that a crash wouldn't otherwise surface.

**Readiness Probe**

- **Slow Startup / Cache Warm-up**: A readiness probe keeps a pod out of rotation until it finishes loading data or establishing connections, preventing early requests from failing.
- **Temporary Dependency Outage**: If a downstream database briefly drops, readiness lets the pod stop receiving traffic without being killed and restarted for a problem it can't fix by restarting.
- **Zero-Downtime Rolling Deployments**: Readiness gates when a new pod is considered available, so the rollout only shifts traffic once the replacement is actually able to serve it.
