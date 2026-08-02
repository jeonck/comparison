---
title: "Precision vs Recall: Predicted-Positive Accuracy vs Actual-Positive Coverage"
date: 2026-08-03T03:36:08.607886+09:00
tags: ["machine-learning", "classification-metrics", "model-evaluation", "data-science"]
---
## Overview

Precision and recall are two classification metrics computed from the same confusion matrix but answering different questions about a model's positive predictions. <strong class="kw">Precision</strong> asks how many predicted positives were correct, while <strong class="kw">recall</strong> asks how many actual positives were found. Optimizing one in isolation almost always trades off against the other.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><text x="180" y="40" text-anchor="middle" font-size="16" style="fill:var(--compare-a)">Predicted Positive</text><text x="460" y="40" text-anchor="middle" font-size="16" style="fill:var(--compare-b)">Actual Positive</text><circle cx="270" cy="165" r="95" fill-opacity="0.55" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="2"/><circle cx="390" cy="165" r="95" fill-opacity="0.55" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="2"/><text x="215" y="170" text-anchor="middle" font-size="20" style="fill:var(--compare-a)">FP</text><text x="330" y="170" text-anchor="middle" font-size="22" style="fill:var(--primary)">TP</text><text x="445" y="170" text-anchor="middle" font-size="20" style="fill:var(--compare-b)">FN</text><text x="215" y="195" text-anchor="middle" font-size="11" style="fill:var(--secondary)">wrong alarms</text><text x="445" y="195" text-anchor="middle" font-size="11" style="fill:var(--secondary)">missed cases</text><line x1="270" y1="270" x2="270" y2="300" style="stroke:var(--compare-a)" stroke-width="1.5"/><text x="270" y="320" text-anchor="middle" font-size="15" style="fill:var(--compare-a)">Precision = TP / (TP + FP)</text><line x1="390" y1="270" x2="390" y2="300" style="stroke:var(--compare-b)" stroke-width="1.5"/><text x="390" y="345" text-anchor="middle" font-size="15" style="fill:var(--compare-b)">Recall = TP / (TP + FN)</text></svg>
</div>

## Comparison Table

| Aspect | Precision | Recall |
| --- | --- | --- |
| Question answered | Of items predicted positive, how many actually are positive? | Of items that are actually positive, how many did the model find? |
| Formula | TP / (TP + FP) | TP / (TP + FN) |
| Denominator basis | Total predicted positive (TP + FP) | Total actual positive (TP + FN) |
| Error type penalized | False positives (false alarms) | False negatives (missed detections) |
| Increases when | Model makes fewer incorrect positive calls | Model catches more of the true positive cases |
| Threshold trade-off | Raising the decision threshold typically raises precision | Lowering the decision threshold typically raises recall |
| Failure mode at extreme | High precision, low recall: model is overly conservative and misses real cases | High recall, low precision: model is overly liberal and floods results with false alarms |

## Key Differences

- Precision's denominator is predicted positives; recall's denominator is actual positives, so they measure against different totals
- Precision is hurt by <strong class="kw">false positives</strong>; recall is hurt by <strong class="kw">false negatives</strong>
- Adjusting the classification <strong class="kw">threshold</strong> pushes precision and recall in opposite directions
- Neither metric alone summarizes model quality, which is why the <strong class="kw">F1 score</strong> combines them
- A model with 100% recall can trivially predict everyone positive, and a model with 100% precision can trivially predict almost no one positive

## When to Use Each

**Precision**

- **Spam Filtering**: Marking a real email as spam is costly, so you want confidence that flagged items truly are spam.
- **Content Recommendation**: Showing irrelevant recommendations degrades trust more than missing a few good ones.
- **Automated Content Moderation**: Wrongly banning legitimate users is more damaging than letting a few bad posts slip through.

**Recall**

- **Disease Screening**: Missing a true cancer case is far more dangerous than a false positive that gets ruled out later.
- **Fraud Detection**: Failing to catch fraudulent transactions costs more than flagging a few legitimate ones for review.
- **Search and Retrieval Completeness**: Legal or research search systems need to surface nearly every relevant document, even at the cost of some noise.
