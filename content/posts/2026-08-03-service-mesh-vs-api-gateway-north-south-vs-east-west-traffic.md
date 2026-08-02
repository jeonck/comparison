---
title: "Service Mesh vs API Gateway: North-South vs East-West Traffic"
date: 2026-08-03T05:19:35.145452+09:00
tags: ["service-mesh", "api-gateway", "microservices", "networking"]
---
## Overview

An <strong class="kw">API gateway</strong> sits at the edge of your system, managing traffic between external clients and your services. A <strong class="kw">service mesh</strong> operates inside the cluster, managing traffic between services themselves. Confusing the two leads teams to either duplicate cross-cutting concerns or push edge-only features into infrastructure that was never designed for public-facing traffic.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="235" y="24" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--compare-a)">API Gateway</text><text x="480" y="24" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--compare-b)">Service Mesh</text><rect x="20" y="160" width="80" height="50" rx="6" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="60" y="190" text-anchor="middle" font-size="13" style="fill:var(--content)">Client</text><line x1="100" y1="185" x2="165" y2="185" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="165,180 175,185 165,190" style="fill:var(--content)"/><rect x="175" y="130" width="120" height="110" rx="8" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="235" y="158" text-anchor="middle" font-size="12" font-weight="bold" style="fill:var(--compare-a)">API Gateway</text><text x="235" y="178" text-anchor="middle" font-size="11" style="fill:var(--secondary)">authN · rate limit</text><text x="235" y="194" text-anchor="middle" font-size="11" style="fill:var(--secondary)">routing · transform</text><line x1="295" y1="185" x2="335" y2="185" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="335,180 345,185 335,190" style="fill:var(--content)"/><rect x="345" y="45" width="270" height="280" rx="10" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="6,4"/><text x="480" y="64" text-anchor="middle" font-size="11" style="fill:var(--secondary)">cluster</text><rect x="380" y="85" width="90" height="45" rx="6" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="425" y="111" text-anchor="middle" font-size="12" style="fill:var(--content)">Service A</text><rect x="475" y="95" width="22" height="22" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="380" y="165" width="90" height="45" rx="6" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="425" y="191" text-anchor="middle" font-size="12" style="fill:var(--content)">Service B</text><rect x="475" y="175" width="22" height="22" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="380" y="245" width="90" height="45" rx="6" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="425" y="271" text-anchor="middle" font-size="12" style="fill:var(--content)">Service C</text><rect x="475" y="255" width="22" height="22" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><line x1="497" y1="106" x2="497" y2="186" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="4,3"/><line x1="497" y1="186" x2="497" y2="266" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="4,3"/><path d="M497,106 C560,150 560,220 497,266" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="4,3"/><text x="600" y="100" text-anchor="end" font-size="10" style="fill:var(--secondary)">sidecar proxy</text><text x="600" y="196" text-anchor="end" font-size="10" style="fill:var(--secondary)">mTLS · retries</text><text x="235" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">north–south: client-to-service</text><text x="480" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">east–west: service-to-service</text></svg>
</div>

## Comparison Table

| Aspect | API Gateway | Service Mesh |
| --- | --- | --- |
| Traffic direction | North-south: external clients entering the system | East-west: internal service-to-service calls |
| Deployment topology | Centralized cluster of edge instances fronting all traffic | Sidecar proxy injected alongside every service instance |
| Primary concerns | AuthN/authZ, rate limiting, request/response transformation, API versioning | mTLS, load balancing, retries, circuit breaking between services |
| Routing basis | Public API path, host, or version mapped to a backend service | Service identity and destination within the internal network |
| Observability scope | Per-endpoint metrics: request volume, latency, errors by client | Full service dependency graph: per-hop latency and error rates |
| Failure containment | Blocks or throttles bad traffic before it reaches any backend | Isolates failures at individual hops so one bad service doesn't cascade |
| Operational overhead | Few instances to scale and configure centrally | One proxy per workload, plus a control plane to manage them all |

## Key Differences

- An <strong class="kw">API gateway</strong> is the single entry point clients hit; a <strong class="kw">service mesh</strong> has no single entry point, it's woven through every service.
- Gateways enforce policy once at the edge; meshes enforce policy per <strong class="kw">sidecar</strong> on every call.
- Gateways typically run as a small number of centralized instances; meshes scale linearly with your <strong class="kw">service count</strong>.
- Meshes give you <strong class="kw">mTLS</strong> and retries between internal services, something a gateway never sees because that traffic never reaches it.
- Many production systems run both together, not as alternatives, since they solve problems at different layers.

## When to Use Each

**API Gateway**

- **Public API management**: You need a single point to apply auth, rate limiting, and versioning for external consumers.
- **Protocol translation at the edge**: Clients speak REST or GraphQL but internal services use gRPC, so translation belongs at the boundary.
- **Simple microservice setups**: With only a handful of services, a gateway alone can handle routing without the overhead of a full mesh.

**Service Mesh**

- **Zero-trust internal networking**: You need automatic mTLS and identity verification between every service without changing application code.
- **Large microservice fleets**: Dozens or hundreds of services need consistent retries, timeouts, and circuit breaking without per-service implementation.
- **Fine-grained traffic control**: You want canary rollouts or traffic shifting between service versions at the network layer.
