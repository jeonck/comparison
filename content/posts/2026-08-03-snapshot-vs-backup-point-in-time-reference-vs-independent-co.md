---
title: "Snapshot vs Backup: Point-in-Time Reference vs Independent Copy"
date: 2026-08-03T06:29:19.124910+09:00
tags: ["storage", "backup", "snapshot", "disaster-recovery"]
---
## Overview

A snapshot captures a volume's state using <strong class="kw">copy-on-write</strong> pointers that still depend on the original data, while a backup creates a fully <strong class="kw">independent copy</strong> stored elsewhere. The distinction matters because snapshots are fast and space-efficient for short-term rollback, but only backups protect against loss or corruption of the source system itself.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 z" style="fill:var(--compare-b)"/></marker></defs><rect x="250" y="30" width="140" height="60" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="320" y="46" text-anchor="middle" style="fill:var(--content)" font-size="12">Source Volume</text><rect x="271" y="55" width="18" height="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><rect x="295" y="55" width="18" height="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><rect x="319" y="55" width="18" height="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><rect x="343" y="55" width="18" height="18" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><line x1="300" y1="90" x2="170" y2="168" style="stroke:var(--compare-a)" stroke-width="2" stroke-dasharray="5,4" marker-end="url(#arrowA)"/><text x="205" y="125" text-anchor="middle" style="fill:var(--secondary)" font-size="11">pointer (COW)</text><line x1="340" y1="90" x2="470" y2="168" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="430" y="125" text-anchor="middle" style="fill:var(--secondary)" font-size="11">full copy</text><rect x="60" y="170" width="200" height="120" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="197" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Snapshot</text><text x="160" y="216" text-anchor="middle" style="fill:var(--secondary)" font-size="12">references source blocks</text><rect x="115" y="233" width="18" height="18" style="fill:none;stroke:var(--compare-a)" stroke-dasharray="3,2" stroke-width="1.5"/><rect x="139" y="233" width="18" height="18" style="fill:none;stroke:var(--compare-a)" stroke-dasharray="3,2" stroke-width="1.5"/><rect x="163" y="233" width="18" height="18" style="fill:none;stroke:var(--compare-a)" stroke-dasharray="3,2" stroke-width="1.5"/><rect x="187" y="233" width="18" height="18" style="fill:none;stroke:var(--compare-a)" stroke-dasharray="3,2" stroke-width="1.5"/><text x="160" y="270" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Same volume, low overhead</text><text x="160" y="285" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Invalid if source is lost</text><rect x="380" y="170" width="200" height="120" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="197" text-anchor="middle" style="fill:var(--primary)" font-size="18" font-weight="bold">Backup</text><text x="480" y="216" text-anchor="middle" style="fill:var(--secondary)" font-size="12">independent full copy</text><rect x="435" y="233" width="18" height="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="459" y="233" width="18" height="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="483" y="233" width="18" height="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="507" y="233" width="18" height="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="270" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Separate storage, survives loss</text><text x="480" y="285" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Slower, higher storage cost</text></svg>
</div>

## Comparison Table

| Aspect | Snapshot | Backup |
| --- | --- | --- |
| Primary purpose | Quick rollback to a prior state | Durable copy for disaster recovery and compliance |
| Capture mechanism | Copy-on-write or redirect-on-write pointers to existing blocks | Full or incremental copy of data written to separate storage |
| Storage location | Same storage system or volume as the source | Separate system, often offsite or on a different medium |
| Dependency on source | Invalidated if the source volume is deleted or corrupted | Independent copy, survives loss of the source |
| Creation speed and overhead | Near-instant, minimal I/O impact | Slower, with higher I/O, network, and storage cost |
| Retention and lifecycle | Short-lived, few kept because storage grows with changes | Long-term retention on a scheduled rotation policy |
| Recovery scope | Instant rollback on the same system or volume | Restore to a new or different system, file- or volume-level |
| Failure resilience | Vulnerable to the same hardware or storage failure as the source | Resilient to source failure or a site-wide disaster |

## Key Differences

- A snapshot stores <strong class="kw">pointers</strong> to existing blocks; a backup writes a full, separate copy of the data.
- Snapshots normally live on the <strong class="kw">same storage</strong> as the source; backups are placed on independent, often offsite media.
- Deleting the source volume can <strong class="kw">invalidate</strong> a snapshot, while a backup remains intact and restorable.
- Snapshots are created almost instantly with <strong class="kw">low overhead</strong>; backups take longer and consume more storage and bandwidth.
- Snapshots are typically kept briefly for rollback; backups follow a <strong class="kw">long-term retention</strong> policy for compliance.

## When to Use Each

**Snapshot**

- **Pre-upgrade rollback**: Take a snapshot right before a risky OS patch, schema migration, or config change so you can revert instantly if it fails.
- **Dev/test environment cloning**: Branch a VM or volume in seconds to spin up a test copy without waiting for a full data transfer.
- **Frequent low-overhead checkpoints**: Schedule hourly snapshots on active volumes where speed and minimal storage cost matter more than long-term durability.

**Backup**

- **Disaster recovery**: Backups stored independently protect against ransomware, accidental volume deletion, or total site failure that a snapshot cannot survive.
- **Regulatory retention**: Compliance mandates often require data to be retrievable for years, which needs a durable, independently retained backup.
- **Offsite or off-account protection**: Storing backups in a different location or cloud account guards against provider-level outages or compromised credentials.
