# Step 3 — Slide Material for the Team
**Bank Customer Churn Prediction — Progress Update**
*Due: April 15 (Wed) Noon — No presentation needed, written submission only*

---

## What to cover (per the guidelines)

> Describe whether you have reached the goal you set for yourself for this milestone, whether your plans have changed, what work is pending, any difficulties, the featurization and classification algorithms you are trying, and how you will evaluate your results.

---

## Slide 1 — Project Status: On Track

**Goal set at proposal:** Predict bank customer churn using ML classifiers on the Kaggle Churn Modelling dataset.

**Milestones completed:**
- Step 1 (Proposal, Mar 11): Problem defined, dataset selected, team roles assigned
- Step 2 (EDA, Mar 25): Data explored, cleaned, featurized, and split into train/val/test
- Step 3 (Now): 4 classifiers trained and evaluated — **on track**

**No major plan changes.** We confirmed during EDA that supervised classification is the right approach. The dataset is clean and well-suited.

---

## Slide 2 — Featurization (from Step 2)

**Dataset:** 10,000 bank customers, binary target: `Exited` (churned = 1)

**Feature engineering applied:**
- Dropped identifier columns: `RowNumber`, `CustomerId`, `Surname`
- One-hot encoded categorical features: `Geography` (→ Germany, Spain dummies) and `Gender` (→ Male dummy)
- Applied `StandardScaler` fitted on training set only (no data leakage)

**Final feature set (11 features):**

| # | Feature | Type |
|---|---|---|
| 1 | CreditScore | Numeric |
| 2 | Age | Numeric |
| 3 | Tenure | Numeric |
| 4 | Balance | Numeric |
| 5 | NumOfProducts | Numeric |
| 6 | HasCrCard | Binary |
| 7 | IsActiveMember | Binary |
| 8 | EstimatedSalary | Numeric |
| 9 | Geography_Germany | Binary (OHE) |
| 10 | Geography_Spain | Binary (OHE) |
| 11 | Gender_Male | Binary (OHE) |

**Stratified 80/10/10 split** (stratified to preserve the 20.4% churn rate in each split):
- Train: 7,999 samples — Val: 1,001 — Test: 1,000

---

## Slide 3 — Classification Algorithms

We trained and compared **4 classifiers**. All use `class_weight='balanced'` where available to handle the 20/80 class imbalance.

| Model | Why we chose it |
|---|---|
| **Logistic Regression** | Interpretable linear baseline; coefficients show direction of each feature's effect |
| **Random Forest** | Handles non-linear relationships (e.g. NumOfProducts spike at 3-4); robust to outliers |
| **Gradient Boosting** | Typically best performance on tabular data; builds trees sequentially to correct errors |
| **SVM (RBF kernel)** | Strong on high-dimensional scaled data; kernel trick captures non-linearity |

---

## Slide 4 — Results: Validation Set Comparison

| Model | Accuracy | Precision | Recall | F1 | **AUC-ROC** |
|---|---|---|---|---|---|
| Logistic Regression | 71.7% | 38.4% | 64.2% | 0.481 | 0.747 |
| **Random Forest** | **82.5%** | **56.9%** | 58.8% | **0.578** | **0.827** |
| Gradient Boosting | 83.9% | 66.9% | 41.7% | 0.514 | 0.826 |
| SVM (RBF) | 78.8% | 48.6% | 66.2% | 0.560 | 0.822 |

**Why AUC-ROC is the primary metric:** The dataset is imbalanced (20% churn). A model that predicts "everyone stays" gets 80% accuracy but is useless. AUC-ROC measures how well the model ranks churners above non-churners across all thresholds, regardless of class balance.

**Why not just use accuracy?** Accuracy is misleading here — even the worst model scores 71.7%, but Logistic Regression only catches 64% of actual churners with poor precision.

---

## Slide 5 — Cross-Validation Confirms Results

5-fold stratified cross-validation (AUC-ROC) on the full training set:

| Model | Mean AUC | Std | Range |
|---|---|---|---|
| Logistic Regression | 0.771 | ±0.015 | [0.754 – 0.794] |
| **Random Forest** | **0.865** | ±0.009 | [0.849 – 0.872] |
| Gradient Boosting | 0.859 | ±0.013 | [0.834 – 0.872] |
| SVM (RBF) | 0.853 | ±0.012 | [0.830 – 0.860] |

Cross-validation confirms the validation set results are **stable and not due to chance**. Random Forest and Gradient Boosting are neck-and-neck.

---

## Slide 6 — Best Model Test Set Results

**Best model selected: Random Forest** (highest validation AUC-ROC = 0.827)

Final evaluation on the **held-out test set** (touched only once):

| Metric | Score |
|---|---|
| Accuracy | 84.3% |
| Precision (churned) | 60.3% |
| Recall (churned) | 67.7% |
| **F1-score (churned)** | **0.637** |
| **AUC-ROC** | **0.864** |

**Interpretation:**
- Of 204 actual churners in the test set, the model correctly identified **138** (recall = 67.7%)
- Of the customers it flagged as churners, **60.3% were actually churning** (precision)
- AUC-ROC of 0.864 means the model correctly ranks a churner above a non-churner **86.4% of the time**

---

## Slide 7 — How We Evaluate Results (Evaluation Framework)

**Primary metric: AUC-ROC**
- Measures overall discrimination ability
- Threshold-independent: works across all operating points
- Ideal for imbalanced datasets

**Secondary metric: F1-score**
- Harmonic mean of precision and recall
- Captures the trade-off: high precision = fewer false alarms; high recall = fewer missed churners

**Business interpretation of the trade-off:**
- **High recall** = catch more churners, but also call some loyal customers "at-risk" → more retention campaign spend
- **High precision** = only intervene when very confident → miss some churners but spend less
- In a real bank, the optimal threshold depends on the cost of a false alarm vs. the value of retaining a churner

**We will also report:** Confusion matrices, Precision-Recall curves, and feature importance plots.

---

## Slide 8 — Key Feature Findings (Consistent Across Models)

Both Random Forest and Gradient Boosting rank the same top features:

1. **Age** — strongest predictor; older customers churn significantly more
2. **NumOfProducts** — non-linear: 2 products = lowest churn (7.6%); 3+ products = near-certain churn (83–100%)
3. **IsActiveMember** — inactive customers churn at 2× the rate of active ones
4. **Balance** — customers with mid-range balances churn more than those with zero balance
5. **Geography_Germany** — German customers churn at 2× the rate of French/Spanish

Logistic Regression agrees on direction but misses the NumOfProducts non-linearity (its linear correlation is only −0.048).

---

## Slide 9 — Difficulties Encountered

1. **Class imbalance (20/80):** Raw accuracy is misleading. Solved with `class_weight='balanced'` and stratified splits. Will also explore SMOTE oversampling for the final submission.

2. **NumOfProducts non-linearity:** Linear models (Logistic Regression) cannot capture the extreme churn at 3–4 products. Tree-based models handle this naturally. May add a polynomial or categorical encoding for the final model.

3. **Gradient Boosting precision-recall trade-off:** GB has the best precision (66.9%) but lowest recall (41.7%), meaning it misses many churners. Threshold tuning will be needed for business use.

4. **Computation time:** SVM and cross-validation on 8,000 samples take several minutes. This is manageable but will need care if we run grid search.

---

## Slide 10 — Work Pending Before Final Submission

| Task | Priority |
|---|---|
| Hyperparameter tuning (grid search on RF and GB) | High |
| Threshold tuning for business-optimal precision/recall | High |
| SMOTE oversampling comparison vs. class weighting | Medium |
| Try XGBoost / LightGBM | Medium |
| SHAP values for explainability | Medium |
| Final 8–10 page report | High |
| Upload all code + data to LMS | High |

**Timeline:**
- April 20 (Mon): Final presentation (15–20 min incl. Q&A)
- April 24 (Fri): Final report + all code due

---

## Figures produced (attach to slides)

| File | Content |
|---|---|
| `fig_roc_curves.png` | ROC curves for all 4 models on validation set |
| `fig_precision_recall.png` | Precision-Recall curves for all 4 models |
| `fig_confusion_matrices.png` | 4 confusion matrices side by side |
| `fig_feature_importance.png` | LR coefficients, RF importance, GB importance |
| `fig_model_comparison.png` | Bar chart comparing F1, AUC, Precision, Recall |
| `fig_test_confusion.png` | Final test set confusion matrix (Random Forest) |

---

*Code: `modeling.ipynb` — trains all models, outputs all figures and metrics*
*Data: `data_splits/` — preprocessed splits from Step 2 EDA*
