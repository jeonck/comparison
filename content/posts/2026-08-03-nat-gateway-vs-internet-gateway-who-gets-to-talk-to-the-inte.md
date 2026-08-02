---
title: "NAT Gateway vs Internet Gateway: Who Gets to Talk to the Internet"
date: 2026-08-03T06:27:51.343218+09:00
tags: ["aws", "networking", "vpc", "cloud-infrastructure"]
---
## Overview

Both connect a VPC to the internet, but they serve opposite purposes: an <strong class="kw">Internet Gateway</strong> lets public-facing resources send and receive traffic directly, while a <strong class="kw">NAT Gateway</strong> lets private resources reach out without ever being reachable from outside. Picking the wrong one either exposes resources you meant to keep private or silently blocks the outbound access your servers need.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 Z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="25" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Internet Gateway</text><text x="480" y="25" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">NAT Gateway</text><rect x="110" y="45" width="100" height="36" rx="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="68" text-anchor="middle" font-size="12" style="fill:var(--content)">Internet</text><rect x="430" y="45" width="100" height="36" rx="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="68" text-anchor="middle" font-size="12" style="fill:var(--content)">Internet</text><line x1="160" y1="82" x2="160" y2="109" style="stroke:var(--compare-a)" stroke-width="2" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/><line x1="480" y1="110" x2="480" y2="82" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><circle cx="560" cy="95" r="10" style="fill:none;stroke:var(--secondary)" stroke-width="1.5"/><line x1="553" y1="88" x2="567" y2="102" style="stroke:var(--secondary)" stroke-width="1.5"/><text x="560" y="120" text-anchor="middle" font-size="9" style="fill:var(--secondary)">no inbound</text><rect x="110" y="110" width="100" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="134" text-anchor="middle" font-size="12" style="fill:var(--content)">IGW</text><rect x="430" y="110" width="100" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="134" text-anchor="middle" font-size="11" style="fill:var(--content)">NAT Gateway</text><line x1="160" y1="150" x2="160" y2="189" style="stroke:var(--compare-a)" stroke-width="2" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/><line x1="480" y1="190" x2="480" y2="151" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><rect x="70" y="190" width="180" height="120" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="160" y="206" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Public Subnet</text><rect x="110" y="228" width="100" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="252" text-anchor="middle" font-size="12" style="fill:var(--content)">Instance</text><text x="160" y="285" text-anchor="middle" font-size="10" style="fill:var(--secondary)">has public IP</text><rect x="390" y="190" width="180" height="120" rx="8" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="480" y="206" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Private Subnet</text><rect x="430" y="228" width="100" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="252" text-anchor="middle" font-size="12" style="fill:var(--content)">Instance</text><text x="480" y="285" text-anchor="middle" font-size="10" style="fill:var(--secondary)">private IP only</text><text x="160" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">bidirectional traffic</text><text x="480" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">outbound only</text></svg>
</div>

## Comparison Table

| Aspect | Internet Gateway | NAT Gateway |
| --- | --- | --- |
| Primary purpose | Enables communication between a VPC and the internet in both directions | Enables outbound-only internet access for resources without public IPs |
| Traffic direction | Bidirectional — accepts inbound connections and sends outbound | Outbound only — inbound traffic allowed only as replies to established connections |
| Placement | Attaches directly to the VPC as a whole | Deployed inside a specific public subnet |
| IP address handling | 1:1 NAT between a private IP and an Elastic/public IP | Many-to-one PAT — many private IPs share the gateway's public IP |
| Which resources use it | Instances with a public/Elastic IP routed via a public subnet route table | Instances with only private IPs routed via a private subnet route table |
| Scaling and availability | Managed, horizontally scaled, highly available with no bandwidth cap | Bandwidth-bounded per gateway; needs one per AZ for high availability |
| Cost model | No hourly charge and no data processing fee | Hourly charge plus per-GB data processing fee |
| Failure impact | Loss cuts off all direct internet reachability for the public subnet | Loss cuts off outbound internet access for the private subnet only |

## Key Differences

- Internet Gateway provides <strong class="kw">bidirectional</strong> access; NAT Gateway only permits <strong class="kw">outbound</strong> connections.
- Internet Gateway attaches to the whole <strong class="kw">VPC</strong>; NAT Gateway lives inside a specific <strong class="kw">subnet</strong>.
- Internet Gateway does 1:1 <strong class="kw">Elastic IP</strong> mapping; NAT Gateway does many-to-one <strong class="kw">PAT</strong>.
- NAT Gateway bills per <strong class="kw">GB processed</strong>; Internet Gateway is <strong class="kw">free</strong>.

## When to Use Each

**Internet Gateway**

- **Public-facing web servers**: Load balancers or web servers that must accept inbound connections from arbitrary internet clients need an Internet Gateway.
- **Bastion or jump hosts**: A host that administrators SSH into from outside the VPC needs direct, inbound-reachable internet access.

**NAT Gateway**

- **Private backend patching**: Database or app servers in a private subnet need outbound access to pull OS updates without ever accepting inbound connections.
- **Protecting backend tiers**: Keeping application and database instances without public IPs while still allowing them to call external APIs.
- **Multi-AZ egress resilience**: Deploying one NAT Gateway per availability zone avoids a single point of failure and cross-AZ data transfer charges.
