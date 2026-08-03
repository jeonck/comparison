---
title: "Stateless vs Stateful Architecture: Where Request Context Lives"
date: 2026-08-04T05:18:18.046436+09:00
tags: ["stateless", "stateful", "system-design", "scalability"]
---
## Overview

This comparison covers how servers handle client context across requests: a <strong class="kw">stateless</strong> server treats every request as self-contained with no memory of what came before, while a <strong class="kw">stateful</strong> server keeps track of session data tied to a specific client across multiple requests. The choice shapes how easily a system scales, recovers from failure, and routes traffic under load.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg" font-family="sans-serif"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="35" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Stateless</text><text x="480" y="35" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Stateful</text><rect x="20" y="150" width="70" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="55" y="174" text-anchor="middle" font-size="12" style="fill:var(--content)">Client</text><rect x="130" y="150" width="70" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="165" y="168" text-anchor="middle" font-size="11" style="fill:var(--content)">Load</text><text x="165" y="182" text-anchor="middle" font-size="11" style="fill:var(--content)">Balancer</text><rect x="240" y="95" width="65" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="272" y="117" text-anchor="middle" font-size="11" style="fill:var(--content)">Server A</text><rect x="240" y="152" width="65" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="272" y="174" text-anchor="middle" font-size="11" style="fill:var(--content)">Server B</text><rect x="240" y="209" width="65" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="272" y="231" text-anchor="middle" font-size="11" style="fill:var(--content)">Server C</text><line x1="90" y1="170" x2="128" y2="170" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="200" y1="165" x2="238" y2="130" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="200" y1="170" x2="238" y2="170" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="200" y1="175" x2="238" y2="212" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="160" y="280" text-anchor="middle" font-size="10" style="fill:var(--secondary)">any server can handle any request</text><text x="160" y="296" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no session stored server-side</text><rect x="355" y="150" width="70" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="174" text-anchor="middle" font-size="12" style="fill:var(--content)">Client</text><rect x="470" y="140" width="90" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="515" y="165" text-anchor="middle" font-size="11" style="fill:var(--content)">Server</text><text x="515" y="180" text-anchor="middle" font-size="10" style="fill:var(--content)">+ Session</text><line x1="425" y1="170" x2="468" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="447" y="160" text-anchor="middle" font-size="9" style="fill:var(--secondary)">sticky</text><rect x="580" y="95" width="45" height="30" rx="4" stroke-dasharray="3 3" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="602" y="114" text-anchor="middle" font-size="9" style="fill:var(--secondary)">Server</text><rect x="580" y="215" width="45" height="30" rx="4" stroke-dasharray="3 3" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="602" y="234" text-anchor="middle" font-size="9" style="fill:var(--secondary)">Server</text><text x="480" y="280" text-anchor="middle" font-size="10" style="fill:var(--secondary)">must always return to</text><text x="480" y="296" text-anchor="middle" font-size="10" style="fill:var(--secondary)">same server holding session</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Stateless | Stateful |
| --- | --- | --- |
| Request self-sufficiency | Each request carries all data needed to process it, independent of prior requests | Each request depends on context accumulated from prior requests in the same session |
| State storage location | Held externally (client token, database, cache) or not persisted at all | Held in server memory or local storage tied to a specific server instance |
| Load balancing | Any available server can handle any request; simple round-robin routing | Requests must be routed to the specific server holding the session (sticky sessions) |
| Horizontal scaling | Add or remove server instances freely with no coordination needed | Requires state replication or migration before instances can be added or removed |
| Failure recovery | A crashed server loses nothing; the next request is simply retried elsewhere | A crashed server can drop the active session unless state was replicated |
| Resource footprint per server | Lower memory overhead since no per-client data is retained between requests | Higher memory/storage overhead from tracking active sessions |
| Typical examples | REST APIs, DNS lookups, serverless functions, CDN edge nodes | Database connections, WebSocket sessions, FTP, multiplayer game servers |

## Key Differences

- Stateless servers require no <strong class="kw">session affinity</strong>; stateful servers need sticky routing to reach the same instance.
- State in stateless systems lives in <strong class="kw">external stores</strong> or the client; state in stateful systems lives in server memory.
- Stateless architectures scale <strong class="kw">horizontally</strong> with ease, while stateful ones need state replication to scale out.
- A crashed stateless server loses nothing, but a crashed stateful server can drop an active <strong class="kw">session</strong>.

## When to Use Each

**Stateless**

- **Public REST APIs**: Decoupling servers from client identity lets any instance handle any request, simplifying horizontal scaling.
- **Auto-scaling microservices**: Instances can be spun up or torn down freely without needing to preserve or migrate in-memory state.
- **CDN and edge functions**: Any edge node can serve a request without needing shared context from a prior request.

**Stateful**

- **Real-time multiplayer games**: Low-latency in-memory state per player session is essential and can't be re-fetched on every tick.
- **Database transactions**: A transaction requires a continuous connection and context that must persist across multiple operations.
- **Interactive terminal sessions**: SSH or REPL sessions depend on maintaining command history and environment state across the connection.
