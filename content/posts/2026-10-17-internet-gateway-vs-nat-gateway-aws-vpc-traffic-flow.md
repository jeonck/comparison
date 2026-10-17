---
title: "Internet Gateway vs NAT Gateway: AWS VPC Traffic Flow"
date: 2026-10-17T22:15:17.980271+09:00
tags: ["aws", "vpc", "networking", "cloud-infrastructure"]
---
## Overview

Both are AWS VPC components that connect subnets to the internet, but they serve opposite access models. An <strong class="kw">Internet Gateway</strong> gives resources with public IPs full bidirectional internet access, while a <strong class="kw">NAT Gateway</strong> lets private resources reach outbound without ever being directly reachable from the internet. Picking the wrong one either exposes resources you meant to keep private or blocks outbound traffic a backend service needs.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="300" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="600">Internet Gateway</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="600">NAT Gateway</text><text x="160" y="52" text-anchor="middle" style="fill:var(--secondary)" font-size="13">Internet</text><text x="480" y="52" text-anchor="middle" style="fill:var(--secondary)" font-size="13">Internet</text><line x1="160" y1="60" x2="160" y2="106" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,58 155,66 165,66" style="fill:var(--compare-a)"/><polygon points="160,108 155,100 165,100" style="fill:var(--compare-a)"/><rect x="100" y="108" width="120" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="132" text-anchor="middle" style="fill:var(--content)" font-size="12">IGW</text><line x1="160" y1="148" x2="160" y2="198" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,146 155,154 165,154" style="fill:var(--compare-a)"/><polygon points="160,200 155,192 165,192" style="fill:var(--compare-a)"/><rect x="70" y="200" width="180" height="56" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="222" text-anchor="middle" style="fill:var(--content)" font-size="11">Public Subnet</text><text x="160" y="240" text-anchor="middle" style="fill:var(--content)" font-size="11">EC2 (public IP)</text><text x="160" y="282" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Bidirectional traffic allowed</text><line x1="470" y1="106" x2="470" y2="62" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="470,58 465,66 475,66" style="fill:var(--compare-b)"/><text x="452" y="88" text-anchor="end" style="fill:var(--content)" font-size="10">outbound</text><line x1="492" y1="62" x2="492" y2="106" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><line x1="486" y1="56" x2="498" y2="68" style="stroke:var(--border)" stroke-width="1.5"/><line x1="498" y1="56" x2="486" y2="68" style="stroke:var(--border)" stroke-width="1.5"/><text x="508" y="88" text-anchor="start" style="fill:var(--secondary)" font-size="10">blocked</text><rect x="420" y="108" width="120" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="132" text-anchor="middle" style="fill:var(--content)" font-size="12">NAT Gateway</text><line x1="470" y1="198" x2="470" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="470,148 465,156 475,156" style="fill:var(--compare-b)"/><text x="452" y="178" text-anchor="end" style="fill:var(--content)" font-size="10">outbound</text><line x1="492" y1="150" x2="492" y2="198" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3,3"/><line x1="486" y1="144" x2="498" y2="156" style="stroke:var(--border)" stroke-width="1.5"/><line x1="498" y1="144" x2="486" y2="156" style="stroke:var(--border)" stroke-width="1.5"/><text x="508" y="178" text-anchor="start" style="fill:var(--secondary)" font-size="10">blocked</text><rect x="390" y="200" width="180" height="56" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="222" text-anchor="middle" style="fill:var(--content)" font-size="11">Private Subnet</text><text x="480" y="240" text-anchor="middle" style="fill:var(--content)" font-size="11">EC2 (private IP)</text><text x="480" y="282" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Outbound-initiated only</text></svg>
</div>

## Comparison Table

| Aspect | Internet Gateway | NAT Gateway |
| --- | --- | --- |
| Primary purpose | Routes traffic between a VPC and the public internet | Lets private-subnet resources initiate outbound internet traffic |
| Traffic direction | Bidirectional (inbound and outbound) | Outbound-initiated only; unsolicited inbound is dropped |
| Subnet placement | Attached to the VPC itself; referenced by public subnet route tables | Deployed inside a public subnet, referenced by private subnet route tables |
| IP translation | None performed; relies on 1:1 mapping of public/Elastic IPs to instances | Performs many-to-one NAT, translating many private IPs to its own Elastic IP |
| Scaling & bandwidth | Horizontally scaled, redundant, no bandwidth cap | Scales automatically per AZ, bursts to tens of Gbps as a single managed appliance |
| High availability | Inherently redundant across the VPC, not AZ-scoped | Zonal resource; needs one NAT Gateway per AZ for resilience |
| Cost | No charge from AWS | Hourly charge plus per-GB data processing fee |
| Failure mode | Instance unreachable from outside if route or public IP missing | All private instances behind it lose internet access if it fails or is misconfigured |

## Key Differences

- <strong class="kw">Internet Gateway</strong> allows unsolicited inbound connections; <strong class="kw">NAT Gateway</strong> only permits traffic the private instance initiated.
- NAT Gateway performs <strong class="kw">address translation</strong> to a shared Elastic IP; Internet Gateway does no translation and depends on each instance already having a public IP.
- Internet Gateway is <strong class="kw">free</strong> and VPC-wide; NAT Gateway is a billed, <strong class="kw">zonal resource</strong> that needs per-AZ redundancy.
- Losing the Internet Gateway cuts off public-subnet reachability; losing a NAT Gateway cuts off all outbound access for its entire private subnet.

## When to Use Each

**Internet Gateway**

- **Public web/app servers**: Instances that must accept inbound requests directly from users need an Internet Gateway route and a public IP.
- **Bastion hosts**: A jump box needs inbound SSH/RDP from the internet, which only an Internet Gateway permits.
- **Load balancer front ends**: Internet-facing ALBs/NLBs sit in public subnets routed through an Internet Gateway to receive client traffic.

**NAT Gateway**

- **Private app/database tier**: Backend servers that need OS patches or API calls out but must never accept inbound internet connections.
- **Compliance-restricted workloads**: Workloads required to have no direct inbound internet path still need outbound connectivity for updates or telemetry.
- **Multi-tier architectures**: Private subnets behind a public-facing tier use a NAT Gateway so only the front end is directly internet-reachable.
