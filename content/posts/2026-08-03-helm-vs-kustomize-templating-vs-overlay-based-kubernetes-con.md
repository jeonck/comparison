---
title: "Helm vs Kustomize: Templating vs Overlay-Based Kubernetes Config"
date: 2026-08-03T05:20:13.193113+09:00
tags: ["kubernetes", "helm", "kustomize", "devops"]
---
## Overview

Helm packages Kubernetes manifests as parameterized <strong class="kw">charts</strong> rendered through a Go templating engine, then tracks each install as a versioned release. Kustomize takes plain manifests and applies declarative <strong class="kw">overlays</strong> that patch a base configuration per environment, with no templating language or release state at all.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">Helm</text><text x="480" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">Kustomize</text><rect x="40" y="55" width="240" height="60" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="80" text-anchor="middle" font-size="13" style="fill:var(--content)">Chart</text><text x="160" y="100" text-anchor="middle" font-size="11" style="fill:var(--secondary)">templates/*.yaml + values.yaml</text><line x1="160" y1="115" x2="160" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="40" y="150" width="240" height="45" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="178" text-anchor="middle" font-size="12" style="fill:var(--content)">helm template / install</text><line x1="160" y1="195" x2="160" y2="230" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="40" y="230" width="240" height="45" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="253" text-anchor="middle" font-size="12" style="fill:var(--content)">Rendered manifests</text><text x="160" y="268" text-anchor="middle" font-size="10" style="fill:var(--secondary)">tracked as a Release</text><line x1="160" y1="275" x2="160" y2="305" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="40" y="305" width="240" height="40" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="160" y="330" text-anchor="middle" font-size="12" style="fill:var(--content)">Cluster</text><rect x="400" y="55" width="110" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="455" y="78" text-anchor="middle" font-size="12" style="fill:var(--content)">base/</text><text x="455" y="95" text-anchor="middle" font-size="10" style="fill:var(--secondary)">plain manifests</text><rect x="520" y="55" width="110" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="575" y="78" text-anchor="middle" font-size="12" style="fill:var(--content)">overlays/prod</text><text x="575" y="95" text-anchor="middle" font-size="10" style="fill:var(--secondary)">patches</text><line x1="455" y1="115" x2="500" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="575" y1="115" x2="510" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="150" width="230" height="45" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="515" y="178" text-anchor="middle" font-size="12" style="fill:var(--content)">kustomize build</text><line x1="515" y1="195" x2="515" y2="230" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="230" width="230" height="45" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="515" y="253" text-anchor="middle" font-size="12" style="fill:var(--content)">Merged manifests</text><text x="515" y="268" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no state tracked</text><line x1="515" y1="275" x2="515" y2="305" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><rect x="400" y="305" width="230" height="40" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="515" y="330" text-anchor="middle" font-size="12" style="fill:var(--content)">Cluster</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Helm | Kustomize |
| --- | --- | --- |
| Configuration model | Go template engine that generates YAML text before it's parsed | Native Kubernetes objects patched via strategic merge or JSON patch |
| Input format | Chart with templates/, values.yaml, and Chart.yaml metadata | Plain, valid YAML manifests plus a kustomization.yaml |
| Parameterization | Placeholder values injected as text, so output can become invalid YAML if misused | Structured patches applied to already-valid objects, so output stays schema-correct |
| Environment customization | Layered values files (values-prod.yaml) merged into one chart | Overlay directories per environment referencing a shared base |
| Packaging & distribution | Versioned, shareable chart archives published to chart repositories or OCI registries | No packaging format; kustomization directories are just checked into git |
| Dependency management | Subcharts declared in Chart.yaml and pulled via helm dependency update | Bases and components composed by referencing other directories |
| Deployment execution | helm install/upgrade tracks a named release and its revision history | kustomize build pipes to kubectl apply with no release object created |
| Rollback & drift | helm rollback reverts to a stored prior release revision | No built-in rollback; relies on git revert or kubectl's own history |

## Key Differences

- Helm renders manifests through <strong class="kw">text templating</strong>, while Kustomize edits already-parsed objects via <strong class="kw">structural patches</strong>
- Helm tracks installs as stateful <strong class="kw">releases</strong> with revision history; Kustomize has <strong class="kw">no release state</strong> at all
- Helm charts are <strong class="kw">packaged and versioned</strong> for reuse; Kustomize configs are just plain manifests in git
- Kustomize is built into <strong class="kw">kubectl</strong> directly, while Helm requires installing a separate CLI/tool
- Many teams combine both: a Helm chart as the base, customized per environment with Kustomize's <strong class="kw">overlay patches</strong>

## When to Use Each

**Helm**

- **Distributing reusable software**: Helm charts let third parties install your app with one command and sensible defaults via values.yaml.
- **Complex conditional logic**: Templating supports loops, conditionals, and helper functions that pure YAML patching can't express.
- **Release history and rollback**: Helm's revision tracking lets you roll back a bad upgrade to an exact prior state.
- **Public chart ecosystem**: Artifact Hub and vendor-maintained charts give you production-ready configs to start from.

**Kustomize**

- **Environment-specific overlays**: Kustomize cleanly separates a shared base from per-environment patches without templating logic.
- **GitOps-native workflows**: Plain YAML with no rendering step is easy for tools like Argo CD or Flux to diff and reconcile.
- **No extra tooling**: Kustomize ships inside kubectl, so there's nothing extra to install or version-match.
- **Patching third-party manifests**: You can customize vendor-supplied YAML you don't control without forking it into a chart.
