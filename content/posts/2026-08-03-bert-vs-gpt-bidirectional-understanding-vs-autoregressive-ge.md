---
title: "BERT vs GPT: Bidirectional Understanding vs Autoregressive Generation"
date: 2026-08-03T03:42:01.486518+09:00
tags: ["nlp", "transformers", "language-models", "machine-learning"]
---
## Overview

BERT and GPT are both transformer-based language models, but they're built from opposite halves of the transformer and trained for opposite jobs. BERT uses an <strong class="kw">encoder</strong> trained to fill in masked words using context from both directions, making it suited to understanding text, while GPT uses a <strong class="kw">decoder</strong> trained to predict the next word from only what came before, making it suited to generating text.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrowA" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M0,0L10,5L0,10z" style="fill:var(--compare-a)"/>
    </marker>
    <marker id="arrowB" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M0,0L10,5L0,10z" style="fill:var(--compare-b)"/>
    </marker>
  </defs>
  <line x1="320" y1="20" x2="320" y2="300" style="stroke:var(--border)" stroke-width="1.5" stroke-dasharray="4 4"/>
  <text x="160" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="22" font-weight="bold">BERT</text>
  <text x="480" y="36" text-anchor="middle" style="fill:var(--primary)" font-size="22" font-weight="bold">GPT</text>
  <text x="160" y="56" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Bidirectional Encoder</text>
  <text x="480" y="56" text-anchor="middle" style="fill:var(--secondary)" font-size="12">Autoregressive Decoder</text>
  <path d="M50,153 Q100,85 150,153" style="stroke:var(--compare-a);fill:none" stroke-width="1.3" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/>
  <path d="M100,153 Q125,120 150,153" style="stroke:var(--compare-a);fill:none" stroke-width="1.3" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/>
  <path d="M150,153 Q175,120 200,153" style="stroke:var(--compare-a);fill:none" stroke-width="1.3" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/>
  <path d="M150,153 Q200,85 250,153" style="stroke:var(--compare-a);fill:none" stroke-width="1.3" marker-start="url(#arrowA)" marker-end="url(#arrowA)"/>
  <rect x="30" y="155" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="50" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">the</text>
  <rect x="80" y="155" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="100" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">cat</text>
  <rect x="130" y="155" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2" stroke-dasharray="3 2"/>
  <text x="150" y="173" text-anchor="middle" style="fill:var(--primary)" font-size="9" font-weight="bold">[MASK]</text>
  <rect x="180" y="155" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="200" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">on</text>
  <rect x="230" y="155" width="40" height="30" rx="4" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/>
  <text x="250" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">mat</text>
  <text x="160" y="222" text-anchor="middle" style="fill:var(--secondary)" font-size="11">sees full sentence context</text>
  <text x="160" y="237" text-anchor="middle" style="fill:var(--secondary)" font-size="11">(left + right) to fill the mask</text>
  <text x="160" y="270" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">-&gt; classification, embeddings, NER</text>
  <path d="M370,153 Q460,90 550,153" style="stroke:var(--compare-b);fill:none" stroke-width="1.3" marker-end="url(#arrowB)"/>
  <path d="M415,153 Q482,110 550,153" style="stroke:var(--compare-b);fill:none" stroke-width="1.3" marker-end="url(#arrowB)"/>
  <path d="M460,153 Q505,125 550,153" style="stroke:var(--compare-b);fill:none" stroke-width="1.3" marker-end="url(#arrowB)"/>
  <path d="M505,153 Q527,135 550,153" style="stroke:var(--compare-b);fill:none" stroke-width="1.3" marker-end="url(#arrowB)"/>
  <path d="M570,170 L574,170" style="stroke:var(--compare-b);fill:none" stroke-width="1.5" marker-end="url(#arrowB)"/>
  <rect x="350" y="155" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="370" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">the</text>
  <rect x="395" y="155" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="415" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">cat</text>
  <rect x="440" y="155" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="460" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">sat</text>
  <rect x="485" y="155" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/>
  <text x="505" y="174" text-anchor="middle" style="fill:var(--content)" font-size="11">on</text>
  <rect x="530" y="155" width="40" height="30" rx="4" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/>
  <text x="550" y="174" text-anchor="middle" style="fill:var(--primary)" font-size="11" font-weight="bold">mat</text>
  <rect x="575" y="155" width="34" height="30" rx="4" style="fill:none;stroke:var(--border)" stroke-width="1.5" stroke-dasharray="3 2"/>
  <text x="592" y="174" text-anchor="middle" style="fill:var(--secondary)" font-size="12">?</text>
  <text x="480" y="222" text-anchor="middle" style="fill:var(--secondary)" font-size="11">each token sees only itself</text>
  <text x="480" y="237" text-anchor="middle" style="fill:var(--secondary)" font-size="11">+ prior tokens (causal mask)</text>
  <text x="480" y="270" text-anchor="middle" style="fill:var(--content)" font-size="12" font-weight="bold">-&gt; generation, chat, completion</text>
</svg>
</div>

## Comparison Table

| Aspect | BERT | GPT |
| --- | --- | --- |
| Architecture | Encoder-only transformer stack | Decoder-only transformer stack |
| Pretraining objective | Masked language modeling: predict randomly hidden tokens, plus next-sentence prediction | Causal language modeling: predict the next token given all prior tokens |
| Attention pattern | Bidirectional self-attention; every token attends to the full sequence | Causal (masked) self-attention; each token attends only to itself and earlier tokens |
| Output generation | One contextual embedding per input token, produced in a single forward pass | Text generated autoregressively, one token at a time, each output fed back as input |
| Typical adaptation | Fine-tuned with a task-specific head on top of the pretrained encoder | Adapted via prompting, instruction tuning, or fine-tuning to continue text |
| Primary use cases | Classification, named entity recognition, semantic search, sentence embeddings | Open-ended generation, chat, code completion, summarization |
| Inference cost per query | Fixed: one pass regardless of desired output | Scales with number of generated tokens, each requiring a forward pass |

## Key Differences

- BERT's <strong class="kw">encoder</strong> attends to both left and right context; GPT's <strong class="kw">decoder</strong> attends only to prior tokens.
- BERT trains on <strong class="kw">masked language modeling</strong>; GPT trains on <strong class="kw">next-token prediction</strong>.
- BERT produces embeddings in a <strong class="kw">single pass</strong>; GPT produces text through <strong class="kw">autoregressive decoding</strong>.
- BERT is optimized for <strong class="kw">understanding tasks</strong>; GPT is optimized for <strong class="kw">generation tasks</strong>.

## When to Use Each

**BERT**

- **Semantic Search & Embeddings**: BERT's bidirectional context produces dense sentence/document vectors well suited to similarity search and retrieval.
- **Classification & NER**: A lightweight task head fine-tuned on top of BERT's embeddings handles sentiment, intent, or entity extraction efficiently.
- **Extractive Question Answering**: Seeing the full passage in both directions helps BERT locate the exact answer span within a reference text.

**GPT**

- **Open-Ended Text Generation**: GPT's autoregressive design is built to produce novel, coherent multi-sentence text like drafts, code, or articles.
- **Conversational Agents**: Sequential next-token prediction naturally extends to multi-turn dialogue and chat-style interaction.
- **Few-Shot & Zero-Shot Prompting**: Large GPT models can adapt to new tasks purely through prompting, without task-specific fine-tuning.
