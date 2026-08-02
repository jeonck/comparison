---
title: "DNS vs DHCP: Naming the Network vs Configuring It"
date: 2026-08-01T20:04:00+09:00
tags: ["dns", "dhcp", "networking", "protocols"]
---
## Overview

DHCP and DNS are both foundational network services, but they solve different problems in a device's journey onto the network. <strong class="kw">DHCP</strong> automatically assigns a device its IP address and network configuration when it joins a subnet, while <strong class="kw">DNS</strong> translates human-readable domain names into the IP addresses needed to actually reach other hosts.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <text x="170" y="45" text-anchor="middle" font-size="18" style="fill:var(--primary)">DHCP</text>
  <text x="490" y="45" text-anchor="middle" font-size="18" style="fill:var(--primary)">DNS</text>
  <line x1="320" y1="15" x2="320" y2="345" stroke-dasharray="4,4" style="stroke:var(--border)" stroke-width="1"/>

  <rect x="40" y="140" width="100" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="90" y="169" text-anchor="middle" font-size="11" style="fill:var(--content)">Client (no IP)</text>

  <rect x="200" y="140" width="100" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="250" y="163" text-anchor="middle" font-size="11" style="fill:var(--content)">DHCP</text>
  <text x="250" y="177" text-anchor="middle" font-size="11" style="fill:var(--content)">Server</text>

  <line x1="140" y1="155" x2="196" y2="155" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <polygon points="200,155 194,151 194,159" style="fill:var(--compare-a)"/>
  <text x="170" y="146" text-anchor="middle" font-size="9" style="fill:var(--secondary)">DHCPDISCOVER</text>

  <line x1="200" y1="175" x2="144" y2="175" style="stroke:var(--compare-a)" stroke-width="1.5"/>
  <polygon points="140,175 146,171 146,179" style="fill:var(--compare-a)"/>
  <text x="170" y="197" text-anchor="middle" font-size="9" style="fill:var(--secondary)">leased IP + gateway</text>

  <text x="170" y="230" text-anchor="middle" font-size="10" style="fill:var(--secondary)">local subnet, broadcast</text>

  <rect x="360" y="140" width="100" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="410" y="169" text-anchor="middle" font-size="11" style="fill:var(--content)">Client (has IP)</text>

  <rect x="520" y="140" width="100" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="570" y="163" text-anchor="middle" font-size="11" style="fill:var(--content)">DNS</text>
  <text x="570" y="177" text-anchor="middle" font-size="11" style="fill:var(--content)">Resolver</text>

  <line x1="460" y1="155" x2="516" y2="155" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <polygon points="520,155 514,151 514,159" style="fill:var(--compare-b)"/>
  <text x="490" y="146" text-anchor="middle" font-size="9" style="fill:var(--secondary)">example.com?</text>

  <line x1="520" y1="175" x2="464" y2="175" style="stroke:var(--compare-b)" stroke-width="1.5"/>
  <polygon points="460,175 466,171 466,179" style="fill:var(--compare-b)"/>
  <text x="490" y="197" text-anchor="middle" font-size="9" style="fill:var(--secondary)">93.184.216.34</text>

  <text x="490" y="230" text-anchor="middle" font-size="10" style="fill:var(--secondary)">global, hierarchical</text>

  <text x="170" y="280" text-anchor="middle" font-size="11" style="fill:var(--content)">gives device an address</text>
  <text x="490" y="280" text-anchor="middle" font-size="11" style="fill:var(--content)">gives a name an address</text>
</svg>
</div>

## Comparison Table

| Aspect | DHCP | DNS |
| --- | --- | --- |
| Primary purpose | Assigns an IP address and network configuration to a device | Translates a domain name into an IP address |
| Triggered by | A device connecting or booting onto the network | An application needing to resolve a hostname |
| Transport protocol | UDP, ports 67 (server) and 68 (client) | UDP or TCP, port 53 |
| Discovery mechanism | Client broadcasts DHCPDISCOVER on the local subnet | Client sends a unicast query to a configured resolver address |
| Data returned | IP address, subnet mask, default gateway, DNS server list | IP address (A/AAAA record) or other record types like MX, CNAME, TXT |
| State and validity | Lease with an expiration time that must be renewed | Record with a TTL, cached locally then re-queried after expiry |
| Scope | Local network segment or subnet | Global, hierarchical, distributed across the internet |

## Key Differences

- DHCP assigns <strong class="kw">IP addresses</strong> to devices; DNS resolves <strong class="kw">domain names</strong> to those addresses.
- DHCP requests use <strong class="kw">broadcast</strong> discovery on the local subnet; DNS clients send <strong class="kw">unicast queries</strong> to a configured resolver.
- DHCP assignments are <strong class="kw">leases</strong> that expire and renew; DNS answers are <strong class="kw">cached</strong> per record TTL.
- DHCP typically hands out the <strong class="kw">DNS server addresses</strong> a client should use, linking the two protocols at boot time.

## When to Use Each

**DHCP**

- **New device onboarding**: DHCP automatically configures IP, gateway, and DNS servers the moment a device joins the network, with no manual setup.
- **Large dynamic networks**: DHCP simplifies managing IP allocation across many transient devices like laptops and phones through a central address pool.
- **Network-wide reconfiguration**: Changing subnet or gateway settings network-wide is done centrally on the DHCP server without touching each device.

**DNS**

- **Human-readable addressing**: DNS lets users and applications reference services by name instead of memorizing and updating IP addresses.
- **Service discovery and failover**: DNS records can point a name to different IPs for load balancing, geo-routing, or failover without changing client configuration.
- **Email routing**: DNS MX records direct mail delivery to the correct mail servers for a domain.
