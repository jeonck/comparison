---
title: "Declarative vs Imperative IaC: Describing the End State vs Scripting the Steps"
date: 2026-08-02T08:55:05.720390+09:00
tags: ["iac", "terraform", "devops", "cloud-infrastructure"]
---
## Overview

Declarative IaC (e.g. Terraform, CloudFormation) has you specify the desired end state of infrastructure and lets an engine figure out how to get there. Imperative IaC (e.g. shell scripts, Chef recipes, raw CLI calls) has you write the exact sequence of commands to execute. The distinction matters because it determines who — you or the tool — is responsible for ordering, idempotency, and reconciling drift.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="28" text-anchor="middle" font-size="16" style="fill:var(--primary)">Declarative</text><rect x="40" y="50" width="240" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="72" text-anchor="middle" font-size="12" style="fill:var(--content)">Desired State</text><text x="160" y="90" text-anchor="middle" font-size="11" style="fill:var(--secondary)">"3 servers, 1 LB"</text><line x1="160" y1="105" x2="160" y2="130" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,140 155,130 165,130" style="fill:var(--compare-a)"/><rect x="40" y="145" width="240" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="167" text-anchor="middle" font-size="12" style="fill:var(--content)">Engine Computes Diff</text><text x="160" y="185" text-anchor="middle" font-size="11" style="fill:var(--secondary)">plan + dependency graph</text><line x1="160" y1="200" x2="160" y2="225" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,235 155,225 165,225" style="fill:var(--compare-a)"/><rect x="40" y="240" width="240" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="262" text-anchor="middle" font-size="12" style="fill:var(--content)">Infrastructure</text><text x="160" y="280" text-anchor="middle" font-size="11" style="fill:var(--secondary)">converges to match state</text><text x="160" y="320" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Engine decides how &amp; in what order</text><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="480" y="28" text-anchor="middle" font-size="16" style="fill:var(--primary)">Imperative</text><rect x="360" y="45" width="240" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="70" text-anchor="middle" font-size="12" style="fill:var(--content)">Step 1: Create VPC</text><line x1="480" y1="85" x2="480" y2="100" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,110 475,100 485,100" style="fill:var(--compare-b)"/><rect x="360" y="115" width="240" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="140" text-anchor="middle" font-size="12" style="fill:var(--content)">Step 2: Launch Servers</text><line x1="480" y1="155" x2="480" y2="170" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,180 475,170 485,170" style="fill:var(--compare-b)"/><rect x="360" y="185" width="240" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="210" text-anchor="middle" font-size="12" style="fill:var(--content)">Step 3: Attach LB</text><line x1="480" y1="225" x2="480" y2="240" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,250 475,240 485,240" style="fill:var(--compare-b)"/><rect x="360" y="255" width="240" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="280" text-anchor="middle" font-size="12" style="fill:var(--content)">Infrastructure</text><text x="480" y="320" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Author decides exact steps &amp; order</text></svg>
</div>

## Comparison Table

| Aspect | Declarative | Imperative |
| --- | --- | --- |
| Authoring model | Write a <strong class="kw">desired-state spec</strong> | Write an <strong class="kw">ordered command list</strong> |
| Execution engine | Resolves a <strong class="kw">dependency graph</strong> | Runs a <strong class="kw">sequential interpreter</strong> |
| State tracking | Maintains a <strong class="kw">state file</strong> | <strong class="kw">Stateless</strong> execution |
| Applying changes | Single <strong class="kw">apply</strong> command | Run the <strong class="kw">script/playbook</strong> |
| Ordering & dependencies | <strong class="kw">Auto-resolved</strong> by engine | <strong class="kw">Manually sequenced</strong> by author |
| Idempotency | <strong class="kw">Guaranteed</strong> by design | <strong class="kw">Developer-enforced</strong> |
| Drift detection | Built-in <strong class="kw">plan diff</strong> | <strong class="kw">Not built-in</strong> |
| Failure handling | Partial apply, <strong class="kw">replan</strong> | <strong class="kw">Manual rollback</strong> |

## Key Differences

- Declarative code answers 'what', imperative code answers <strong class="kw">'how'</strong>.
- Declarative tools rely on a <strong class="kw">state file</strong> to know current vs. desired infrastructure; imperative scripts have no memory of prior runs.
- Idempotent re-runs are <strong class="kw">automatic</strong> in declarative tools but must be hand-coded (checks, conditionals) in imperative scripts.
- Declarative engines build a <strong class="kw">dependency graph</strong> to order operations; imperative code hardcodes that order line by line.
- Drift correction in declarative IaC is a matter of re-running <strong class="kw">plan/apply</strong>; imperative approaches require re-running or rewriting the exact script.

## When to Use Each

**Declarative**

- **Long-Lived Cloud Infrastructure Provisioning**: A state file that tracks current vs. desired infrastructure gives consistent, repeatable results across repeated applies, which is what tools like Terraform and CloudFormation are built for.
- **Automatic Drift Reconciliation**: The built-in plan diff lets teams detect and correct drift by re-running plan/apply instead of manually auditing what changed.
- **Idempotent, Repeatable Deployments**: Idempotency guaranteed by the engine means re-running the same configuration safely converges to the desired state without hand-written guards.

**Imperative**

- **One-Off Operational Tasks**: A stateless script is simpler for a single ad hoc action that doesn't need lasting drift tracking or a state file.
- **Complex Multi-Step Orchestration With Conditional Logic**: Because imperative code runs as a sequential interpreter, it fits logic-heavy bootstrap or configuration flows that don't map cleanly to a state model.
- **Fine-Grained Manual Control Over Ordering**: Authors who need to hand-sequence operations, such as in Chef recipes or CLI automation, get direct control over execution order that declarative engines abstract away.
