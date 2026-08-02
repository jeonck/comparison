---
title: "Immutable vs Mutable Infrastructure: Replace vs Patch"
date: 2026-08-03T05:18:27.940116+09:00
tags: ["infrastructure", "devops", "deployment", "configuration-management"]
---
## Overview

Immutable infrastructure treats servers as disposable artifacts — every change ships as a brand-new <strong class="kw">image</strong> that replaces the running instance rather than editing it. Mutable infrastructure instead <strong class="kw">updates in place</strong>, applying patches and config changes directly to long-lived servers. The choice determines how predictable, auditable, and drift-resistant your environments are.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--compare-b)"/></marker><marker id="arrowN" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 z" style="fill:var(--secondary)"/></marker></defs><line x1="320" y1="20" x2="320" y2="335" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="15" font-weight="bold">Immutable Infrastructure</text><text x="480" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="15" font-weight="bold">Mutable Infrastructure</text><rect x="45" y="55" width="100" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="78" text-anchor="middle" style="fill:var(--content)" font-size="12">Image v1</text><line x1="95" y1="93" x2="95" y2="118" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="45" y="120" width="100" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="149" text-anchor="middle" style="fill:var(--content)" font-size="12">Server v1</text><line x1="95" y1="170" x2="95" y2="195" style="stroke:var(--secondary)" stroke-width="1.5" marker-end="url(#arrowN)"/><rect x="45" y="197" width="100" height="34" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3 3"/><text x="95" y="218" text-anchor="middle" style="fill:var(--secondary)" font-size="11">terminated</text><rect x="175" y="55" width="100" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="225" y="78" text-anchor="middle" style="fill:var(--content)" font-size="12">Image v2</text><line x1="225" y1="93" x2="225" y2="118" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="175" y="120" width="100" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="225" y="149" text-anchor="middle" style="fill:var(--content)" font-size="12">Server v2</text><line x1="225" y1="170" x2="225" y2="195" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="175" y="197" width="100" height="34" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="225" y="218" text-anchor="middle" style="fill:var(--content)" font-size="11">live traffic</text><text x="160" y="255" text-anchor="middle" style="fill:var(--secondary)" font-size="11">change = build new image,</text><text x="160" y="270" text-anchor="middle" style="fill:var(--secondary)" font-size="11">deploy new instance, discard old</text><rect x="410" y="130" width="140" height="80" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="160" text-anchor="middle" style="fill:var(--content)" font-size="12">Server</text><text x="480" y="180" text-anchor="middle" style="fill:var(--content)" font-size="11">v1 &#8594; v1.1 &#8594; v1.2</text><path d="M 460 130 A 40 30 0 1 1 500 130" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><text x="480" y="95" text-anchor="middle" style="fill:var(--content)" font-size="11">patch in place</text><text x="480" y="255" text-anchor="middle" style="fill:var(--secondary)" font-size="11">change = SSH in / run config</text><text x="480" y="270" text-anchor="middle" style="fill:var(--secondary)" font-size="11">management, edit same instance</text></svg>
</div>

## Comparison Table

| Aspect | Immutable Infrastructure | Mutable Infrastructure |
| --- | --- | --- |
| Initial provisioning | Instance built once from a versioned image or template | Instance provisioned once, then edited repeatedly over its life |
| Applying a change | Rebuild the image with the change baked in | SSH in, or run a config-management tool, against the live server |
| Deployment mechanism | Orchestrator replaces old instances with new ones (rolling/blue-green) | Update scripts or agents (Ansible, Chef, Puppet) mutate the running instance |
| Configuration drift | Cannot occur — every instance matches its source image exactly | Accumulates over time as ad hoc changes diverge from documented state |
| Rollback | Redeploy the prior image version, deterministic and fast | Manually reverse changes on the server, often incomplete or unreliable |
| Emergency hotfixes | Requires rebuilding and redeploying an image, slower to react | Can be patched directly on the box in seconds |
| Auditability & reproducibility | Image is a versioned artifact; environment is fully reproducible | True state only knowable by inspecting the live server |
| Pipeline & storage overhead | Needs an image build pipeline and artifact/image registry | Lower tooling overhead, no build pipeline required |

## Key Differences

- Immutable instances are never touched after launch; mutable servers are <strong class="kw">patched in place</strong>.
- Immutable infrastructure eliminates <strong class="kw">configuration drift</strong> by construction.
- Rolling back immutable infra just means redeploying a prior <strong class="kw">image version</strong>.
- Mutable infra depends on ongoing <strong class="kw">config management</strong> tooling to keep state converged.
- Immutable workflows require an <strong class="kw">image build pipeline</strong> and registry that mutable setups skip.

## When to Use Each

**Immutable Infrastructure**

- **Cloud-native microservices**: Autoscaled, stateless services fit naturally into a build-once-deploy-many-instances model.
- **Compliance-heavy environments**: A versioned image gives a reproducible, auditable artifact trail for every deployed change.
- **Frequent, automated releases**: CI/CD pipelines can produce and roll out new images with predictable, revertible deployments.

**Mutable Infrastructure**

- **Stateful legacy systems**: Servers with hard-to-migrate local state are cheaper to patch than to rebuild and replace.
- **Urgent production hotfix**: A direct in-place change resolves an incident faster than a full rebuild-and-redeploy cycle.
- **Bare-metal or hardware-bound hosts**: Physical or specialized hardware can't simply be discarded and replaced like a cloud instance.
