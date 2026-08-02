---
title: "Object Storage vs Block Storage: Data Model and Access Pattern"
date: 2026-08-03T06:22:17.364040+09:00
tags: ["object-storage", "block-storage", "cloud-storage", "infrastructure"]
---
## Overview

Block storage exposes raw, fixed-size <strong class="kw">disk blocks</strong> to a single attached server, just like a physical hard drive — ideal for databases and boot volumes needing low-latency random reads and writes. Object storage instead organizes data as whole items with <strong class="kw">rich metadata</strong> in a flat, HTTP-accessible namespace, trading fine-grained in-place edits for virtually unlimited scale. The choice determines whether your application talks to storage like a disk or like a web API.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="150" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Block Storage</text><text x="490" y="28" text-anchor="middle" font-size="16" font-weight="bold" style="fill:var(--primary)">Object Storage</text><rect x="30" y="60" width="90" height="40" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="75" y="84" text-anchor="middle" font-size="11" style="fill:var(--content)">VM / Server</text><line x1="75" y1="100" x2="75" y2="150" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="98" y="128" font-size="9" style="fill:var(--secondary)">iSCSI</text><rect x="20" y="150" width="240" height="150" rx="4" style="fill:none;stroke:var(--compare-a)" stroke-width="1.5"/><rect x="30" y="165" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="55" y="189" text-anchor="middle" font-size="10" style="fill:var(--content)">0</text><rect x="86" y="165" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="111" y="189" text-anchor="middle" font-size="10" style="fill:var(--content)">1</text><rect x="142" y="165" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="167" y="189" text-anchor="middle" font-size="10" style="fill:var(--content)">2</text><rect x="198" y="165" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="223" y="189" text-anchor="middle" font-size="10" style="fill:var(--content)">3</text><rect x="30" y="211" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="55" y="235" text-anchor="middle" font-size="10" style="fill:var(--content)">4</text><rect x="86" y="211" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="111" y="235" text-anchor="middle" font-size="10" style="fill:var(--content)">5</text><rect x="142" y="211" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="167" y="235" text-anchor="middle" font-size="10" style="fill:var(--content)">6</text><rect x="198" y="211" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="223" y="235" text-anchor="middle" font-size="10" style="fill:var(--content)">7</text><rect x="30" y="257" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="55" y="281" text-anchor="middle" font-size="10" style="fill:var(--content)">8</text><rect x="86" y="257" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="111" y="281" text-anchor="middle" font-size="10" style="fill:var(--content)">9</text><rect x="142" y="257" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="167" y="281" text-anchor="middle" font-size="10" style="fill:var(--content)">10</text><rect x="198" y="257" width="50" height="40" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1"/><text x="223" y="281" text-anchor="middle" font-size="10" style="fill:var(--content)">11</text><text x="140" y="320" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Blocks addressed by LBA, no inherent metadata</text><rect x="360" y="55" width="60" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="74" text-anchor="middle" font-size="10" style="fill:var(--content)">App A</text><rect x="560" y="55" width="60" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="590" y="74" text-anchor="middle" font-size="10" style="fill:var(--content)">App B</text><line x1="390" y1="85" x2="450" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><line x1="590" y1="85" x2="530" y2="150" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="112" text-anchor="middle" font-size="9" style="fill:var(--secondary)">HTTP / REST</text><rect x="360" y="150" width="260" height="150" rx="4" stroke-dasharray="4,3" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="490" y="142" text-anchor="middle" font-size="9" style="fill:var(--secondary)">bucket (flat namespace)</text><rect x="375" y="165" width="90" height="40" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="420" y="189" text-anchor="middle" font-size="9" style="fill:var(--content)">photo.jpg</text><rect x="475" y="165" width="130" height="30" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="540" y="184" text-anchor="middle" font-size="9" style="fill:var(--content)">video.mp4</text><rect x="375" y="215" width="60" height="60" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="405" y="248" text-anchor="middle" font-size="9" style="fill:var(--content)">backup.tar</text><rect x="445" y="215" width="160" height="60" rx="3" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="525" y="248" text-anchor="middle" font-size="9" style="fill:var(--content)">dataset.csv</text><rect x="560" y="208" width="42" height="14" rx="3" style="fill:var(--compare-b);stroke:var(--compare-b)"/><text x="581" y="218" text-anchor="middle" font-size="7" style="fill:var(--content)">metadata</text><text x="490" y="320" text-anchor="middle" font-size="10" style="fill:var(--secondary)">Each object = data + key + custom metadata</text></svg>
</div>

## Comparison Table

| Aspect | Block Storage | Object Storage |
| --- | --- | --- |
| Unit of storage | Fixed-size blocks (e.g. 512B-4KB sectors) addressed by LBA | Whole objects (data + metadata) addressed by a unique key |
| Access protocol | Block-level protocols: iSCSI, Fibre Channel, NVMe | HTTP/HTTPS REST API (e.g. S3, Swift, Azure Blob) |
| Attachment model | Mounted as a raw device to one VM/host at a time | Accessed concurrently by any number of clients over the network |
| Update pattern | In-place partial writes to specific blocks/offsets | Whole-object replace; no partial in-place edits |
| Metadata support | Minimal - a filesystem layered on top supplies metadata | Rich, user-defined key-value metadata stored with each object |
| Namespace/hierarchy | Formatted with a filesystem (directories, inodes) | Flat namespace of buckets/containers, no true directories |
| Scalability | Limited by provisioned volume size; resize needed to grow | Virtually unlimited, scales horizontally without pre-provisioning |
| Typical consumers | Databases, boot disks, VM volumes needing low latency | Backups, media files, static assets, data lakes |

## Key Differences

- Block storage operates on raw <strong class="kw">disk blocks</strong>; object storage operates on whole <strong class="kw">objects</strong>.
- Block volumes attach to a single host via <strong class="kw">iSCSI</strong>; objects are reached by any client via <strong class="kw">HTTP</strong>.
- Block storage supports in-place <strong class="kw">partial writes</strong>; object storage requires full <strong class="kw">object replacement</strong>.
- Object storage carries custom <strong class="kw">metadata</strong> per item; block storage has none natively.
- Object storage scales in a flat <strong class="kw">namespace</strong> without pre-provisioning; block volumes must be resized manually.

## When to Use Each

**Block Storage**

- **Relational Database Volumes**: Databases need low-latency random reads/writes at fixed offsets, which only block storage provides.
- **Boot Disks for VMs**: Operating systems require a raw block device to format with a filesystem and boot from.
- **High-IOPS Transactional Workloads**: Latency-sensitive apps like OLTP systems benefit from direct block-level access without HTTP overhead.

**Object Storage**

- **Static Website Assets & Media**: Images, videos, and backups are written once and served whole, matching object storage's whole-object model.
- **Data Lakes and Analytics**: Massive, ever-growing datasets benefit from a flat namespace that scales without manual volume resizing.
- **Multi-Region Backup Archives**: Objects can be fetched by any client worldwide over HTTP without attaching a device to a specific host.
