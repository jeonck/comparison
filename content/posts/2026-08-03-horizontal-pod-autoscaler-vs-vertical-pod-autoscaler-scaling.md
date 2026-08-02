---
title: "Horizontal Pod Autoscaler vs Vertical Pod Autoscaler: Scaling Out vs Scaling Up"
date: 2026-08-03T05:24:29.720600+09:00
tags: ["kubernetes", "autoscaling", "devops"]
---
## Overview

Both controllers watch metrics and adjust Kubernetes workloads automatically, but they scale in different dimensions. The <strong class="kw">Horizontal Pod Autoscaler</strong> adds or removes pod replicas to handle load, while the <strong class="kw">Vertical Pod Autoscaler</strong> resizes the CPU and memory requests/limits of existing pods.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="50" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Horizontal Pod Autoscaler</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Vertical Pod Autoscaler</text><text x="160" y="58" text-anchor="middle" style="fill:var(--secondary)" font-size="11">before</text><rect x="130" y="68" width="60" height="48" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="96" text-anchor="middle" style="fill:var(--content)" font-size="11">pod</text><line x1="160" y1="122" x2="160" y2="148" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><text x="160" y="166" text-anchor="middle" style="fill:var(--secondary)" font-size="11">after (load increases)</text><rect x="70" y="178" width="50" height="44" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="135" y="178" width="50" height="44" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="200" y="178" width="50" height="44" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="204" text-anchor="middle" style="fill:var(--content)" font-size="10">pod</text><text x="160" y="204" text-anchor="middle" style="fill:var(--content)" font-size="10">pod</text><text x="225" y="204" text-anchor="middle" style="fill:var(--content)" font-size="10">pod</text><text x="160" y="246" text-anchor="middle" style="fill:var(--primary)" font-size="13" font-weight="bold">Scale OUT</text><text x="160" y="264" text-anchor="middle" style="fill:var(--secondary)" font-size="11">more replicas, same pod size</text><text x="480" y="58" text-anchor="middle" style="fill:var(--secondary)" font-size="11">before</text><rect x="455" y="70" width="50" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="92" text-anchor="middle" style="fill:var(--content)" font-size="11">pod</text><line x1="480" y1="122" x2="480" y2="150" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="480" y="168" text-anchor="middle" style="fill:var(--secondary)" font-size="11">after (load increases)</text><rect x="430" y="180" width="100" height="96" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="224" text-anchor="middle" style="fill:var(--content)" font-size="11">pod</text><text x="480" y="242" text-anchor="middle" style="fill:var(--content)" font-size="10">CPU/mem ↑</text><text x="480" y="300" text-anchor="middle" style="fill:var(--primary)" font-size="13" font-weight="bold">Scale UP</text><text x="480" y="318" text-anchor="middle" style="fill:var(--secondary)" font-size="11">bigger pod, same replica count</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Horizontal Pod Autoscaler | Vertical Pod Autoscaler |
| --- | --- | --- |
| What it adjusts | Number of pod replicas in a Deployment/ReplicaSet/StatefulSet | CPU and memory requests/limits on the pod's containers |
| Metrics source | Metrics Server or custom/external metrics (CPU, memory, custom queries) via metrics.k8s.io API | Historical and current usage sampled by the VPA recommender component |
| Trigger condition | Observed metric crosses a target threshold averaged across pods | Recommender detects requests are consistently over- or under-provisioned |
| Action taken | Creates or deletes pod replicas to match target replica count | Evicts and recreates pods with new resource requests (or just recommends, depending on updateMode) |
| Disruption to running pods | None — existing pods are untouched, new ones are added or removed | Pod restart required to apply new resource values, causing brief downtime unless using in-place resize |
| Best fit for workload type | Stateless, horizontally scalable services behind a Service/load balancer | Single-instance or hard-to-replicate workloads, or right-sizing before enabling HPA |
| Conflict risk | Can fight with VPA if both manage CPU on the same workload | Should not manage CPU/memory targeted by HPA on the same workload simultaneously |
| Configuration object | HorizontalPodAutoscaler resource with min/max replicas and target metrics | VerticalPodAutoscaler resource with updateMode (Off, Initial, Recreate, Auto) |

## Key Differences

- HPA changes <strong class="kw">replica count</strong>, VPA changes <strong class="kw">resource requests</strong> on existing pods.
- VPA updates typically require a <strong class="kw">pod restart</strong> to take effect, while HPA scaling adds/removes pods without disrupting the rest.
- Running both on the <strong class="kw">same metric</strong> (like CPU) causes conflicting decisions unless carefully scoped.
- VPA is often used in <strong class="kw">recommendation-only mode</strong> to right-size requests before HPA takes over scaling.
- HPA assumes the workload is <strong class="kw">stateless and replicable</strong>; VPA fits singleton or stateful workloads that can't simply be duplicated.

## When to Use Each

**Horizontal Pod Autoscaler**

- **Stateless web/API services**: Traffic-driven services behind a load balancer scale cleanly by adding more identical replicas.
- **Bursty, unpredictable load**: HPA reacts quickly by spinning replicas up or down as request volume changes.
- **Queue-depth driven workers**: Custom metrics like queue length map naturally to 'how many workers do I need' rather than 'how big should each worker be'.

**Vertical Pod Autoscaler**

- **Single-instance or stateful services**: Workloads like a primary database or leader-elected process can't be horizontally duplicated, so resizing is the only scaling lever.
- **Right-sizing resource requests**: Run VPA in recommendation mode to find correct CPU/memory requests and eliminate guesswork or over-provisioning.
- **Memory-bound workloads**: Some processes need more memory per instance rather than more instances, which only vertical scaling addresses.
