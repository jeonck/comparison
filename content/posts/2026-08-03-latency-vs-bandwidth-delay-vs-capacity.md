---
title: "Latency vs Bandwidth: Delay vs Capacity"
date: 2026-08-03T08:12:18.126489+09:00
tags: ["networking", "latency", "bandwidth", "performance"]
---
## Overview

Latency and bandwidth both describe network performance, but they measure completely different things: <strong class="kw">latency</strong> is how long a single piece of data takes to travel from source to destination, while <strong class="kw">bandwidth</strong> is how much data can move through the connection per second. A link can have huge bandwidth and still feel laggy, or tiny bandwidth and still respond instantly — understanding which one is limiting you determines whether the fix is a faster link or a shorter path.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="20" y="35" font-size="16" font-weight="bold" style="fill:var(--primary)">LATENCY</text><rect x="40" y="70" width="90" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="85" y="100" text-anchor="middle" font-size="12" style="fill:var(--content)">Sender</text><rect x="510" y="70" width="90" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="555" y="100" text-anchor="middle" font-size="12" style="fill:var(--content)">Receiver</text><line x1="130" y1="95" x2="505" y2="95" style="stroke:var(--compare-a)" stroke-width="2"/><polygon points="505,90 505,100 515,95" style="fill:var(--compare-a)"/><circle cx="280" cy="95" r="6" style="fill:var(--compare-a)"/><circle cx="320" cy="140" r="14" style="fill:none;stroke:var(--secondary)" stroke-width="1.5"/><line x1="320" y1="140" x2="320" y2="129" style="stroke:var(--secondary)" stroke-width="1.5"/><line x1="320" y1="140" x2="329" y2="143" style="stroke:var(--secondary)" stroke-width="1.5"/><text x="320" y="168" text-anchor="middle" font-size="11" style="fill:var(--secondary)">one packet, ~50 ms one-way delay</text><line x1="20" y1="188" x2="620" y2="188" style="stroke:var(--border)" stroke-width="1"/><text x="20" y="210" font-size="16" font-weight="bold" style="fill:var(--primary)">BANDWIDTH</text><rect x="40" y="235" width="90" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="85" y="265" text-anchor="middle" font-size="12" style="fill:var(--content)">Sender</text><rect x="510" y="235" width="90" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="555" y="265" text-anchor="middle" font-size="12" style="fill:var(--content)">Receiver</text><rect x="130" y="250" width="380" height="20" rx="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><circle cx="165" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="220" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="275" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="330" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="385" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="440" cy="260" r="5" style="fill:var(--compare-b)"/><circle cx="475" cy="260" r="5" style="fill:var(--compare-b)"/><text x="320" y="305" text-anchor="middle" font-size="11" style="fill:var(--secondary)">many bits in parallel, ~1 Gbps throughput</text></svg>
</div>

## Comparison Table

| Aspect | Latency | Bandwidth |
| --- | --- | --- |
| What it measures | Time for one unit of data to travel from source to destination | Volume of data that can pass through the link per unit of time |
| Unit of measurement | Milliseconds (ms) | Bits per second (Mbps, Gbps) |
| Primary determinant | Physical distance, propagation medium, and number of hops/routers | Link capacity — cable/fiber type, modulation, or channel width |
| Pipe analogy | Length of the pipe (travel time) | Diameter of the pipe (volume capacity) |
| User-facing symptom when poor | Sluggish response or lag in real-time interaction | Slow downloads or buffering during bulk transfer |
| Standard measurement tool | ping, traceroute (round-trip time) | speedtest, iperf (throughput) |
| How it's improved | Shorten the path — CDNs, edge servers, fewer hops | Upgrade link capacity or add parallel channels |
| Effect of upgrading the other | Unaffected by adding more bandwidth to the link | Unaffected by reducing physical distance alone |

## Key Differences

- High <strong class="kw">bandwidth</strong> does not shrink latency — a 10 Gbps satellite link can still have a long round-trip delay.
- <strong class="kw">Latency</strong> is bounded by the speed of light and physical distance, not by faster hardware alone.
- Bandwidth determines how much data fits through the connection per second, which matters most for <strong class="kw">throughput</strong>-heavy tasks.
- Interactive apps like <strong class="kw">gaming</strong> or video calls suffer more from latency than from limited bandwidth.
- <strong class="kw">Round-trip time</strong> combines outbound and return latency and is what users actually perceive as lag.

## When to Use Each

**Latency**

- **Real-time gaming**: Every extra millisecond of latency directly delays input response, regardless of how much bandwidth is available.
- **VoIP and video calls**: Low latency keeps conversation natural; high latency causes talk-over and awkward pauses even on fast connections.
- **High-frequency trading**: Microseconds of round-trip delay determine execution priority, making latency the dominant cost to optimize.
- **Remote shell / SSH sessions**: Keystroke echo feels instant only when latency is low, since each keystroke is a tiny, latency-bound round trip.

**Bandwidth**

- **Large file downloads**: Total transfer time is dominated by how many bits per second the pipe can push, not the initial delay.
- **Bulk video streaming**: Sustained high-resolution playback needs enough throughput to keep the buffer filled faster than it drains.
- **Backup and replication jobs**: Moving large datasets between sites is throughput-bound, so more bandwidth shortens the job directly.
- **Many concurrent users on one link**: Shared bandwidth capacity, not per-request delay, determines whether the connection can serve everyone at once.
