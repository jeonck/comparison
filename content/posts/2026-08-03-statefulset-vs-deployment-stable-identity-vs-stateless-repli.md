---
title: "StatefulSet vs Deployment: Stable Identity vs Stateless Replicas"
date: 2026-08-03T05:23:54.598634+09:00
tags: ["kubernetes", "statefulset", "deployment", "orchestration"]
---
## Overview

Both are Kubernetes controllers that manage sets of pods from a template, but they solve different problems: a <strong class="kw">Deployment</strong> treats pods as interchangeable, disposable replicas, while a <strong class="kw">StatefulSet</strong> gives each pod a stable name, network identity, and persistent storage that survives rescheduling. The choice matters because workloads like databases or clustered systems break if pod identity or storage isn't preserved across restarts.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Deployment</text><text x="480" y="30" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">StatefulSet</text><line x1="320" y1="45" x2="320" y2="330" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><rect x="55" y="70" width="190" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="100" text-anchor="middle" font-size="13" style="fill:var(--content)">web-7f9d4c</text><rect x="55" y="150" width="190" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="180" text-anchor="middle" font-size="13" style="fill:var(--content)">web-x9y8z2</text><rect x="55" y="230" width="190" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">web-m4n5p6</text><path d="M245,95 C280,110 280,160 245,175" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="3 3"/><path d="M245,175 C280,190 280,240 245,255" fill="none" style="stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="3 3"/><text x="150" y="315" text-anchor="middle" font-size="12" style="fill:var(--secondary)">interchangeable, no stable identity</text><rect x="395" y="70" width="150" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="470" y="100" text-anchor="middle" font-size="13" style="fill:var(--content)">web-0</text><rect x="395" y="150" width="150" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="470" y="180" text-anchor="middle" font-size="13" style="fill:var(--content)">web-1</text><rect x="395" y="230" width="150" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="470" y="260" text-anchor="middle" font-size="13" style="fill:var(--content)">web-2</text><line x1="470" y1="120" x2="470" y2="146" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="470,150 465,140 475,140" style="fill:var(--compare-b)"/><line x1="470" y1="200" x2="470" y2="226" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="470,230 465,220 475,220" style="fill:var(--compare-b)"/><line x1="545" y1="95" x2="565" y2="95" style="stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="80" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="568" y="80" width="44" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="110" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="99" text-anchor="middle" font-size="9" style="fill:var(--content)">pvc-0</text><line x1="545" y1="175" x2="565" y2="175" style="stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="160" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="568" y="160" width="44" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="190" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="179" text-anchor="middle" font-size="9" style="fill:var(--content)">pvc-1</text><line x1="545" y1="255" x2="565" y2="255" style="stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="240" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="568" y="240" width="44" height="30" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="590" cy="270" rx="22" ry="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="259" text-anchor="middle" font-size="9" style="fill:var(--content)">pvc-2</text><text x="470" y="315" text-anchor="middle" font-size="12" style="fill:var(--secondary)">stable name, network ID &amp; storage per pod</text></svg>
</div>

## Comparison Table

| Aspect | Deployment | StatefulSet |
| --- | --- | --- |
| Pod naming | Random hash suffix per replica (web-7f9d4c-x2z9p), changes on every recreation | Stable ordinal index (web-0, web-1, web-2) fixed for the pod's lifetime |
| Network identity | Pods share a single Service VIP/DNS; individual pods have no predictable DNS name | Requires a headless Service; each pod gets a stable DNS entry (web-0.svc.namespace) |
| Storage | PVCs, if used, aren't guaranteed to reattach to the same pod on recreation | volumeClaimTemplates provision a dedicated PVC per pod that persists and reattaches |
| Scaling | Creates or removes pods in parallel, in any order | Scales one pod at a time in strict ordinal order (0, 1, 2, ...) |
| Rolling updates | Replaces pods per maxSurge/maxUnavailable, order not guaranteed | Updates pods one at a time in reverse ordinal order (N-1 down to 0) by default |
| Failure recovery | Replacement pod gets a new name and no guaranteed storage continuity | Replacement pod keeps the same name/identity and reattaches its original PVC |
| Typical workload | Stateless web servers, APIs, workers that scale horizontally | Databases, message queues, and clustered systems needing stable peers |

## Key Differences

- Deployment pods get <strong class="kw">random names</strong> on every recreation, while StatefulSet pods keep a fixed <strong class="kw">ordinal name</strong> for life
- Only StatefulSet supports <strong class="kw">volumeClaimTemplates</strong>, giving each pod its own persistent volume that survives rescheduling
- StatefulSet requires a <strong class="kw">headless Service</strong> to give each pod a resolvable, stable DNS entry
- StatefulSet scales and updates pods in strict <strong class="kw">ordinal order</strong>; Deployment does both in parallel
- On node failure, Deployment pods lose their identity entirely, while StatefulSet pods are recreated with the <strong class="kw">same identity</strong>

## When to Use Each

**Deployment**

- **Stateless web/API tier**: Deployment's interchangeable pods fit services where any replica can serve any request without local state.
- **Fast, parallel rollouts**: Deployment can replace or scale many pods simultaneously, minimizing rollout time for stateless services.
- **Horizontal autoscaling**: HPA-driven scale-out works cleanly when pods have no identity or storage dependencies to preserve.

**StatefulSet**

- **Clustered databases**: Systems like Postgres, MongoDB, or Cassandra rely on stable pod identity and dedicated storage per replica for replication.
- **Peer-discovery systems**: Kafka, Zookeeper, and Elasticsearch nodes need predictable DNS names to find and address each other.
- **Per-pod persistent storage**: Workloads needing a durable volume tied to a specific replica require StatefulSet's volumeClaimTemplates.
