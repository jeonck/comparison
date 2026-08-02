---
title: "Zero-Shot Learning vs Few-Shot Learning: No Examples vs a Handful of Examples"
date: 2026-08-03T03:45:54.343154+09:00
tags: ["machine-learning", "nlp", "prompt-engineering", "llm"]
---
## Overview

Zero-shot and few-shot learning describe how much task-specific example data a model is given before it has to perform a task. Zero-shot relies solely on a <strong class="kw">task description</strong>, while few-shot conditions its predictions on a small set of <strong class="kw">labeled examples</strong>, usually trading a little setup cost for higher accuracy.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="160" y="30" text-anchor="middle" font-size="16" font-weight="600" style="fill:var(--primary)">Zero-Shot</text><text x="480" y="30" text-anchor="middle" font-size="16" font-weight="600" style="fill:var(--primary)">Few-Shot</text><line x1="320" y1="45" x2="320" y2="335" stroke-dasharray="4,4" style="stroke:var(--border)" stroke-width="1.5"/><rect x="60" y="55" width="200" height="95" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="85" text-anchor="middle" font-size="13" style="fill:var(--content)">Task instruction</text><text x="160" y="103" text-anchor="middle" font-size="13" style="fill:var(--content)">only</text><text x="160" y="128" text-anchor="middle" font-size="11" style="fill:var(--secondary)">(0 examples)</text><rect x="380" y="55" width="200" height="95" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="78" text-anchor="middle" font-size="13" style="fill:var(--content)">Task instruction</text><text x="480" y="96" text-anchor="middle" font-size="13" style="fill:var(--content)">+ K examples</text><rect x="400" y="108" width="25" height="22" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><text x="412" y="123" text-anchor="middle" font-size="10" style="fill:var(--content)">1</text><rect x="430" y="108" width="25" height="22" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><text x="442" y="123" text-anchor="middle" font-size="10" style="fill:var(--content)">2</text><rect x="460" y="108" width="25" height="22" rx="3" style="fill:none;stroke:var(--compare-b)" stroke-width="1"/><text x="472" y="123" text-anchor="middle" font-size="10" style="fill:var(--content)">3</text><line x1="160" y1="150" x2="160" y2="183" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="160,185 155,177 165,177" style="fill:var(--secondary)"/><line x1="480" y1="150" x2="480" y2="183" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,185 475,177 485,177" style="fill:var(--secondary)"/><rect x="60" y="185" width="200" height="50" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="160" y="206" text-anchor="middle" font-size="12" style="fill:var(--content)">Pretrained</text><text x="160" y="222" text-anchor="middle" font-size="12" style="fill:var(--content)">Model</text><rect x="380" y="185" width="200" height="50" rx="6" style="fill:none;stroke:var(--border)" stroke-width="1.5"/><text x="480" y="206" text-anchor="middle" font-size="12" style="fill:var(--content)">Pretrained</text><text x="480" y="222" text-anchor="middle" font-size="12" style="fill:var(--content)">Model</text><line x1="160" y1="235" x2="160" y2="268" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="160,270 155,262 165,262" style="fill:var(--secondary)"/><line x1="480" y1="235" x2="480" y2="268" style="stroke:var(--secondary)" stroke-width="1.5"/><polygon points="480,270 475,262 485,262" style="fill:var(--secondary)"/><rect x="60" y="270" width="200" height="55" rx="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="160" y="302" text-anchor="middle" font-size="13" style="fill:var(--content)">Prediction</text><rect x="380" y="270" width="200" height="55" rx="6" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><text x="480" y="302" text-anchor="middle" font-size="13" style="fill:var(--content)">Prediction</text><text x="160" y="350" text-anchor="middle" font-size="10" style="fill:var(--secondary)">no task-specific data</text><text x="480" y="350" text-anchor="middle" font-size="10" style="fill:var(--secondary)">learns from few examples</text></svg>
</div>

## Comparison Table

| Aspect | Zero-Shot Learning | Few-Shot Learning |
| --- | --- | --- |
| Core definition | Model performs a task it was never explicitly shown examples for, guided only by natural-language instructions or class descriptions | Model performs a task after being shown a small number (typically 1-100) of labeled examples at inference or fine-tuning time |
| Examples provided at inference | None — only a task description or prompt | A handful of input-output pairs included in the prompt or used for fine-tuning |
| Underlying mechanism | Relies entirely on knowledge encoded during pretraining plus semantic alignment between labels and text | Uses in-context learning or lightweight fine-tuning to infer the task pattern directly from the provided examples |
| Labeling/data cost | Effectively zero — no labeled data needed for the target task | Low but nonzero — requires curating a small, representative set of examples |
| Prompt/context length | Short — just the instruction or class names | Longer — instruction plus example pairs, consuming more context tokens |
| Typical accuracy | Lower and more variable, especially on niche or ambiguous tasks | Generally higher and more stable since examples disambiguate intent |
| Sensitivity to example choice | Not applicable — there are no examples to choose | High — accuracy can swing significantly with example selection, order, and count |
| Common techniques | Prompt engineering, CLIP-style embedding matching, instruction-tuned LLMs | Few-shot prompting, meta-learning (e.g. MAML), lightweight fine-tuning or LoRA |

## Key Differences

- Zero-shot uses no <strong class="kw">task examples</strong> at all, relying purely on pretrained knowledge and instructions
- Few-shot conditions the model on a small <strong class="kw">support set</strong> of labeled examples at inference time
- Few-shot generally achieves higher <strong class="kw">accuracy</strong> because examples disambiguate an otherwise vague instruction
- Zero-shot has zero <strong class="kw">labeling cost</strong>, while few-shot requires curating representative examples
- Few-shot performance is sensitive to <strong class="kw">example selection</strong>, a variable that zero-shot simply doesn't have

## When to Use Each

**Zero-Shot Learning**

- **Rapid prototyping**: Test a task idea instantly without collecting or labeling any examples.
- **No labeled data available**: The domain has no existing examples to draw from, such as a brand-new product category.
- **Simple, well-known tasks**: The task closely matches something the model already learned during pretraining, like basic sentiment polarity.

**Few-Shot Learning**

- **Ambiguous or niche tasks**: A short instruction alone leaves room for misinterpretation, and examples pin down the exact intent or format.
- **Format-sensitive output**: You need consistent structure, like an exact JSON schema, that's easier to demonstrate than to describe in words.
- **Small labeled dataset exists**: You have a handful of good examples but not enough data to justify full fine-tuning.
