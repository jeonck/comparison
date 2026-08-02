---
title: "Load Balancer vs Reverse Proxy: Traffic Distribution vs Request Mediation"
date: 2026-08-03T06:30:57.372910+09:00
tags: ["networking", "load-balancer", "reverse-proxy", "infrastructure"]
---
## Overview

A <strong class="kw">load balancer</strong> spreads incoming traffic across many identical backend servers so no single machine gets overwhelmed, while a <strong class="kw">reverse proxy</strong> sits in front of one or more servers to mediate, secure, and transform requests on their behalf. The two overlap heavily in practice — most modern reverse proxies (NGINX, Envoy, HAProxy) can also load balance — but the distinction matters when you're deciding which capability you actually need to configure or scale for.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Load Balancer</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Reverse Proxy</text><line x1="320" y1="50" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><circle cx="70" cy="90" r="18" style="fill:none;stroke:var(--content)" stroke-width="2"/><text x="70" y="95" text-anchor="middle" style="fill:var(--content)" font-size="12">C</text><text x="70" y="122" text-anchor="middle" style="fill:var(--secondary)" font-size="11">clients</text><rect x="130" y="75" width="110" height="36" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="185" y="98" text-anchor="middle" style="fill:var(--content)" font-size="12">Balancer</text><line x1="88" y1="90" x2="128" y2="93" style="stroke:var(--content)" stroke-width="1.5"/><rect x="150" y="160" width="90" height="34" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="181" text-anchor="middle" style="fill:var(--content)" font-size="11">Server A</text><rect x="150" y="210" width="90" height="34" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="231" text-anchor="middle" style="fill:var(--content)" font-size="11">Server B</text><rect x="150" y="260" width="90" height="34" rx="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="281" text-anchor="middle" style="fill:var(--content)" font-size="11">Server C</text><line x1="185" y1="111" x2="195" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="185" y1="111" x2="195" y2="210" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="185" y1="111" x2="195" y2="260" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="185" y="320" text-anchor="middle" style="fill:var(--secondary)" font-size="11">even work distribution</text><circle cx="390" cy="90" r="18" style="fill:none;stroke:var(--content)" stroke-width="2"/><text x="390" y="95" text-anchor="middle" style="fill:var(--content)" font-size="12">C</text><text x="390" y="122" text-anchor="middle" style="fill:var(--secondary)" font-size="11">clients</text><rect x="450" y="75" width="110" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="505" y="98" text-anchor="middle" style="fill:var(--content)" font-size="12">Rev Proxy</text><line x1="408" y1="90" x2="448" y2="93" style="stroke:var(--content)" stroke-width="1.5"/><rect x="430" y="120" width="70" height="26" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><text x="465" y="137" text-anchor="middle" style="fill:var(--secondary)" font-size="9">TLS + cache</text><rect x="460" y="185" width="90" height="40" rx="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="505" y="209" text-anchor="middle" style="fill:var(--content)" font-size="11">App Server</text><line x1="505" y1="111" x2="505" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="505" y="260" text-anchor="middle" style="fill:var(--secondary)" font-size="11">shields, secures,</text><text x="505" y="276" text-anchor="middle" style="fill:var(--secondary)" font-size="11">transforms one origin</text></svg>
</div>

## Comparison Table

| Aspect | Load Balancer | Reverse Proxy |
| --- | --- | --- |
| Primary goal | Distribute traffic across many backend instances to avoid overload | Mediate and forward requests to one or more origin servers on their behalf |
| OSI layer | Operates at L4 (TCP/UDP) or L7 (HTTP), often chosen for raw throughput | Almost always operates at L7, inspecting and rewriting HTTP requests |
| Routing decision | Algorithmic: round-robin, least-connections, weighted, or hashed | Rule-based: URL path, host header, headers, or cookies map to a backend |
| Backend topology | Assumes a pool of interchangeable, horizontally scaled servers | Commonly fronts a single origin, or routes distinct paths to different services |
| TLS termination | Supported, mainly to offload encryption before distributing load | Core use case, paired with header rewriting and request/response filtering |
| Caching & transformation | Not a typical feature; focus stays on connection distribution | Frequently caches responses, compresses, or rewrites headers/body |
| Failure handling | Health checks remove unresponsive nodes from the rotation automatically | Can retry or failover, but its main job is passing requests through correctly |
| Client-facing identity | May be invisible to clients, just an IP fronting the pool | Deliberately presents a single unified identity, hiding origin topology |

## Key Differences

- A load balancer's job is <strong class="kw">scaling out</strong> — spreading load across a pool; a reverse proxy's job is <strong class="kw">mediating access</strong> to one or more origins.
- Load balancers lean on <strong class="kw">distribution algorithms</strong>, while reverse proxies lean on <strong class="kw">content-based routing rules</strong>.
- Reverse proxies commonly own <strong class="kw">TLS termination</strong> and response caching as first-class features, not just add-ons.
- Modern tools like <strong class="kw">NGINX or Envoy</strong> blur the line by implementing both roles in a single process.
- A reverse proxy can front a <strong class="kw">single server</strong> for security/caching alone, with no load distribution involved at all.

## When to Use Each

**Load Balancer**

- **Horizontal scaling**: You have multiple identical app instances and need traffic spread evenly to keep latency and CPU load balanced.
- **High-throughput L4 traffic**: You need to distribute raw TCP/UDP connections with minimal overhead, without inspecting HTTP content.
- **Zero-downtime deploys**: Health checks let you pull unhealthy or draining instances out of rotation automatically during rollouts.

**Reverse Proxy**

- **Centralizing TLS and auth**: You want one place to terminate HTTPS, enforce headers, or add authentication in front of internal services.
- **Hiding internal topology**: Clients should see one hostname while requests are routed to different internal services by path or header.
- **Response caching**: You want to cache or compress responses close to the client without modifying the origin server itself.
