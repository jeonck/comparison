---
title: "Strangler Fig Pattern vs Big Bang Migration: Incremental vs All-at-Once System Replacement"
date: 2026-08-04T05:25:26.635605+09:00
tags: ["migration-patterns", "legacy-systems", "software-architecture", "risk-management"]
---
## Overview

Both are strategies for replacing a legacy system, but they differ in how risk and time are distributed. The <strong class="kw">Strangler Fig Pattern</strong> incrementally routes traffic from old to new components behind a facade until the legacy system is gone, while <strong class="kw">Big Bang Migration</strong> replaces the entire system in one coordinated cutover. The choice shapes how much downtime, rollback flexibility, and sustained engineering effort a team is willing to accept.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Strangler Fig</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Big Bang</text><rect x="30" y="55" width="230" height="30" style="stroke:var(--compare-a);fill:none" stroke-width="1.5"/><rect x="260" y="55" width="30" height="30" style="stroke:var(--compare-a);fill:var(--compare-a-soft)" stroke-width="1.5"/><text x="15" y="75" text-anchor="end" style="fill:var(--secondary)" font-size="11">T1</text><rect x="30" y="125" width="130" height="30" style="stroke:var(--compare-a);fill:none" stroke-width="1.5"/><rect x="160" y="125" width="130" height="30" style="stroke:var(--compare-a);fill:var(--compare-a-soft)" stroke-width="1.5"/><text x="15" y="145" text-anchor="end" style="fill:var(--secondary)" font-size="11">T2</text><rect x="30" y="195" width="30" height="30" style="stroke:var(--compare-a);fill:none" stroke-width="1.5"/><rect x="60" y="195" width="230" height="30" style="stroke:var(--compare-a);fill:var(--compare-a-soft)" stroke-width="1.5"/><text x="15" y="215" text-anchor="end" style="fill:var(--secondary)" font-size="11">T3</text><rect x="30" y="250" width="14" height="14" style="stroke:var(--compare-a);fill:none" stroke-width="1.5"/><text x="50" y="261" style="fill:var(--content)" font-size="11">legacy</text><rect x="150" y="250" width="14" height="14" style="stroke:var(--compare-a);fill:var(--compare-a-soft)" stroke-width="1.5"/><text x="170" y="261" style="fill:var(--content)" font-size="11">new</text><text x="160" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Facade routes traffic; legacy shrinks gradually</text><rect x="350" y="140" width="100" height="60" style="stroke:var(--compare-b);fill:var(--compare-b-soft)" stroke-width="1.5"/><text x="400" y="165" text-anchor="middle" style="fill:var(--content)" font-size="12">Legacy</text><text x="400" y="182" text-anchor="middle" style="fill:var(--content)" font-size="12">System</text><rect x="530" y="140" width="100" height="60" style="stroke:var(--compare-b);fill:var(--compare-b-soft)" stroke-width="1.5"/><text x="580" y="165" text-anchor="middle" style="fill:var(--content)" font-size="12">New</text><text x="580" y="182" text-anchor="middle" style="fill:var(--content)" font-size="12">System</text><line x1="452" y1="170" x2="525" y2="170" style="stroke:var(--compare-b)" stroke-width="2"/><polygon points="525,170 515,164 515,176" style="fill:var(--compare-b);stroke:var(--compare-b)"/><text x="488" y="130" text-anchor="middle" style="fill:var(--secondary)" font-size="11">single cutover</text><polygon points="488,90 478,108 498,108" style="stroke:var(--compare-b);fill:var(--compare-b-soft)" stroke-width="1.5"/><text x="488" y="105" text-anchor="middle" style="fill:var(--primary)" font-size="12" font-weight="bold">!</text><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Entire system replaced at once, high risk</text></svg>
</div>

## Comparison Table

| Aspect | Strangler Fig Pattern | Big Bang Migration |
| --- | --- | --- |
| Core approach | Incrementally replace pieces of the legacy system behind a facade until nothing legacy remains | Replace the entire legacy system with the new system in one coordinated cutover |
| Upfront design | Requires a routing/facade layer and clear service boundaries defined before starting | Requires the new system built to full feature parity before any cutover |
| Traffic routing | A facade or proxy intercepts requests and routes them to legacy or new components as they're migrated | No intermediate routing layer; all traffic points to legacy until the single cutover moment |
| Execution timeline | Spans weeks to years, delivered as many small, independent releases | Concentrated into a single migration event, often a scheduled outage window |
| Rollback capability | Easy to roll back a single migrated component by routing traffic back to legacy | Rolling back means reverting the entire system, which is costly and often impractical |
| Risk exposure | Risk is distributed across many small, independently testable changes | Risk is concentrated in one high-stakes event where failure affects the whole system |
| Legacy decommissioning | Legacy system shrinks piece by piece until it can be retired entirely | Legacy system is decommissioned immediately once the new system goes live |

## Key Differences

- Strangler Fig routes traffic through a <strong class="kw">facade</strong>, letting legacy and new code coexist; Big Bang has no such intermediary.
- Big Bang concentrates all risk into one <strong class="kw">cutover</strong> event, while Strangler Fig spreads it across many small releases.
- Strangler Fig supports granular <strong class="kw">rollback</strong> per component; Big Bang rollback requires reverting the whole system.
- Strangler Fig migrations run for months or years; Big Bang fits a fixed <strong class="kw">deadline</strong>.
- Strangler Fig requires maintaining <strong class="kw">dual systems</strong> temporarily; Big Bang avoids that overhead entirely.

## When to Use Each

**Strangler Fig Pattern**

- **Large, business-critical systems**: Downtime or full-system risk is unacceptable, so replacing components incrementally keeps the system running throughout.
- **Poorly understood legacy systems**: Incremental replacement surfaces hidden behavior and dependencies as each piece is migrated, rather than assuming full understanding upfront.
- **Long-lived migrations with limited staffing**: Work can be spread across many small releases over an extended timeframe without needing a dedicated migration freeze.

**Big Bang Migration**

- **Small, well-scoped systems**: Full feature parity is achievable in a single build, making a facade and routing layer unnecessary overhead.
- **Systems with a hard deadline**: A vendor contract expiration or license end-of-life can force a one-time cutover on a fixed date.
- **Short-lived or rarely used systems**: Building a facade layer isn't worth it when the system will only exist briefly or sees infrequent traffic.
