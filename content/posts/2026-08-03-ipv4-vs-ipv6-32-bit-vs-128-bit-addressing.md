---
title: "IPv4 vs IPv6: 32-bit vs 128-bit Addressing"
date: 2026-08-01T20:03:00+09:00
tags: ["networking", "ipv4", "ipv6", "protocols"]
---
## Overview

IPv4 and IPv6 are the two versions of the Internet Protocol responsible for addressing and routing packets across networks. IPv4 relies on <strong class="kw">32-bit addresses</strong> that ran out of unique combinations, while IPv6 was designed around <strong class="kw">128-bit addresses</strong> to give every device a globally unique, non-NAT'd address. The distinction matters because it affects address exhaustion, header processing overhead, and whether NAT traversal is required for peer-to-peer connectivity.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="157" y="40" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">IPv4</text><text x="477" y="40" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">IPv6</text><g><rect x="40" y="60" width="55" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="67" y="85" text-anchor="middle" font-size="14" style="fill:var(--content)">192</text><rect x="100" y="60" width="55" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="127" y="85" text-anchor="middle" font-size="14" style="fill:var(--content)">168</text><rect x="160" y="60" width="55" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="187" y="85" text-anchor="middle" font-size="14" style="fill:var(--content)">1</text><rect x="220" y="60" width="55" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="247" y="85" text-anchor="middle" font-size="14" style="fill:var(--content)">1</text></g><text x="157" y="118" text-anchor="middle" font-size="12" style="fill:var(--secondary)">32 bits · dotted-decimal</text><g><rect x="350" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="365" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">2001</text><rect x="382" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="397" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">0db8</text><rect x="414" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="429" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">85a3</text><rect x="446" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="461" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">0000</text><rect x="478" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="493" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">0000</text><rect x="510" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="525" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">8a2e</text><rect x="542" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="557" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">0370</text><rect x="574" y="60" width="30" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="589" y="83" text-anchor="middle" font-size="8" style="fill:var(--content)">7334</text></g><text x="477" y="118" text-anchor="middle" font-size="12" style="fill:var(--secondary)">128 bits · hex colon-notation</text><text x="157" y="150" text-anchor="middle" font-size="13" style="fill:var(--content)">~4.3 billion addresses</text><text x="477" y="150" text-anchor="middle" font-size="13" style="fill:var(--content)">~340 undecillion addresses</text><line x1="320" y1="30" x2="320" y2="330" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="40" y="200" font-size="12" style="fill:var(--secondary)">Relative address length</text><rect x="40" y="215" width="48" height="18" rx="2" style="fill:var(--compare-a);stroke:var(--compare-a)"/><text x="94" y="228" font-size="12" style="fill:var(--content)">32 bits (IPv4)</text><rect x="40" y="245" width="192" height="18" rx="2" style="fill:var(--compare-b);stroke:var(--compare-b)"/><text x="238" y="258" font-size="12" style="fill:var(--content)">128 bits (IPv6) — 4x longer</text><g><rect x="360" y="200" width="240" height="70" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1"/><text x="480" y="220" text-anchor="middle" font-size="12" style="fill:var(--secondary)">NAT dependency</text><text x="480" y="242" text-anchor="middle" font-size="12" style="fill:var(--compare-a)">IPv4: needs NAT (scarce space)</text><text x="480" y="260" text-anchor="middle" font-size="12" style="fill:var(--compare-b)">IPv6: end-to-end, no NAT needed</text></g></svg>
</div>

## Comparison Table

| Aspect | IPv4 | IPv6 |
| --- | --- | --- |
| Address length & notation | 32-bit, dotted-decimal (e.g. 192.168.1.1) | 128-bit, hexadecimal colon-separated (e.g. 2001:0db8::7334) |
| Address space size | ~4.3 billion addresses | ~340 undecillion addresses |
| Address assignment | Manual configuration or DHCP | Stateless Address Autoconfiguration (SLAAC) or DHCPv6 |
| Header structure | Variable-length header with options field and checksum | Fixed 40-byte header, no checksum, optional extension headers |
| NAT requirement | Commonly required due to address scarcity | Not needed; supports true end-to-end addressing |
| Broadcast/discovery | Uses broadcast (e.g. ARP) for local discovery | Broadcast eliminated; uses multicast Neighbor Discovery |
| Built-in security | IPsec is an optional add-on | IPsec support is part of the core protocol spec |
| Adoption & compatibility | Universally supported, legacy infrastructure | Growing adoption, requires dual-stack or tunneling for legacy interop |

## Key Differences

- IPv6 addresses are <strong class="kw">128-bit</strong>, four times longer than IPv4's <strong class="kw">32-bit</strong> addresses, resolving address exhaustion
- IPv6 removes the need for <strong class="kw">NAT</strong>, restoring true end-to-end connectivity between hosts
- IPv6 uses a simplified, <strong class="kw">fixed-length header</strong> that speeds up router processing compared to IPv4's variable header
- IPv6 replaces ARP broadcasts with <strong class="kw">Neighbor Discovery</strong> multicast for local address resolution

## When to Use Each

**IPv4**

- **Legacy Network Compatibility**: Many ISPs, routers, and enterprise systems still only support IPv4, making it necessary for interoperability.
- **Small Networks Behind NAT**: Home and small office networks rarely need more addresses than NAT and a single public IPv4 address can provide.
- **Familiar Tooling & Troubleshooting**: Dotted-decimal notation and decades of IPv4-centric tooling make debugging and documentation more accessible.

**IPv6**

- **Large-Scale IoT Deployments**: Massive fleets of sensors and devices need the vast address space IPv6 provides without relying on NAT.
- **Mobile Carrier Networks**: Modern cellular networks run IPv6-first internally to assign unique addresses to billions of subscriber devices.
- **Peer-to-Peer & Real-Time Apps**: VoIP, gaming, and P2P applications benefit from IPv6's end-to-end connectivity without NAT traversal workarounds.
