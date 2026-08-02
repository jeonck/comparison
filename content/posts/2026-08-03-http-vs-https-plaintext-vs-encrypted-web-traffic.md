---
title: "HTTP vs HTTPS: Plaintext vs Encrypted Web Traffic"
date: 2026-08-01T20:01:00+09:00
tags: ["networking", "web-security", "http", "tls"]
---
## Overview

HTTP and HTTPS are the same application-layer protocol for transferring web resources, but HTTPS wraps every request and response in a <strong class="kw">TLS</strong> tunnel before it touches the network. That single layer determines whether credentials, cookies, and page content travel as <strong class="kw">plaintext</strong> visible to anyone on the path, or as ciphertext only the two endpoints can read.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="20" y="32" font-size="18" font-weight="700" style="fill:var(--primary)">HTTP</text><rect x="40" y="70" width="110" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="95" y="100" font-size="13" text-anchor="middle" style="fill:var(--content)">Client</text><rect x="490" y="70" width="110" height="50" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="545" y="100" font-size="13" text-anchor="middle" style="fill:var(--content)">Server</text><line x1="150" y1="95" x2="490" y2="95" style="stroke:var(--compare-a)" stroke-width="2" stroke-dasharray="5,4" marker-end="url(#arrowA)"/><text x="320" y="82" font-size="12" text-anchor="middle" style="fill:var(--content)">GET /login?pwd=hunter2</text><circle cx="320" cy="140" r="14" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><path d="M310 140 Q320 132 330 140 Q320 148 310 140 Z" style="fill:none;stroke:var(--compare-a)" stroke-width="1.3"/><circle cx="320" cy="140" r="2.5" style="fill:var(--compare-a)"/><text x="320" y="165" font-size="11" text-anchor="middle" style="fill:var(--secondary)">visible to anyone on path</text><line x1="0" y1="195" x2="640" y2="195" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><text x="20" y="225" font-size="18" font-weight="700" style="fill:var(--primary)">HTTPS</text><rect x="40" y="260" width="110" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="95" y="290" font-size="13" text-anchor="middle" style="fill:var(--content)">Client</text><rect x="490" y="260" width="110" height="50" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="545" y="290" font-size="13" text-anchor="middle" style="fill:var(--content)">Server</text><line x1="150" y1="285" x2="490" y2="285" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="320" y="272" font-size="12" text-anchor="middle" style="fill:var(--content)">x8f#9a2$qL0e...</text><rect x="308" y="296" width="24" height="18" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><path d="M313 296 v-8 a7 7 0 0 1 14 0 v8" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5"/><text x="320" y="335" font-size="11" text-anchor="middle" style="fill:var(--secondary)">TLS-encrypted, tamper-evident</text><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs></svg>
</div>

## Comparison Table

| Aspect | HTTP | HTTPS |
| --- | --- | --- |
| Default port | 80 | 443 |
| Connection establishment | Single TCP three-way handshake | TCP handshake plus a TLS handshake to negotiate cipher and exchange keys |
| Certificate requirement | None | X.509 certificate issued by a trusted CA (or self-signed) required |
| Data encryption | Plaintext — headers, cookies, and body sent unencrypted | Encrypted end-to-end using TLS/SSL symmetric ciphers |
| Data integrity | No built-in tamper detection | MAC/AEAD in TLS detects in-transit tampering |
| Browser indicator | "Not secure" warning in modern browsers | Padlock icon; no warning shown |
| Performance overhead | Lower — no crypto or extra round trip | Slightly higher handshake/CPU cost, largely offset by TLS 1.3 and session resumption |
| Typical use case | Local development, internal tools on trusted networks, legacy static content | Any production site, especially logins, payments, and APIs handling sensitive data |

## Key Differences

- HTTPS is HTTP tunneled through <strong class="kw">TLS</strong>, not a separate application protocol
- HTTP traffic is readable in plaintext by anyone with network access; HTTPS traffic is <strong class="kw">encrypted</strong>
- HTTPS requires a valid <strong class="kw">certificate</strong> from a trusted CA to establish trust
- Modern browsers flag HTTP sites as <strong class="kw">not secure</strong>, pushing HTTPS as the default
- TLS 1.3 has shrunk the historical HTTPS <strong class="kw">handshake</strong> cost to near parity with plain TCP

## When to Use Each

**HTTP**

- **Local Development Servers**: Spinning up a dev server on localhost carries no real eavesdropping risk, so plain HTTP avoids the friction of self-signed certs.
- **Internal Trusted Networks**: Tools running strictly within an isolated, access-controlled LAN may accept plaintext transport where the network itself is the trust boundary.
- **Learning Protocol Basics**: Studying raw HTTP request/response mechanics is easier without the TLS handshake obscuring the underlying messages.

**HTTPS**

- **Any Production Website**: Public-facing sites need HTTPS by default since browsers and search engines now penalize or block plain HTTP.
- **Login and Payment Forms**: Encryption is mandatory whenever credentials, tokens, or financial data cross the network.
- **Regulatory Compliance**: Standards like PCI-DSS and general privacy law effectively require TLS for any handling of personal or payment data.
- **API Authentication**: Bearer tokens and API keys sent in headers must be encrypted in transit to prevent interception.
