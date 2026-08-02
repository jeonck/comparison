---
title: "MAC Address vs IP Address: Hardware Identity vs Network Location"
date: 2026-08-03T08:08:54.977237+09:00
tags: ["networking", "mac-address", "ip-address", "osi-model"]
---
## Overview

A MAC address is a <strong class="kw">hardware identifier</strong> burned into a network interface card and used to move frames across a single local link. An IP address is a <strong class="kw">logical network address</strong> assigned to a device and used to route packets across interconnected networks, including the internet. They operate at different OSI layers, working together to get data from one physical wire to a destination anywhere in the world.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><rect x="40" y="140" width="110" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="165" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="bold">Host A</text><text x="95" y="183" text-anchor="middle" style="fill:var(--content)" font-size="11">IP 10.0.0.5</text><rect x="265" y="140" width="110" height="60" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="320" y="165" text-anchor="middle" style="fill:var(--content)" font-size="14" font-weight="bold">Router</text><text x="320" y="183" text-anchor="middle" style="fill:var(--secondary)" font-size="10">rewrites MAC per hop</text><rect x="490" y="140" width="110" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="545" y="165" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="bold">Host B</text><text x="545" y="183" text-anchor="middle" style="fill:var(--content)" font-size="11">IP 10.0.0.9</text><line x1="150" y1="170" x2="265" y2="170" style="stroke:var(--compare-a)" stroke-width="2"/><polygon points="265,170 255,165 255,175" style="fill:var(--compare-a)"/><text x="207" y="128" text-anchor="middle" style="fill:var(--compare-a)" font-size="10">MAC AA:01 to MAC RR:01</text><line x1="375" y1="170" x2="490" y2="170" style="stroke:var(--compare-b)" stroke-width="2"/><polygon points="490,170 480,165 480,175" style="fill:var(--compare-b)"/><text x="432" y="128" text-anchor="middle" style="fill:var(--compare-b)" font-size="10">MAC RR:02 to MAC BB:01</text><text x="320" y="108" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Layer 2 - MAC changes at every hop</text><line x1="95" y1="260" x2="545" y2="260" style="stroke:var(--primary)" stroke-dasharray="6,4" stroke-width="2"/><polygon points="545,260 535,255 535,265" style="fill:var(--primary)"/><text x="320" y="245" text-anchor="middle" style="fill:var(--primary)" font-size="12" font-weight="bold">IP 10.0.0.5 to 10.0.0.9</text><text x="320" y="280" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Layer 3 - IP stays constant end-to-end</text><line x1="95" y1="200" x2="95" y2="260" style="stroke:var(--border)" stroke-dasharray="2,3" stroke-width="1"/><line x1="545" y1="200" x2="545" y2="260" style="stroke:var(--border)" stroke-dasharray="2,3" stroke-width="1"/><text x="320" y="330" text-anchor="middle" style="fill:var(--content)" font-size="13" font-weight="bold">MAC = local hop identity; IP = end-to-end address</text></svg>
</div>

## Comparison Table

| Aspect | MAC Address | IP Address |
| --- | --- | --- |
| OSI layer | Layer 2 (Data Link) | Layer 3 (Network) |
| Format | 48-bit hex, e.g. 00:1A:2B:3C:4D:5E | 32-bit (IPv4) or 128-bit (IPv6) dotted/colon notation |
| Assignment | Burned in by the NIC manufacturer at production | Assigned by a network admin or DHCP server |
| Structure | Flat, no hierarchy — vendor prefix plus serial | Hierarchical — network portion plus host portion for routing |
| Persistence | Fixed to the physical interface (though spoofable) | Can change when a device moves to a different network |
| Role in delivery | Identifies the next-hop device on the local link | Identifies source and destination across the whole path |
| Behavior across hops | Rewritten by every router at each hop | Preserved end-to-end (barring NAT) |
| Resolution mechanism | Discovered via ARP (IPv4) or NDP (IPv6) | Discovered via DNS for hostnames |

## Key Differences

- MAC address operates at <strong class="kw">Layer 2</strong> while IP address operates at <strong class="kw">Layer 3</strong>.
- MAC is <strong class="kw">burned into hardware</strong> by the manufacturer, whereas IP is <strong class="kw">assigned by the network</strong>.
- A frame's MAC addresses are <strong class="kw">rewritten at every hop</strong>, but the packet's IP addresses stay <strong class="kw">end-to-end constant</strong>.
- <strong class="kw">ARP</strong> maps an IP address to the MAC address needed for delivery on the local segment.
- MAC addresses are <strong class="kw">flat</strong> with no structure, while IP addresses are <strong class="kw">hierarchical</strong> to support routing.

## When to Use Each

**MAC Address**

- **Switch Forwarding**: Switches build MAC address tables to forward Ethernet frames to the correct port within a LAN segment.
- **MAC-based Access Control**: Wi-Fi or switch port security can allow or block traffic based on the known hardware address of a device.
- **DHCP Reservations**: Binding a specific IP lease to a device's MAC ensures it always receives the same address on the network.

**IP Address**

- **Routing Across Networks**: Routers forward packets between distinct networks, including the internet, using destination IP addresses.
- **Firewall and ACL Rules**: Access policies are typically defined against source and destination IP ranges rather than hardware addresses.
- **Subnetting and Network Design**: The hierarchical structure of IP addresses lets engineers divide networks into logical, routable subnets.
