---
title: "IaaS vs PaaS: How Much of the Stack You Manage"
date: 2026-08-03T06:18:04.799776+09:00
tags: ["cloud-computing", "iaas", "paas", "cloud-service-models"]
---
## Overview

IaaS and PaaS are cloud service models that differ in how much of the technology stack the provider abstracts away from you. <strong class="kw">IaaS</strong> hands you raw virtualized infrastructure and leaves the OS, runtime, and app management to you, while <strong class="kw">PaaS</strong> also manages the OS and runtime so you only push application code. The distinction matters because it determines your team's operational burden, control, and how fast you can ship.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">IaaS</text><text x="480" y="36" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">PaaS</text><rect x="70" y="60" width="180" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="83" text-anchor="middle" font-size="12" style="fill:var(--content)">Application</text><rect x="70" y="96" width="180" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="119" text-anchor="middle" font-size="12" style="fill:var(--content)">Runtime</text><rect x="70" y="132" width="180" height="36" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="155" text-anchor="middle" font-size="12" style="fill:var(--content)">OS</text><line x1="70" y1="168" x2="250" y2="168" stroke-dasharray="4 3" style="stroke:var(--secondary)" stroke-width="1.5"/><rect x="70" y="168" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="191" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Network &amp; Storage</text><rect x="70" y="204" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="227" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Virtualization</text><rect x="70" y="240" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="263" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Hardware</text><text x="160" y="290" text-anchor="middle" font-size="11" style="fill:var(--compare-a)">you manage 3 layers</text><rect x="390" y="60" width="180" height="36" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="83" text-anchor="middle" font-size="12" style="fill:var(--content)">Application</text><line x1="390" y1="96" x2="570" y2="96" stroke-dasharray="4 3" style="stroke:var(--secondary)" stroke-width="1.5"/><rect x="390" y="96" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="119" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Runtime</text><rect x="390" y="132" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="155" text-anchor="middle" font-size="12" style="fill:var(--secondary)">OS</text><rect x="390" y="168" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="191" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Network &amp; Storage</text><rect x="390" y="204" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="227" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Virtualization</text><rect x="390" y="240" width="180" height="36" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="263" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Hardware</text><text x="480" y="290" text-anchor="middle" font-size="11" style="fill:var(--compare-b)">you manage 1 layer</text><text x="320" y="320" text-anchor="middle" font-size="11" style="fill:var(--secondary)">colored = customer-managed · outlined = provider-managed</text></svg>
</div>

## Comparison Table

| Aspect | IaaS | PaaS |
| --- | --- | --- |
| Provisioning unit | Virtual machines, block storage, virtual networks | Application slots or containers bound to a managed runtime |
| OS and runtime management | Customer installs, configures, and patches OS and runtime | Provider installs and patches OS and runtime automatically |
| Deployment workflow | Customer scripts server setup, then deploys app via SSH/config management | Customer pushes code (git push, CI artifact); platform builds and deploys |
| Scaling | Customer configures auto-scaling groups and load balancers manually | Platform scales instances automatically based on traffic or rules |
| Customization and control | Full root access, any OS, any custom software stack | Constrained to platform-supported languages, frameworks, and versions |
| Failure recovery | Customer builds and monitors health checks, failover, and backups | Platform handles instance replacement and basic health monitoring |
| Vendor lock-in | Low — standard VM images port across most cloud providers | Higher — apps depend on platform-specific APIs and build conventions |
| Typical adopter | Infrastructure/ops teams migrating or replicating existing systems | Application developers who want to ship features without managing servers |

## Key Differences

- IaaS gives you a <strong class="kw">virtual machine</strong>; PaaS gives you a <strong class="kw">managed runtime</strong> for your code
- PaaS abstracts away <strong class="kw">OS patching</strong>, which IaaS leaves entirely to the customer
- IaaS deployment means configuring servers yourself; PaaS deployment is typically a <strong class="kw">git push</strong>
- PaaS trades flexibility for speed, increasing <strong class="kw">vendor lock-in</strong> compared to IaaS
- Auto-scaling is built into PaaS, whereas IaaS requires customer-configured <strong class="kw">scaling groups</strong>

## When to Use Each

**IaaS**

- **Lift-and-shift migration**: IaaS lets you replicate an existing on-prem server setup with minimal changes to the OS or software stack.
- **Custom or legacy software stacks**: Full root access lets you run non-standard runtimes, kernel modules, or licensed software PaaS platforms don't support.
- **Fine-grained cost and performance tuning**: Direct control over VM sizing, storage type, and networking lets ops teams optimize for specific workloads.

**PaaS**

- **Rapid application deployment**: Developers push code and let the platform handle builds, patching, and scaling without provisioning servers.
- **Small teams without dedicated ops**: PaaS removes the need for in-house expertise in OS hardening, patch cycles, and infrastructure monitoring.
- **Standard web and API workloads**: Common frameworks and languages are natively supported, so there's little benefit to managing the OS layer yourself.
