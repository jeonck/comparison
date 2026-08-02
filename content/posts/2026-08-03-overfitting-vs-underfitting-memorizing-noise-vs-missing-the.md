---
title: "Overfitting vs Underfitting: Memorizing Noise vs Missing the Signal"
date: 2026-08-03T03:37:23.418968+09:00
tags: ["machine-learning", "model-training", "bias-variance", "overfitting"]
---
## Overview

Overfitting and underfitting describe the two ways a model can fail to generalize: one learns the training data too well, the other not well enough. Understanding which failure mode you're in determines whether you should simplify or add regularization, or instead increase capacity and train longer. <strong class="kw">Overfitting</strong> traps a model in the noise of its training set, while <strong class="kw">underfitting</strong> leaves it unable to capture the underlying pattern at all.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="20" font-weight="700" style="fill:var(--primary)">Overfitting</text><text x="480" y="30" text-anchor="middle" font-size="20" font-weight="700" style="fill:var(--primary)">Underfitting</text><line x1="40" y1="50" x2="40" y2="300" stroke-width="1.5" style="stroke:var(--border)"/><line x1="40" y1="300" x2="280" y2="300" stroke-width="1.5" style="stroke:var(--border)"/><line x1="360" y1="50" x2="360" y2="300" stroke-width="1.5" style="stroke:var(--border)"/><line x1="360" y1="300" x2="600" y2="300" stroke-width="1.5" style="stroke:var(--border)"/><path d="M60,260 L95,180 L130,220 L165,140 L200,190 L235,110 L262,150" fill="none" stroke-width="2.5" style="stroke:var(--compare-a)"/><circle cx="60" cy="260" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="95" cy="180" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="130" cy="220" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="165" cy="140" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="200" cy="190" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="235" cy="110" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="262" cy="150" r="5" stroke-width="2" style="fill:var(--compare-a-soft);stroke:var(--compare-a)"/><circle cx="360" cy="260" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="395" cy="180" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="430" cy="220" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="465" cy="140" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="500" cy="190" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="535" cy="110" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><circle cx="562" cy="150" r="5" stroke-width="2" style="fill:var(--compare-b-soft);stroke:var(--compare-b)"/><line x1="345" y1="225" x2="575" y2="150" stroke-width="2.5" style="stroke:var(--compare-b)"/><text x="160" y="325" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Fits every point exactly</text><text x="160" y="345" text-anchor="middle" font-size="13" style="fill:var(--secondary)">captures noise, not signal</text><text x="480" y="325" text-anchor="middle" font-size="13" style="fill:var(--secondary)">Misses the curved trend</text><text x="480" y="345" text-anchor="middle" font-size="13" style="fill:var(--secondary)">too simple for the pattern</text></svg>
</div>

## Comparison Table

| Aspect | Overfitting | Underfitting |
| --- | --- | --- |
| Underlying cause | Model too complex relative to the data, so it learns noise and idiosyncrasies | Model too simple to represent the true relationship in the data |
| Training error | Very low, often near zero | High, the model struggles even on data it was trained on |
| Validation/test error | High, much worse than training error | High, similar in magnitude to training error |
| Bias-variance profile | Low bias, high variance | High bias, low variance |
| Generalization to new data | Poor, predictions swing wildly on unseen inputs | Poor, predictions are consistently and systematically off |
| Learning curve signature | Training and validation loss diverge as training continues | Training and validation loss both plateau high and close together |
| Typical remedies | Regularization, more training data, dropout, early stopping, simpler model | Increase model capacity, add features, train longer, reduce regularization |

## Key Differences

- Overfitting memorizes <strong class="kw">noise</strong> in the training set, while underfitting never learns the underlying <strong class="kw">pattern</strong> at all.
- Overfitting shows near-zero training error but a wide train/validation gap, a sign of <strong class="kw">high variance</strong>; underfitting shows poor performance on both, a sign of <strong class="kw">high bias</strong>.
- Overfitting is treated with <strong class="kw">regularization</strong> or more data; underfitting is treated by increasing <strong class="kw">model capacity</strong>.
- Overfitting gets worse the longer an overly flexible model <strong class="kw">keeps training</strong>; underfitting persists regardless of duration, since it's a <strong class="kw">structural limit</strong>.

## When to Use Each

**Overfitting**

- **Complex Model, Small Dataset**: A high-capacity model like a deep neural net or high-degree polynomial trained on a small dataset will readily memorize training examples instead of generalizing.
- **Perfect Training Accuracy**: If training accuracy is near 100% but validation accuracy is much lower, the model has likely learned noise specific to the training set.
- **Too Many Training Epochs**: Validation loss rising while training loss keeps falling is a classic sign the model is overfitting to the training data.

**Underfitting**

- **Oversimplified Model**: Using a linear model to fit a clearly nonlinear relationship leaves the model unable to capture the true pattern.
- **Both Errors Stay High**: If training and validation accuracy are both low and close together, the model lacks the capacity to learn the data.
- **Excessive Regularization**: Applying too much L1/L2 penalty, dropout, or early stopping can prevent even a capable model from fitting the data well.
