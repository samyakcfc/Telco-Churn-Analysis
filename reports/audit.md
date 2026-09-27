# Model Audit: Churn Prediction Script Review

## Headline

The reported figure was **96.4% accuracy, F1 of 0.94**. The honest figure, on a correctly built pipeline, is **PR-AUC of 0.6341** (precision 0.401, recall 0.949 at a cost-chosen threshold of 0.18). The reported numbers were inflated primarily by two compounding leaks — SMOTE applied before the train/test split, and hyperparameter tuning performed directly on the test set — each independently inflating PR-AUC by roughly 0.22–0.28 when isolated and measured.

## Defect Table (ranked by impact)

| # | Defect | Correct | With defect | Inflation | Verdict |
|---|---|---|---|---|---|
| 1 | SMOTE before split | 0.6341 | 0.9114 | +0.2773 | Fatal |
| 2 | Tuned on test set | 0.6341 | 0.8560 | +0.2219 | Fatal |
| 3 | Accuracy on imbalanced data | 0.7665 (acc) | — | — | Misleading (73.5% is the do-nothing floor) |
| 4 | customerID as a numeric feature | 0.6341 | 0.6234 | -0.0107 | Distorts meaning, not score |
| 5 | Scaling fitted before split | 0.6341 | 0.6329 | -0.0012 | Small but real (model-dependent) |
| 6 | Impurity importance interpreted causally | N/A | N/A | N/A | Distorts interpretation |
| 7 | GridSearchCV scored on accuracy | N/A | N/A | N/A | Distorts interpretation |
| 8 | "Target high-ID customers" recommendation | N/A | N/A | N/A | Distorts interpretation (conclusion-only defect, not in the code) |

## The Most Serious Defect, Explained Plainly

The script balanced the dataset with a technique called SMOTE before splitting it into training and test sets, rather than after. This means some of the test examples were built using information from what should have been unseen data, similar to letting a student see practice questions that are near-copies of their final exam. As a result, the reported 96.4% accuracy reflects the model's ability to recognize data it effectively already saw, not its ability to predict churn for genuinely new customers.

## What the Model Is Actually Worth

At the cost-chosen threshold of 0.18 (assuming a missed churner costs roughly 10x more than a wasted retention contact), the corrected model catches 94.9% of actual churners at 40.1% precision. This is a real, usable result — it beats doing nothing — but it is far more modest than the original 96.4%/94% figures suggested, and it should be evaluated against that honest baseline, not the inflated one.

## Process Recommendation

Before any model result reaches a management recommendation, require that the train/test split happen first, before any resampling, scaling, or tuning step touches the data — and that this be checked by a second person reviewing the code, not just the printed metrics. A simple pre-deployment checklist item ("was the test set touched more than once, and was it touched before the final evaluation?") would have caught both fatal defects in this script before they reached a business decision.
