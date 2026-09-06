---
title: "Unary vs Streaming RPC: One Request-Response vs Continuous Message Flow"
date: 2026-09-06T10:04:00.793438+09:00
tags: ["grpc", "rpc", "streaming", "api-design"]
---
## Overview

Unary and streaming are the two call shapes gRPC (and similar RPC frameworks) support over HTTP/2. A <strong class="kw">unary call</strong> behaves like a classic function call — one request in, one response out, then done — while a <strong class="kw">streaming call</strong> keeps the connection open so either side can send multiple messages over time. The choice affects latency, backpressure handling, and how errors surface mid-exchange.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Unary RPC</text><text x="480" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Streaming RPC</text><rect x="110" y="50" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Client</text><rect x="110" y="280" width="100" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="305" text-anchor="middle" style="fill:var(--content)" font-size="12">Server</text><line x1="140" y1="90" x2="140" y2="278" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="140,278 135,268 145,268" style="fill:var(--compare-a)"/><text x="98" y="185" text-anchor="middle" style="fill:var(--content)" font-size="11">request</text><line x1="180" y1="278" x2="180" y2="90" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="180,90 175,100 185,100" style="fill:var(--compare-a)"/><text x="224" y="185" text-anchor="middle" style="fill:var(--content)" font-size="11">response</text><text x="160" y="345" text-anchor="middle" style="fill:var(--secondary)" font-size="11">one call = one request + one response, then closed</text><rect x="430" y="50" width="100" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="75" text-anchor="middle" style="fill:var(--content)" font-size="12">Client</text><rect x="430" y="280" width="100" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="305" text-anchor="middle" style="fill:var(--content)" font-size="12">Server</text><line x1="480" y1="90" x2="480" y2="278" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><line x1="450" y1="120" x2="450" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="450,150 445,140 455,140" style="fill:var(--compare-b)"/><line x1="450" y1="165" x2="450" y2="195" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="450,195 445,185 455,185" style="fill:var(--compare-b)"/><line x1="450" y1="210" x2="450" y2="240" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="450,240 445,230 455,230" style="fill:var(--compare-b)"/><text x="406" y="188" text-anchor="middle" style="fill:var(--content)" font-size="11">msg 1..N</text><text x="480" y="345" text-anchor="middle" style="fill:var(--secondary)" font-size="11">one call = many messages over a long-lived connection</text></svg>
</div>

## Comparison Table

| Aspect | Unary RPC | Streaming RPC |
| --- | --- | --- |
| Call initiation | Client opens the call and immediately sends the complete request | Client opens the call, which may send zero, one, or many messages before or while reading responses |
| Client-to-server messages | Exactly one request message per call | One (server-streaming) or many (client-streaming, bidi) messages per call |
| Server-to-client messages | Exactly one response message per call | One (client-streaming) or many (server-streaming, bidi) messages per call |
| Flow control | Not needed — a single frame per direction fits within normal HTTP/2 windows | HTTP/2 flow-control windows and backpressure govern how fast messages can be sent |
| Connection/call lifetime | Logically short-lived: opens and closes within one round trip | Can stay open for the duration of a long-running exchange, sometimes indefinitely |
| Latency and overhead | Full connection/setup overhead paid per call since each call is independent | Setup overhead amortized across many messages, lowering per-message latency |
| Termination and errors | A single status code ends the call atomically — it either succeeded or failed | Status is sent only when the stream closes; errors can occur mid-stream after partial data was already delivered |

## Key Differences

- Unary sends exactly one <strong class="kw">request</strong> and gets exactly one response, while streaming allows either side to send a <strong class="kw">sequence</strong> of messages over the same call
- Streaming relies on HTTP/2 <strong class="kw">flow control</strong> to manage backpressure across many frames; unary has nothing to manage
- A streaming call's <strong class="kw">connection</strong> stays open far longer than a unary call's brief request-response window
- Unary calls fail or succeed as a single <strong class="kw">atomic</strong> unit; streaming calls can deliver partial results before an error terminates them
- Streaming amortizes per-call <strong class="kw">overhead</strong> across many messages, cutting per-message latency compared to repeated unary calls

## When to Use Each

**Unary RPC**

- **Simple CRUD operations**: A single lookup or update maps naturally to one request and one response with no ongoing state.
- **REST-like API parity**: Unary calls mirror the request/response semantics developers already expect from HTTP APIs, easing adoption.
- **Idempotent, retryable actions**: A self-contained request/response pair is easy to retry safely on failure or timeout.

**Streaming RPC**

- **Real-time server push**: Server-streaming lets a service push live updates (e.g. price ticks, logs) without the client polling repeatedly.
- **Large payload chunking**: Client-streaming lets a client upload large data (e.g. a file) as a sequence of chunks instead of one huge message.
- **Bidirectional interactive exchange**: Bidi streaming supports chat, telemetry, or negotiation protocols where both sides send messages independently and continuously.
