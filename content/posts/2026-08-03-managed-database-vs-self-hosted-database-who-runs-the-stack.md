---
title: "Managed Database vs Self-Hosted Database: Who Runs the Stack"
date: 2026-08-03T06:30:21.915732+09:00
tags: ["database", "cloud", "devops", "infrastructure"]
---
## Overview

A managed database hands the operating system, patching, backups, and failover to a <strong class="kw">cloud provider</strong>, while a self-hosted database keeps the entire stack under your team's <strong class="kw">direct control</strong>. The choice trades operational convenience against flexibility, cost structure, and how much low-level tuning you're allowed to do.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Managed Database</text><text x="480" y="30" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Self-Hosted Database</text><rect x="60" y="55" width="200" height="45" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="82" text-anchor="middle" font-size="12" style="fill:var(--content)">Application / Queries</text><rect x="50" y="110" width="220" height="195" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="128" text-anchor="middle" font-size="12" font-weight="600" style="fill:var(--compare-a)">Provider Manages</text><rect x="70" y="140" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">DB Engine</text><rect x="70" y="195" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="219" text-anchor="middle" font-size="12" style="fill:var(--content)">Operating System</text><rect x="70" y="250" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="274" text-anchor="middle" font-size="12" style="fill:var(--content)">Hardware / Storage</text><text x="160" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Less control, less toil</text><rect x="370" y="55" width="220" height="250" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="73" text-anchor="middle" font-size="12" font-weight="600" style="fill:var(--compare-b)">You Manage Everything</text><rect x="390" y="85" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="109" text-anchor="middle" font-size="12" style="fill:var(--content)">Application / Queries</text><rect x="390" y="140" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">DB Engine</text><rect x="390" y="195" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="219" text-anchor="middle" font-size="12" style="fill:var(--content)">Operating System</text><rect x="390" y="250" width="180" height="40" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="274" text-anchor="middle" font-size="12" style="fill:var(--content)">Hardware / Storage</text><text x="480" y="330" text-anchor="middle" font-size="11" style="fill:var(--secondary)">More control, more toil</text></svg>
</div>

## Comparison Table

| Aspect | Managed Database | Self-Hosted Database |
| --- | --- | --- |
| Provisioning & setup | Spin up via console or API in minutes; provider installs and configures the engine | Manually install and configure the OS, storage, and database software yourself |
| Configuration & tuning access | Limited to exposed parameters and flags; some engine internals and OS access are locked | Full root or admin access to every config file, kernel setting, and storage layout |
| Scaling | Resize compute or add read replicas with a click or API call; provider automates the process | Provision new hardware and reconfigure sharding or replication topology yourself |
| Backups & recovery | Automated snapshots and point-in-time restore built into the service | You script, schedule, and test your own backup and restore procedures |
| Patching & upgrades | Provider applies OS and engine security patches on a maintenance schedule | You plan, test, and execute every patch and major version upgrade |
| High availability & failover | Multi-AZ replication and automatic failover configured with a toggle | You design, build, and test the replication and failover setup yourself |
| Monitoring & support | Built-in dashboards and alerts, plus vendor support tickets for engine-level issues | You assemble your own monitoring stack; support is internal or community-based |
| Cost model | Higher per-hour price that bundles operational labor into the bill | Lower raw infrastructure cost but a hidden cost in engineering time |

## Key Differences

- Managed services abstract patching and OS maintenance behind a <strong class="kw">provider</strong> SLA.
- Self-hosted setups grant full <strong class="kw">root access</strong> to tune kernel, storage, and engine internals.
- Failover and multi-AZ replication are <strong class="kw">automated</strong> in managed offerings but hand-built elsewhere.
- Cost shifts from engineering hours to a recurring <strong class="kw">subscription fee</strong> with managed databases.
- Self-hosting permits any <strong class="kw">custom extension</strong> or fork that managed platforms often restrict.

## When to Use Each

**Managed Database**

- **Small or lean ops team**: Offloading patching, backups, and failover to a provider frees a small team from round-the-clock database maintenance.
- **Rapid, unpredictable scaling**: Managed platforms let you resize compute or add replicas on demand without procuring new hardware.
- **Compliance via provider certifications**: Provider-held certifications (SOC 2, HIPAA, PCI) can shortcut your own compliance audit work.

**Self-Hosted Database**

- **Custom engine forks or extensions**: Self-hosting lets you run patched forks, exotic extensions, or plugins that managed services won't allow.
- **Strict data residency control**: Full control over hardware and network placement satisfies data sovereignty rules managed offerings can't guarantee.
- **Predictable high-volume workloads**: At steady, large scale, owning the infrastructure can be cheaper than paying a per-hour managed premium indefinitely.
