---
title: "Model Quantization vs Model Pruning: Fewer Bits vs Fewer Parameters"
date: 2026-08-03T03:48:01.679183+09:00
tags: ["machine-learning", "model-compression", "deep-learning", "inference-optimization"]
---
## Overview

Model quantization and model pruning are both techniques for shrinking neural networks and speeding up inference, but they compress different things. <strong class="kw">Quantization</strong> keeps every weight but represents each one with fewer bits (e.g. FP32 → INT8), while <strong class="kw">pruning</strong> keeps full precision but removes weights, neurons, or channels judged unimportant. The two techniques are complementary and are frequently chained together in a single compression pipeline.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="40" x2="320" y2="300" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="160" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Model Quantization</text><text x="160" y="50" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Before: FP32 (32-bit)</text><rect x="62" y="58" width="40" height="42" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="82" y="83" text-anchor="middle" style="fill:var(--content)" font-size="9">0.482</text><rect x="114" y="58" width="40" height="42" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="134" y="83" text-anchor="middle" style="fill:var(--content)" font-size="9">-1.037</text><rect x="166" y="58" width="40" height="42" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="186" y="83" text-anchor="middle" style="fill:var(--content)" font-size="9">0.917</text><rect x="218" y="58" width="40" height="42" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="238" y="83" text-anchor="middle" style="fill:var(--content)" font-size="9">-0.203</text><line x1="160" y1="104" x2="160" y2="132" style="stroke:var(--secondary)" stroke-width="2"/><polygon points="153,132 167,132 160,142" style="fill:var(--secondary)"/><text x="172" y="122" style="fill:var(--secondary)" font-size="10">quantize</text><text x="160" y="158" text-anchor="middle" style="fill:var(--secondary)" font-size="11">After: INT8 (8-bit)</text><rect x="69" y="166" width="26" height="26" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="82" y="183" text-anchor="middle" style="fill:var(--content)" font-size="9">61</text><rect x="121" y="166" width="26" height="26" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="134" y="183" text-anchor="middle" style="fill:var(--content)" font-size="9">-132</text><rect x="173" y="166" width="26" height="26" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="186" y="183" text-anchor="middle" style="fill:var(--content)" font-size="9">117</text><rect x="225" y="166" width="26" height="26" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="238" y="183" text-anchor="middle" style="fill:var(--content)" font-size="9">-26</text><text x="160" y="220" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Same 4 values, fewer bits each</text><text x="160" y="236" text-anchor="middle" style="fill:var(--secondary)" font-size="11">≈4× smaller, faster math</text><text x="480" y="28" text-anchor="middle" style="fill:var(--primary)" font-size="16" font-weight="bold">Model Pruning</text><text x="480" y="50" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Before: dense network</text><g style="stroke:var(--compare-b)" stroke-width="1" opacity="0.7"><line x1="420" y1="65" x2="480" y2="55"/><line x1="420" y1="65" x2="480" y2="87"/><line x1="420" y1="65" x2="480" y2="120"/><line x1="420" y1="110" x2="480" y2="55"/><line x1="420" y1="110" x2="480" y2="87"/><line x1="420" y1="110" x2="480" y2="120"/><line x1="480" y1="55" x2="540" y2="65"/><line x1="480" y1="55" x2="540" y2="110"/><line x1="480" y1="87" x2="540" y2="65"/><line x1="480" y1="87" x2="540" y2="110"/><line x1="480" y1="120" x2="540" y2="65"/><line x1="480" y1="120" x2="540" y2="110"/></g><g style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"><circle cx="420" cy="65" r="5"/><circle cx="420" cy="110" r="5"/><circle cx="480" cy="55" r="5"/><circle cx="480" cy="87" r="5"/><circle cx="480" cy="120" r="5"/><circle cx="540" cy="65" r="5"/><circle cx="540" cy="110" r="5"/></g><line x1="480" y1="138" x2="480" y2="166" style="stroke:var(--secondary)" stroke-width="2"/><polygon points="473,166 487,166 480,176" style="fill:var(--secondary)"/><text x="492" y="156" style="fill:var(--secondary)" font-size="10">prune</text><text x="480" y="190" text-anchor="middle" style="fill:var(--secondary)" font-size="11">After: sparse (pruned)</text><line x1="420" y1="240" x2="480" y2="185" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><line x1="480" y1="185" x2="540" y2="240" style="stroke:var(--border)" stroke-width="1" stroke-dasharray="3,3"/><g style="stroke:var(--compare-b)" stroke-width="1.5"><line x1="420" y1="195" x2="480" y2="185"/><line x1="420" y1="195" x2="480" y2="217"/><line x1="420" y1="240" x2="480" y2="217"/><line x1="480" y1="185" x2="540" y2="195"/><line x1="480" y1="217" x2="540" y2="195"/><line x1="480" y1="217" x2="540" y2="240"/></g><circle cx="480" cy="250" r="5" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="2,2"/><g style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"><circle cx="420" cy="195" r="5"/><circle cx="420" cy="240" r="5"/><circle cx="480" cy="185" r="5"/><circle cx="480" cy="217" r="5"/><circle cx="540" cy="195" r="5"/><circle cx="540" cy="240" r="5"/></g><text x="480" y="277" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Fewer neurons &amp; connections</text><line x1="380" y1="300" x2="400" y2="300" style="stroke:var(--compare-b)" stroke-width="2"/><text x="405" y="304" style="fill:var(--secondary)" font-size="10">kept</text><line x1="450" y1="300" x2="470" y2="300" style="stroke:var(--border)" stroke-width="2" stroke-dasharray="3,3"/><text x="475" y="304" style="fill:var(--secondary)" font-size="10">removed</text></svg>
</div>

## Comparison Table

| Aspect | Model Quantization | Model Pruning |
| --- | --- | --- |
| Core mechanism | Reduces numeric precision of weights/activations (e.g. FP32 → INT8/INT4) | Removes individual weights, neurons, or channels judged low-importance |
| What changes | Same parameter count, smaller representation per value | Fewer parameters; model becomes sparse or physically smaller |
| Granularity | Per-tensor, per-channel, or per-group bit-width choices | Unstructured (single weights) vs structured (filters/channels/layers) |
| When applied | Post-training quantization (PTQ) or quantization-aware training (QAT) | Iterative pruning during training or magnitude-based pruning after training, usually with fine-tuning |
| Hardware/runtime requirement | Needs low-precision kernel support (INT8 cores, TensorRT, XNNPACK) | Unstructured pruning needs sparse-matrix kernels for real speedup; structured pruning runs on standard dense hardware |
| Compression achieved | Typically 2-4x size reduction (FP32→INT8); INT4 pushes further at higher accuracy risk | Can reach 50-90% sparsity, but unstructured sparsity often doesn't translate to real speedup without special hardware |
| Accuracy impact & recovery | Small accuracy drop, usually recovered via calibration or QAT | Larger accuracy drop at high sparsity, recovered via iterative fine-tuning/retraining |
| Combinability | Often applied last, to shrink an already-pruned model further | Often applied first, before the pruned model is quantized |

## Key Differences

- Quantization changes each value's <strong class="kw">bit-width</strong>; pruning changes the model's <strong class="kw">parameter count</strong>.
- Realizing pruning's theoretical speedup often requires <strong class="kw">sparse kernels</strong>, while quantization's speedup comes from standard <strong class="kw">INT8 hardware</strong> support.
- Quantization degrades accuracy gradually and predictably; aggressive pruning risks a <strong class="kw">sharp accuracy cliff</strong> without fine-tuning.
- The two are commonly chained into a single <strong class="kw">compression pipeline</strong>, pruning first and quantizing the result.
- Structured pruning changes the model's <strong class="kw">architecture shape</strong>; quantization never touches the architecture.

## When to Use Each

**Model Quantization**

- **Deploying to edge/mobile hardware**: INT8 accelerators are ubiquitous on mobile and edge chips, giving guaranteed speedup with minimal engineering effort.
- **Shrinking model for storage/bandwidth**: Cuts file size roughly 2-4x without touching the architecture or parameter count.
- **Fast, low-risk compression**: Post-training quantization can be applied in minutes with a small calibration set and usually recovers most accuracy.

**Model Pruning**

- **Removing redundant capacity**: Overparameterized models with many near-zero weights can shed parameters with little accuracy loss.
- **Structured pruning for latency**: Removing whole channels or filters yields real speedup on standard hardware, unlike unstructured sparsity.
- **Model slimming for redeploys**: Iterative pruning can reveal a genuinely smaller architecture, useful when the same model is repeatedly retrained and shipped.
