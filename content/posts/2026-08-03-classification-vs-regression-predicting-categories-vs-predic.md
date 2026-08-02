---
title: "Classification vs Regression: Predicting Categories vs Predicting Numbers"
date: 2026-08-03T03:28:23.233983+09:00
tags: ["machine-learning", "classification", "regression", "supervised-learning"]
---
## Overview

Classification and regression are the two core types of supervised learning, distinguished by what kind of output they predict. Classification assigns inputs to a <strong class="kw">discrete class</strong>, while regression estimates a <strong class="kw">continuous value</strong>. Picking the wrong one for your target variable leads to mismatched loss functions, evaluation metrics, and model outputs.

## Comparison Diagram

<div class="compare-diagram">
<svg viewBox="0 0 640 360" xmlns="http://www.w3.org/2000/svg"><line x1="320" y1="20" x2="320" y2="340" stroke-dasharray="4 4" style="stroke:var(--border)" stroke-width="1"/><text x="160" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Classification</text><text x="480" y="30" text-anchor="middle" style="fill:var(--primary)" font-size="20" font-weight="bold">Regression</text><line x1="60" y1="280" x2="280" y2="280" style="stroke:var(--border)" stroke-width="1"/><line x1="60" y1="80" x2="60" y2="280" style="stroke:var(--border)" stroke-width="1"/><line x1="150" y1="85" x2="270" y2="270" stroke-dasharray="5 3" style="stroke:var(--border)" stroke-width="1.5"/><circle cx="90" cy="150" r="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="110" cy="130" r="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="95" cy="175" r="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="125" cy="160" r="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="80" cy="200" r="6" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="214" y="214" width="12" height="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="234" y="194" width="12" height="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="249" y="229" width="12" height="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="224" y="245" width="12" height="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><rect x="199" y="204" width="12" height="12" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><circle cx="70" cy="305" r="5" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="82" y="309" style="fill:var(--content)" font-size="11">Class A</text><rect x="150" y="300" width="10" height="10" style="fill:var(--compare-a-soft);stroke:var(--compare-a)" stroke-width="1.5"/><text x="165" y="309" style="fill:var(--content)" font-size="11">Class B</text><text x="160" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="12">output: discrete category</text><line x1="380" y1="280" x2="600" y2="280" style="stroke:var(--border)" stroke-width="1"/><line x1="380" y1="80" x2="380" y2="280" style="stroke:var(--border)" stroke-width="1"/><circle cx="400" cy="245" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="420" cy="225" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="440" cy="230" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="460" cy="205" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="480" cy="195" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="500" cy="175" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="520" cy="165" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="540" cy="145" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><circle cx="560" cy="135" r="5" style="fill:var(--compare-b-soft);stroke:var(--compare-b)" stroke-width="1.5"/><line x1="392" y1="255" x2="588" y2="120" style="stroke:var(--compare-b)" stroke-width="2"/><text x="480" y="335" text-anchor="middle" style="fill:var(--secondary)" font-size="12">output: continuous number</text></svg>
</div>

## Comparison Table

| Aspect | Classification | Regression |
| --- | --- | --- |
| Target variable type | Discrete, categorical labels from a finite set of classes | Continuous, ordered numeric values |
| Learning objective | Learn a decision boundary that separates classes | Learn a function mapping inputs to a continuous output |
| Typical loss function | Cross-entropy, log loss, or hinge loss | Mean squared error or mean absolute error |
| Model output format | Class label or probability distribution over classes | Single scalar value (or vector of scalars) |
| Common algorithms | Logistic regression, SVM, decision trees, kNN, softmax networks | Linear regression, ridge/lasso, decision trees, kNN, regression networks |
| Evaluation metrics | Accuracy, precision/recall, F1, ROC-AUC, confusion matrix | RMSE, MAE, R-squared, MAPE |
| Error interpretation | Prediction is simply right, wrong, or confused with another class | Prediction error has magnitude and direction, showing how far off it was |

## Key Differences

- Classification predicts a <strong class="kw">discrete label</strong> from a fixed set of classes, while regression predicts a <strong class="kw">continuous value</strong> on a numeric scale.
- Classification models typically optimize <strong class="kw">cross-entropy loss</strong> to separate classes, while regression models optimize <strong class="kw">squared error</strong> to minimize distance from the true value.
- Classification is evaluated with metrics like <strong class="kw">accuracy/F1</strong>, while regression is evaluated with metrics like <strong class="kw">RMSE/R-squared</strong>.
- A classification error is simply right, wrong, or a class confusion, while a regression error carries a <strong class="kw">magnitude</strong> showing how far off the prediction was.

## When to Use Each

**Classification**

- **Spam Detection**: Binary classification decides whether an email belongs to the spam or not-spam class.
- **Image Recognition**: Assigns an image to one of several known object categories.
- **Medical Diagnosis**: Predicts whether a patient falls into a disease-positive or disease-negative category.

**Regression**

- **House Price Prediction**: Estimates a continuous dollar value based on property features.
- **Demand Forecasting**: Predicts a continuous quantity like units sold next month.
- **Temperature Prediction**: Predicts a continuous numeric value like tomorrow's high temperature.
