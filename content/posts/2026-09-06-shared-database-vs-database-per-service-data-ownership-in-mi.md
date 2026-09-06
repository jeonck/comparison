---
title: "Shared Database vs Database Per Service: Data Ownership in Microservices"
date: 2026-09-06T09:52:35.195620+09:00
tags: ["microservices", "database-design", "system-architecture", "distributed-systems"]
---
## Overview

This compares two data architecture patterns for microservices: a <strong class="kw">shared database</strong> where multiple services read and write the same schema, versus <strong class="kw">database per service</strong> where each service owns an isolated data store. The choice determines how tightly services are coupled, how transactions and queries span service boundaries, and how independently teams can deploy and scale.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Shared Database</text><text x="480" y="30" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--primary)">Database Per Service</text><line x1="320" y1="50" x2="320" y2="330" stroke-width="1" stroke-dasharray="4,4" style="stroke:var(--border)"/><rect x="30" y="70" width="110" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="85" y="97" text-anchor="middle" font-size="13" style="fill:var(--content)">Service A</text><rect x="30" y="155" width="110" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="85" y="182" text-anchor="middle" font-size="13" style="fill:var(--content)">Service B</text><rect x="30" y="240" width="110" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="85" y="267" text-anchor="middle" font-size="13" style="fill:var(--content)">Service C</text><rect x="210" y="110" width="90" height="150" rx="8" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><text x="255" y="180" text-anchor="middle" font-size="13" style="fill:var(--content)">Shared</text><text x="255" y="198" text-anchor="middle" font-size="13" style="fill:var(--content)">DB</text><line x1="140" y1="92" x2="210" y2="145" stroke-width="1.5" style="stroke:var(--compare-a)"/><line x1="140" y1="177" x2="210" y2="185" stroke-width="1.5" style="stroke:var(--compare-a)"/><line x1="140" y1="262" x2="210" y2="225" stroke-width="1.5" style="stroke:var(--compare-a)"/><rect x="350" y="70" width="100" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="400" y="97" text-anchor="middle" font-size="13" style="fill:var(--content)">Service A</text><rect x="480" y="70" width="90" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="525" y="97" text-anchor="middle" font-size="13" style="fill:var(--content)">DB A</text><line x1="450" y1="92" x2="480" y2="92" stroke-width="1.5" style="stroke:var(--compare-b)"/><rect x="350" y="155" width="100" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="400" y="182" text-anchor="middle" font-size="13" style="fill:var(--content)">Service B</text><rect x="480" y="155" width="90" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="525" y="182" text-anchor="middle" font-size="13" style="fill:var(--content)">DB B</text><line x1="450" y1="177" x2="480" y2="177" stroke-width="1.5" style="stroke:var(--compare-b)"/><rect x="350" y="240" width="100" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="400" y="267" text-anchor="middle" font-size="13" style="fill:var(--content)">Service C</text><rect x="480" y="240" width="90" height="44" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><text x="525" y="267" text-anchor="middle" font-size="13" style="fill:var(--content)">DB C</text><line x1="450" y1="262" x2="480" y2="262" stroke-width="1.5" style="stroke:var(--compare-b)"/><text x="160" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Single point of coupling &amp; contention</text><text x="480" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Isolated data, independent scaling</text></svg>
</div>

## Comparison Table

| Aspect | Shared Database | Database Per Service |
| --- | --- | --- |
| Schema ownership | One schema shared and often co-owned by multiple teams | Each service exclusively owns and evolves its own schema |
| Write path | Any service can write directly to shared tables | Writes go only through the owning service's API |
| Cross-service queries | Simple SQL joins across tables in one database | Requires API calls, data replication, or an aggregation layer |
| Distributed transactions | Native ACID transactions across affected tables | Needs sagas or eventual consistency to span services |
| Schema migrations | Any change risks breaking other services using the table | Migrations are local and safe to run independently |
| Independent scaling | Database becomes a shared bottleneck under load | Each store can be scaled or tuned to its own service's needs |
| Technology choice | All services locked into one database engine | Each service can pick the best-fit database technology |
| Failure isolation | A database outage or lock contention affects every service | An outage is contained to the owning service's data |

## Key Differences

- Shared database allows cheap <strong class="kw">cross-table joins</strong> but couples every consuming service to one schema
- Database per service enforces <strong class="kw">service autonomy</strong> at the cost of needing sagas for cross-service transactions
- Schema changes in a shared database require coordinating <strong class="kw">multiple teams</strong>, while per-service schemas change independently
- A shared database creates a single <strong class="kw">failure domain</strong>; per-service databases contain outages to one service
- Polyglot persistence — choosing different database engines per need — is only possible with <strong class="kw">database per service</strong>

## When to Use Each

**Shared Database**

- **Early-stage monolith migration**: A shared database lets teams split code into services without immediately solving distributed data problems.
- **Heavy reporting and ad-hoc joins**: Analysts and BI tools benefit from querying all data in one place with plain SQL joins.
- **Small, tightly coordinated team**: When one team owns all services, schema coordination overhead is minimal.

**Database Per Service**

- **Independently deployable microservices**: Each team can release schema and code changes without cross-team approval or downtime risk to others.
- **Domain-driven service boundaries**: Data isolation reinforces bounded contexts and prevents hidden coupling through shared tables.
- **Mixed workload requirements**: Services needing different database types (graph, document, relational) can each pick the right engine.
- **High-scale, high-availability systems**: Isolating data stores prevents one service's load or outage from cascading to others.
