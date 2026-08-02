---
title: "CNN vs RNN: Spatial Convolution vs Sequential Recurrence"
date: 2026-08-03T03:31:06.279580+09:00
tags: ["neural-networks", "deep-learning", "cnn", "rnn"]
---
## Overview

Convolutional Neural Networks (CNNs) and Recurrent Neural Networks (RNNs) are neural architectures built for different data shapes: CNNs slide <strong class="kw">convolutional filters</strong> across a spatial grid to detect local patterns, while RNNs pass a <strong class="kw">recurrent hidden state</strong> across time steps to model sequential dependencies. Choosing between them (or their modern successors) depends on whether your data's structure is spatial, temporal, or both.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><defs><marker id="arrowA" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-a)"/></marker><marker id="arrowB" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto"><path d="M0,0 L8,4 L0,8 Z" style="fill:var(--compare-b)"/></marker></defs><line x1="320" y1="65" x2="320" y2="310" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4,4"/><text x="150" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">CNN</text><text x="150" y="52" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Convolution over a spatial grid</text><rect x="52" y="90" width="104" height="104" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="78" y1="90" x2="78" y2="194" style="stroke:var(--border)" stroke-width="1"/><line x1="104" y1="90" x2="104" y2="194" style="stroke:var(--border)" stroke-width="1"/><line x1="130" y1="90" x2="130" y2="194" style="stroke:var(--border)" stroke-width="1"/><line x1="52" y1="116" x2="156" y2="116" style="stroke:var(--border)" stroke-width="1"/><line x1="52" y1="142" x2="156" y2="142" style="stroke:var(--border)" stroke-width="1"/><line x1="52" y1="168" x2="156" y2="168" style="stroke:var(--border)" stroke-width="1"/><rect x="52" y="90" width="52" height="52" style="fill:none;stroke:var(--compare-a)" stroke-width="3"/><line x1="108" y1="116" x2="123" y2="116" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><line x1="160" y1="142" x2="191" y2="142" style="stroke:var(--compare-a)" stroke-width="2" marker-end="url(#arrowA)"/><text x="176" y="132" text-anchor="middle" style="fill:var(--secondary)" font-size="10">convolve</text><rect x="196" y="112" width="54" height="54" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><line x1="214" y1="112" x2="214" y2="166" style="stroke:var(--border)" stroke-width="1"/><line x1="232" y1="112" x2="232" y2="166" style="stroke:var(--border)" stroke-width="1"/><line x1="196" y1="130" x2="250" y2="130" style="stroke:var(--border)" stroke-width="1"/><line x1="196" y1="148" x2="250" y2="148" style="stroke:var(--border)" stroke-width="1"/><text x="150" y="215" text-anchor="middle" style="fill:var(--secondary)" font-size="10.5">Filter weights shared across all positions</text><text x="150" y="229" text-anchor="middle" style="fill:var(--secondary)" font-size="10.5">Captures local spatial patterns</text><text x="150" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Best for grid-structured data (images)</text><text x="460" y="32" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">RNN</text><text x="460" y="52" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Recurrence over a sequence</text><rect x="360" y="130" width="40" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="440" y="130" width="40" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><rect x="520" y="130" width="40" height="40" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="380" y="155" text-anchor="middle" style="fill:var(--content)" font-size="14">h1</text><text x="460" y="155" text-anchor="middle" style="fill:var(--content)" font-size="14">h2</text><text x="540" y="155" text-anchor="middle" style="fill:var(--content)" font-size="14">h3</text><line x1="400" y1="150" x2="439" y2="150" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="480" y1="150" x2="519" y2="150" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="380" y1="214" x2="380" y2="171" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="460" y1="214" x2="460" y2="171" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="540" y1="214" x2="540" y2="171" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="380" y="228" text-anchor="middle" style="fill:var(--content)" font-size="12">x1</text><text x="460" y="228" text-anchor="middle" style="fill:var(--content)" font-size="12">x2</text><text x="540" y="228" text-anchor="middle" style="fill:var(--content)" font-size="12">x3</text><line x1="380" y1="129" x2="380" y2="87" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="460" y1="129" x2="460" y2="87" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><line x1="540" y1="129" x2="540" y2="87" style="stroke:var(--compare-b)" stroke-width="2" marker-end="url(#arrowB)"/><text x="380" y="78" text-anchor="middle" style="fill:var(--content)" font-size="12">y1</text><text x="460" y="78" text-anchor="middle" style="fill:var(--content)" font-size="12">y2</text><text x="540" y="78" text-anchor="middle" style="fill:var(--content)" font-size="12">y3</text><text x="460" y="246" text-anchor="middle" style="fill:var(--secondary)" font-size="10.5">Hidden state carries context</text><text x="460" y="260" text-anchor="middle" style="fill:var(--secondary)" font-size="10.5">forward through the sequence</text><text x="460" y="300" text-anchor="middle" style="fill:var(--secondary)" font-size="11">Best for sequential/time-ordered data</text></svg>
</div>

## Comparison Table

| Aspect | CNN | RNN |
| --- | --- | --- |
| Input data shape | Fixed-size spatial grid (2D/3D tensors like images) | Variable-length ordered sequence (text, time series, audio) |
| Core operation | Convolution: a filter slides over local receptive fields | Recurrence: hidden state updated step-by-step from previous state plus current input |
| Weight sharing | Same filter weights reused across all spatial positions | Same weight matrices reused across all time steps |
| Context captured | Local spatial neighborhoods, expanded via depth/pooling | Temporal history accumulated in the hidden state over prior steps |
| Order sensitivity | Largely order-invariant beyond local structure; pooling discards exact position | Strictly order-dependent; reordering the sequence changes the output |
| Training parallelization | Highly parallelizable across positions, channels, and layers | Inherently sequential; each step waits on the previous hidden state |
| Common failure mode | Limited receptive field unless network is deep or uses dilation | Vanishing/exploding gradients over long sequences |
| Typical applications | Image classification, object detection, segmentation | Language modeling, time-series forecasting, speech recognition |

## Key Differences

- CNNs assume spatially local structure and share <strong class="kw">filter weights</strong> across the whole input; RNNs share weights across time steps instead.
- CNN layers process all positions in parallel, while RNNs are <strong class="kw">sequential</strong> by construction since each step needs the prior hidden state.
- RNNs suffer from <strong class="kw">vanishing gradients</strong> over long sequences; CNNs sidestep this but need deeper stacks to grow their receptive field.
- Shuffling pixels barely changes what a CNN detects, but reordering a sequence fed to an RNN changes the output entirely, since RNNs are <strong class="kw">order-sensitive</strong>.
- CNNs expect fixed-size grid inputs, whereas RNNs natively handle <strong class="kw">variable-length</strong> sequences.

## When to Use Each

**CNN**

- **Image Classification & Detection**: CNNs exploit spatial locality and translation invariance, making them efficient at recognizing objects regardless of where they appear in the frame.
- **Fixed-Size Grid Data**: When input naturally forms a 2D or 3D grid (images, spectrograms, board states), convolution captures local structure efficiently.
- **Large-Scale Parallel Training**: Because convolutions don't depend on previous outputs, CNNs train fast across large batches on GPUs/TPUs.

**RNN**

- **Variable-Length Sequences**: RNNs consume sequences of arbitrary length directly, without padding to a fixed grid like a CNN would need.
- **Online/Streaming Prediction**: RNNs update a compact hidden state incrementally, suiting real-time processing where inputs arrive one step at a time.
- **Lightweight Sequence Modeling**: For small datasets or constrained compute, RNNs offer a simpler, lower-overhead alternative to attention-based models for order-dependent data.
