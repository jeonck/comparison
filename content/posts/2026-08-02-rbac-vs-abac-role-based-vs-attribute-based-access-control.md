---
title: "RBAC vs ABAC: Role-Based vs Attribute-Based Access Control"
date: 2026-08-02T11:22:33.082003+09:00
tags: ["access-control", "security", "authorization", "identity-management"]
---
## Overview

RBAC and ABAC are two models for deciding whether a subject can perform an action on a resource. RBAC grants access based on a user's assigned role and that role's fixed permission set, while ABAC evaluates a policy against attributes of the user, resource, action, and environment at request time. The choice affects how fine-grained, dynamic, and auditable your authorization system can be.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="325" y1="40" x2="325" y2="320" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="170" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">RBAC</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">ABAC</text><rect x="110" y="50" width="120" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="75" text-anchor="middle" style="fill:var(--content)" font-size="13">User</text><line x1="170" y1="90" x2="170" y2="116" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="170,120 165,110 175,110" style="fill:var(--compare-a)"/><rect x="100" y="120" width="140" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="145" text-anchor="middle" style="fill:var(--content)" font-size="13">Role: Editor</text><line x1="170" y1="160" x2="170" y2="186" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="170,190 165,180 175,180" style="fill:var(--compare-a)"/><rect x="90" y="190" width="160" height="90" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="170" y="210" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">Permissions</text><text x="170" y="230" text-anchor="middle" style="fill:var(--content)" font-size="12">Read</text><text x="170" y="248" text-anchor="middle" style="fill:var(--content)" font-size="12">Write</text><text x="170" y="266" text-anchor="middle" style="fill:var(--content)" font-size="12">Publish</text><text x="170" y="305" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Fixed, regardless of context</text><rect x="350" y="50" width="110" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="405" y="72" text-anchor="middle" style="fill:var(--content)" font-size="11">User attrs</text><rect x="470" y="50" width="120" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="72" text-anchor="middle" style="fill:var(--content)" font-size="11">Resource attrs</text><rect x="410" y="96" width="140" height="36" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="118" text-anchor="middle" style="fill:var(--content)" font-size="11">Env attrs</text><line x1="405" y1="86" x2="470" y2="158" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="530" y1="86" x2="490" y2="158" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="480" y1="132" x2="480" y2="158" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,162 474,150 486,150" style="fill:var(--compare-b)"/><rect x="410" y="162" width="140" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="187" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">Policy Engine</text><line x1="480" y1="202" x2="480" y2="221" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="480,225 475,215 485,215" style="fill:var(--compare-b)"/><rect x="420" y="225" width="120" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="250" text-anchor="middle" style="fill:var(--content)" font-size="12">Allow / Deny</text><text x="480" y="305" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Evaluated per request, in context</text></svg>
</div>

## Comparison Table

| Aspect | RBAC | ABAC |
| --- | --- | --- |
| Access decision basis | A user's assigned role | Attributes of the user, resource, action, and environment |
| Permission structure | Static, predefined role-to-permission mappings | Dynamic policies expressed as attribute-based rules |
| Administration | Admin assigns users to existing roles | Policy author writes rules combining attribute conditions |
| Runtime evaluation | Check whether the user's role includes the requested permission | Policy engine evaluates rules against current attribute values |
| Context sensitivity | Same result regardless of time, location, or device | Can factor in time, location, device, and other real-time signals |
| Granularity | Coarse-grained, applied per role | Fine-grained, applied per request or condition |
| Scalability with complexity | Role explosion as requirements diversify | Policy complexity grows, but avoids proliferating roles |
| Auditability | Easy to audit — list who holds a given role | Harder to audit — requires tracing policy logic across attributes |

## Key Differences

- RBAC ties access to <strong class="kw">roles</strong>; ABAC ties access to <strong class="kw">attributes</strong>
- RBAC decisions are <strong class="kw">static</strong>; ABAC decisions are <strong class="kw">context-aware</strong>
- ABAC enables <strong class="kw">fine-grained</strong> control at the cost of policy complexity
- RBAC suffers from <strong class="kw">role explosion</strong> as requirements grow
- RBAC is generally easier to <strong class="kw">audit</strong> than ABAC

## When to Use Each

**RBAC** — Use RBAC when job functions are well-defined and stable, and you need a simple, easily auditable model — e.g. internal admin tools or systems with a small, fixed set of user types.

**ABAC** — Use ABAC when access decisions depend on dynamic context like time, location, or resource ownership — e.g. multi-tenant SaaS, regulated data access, or environments requiring fine-grained, condition-based policies.
