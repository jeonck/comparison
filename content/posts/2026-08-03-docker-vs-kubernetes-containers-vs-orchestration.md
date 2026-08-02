---
title: "Docker vs Kubernetes: Containers vs Orchestration"
date: 2026-08-03T05:15:40.136576+09:00
tags: ["docker", "kubernetes", "containers", "orchestration"]
---
## Overview

<strong class="kw">Docker</strong> packages an application and its dependencies into a portable container image and runs it on a single host, while <strong class="kw">Kubernetes</strong> schedules, scales, and heals many containers across a cluster of machines. They aren't direct substitutes — Kubernetes typically runs containers built by Docker (or another OCI-compatible tool), sitting one layer above it.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="32" text-anchor="middle" font-size="18" style="fill:var(--primary)">Docker</text><text x="480" y="32" text-anchor="middle" font-size="18" style="fill:var(--primary)">Kubernetes</text><rect x="40" y="55" width="240" height="270" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="78" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Single Host</text><rect x="65" y="95" width="70" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="100" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">app A</text><rect x="150" y="95" width="70" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="185" y="130" text-anchor="middle" font-size="11" style="fill:var(--content)">app B</text><rect x="65" y="170" width="70" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="100" y="205" text-anchor="middle" font-size="11" style="fill:var(--content)">app C</text><rect x="150" y="170" width="70" height="60" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4"/><text x="185" y="205" text-anchor="middle" font-size="11" style="fill:var(--secondary)">idle</text><text x="160" y="265" text-anchor="middle" font-size="11" style="fill:var(--secondary)">docker run</text><text x="160" y="282" text-anchor="middle" font-size="11" style="fill:var(--secondary)">manual, per-host</text><text x="160" y="310" text-anchor="middle" font-size="11" style="fill:var(--secondary)">if host dies, all lost</text><rect x="400" y="55" width="160" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="78" text-anchor="middle" font-size="11" style="fill:var(--content)">Control Plane</text><line x1="440" y1="91" x2="400" y2="120" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3"/><line x1="480" y1="91" x2="480" y2="120" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3"/><line x1="520" y1="91" x2="560" y2="120" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3"/><rect x="360" y="120" width="80" height="90" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="400" y="135" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Node 1</text><rect x="370" y="145" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="404" y="145" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="370" y="178" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="120" width="80" height="90" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="135" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Node 2</text><rect x="450" y="145" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="484" y="145" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="520" y="120" width="80" height="90" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="560" y="135" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Node 3</text><rect x="530" y="145" width="26" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="564" y="145" width="26" height="26" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3"/><text x="480" y="235" text-anchor="middle" font-size="11" style="fill:var(--secondary)">scheduler places pods</text><text x="480" y="252" text-anchor="middle" font-size="11" style="fill:var(--secondary)">auto-reschedules on failure</text><text x="480" y="280" text-anchor="middle" font-size="11" style="fill:var(--secondary)">declarative, cluster-wide</text></svg>
</div>

## Comparison Table

| Aspect | Docker | Kubernetes |
| --- | --- | --- |
| Core purpose | Build, package, and run containers from a single image spec | Orchestrate and manage many containers across a fleet of machines |
| Unit of work | Container, defined by a Dockerfile and run via docker run | Pod, a group of one or more containers scheduled together |
| Deployment scope | Single host (or manually scripted across hosts) | Multi-node cluster with a control plane scheduling workloads |
| Configuration model | Imperative CLI commands or docker-compose.yml | Declarative YAML manifests reconciled continuously toward desired state |
| Networking & discovery | User-defined bridge networks and container name resolution | Cluster-wide Services, DNS, and Ingress across nodes |
| Scaling | Manual — start more containers or use docker-compose scale | Automated via ReplicaSets and Horizontal Pod Autoscaler |
| Failure recovery | No built-in restart across host failure; relies on restart policies per host | Self-healing — reschedules pods automatically if a node or container fails |
| Rollouts & updates | Rebuild image and manually restart containers | Rolling updates and rollbacks managed declaratively per Deployment |

## Key Differences

- Docker operates at the level of a single <strong class="kw">container</strong>; Kubernetes operates at the level of a <strong class="kw">cluster</strong>.
- Kubernetes doesn't replace Docker — it typically schedules containers that Docker (or another <strong class="kw">container runtime</strong>) built and runs.
- Docker's model is largely <strong class="kw">imperative</strong>, while Kubernetes is fundamentally <strong class="kw">declarative</strong>, continuously reconciling actual state to desired state.
- Kubernetes adds <strong class="kw">self-healing</strong> and autoscaling that plain Docker has no native concept of.
- For a single app on one machine, Kubernetes' <strong class="kw">control plane</strong> overhead is often unjustified complexity.

## When to Use Each

**Docker**

- **Local development**: Docker gives fast, reproducible builds and a single command to run an app on a laptop without cluster overhead.
- **Single-server deployment**: A small app on one VM needs docker run or docker-compose, not a full orchestration layer.
- **CI build pipelines**: Docker is the standard way to build, tag, and push images regardless of where they'll ultimately run.

**Kubernetes**

- **Multi-service production systems**: Kubernetes coordinates many interdependent services across nodes with shared networking and service discovery.
- **High-availability requirements**: Automatic rescheduling and health checks keep workloads running through node or container failures.
- **Elastic, variable-load workloads**: Horizontal Pod Autoscaler adjusts replica counts in response to real traffic and resource usage.
- **Complex release management**: Rolling updates, canary rollouts, and declarative rollbacks are built into Kubernetes' Deployment model.
