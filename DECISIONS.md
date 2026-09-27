# DECISIONS.md

## Framing
- **Decision sentence:** Because of this model, the retention team will call any customer whose predicted churn probability exceeds the chosen threshold, within 3 days of their next billing date.
- **Target:** Churn = 1 if Churn == "Yes", observed at the point of the snapshot in this dataset.
- **Base rate:** 26.54%

## Metric
- **Primary metric (classification):** PR-AUC, because the positive class (churn) is the minority class and accuracy would be misleading given the ~73.5% majority baseline.
- **Primary metric (regression, Stage 2):** MAE, to avoid letting a small number of high-tenure, high-bill customers dominate the score.

## Model & Threshold
- **Chosen model:** Logistic Regression (best validation PR-AUC: 0.663, most honest train/validation gap of any model tested)
- **Cost ratio:** Missing a churner (FN) costs 10x more than a wasted retention offer (FP), based on estimated customer lifetime value vs. contact cost.
- **Locked threshold:** 0.10

## Tuning
- **Trial budget (Optuna):** 60 trials, declared in advance
- **Search space:** learning_rate, max_leaf_nodes, min_samples_leaf, l2_regularization
- **Best tuned model (gradient boosting):** CV PR-AUC 0.6687, held-out PR-AUC 0.6723

## Locked Decision Rule
Contact customers whose predicted churn probability (from the logistic regression pipeline, trained on train+validation) exceeds 0.10.

Date locked: 2026-09-05
