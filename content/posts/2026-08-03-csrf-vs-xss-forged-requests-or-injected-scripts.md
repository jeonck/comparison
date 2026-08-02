---
title: "CSRF vs XSS: Forged Requests or Injected Scripts"
date: 2026-08-03T04:27:16.123965+09:00
tags: ["web-security", "csrf", "xss", "appsec"]
---
## Overview

CSRF and XSS are both web attacks that abuse a victim's trusted relationship with a site, but they exploit opposite ends of that trust. CSRF forges a request using the victim's own <strong class="kw">session cookie</strong> without ever running attacker code in the browser, while XSS smuggles <strong class="kw">injected script</strong> into a vulnerable page so it executes directly inside the victim's browser.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="26" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">CSRF</text><text x="480" y="26" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">XSS</text><line x1="320" y1="40" x2="320" y2="345" style="stroke:var(--border)" stroke-width="1"/><rect x="30" y="45" width="260" height="42" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="62" text-anchor="middle" font-size="12" style="fill:var(--content)">Victim Browser</text><text x="160" y="76" text-anchor="middle" font-size="10" style="fill:var(--secondary)">active session cookie</text><rect x="30" y="112" width="260" height="42" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="129" text-anchor="middle" font-size="12" style="fill:var(--content)">Attacker Site</text><text x="160" y="143" text-anchor="middle" font-size="10" style="fill:var(--secondary)">forged auto-submit form</text><rect x="30" y="179" width="260" height="42" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="196" text-anchor="middle" font-size="12" style="fill:var(--content)">Target Server</text><text x="160" y="210" text-anchor="middle" font-size="10" style="fill:var(--secondary)">e.g. bank / app backend</text><rect x="30" y="246" width="260" height="42" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="263" text-anchor="middle" font-size="12" style="fill:var(--content)">Action Executed</text><text x="160" y="277" text-anchor="middle" font-size="10" style="fill:var(--secondary)">using victim's cookie</text><line x1="160" y1="87" x2="160" y2="108" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="154,104 166,104 160,113" style="fill:var(--compare-a)"/><text x="200" y="100" text-anchor="middle" font-size="9" style="fill:var(--secondary)">visits page</text><line x1="160" y1="154" x2="160" y2="175" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="154,171 166,171 160,180" style="fill:var(--compare-a)"/><text x="215" y="167" text-anchor="middle" font-size="9" style="fill:var(--secondary)">cookie auto-attached</text><line x1="160" y1="221" x2="160" y2="242" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="154,238 166,238 160,247" style="fill:var(--compare-a)"/><text x="205" y="234" text-anchor="middle" font-size="9" style="fill:var(--secondary)">no injected code</text><rect x="350" y="45" width="260" height="42" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="62" text-anchor="middle" font-size="12" style="fill:var(--content)">Vulnerable Site</text><text x="480" y="76" text-anchor="middle" font-size="10" style="fill:var(--secondary)">renders unsanitized input</text><rect x="350" y="112" width="260" height="42" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="129" text-anchor="middle" font-size="12" style="fill:var(--content)">Injected &lt;script&gt;</text><text x="480" y="143" text-anchor="middle" font-size="10" style="fill:var(--secondary)">runs in page's own origin</text><rect x="350" y="179" width="260" height="42" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="196" text-anchor="middle" font-size="12" style="fill:var(--content)">Victim Browser</text><text x="480" y="210" text-anchor="middle" font-size="10" style="fill:var(--secondary)">executes attacker JS</text><rect x="350" y="246" width="260" height="42" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="263" text-anchor="middle" font-size="12" style="fill:var(--content)">Attacker Server</text><text x="480" y="277" text-anchor="middle" font-size="10" style="fill:var(--secondary)">receives stolen data</text><line x1="480" y1="87" x2="480" y2="108" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="474,104 486,104 480,113" style="fill:var(--compare-b)"/><text x="530" y="100" text-anchor="middle" font-size="9" style="fill:var(--secondary)">input reflected as code</text><line x1="480" y1="154" x2="480" y2="175" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="474,171 486,171 480,180" style="fill:var(--compare-b)"/><text x="535" y="167" text-anchor="middle" font-size="9" style="fill:var(--secondary)">full DOM access</text><line x1="480" y1="221" x2="480" y2="242" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="474,238 486,238 480,247" style="fill:var(--compare-b)"/><text x="540" y="234" text-anchor="middle" font-size="9" style="fill:var(--secondary)">cookie exfiltrated</text></svg>
</div>

## Comparison Table

| Aspect | CSRF | XSS |
| --- | --- | --- |
| Attack vector | Forged cross-site request, e.g. an auto-submitting form or image tag on the attacker's page that targets the victim site | Malicious script injected into a vulnerable page's HTML or JS output, often via unsanitized user input |
| Trust exploited | Server's trust that any request carrying a valid session cookie came from the legitimate user | Browser's trust that all script served from the site's origin is safe to execute |
| Where the payload runs | Nowhere on the victim's browser beyond a normal HTTP request; the 'payload' is the request itself | Attacker's JavaScript executes directly inside the victim's browser, in the vulnerable site's own origin |
| Prerequisite for success | Victim must have an active authenticated session with the target site when the forged request fires | Vulnerable site must reflect, store, or render attacker-controlled input without proper sanitization or escaping |
| Attacker capability | Limited to whatever action the victim's existing session is authorized to perform, like a transfer or settings change | Broad: read cookies and localStorage, capture input, deface the page, or pivot into session hijacking |
| Primary defense | Anti-CSRF tokens, SameSite cookies, and origin or referer checks | Output encoding, Content Security Policy, and strict input sanitization |
| Typical impact scope | A single forged action, bounded by what the target endpoint allows | Potential full account takeover or persistent compromise if the injection is stored |

## Key Differences

- CSRF forges a request using the victim's existing <strong class="kw">session cookie</strong>; XSS injects <strong class="kw">attacker script</strong> that runs inside the victim's browser.
- CSRF requires no <strong class="kw">code injection</strong> into the target site, while XSS depends entirely on unsanitized input reaching the page.
- XSS can read and exfiltrate data straight from the DOM, whereas CSRF is limited to <strong class="kw">blind requests</strong> with no response visibility.
- <strong class="kw">SameSite cookies</strong> mitigate CSRF but do nothing against XSS, which is stopped primarily by <strong class="kw">CSP</strong> and output encoding.

## When to Use Each

**CSRF**

- **Auditing state-changing endpoints**: Any POST, PUT, or DELETE that relies solely on cookie auth should be reviewed for CSRF exposure.
- **Reviewing cookie-based auth systems**: Sites using cookies without SameSite protection need explicit anti-CSRF tokens on sensitive actions.
- **Assessing forged submission risk**: Relevant when evaluating whether an attacker's page could trigger unwanted actions on your site via the victim's browser.

**XSS**

- **Reviewing user-generated content**: Comments, profiles, and search results that echo input back to the page need output-escaping review.
- **Auditing third-party script inclusion**: Anywhere external or user-supplied scripts run in your origin is a direct XSS risk surface.
- **Investigating account takeover reports**: Session or cookie theft complaints often trace back to an XSS injection point somewhere in the app.
