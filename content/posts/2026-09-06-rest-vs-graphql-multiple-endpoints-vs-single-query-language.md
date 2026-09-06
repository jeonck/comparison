---
title: "REST vs GraphQL: Multiple Endpoints vs Single Query Language"
date: 2026-09-06T10:03:08.944309+09:00
tags: ["rest", "graphql", "api-design", "web-architecture"]
---
## Overview

REST structures an API as a fixed set of <strong class="kw">endpoints</strong>, each returning a predetermined shape of data tied to a resource. GraphQL exposes a single endpoint driven by a client-specified <strong class="kw">query</strong>, letting callers request exactly the fields they need across related resources in one round trip.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">REST</text><text x="480" y="30" text-anchor="middle" font-size="18" style="fill:var(--primary)">GraphQL</text><circle cx="60" cy="90" r="18" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="60" y="95" text-anchor="middle" font-size="11" style="fill:var(--content)">Client</text><rect x="140" y="55" width="100" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="190" y="75" text-anchor="middle" font-size="11" style="fill:var(--content)">/users/1</text><rect x="140" y="100" width="100" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="190" y="120" text-anchor="middle" font-size="11" style="fill:var(--content)">/users/1/posts</text><rect x="140" y="145" width="100" height="32" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="190" y="165" text-anchor="middle" font-size="11" style="fill:var(--content)">/posts/1/comments</text><line x1="78" y1="90" x2="140" y2="71" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="78" y1="90" x2="140" y2="116" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="78" y1="90" x2="140" y2="161" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="190" y="200" text-anchor="middle" font-size="11" style="fill:var(--secondary)">3 requests, fixed shapes</text><rect x="20" y="230" width="340" height="70" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><text x="190" y="255" text-anchor="middle" font-size="11" style="fill:var(--content)">Response 1: full user object</text><text x="190" y="273" text-anchor="middle" font-size="11" style="fill:var(--content)">Response 2: full posts array</text><text x="190" y="291" text-anchor="middle" font-size="11" style="fill:var(--secondary)">may over- or under-fetch fields</text><circle cx="400" cy="90" r="18" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="95" text-anchor="middle" font-size="11" style="fill:var(--content)">Client</text><rect x="480" y="75" width="120" height="32" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="540" y="95" text-anchor="middle" font-size="11" style="fill:var(--content)">/graphql</text><line x1="418" y1="90" x2="480" y2="91" style="stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="130" width="160" height="70" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1" stroke-dasharray="4 3"/><text x="520" y="150" text-anchor="middle" font-size="10" style="fill:var(--content)">{ user(id:1){</text><text x="520" y="165" text-anchor="middle" font-size="10" style="fill:var(--content)">name posts{ title }</text><text x="520" y="180" text-anchor="middle" font-size="10" style="fill:var(--content)">} }</text><text x="520" y="225" text-anchor="middle" font-size="11" style="fill:var(--secondary)">1 request, client-shaped</text><rect x="440" y="250" width="160" height="50" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="520" y="272" text-anchor="middle" font-size="11" style="fill:var(--content)">Single JSON response</text><text x="520" y="288" text-anchor="middle" font-size="11" style="fill:var(--secondary)">matches requested fields</text></svg>
</div>

## Comparison Table

| Aspect | REST | GraphQL |
| --- | --- | --- |
| Request entry point | Multiple resource-based URLs (e.g. /users, /posts) | Single endpoint (e.g. /graphql) for all operations |
| Query specification | Server defines response shape per endpoint | Client defines response shape via query document |
| Fetching related data | Requires multiple round trips or ad-hoc nested routes | Nested relations resolved in one request via resolvers |
| Over/under-fetching | Common — fixed payloads return unused or missing fields | Minimized — client requests exactly the fields it needs |
| Caching | Leverages HTTP caching (ETags, CDNs, cache-control) | Requires custom client-side or persisted-query caching |
| Versioning strategy | New versions (/v2/) or new endpoints for breaking changes | Schema evolves additively; fields deprecated in place |
| Error handling | HTTP status codes signal success/failure per request | 200 OK typical even on partial errors; errors array in body |
| Tooling and discovery | Relies on external docs (OpenAPI/Swagger) for contracts | Self-describing schema with built-in introspection |

## Key Differences

- REST models an API around <strong class="kw">resources</strong> and HTTP verbs; GraphQL models it around a typed <strong class="kw">schema</strong> and queries.
- REST responses have a shape fixed by the server; GraphQL responses are shaped by the <strong class="kw">client query</strong> itself.
- REST benefits from standard HTTP <strong class="kw">caching</strong> infrastructure; GraphQL typically needs bespoke caching layers.
- Fetching nested or related data usually takes REST multiple <strong class="kw">round trips</strong>, while GraphQL resolves it in a single request.
- REST signals failures through HTTP <strong class="kw">status codes</strong>; GraphQL usually returns 200 with errors embedded in the payload.

## When to Use Each

**REST**

- **Simple CRUD services**: REST's resource/verb model maps directly onto straightforward create-read-update-delete operations without extra query machinery.
- **Public APIs needing HTTP caching**: REST responses cache naturally at the HTTP layer via CDNs, proxies, and browser caches.
- **File uploads and streaming**: REST handles binary payloads and streaming responses more directly than GraphQL's JSON-centric transport.

**GraphQL**

- **Complex, nested data needs**: GraphQL lets a mobile or web client fetch deeply related objects in one request instead of chaining several REST calls.
- **Multiple client types with differing needs**: Each client can request only the fields it needs from a shared schema, avoiding endpoint proliferation.
- **Rapidly evolving frontend requirements**: Fields can be added to the schema without versioning, and clients adopt them without breaking existing queries.
