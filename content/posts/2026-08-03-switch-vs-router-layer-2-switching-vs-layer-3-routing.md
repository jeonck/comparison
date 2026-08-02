---
title: "Switch vs Router: Layer 2 Switching vs Layer 3 Routing"
date: 2026-08-01T20:05:00+09:00
tags: ["networking", "switch", "router", "osi-layer"]
---
## Overview

A switch connects devices within a single network by forwarding traffic based on <strong class="kw">MAC addresses</strong>, while a router connects separate networks together by forwarding traffic based on <strong class="kw">IP addresses</strong>. Plugging a device into the wrong one is a common source of confusion — one expands a LAN, the other bridges LANs to each other or to the internet.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Switch</text><text x="480" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Router</text><rect x="20" y="55" width="280" height="260" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="6,4"/><text x="160" y="75" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Single broadcast domain</text><rect x="130" y="165" width="60" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="185" text-anchor="middle" style="fill:var(--content)" font-size="12">Switch</text><circle cx="60" cy="100" r="18" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="60" y="104" text-anchor="middle" style="fill:var(--content)" font-size="10">PC1</text><circle cx="260" cy="100" r="18" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="260" y="104" text-anchor="middle" style="fill:var(--content)" font-size="10">PC2</text><circle cx="160" cy="280" r="18" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="284" text-anchor="middle" style="fill:var(--content)" font-size="10">PC3</text><line x1="70" y1="112" x2="140" y2="172" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="250" y1="112" x2="180" y2="172" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="262" x2="160" y2="195" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="330" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Forwards by MAC address</text><rect x="330" y="60" width="110" height="90" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="6,4"/><text x="385" y="78" text-anchor="middle" style="fill:var(--secondary)" font-size="10">10.0.1.0/24</text><circle cx="360" cy="115" r="14" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="360" y="118" text-anchor="middle" style="fill:var(--content)" font-size="9">PC</text><circle cx="410" cy="115" r="14" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="410" y="118" text-anchor="middle" style="fill:var(--content)" font-size="9">PC</text><rect x="520" y="60" width="100" height="90" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="6,4"/><text x="570" y="78" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Internet</text><circle cx="570" cy="115" r="16" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="570" y="118" text-anchor="middle" style="fill:var(--content)" font-size="9">WAN</text><rect x="430" y="195" width="90" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="475" y="219" text-anchor="middle" style="fill:var(--content)" font-size="12">Router</text><line x1="390" y1="150" x2="450" y2="197" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="560" y1="150" x2="500" y2="197" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="330" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Forwards by IP address, links networks</text></svg>
</div>

## Comparison Table

| Aspect | Switch | Router |
| --- | --- | --- |
| OSI layer | Layer 2 (Data Link) | Layer 3 (Network) |
| Addressing used | MAC addresses | IP addresses |
| Forwarding table | MAC address table (CAM table), learned automatically | Routing table, built via static routes or routing protocols |
| Broadcast domain | Devices share one broadcast domain (unless VLANs are configured) | Separates traffic into distinct broadcast domains per interface |
| Typical placement | Connects devices within a single LAN segment | Connects different networks together, e.g. LAN to LAN or LAN to WAN |
| Unknown-destination handling | Floods frame to all ports in the VLAN when MAC is unknown | Drops or rejects packets with no matching route |
| Built-in services | Basic switching only, plus optional VLANs/QoS on managed models | Often includes NAT, DHCP, firewall, and VPN functions |
| Typical device | Cisco Catalyst switch in a wiring closet | Home router or edge router linking a LAN to an ISP |

## Key Differences

- Switches forward traffic using <strong class="kw">MAC addresses</strong> at Layer 2, while routers forward using <strong class="kw">IP addresses</strong> at Layer 3.
- A switch keeps connected devices in one <strong class="kw">broadcast domain</strong>; a router splits traffic into separate domains.
- Switches <strong class="kw">flood</strong> frames to unknown destinations within a VLAN; routers simply drop packets they can't route.
- Routers commonly bundle <strong class="kw">NAT</strong> and firewall features that switches don't provide.
- Switches scale port count for a single network; routers scale the number of distinct networks a device can reach.

## When to Use Each

**Switch**

- **Connecting Local Devices**: Use a switch to link PCs, printers, and servers within the same LAN for fast, low-latency communication.
- **Expanding Port Capacity**: A switch adds more physical ports to an existing network segment without changing its addressing.
- **VLAN Segmentation**: Managed switches can logically separate traffic into VLANs without deploying separate physical networks.

**Router**

- **Connecting Different Networks**: A router is required to pass traffic between separate subnets or between a LAN and the internet.
- **Providing Internet Access**: Home and office routers route traffic from the local network to the ISP's network.
- **NAT and Firewall Needs**: Routers translate private addresses to public ones and enforce access rules between networks.
