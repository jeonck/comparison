---
title: "SAST vs DAST: Static vs Dynamic Application Security Testing"
date: 2026-08-03T04:19:31.769057+09:00
tags: ["appsec", "sast", "dast", "devsecops"]
---
## Overview

SAST scans an application's <strong class="kw">source code</strong> at rest to catch insecure patterns before the app ever runs, while DAST attacks a <strong class="kw">running application</strong> from the outside to find exploitable flaws in its live behavior. Teams use both because each catches vulnerability classes the other structurally cannot see.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="4 4"/><text x="150" y="40" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">SAST</text><text x="150" y="60" text-anchor="middle" style="fill:var(--secondary)" font-size="12">source code, no execution</text><rect x="60" y="80" width="180" height="180" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="72" y="102" style="fill:var(--content)" font-size="11" font-family="monospace">function login(u,p) {</text><text x="72" y="120" style="fill:var(--content)" font-size="11" font-family="monospace">  const q =</text><rect x="68" y="128" width="162" height="18" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="72" y="141" style="fill:var(--content)" font-size="11" font-family="monospace">    "SELECT..+p";</text><text x="72" y="159" style="fill:var(--content)" font-size="11" font-family="monospace">  db.exec(q);</text><text x="72" y="177" style="fill:var(--content)" font-size="11" font-family="monospace">}</text><circle cx="215" cy="137" r="16" style="fill:none;stroke:var(--compare-a)" stroke-width="2.5"/><line x1="226" y1="148" x2="238" y2="160" style="stroke:var(--compare-a)" stroke-width="2.5" stroke-linecap="round"/><text x="150" y="290" text-anchor="middle" style="fill:var(--content)" font-size="12">reads code, flags line 128</text><text x="150" y="308" text-anchor="middle" style="fill:var(--secondary)" font-size="11">e.g. unsanitized SQL concat</text><text x="490" y="40" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">DAST</text><text x="490" y="60" text-anchor="middle" style="fill:var(--secondary)" font-size="12">running app, black-box</text><rect x="400" y="80" width="180" height="120" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="490" y="110" text-anchor="middle" style="fill:var(--content)" font-size="12">Live App</text><text x="490" y="128" text-anchor="middle" style="fill:var(--content)" font-size="12">/login endpoint</text><circle cx="490" cy="160" r="14" style="fill:none;stroke:var(--compare-b)" stroke-width="2"/><path d="M484 160 L496 160 M490 154 L490 166" style="stroke:var(--compare-b)" stroke-width="2"/><rect x="430" y="235" width="120" height="36" rx="5" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5"/><text x="490" y="258" text-anchor="middle" style="fill:var(--content)" font-size="10" font-family="monospace">POST u=' OR 1=1--</text><path d="M490 205 L490 232" style="stroke:var(--compare-b)" stroke-width="1.5" marker-end="url(#arrow)"/><path d="M460 235 Q 460 210 480 202" style="stroke:var(--compare-b);fill:none" stroke-width="1.5"/><text x="490" y="300" text-anchor="middle" style="fill:var(--content)" font-size="12">sends live requests, observes response</text><text x="490" y="318" text-anchor="middle" style="fill:var(--secondary)" font-size="11">e.g. auth bypass returned</text></svg>
</div>

## Comparison Table

| Aspect | SAST | DAST |
| --- | --- | --- |
| What it examines | Source code, bytecode, or binaries at rest | A running, deployed application from outside |
| Access level | White-box — full visibility into code internals | Black-box — only sees inputs and outputs like an attacker |
| When in SDLC | Early, during coding and in CI on every commit | Later, once a build is deployed to a test or staging environment |
| Environment needed | None — analyzes files directly, no app needs to run | A live, reachable instance of the application |
| Vulnerability classes found | Insecure code patterns: SQL string building, hardcoded secrets, unsafe deserialization | Exploitable runtime behavior: auth bypass, injection responses, misconfigured headers |
| Language/stack dependency | Tied to the language and framework being parsed | Language-agnostic — probes over HTTP/HTTPS regardless of stack |
| False positive tendency | Higher — flags patterns that may not be reachable or exploitable | Lower — findings are confirmed by actual exploit attempts |
| Remediation output | Exact file and line number to fix | Vulnerable URL, parameter, and request/response evidence |

## Key Differences

- SAST inspects <strong class="kw">source code</strong> without running it; DAST attacks a <strong class="kw">live instance</strong> without seeing its internals
- SAST fits early into <strong class="kw">CI pipelines</strong> per-commit; DAST needs a <strong class="kw">deployed build</strong> to test against
- SAST pinpoints the exact <strong class="kw">line number</strong>; DAST reports the vulnerable <strong class="kw">endpoint</strong> and payload
- SAST is prone to <strong class="kw">false positives</strong> from unreachable code paths; DAST confirms via <strong class="kw">actual exploitation</strong>
- SAST misses <strong class="kw">runtime configuration</strong> flaws that DAST catches, like missing security headers or session issues

## When to Use Each

**SAST**

- **Shift-left in CI**: SAST runs on every pull request without needing a deployed environment, catching insecure code before merge.
- **Pinpointing exact fix location**: Developers get the specific file and line to patch, shortening remediation time.
- **Auditing proprietary logic**: Only SAST can see business logic and internal functions never exposed over the network.

**DAST**

- **Pre-release penetration testing**: DAST validates the deployed application the way a real attacker would interact with it.
- **Testing third-party or legacy components**: DAST works without source access, useful for vendored code or compiled binaries.
- **Catching runtime misconfigurations**: Issues like exposed debug endpoints or weak TLS settings only appear once the app is actually running.
