---
title: "TLS vs SSL: Encryption Protocol Evolution"
date: 2026-08-03T04:17:43.312659+09:00
tags: ["tls", "ssl", "encryption", "network-security"]
---
## Overview

SSL and TLS are cryptographic protocols that secure data in transit between clients and servers, but SSL is the deprecated <strong class="kw">predecessor</strong> while TLS is its actively maintained <strong class="kw">successor</strong>. Every SSL version is now broken or prohibited, yet the term "SSL" persists in everyday usage even though modern connections actually negotiate TLS.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="140" y="36" font-size="22" font-weight="bold" text-anchor="middle" style="fill:var(--primary)">SSL</text><text x="480" y="36" font-size="22" font-weight="bold" text-anchor="middle" style="fill:var(--primary)">TLS</text><line x1="20" y1="60" x2="620" y2="60" stroke-width="1.5" style="stroke:var(--border)"/><rect x="40" y="80" width="180" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="98" font-size="14" text-anchor="middle" style="fill:var(--content)">SSL 2.0 (1995)</text><text x="130" y="115" font-size="11" text-anchor="middle" style="fill:var(--secondary)">broken by DROWN</text><rect x="40" y="140" width="180" height="44" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="158" font-size="14" text-anchor="middle" style="fill:var(--content)">SSL 3.0 (1996)</text><text x="130" y="175" font-size="11" text-anchor="middle" style="fill:var(--secondary)">broken by POODLE</text><rect x="40" y="200" width="180" height="36" rx="6" stroke-dasharray="4 3" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="130" y="223" font-size="13" text-anchor="middle" style="fill:var(--secondary)">all versions prohibited</text><path d="M230 118 L390 98" style="stroke:var(--border)" stroke-width="1.5" fill="none" marker-end="url(#arrow)"/><defs><marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" style="fill:var(--border)"/></marker></defs><rect x="400" y="70" width="200" height="38" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="94" font-size="13" text-anchor="middle" style="fill:var(--content)">TLS 1.0 (1999)</text><rect x="400" y="118" width="200" height="38" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="142" font-size="13" text-anchor="middle" style="fill:var(--content)">TLS 1.1 (2006)</text><rect x="400" y="166" width="200" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="190" font-size="13" text-anchor="middle" style="fill:var(--content)">TLS 1.2 (2008)</text><text x="500" y="203" font-size="11" text-anchor="middle" style="fill:var(--secondary)">widely deployed</text><rect x="400" y="216" width="200" height="40" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="500" y="240" font-size="13" text-anchor="middle" style="fill:var(--content)">TLS 1.3 (2018)</text><text x="500" y="253" font-size="11" text-anchor="middle" style="fill:var(--secondary)">current standard</text><line x1="40" y1="300" x2="600" y2="300" stroke-width="1.5" style="stroke:var(--border)" marker-end="url(#arrow)"/><text x="320" y="320" font-size="12" text-anchor="middle" style="fill:var(--secondary)">time →</text><text x="130" y="340" font-size="12" text-anchor="middle" style="fill:var(--compare-a)">deprecated / prohibited</text><text x="500" y="340" font-size="12" text-anchor="middle" style="fill:var(--compare-b)">actively maintained</text></svg>
</div>

## Comparison Table

| Aspect | SSL | TLS |
| --- | --- | --- |
| Origin | Developed by Netscape starting in 1995 | Standardized by the IETF in 1999 as SSL's successor |
| Versions released | SSL 2.0, SSL 3.0 (SSL 1.0 never shipped) | TLS 1.0, 1.1, 1.2, 1.3 |
| Handshake process | Full handshake only, with weaker key exchange options | Streamlined handshake; TLS 1.3 cuts a round trip and defaults to forward secrecy |
| Cipher suite support | Permits weak ciphers like RC4, DES, and export-grade crypto | Mandates modern AEAD ciphers (AES-GCM, ChaCha20-Poly1305); weak ciphers dropped entirely in 1.3 |
| Known vulnerabilities | POODLE broke SSL 3.0; DROWN broke SSL 2.0 | BEAST and CRIME hit early TLS 1.0 but were patched in later versions |
| Current status | All versions formally deprecated and prohibited (RFC 7568) | TLS 1.2 and 1.3 are the current standards; 1.0/1.1 also deprecated |
| Everyday terminology | "SSL certificate" and "SSL/TLS" persist as colloquial shorthand | The protocol actually negotiated by nearly every modern HTTPS connection |

## Key Differences

- SSL is the obsolete <strong class="kw">predecessor</strong>; TLS is the actively maintained <strong class="kw">successor</strong> protocol
- TLS 1.3's handshake trims a <strong class="kw">round trip</strong> compared to SSL's full handshake
- SSL still permits weak ciphers like <strong class="kw">RC4</strong>; TLS mandates modern AEAD ciphers
- The label "<strong class="kw">SSL certificate</strong>" survives in marketing even though browsers negotiate TLS
- SSL 3.0 was broken by <strong class="kw">POODLE</strong>, forcing its complete deprecation

## When to Use Each

**SSL**

- **Legacy system interop**: Only relevant when forced to talk to decades-old hardware or software that can't negotiate anything newer, and even then it should be upgraded rather than relied on.
- **Protocol history research**: Understanding SSL's design and failures explains why TLS made the specific security choices it did.

**TLS**

- **Securing modern web traffic**: TLS 1.2/1.3 is the baseline expectation for any HTTPS endpoint today.
- **API and service-to-service encryption**: TLS's AEAD ciphers and forward secrecy protect internal and external API calls from interception.
- **Meeting compliance mandates**: Standards like PCI-DSS explicitly require TLS 1.2 or higher and prohibit SSL outright.
