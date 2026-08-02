---
title: "Full Duplex vs Half Duplex: Simultaneous vs Alternating Communication"
date: 2026-08-03T08:10:01.134289+09:00
tags: ["networking", "ethernet", "data-transmission", "protocols"]
---
## Overview

Full duplex and half duplex describe how a communication link handles data flowing in both directions. A <strong class="kw">full duplex</strong> link sends and receives at the same time over independent paths, while a <strong class="kw">half duplex</strong> link shares a single channel and must alternate between sending and receiving. The distinction determines whether devices collide, how much of the link's bandwidth is usable, and how much delay is added when a device switches from listening to talking.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox='0 0 640 360' xmlns='http://www.w3.org/2000/svg'><line x1='320' y1='20' x2='320' y2='340' style='stroke:var(--border)' stroke-width='1.5' stroke-dasharray='6 4'/><text x='160' y='40' text-anchor='middle' style='fill:var(--primary)' font-size='20' font-weight='bold'>FULL DUPLEX</text><text x='480' y='40' text-anchor='middle' style='fill:var(--primary)' font-size='20' font-weight='bold'>HALF DUPLEX</text><circle cx='90' cy='150' r='28' style='fill:var(--compare-a-soft);stroke:var(--compare-a)' stroke-width='2'/><text x='90' y='156' text-anchor='middle' style='fill:var(--content)' font-size='16'>A</text><circle cx='270' cy='150' r='28' style='fill:var(--compare-a-soft);stroke:var(--compare-a)' stroke-width='2'/><text x='270' y='156' text-anchor='middle' style='fill:var(--content)' font-size='16'>B</text><line x1='120' y1='125' x2='232' y2='125' style='stroke:var(--compare-a)' stroke-width='3'/><polygon points='232,125 220,119 220,131' style='fill:var(--compare-a)'/><line x1='240' y1='175' x2='128' y2='175' style='stroke:var(--compare-a)' stroke-width='3'/><polygon points='128,175 140,169 140,181' style='fill:var(--compare-a)'/><text x='180' y='115' text-anchor='middle' style='fill:var(--secondary)' font-size='12'>send</text><text x='180' y='195' text-anchor='middle' style='fill:var(--secondary)' font-size='12'>send</text><text x='180' y='250' text-anchor='middle' style='fill:var(--secondary)' font-size='13'>both directions active at once</text><text x='350' y='100' style='fill:var(--secondary)' font-size='12'>t1</text><circle cx='400' cy='110' r='22' style='fill:var(--compare-b-soft);stroke:var(--compare-b)' stroke-width='2'/><text x='400' y='115' text-anchor='middle' style='fill:var(--content)' font-size='14'>A</text><circle cx='580' cy='110' r='22' style='fill:var(--compare-b-soft);stroke:var(--compare-b)' stroke-width='2'/><text x='580' y='115' text-anchor='middle' style='fill:var(--content)' font-size='14'>B</text><line x1='424' y1='110' x2='552' y2='110' style='stroke:var(--compare-b)' stroke-width='3'/><polygon points='552,110 542,105 542,115' style='fill:var(--compare-b)'/><line x1='552' y1='122' x2='424' y2='122' style='stroke:var(--border)' stroke-width='2' stroke-dasharray='4 3'/><text x='350' y='222' style='fill:var(--secondary)' font-size='12'>t2</text><circle cx='400' cy='232' r='22' style='fill:var(--compare-b-soft);stroke:var(--compare-b)' stroke-width='2'/><text x='400' y='237' text-anchor='middle' style='fill:var(--content)' font-size='14'>A</text><circle cx='580' cy='232' r='22' style='fill:var(--compare-b-soft);stroke:var(--compare-b)' stroke-width='2'/><text x='580' y='237' text-anchor='middle' style='fill:var(--content)' font-size='14'>B</text><line x1='552' y1='232' x2='424' y2='232' style='stroke:var(--compare-b)' stroke-width='3'/><polygon points='424,232 434,227 434,237' style='fill:var(--compare-b)'/><line x1='424' y1='244' x2='552' y2='244' style='stroke:var(--border)' stroke-width='2' stroke-dasharray='4 3'/><text x='480' y='300' text-anchor='middle' style='fill:var(--secondary)' font-size='13'>one direction at a time, then switches</text></svg>
</div>

## Comparison Table

| Aspect | Full Duplex | Half Duplex |
| --- | --- | --- |
| Data flow direction | Both directions simultaneously | One direction at a time, alternating |
| Physical channel requirement | Separate transmit and receive paths (or divided by frequency/time) | Single shared channel used by all parties |
| Collision handling | No collisions possible; each direction has its own path | Collisions possible if two devices transmit at once; needs arbitration (e.g. CSMA/CD) |
| Turnaround delay | None; a device can send and receive without waiting | Present; a device must wait for the channel to free up before switching |
| Effective throughput | Up to full bandwidth in each direction concurrently | Bandwidth shared across time, so effective throughput drops under contention |
| Hardware/protocol complexity | Requires dedicated transceivers or duplexing scheme (FDD/TDD) | Simpler transceiver; relies on a media-access protocol to coordinate turns |
| Common examples | Switched Ethernet (twisted pair), telephone calls, fiber links | Walkie-talkies, old hub-based Ethernet, CB radio, shared Wi-Fi channel |

## Key Differences

- Full duplex relies on <strong class="kw">separate paths</strong> for send and receive, while half duplex forces both directions onto one shared medium
- Half duplex needs a <strong class="kw">media-access protocol</strong> like CSMA/CD to prevent two devices transmitting at once
- Switching directions in half duplex introduces <strong class="kw">turnaround delay</strong> that full duplex never has
- Full duplex effectively <strong class="kw">doubles usable capacity</strong> per link compared to a half duplex link of the same rated speed

## When to Use Each

**Full Duplex**

- **Modern switched Ethernet**: Point-to-point switch links use full duplex so both ends can send and receive at line rate with zero collisions.
- **Voice and video calls**: Real-time two-way conversation needs simultaneous talk and listen, which full duplex provides natively.
- **High-throughput server links**: Data center and storage links use full duplex to maximize aggregate throughput on each connection.

**Half Duplex**

- **Legacy shared-medium LANs**: Old hub-based Ethernet or coax segments only have one physical channel, so devices must take turns transmitting.
- **Two-way radio communication**: Walkie-talkies and CB radios use a single RF channel, requiring users to press-to-talk and release to listen.
- **Contended wireless spectrum**: Wi-Fi devices on the same channel behave half duplex at the medium level since only one can transmit without collision at a time.
