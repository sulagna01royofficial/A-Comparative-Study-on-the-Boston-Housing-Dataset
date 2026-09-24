# Prediction of Median Home Value: Multiple Linear Regression vs. Decision Tree

A comparative study on the **Boston Housing dataset**, predicting the median value of owner-occupied homes (`medv`) with a parametric model (Multiple Linear Regression) and a non-parametric model (Decision Tree Regressor), evaluated on an identical train-test split.

> **Course:** Predictive Analytics & Machine Learning
> **Program:** MSc Data Science, Semester 3 (Batch 2025–2027)
> **Institution:** St. Xavier's College (Autonomous), Kolkata

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Models and Results](#models-and-results)
- [Model Comparison](#model-comparison)
- [Conclusion](#conclusion)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Team](#team)

---

## Overview

The goal is to build and fairly compare two approaches for predicting `medv`:

| Approach | Type | Notes |
| --- | --- | --- |
| Multiple Linear Regression (MLR) | Parametric | Fit on scaled predictors |
| Decision Tree Regressor | Non-parametric | Fit on scaled predictors |

Both models use the **same 80/20 train-test split** (fixed random seed), so any difference in performance comes from the model and not from the data partition.

## Dataset

The Boston Housing dataset contains **506 observations** (towns/suburbs of Boston) and **14 variables**: 13 predictors and 1 target. There are **no missing values**.

| Variable | Description |
| --- | --- |
| `crim` | Per-capita crime rate by town |
| `zn` | % of residential land zoned for lots over 25,000 sq ft |
| `indus` | % of non-retail business acres per town |
| `chas` | Charles River dummy (1 = tract bounds river, 0 = otherwise) |
| `nox` | Nitric oxide concentration (parts per 10 million) |
| `rm` | Average number of rooms per dwelling |
| `age` | % of owner-occupied units built before 1940 |
| `dis` | Weighted mean distance to 5 Boston employment centres |
| `rad` | Index of accessibility to radial highways |
| `tax` | Full-value property-tax rate per $10,000 |
| `ptratio` | Pupil-teacher ratio by town |
| `black` | 1000(Bk − 0.63)², where Bk is the proportion of Black residents by town |
| `lstat` | % lower status of the population |
| **`medv`** | **Median value of owner-occupied homes ($1000s), the target** |

> **Note on `black`:** This variable is a constructed feature built on an assumption about racial segregation and is widely considered problematic. It is retained here only to keep the analysis faithful to the classic dataset. See the [Limitations](#limitations-and-next-steps) section.

## Methodology

1. **Split:** 80/20 train-test split with a fixed random seed, giving **404 training** and **102 test** observations. The split is identical for both models.
2. **Scale:** Predictors are standardized (zero mean, unit variance) before fitting.
3. **Fit:** MLR and Decision Tree are fit on the scaled predictors.
4. **Evaluate:** Train MSE, Test MSE, Train R², and Test R² are computed for both models on the same split.

## Exploratory Data Analysis

**Target distribution (`medv`)**

| Measure | Value |
| --- | --- |
| Mean | 22.5328 |
| Median | 21.2000 |
| Standard deviation | 9.1971 |
| Minimum | 5.0 |
| Maximum | 50.0 |
| Bowley's measure | −0.0470 (slightly negatively skewed) |
| Kappa measure | 1.3824 (leptokurtic) |

**Key observations**

- **Strongest positive correlate:** `rm` (average rooms) with `medv`, r ≈ **+0.70**.
- **Strongest negative correlate:** `lstat` (% lower-status population) with `medv`, r ≈ **−0.74**.
- **Multicollinearity:** `tax` and `rad` are strongly correlated (r ≈ **+0.91**).
- **Charles River (`chas`):** Only about 7% of towns border the river. Those that do tend to show higher and more variable home values, a modest but visible effect.

## Models and Results

### Approach 1: Multiple Linear Regression

Fitted equation (standardized predictors):

```
medv = 22.7965 - 1.0021(crim) + 0.6963(zn) + 0.2781(indus) + 0.7187(chas)
       - 2.0223(nox) + 3.1452(rm) - 0.1760(age) - 3.0819(dis) + 2.2514(rad)
       - 1.7670(tax) - 2.0378(ptratio) + 1.1296(black) - 3.6117(lstat)
```

- **Strongest positive effect:** `rm`. A 1-unit standardized increase adds about **$3,145** to the predicted median value, holding other factors fixed.
- **Strongest negative effect:** `lstat`. A 1-unit standardized increase subtracts about **$3,612**.
- **Highly significant (p < 0.001):** `rm`, `lstat`, `dis`, `nox`, `ptratio`.
- **Moderately significant:** `chas` (p = 0.0109), `rad` (p = 0.0027), `black` (p = 0.0004).
- **Not significant:** `age` (p = 0.70), `indus` (p = 0.59); `zn` is borderline (p = 0.097).

### Approach 2: Decision Tree Regressor

A shallow tree (`max_depth = 3`) is first fit so that every split can be read and interpreted. The depth is then tuned by sweeping `max_depth` and tracking train/test R², and **depth 5** is selected.

- **Feature importance:** `rm` dominates (≈ 0.65), followed by `lstat` (≈ 0.19), then `crim` and `dis`.

### Results summary

| Model | Train MSE | Test MSE | Train R² | Test R² |
| --- | --- | --- | --- | --- |
| Multiple Linear Regression | 21.641 | 24.291 | 0.751 | 0.669 |
| Decision Tree (depth 3) | 15.902 | 16.767 | 0.817 | 0.771 |
| **Decision Tree (depth 5, final)** | **7.079** | **8.554** | **0.919** | **0.883** |

## Model Comparison

| Criterion | Multiple Linear Regression | Decision Tree |
| --- | --- | --- |
| Prediction accuracy | Lower | Higher |
| Test MSE | Higher | Lower |
| Test R² | Lower | Higher |
| Interpretability | High (regression equation) | Moderate (decision rules) |
| Handles non-linearity | No | Yes |
| Prediction pattern | Continuous | Piecewise constant |
| Generalization | Fair | Good |
| **Preferred for this dataset** | | ✔ |

## Conclusion

- **EDA:** `rm` and `lstat` are the strongest linear correlates of `medv`; `tax` and `rad` show multicollinearity.
- **Linear regression:** Simple, interpretable, and stable, but limited to additive, linear relationships.
- **Decision tree:** Captures non-linear patterns and interactions automatically, and outperforms MLR on this dataset (Test R² 0.883 vs. 0.669; Test MSE 8.554 vs. 24.291).
- **Trade-off:** The choice is between accuracy and interpretability, not a search for a single "best" model.

## Limitations and Next Steps

- **High variance in trees:** The R² vs. depth curve is unstable at larger depths. **Cost-complexity pruning** and **Random Forests** are natural next steps to stabilize the tree.
- **Depth selection on the test set:** Choosing `max_depth` by test R² makes the reported test score slightly optimistic. Cross-validation or a separate validation set would give a cleaner estimate.
- **Outliers:** MLR produces some very poor predictions (for example, a test observation with actual `medv` = 17.9 predicted at about −0.16).
- **The `black` variable:** It encodes a racially based construct and is best excluded or handled with care in any real-world modelling.
- **Single split:** Results come from one 80/20 split; repeated splits or k-fold CV would show how stable the comparison is.

## Repository Structure

<!-- Update this to match your actual repo layout -->

```
.
├── data/
│   └── Boston.csv
├── notebooks/
│   └── boston_housing_analysis.ipynb
├── images/
├── presentation/
│   └── Project_4_Group_Project.pptx
├── requirements.txt
└── README.md
```

## Getting Started

```bash
# Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# Install dependencies
pip install -r requirements.txt

# Launch the notebook
jupyter notebook notebooks/boston_housing_analysis.ipynb
```

**Tech stack:** Python, pandas, NumPy, scikit-learn, statsmodels, matplotlib / seaborn

## Team

Semester 3, MSc Data Science (2025–2027), St. Xavier's College (Autonomous), Kolkata

- Oindrila Chakraborty
- Priyanshu Dey
- Anubrata Roy
- Sulagna Roy
- Apratim Dey Tapadar

