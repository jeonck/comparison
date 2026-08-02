---
title: "Cold Start vs Warm Start: Why the First Request Feels Slower"
date: 2026-08-03T06:24:34.362792+09:00
tags: ["serverless", "performance", "cloud-computing", "functions"]
---
## Overview

In serverless and containerized systems, a <strong class="kw">cold start</strong> happens when a request must wait for a new execution environment to be provisioned and initialized before it can run, while a <strong class="kw">warm start</strong> reuses an already-running instance and skips straight to execution. The gap between the two explains why the same function can respond in 5ms or 2 seconds depending on whether an idle instance was standing by.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="20" y="35" font-size="16" font-weight="bold" style="fill:var(--primary)">Cold Start</text><circle cx="55" cy="90" r="18" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="55" y="94" font-size="10" text-anchor="middle" style="fill:var(--content)">Req</text><line x1="73" y1="90" x2="98" y2="90" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="100" y="65" width="110" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="86" font-size="11" text-anchor="middle" style="fill:var(--content)">Provision</text><text x="155" y="100" font-size="11" text-anchor="middle" style="fill:var(--content)">container</text><line x1="210" y1="90" x2="230" y2="90" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="232" y="65" width="110" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="287" y="86" font-size="11" text-anchor="middle" style="fill:var(--content)">Init runtime</text><text x="287" y="100" font-size="11" text-anchor="middle" style="fill:var(--content)">+ code</text><line x1="342" y1="90" x2="362" y2="90" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="364" y="65" width="100" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="414" y="94" font-size="11" text-anchor="middle" style="fill:var(--content)">Execute</text><line x1="100" y1="130" x2="464" y2="130" style="stroke:var(--secondary)" stroke-width="1"/><text x="282" y="146" font-size="11" text-anchor="middle" style="fill:var(--secondary)">latency: tens of ms - several seconds</text><line x1="0" y1="180" x2="640" y2="180" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="20" y="215" font-size="16" font-weight="bold" style="fill:var(--primary)">Warm Start</text><rect x="100" y="205" width="264" height="30" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><text x="232" y="224" font-size="10" text-anchor="middle" style="fill:var(--secondary)">idle, pre-initialized container waiting</text><circle cx="55" cy="270" r="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="55" y="274" font-size="10" text-anchor="middle" style="fill:var(--content)">Req</text><line x1="73" y1="270" x2="362" y2="270" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="364" y="245" width="100" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="414" y="274" font-size="11" text-anchor="middle" style="fill:var(--content)">Execute</text><line x1="100" y1="310" x2="464" y2="310" style="stroke:var(--secondary)" stroke-width="1"/><text x="282" y="326" font-size="11" text-anchor="middle" style="fill:var(--secondary)">latency: sub-ms - low tens of ms</text></svg>
</div>

## Comparison Table

| Aspect | Cold Start | Warm Start |
| --- | --- | --- |
| Trigger condition | No idle instance available (scale-to-zero, scale-out, or fresh deploy) | Idle, already-initialized instance is available to handle the request |
| Environment state at invocation | No running process; container or sandbox must be created from scratch | Process is already running in memory from a prior invocation |
| Steps performed | Provision compute, load code, initialize runtime and dependencies, run init code, then handle request | Skip provisioning and init; execute the handler directly on the existing process |
| Typical latency added | Tens of milliseconds to several seconds depending on runtime and package size | Sub-millisecond to low tens of milliseconds |
| Resource cost to provider | Higher; allocates new compute, memory, and network setup | Lower; reuses resources already allocated |
| Frequency of occurrence | Rare relative to total traffic but concentrated after idle periods, deploys, or scale-out | Common; most requests during steady, active traffic |
| Primary mitigation | Provisioned concurrency, smaller packages, lighter runtimes, scheduled pings | Sustained traffic, minimum instance counts, connection reuse |

## Key Differences

- Cold start pays full <strong class="kw">provisioning</strong> overhead; warm start reuses an already-initialized process.
- The latency gap can span <strong class="kw">orders of magnitude</strong> — low milliseconds versus multiple seconds.
- Cold starts are triggered by <strong class="kw">scale-to-zero</strong> or scale-out events, not by what the request contains.
- Whether a start is warm depends on the platform's <strong class="kw">idle timeout</strong> before it reclaims the instance.
- Avoiding cold starts usually means paying for <strong class="kw">reserved capacity</strong> to keep instances standing by.

## When to Use Each

**Cold Start**

- **Infrequent or bursty jobs**: Cron tasks and low-traffic endpoints rarely stay warm between invocations, so cold starts are expected and usually acceptable.
- **Cost-sensitive scale-to-zero**: Letting compute fully deprovision between calls avoids idle billing at the cost of occasional cold start latency.
- **Just after deployment**: Every new version rollout replaces warm instances with fresh ones, making cold starts unavoidable immediately post-release.

**Warm Start**

- **Latency-sensitive APIs**: User-facing endpoints need consistent low latency, which requires keeping instances warm via traffic or provisioned concurrency.
- **High, steady request volume**: Continuous traffic naturally keeps instances busy, so most requests land on already-warm processes.
- **Real-time interactive workloads**: Chat, gaming, and streaming backends need warm-start response times on nearly every request, not just most of them.
