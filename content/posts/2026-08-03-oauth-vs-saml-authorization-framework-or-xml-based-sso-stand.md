---
title: "OAuth vs SAML: Authorization Framework or XML-Based SSO Standard"
date: 2026-08-03T04:22:17.080642+09:00
tags: ["oauth", "saml", "authentication", "sso"]
---
## Overview

OAuth and SAML both let one system vouch for a user to another, but they solve different problems: OAuth is an <strong class="kw">authorization</strong> framework built to grant apps limited access to APIs, while SAML is an <strong class="kw">XML-based</strong> standard built for enterprise single sign-on. Picking the wrong one means using a token-delegation protocol for an identity-federation problem, or vice versa.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="130" y="28" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--compare-a)">OAuth</text><text x="130" y="46" text-anchor="middle" font-size="11" style="fill:var(--secondary)">delegated authorization</text><text x="510" y="28" text-anchor="middle" font-size="18" font-weight="bold" style="fill:var(--compare-b)">SAML</text><text x="510" y="46" text-anchor="middle" font-size="11" style="fill:var(--secondary)">federated authentication</text><line x1="320" y1="56" x2="320" y2="345" stroke-dasharray="4 4" style="stroke:var(--border)" stroke-width="1.5"/><rect x="60" y="62" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="87" text-anchor="middle" font-size="12" style="fill:var(--content)">Client App</text><line x1="130" y1="102" x2="130" y2="140" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="125,138 135,138 130,148" style="fill:var(--compare-a)"/><rect x="60" y="148" width="140" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="170" text-anchor="middle" font-size="11" style="fill:var(--content)">Authorization</text><text x="130" y="183" text-anchor="middle" font-size="11" style="fill:var(--content)">Server</text><line x1="130" y1="188" x2="130" y2="226" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="125,224 135,224 130,234" style="fill:var(--compare-a)"/><rect x="60" y="234" width="140" height="40" rx="4" stroke-dasharray="3 3" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="252" text-anchor="middle" font-size="11" style="fill:var(--content)">Access Token</text><text x="130" y="265" text-anchor="middle" font-size="10" style="fill:var(--secondary)">{ JSON }</text><line x1="130" y1="274" x2="130" y2="304" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="125,302 135,302 130,312" style="fill:var(--compare-a)"/><rect x="60" y="312" width="140" height="36" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="130" y="334" text-anchor="middle" font-size="11" style="fill:var(--content)">Resource API</text><rect x="440" y="62" width="140" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="87" text-anchor="middle" font-size="12" style="fill:var(--content)">Service Provider</text><line x1="510" y1="102" x2="510" y2="140" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="505,138 515,138 510,148" style="fill:var(--compare-b)"/><rect x="440" y="148" width="140" height="40" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="172" text-anchor="middle" font-size="11" style="fill:var(--content)">Identity Provider</text><line x1="510" y1="188" x2="510" y2="226" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="505,224 515,224 510,234" style="fill:var(--compare-b)"/><rect x="440" y="234" width="140" height="40" rx="4" stroke-dasharray="3 3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="252" text-anchor="middle" font-size="11" style="fill:var(--content)">SAML Assertion</text><text x="510" y="265" text-anchor="middle" font-size="10" style="fill:var(--secondary)">&lt;XML&gt;</text><line x1="510" y1="274" x2="510" y2="304" style="stroke:var(--compare-b)" stroke-width="1.5"/><polygon points="505,302 515,302 510,312" style="fill:var(--compare-b)"/><rect x="440" y="312" width="140" height="36" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="510" y="334" text-anchor="middle" font-size="11" style="fill:var(--content)">SP Session</text></svg>
</div>

## Comparison Table

| Aspect | OAuth | SAML |
| --- | --- | --- |
| Primary purpose | Authorization - grants an app limited, scoped access to resources | Authentication - proves a user's identity for single sign-on |
| Assertion/token format | JSON, most commonly a JWT access token | XML, a digitally signed SAML assertion |
| Flow initiation | Client app redirects the user to an authorization server to request consent | Service provider redirects the user to an identity provider to authenticate |
| Credential delivery | Access token returned via redirect or back-channel to the client | Signed assertion POSTed back to the service provider's endpoint |
| Transport mechanism | REST/HTTP calls carrying a bearer token in the Authorization header | HTTP redirect and POST bindings, historically also SOAP |
| Session establishment | Client presents the token on each API call to prove access rights | Service provider validates the assertion once and creates a local session |
| Lifetime and renewal | Short-lived access tokens paired with long-lived refresh tokens | Assertions tied to the SSO session with no built-in refresh mechanism |
| Typical ecosystem | Mobile apps, SPAs, and third-party API integrations (Google, GitHub) | Enterprise SSO into web apps via IdPs like Okta, ADFS, or Azure AD |

## Key Differences

- OAuth is fundamentally an <strong class="kw">authorization</strong> framework, not an identity protocol, though OpenID Connect layers authentication on top of it.
- SAML assertions are <strong class="kw">XML</strong>-based and signed, while OAuth tokens are typically <strong class="kw">JSON</strong>, often as a JWT.
- SAML is built around browser <strong class="kw">redirects</strong> and POST bindings for SSO, while OAuth is built around bearer tokens for API calls.
- OAuth supports <strong class="kw">refresh tokens</strong> for renewing access; SAML assertions have no native renewal and rely on re-authentication.

## When to Use Each

**OAuth**

- **Third-Party API Access**: OAuth lets a client app call a resource server's API with a scoped, revocable access token instead of sharing credentials.
- **Mobile & SPA Login**: OAuth's token-based model fits public clients that can't securely store long-lived session state the way server-rendered apps can.
- **Delegated Resource Access**: OAuth is designed for scenarios where a user grants one app limited permission to act on their data held by another service.

**SAML**

- **Enterprise Single Sign-On**: SAML is the entrenched standard for letting employees authenticate once with a corporate IdP and access many internal web apps.
- **Legacy Web App Integration**: Many older enterprise SaaS platforms only support SAML-based SSO, not OAuth or OIDC, for federated login.
- **Compliance-Driven Identity Federation**: SAML's signed XML assertions and mature audit trail fit regulated environments that require strict identity federation guarantees.
