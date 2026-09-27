# Phase 2 Summary: Telco Customer Churn

## 1. Problem & Decision
Because of this model, the retention team will call any customer whose predicted churn probability exceeds 0.10, within 3 days of their next billing date. Base rate: 26.54% churn.

## 2. Data & Method
7,043 customers, 19 features after dropping the identifier column. Split 60/20/20, stratified, sealed until Stage 6. Same ColumnTransformer + Pipeline reused across every model (median/most-frequent imputation, standard scaling, one-hot encoding).

## 3. Baselines
| Baseline | Accuracy |
|---|---|
| Majority class | 0.7346 |
| Business rule (month-to-month + no tech support) | 0.7424 |
| Logistic regression (3 features) | 0.7906 |

## 4. Regression Detour (TotalCharges)
Random Forest hit a suspicious R²=0.999. Investigation confirmed `TotalCharges` ≈ tenure × MonthlyCharges; dropping MonthlyCharges only reduced R² to 0.995, since service columns still encode pricing information. Linear models (R²=0.908) were unaffected either way, confirming the leak is structural, not column-specific.

## 5. Classification & Threshold
Logistic Regression was the best and most honest classifier (validation PR-AUC 0.663, smallest train/validation gap). At the assumed 10:1 cost ratio (missed churner vs. wasted offer), the cost-optimal threshold is 0.10, cutting business cost 58% versus the default 0.5 threshold. Imbalance strategies (class_weight, SMOTE) matched each other but did not clearly beat simple threshold tuning on the plain model.

## 6. Ensembles & Tuning
Random Forest and Gradient Boosting did not beat Logistic Regression on validation PR-AUC, despite Random Forest reaching a perfect 1.000 on training data (severe overfitting, 0.376 gap). Optuna tuning (60 trials, declared in advance) improved Gradient Boosting to 0.6687 CV PR-AUC. The winner's-curse check showed a small negative gap (-0.0036), within normal fold-to-fold noise rather than clear evidence either way.

## 7. Interpretability
Permutation importance correctly ranked a planted random ID column near the bottom (#16/20), while impurity importance ranked it #4/46 — demonstrating impurity importance's known bias toward high-cardinality features. SHAP confirmed Contract, tenure, and MonthlyCharges as the top three drivers. K-means (k=3) revealed a genuine, actionable segment: senior citizens churn at 41.7% versus 14.5% for long-tenured high-value customers, a pattern not surfaced by the supervised feature importance methods.

## 8. Checkpoint Result & Business Answer
Test set opened once, after locking the decision rule. Test PR-AUC: 0.6196 (validation-test gap: 0.0584, reported without adjustment). The targeted strategy generates an estimated $61,960 in value on the test set, versus $54,535 for contacting every customer — a $7,425 improvement. **The retention program is worth running, and the model's targeting outperforms a blanket contact-everyone policy.**

## Limitations
The validation-test gap (0.0584) indicates some optimism in validation-stage model selection that did not fully carry to test. Cost assumptions ($25 contact cost, $800 customer value, 30% offer success rate) were not independently verified and materially affect the recommended threshold. Clustering used only numeric features and does not incorporate Contract or TechSupport, the two strongest churn signals identified elsewhere in this analysis.
