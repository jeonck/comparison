---
title: "Layered Architecture vs Hexagonal Architecture: Structuring Business Logic"
date: 2026-08-04T05:08:09.598167+09:00
tags: ["software-architecture", "layered-architecture", "hexagonal-architecture", "design-patterns"]
---
## Overview

Layered Architecture stacks an application into horizontal <strong class="kw">layers</strong>, where each layer may only call the one directly beneath it. Hexagonal Architecture instead puts business logic at the center and connects it to the outside world through <strong class="kw">ports and adapters</strong>, so no top-to-bottom hierarchy exists at all. The distinction matters because it determines how easily you can swap infrastructure, test in isolation, and keep framework details from leaking into core logic.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="15" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="25" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Layered</text><text x="480" y="25" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Hexagonal</text><rect x="40" y="45" width="240" height="42" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="70" text-anchor="middle" style="fill:var(--content)" font-size="13">Presentation</text><rect x="40" y="105" width="240" height="42" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="130" text-anchor="middle" style="fill:var(--content)" font-size="13">Business Logic</text><rect x="40" y="165" width="240" height="42" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="190" text-anchor="middle" style="fill:var(--content)" font-size="13">Persistence</text><rect x="40" y="225" width="240" height="42" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="250" text-anchor="middle" style="fill:var(--content)" font-size="13">Database</text><line x1="160" y1="87" x2="160" y2="103" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,105 155,97 165,97" style="fill:var(--compare-a)"/><line x1="160" y1="147" x2="160" y2="163" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,165 155,157 165,157" style="fill:var(--compare-a)"/><line x1="160" y1="207" x2="160" y2="223" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="160,225 155,217 165,217" style="fill:var(--compare-a)"/><text x="160" y="290" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Strict top-down dependency</text><polygon points="480,95 527.6,122.5 527.6,177.5 480,205 432.4,177.5 432.4,122.5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="145" text-anchor="middle" style="fill:var(--content)" font-size="12">Domain</text><text x="480" y="160" text-anchor="middle" style="fill:var(--content)" font-size="12">Core</text><rect x="440" y="30" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="49" text-anchor="middle" style="fill:var(--content)" font-size="10">REST Adapter</text><line x1="480" y1="95" x2="480" y2="60" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="540" y="107" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="580" y="126" text-anchor="middle" style="fill:var(--content)" font-size="10">DB Adapter</text><line x1="527.6" y1="122.5" x2="540" y2="122" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="240" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="259" text-anchor="middle" style="fill:var(--content)" font-size="10">CLI Adapter</text><line x1="480" y1="205" x2="480" y2="240" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="340" y="107" width="80" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="380" y="126" text-anchor="middle" style="fill:var(--content)" font-size="10">Test Mock</text><line x1="432.4" y1="122.5" x2="420" y2="122" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Adapters depend inward on core</text></svg>
</div>

## Comparison Table

| Aspect | Layered Architecture | Hexagonal Architecture |
| --- | --- | --- |
| Structural organization | Horizontal layers (presentation, business, persistence); each layer only calls the one below it | Concentric layout with a domain core surrounded by ports and adapters; no top/bottom hierarchy |
| Dependency direction | Strictly downward: layer N depends only on layer N-1 | Always inward: adapters depend on the core, the core depends on nothing external |
| Entry and exit points | Requests enter at the presentation layer and exit at the database layer | Requests enter through any driving port and exit through any driven port |
| Business logic isolation | Business layer is often still coupled to persistence details that leak upward | Domain core is fully isolated behind port interfaces and unaware of any adapter |
| Swapping infrastructure | Replacing a database or UI framework often requires touching multiple layers | Swapping an adapter (e.g., DB for an in-memory store) requires no changes to the core |
| Testing strategy | Unit tests typically mock the layer directly beneath the one under test | Core logic is tested directly through ports; adapters are tested separately |
| Common pitfall | "Layered lasagna": persistence concerns bleed upward into the business layer | Over-engineering ports and adapters for simple CRUD apps adds needless indirection |
| Best fit | Traditional CRUD apps with a straightforward request/response flow | Domain-heavy apps needing multiple interfaces (REST, CLI, events) or high testability |

## Key Differences

- Layered enforces dependency in one direction (top-down) while hexagonal enforces dependency inward toward the <strong class="kw">domain core</strong>.
- Hexagonal defines explicit <strong class="kw">ports</strong> as boundaries; layered relies on looser, implicit layer contracts.
- Layered architecture is simpler to learn but prone to <strong class="kw">leaky abstractions</strong> between layers.
- Hexagonal architecture makes it trivial to swap <strong class="kw">infrastructure</strong> without touching business logic.

## When to Use Each

**Layered Architecture**

- **Simple CRUD Services**: Straightforward request/response apps benefit from the familiar top-down structure without added indirection.
- **Small Teams and Onboarding**: New developers grasp layered architecture quickly since it maps directly to the request lifecycle.
- **Rapid Prototyping**: Fewer abstractions mean less boilerplate when speed matters more than long-term flexibility.

**Hexagonal Architecture**

- **Multiple Delivery Mechanisms**: Apps exposed via REST, CLI, and message queues can share one core without duplicating business logic.
- **Heavy Domain Logic**: Complex business rules stay isolated and testable independent of any framework or database.
- **Frequent Infrastructure Changes**: Swapping databases, message brokers, or third-party APIs only requires writing a new adapter.
