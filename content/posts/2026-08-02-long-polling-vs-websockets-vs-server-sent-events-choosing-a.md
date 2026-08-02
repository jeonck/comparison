---
title: "Long Polling vs WebSockets vs Server-Sent Events: Choosing a Real-Time Update Strategy"
date: 2026-08-02T23:49:39.625826+09:00
tags: ["websockets", "long-polling", "server-sent-events", "real-time"]
---
## Overview

Long polling, WebSockets, and Server-Sent Events are three techniques for pushing live updates from a server to a browser without the client repeatedly refreshing the page. <strong class="kw">WebSockets</strong> open a single full-duplex socket for two-way messaging, while <strong class="kw">Server-Sent Events</strong> keep one HTTP connection open for one-way server-to-client streaming; long polling predates both and fakes real-time updates by re-issuing HTTP requests. Picking the right one depends on whether you need bidirectional traffic, binary data, or just simple broadcast updates.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="40" y="50" style="fill:var(--content)" font-size="13">Client</text><line x1="40" y1="70" x2="600" y2="70" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><line x1="40" y1="290" x2="600" y2="290" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="40" y="312" style="fill:var(--content)" font-size="13">Server</text><line x1="110" y1="90" x2="110" y2="150" style="stroke:var(--border)" stroke-width="2"/><polygon points="104,105 116,105 110,117" style="fill:var(--border)"/><polygon points="104,135 116,135 110,123" style="fill:var(--border)"/><line x1="110" y1="168" x2="110" y2="228" style="stroke:var(--border)" stroke-width="2"/><polygon points="104,183 116,183 110,195" style="fill:var(--border)"/><polygon points="104,213 116,213 110,201" style="fill:var(--border)"/><line x1="110" y1="246" x2="110" y2="270" style="stroke:var(--border)" stroke-width="2"/><text x="110" y="333" text-anchor="middle" style="fill:var(--secondary)" font-size="13">Long Polling</text><text x="110" y="349" text-anchor="middle" style="fill:var(--secondary)" font-size="10">reconnects every cycle</text><circle cx="320" cy="90" r="4" style="fill:var(--compare-a)"/><line x1="320" y1="90" x2="320" y2="270" style="stroke:var(--compare-a)" stroke-width="2"/><polygon points="314,125 326,125 320,113" style="fill:var(--compare-a)"/><polygon points="314,145 326,145 320,157" style="fill:var(--compare-a)"/><polygon points="314,175 326,175 320,163" style="fill:var(--compare-a)"/><polygon points="314,195 326,195 320,207" style="fill:var(--compare-a)"/><polygon points="314,225 326,225 320,213" style="fill:var(--compare-a)"/><polygon points="314,245 326,245 320,257" style="fill:var(--compare-a)"/><text x="320" y="333" text-anchor="middle" style="fill:var(--primary)" font-size="13">WebSockets</text><text x="320" y="349" text-anchor="middle" style="fill:var(--secondary)" font-size="10">full-duplex, one socket</text><circle cx="530" cy="90" r="4" style="fill:var(--compare-b)"/><line x1="530" y1="90" x2="530" y2="270" style="stroke:var(--compare-b)" stroke-width="2"/><polygon points="524,140 536,140 530,128" style="fill:var(--compare-b)"/><polygon points="524,190 536,190 530,178" style="fill:var(--compare-b)"/><polygon points="524,240 536,240 530,228" style="fill:var(--compare-b)"/><text x="530" y="333" text-anchor="middle" style="fill:var(--primary)" font-size="13">Server-Sent Events</text><text x="530" y="349" text-anchor="middle" style="fill:var(--secondary)" font-size="10">one-way stream</text></svg>
</div>

## Comparison Table

| Aspect | Long Polling | WebSockets | Server-Sent Events |
| --- | --- | --- | --- |
| Connection setup | Client sends a normal HTTP GET; server holds it open until data is ready or a timeout, then the client immediately re-requests | Client sends an HTTP Upgrade request; the connection switches to a persistent ws:// socket | Client opens a single HTTP GET with Accept: text/event-stream; server keeps that response stream open |
| Underlying protocol | Plain HTTP request/response, repeated | Dedicated ws/wss protocol after the handshake | Standard HTTP with a chunked/streaming response |
| Data direction | Bidirectional, but each direction needs its own HTTP exchange | Full-duplex over one socket; either side can send anytime | Server to client only |
| Client-to-server messages | Sent as the next poll request | Sent directly over the open socket | Requires a separate ordinary HTTP request, e.g. fetch |
| Update latency | Near real-time, bounded by the request/re-request round trip | Lowest; messages are pushed the instant they're sent | Low; comparable to WebSockets for server-to-client pushes |
| Reconnection handling | Manual: app code re-issues the request after every response or timeout | Manual: app must implement its own reconnect/backoff | Automatic: built into the EventSource API with Last-Event-ID resume |
| Server resource footprint | Many short-lived held requests; can spike thread/connection usage under load | One long-lived socket per client; efficient per message but needs socket-aware infra | One long-lived HTTP connection per client; works with standard HTTP servers and proxies |
| Typical use cases | Legacy fallback where WebSockets aren't available, low-frequency updates | Chat, multiplayer games, collaborative editing | Live feeds, notifications, stock tickers, dashboards |

## Key Differences

- Only <strong class="kw">WebSockets</strong> support true bidirectional messaging over a single connection; Long Polling and SSE both need a separate request for client-to-server data.
- <strong class="kw">Server-Sent Events</strong> ship built-in reconnection and event IDs via the EventSource API, while WebSockets and Long Polling require hand-rolled reconnect logic.
- Long Polling reuses plain HTTP semantics with no protocol upgrade, making it the easiest to route through legacy infrastructure but the least efficient under frequent updates.
- WebSockets can carry <strong class="kw">binary frames</strong>, whereas SSE is restricted to UTF-8 text events.

## When to Use Each

**Long Polling**

- **Legacy or Restrictive Networks**: Since it reuses plain HTTP request/response with no protocol upgrade, long polling passes through proxies and firewalls that may block WebSocket handshakes.
- **Infrequent Updates**: When updates are rare, the round-trip delay of re-issuing requests is an acceptable tradeoff against building persistent-connection infrastructure.
- **Fallback for Unsupported Clients**: It serves as a simple degrade path when a client or network can't sustain a WebSocket or SSE connection.

**WebSockets**

- **Chat and Multiplayer Games**: Full-duplex messaging over one socket lets either side push data the instant it's ready, which one-way SSE can't do for client input.
- **Collaborative Editing**: Frequent, low-latency updates in both directions fit a single persistent socket better than repeated HTTP exchanges.
- **Binary Data Transfer**: Only WebSockets carry binary frames, whereas SSE is restricted to UTF-8 text events.

**Server-Sent Events**

- **One-Way Live Feeds**: Notifications, stock tickers, and dashboards only need server-to-client pushes, which is exactly what SSE provides without WebSocket's added complexity.
- **Automatic Reconnection Needed**: The EventSource API's built-in reconnect and Last-Event-ID resume avoid hand-rolling the reconnect logic WebSockets and long polling require.
- **Standard HTTP Infrastructure**: Running over ordinary HTTP means SSE works with existing proxies and servers without needing socket-aware infrastructure.
