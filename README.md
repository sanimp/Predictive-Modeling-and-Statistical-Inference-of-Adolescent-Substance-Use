# Predictive Modeling and Statistical Inference of Adolescent Substance Use: Evidence from Three Decades of CDC Surveillance Data

Code for the paper by **Priscilla Sarfoa Anim, Victor Entsie, Kwame Nyankyerah, and Esther Djan** (Grand Valley State University).

---

## Overview

This repository contains the analysis pipeline for a study of adolescent alcohol and other drug use using the CDC Youth Risk Behavior Surveillance System (YRBSS), 1991–2017. The pipeline covers data quality assessment, exploratory analysis, subgroup comparisons with confidence intervals, and predictive modeling with Random Forest, Gradient Boosting, XGBoost, and CatBoost, interpreted with SHAP and checked against linear/logistic regression baselines and a feature ablation.

**Main finding:** the ensemble models perform moderately well (CatBoost R² = 0.47; tuned XGBoost AUC = 0.78), but an ablation shows that most of the predictive signal comes from *which behavior is measured* (alcohol vs. other drug use), not from demographics or geography. Removing the behavior-type features reduced R² by 87% and untuned XGBoost AUC from 0.77 to 0.60.

---

## Repository contents

| File | Description |
|---|---|
| `Youth_Risk_Analysis.ipynb` | Full analysis notebook (preprocessing, EDA, modeling, SHAP, baselines, ablation) |
| `*.png` | Figures used in the paper (trend, sex, grade, behaviors, maps, correlation) |
| `usa_risk_map_animated.html` | Interactive state-level prevalence map (open in a browser) |
| `README.md` | This file |

---

## Data

The data are **not included** in this repository because of their size. They are publicly available from the CDC:

**CDC – DASH – Youth Risk Behavior Surveillance System (YRBSS): High School – Excluding Sexual Identity, 1991–2017**
https://data.cdc.gov/Youth-Risk-Behaviors/DASH-Youth-Risk-Behavior-Surveillance-System-YRBSS/svam-8dhg/about_data

| Property | Value |
|---|---|
| Raw records | 1,176,120 |
| Variables | 35 (31 after removing 4 fully missing columns) |
| Years | 1991–2017 |
| Unit of analysis | Group-level prevalence estimate (not individual students) |
| Records with a valid prevalence estimate | 353,522 (all in the Alcohol and Other Drug Use topic) |
| Target | `Greater_Risk_Data_Value` (% reporting the greater-risk response) |

The notebook loads a CSV export of this dataset named `Alcohol and Other Drug Use.csv`. On the CDC page, use **Export → CSV** (optionally filtering `Topic` to *Alcohol and Other Drug Use*), save it with that name, and update `file_path` in the data-loading cell if you store it elsewhere.

---

## Methods (summary)

- **Leakage control:** excluded `Greater_Risk_Low_Confidence_Limit`, `Greater_Risk_High_Confidence_Limit`, and `Lesser_Risk_Data_Value`, which are derived from the target.
- **Features:** `YEAR`, `LocationAbbr`, `Topic`, `Subtopic`, `Sex`, `Race`, `Grade`, `Data_Value_Type`.
- **Sample:** simple random sample of 100,000 of the 353,522 complete cases, drawn without replacement; 80/20 train–test split (`random_state = 42`), stratified for classification.
- **Tasks:** regression on prevalence (%) and classification of high vs. low prevalence (threshold = median, 11.35%).
- **Models:** Random Forest, Gradient Boosting, XGBoost, CatBoost; XGBoost tuned with `GridSearchCV` (5-fold CV).
- **Baselines:** linear regression and logistic regression on the same features and split.
- **Ablation:** models retrained without `Topic` and `Subtopic`.
- **Interpretation:** feature importance and SHAP `TreeExplainer`.

---

## Key results

**Regression (predicting prevalence %)**

| Model | R² | RMSE | MAE |
|---|---|---|---|
| CatBoost | 0.4745 | 13.93 | 10.83 |
| XGBoost | 0.4656 | 14.04 | 10.92 |
| Gradient Boosting | 0.4559 | 14.17 | 11.04 |
| Random Forest | 0.2534 | 16.60 | 12.59 |
| Linear regression (baseline) | 0.4134 | 14.71 | – |

**Classification (high vs. low prevalence)**

| Model | Accuracy | AUC-ROC |
|---|---|---|
| XGBoost, tuned | 0.7141 | 0.7790 |
| CatBoost | 0.7094 | 0.7752 |
| Gradient Boosting | 0.7113 | 0.7747 |
| XGBoost, untuned | 0.7127 | 0.7731 |
| Random Forest | 0.6363 | 0.6969 |
| Logistic regression (baseline) | 0.7065 | 0.7591 |

**Ablation (Topic/Subtopic removed)**

| Model | Full features | Topic/Subtopic removed |
|---|---|---|
| CatBoost (R²) | 0.4745 | 0.0596 |
| XGBoost, untuned (AUC-ROC) | 0.7731 | 0.5987 |

**Descriptive findings:** males showed higher mean prevalence than females (19.99% vs. 17.39%); prevalence rose from 9th grade (16.85%) to 12th grade (20.39%); lifetime alcohol use was the most prevalent behavior (68.49%); aggregate prevalence declined from about 21% (1995) to about 17% (2015).

---

## How to run

The notebook was developed in **Google Colab**.

1. Download the dataset from the CDC link above.
2. Open `Youth_Risk_Analysis.ipynb` in Google Colab.
3. Upload the CSV to your Google Drive (the notebook expects `/content/drive/MyDrive/Alcohol and Other Drug Use.csv`), or change `file_path` to where you saved it.
4. Run all cells (`Runtime → Run all`). The first cell installs `catboost` and `shap`. The full pipeline, including `GridSearchCV`, takes about 10–20 minutes.

To run locally instead, remove the `google.colab` lines and install:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn xgboost catboost shap plotly jupyter
```

---

## Limitations

The data are group-level prevalence estimates, so results do not describe individual students, and the analyses do not use YRBSS survey weights or design-based inference. See the paper for the full list of limitations.

---

## Acknowledgments

We thank the Centers for Disease Control and Prevention (CDC) for making the YRBSS data publicly accessible. Analyses used open-source Python libraries including scikit-learn, XGBoost, CatBoost, and SHAP.

## Contact

Priscilla Sarfoa Anim, animp@mail.gvsu.edu
