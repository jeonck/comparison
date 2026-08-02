---
title: "Terraform vs Ansible: Provisioning vs Configuration Management"
date: 2026-08-02T11:25:27.984535+09:00
tags: ["terraform", "ansible", "infrastructure-as-code", "devops"]
---
## Overview

Terraform and Ansible are both infrastructure-as-code tools but solve different problems: Terraform declaratively provisions and tracks cloud infrastructure using a state file, while Ansible procedurally configures and manages software on existing hosts with no persistent state. Many teams use them together — Terraform to stand up infrastructure, Ansible to configure it.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="335" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="170" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Terraform</text><text x="170" y="44" text-anchor="middle" style="fill:var(--secondary)" font-size="11">declarative</text><rect x="70" y="52" width="200" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="72" text-anchor="middle" style="fill:var(--content)" font-size="11">main.tf (desired state)</text><line x1="170" y1="84" x2="170" y2="98" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="70" y="98" width="200" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="118" text-anchor="middle" style="fill:var(--content)" font-size="11">state file (source of truth)</text><line x1="170" y1="130" x2="170" y2="152" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="110" y="152" width="120" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="171" text-anchor="middle" style="fill:var(--content)" font-size="10">dependency graph</text><line x1="150" y1="182" x2="65" y2="244" style="stroke:var(--compare-a)" stroke-width="1.2"/><line x1="170" y1="182" x2="155" y2="244" style="stroke:var(--compare-a)" stroke-width="1.2"/><line x1="190" y1="182" x2="245" y2="244" style="stroke:var(--compare-a)" stroke-width="1.2"/><rect x="25" y="244" width="80" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="65" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">VM</text><rect x="115" y="244" width="80" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">Network</text><rect x="205" y="244" width="80" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="245" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">DB</text><text x="170" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="9.5">converges infra to match state</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Ansible</text><text x="480" y="44" text-anchor="middle" style="fill:var(--secondary)" font-size="11">procedural</text><rect x="380" y="52" width="200" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="72" text-anchor="middle" style="fill:var(--content)" font-size="11">playbook.yml</text><line x1="480" y1="84" x2="480" y2="96" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="400" y="96" width="160" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="113" text-anchor="middle" style="fill:var(--content)" font-size="10">task 1: install pkg</text><line x1="480" y1="122" x2="480" y2="132" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="400" y="132" width="160" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="149" text-anchor="middle" style="fill:var(--content)" font-size="10">task 2: configure</text><line x1="480" y1="158" x2="480" y2="168" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="400" y="168" width="160" height="26" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="185" text-anchor="middle" style="fill:var(--content)" font-size="10">task 3: start service</text><line x1="460" y1="194" x2="385" y2="244" style="stroke:var(--compare-b)" stroke-width="1.2" stroke-dasharray="3 3"/><line x1="480" y1="194" x2="475" y2="244" style="stroke:var(--compare-b)" stroke-width="1.2" stroke-dasharray="3 3"/><line x1="500" y1="194" x2="565" y2="244" style="stroke:var(--compare-b)" stroke-width="1.2" stroke-dasharray="3 3"/><rect x="345" y="244" width="80" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="385" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">Server A</text><rect x="435" y="244" width="80" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="475" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">Server B</text><rect x="525" y="244" width="80" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="565" y="264" text-anchor="middle" style="fill:var(--content)" font-size="10">Server C</text><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="9.5">sequential push over SSH, no state file</text></svg>
</div>

## Comparison Table

| Aspect | Terraform | Ansible |
| --- | --- | --- |
| Primary purpose | Provision and tear down cloud/infra resources (VMs, networks, DBs) | Configure software and manage state on existing hosts |
| Configuration language | HCL (HashiCorp Configuration Language), declarative | YAML playbooks, procedural task lists |
| Execution model | Builds a dependency graph and applies changes in parallel where possible | Executes tasks sequentially, in order, per host |
| State management | Maintains a state file mapping config to real resources | Stateless — queries live system facts on each run |
| Idempotency approach | Diffs desired config against state file before acting | Each module checks current condition before making a change |
| Connectivity/agent requirement | Agentless — calls cloud/provider APIs directly | Agentless — connects over SSH or WinRM to target hosts |
| Failure & recovery handling | Partial applies are resolved by re-running against the state file | Reruns the playbook from the start; tasks are re-checked, not resumed |

## Key Differences

- Terraform tracks infrastructure in a persistent <strong class="kw">state file</strong>; Ansible has no state store and reads live system facts each run.
- Terraform resolves a <strong class="kw">dependency graph</strong> to apply changes in parallel; Ansible runs tasks <strong class="kw">sequentially</strong>.
- Terraform talks to infrastructure through provider <strong class="kw">APIs</strong>; Ansible connects to hosts via <strong class="kw">SSH/WinRM</strong>.
- Terraform is built for <strong class="kw">provisioning</strong> infra; Ansible is built for <strong class="kw">configuration</strong> of what already exists.
- They're commonly paired: Terraform creates the servers, then Ansible <strong class="kw">configures</strong> them.

## When to Use Each

**Terraform**

- **Provisioning Cloud Infrastructure**: Creating and tearing down VMs, networks, and managed services is exactly the resource-lifecycle problem Terraform's provider APIs and dependency graph are built for.
- **Needing a Source of Truth for What Exists**: The state file gives you a queryable record of every resource under management, which Ansible's stateless model doesn't provide.
- **Parallel, Dependency-Aware Rollouts**: When resources depend on each other, Terraform's graph-based apply creates them in the right order automatically, in parallel where safe.

**Ansible**

- **Configuring Software on Existing Hosts**: Installing packages, writing config files, and starting services on servers that already exist is Ansible's core job, with no infrastructure to provision.
- **Orchestrating Multi-Server Deployment Steps**: Sequential, ordered task execution across many hosts over SSH suits rollout playbooks better than a declarative state diff.
- **Agentless Configuration Without Persistent State**: When you don't want to manage a state file and prefer each run to check live system facts directly, Ansible's stateless design fits.
