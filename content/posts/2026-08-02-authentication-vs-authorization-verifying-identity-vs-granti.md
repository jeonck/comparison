---
title: "Authentication vs. Authorization: Verifying Identity vs. Granting Access"
date: 2026-08-02T05:18:32.056083+09:00
tags: ["authentication", "authorization", "security", "access-control"]
---
## Overview

Authentication (AuthN) confirms who a user or system claims to be, typically through credentials like passwords, biometrics, or tokens. Authorization (AuthZ) determines what an already-authenticated identity is permitted to do or access. The two are sequential and often conflated, but security bugs frequently trace back to confusing one for the other — e.g., checking that a user is logged in without checking they're allowed to see a specific resource.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><circle cx="60" cy="182" r="26" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><circle cx="60" cy="172" r="7" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><path d="M48,198 Q60,180 72,198" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="60" y="227" text-anchor="middle" font-size="11" style="fill:var(--content)">Client</text><line x1="88" y1="180" x2="146" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="146,180 138,175 138,185" style="fill:var(--border)"/><text x="117" y="172" text-anchor="middle" font-size="10" style="fill:var(--secondary)">credentials</text><text x="230" y="95" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--compare-a)">AUTHENTICATION</text><text x="230" y="113" text-anchor="middle" font-size="11" style="fill:var(--secondary)">"Who are you?"</text><rect x="150" y="130" width="160" height="100" rx="8" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><circle cx="205" cy="170" r="9" style="fill:none;stroke:var(--compare-a)" stroke-width="2"/><line x1="214" y1="170" x2="236" y2="170" style="stroke:var(--compare-a)" stroke-width="2"/><line x1="228" y1="170" x2="228" y2="178" style="stroke:var(--compare-a)" stroke-width="2"/><line x1="236" y1="170" x2="236" y2="176" style="stroke:var(--compare-a)" stroke-width="2"/><text x="230" y="218" text-anchor="middle" font-size="12" style="fill:var(--content)">Verifies identity</text><text x="230" y="250" text-anchor="middle" font-size="11" style="fill:var(--secondary)">401 Unauthorized</text><text x="230" y="266" text-anchor="middle" font-size="10" style="fill:var(--secondary)">login, MFA, SSO</text><line x1="312" y1="180" x2="346" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="346,180 338,175 338,185" style="fill:var(--border)"/><text x="329" y="172" text-anchor="middle" font-size="10" style="fill:var(--secondary)">identity/token</text><text x="430" y="95" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--compare-b)">AUTHORIZATION</text><text x="430" y="113" text-anchor="middle" font-size="11" style="fill:var(--secondary)">"What can you do?"</text><rect x="350" y="130" width="160" height="100" rx="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><path d="M430,148 L448,156 L448,178 Q448,195 430,204 Q412,195 412,178 L412,156 Z" style="fill:none;stroke:var(--compare-b)" stroke-width="2"/><path d="M421,177 L428,185 L441,165" style="fill:none;stroke:var(--compare-b)" stroke-width="2"/><text x="430" y="218" text-anchor="middle" font-size="12" style="fill:var(--content)">Grants/denies access</text><text x="430" y="250" text-anchor="middle" font-size="11" style="fill:var(--secondary)">403 Forbidden</text><text x="430" y="266" text-anchor="middle" font-size="10" style="fill:var(--secondary)">RBAC, scopes, ACLs</text><line x1="512" y1="180" x2="566" y2="180" style="stroke:var(--border)" stroke-width="1.5"/><polygon points="566,180 558,175 558,185" style="fill:var(--border)"/><text x="539" y="172" text-anchor="middle" font-size="10" style="fill:var(--secondary)">decision</text><circle cx="600" cy="182" r="26" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><rect x="590" y="178" width="20" height="16" rx="2" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><path d="M594,178 L594,172 Q594,166 600,166 Q606,166 606,172 L606,178" style="fill:none;stroke:var(--content)" stroke-width="1.5"/><text x="600" y="223" text-anchor="middle" font-size="11" style="fill:var(--content)">Resource</text></svg>
</div>

## Comparison Table

| Aspect | Authentication | Authorization |
| --- | --- | --- |
| Question answered | Who are you? | What are you allowed to do? |
| Occurs when | At login, before any access is granted | After authentication, on every protected action or resource |
| Input | Credentials — password, biometric, hardware key, OTP | Verified identity plus roles, scopes, or attributes |
| Output | A confirmed identity (session, JWT, ID token) | An allow/deny decision |
| Failure response | 401 Unauthorized | 403 Forbidden |
| Common mechanisms | Passwords, MFA, WebAuthn/passkeys, SSO | RBAC, ABAC, ACLs, OAuth 2.0 scopes |
| Relevant standards | OpenID Connect, SAML, WebAuthn | OAuth 2.0, XACML, policy engines (e.g. OPA) |
| Enforced by | Identity provider or login boundary | Application logic, API gateway, or policy engine |

## Key Differences

- Authorization always presumes authentication already happened — you can't check permissions for an identity you haven't verified.
- 401 vs 403 encode the distinction directly: 401 means 'we don't know who you are,' 403 means 'we know, but you can't do that.'
- Authentication is typically a one-time event per session; authorization is re-evaluated on every sensitive request or resource access.
- OAuth 2.0 is an authorization framework, not an authentication protocol — OpenID Connect layers identity verification on top of it, a distinction frequently misunderstood in practice.

## When to Use Each

**Authentication**

- **Hardening Login Flows**: Authentication is the layer to strengthen when the goal is proving identity through passwords, biometrics, or hardware keys.
- **Adding Multi-Factor Authentication**: Layering in MFA improves how credentials are verified, entirely on the authentication side, before any access decision is made.
- **Integrating SSO or OpenID Connect**: Centralizing identity verification across multiple applications is an authentication concern, built on standards like OpenID Connect and SAML.

**Authorization**

- **Designing RBAC/ABAC Models**: Once identity is confirmed, deciding what a role or attribute set can do is purely an authorization problem.
- **Scoping OAuth 2.0 API Access**: OAuth 2.0 is an authorization framework, making it the right tool for limiting what a token holder can do, not verifying who they are.
- **Enforcing Per-Resource Permission Checks**: Authorization is re-evaluated on every sensitive request, so it's the layer to address when a specific resource needs protection beyond the initial login.
