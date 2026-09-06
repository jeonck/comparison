---
title: "Stateful vs Stateless: Where the Session Lives"
date: 2026-09-06T09:51:46.028663+09:00
tags: ["stateful-stateless", "system-design", "scalability", "networking"]
---
## Overview

This comparison covers whether a server or protocol retains <strong class="kw">session state</strong> between requests, or treats every request as a fully <strong class="kw">self-contained</strong> unit with no memory of prior ones. The choice determines how you scale, fail over, and route traffic across instances.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="32" text-anchor="middle" font-size="16" style="fill:var(--primary)">Stateful</text><text x="480" y="32" text-anchor="middle" font-size="16" style="fill:var(--primary)">Stateless</text><rect x="40" y="60" width="110" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="90" text-anchor="middle" font-size="13" style="fill:var(--content)">Client</text><path d="M150 85 L230 85" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="230" y="60" width="120" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="290" y="84" text-anchor="middle" font-size="12" style="fill:var(--content)">Server A</text><text x="290" y="100" text-anchor="middle" font-size="10" style="fill:var(--secondary)">session: id=42</text><path d="M290 110 L290 150" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,3" marker-end="url(#arrowN)"/><text x="330" y="135" font-size="10" style="fill:var(--secondary)">must return</text><text x="330" y="148" font-size="10" style="fill:var(--secondary)">to same server</text><rect x="230" y="160" width="120" height="50" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,3"/><text x="290" y="189" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Server B</text><text x="290" y="204" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no session data</text><path d="M95 110 L95 200 L230 200" style="stroke:var(--compare-a)" stroke-width="1.5" fill="none" marker-end="url(#arrowA)"/><line x1="20" y1="260" x2="620" y2="260" style="stroke:var(--border)" stroke-width="1"/><rect x="360" y="60" width="110" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="415" y="90" text-anchor="middle" font-size="13" style="fill:var(--content)">Client</text><text x="415" y="106" text-anchor="middle" font-size="9" style="fill:var(--secondary)">carries token</text><path d="M470 85 L540 85" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="540 " y="60" width="70" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="575" y="90" text-anchor="middle" font-size="11" style="fill:var(--content)">Server</text><path d="M470 85 L540 160" style="stroke:var(--compare-b)" stroke-width="1.5" fill="none" marker-end="url(#arrowB)"/><rect x="540" y="150" width="70" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="575" y="180" text-anchor="middle" font-size="11" style="fill:var(--content)">Server</text><text x="415" y="200" font-size="10" style="fill:var(--secondary)">any server can</text><text x="415" y="213" font-size="10" style="fill:var(--secondary)">handle the request</text><text x="320" y="300" text-anchor="middle" font-size="12" style="fill:var(--content)">Stateful: server pins session context and routing depends on it</text><text x="320" y="325" text-anchor="middle" font-size="12" style="fill:var(--content)">Stateless: request carries all context, any node can serve it</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-b)"/></marker><marker id="arrowN" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--border)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Stateful | Stateless |
| --- | --- | --- |
| Request context | Server retains prior interaction data across requests | Each request carries all context needed to process it |
| Session storage | Held in server memory or local session store | None on server; state lives in client token or database |
| Routing requirement | Requests must reach the same server (sticky sessions) | Any server instance can handle any request |
| Scaling model | Vertical or sticky-session horizontal scaling only | Trivial horizontal scaling, load balance freely |
| Failure recovery | Server crash loses in-memory session unless replicated | Server crash has no session impact, retry hits any node |
| Client design | Client can be thin, server tracks progress | Client or token must resend full context each call |
| Typical examples | Database connections, WebSocket sessions, FTP | REST APIs, HTTP with JWT, DNS lookups |

## Key Differences

- Stateful servers keep <strong class="kw">session memory</strong>; stateless servers keep none between calls
- Stateless systems need <strong class="kw">no sticky routing</strong>, simplifying load balancers
- Stateful failover requires <strong class="kw">session replication</strong> to avoid data loss
- Stateless designs push state into the <strong class="kw">client or token</strong> instead of the server
- Horizontal scaling is <strong class="kw">near-free</strong> for stateless architectures

## When to Use Each

**Stateful**

- **Real-time multiplayer games**: Low-latency in-memory game state per connection benefits from a persistent stateful server.
- **Database transactions**: Multi-step transactions need the connection to remember uncommitted work until commit or rollback.
- **Streaming media sessions**: Protocols like RTSP or WebSockets keep a live connection open with ongoing playback state.

**Stateless**

- **Public REST APIs**: Stateless requests let any server or region handle calls without session affinity.
- **Auto-scaling web services**: New instances can join a pool instantly since no server holds unique session data.
- **Serverless functions**: Ephemeral compute like Lambda fits stateless design since instances are created and destroyed freely.
