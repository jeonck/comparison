---
title: "Role vs ClusterRole: Kubernetes RBAC Scope Compared"
date: 2026-08-02T11:24:24.318508+09:00
tags: ["kubernetes", "rbac", "access-control", "cluster-administration"]
---
## Overview

Role and ClusterRole are both Kubernetes RBAC objects that define sets of permission rules (verbs on resources), but they differ in scope: a Role only applies within a single namespace, while a ClusterRole is defined once for the whole cluster and can be bound either cluster-wide or scoped down to one namespace. Understanding this distinction is essential for applying least-privilege access control in multi-tenant clusters.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0L10,5L0,10z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0L10,5L0,10z" style="fill:var(--compare-b)"/></marker></defs><text x="155" y="35" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Role</text><text x="477" y="35" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">ClusterRole</text><rect x="30" y="55" width="250" height="270" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="5 5"/><text x="45" y="75" font-size="12" style="fill:var(--secondary)">Namespace: dev</text><rect x="90" y="95" width="130" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="122" text-anchor="middle" font-size="14" style="fill:var(--content)">Role</text><line x1="155" y1="139" x2="155" y2="174" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><rect x="90" y="177" width="130" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="202" text-anchor="middle" font-size="13" style="fill:var(--content)">RoleBinding</text><line x1="155" y1="217" x2="155" y2="254" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><circle cx="155" cy="278" r="20" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="155" y="282" text-anchor="middle" font-size="11" style="fill:var(--content)">User</text><text x="155" y="316" text-anchor="middle" font-size="10" style="fill:var(--secondary)">limited to this namespace</text><rect x="345" y="55" width="265" height="270" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="360" y="75" font-size="12" style="fill:var(--secondary)">Cluster scope</text><rect x="412" y="95" width="140" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="482" y="122" text-anchor="middle" font-size="14" style="fill:var(--content)">ClusterRole</text><line x1="450" y1="139" x2="417" y2="174" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="514" y1="139" x2="558" y2="174" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><rect x="360" y="177" width="110" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="415" y="199" text-anchor="middle" font-size="11" style="fill:var(--content)">RoleBinding</text><rect x="495" y="177" width="130" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="199" text-anchor="middle" font-size="10" style="fill:var(--content)">ClusterRoleBinding</text><line x1="415" y1="213" x2="415" y2="248" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="560" y1="213" x2="560" y2="248" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><circle cx="415" cy="268" r="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="415" y="272" text-anchor="middle" font-size="10" style="fill:var(--content)">User</text><text x="415" y="300" text-anchor="middle" font-size="10" style="fill:var(--secondary)">this ns only</text><circle cx="560" cy="268" r="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="272" text-anchor="middle" font-size="10" style="fill:var(--content)">User</text><text x="560" y="300" text-anchor="middle" font-size="10" style="fill:var(--secondary)">all namespaces</text><text x="477" y="318" text-anchor="middle" font-size="10" style="fill:var(--secondary)">one definition, reused via either binding</text></svg>
</div>

## Comparison Table

| Aspect | Role | ClusterRole |
| --- | --- | --- |
| API object scope | Namespaced object; exists only within one Namespace | Cluster-scoped object; exists once for the entire cluster |
| Resources it can grant access to | Only namespaced resources (pods, configmaps, secrets, etc.) within its own namespace | Namespaced resources cluster-wide plus cluster-scoped resources such as nodes, persistentvolumes, and namespaces |
| Non-resource URLs (e.g. /healthz, /metrics) | Cannot reference non-resource URLs | Can include rules for non-resource URLs |
| Binding object required | RoleBinding only, created in the same namespace | RoleBinding for a namespace-scoped grant, or ClusterRoleBinding for a cluster-wide grant |
| Effective grant when bound | Permissions always limited to the Role's own namespace | Spans every namespace when bound via ClusterRoleBinding, or just one namespace when bound via RoleBinding |
| Reuse across namespaces | Must be duplicated in each namespace that needs the same rules | Defined once, reused across many namespaces or cluster-wide via separate bindings |
| Aggregation support | None; rules are static within the object | Supports aggregationRule to auto-combine rules from other ClusterRoles by label selector |
| Typical built-in examples | None shipped by default; teams author their own per namespace | cluster-admin, admin, edit, view, and system: component roles ship as default ClusterRoles |

## Key Differences

- <strong class="kw">Role</strong> is namespace-scoped while <strong class="kw">ClusterRole</strong> is cluster-scoped by definition, regardless of how it's later bound.
- Only ClusterRole can grant access to cluster-scoped resources like nodes or to <strong class="kw">non-resource URLs</strong> such as /metrics.
- A ClusterRole can still be restricted to one namespace by binding it with a <strong class="kw">RoleBinding</strong> instead of a ClusterRoleBinding.
- ClusterRole supports <strong class="kw">aggregation</strong> to compose permissions from labeled ClusterRoles; Role has no equivalent mechanism.
- Kubernetes ships default admin/edit/view permission sets as <strong class="kw">built-in ClusterRoles</strong>, never as Roles.

## When to Use Each

**Role** — Use a Role when permissions should stay confined to a single namespace or team, which keeps the blast radius minimal and matches a least-privilege, per-tenant access model.

**ClusterRole** — Use a ClusterRole when you need to grant access to cluster-scoped resources or non-resource endpoints, or when you want one reusable permission set applied across many namespaces or the whole cluster.
