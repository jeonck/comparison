---
title: "Unicast vs Multicast: One-to-One vs One-to-Many Delivery"
date: 2026-08-01T20:07:00+09:00
tags: ["networking", "unicast", "multicast", "routing"]
---
## Overview

Unicast and multicast are IP transmission models that differ in how a sender's data reaches its destinations. Unicast sends a <strong class="kw">dedicated copy</strong> to each individual recipient, while multicast sends a <strong class="kw">single stream</strong> that network devices replicate only where delivery paths actually diverge. The choice shapes bandwidth usage, routing complexity, and how receivers subscribe to traffic.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Unicast</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Multicast</text><rect x="50" y="150" width="80" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="90" y="174" text-anchor="middle" style="fill:var(--content)" font-size="13">Sender</text><rect x="230" y="60" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="265" y="82" text-anchor="middle" style="fill:var(--content)" font-size="12">R1</text><rect x="230" y="152" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="265" y="174" text-anchor="middle" style="fill:var(--content)" font-size="12">R2</text><rect x="230" y="245" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="265" y="267" text-anchor="middle" style="fill:var(--content)" font-size="12">R3</text><line x1="130" y1="162" x2="230" y2="78" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="130" y1="170" x2="230" y2="170" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="130" y1="178" x2="230" y2="262" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="310" text-anchor="middle" style="fill:var(--secondary)" font-size="11">3 separate packet copies</text><text x="160" y="324" text-anchor="middle" style="fill:var(--secondary)" font-size="11">sent end-to-end</text><rect x="370" y="150" width="80" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="410" y="174" text-anchor="middle" style="fill:var(--content)" font-size="13">Sender</text><line x1="450" y1="170" x2="484" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="490" cy="170" r="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><line x1="496" y1="168" x2="555" y2="78" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="496" y1="170" x2="555" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="496" y1="172" x2="555" y2="262" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="555" y="60" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="82" text-anchor="middle" style="fill:var(--content)" font-size="12">R1</text><rect x="555" y="152" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="174" text-anchor="middle" style="fill:var(--content)" font-size="12">R2</text><rect x="555" y="245" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="267" text-anchor="middle" style="fill:var(--content)" font-size="12">R3</text><text x="480" y="310" text-anchor="middle" style="fill:var(--secondary)" font-size="11">single stream, replicated</text><text x="480" y="324" text-anchor="middle" style="fill:var(--secondary)" font-size="11">only at the branch point</text></svg>
</div>

## Comparison Table

| Aspect | Unicast | Multicast |
| --- | --- | --- |
| Addressing model | One-to-one; packet is addressed to a single destination IP | One-to-many; packet is addressed to a shared multicast group IP |
| Sender behavior | Sends a separate copy of the data for each recipient | Sends one copy regardless of how many receivers exist |
| Network replication | No replication; each copy travels its own end-to-end path | Routers/switches replicate the packet only at points where paths diverge |
| Receiver participation | Implicit; determined solely by the destination address | Explicit; hosts must join the group (IGMP/MLD membership) |
| Bandwidth scaling | Grows linearly with the number of receivers | Stays roughly constant on the sender's link as receivers grow |
| Routing requirements | Standard unicast routing (OSPF, BGP, static routes) | Requires multicast-aware routing (PIM-SM/DM plus IGMP) |
| Delivery reliability | Can run over TCP for guaranteed, ordered delivery | Almost always UDP-based, with no built-in delivery guarantee |
| Typical use cases | Web browsing, file transfer, email, SSH | Live video/audio streaming, market data feeds, routing protocol updates |

## Key Differences

- Unicast requires a separate <strong class="kw">packet copy</strong> per receiver; multicast needs only one.
- Multicast pushes replication into the <strong class="kw">network fabric</strong> instead of the sender.
- Multicast receivers must <strong class="kw">explicitly join</strong> a group via IGMP before traffic arrives.
- Unicast bandwidth <strong class="kw">scales linearly</strong> with recipients; multicast stays flat.
- Multicast typically rides over <strong class="kw">UDP</strong>, sacrificing delivery guarantees for efficiency.

## When to Use Each

**Unicast**

- **Point-to-Point Transfers**: Each recipient needs distinct, personalized data, so a dedicated copy per connection makes sense.
- **Interactive Sessions**: SSH, HTTPS, and RPC calls require bidirectional, per-client state that only a one-to-one path provides.
- **Guaranteed Delivery**: Layering TCP over unicast gives retransmission and ordering guarantees multicast can't offer natively.

**Multicast**

- **Live Streaming to Many**: One video feed reaches thousands of viewers without duplicating the stream at the source.
- **Financial Market Data**: Trading systems need simultaneous, low-latency delivery of the same price feed to many subscribers.
- **Routing Protocol Updates**: OSPF and EIGRP send hello and update packets to all routers on a segment using well-known multicast addresses.
