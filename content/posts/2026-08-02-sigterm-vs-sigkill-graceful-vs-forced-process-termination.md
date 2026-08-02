---
title: "SIGTERM vs SIGKILL: Graceful vs Forced Process Termination"
date: 2026-08-02T09:26:50.280637+09:00
tags: ["linux", "signals", "process-management", "unix"]
---
## Overview

SIGTERM and SIGKILL are both Unix signals used to stop a running process, but they differ in whether the process gets a chance to clean up after itself. SIGTERM asks a process to terminate and lets it run its own shutdown logic, while SIGKILL is an unconditional kernel-level termination that the process cannot intercept or delay. The distinction matters for avoiding data corruption, leaked resources, and orphaned locks during shutdown.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="15" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><text x="160" y="30" text-anchor="middle" style="fill:var(--compare-a)" font-size="20" font-weight="bold">SIGTERM</text><text x="160" y="48" text-anchor="middle" style="fill:var(--secondary)" font-size="12">signal 15</text><line x1="160" y1="56" x2="160" y2="94" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="100" y="94" width="120" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="118" text-anchor="middle" style="fill:var(--content)" font-size="13">Process</text><line x1="160" y1="132" x2="160" y2="166" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="85" y="166" width="150" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="183" text-anchor="middle" style="fill:var(--content)" font-size="12">Signal handler runs</text><text x="160" y="197" text-anchor="middle" style="fill:var(--secondary)" font-size="10">(can also be ignored)</text><line x1="160" y1="204" x2="160" y2="238" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="70" y="238" width="180" height="38" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="262" text-anchor="middle" style="fill:var(--content)" font-size="12">Flush, close, release locks</text><line x1="160" y1="276" x2="160" y2="310" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="330" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="bold">exit 143</text><text x="480" y="30" text-anchor="middle" style="fill:var(--compare-b)" font-size="20" font-weight="bold">SIGKILL</text><text x="480" y="48" text-anchor="middle" style="fill:var(--secondary)" font-size="12">signal 9</text><line x1="480" y1="56" x2="480" y2="94" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="420" y="94" width="120" height="38" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="118" text-anchor="middle" style="fill:var(--content)" font-size="13">Process</text><line x1="480" y1="132" x2="480" y2="166" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="405" y="166" width="150" height="38" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 3"/><text x="480" y="189" text-anchor="middle" style="fill:var(--secondary)" font-size="12">no handler runs</text><line x1="415" y1="172" x2="545" y2="196" style="stroke:var(--secondary)" stroke-width="1"/><line x1="545" y1="172" x2="415" y2="196" style="stroke:var(--secondary)" stroke-width="1"/><line x1="480" y1="204" x2="480" y2="238" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="390" y="238" width="180" height="38" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="262" text-anchor="middle" style="fill:var(--content)" font-size="12">Kernel force-removes process</text><line x1="480" y1="276" x2="480" y2="310" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="330" text-anchor="middle" style="fill:var(--primary)" font-size="14" font-weight="bold">exit 137</text></svg>
</div>

## Comparison Table

| Aspect | SIGTERM | SIGKILL |
| --- | --- | --- |
| Signal number | 15 | 9 |
| Triggered by | kill, docker stop, systemctl stop, Kubernetes preStop hook | kill -9, orchestrator escalation after grace period, OOM killer |
| Can be caught or blocked | Yes, process can install a handler, ignore it, or delay it | No, delivered directly by the kernel and cannot be caught, blocked, or ignored |
| Cleanup opportunity | Handler can flush buffers, close sockets, release locks, save state | None, process's own code never runs before removal |
| Termination timing | Not guaranteed, depends on handler duration or can hang indefinitely | Near-immediate, enforced by the kernel except for processes stuck in uninterruptible D state |
| Resulting exit status | Depends on handler logic, default 143 (128+15) if unhandled | Always 137 (128+9), no application-level exit path |
| Typical role | First step in a graceful shutdown sequence | Last-resort escalation when SIGTERM is ignored or the process is hung |

## Key Differences

- SIGTERM is <strong class="kw">catchable</strong>, letting a process run its own shutdown handler before exiting.
- SIGKILL bypasses userspace entirely and is enforced directly by the <strong class="kw">kernel</strong>, so it cannot be intercepted.
- Only SIGTERM gives a process the chance to perform <strong class="kw">cleanup</strong> like flushing writes or releasing locks.
- Orchestrators like Docker and Kubernetes send SIGTERM first, then escalate to SIGKILL after a <strong class="kw">grace period</strong>.
- Processes stuck in uninterruptible <strong class="kw">D state</strong> I/O wait can delay even SIGKILL until the I/O completes.

## When to Use Each

**SIGTERM** — Use SIGTERM as the default way to stop a process, giving it a chance to shut down cleanly by closing connections, saving state, and exiting child processes.

**SIGKILL** — Use SIGKILL only when a process ignores SIGTERM or becomes unresponsive, since it forces immediate termination with no cleanup and risks orphaned resources or data corruption.
