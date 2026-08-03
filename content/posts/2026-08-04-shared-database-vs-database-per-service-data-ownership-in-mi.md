---
title: "Shared Database vs Database per Service: Data Ownership in Microservices"
date: 2026-08-04T05:14:24.958456+09:00
tags: ["microservices", "database-design", "system-architecture", "data-ownership"]
---
## Overview

A <strong class="kw">shared database</strong> lets multiple services read and write the same tables through one common schema, while <strong class="kw">database per service</strong> gives each service its own private data store that only it can touch directly. The choice determines how tightly services are coupled at the data layer, how independently teams can deploy, and how much work cross-service queries and transactions require.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="160" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Shared Database</text><text x="480" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Database per Service</text><rect x="30" y="55" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="65" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service A</text><rect x="125" y="55" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service B</text><rect x="220" y="55" width="70" height="35" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="255" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service C</text><line x1="65" y1="90" x2="140" y2="168" style="stroke:var(--secondary)" stroke-width="1.5"/><line x1="160" y1="90" x2="160" y2="168" style="stroke:var(--secondary)" stroke-width="1.5"/><line x1="255" y1="90" x2="180" y2="168" style="stroke:var(--secondary)" stroke-width="1.5"/><rect x="115" y="170" width="90" height="55" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><ellipse cx="160" cy="170" rx="45" ry="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><ellipse cx="160" cy="225" rx="45" ry="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="204" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">Shared DB</text><text x="160" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">one schema, every service reads/writes it directly</text><rect x="355" y="48" width="100" height="180" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><rect x="450" y="48" width="100" height="180" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><rect x="545" y="48" width="90" height="180" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><rect x="370" y="55" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="405" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service X</text><rect x="465" y="55" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service Y</text><rect x="560" y="55" width="70" height="35" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="595" y="77" text-anchor="middle" style="fill:var(--content)" font-size="11">Service Z</text><line x1="405" y1="90" x2="405" y2="150" style="stroke:var(--secondary)" stroke-width="1.5"/><line x1="500" y1="90" x2="500" y2="150" style="stroke:var(--secondary)" stroke-width="1.5"/><line x1="595" y1="90" x2="595" y2="150" style="stroke:var(--secondary)" stroke-width="1.5"/><rect x="375" y="150" width="60" height="42" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="405" cy="150" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="405" cy="192" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="405" y="175" text-anchor="middle" style="fill:var(--content)" font-size="10">DB X</text><rect x="470" y="150" width="60" height="42" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="500" cy="150" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="500" cy="192" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="175" text-anchor="middle" style="fill:var(--content)" font-size="10">DB Y</text><rect x="565" y="150" width="60" height="42" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="595" cy="150" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><ellipse cx="595" cy="192" rx="30" ry="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="595" y="175" text-anchor="middle" style="fill:var(--content)" font-size="10">DB Z</text><text x="480" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">each service owns a private schema, accessed only via its API</text></svg>
</div>

## Comparison Table

| Aspect | Shared Database | Database per Service |
| --- | --- | --- |
| Data ownership | No single owner — all services see and can modify the same tables | Each service exclusively owns its schema; no one else can touch it directly |
| Access path | Services query the database directly, often with raw SQL against shared tables | Other services only get data through the owning service's API or published events |
| Cross-service transactions | Native ACID transactions and joins span all the data in one commit | No shared transaction; consistency across services needs sagas or eventual consistency |
| Cross-service queries | Simple SQL joins pull data from any table in one query | Requires API composition, data replication, or a separate CQRS read model |
| Schema changes | A column or table change can silently break unrelated services | Schema changes are internal; only the public API contract must stay stable |
| Technology choice | All services are locked into one database engine and schema | Each service can pick the storage engine that fits its data (polyglot persistence) |
| Failure isolation | A database outage or lock contention affects every service at once | An outage in one service's database doesn't directly take down the others |
| Operational overhead | One database to provision, back up, and tune | N databases to provision, monitor, back up, and scale independently |

## Key Differences

- Shared database gives every service direct access to the same tables, so a change in one place can silently break another service's queries — a form of <strong class="kw">tight coupling</strong>.
- Database per service forces all cross-service data access through an API, giving each service true <strong class="kw">encapsulation</strong> of its data.
- Cross-entity consistency is a native <strong class="kw">ACID transaction</strong> in a shared database, but needs a <strong class="kw">saga pattern</strong> or eventual consistency once data is split per service.
- Reporting and ad-hoc joins are trivial with a shared database's SQL, while database per service usually needs a separate <strong class="kw">CQRS read model</strong> to answer cross-service queries.
- Database per service allows <strong class="kw">polyglot persistence</strong> — each service picks its own database engine — whereas shared database locks every service to one engine and schema.

## When to Use Each

**Shared Database**

- **Early-stage monolith**: When the system is small and still finding its shape, a shared database avoids premature distributed-systems complexity.
- **Heavy cross-entity reporting**: When the main need is ad-hoc SQL joins and reports across all business data, one schema is far simpler than composing across services.
- **Single team, single deploy cadence**: When one team owns the whole system, they can coordinate schema changes directly without cross-team negotiation.

**Database per Service**

- **Independently deployable services**: When teams need to ship and scale their service without coordinating schema changes with other teams.
- **Polyglot data needs**: When different services fit different storage models, such as graph, document, relational, or time-series data.
- **Fault isolation required**: When one service's data outage or overload must not cascade into unrelated services.
- **Clear bounded contexts**: When domain boundaries from DDD map cleanly onto services that should each own their own data.
