---
title: "TCP vs UDP: Reliable Streams vs Fast Datagrams"
date: 2026-08-01T20:00:00+09:00
tags: ["networking", "tcp", "udp", "transport-layer"]
---
## Overview

TCP and UDP are the two core transport-layer protocols used to move data between hosts, but they trade reliability for speed in opposite directions. TCP prioritizes <strong class="kw">reliable delivery</strong> through handshakes, acknowledgments, and retransmission, while UDP prioritizes <strong class="kw">low-latency delivery</strong> by sending datagrams with no setup or delivery guarantees. Choosing between them shapes how an application handles packet loss, ordering, and throughput.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-b)"/></marker></defs><text x="180" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">TCP</text><text x="490" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">UDP</text><line x1="340" y1="45" x2="340" y2="330" stroke-dasharray="4 4" style="stroke:var(--border)"/><line x1="100" y1="50" x2="100" y2="330" style="stroke:var(--border)"/><line x1="260" y1="50" x2="260" y2="330" style="stroke:var(--border)"/><line x1="400" y1="50" x2="400" y2="330" style="stroke:var(--border)"/><line x1="580" y1="50" x2="580" y2="330" style="stroke:var(--border)"/><circle cx="100" cy="50" r="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="260" cy="50" r="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="400" cy="50" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="580" cy="50" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="100" y="42" text-anchor="middle" font-size="10" style="fill:var(--content)">Client</text><text x="260" y="42" text-anchor="middle" font-size="10" style="fill:var(--content)">Server</text><text x="400" y="42" text-anchor="middle" font-size="10" style="fill:var(--content)">Sender</text><text x="580" y="42" text-anchor="middle" font-size="10" style="fill:var(--content)">Receiver</text><line x1="100" y1="65" x2="260" y2="65" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="60" text-anchor="middle" font-size="11" style="fill:var(--content)">SYN</text><line x1="260" y1="90" x2="100" y2="90" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="85" text-anchor="middle" font-size="11" style="fill:var(--content)">SYN-ACK</text><line x1="100" y1="115" x2="260" y2="115" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="110" text-anchor="middle" font-size="11" style="fill:var(--content)">ACK</text><text x="180" y="138" text-anchor="middle" font-size="10" font-style="italic" style="fill:var(--secondary)">connection established</text><line x1="100" y1="163" x2="260" y2="163" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="158" text-anchor="middle" font-size="11" style="fill:var(--content)">Data</text><line x1="260" y1="188" x2="100" y2="188" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="183" text-anchor="middle" font-size="11" style="fill:var(--content)">ACK</text><line x1="100" y1="213" x2="260" y2="213" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="208" text-anchor="middle" font-size="11" style="fill:var(--content)">Data</text><line x1="260" y1="238" x2="100" y2="238" stroke-width="1.5" marker-end="url(#arrowA)" style="stroke:var(--compare-a)"/><text x="180" y="233" text-anchor="middle" font-size="11" style="fill:var(--content)">ACK</text><text x="180" y="265" text-anchor="middle" font-size="10" style="fill:var(--secondary)">FIN / ACK teardown</text><line x1="400" y1="65" x2="580" y2="65" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="490" y="60" text-anchor="middle" font-size="11" style="fill:var(--content)">Datagram</text><line x1="400" y1="90" x2="580" y2="90" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="490" y="85" text-anchor="middle" font-size="11" style="fill:var(--content)">Datagram</text><line x1="400" y1="115" x2="500" y2="115" stroke-dasharray="4 3" style="stroke:var(--border)"/><line x1="496" y1="111" x2="504" y2="119" style="stroke:var(--secondary)"/><line x1="504" y1="111" x2="496" y2="119" style="stroke:var(--secondary)"/><text x="490" y="131" text-anchor="middle" font-size="10" style="fill:var(--secondary)">lost, no retry</text><line x1="400" y1="153" x2="580" y2="153" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="490" y="148" text-anchor="middle" font-size="11" style="fill:var(--content)">Datagram</text><line x1="400" y1="178" x2="580" y2="178" stroke-width="1.5" marker-end="url(#arrowB)" style="stroke:var(--compare-b)"/><text x="490" y="173" text-anchor="middle" font-size="11" style="fill:var(--content)">Datagram</text><text x="490" y="205" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no ACKs, no ordering</text></svg>
</div>

## Comparison Table

| Aspect | TCP | UDP |
| --- | --- | --- |
| Connection setup | Three-way handshake (SYN, SYN-ACK, ACK) establishes a stateful connection before any data moves | No handshake — sender transmits datagrams immediately with no prior negotiation |
| Delivery guarantee | Guaranteed via sequence numbers and acknowledgments; lost segments are detected and resent | Best-effort only; lost packets vanish silently with no notification to either side |
| Ordering | Segments are reassembled in the original order regardless of arrival sequence | No ordering guarantee; packets are delivered to the application in whatever order they arrive |
| Flow & congestion control | Dynamic window sizing and congestion-avoidance algorithms throttle the sender to match network capacity | None; the application sends at whatever rate it chooses, independent of network conditions |
| Error handling | Checksum plus automatic retransmission recovers corrupted or missing segments | Checksum only; corrupted or missing packets are simply dropped, not recovered |
| Overhead & latency | Larger 20+ byte header and handshake/ACK round trips add processing and latency | Minimal 8-byte header and no round trips keep per-packet overhead and latency low |
| Connection teardown | Explicit four-way FIN/ACK exchange formally closes the connection on both sides | No connection state exists, so transmission simply stops with nothing to tear down |

## Key Differences

- TCP is <strong class="kw">connection-oriented</strong>, requiring a handshake before data flows, while UDP is connectionless
- TCP guarantees <strong class="kw">reliable delivery</strong> through acknowledgments and retransmission; UDP offers none
- TCP performs <strong class="kw">congestion control</strong> to avoid overwhelming the network; UDP has no such mechanism
- UDP's minimal <strong class="kw">header overhead</strong> gives it consistently lower latency than TCP
- TCP preserves <strong class="kw">packet ordering</strong> end-to-end; UDP delivers packets in whatever order they arrive

## When to Use Each

**TCP**

- **Web & API traffic (HTTP/HTTPS)**: Requests and responses must arrive complete and in order, making TCP's reliability essential
- **File transfer (FTP, SFTP)**: Every byte must arrive intact, so retransmission of lost segments is worth the added latency
- **Database connections**: Queries and results demand guaranteed, ordered delivery to keep transactions consistent
- **Email transmission (SMTP)**: Message integrity matters more than speed, favoring TCP's reliability guarantees

**UDP**

- **DNS lookups**: A single small request/response benefits from UDP's low overhead, with the application retrying on its own if needed
- **Live video & voice calls**: A dropped frame is preferable to the added latency of waiting for retransmission
- **Online multiplayer gaming**: Stale position data is worthless, so UDP's speed matters more than guaranteed delivery
- **IoT sensor telemetry**: High-frequency readings can tolerate occasional loss, and UDP's low overhead suits constrained devices
