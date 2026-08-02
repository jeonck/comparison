---
title: "LSTM vs GRU: Three Gates vs Two Gates in Recurrent Memory"
date: 2026-08-03T03:33:36.323898+09:00
tags: ["deep-learning", "rnn", "neural-networks", "sequence-modeling"]
---
## Overview

LSTM and GRU are both gated recurrent architectures built to capture long-range dependencies in sequences while avoiding the vanishing-gradient problem of vanilla RNNs. LSTM keeps a dedicated <strong class="kw">cell state</strong> alongside its hidden state, regulated by three gates, while GRU folds everything into a single <strong class="kw">hidden state</strong> updated by just two gates. That structural difference drives everything else: parameter count, training speed, and how precisely you can control what the network remembers.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="165" y="28" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">LSTM</text><text x="475" y="28" text-anchor="middle" font-size="20" font-weight="bold" style="fill:var(--primary)">GRU</text><rect x="30" y="50" width="270" height="280" rx="12" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><rect x="340" y="50" width="270" height="280" rx="12" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/><line x1="30" y1="90" x2="300" y2="90" style="stroke:var(--compare-a)" stroke-width="2.5"/><polygon points="300,90 289,85 289,95" style="fill:var(--compare-a)"/><text x="35" y="78" font-size="11" style="fill:var(--secondary)">Ct-1</text><text x="295" y="78" text-anchor="end" font-size="11" style="fill:var(--secondary)">Ct</text><circle cx="80" cy="90" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="80" y="95" text-anchor="middle" font-size="14" style="fill:var(--content)">&#215;</text><circle cx="165" cy="90" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="165" y="95" text-anchor="middle" font-size="14" style="fill:var(--content)">+</text><rect x="60" y="145" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="80" y="165" text-anchor="middle" font-size="14" style="fill:var(--content)">f</text><rect x="145" y="145" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="165" y="165" text-anchor="middle" font-size="14" style="fill:var(--content)">i</text><rect x="230" y="145" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="165" text-anchor="middle" font-size="14" style="fill:var(--content)">o</text><line x1="80" y1="145" x2="80" y2="102" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="165" y1="145" x2="165" y2="102" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="250" y1="175" x2="250" y2="268" style="stroke:var(--compare-a)" stroke-width="1.5"/><line x1="30" y1="280" x2="300" y2="280" style="stroke:var(--compare-a)" stroke-width="2.5"/><polygon points="300,280 289,275 289,285" style="fill:var(--compare-a)"/><text x="35" y="270" font-size="11" style="fill:var(--secondary)">ht-1</text><text x="295" y="270" text-anchor="end" font-size="11" style="fill:var(--secondary)">ht</text><circle cx="250" cy="280" r="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="250" y="285" text-anchor="middle" font-size="14" style="fill:var(--content)">&#215;</text><text x="165" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">f = forget   i = input   o = output</text><line x1="340" y1="185" x2="610" y2="185" style="stroke:var(--compare-b)" stroke-width="2.5"/><polygon points="610,185 599,180 599,190" style="fill:var(--compare-b)"/><text x="345" y="175" font-size="11" style="fill:var(--secondary)">ht-1</text><text x="605" y="175" text-anchor="end" font-size="11" style="fill:var(--secondary)">ht</text><rect x="380" y="140" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="400" y="160" text-anchor="middle" font-size="14" style="fill:var(--content)">r</text><rect x="520" y="140" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="540" y="160" text-anchor="middle" font-size="14" style="fill:var(--content)">z</text><line x1="400" y1="170" x2="400" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="400" cy="185" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><line x1="540" y1="170" x2="540" y2="185" style="stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="540" cy="185" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="475" y="345" text-anchor="middle" font-size="11" style="fill:var(--secondary)">r = reset   z = update</text></svg>
</div>

## Comparison Table

| Aspect | LSTM | GRU |
| --- | --- | --- |
| Internal state | Separate cell state (Ct) and hidden state (ht) | Single hidden state (ht) that also serves as output |
| Gating mechanism | Three gates: forget, input, output | Two gates: reset, update |
| Weight matrices / parameters | Four sets of weights per unit, roughly 4x hidden_size^2 | Three sets of weights per unit, roughly 3x hidden_size^2 |
| How output is exposed | Output gate filters how much of the cell state becomes the hidden state | Update gate directly interpolates old and candidate hidden state |
| Training and inference cost | More matrix multiplications, slower per step | Fewer parameters, faster training and inference |
| Long-range memory control | Explicit forget gate gives fine-grained control over long-term retention | Reset gate can drop past context more abruptly, sometimes weaker on very long sequences |
| Typical empirical results | Matches or slightly outperforms on complex, long-sequence tasks | Comparable accuracy with less data and fewer epochs |
| Common use today | Large datasets, tasks needing precise long-term memory control | Resource-constrained or smaller-data settings, faster iteration |

## Key Differences

- LSTM separates memory into a dedicated <strong class="kw">cell state</strong> plus hidden state, giving finer control over what persists.
- GRU collapses everything into one <strong class="kw">hidden state</strong> updated directly by its gates.
- LSTM's three gates (forget, input, output) require more parameters than GRU's <strong class="kw">two gates</strong> (reset, update).
- GRU trains and infers faster thanks to <strong class="kw">fewer parameters</strong>, useful for constrained hardware or smaller datasets.
- LSTM tends to edge out GRU on tasks needing very <strong class="kw">long-term dependencies</strong>, due to explicit forget-gate control.

## When to Use Each

**LSTM**

- **Long document or sequence modeling**: The dedicated cell state and forget gate give precise control over very long-range dependencies, such as long text or speech.
- **Large training datasets available**: The extra parameters can be fully utilized without overfitting when there's abundant labeled data.
- **Fine-grained memory control needed**: Separate input, forget, and output gates let you tune exactly how much old versus new information persists.

**GRU**

- **Limited training data**: Fewer parameters reduce overfitting risk when labeled examples are scarce.
- **Latency-constrained deployment**: Fewer gates and weight matrices mean faster inference on edge devices or mobile hardware.
- **Rapid experimentation**: Faster training loops let you iterate on architecture and hyperparameters more quickly.
