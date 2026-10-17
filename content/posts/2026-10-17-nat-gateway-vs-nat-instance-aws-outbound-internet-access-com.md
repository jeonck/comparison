---
title: "NAT Gateway vs NAT Instance: AWS Outbound Internet Access Compared"
date: 2026-10-17T22:16:28.350697+09:00
tags: ["aws", "networking", "nat-gateway", "vpc"]
---
## Overview

Both let private-subnet resources reach the internet without being directly reachable from it, but they differ in who operates the underlying infrastructure. A <strong class="kw">NAT Gateway</strong> is a fully managed AWS resource, while a <strong class="kw">NAT Instance</strong> is a plain EC2 instance you configure and maintain yourself. The choice mainly comes down to operational overhead, throughput, and cost at your traffic volume.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="28" text-anchor="middle" font-size="15" font-weight="700" style="fill:var(--primary)">NAT Gateway</text><text x="480" y="28" text-anchor="middle" font-size="15" font-weight="700" style="fill:var(--primary)">NAT Instance</text><line x1="320" y1="10" x2="320" y2="345" stroke-dasharray="4,4" style="stroke:var(--border)" stroke-width="1"/><rect x="60" y="45" width="200" height="45" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="72" text-anchor="middle" font-size="12" style="fill:var(--content)">Private Subnet (EC2 apps)</text><line x1="160" y1="90" x2="160" y2="140" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="160,148 155,138 165,138" style="fill:var(--border)"/><rect x="50" y="150" width="220" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="173" text-anchor="middle" font-size="13" font-weight="600" style="fill:var(--primary)">NAT Gateway</text><text x="160" y="190" text-anchor="middle" font-size="10.5" style="fill:var(--content)">AWS-managed, auto-scales to 100 Gbps</text><text x="160" y="203" text-anchor="middle" font-size="10" style="fill:var(--secondary)">HA within its AZ</text><line x1="160" y1="210" x2="160" y2="250" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="160,258 155,248 165,248" style="fill:var(--border)"/><rect x="60" y="260" width="200" height="38" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="283" text-anchor="middle" font-size="11.5" style="fill:var(--content)">Internet Gateway</text><line x1="160" y1="298" x2="160" y2="325" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="160,333 155,323 165,323" style="fill:var(--border)"/><text x="160" y="348" text-anchor="middle" font-size="12" style="fill:var(--content)">Internet</text><rect x="380" y="45" width="200" height="45" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="72" text-anchor="middle" font-size="12" style="fill:var(--content)">Private Subnet (EC2 apps)</text><line x1="480" y1="90" x2="480" y2="140" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="480,148 475,138 485,138" style="fill:var(--border)"/><rect x="370" y="150" width="220" height="60" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="173" text-anchor="middle" font-size="13" font-weight="600" style="fill:var(--primary)">NAT Instance</text><text x="480" y="190" text-anchor="middle" font-size="10.5" style="fill:var(--content)">Plain EC2, throughput = instance type</text><text x="480" y="203" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Single point of failure</text><line x1="480" y1="210" x2="480" y2="250" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="480,258 475,248 485,248" style="fill:var(--border)"/><rect x="380" y="260" width="200" height="38" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="283" text-anchor="middle" font-size="11.5" style="fill:var(--content)">Internet Gateway</text><line x1="480" y1="298" x2="480" y2="325" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="480,333 475,323 485,323" style="fill:var(--border)"/><text x="480" y="348" text-anchor="middle" font-size="12" style="fill:var(--content)">Internet</text></svg>
</div>

## Comparison Table

| Aspect | NAT Gateway | NAT Instance |
| --- | --- | --- |
| Provisioning | Create as a managed AWS resource via console/CLI in minutes; no AMI to pick | Launch a regular EC2 instance from a NAT-capable AMI and configure it manually |
| Network placement | Deployed in a public subnet; referenced as the target in private subnet route tables | EC2 instance in a public subnet with source/destination check disabled |
| Throughput & scaling | Automatically scales up to 100 Gbps of bandwidth | Capped by the chosen instance type's network performance; resize to scale |
| High availability | HA automatically within its AZ; deploy one per AZ for cross-AZ resilience | Single point of failure unless you script failover (Auto Scaling, health checks, Lambda) |
| Security controls | No security groups attached; cannot double as a bastion or proxy | Has security groups and can run additional software, e.g. as a bastion or custom proxy |
| Patching & maintenance | Fully managed by AWS; no OS or software to patch | You patch the OS, monitor health, and manage the AMI lifecycle |
| Cost model | Hourly charge plus per-GB data processing fee | Standard EC2 instance-hour cost only, no extra NAT data fee |
| Teardown | Delete the NAT Gateway resource and release its Elastic IP | Terminate the EC2 instance and release its Elastic IP |

## Key Differences

- <strong class="kw">NAT Gateway</strong> is a fully managed AWS service with no OS to maintain, unlike a self-managed <strong class="kw">NAT Instance</strong>
- Gateway auto-scales bandwidth up to 100 Gbps; an Instance's throughput is capped by its <strong class="kw">instance type</strong>
- Gateway has no <strong class="kw">security groups</strong>; an Instance supports them and can double as a bastion
- Gateway HA is automatic within an AZ; an Instance needs manual <strong class="kw">failover scripting</strong>
- Gateway bills per-GB processed; an Instance only incurs standard <strong class="kw">EC2 pricing</strong>

## When to Use Each

**NAT Gateway**

- **Production, high-throughput traffic**: NAT Gateway auto-scales to 100 Gbps without capacity planning or instance resizing.
- **Minimizing operational overhead**: There's no OS to patch, monitor, or replace, which suits teams that want to avoid NAT-layer maintenance.
- **Multi-AZ fault isolation**: Deploying one NAT Gateway per AZ keeps outbound traffic resilient to a single AZ failure with built-in HA.

**NAT Instance**

- **Low-traffic dev/test environments**: A small EC2 instance is cheaper than NAT Gateway's per-GB processing fee when data volume is minimal.
- **Custom NAT or proxy logic**: Running your own instance lets you add traffic filtering, logging, or proxy behavior NAT Gateway doesn't expose.
- **Security-group-level control**: Teams needing fine-grained security group rules on the NAT path itself must use an instance, since NAT Gateway has none.
