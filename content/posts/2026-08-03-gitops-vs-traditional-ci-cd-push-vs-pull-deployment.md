---
title: "GitOps vs Traditional CI/CD: Push vs Pull Deployment"
date: 2026-08-03T05:17:05.632017+09:00
tags: ["gitops", "ci-cd", "kubernetes", "devops"]
---
## Overview

Both aim to automate software delivery, but they differ in who initiates the deployment and where the source of truth lives. Traditional <strong class="kw">CI/CD</strong> pushes changes into infrastructure from an external pipeline, while <strong class="kw">GitOps</strong> has an in-cluster agent continuously pull and reconcile state against a Git repository. The distinction matters most for security posture, drift handling, and auditability in Kubernetes-native environments.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrowA" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-a)"/>
    </marker>
    <marker id="arrowB" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-b)"/>
    </marker>
  </defs>
  <line x1="320" y1="10" x2="320" y2="350" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/>
  <text x="170" y="26" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Traditional CI/CD</text>
  <text x="170" y="44" text-anchor="middle" style="fill:var(--secondary)" font-size="11">push-based</text>
  <text x="490" y="26" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">GitOps</text>
  <text x="490" y="44" text-anchor="middle" style="fill:var(--secondary)" font-size="11">pull-based</text>
  <rect x="60" y="60" width="160" height="45" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="140" y="87" text-anchor="middle" style="fill:var(--content)" font-size="13">Git Repo</text>
  <line x1="140" y1="105" x2="140" y2="158" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/>
  <text x="150" y="135" style="fill:var(--secondary)" font-size="10">merge trigger</text>
  <rect x="60" y="160" width="160" height="45" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="140" y="187" text-anchor="middle" style="fill:var(--content)" font-size="13">CI/CD Pipeline</text>
  <line x1="140" y1="205" x2="140" y2="258" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/>
  <text x="150" y="235" style="fill:var(--secondary)" font-size="10">kubectl apply</text>
  <text x="150" y="248" style="fill:var(--secondary)" font-size="10">(holds cluster creds)</text>
  <rect x="60" y="260" width="160" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="140" y="291" text-anchor="middle" style="fill:var(--content)" font-size="13">Production</text>
  <text x="140" y="306" text-anchor="middle" style="fill:var(--content)" font-size="13">Cluster</text>
  <text x="140" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="10">external system pushes with cluster creds</text>
  <rect x="420" y="60" width="160" height="45" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="500" y="87" text-anchor="middle" style="fill:var(--content)" font-size="13">Git Repo</text>
  <rect x="420" y="260" width="160" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="500" y="284" text-anchor="middle" style="fill:var(--content)" font-size="13">Production Cluster</text>
  <text x="500" y="300" text-anchor="middle" style="fill:var(--content)" font-size="11">(GitOps agent)</text>
  <path d="M 460,260 C 600,225 600,140 465,107" fill="none" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/>
  <text x="605" y="185" text-anchor="middle" style="fill:var(--secondary)" font-size="10">pulls &amp;</text>
  <text x="605" y="198" text-anchor="middle" style="fill:var(--secondary)" font-size="10">diffs state</text>
  <text x="500" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="10">agent auto-reconciles drift, no external creds</text>
</svg>
</div>

## Comparison Table

| Aspect | Traditional CI/CD | GitOps |
| --- | --- | --- |
| Deployment trigger | Pipeline job runs on merge/tag and executes a deploy step | In-cluster agent continuously polls or watches the Git repo for changes |
| Source of truth | Pipeline scripts and job history define what was deployed | Git repository is the sole declarative source of desired state |
| Cluster access model | CI server holds cluster credentials and pushes from outside the network | Agent runs inside the cluster; no external system needs cluster credentials |
| Drift detection | None built-in; manual kubectl edits go unnoticed until the next run | Agent continuously compares live state to Git and flags or corrects drift |
| Rollback | Re-run the pipeline against a previous artifact or commit | git revert triggers an automatic re-sync to the prior state |
| Audit trail | Split across CI logs, deploy scripts, and any manual changes | Single, complete history captured in Git commit log |
| Multi-cluster scaling | Pipeline needs explicit logic and credentials per target environment | Each cluster runs its own agent watching the same or a branched repo |

## Key Differences

- CI/CD is <strong class="kw">push-based</strong> from an external system; GitOps is <strong class="kw">pull-based</strong> from inside the cluster
- GitOps treats the Git repo as the exclusive <strong class="kw">source of truth</strong>; CI/CD's truth lives in pipeline state
- CI/CD requires the pipeline to hold <strong class="kw">cluster credentials</strong>; GitOps keeps them inside the cluster boundary
- GitOps performs automatic <strong class="kw">drift correction</strong>; CI/CD has no ongoing reconciliation
- Rollback in GitOps is a simple <strong class="kw">git revert</strong> instead of re-running a pipeline job

## When to Use Each

**Traditional CI/CD**

- **Non-Kubernetes targets**: Deploying to VMs, serverless functions, or platforms without a reconciliation-friendly API fits a push pipeline better.
- **Complex multi-stage pipelines**: Elaborate build, test, and approval gates before deployment are easier to express as explicit pipeline steps.
- **Existing CI investment**: Teams with mature Jenkins/GitHub Actions pipelines can avoid the cost of introducing a new operational model.

**GitOps**

- **Kubernetes-native platforms**: Tools like Argo CD or Flux integrate natively with the Kubernetes API to reconcile manifests continuously.
- **Self-healing infrastructure**: Automatic drift correction matters when manual or out-of-band cluster changes need to be caught and reverted.
- **Large multi-cluster fleets**: Each cluster running its own pull agent scales more predictably than a pipeline managing credentials per environment.
- **Strict compliance auditing**: Regulated environments benefit from Git's commit history serving as a single, tamper-evident change log.
