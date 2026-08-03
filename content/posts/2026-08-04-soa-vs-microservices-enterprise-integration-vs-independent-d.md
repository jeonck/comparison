---
title: "SOA vs Microservices: Enterprise Integration vs Independent Deployability"
date: 2026-08-04T05:17:08.737070+09:00
tags: ["soa", "microservices", "software-architecture", "distributed-systems"]
---
## Overview

SOA and microservices are both approaches to composing systems from independently callable services, but they differ sharply in scope and philosophy. <strong class="kw">SOA</strong> centralizes communication and governance through an enterprise service bus to integrate large, often legacy systems, while <strong class="kw">microservices</strong> decentralize communication, data, and deployment into small, independently shippable units. The distinction matters because it drives very different tooling, team structures, and failure characteristics.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="150" y="30" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">SOA</text><text x="480" y="30" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">Microservices</text><line x1="320" y1="50" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><circle cx="70" cy="90" r="20" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="94" text-anchor="middle" font-size="11" style="fill:var(--content)">S1</text><circle cx="230" cy="90" r="20" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="230" y="94" text-anchor="middle" font-size="11" style="fill:var(--content)">S2</text><circle cx="70" cy="230" r="20" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="70" y="234" text-anchor="middle" font-size="11" style="fill:var(--content)">S3</text><circle cx="230" cy="230" r="20" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="230" y="234" text-anchor="middle" font-size="11" style="fill:var(--content)">S4</text><line x1="70" y1="90" x2="150" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="230" y1="90" x2="150" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="70" y1="230" x2="150" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="230" y1="230" x2="150" y2="160" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="95" y="140" width="110" height="40" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="164" text-anchor="middle" font-size="12" style="fill:var(--content)">ESB</text><line x1="150" y1="180" x2="150" y2="290" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="95" y="290" width="110" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="150" y="313" text-anchor="middle" font-size="11" style="fill:var(--content)">Shared DB</text><text x="150" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Central bus + shared data</text><circle cx="400" cy="90" r="20" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="94" text-anchor="middle" font-size="11" style="fill:var(--content)">M1</text><circle cx="560" cy="90" r="20" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="94" text-anchor="middle" font-size="11" style="fill:var(--content)">M2</text><circle cx="400" cy="230" r="20" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="234" text-anchor="middle" font-size="11" style="fill:var(--content)">M3</text><circle cx="560" cy="230" r="20" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="560" y="234" text-anchor="middle" font-size="11" style="fill:var(--content)">M4</text><line x1="400" y1="90" x2="560" y2="90" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="400" y1="90" x2="400" y2="230" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="560" y1="90" x2="560" y2="230" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="400" y1="230" x2="560" y2="230" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="330" y="55" width="40" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="350" y="68" text-anchor="middle" font-size="9" style="fill:var(--content)">db</text><rect x="590" y="55" width="40" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="610" y="68" text-anchor="middle" font-size="9" style="fill:var(--content)">db</text><rect x="330" y="252" width="40" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="350" y="265" text-anchor="middle" font-size="9" style="fill:var(--content)">db</text><rect x="590" y="252" width="40" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.2"/><text x="610" y="265" text-anchor="middle" font-size="9" style="fill:var(--content)">db</text><text x="480" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Direct calls, DB per service</text></svg>
</div>

## Comparison Table

| Aspect | SOA | Microservices |
| --- | --- | --- |
| Communication mechanism | Services talk through a central Enterprise Service Bus using protocols like SOAP/WS-* | Services talk directly via lightweight REST/gRPC calls or message brokers, no mandatory central bus |
| Service granularity | Coarse-grained, often modeling whole business processes | Fine-grained, each service owns a single business capability |
| Data ownership | Services frequently share a common database or canonical data model | Each service owns and manages its own private database |
| Deployment unit | Services often share application servers or deployment packages | Each service is deployed and scaled independently, typically in its own container |
| Technology stack | Standardized enterprise-wide on common platforms and protocols | Polyglot — each team chooses its own language, framework, and datastore |
| Fault isolation | The ESB is a potential single point of failure affecting many services | Failures are isolated to individual services, limiting blast radius |
| Governance & teams | Centralized architecture review board and IT governance | Decentralized ownership by small, autonomous teams per service |
| Typical origin | Emerged from large-scale enterprise integration needs in the 2000s | Emerged from cloud-native, DevOps-driven practices in the 2010s |

## Key Differences

- SOA centralizes routing and transformation logic in an <strong class="kw">ESB</strong>, while microservices push that logic into the endpoints themselves.
- Microservices mandate <strong class="kw">database per service</strong>, whereas SOA services commonly share a data layer.
- SOA favors <strong class="kw">reusable coarse-grained services</strong> across the enterprise; microservices favor small, single-purpose services.
- Microservices deploy and scale via <strong class="kw">independent containers</strong>, while SOA services often share application servers.
- SOA relies on heavyweight standards like <strong class="kw">SOAP/WS-*</strong>; microservices typically use lightweight REST or gRPC.

## When to Use Each

**SOA**

- **Enterprise System Integration**: Legacy mainframes, ERPs, and CRMs need to interoperate under a common governance model.
- **Reusable Business Services**: You want centrally reusable services shared across many applications, like a shared billing or customer-lookup service.
- **Regulated, Centralized IT**: Strong central governance and security policy enforcement is required across all services, as in finance or government.

**Microservices**

- **Independently Scaling Components**: Specific parts of an app, like checkout versus catalog browsing, need to scale and deploy on their own schedule.
- **Fast-Moving Product Teams**: Small autonomous teams want to ship features without coordinating a shared release train.
- **Cloud-Native Greenfield Apps**: You're building a new application targeting containers, Kubernetes, and CI/CD pipelines from day one.
