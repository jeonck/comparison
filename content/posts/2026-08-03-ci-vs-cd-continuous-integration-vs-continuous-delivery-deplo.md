---
title: "CI vs CD: Continuous Integration vs Continuous Delivery/Deployment"
date: 2026-08-03T05:15:02.837625+09:00
tags: ["ci-cd", "devops", "software-delivery", "automation"]
---
## Overview

CI (Continuous Integration) and CD (Continuous Delivery/Deployment) are two connected stages of the same pipeline, not synonyms. CI focuses on <strong class="kw">automated testing</strong> of every code change as it merges, while CD focuses on <strong class="kw">automated deployment</strong> of that validated code into staging or production. Teams that only automate the first half often assume they have full 'CI/CD' when they've really just automated build-and-test.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg" font-family="sans-serif"><line x1="25" y1="140" x2="315" y2="140" style="stroke:var(--compare-a)" stroke-width="2"/><line x1="25" y1="140" x2="25" y2="150" style="stroke:var(--compare-a)" stroke-width="2"/><line x1="315" y1="140" x2="315" y2="150" style="stroke:var(--compare-a)" stroke-width="2"/><text x="170" y="122" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">CI</text><line x1="325" y1="140" x2="615" y2="140" style="stroke:var(--compare-b)" stroke-width="2"/><line x1="325" y1="140" x2="325" y2="150" style="stroke:var(--compare-b)" stroke-width="2"/><line x1="615" y1="140" x2="615" y2="150" style="stroke:var(--compare-b)" stroke-width="2"/><text x="470" y="122" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">CD</text><rect x="25" y="150" width="90" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Commit</text><rect x="125" y="150" width="90" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Build</text><rect x="225" y="150" width="90" height="60" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="270" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Test</text><rect x="325" y="150" width="90" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="370" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Package</text><rect x="425" y="150" width="90" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="470" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Staging</text><rect x="525" y="150" width="90" height="60" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="570" y="185" text-anchor="middle" font-size="13" style="fill:var(--content)">Production</text><line x1="115" y1="180" x2="122" y2="180" style="stroke:var(--border)" stroke-width="2"/><polygon points="125,180 120,176 120,184" style="fill:var(--border)"/><line x1="215" y1="180" x2="222" y2="180" style="stroke:var(--border)" stroke-width="2"/><polygon points="225,180 220,176 220,184" style="fill:var(--border)"/><line x1="315" y1="180" x2="322" y2="180" style="stroke:var(--border)" stroke-width="2"/><polygon points="325,180 320,176 320,184" style="fill:var(--border)"/><line x1="415" y1="180" x2="422" y2="180" style="stroke:var(--border)" stroke-width="2"/><polygon points="425,180 420,176 420,184" style="fill:var(--border)"/><line x1="515" y1="180" x2="522" y2="180" style="stroke:var(--border)" stroke-width="2"/><polygon points="525,180 520,176 520,184" style="fill:var(--border)"/><text x="170" y="245" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Runs automatically on every commit</text><rect x="325" y="225" width="290" height="55" rx="4" fill="none" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="470" y="248" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Delivery: manual approval before Production</text><text x="470" y="266" text-anchor="middle" font-size="12" style="fill:var(--secondary)">Deployment: fully automatic, no gate</text><text x="320" y="330" text-anchor="middle" font-size="13" style="fill:var(--content)">CI verifies the code — CD gets it running</text></svg>
</div>

## Comparison Table

| Aspect | CI (Continuous Integration) | CD (Continuous Delivery/Deployment) |
| --- | --- | --- |
| Trigger | Code commit or push to a shared repository | A successful CI run producing a new build artifact |
| Core process | Compile, run unit/integration tests, static analysis | Package the artifact, provision environments, run deployment scripts |
| Primary output | A verified, mergeable build artifact | A running release in staging and/or production |
| Environment scope | Build server or ephemeral test environment | Staging and/or production environments |
| Human gate | None — fully automated on every commit | Optional manual approval (Delivery) or none (Deployment) |
| Release cadence | N/A — runs per commit, not a release event | Can range from multiple times a day to once per commit |
| Failure consequence | Blocks the merge; breaks the build for the team | Blocks or rolls back the release; production stays on the last good version |
| Primary goal | Catch integration bugs early, keep the main branch releasable | Get every releasable build to users quickly and safely |

## Key Differences

- CI's output is a verified <strong class="kw">build artifact</strong>, not a live release
- CD adds environment <strong class="kw">provisioning</strong> and deployment steps that CI never performs
- A CI failure blocks the <strong class="kw">merge</strong>; a CD failure blocks the rollout instead
- Continuous Delivery keeps a manual <strong class="kw">approval gate</strong> before production, while Continuous Deployment removes it
- CI runs on every single commit; CD can be throttled by environment or business <strong class="kw">readiness</strong>

## When to Use Each

**CI (Continuous Integration)**

- **Fast feedback on commits**: CI gives every developer immediate signal on whether their change broke the build or existing tests.
- **Enforcing a releasable main branch**: CI keeps the trunk always in a mergeable, working state by rejecting broken commits before they combine.
- **Catching integration bugs early**: Running the full test suite on every merge surfaces conflicts between parallel changes before they compound.

**CD (Continuous Delivery/Deployment)**

- **Automating environment rollout**: CD removes manual, error-prone steps like provisioning servers or running migration scripts on release day.
- **Shipping features quickly**: CD (as Continuous Deployment) lets validated changes reach users within minutes of merging, without a release queue.
- **Staged rollouts with sign-off**: CD (as Continuous Delivery) keeps a human approval step for regulated or high-risk production releases.
