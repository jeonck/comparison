---
title: "Lambda Architecture vs Kappa Architecture: Batch-Plus-Speed vs Single-Stream Pipelines"
date: 2026-08-04T05:21:28.497015+09:00
tags: ["data-engineering", "stream-processing", "big-data-architecture", "event-driven"]
---
## Overview

Lambda Architecture and Kappa Architecture are two patterns for building big-data pipelines that need both real-time and historical results. <strong class="kw">Lambda</strong> runs parallel batch and speed layers that get merged at query time, while <strong class="kw">Kappa</strong> pushes everything through a single, replayable stream pipeline. The choice determines how much duplicate logic you maintain and how reprocessing actually works.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4,4"/><text x="160" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Lambda Architecture</text><text x="480" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Kappa Architecture</text><rect x="20" y="160" width="70" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="55" y="182" text-anchor="middle" font-size="10" style="fill:var(--content)">Data Source</text><rect x="130" y="70" width="130" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="92" text-anchor="middle" font-size="11" style="fill:var(--content)">Batch Layer</text><text x="195" y="106" text-anchor="middle" font-size="9" style="fill:var(--secondary)">full recompute</text><rect x="130" y="240" width="130" height="50" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="195" y="262" text-anchor="middle" font-size="11" style="fill:var(--content)">Speed Layer</text><text x="195" y="276" text-anchor="middle" font-size="9" style="fill:var(--secondary)">recent, approximate</text><rect x="266" y="158" width="46" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="289" y="174" text-anchor="middle" font-size="8" style="fill:var(--content)">Serving</text><text x="289" y="186" text-anchor="middle" font-size="8" style="fill:var(--content)">Layer</text><line x1="92" y1="168" x2="128" y2="98" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="92" y1="188" x2="128" y2="258" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="261" y1="110" x2="267" y2="172" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="261" y1="250" x2="267" y2="186" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><line x1="313" y1="178" x2="317" y2="178" style="stroke:var(--compare-a)" stroke-width="1.5" marker-end="url(#arrowA)"/><text x="195" y="322" text-anchor="middle" font-size="9" style="fill:var(--secondary)">reprocess = rerun batch job</text><rect x="340" y="160" width="70" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="375" y="182" text-anchor="middle" font-size="10" style="fill:var(--content)">Event Log</text><rect x="450" y="130" width="150" height="90" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="525" y="168" text-anchor="middle" font-size="11" style="fill:var(--content)">Stream Processing</text><text x="525" y="182" text-anchor="middle" font-size="11" style="fill:var(--content)">Layer</text><text x="525" y="200" text-anchor="middle" font-size="9" style="fill:var(--secondary)">single codebase</text><line x1="411" y1="178" x2="448" y2="175" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><line x1="601" y1="175" x2="628" y2="175" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrowB)"/><path d="M460,220 C 420,290 380,290 358,198" fill="none" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="4,3" marker-end="url(#arrowB)"/><text x="470" y="322" text-anchor="middle" font-size="9" style="fill:var(--secondary)">reprocess = replay the log</text><text x="316" y="195" text-anchor="middle" font-size="9" style="fill:var(--secondary)">query</text><text x="614" y="195" text-anchor="middle" font-size="9" style="fill:var(--secondary)">query</text></svg>
</div>

## Comparison Table

| Aspect | Lambda Architecture | Kappa Architecture |
| --- | --- | --- |
| Ingestion path | Raw events are forked to both a batch store and a stream processor at once | Raw events are written once to an immutable, replayable log (e.g. Kafka) |
| Processing model | Two independent codebases — a batch job and a stream job — implement the same logic twice | One stream-processing codebase handles both real-time and historical computation |
| Historical reprocessing | The batch layer periodically recomputes results over the entire raw dataset | Reprocessing means replaying the log from an earlier offset through the same stream job |
| State and storage | Separate batch views and speed views are maintained independently, often in different stores | A single serving store is continuously updated by the stream processor |
| Result merging | The query layer merges or reconciles batch and speed views at read time | No merge step — the stream processor's output is the only view |
| Consistency behavior | Speed layer results are approximate until the batch layer overwrites them, so the two can disagree temporarily | One computation path avoids batch/speed drift, but correctness depends entirely on the stream engine's guarantees |
| Operational overhead | Higher — two parallel pipelines to build, deploy, and monitor, with duplicated logic | Lower pipeline count, but requires a log system with long enough retention to support full replays |
| Best-fit scenario | Batch and speed logic genuinely differ, or the org already has mature batch infrastructure | Team wants one canonical pipeline and has a stream engine that can absorb both live and replay traffic |

## Key Differences

- Lambda splits ingestion across a <strong class="kw">batch layer</strong> and a speed layer, while Kappa sends everything through one stream.
- Reprocessing history in Lambda means rerunning a full batch job; in Kappa it means <strong class="kw">replaying the log</strong> through the same stream code.
- Lambda's query layer must reconcile two separate views; Kappa exposes a single <strong class="kw">serving store</strong> with no merge step.
- Kappa's design hinges on a durable, long-retention <strong class="kw">event log</strong> capable of full replays, which Lambda does not require.

## When to Use Each

**Lambda Architecture**

- **Divergent batch/speed logic**: Use Lambda when the approximate real-time algorithm genuinely differs from the exact batch algorithm, such as retrained offline models versus simple real-time aggregates.
- **Existing batch infrastructure**: Organizations with mature Hadoop or Spark batch pipelines can bolt on a speed layer without migrating everything to streaming.
- **Infrequent, cheap recompute**: When periodic full recomputation is inexpensive and rare, the overhead of maintaining two pipelines is easier to absorb.

**Kappa Architecture**

- **Single source of truth**: Use Kappa when eliminating batch/speed drift and reconciliation logic is a priority for correctness or simplicity.
- **Kafka-centric stack**: Teams already running a durable, replayable log with sufficient retention can reprocess history without a separate batch system.
- **Frequent logic changes**: Deploying updated business logic is simpler with one pipeline than coordinating synchronized changes across batch and speed code.
