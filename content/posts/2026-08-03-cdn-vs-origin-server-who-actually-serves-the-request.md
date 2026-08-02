---
title: "CDN vs Origin Server: Who Actually Serves the Request"
date: 2026-08-03T06:23:28.570826+09:00
tags: ["cdn", "origin-server", "caching", "networking"]
---
## Overview

A <strong class="kw">CDN</strong> is a distributed network of edge servers that caches and delivers content close to users, while the <strong class="kw">origin server</strong> is the single authoritative source where that content is created or stored. The distinction matters for latency, scalability, and resilience, since most requests should never need to reach the origin at all.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><circle cx="70" cy="200" r="24" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="70" y="205" text-anchor="middle" style="fill:var(--content)" font-size="11">Client</text><rect x="150" y="70" width="200" height="220" rx="8" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="52" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">CDN</text><text x="250" y="90" text-anchor="middle" style="fill:var(--content)" font-size="11">Distributed edge locations</text><circle cx="200" cy="140" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="300" cy="140" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="250" cy="200" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="200" cy="250" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="300" cy="250" r="16" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="278" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Cache: static &amp; edge-computed content</text><rect x="460" y="150" width="140" height="90" rx="8" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="530" y="133" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Origin Server</text><text x="530" y="192" text-anchor="middle" style="fill:var(--content)" font-size="11">Single source</text><text x="530" y="207" text-anchor="middle" style="fill:var(--content)" font-size="11">of truth</text><line x1="95" y1="190" x2="148" y2="165" style="stroke:var(--content)" stroke-width="1.5"/><polygon points="148,165 139,163 143,171" style="fill:var(--content)"/><text x="115" y="150" text-anchor="middle" style="fill:var(--secondary)" font-size="9">request</text><line x1="148" y1="215" x2="95" y2="210" style="stroke:var(--compare-a)" stroke-width="1.5"/><polygon points="95,210 104,206 104,215" style="fill:var(--compare-a)"/><text x="118" y="235" text-anchor="middle" style="fill:var(--secondary)" font-size="9">cached response</text><line x1="352" y1="180" x2="458" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="5,4"/><polygon points="458,185 449,181 450,190" style="fill:var(--compare-b)"/><text x="405" y="165" text-anchor="middle" style="fill:var(--secondary)" font-size="9">on cache miss</text><line x1="458" y1="215" x2="352" y2="210" style="stroke:var(--compare-b)" stroke-width="1.5" stroke-dasharray="5,4"/><polygon points="352,210 361,206 361,215" style="fill:var(--compare-b)"/><text x="405" y="233" text-anchor="middle" style="fill:var(--secondary)" font-size="9">origin pull &amp; cache</text><text x="320" y="330" text-anchor="middle" style="fill:var(--secondary)" font-size="10">Most requests resolve at the edge; only misses/dynamic content reach the origin</text></svg>
</div>

## Comparison Table

| Aspect | CDN | Origin Server |
| --- | --- | --- |
| Role in request path | Intercepts requests at the edge and serves cached content directly | Authoritative backend that generates or stores the original content |
| Geographic distribution | Many points of presence worldwide, close to end users | Typically one or a few fixed data center locations |
| Content served | Cached copies of static assets or cacheable API responses | Dynamically generated pages or master copies of files |
| Cache miss handling | Forwards uncached requests to the origin and stores the response per TTL | Processes every request that reaches it; has no caching layer of its own |
| Latency | Low, since content is served from the nearest edge node | Higher, since every request travels to one fixed location |
| Load on backend | Absorbs most traffic, shielding the origin from direct load | Only handles cache misses and non-cacheable requests |
| Resilience to outages | Can keep serving stale cached content if the origin goes down | Site is effectively unavailable for anything not already cached elsewhere |
| Scaling and cost | Scales via the CDN provider's global network, billed per bandwidth/requests | Must be scaled and provisioned directly, billed for compute and hosting |

## Key Differences

- CDN content lives on distributed <strong class="kw">edge nodes</strong>; the origin server is the single <strong class="kw">source of truth</strong>.
- CDN caching cuts <strong class="kw">latency</strong> for users, but every <strong class="kw">cache miss</strong> still lands on the origin.
- CDN edge caching gives resilience against <strong class="kw">origin outages</strong> by serving stale content.
- Origin servers own <strong class="kw">dynamic content</strong> generation; CDNs only store what's actually <strong class="kw">cacheable</strong>.

## When to Use Each

**CDN**

- **Global static asset delivery**: Images, JS, and CSS reach worldwide users fastest when served from a nearby edge node instead of a single origin.
- **Traffic spike absorption**: The CDN's edge network absorbs sudden surges in requests before they ever reach and overload the origin.
- **Video and streaming distribution**: Large media files benefit heavily from edge caching, cutting both latency and origin bandwidth costs.

**Origin Server**

- **Dynamic personalized responses**: User-specific or real-time computed content can't be cached, so it must be generated at the origin on every request.
- **State-changing API calls**: Writes like POST or PUT mutate application state and must always reach the origin, not a cache.
- **Small low-traffic applications**: For limited or internal audiences, the added complexity and cost of a CDN layer often isn't justified.
