---
title: "Subnet vs VLAN: Layer 3 IP Segmentation vs Layer 2 Port Segmentation"
date: 2026-08-03T08:01:43.117158+09:00
tags: ["networking", "subnetting", "vlan", "lan-design"]
---
## Overview

A subnet and a VLAN both carve a large network into smaller, more manageable pieces, but they operate at different layers and are configured in different places. A <strong class="kw">subnet</strong> divides IP address space at Layer 3 based on address range and mask, independent of physical wiring, while a <strong class="kw">VLAN</strong> divides switch ports at Layer 2, creating separate broadcast domains on shared physical hardware. In most enterprise designs the two are paired one-to-one, but knowing which layer each governs matters for troubleshooting, security, and scaling.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Subnet (Layer 3)</text><rect x="130" y="55" width="60" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="74" text-anchor="middle" style="fill:var(--content)" font-size="11">Router</text><line x1="160" y1="85" x2="90" y2="130" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="85" x2="230" y2="130" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="30" y="130" width="120" height="90" rx="6" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="5 3"/><text x="90" y="148" text-anchor="middle" style="fill:var(--content)" font-size="11">10.0.1.0/24</text><circle cx="60" cy="180" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="120" cy="180" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="60" y="200" text-anchor="middle" style="fill:var(--secondary)" font-size="9">.10</text><text x="120" y="200" text-anchor="middle" style="fill:var(--secondary)" font-size="9">.11</text><rect x="170" y="130" width="120" height="90" rx="6" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="5 3"/><text x="230" y="148" text-anchor="middle" style="fill:var(--content)" font-size="11">10.0.2.0/24</text><circle cx="200" cy="180" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="260" cy="180" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="200" y="200" text-anchor="middle" style="fill:var(--secondary)" font-size="9">.10</text><text x="260" y="200" text-anchor="middle" style="fill:var(--secondary)" font-size="9">.11</text><text x="160" y="250" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Grouped by IP address range</text><text x="160" y="266" text-anchor="middle" style="fill:var(--secondary)" font-size="10">independent of physical wiring</text><text x="480" y="35" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">VLAN (Layer 2)</text><rect x="360" y="60" width="240" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="90" text-anchor="middle" style="fill:var(--content)" font-size="11">Switch</text><rect x="368" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="392" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="416" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="440" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="524" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="548" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="572" y="105" width="20" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><rect x="596" y="105" width="4" height="10" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1"/><line x1="480" y1="115" x2="480" y2="230" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><text x="415" y="135" text-anchor="middle" style="fill:var(--content)" font-size="11">VLAN 10</text><circle cx="395" cy="170" r="12" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="435" cy="170" r="12" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="545" y="135" text-anchor="middle" style="fill:var(--content)" font-size="11">VLAN 20</text><circle cx="525" cy="170" r="12" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="565" cy="170" r="12" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="250" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Grouped by switch port tag (802.1Q)</text><text x="480" y="266" text-anchor="middle" style="fill:var(--secondary)" font-size="10">separate broadcast domains, same switch</text></svg>
</div>

## Comparison Table

| Aspect | Subnet | VLAN |
| --- | --- | --- |
| OSI layer | Layer 3 (Network) | Layer 2 (Data Link) |
| Defined by | IP address range and subnet mask (CIDR) | VLAN ID tag (802.1Q) assigned to switch ports |
| What it segments | IP address space into logical networks | Physical switch ports into separate broadcast domains |
| Where it's configured | Host IP settings, DHCP scopes, and router interfaces | Switch port assignments and trunk configuration |
| Spanning multiple switches | Works automatically if routing between switches is correct; not tied to topology | Requires 802.1Q trunk links to extend the same VLAN across switches |
| Broadcast isolation | Implied by routing, but a switch can still flood broadcasts within the same physical segment | Actively enforced by the switch hardware, regardless of IP addressing |
| Cross-segment communication | Requires a Layer 3 router or route between subnets | Requires a router or Layer 3 switch to route between VLANs |
| Typical relationship | Usually mapped 1:1 to a VLAN by convention, not by protocol requirement | Usually mapped 1:1 to a subnet by convention, not by protocol requirement |

## Key Differences

- A <strong class="kw">subnet</strong> is a Layer 3 IP addressing construct; a <strong class="kw">VLAN</strong> is a Layer 2 switching construct.
- Subnets are identified by a <strong class="kw">CIDR mask</strong>, VLANs by an <strong class="kw">802.1Q tag</strong>.
- Extending a subnet across switches just needs correct routing; extending a VLAN needs <strong class="kw">trunking</strong>.
- VLANs enforce broadcast domain isolation in hardware; subnets rely on <strong class="kw">routing</strong> to separate traffic.
- Best practice pairs one VLAN with one subnet, but this <strong class="kw">1:1 mapping</strong> is a design choice, not a protocol rule.

## When to Use Each

**Subnet**

- **IP address planning**: Subnetting is how you allocate and document address space across sites, VPNs, and routers.
- **Firewall/ACL scoping**: Security rules are almost always written against IP subnet ranges, not VLAN tags.
- **Routing between sites**: WAN and inter-site connectivity is governed by subnet reachability, independent of local switch VLANs.

**VLAN**

- **Isolating shared switches**: VLANs let multiple departments or tenants share the same physical switch hardware without seeing each other's broadcast traffic.
- **Reducing broadcast domain size**: Splitting a large LAN into VLANs limits ARP/broadcast flooding without requiring new cabling.
- **Segmenting without new hardware**: A single switch stack can support many logically separate networks purely through port/tag configuration.
