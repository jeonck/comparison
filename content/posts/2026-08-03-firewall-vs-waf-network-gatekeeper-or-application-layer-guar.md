---
title: "Firewall vs WAF: Network Gatekeeper or Application-Layer Guard"
date: 2026-08-03T04:25:25.812253+09:00
tags: ["firewall", "waf", "network-security", "application-security"]
---
## Overview

A firewall and a web application firewall (WAF) both filter traffic, but they operate at different layers of the stack. A firewall makes allow/deny decisions based on <strong class="kw">IP and port</strong>, while a WAF inspects the actual <strong class="kw">HTTP payload</strong> of requests to catch application-layer attacks like SQL injection and XSS. Most production environments deploy both, since neither can see what the other is built to catch.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">Firewall</text><text x="480" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">WAF</text><line x1="320" y1="45" x2="320" y2="335" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="160" y="58" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Raw network traffic</text><rect x="95" y="65" width="130" height="25" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="82" text-anchor="middle" font-size="10" style="fill:var(--content)">TCP SYN, dst port 22</text><line x1="160" y1="90" x2="160" y2="115" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="155,112 165,112 160,120" style="fill:var(--compare-a)"/><rect x="85" y="120" width="150" height="55" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><text x="160" y="142" text-anchor="middle" font-size="12" style="fill:var(--content)">Firewall</text><text x="160" y="158" text-anchor="middle" font-size="9.5" style="fill:var(--secondary)">L3/L4: IP, port, protocol</text><line x1="160" y1="175" x2="122" y2="200" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="160" y1="175" x2="197" y2="200" style="stroke:var(--compare-a)" stroke-width="1.5"/><rect x="90" y="200" width="65" height="28" rx="3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="122" y="218" text-anchor="middle" font-size="9.5" style="fill:var(--content)">Allow 443</text><rect x="165" y="200" width="65" height="28" rx="3" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="197" y="218" text-anchor="middle" font-size="9.5" style="fill:var(--content)">Block 22</text><text x="160" y="252" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Cannot see inside the</text><text x="160" y="266" text-anchor="middle" font-size="10" style="fill:var(--secondary)">HTTP request body</text><text x="480" y="55" text-anchor="middle" font-size="11" style="fill:var(--secondary)">HTTP request</text><rect x="395" y="65" width="170" height="25" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="82" text-anchor="middle" font-size="9" style="fill:var(--content)">GET /login?id=1' OR '1'='1</text><line x1="480" y1="90" x2="480" y2="115" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="475,112 485,112 480,120" style="fill:var(--compare-b)"/><rect x="405" y="120" width="150" height="55" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="480" y="142" text-anchor="middle" font-size="12" style="fill:var(--content)">WAF</text><text x="480" y="158" text-anchor="middle" font-size="9.5" style="fill:var(--secondary)">L7: URL, headers, body</text><line x1="480" y1="175" x2="442" y2="200" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="480" y1="175" x2="517" y2="200" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="410" y="200" width="65" height="28" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="442" y="218" text-anchor="middle" font-size="9.5" style="fill:var(--content)">Allow normal</text><rect x="485" y="200" width="65" height="28" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="3,3"/><text x="517" y="218" text-anchor="middle" font-size="9.5" style="fill:var(--content)">Block SQLi</text><text x="480" y="252" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Decrypts TLS to inspect</text><text x="480" y="266" text-anchor="middle" font-size="10" style="fill:var(--secondary)">the request payload</text><text x="320" y="310" text-anchor="middle" font-size="11" style="fill:var(--secondary)">Layered defense: firewall blocks unauthorized access, WAF blocks malicious payloads</text></svg>
</div>

## Comparison Table

| Aspect | Firewall | WAF |
| --- | --- | --- |
| OSI layer inspected | Network/transport (L3/L4) | Application (L7) |
| Traffic filtered | All IP traffic, any protocol or port | HTTP/HTTPS requests only |
| Inspection criteria | Source/destination IP, port, protocol, connection state | URL, headers, cookies, and request body content |
| Rule basis | Static allow/deny rules and ACLs | Signature and behavioral rules for known attack patterns |
| Typical deployment point | Network perimeter or between internal subnets | In front of or alongside web servers/load balancers |
| Attacks stopped | Port scans, unauthorized network access, network-layer floods | SQL injection, XSS, CSRF, other OWASP Top 10 exploits |
| Encrypted traffic handling | Sees only packet headers, not TLS-encrypted payload | Typically terminates TLS to inspect decrypted HTTP content |
| Maintenance cadence | Relatively static rule sets, infrequent changes | Frequent signature updates as new exploits are discovered |

## Key Differences

- A firewall filters at the <strong class="kw">network layer</strong> using IP and port, while a WAF filters at the <strong class="kw">application layer</strong> using HTTP content.
- Firewalls control which connections are permitted; WAFs inspect the <strong class="kw">payload</strong> within connections already allowed through.
- A WAF typically must <strong class="kw">decrypt TLS</strong> to read requests, whereas a firewall generally cannot see inside encrypted traffic.
- The two are <strong class="kw">complementary</strong> controls, not substitutes — each blocks a different class of attack the other misses.

## When to Use Each

**Firewall**

- **Network segmentation**: Restrict east-west traffic between VLANs or subnets to limit lateral movement.
- **Closing exposed ports**: Block public access to SSH, RDP, or database ports that should never face the internet.
- **Perimeter access control**: Allow traffic only from known IP ranges or trusted networks at the network edge.
- **Network-layer DDoS mitigation**: Drop malformed or excessive packets before they consume server resources.

**WAF**

- **Protecting public web apps**: Shield internet-facing applications from SQL injection, XSS, and CSRF attempts.
- **Compliance requirements**: Standards like PCI DSS require a WAF in front of web apps handling cardholder data.
- **Virtual patching**: Block exploitation of a known application vulnerability while a code fix is developed.
- **API endpoint protection**: Filter malicious payloads targeting REST or GraphQL endpoints before they reach app logic.
