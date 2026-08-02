---
title: "Prometheus vs Grafana: Metrics Collection vs Visualization"
date: 2026-08-03T05:30:34.835710+09:00
tags: ["prometheus", "grafana", "monitoring", "observability"]
---
## Overview

Prometheus is a <strong class="kw">time-series database</strong> that scrapes and stores metrics with its own query language and alerting engine, while Grafana is a <strong class="kw">visualization layer</strong> that queries Prometheus (and many other backends) to render dashboards. They're often deployed together, not as alternatives, because each solves a different half of the observability pipeline.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="150" y="30" text-anchor="middle" font-size="18" font-weight="600" style="fill:var(--primary)">Prometheus</text><text x="520" y="30" text-anchor="middle" font-size="18" font-weight="600" style="fill:var(--primary)">Grafana</text><rect x="40" y="55" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="79" text-anchor="middle" font-size="12" style="fill:var(--content)">Exporters / Targets</text><line x1="110" y1="95" x2="110" y2="120" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="40" y="120" width="140" height="75" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="143" text-anchor="middle" font-size="12" style="fill:var(--content)">Scrape Engine</text><line x1="55" y1="153" x2="165" y2="153" style="stroke:var(--compare-a)" stroke-width="1"/><text x="110" y="172" text-anchor="middle" font-size="12" style="fill:var(--content)">TSDB (storage)</text><text x="110" y="186" text-anchor="middle" font-size="10" style="fill:var(--secondary)">PromQL</text><line x1="110" y1="195" x2="110" y2="220" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><rect x="40" y="220" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="110" y="244" text-anchor="middle" font-size="12" style="fill:var(--content)">Alertmanager</text><rect x="220" y="137" width="110" height="58" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="275" y="160" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Other Data</text><text x="275" y="174" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Sources</text><text x="275" y="188" text-anchor="middle" font-size="9" style="fill:var(--secondary)">(MySQL, Loki...)</text><rect x="420" y="120" width="180" height="140" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="142" text-anchor="middle" font-size="13" style="fill:var(--content)">Dashboards</text><rect x="435" y="155" width="70" height="40" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><polyline points="440,185 452,168 462,178 475,162 500,172" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5"/><rect x="515" y="155" width="70" height="40" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><rect x="525" y="180" width="8" height="12" style="fill:var(--compare-b)"/><rect x="538" y="172" width="8" height="20" style="fill:var(--compare-b)"/><rect x="551" y="165" width="8" height="27" style="fill:var(--compare-b)"/><rect x="564" y="176" width="8" height="16" style="fill:var(--compare-b)"/><rect x="435" y="205" width="150" height="40" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><text x="510" y="228" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Alerting + Notifications</text><line x1="180" y1="157" x2="415" y2="180" style="stroke:var(--content)" stroke-width="1.2" marker-end="url(#arrowC)"/><text x="290" y="145" text-anchor="middle" font-size="9" style="fill:var(--secondary)">PromQL query</text><line x1="330" y1="166" x2="415" y2="175" style="stroke:var(--content)" stroke-width="1.2" marker-end="url(#arrowC)"/><text x="110" y="290" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Collects &amp; Stores Metrics</text><text x="510" y="290" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Queries &amp; Visualizes</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowC" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--content)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | Prometheus | Grafana |
| --- | --- | --- |
| Primary purpose | Collect, store, and query time-series metrics | Visualize and correlate data from one or more sources |
| Data collection | Pull-based scraping of HTTP /metrics endpoints on a schedule | None — has no collector, relies entirely on configured data sources |
| Data storage | Built-in on-disk time-series database (TSDB) | Stateless; stores only dashboard/config metadata, not metric data |
| Query language | PromQL, native to its own TSDB | Delegates to whatever query language the connected data source uses |
| Supported data sources | Only its own TSDB (federation aside) | 150+ backends including Prometheus, Loki, InfluxDB, Elasticsearch, SQL |
| Alerting | Native alerting rules evaluated in-server, routed via Alertmanager | Unified alerting engine that layers on top of any connected data source |
| Visualization | Minimal built-in expression browser, no dashboarding | Rich, customizable panels, graphs, and shareable dashboards |
| Typical deployment | Runs per cluster/environment alongside exporters | Central server pointing at multiple backends across teams |

## Key Differences

- Prometheus owns the <strong class="kw">TSDB</strong>; Grafana holds no metric data of its own.
- Prometheus <strong class="kw">scrapes</strong> targets directly; Grafana only issues read queries.
- Prometheus alerting runs server-side via <strong class="kw">Alertmanager</strong>; Grafana's alerting spans any connected source.
- Grafana supports <strong class="kw">multi-source</strong> dashboards, unlike Prometheus's single-source expression browser.
- Neither replaces the other — most stacks run <strong class="kw">both together</strong>.

## When to Use Each

**Prometheus**

- **Kubernetes metric scraping**: Prometheus's pull model and service discovery are purpose-built for scraping dynamic pods and services.
- **Threshold-based alerting**: Its native alerting rules and Alertmanager routing don't require any external visualization layer.
- **Standalone metrics storage**: When you just need a reliable, queryable TSDB for your own microservices without a UI requirement.

**Grafana**

- **Cross-team observability dashboards**: Grafana can pull Prometheus metrics, Loki logs, and traces into one shared view.
- **Mixed-backend visualization**: Ideal when metrics live in multiple systems (Prometheus, InfluxDB, SQL) that need a single pane of glass.
- **Executive/team-facing reporting**: Its polished panels and sharing features suit stakeholders who never touch PromQL directly.
