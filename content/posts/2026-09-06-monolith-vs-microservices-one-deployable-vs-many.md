---
title: "Monolith vs Microservices: One Deployable vs Many"
date: 2026-09-06T09:51:18.391998+09:00
tags: ["architecture", "system-design", "monolith", "microservices"]
---
## Overview

A <strong class="kw">monolith</strong> packages an application's entire codebase and functionality into a single deployable unit running as one process, while <strong class="kw">microservices</strong> split that same functionality into independently deployable services that communicate over a network. The choice shapes how teams build, deploy, scale, and recover from failures, so it matters far beyond just code organization.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="36" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Monolith</text><rect x="40" y="60" width="240" height="260" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><line x1="160" y1="60" x2="160" y2="320" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><line x1="40" y1="190" x2="280" y2="190" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><text x="100" y="128" text-anchor="middle" font-size="13" style="fill:var(--content)">Auth</text><text x="220" y="128" text-anchor="middle" font-size="13" style="fill:var(--content)">Orders</text><text x="100" y="258" text-anchor="middle" font-size="13" style="fill:var(--content)">Inventory</text><text x="220" y="258" text-anchor="middle" font-size="13" style="fill:var(--content)">Payments</text><text x="160" y="338" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Single process</text><text x="160" y="352" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Single deploy</text><text x="480" y="36" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Microservices</text><rect x="395" y="60" width="110" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="450" y="83" text-anchor="middle" font-size="12" style="fill:var(--content)">API Gateway</text><line x1="420" y1="96" x2="375" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="445" y1="96" x2="445" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="455" y1="96" x2="515" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="480" y1="96" x2="575" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="345" y="150" width="60" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="375" y="180" text-anchor="middle" font-size="11" style="fill:var(--content)">Auth</text><rect x="415" y="150" width="60" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="445" y="180" text-anchor="middle" font-size="11" style="fill:var(--content)">Orders</text><rect x="485" y="150" width="60" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="515" y="180" text-anchor="middle" font-size="11" style="fill:var(--content)">Inventory</text><rect x="545" y="150" width="60" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="575" y="180" text-anchor="middle" font-size="11" style="fill:var(--content)">Payments</text><line x1="375" y1="215" x2="375" y2="245" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><line x1="445" y1="215" x2="445" y2="245" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><line x1="515" y1="215" x2="515" y2="245" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><line x1="575" y1="215" x2="575" y2="245" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><rect x="345" y="245" width="260" height="30" rx="3" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="3 3"/><text x="475" y="264" text-anchor="middle" font-size="10" style="fill:var(--secondary)">separate databases</text><text x="475" y="338" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Independent services</text><text x="475" y="352" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Independent deploys</text></svg>
</div>

## Comparison Table

| Aspect | Monolith | Microservices |
| --- | --- | --- |
| Codebase structure | Single repository, one shared codebase for all functionality | Multiple repositories, one per service with its own codebase |
| Deployment unit | Whole application built and shipped as one artifact | Each service built, versioned, and shipped independently |
| Inter-module communication | In-process function calls within the same runtime | Network calls via HTTP, gRPC, or messaging between services |
| Data storage | Typically one shared database for the whole app | Each service usually owns its own database or schema |
| Scaling | Scale the entire application even if only one part is hot | Scale only the specific services that need more capacity |
| Fault isolation | A crash or memory leak in one module can take down the app | A failing service degrades its own function without necessarily crashing others |
| Technology stack | One language and framework across the whole application | Each service can use the language/framework best suited to it |
| Team ownership and releases | One team or a coordinated release train ships the whole app together | Independent teams own and release their services on their own schedules |

## Key Differences

- A monolith runs as a <strong class="kw">single process</strong>, while microservices are <strong class="kw">distributed processes</strong> talking over the network
- Microservices trade in-process call reliability for <strong class="kw">network latency</strong> and partial failure handling
- Independent deployability lets microservices teams ship on <strong class="kw">separate release cadences</strong>, which a monolith can't offer
- Splitting services adds real <strong class="kw">operational overhead</strong> — service discovery, monitoring, and distributed tracing
- Data ownership per service enables <strong class="kw">polyglot persistence</strong> but sacrifices easy cross-entity transactions

## When to Use Each

**Monolith**

- **Early-stage MVP**: A monolith lets a small team ship and iterate fast without the overhead of managing distributed infrastructure.
- **Small, tightly coupled domain**: When features are inherently interdependent, keeping them in one codebase avoids artificial network boundaries.
- **Limited ops capacity**: A single deployable is far simpler to monitor, debug, and operate for a team without dedicated platform engineers.

**Microservices**

- **Large multi-team org**: Independent services let dozens of teams own, build, and release their piece without blocking on each other.
- **Uneven scaling needs**: Services with very different load profiles can be scaled independently instead of over-provisioning the whole app.
- **Polyglot requirements**: Different services can use the language or data store best suited to their specific workload.
