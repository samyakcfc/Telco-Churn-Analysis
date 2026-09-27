# Telco Customer Churn Analysis

## Objective

A business analyst project analyzing Telco Customer Churn data to predict which customers are likely to leave, quantify whether a retention program is worth running, and audit a flawed model to demonstrate common machine learning pitfalls and how to catch them.

## Dataset Information

7,043 customer records with demographic info (gender, senior citizen status), account details (contract type, payment method, tenure), service subscriptions (internet, phone, streaming, tech support), and churn outcome. Source: Kaggle Telco Customer Churn dataset.

## Folder Structure

Telco-Churn-Analysis/ |— data/ (Telco-Customer-Churn.csv) |— notebook/ (case_study.ipynb, model_audit.ipynb) |— reports/ (figures, phase2_summary.md, audit.md) |— DECISIONS.md |— README.md

## Libraries Used

- pandas
- numpy
- scikit-learn
- imbalanced-learn
- optuna
- shap
- matplotlib

## Installation Instructions

pip install -r requirements.txt

## How to Run

1. Ensure `Telco-Customer-Churn.csv` is inside the `data/` folder.
2. Open `notebook/case_study.ipynb` in VS Code or Jupyter for the full case study.
3. Open `notebook/model_audit.ipynb` for the model audit.
4. Run all cells from top to bottom.

## Key Findings

- Data leakage identified: `TotalCharges` is nearly the product of `tenure` and `MonthlyCharges`, causing an inflated R² of 0.999 in Random Forest.
- A cost-based decision threshold (0.10) reduced business cost by 58% compared to the default 0.5 threshold.
- A targeted retention strategy generates an estimated $61,960 in value versus $54,535 for contacting every customer.
- Permutation importance correctly identified a planted random ID column as unimportant (#16 of 20), while impurity importance ranked it #4 of 46 — exposing a known bias.
- Model audit found 8 distinct defects in a flawed script; the two most severe (SMOTE before split, tuning on the test set) inflated PR-AUC by +0.28 and +0.22 respectively.

## Author Information

Samyak Pratap Shah
[GitHub Profile](https://github.com/samyakcfc)
