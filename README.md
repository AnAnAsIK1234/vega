# Model Comparison: Kalman TVP vs Linear Benchmarks

## Overview

This experiment compares the predictive performance of the following models on the same quarterly out-of-sample backtest pipeline:

- Kalman TVP
- OLS
- Ridge
- Lasso
- Elastic Net
- Rolling OLS

The goal was to evaluate whether a time-varying parameter model (Kalman TVP) provides a meaningful advantage over simpler linear alternatives.

In addition to forecast accuracy, the comparison also considered:

- ranking quality via Spearman correlation
- regime robustness
- coefficient stability
- practical usefulness for production

---

## Executive Summary

The main conclusion is:

**Kalman TVP is not the best predictive model in this setup.**

Its main value is interpretability of time-varying coefficients rather than clear out-of-sample superiority.

### Practical winners by objective

- **Best for error minimization:** Elastic Net, Lasso
- **Best for cross-sectional ranking:** Ridge, OLS
- **Best for dynamic interpretation:** Kalman TVP
- **Model to avoid:** Rolling OLS

---

## Final Metric Comparison

### Mean MSE

- **Elastic Net:** 0.0661
- **Lasso:** 0.0665
- **Kalman TVP:** 0.0761
- **Ridge:** 0.0845
- **OLS:** 0.0866
- **Rolling OLS:** 0.2381

### Mean MAE

- **Elastic Net:** 0.1765
- **Lasso:** 0.1765
- **Kalman TVP:** 0.1991
- **Ridge:** 0.2049
- **OLS:** 0.2067
- **Rolling OLS:** 0.3126

### Mean Spearman

- **Ridge:** 0.0484
- **OLS:** 0.0477
- **Kalman TVP:** 0.0327
- **Rolling OLS:** 0.0310
- **Elastic Net:** -0.0165
- **Lasso:** -0.0172

---

## Key Interpretation

### 1. Best models depend on the objective

#### If the objective is forecast accuracy
Elastic Net and Lasso are the strongest models.

- Elastic Net improves on Kalman TVP by about **13.2% in mean MSE**
- Lasso improves on Kalman TVP by about **12.6% in mean MSE**

#### If the objective is cross-sectional ranking
Ridge and OLS perform best.

Kalman TVP ranks below both Ridge and OLS on Spearman correlation.

---

## Typical Quarters vs Stress Periods

A very important result is that in normal quarters, almost all models perform similarly.

### Median MSE

- Elastic Net: 0.0465
- Lasso: 0.0465
- Ridge: 0.0486
- OLS: 0.0492
- Kalman TVP: 0.0494
- Rolling OLS: 0.0500

### Interpretation

This means the main differences between models do **not** come from ordinary periods.

The real separation appears in:

- difficult quarters
- stressed regimes
- tail events

---

## Stability in the Tails

A useful proxy for instability is:

`mean MSE / median MSE`

- Elastic Net: 1.42
- Lasso: 1.43
- Kalman TVP: 1.54
- Ridge: 1.74
- OLS: 1.76
- Rolling OLS: 4.76

### Interpretation

- **Elastic Net and Lasso** have the best tail control
- **Kalman TVP** is more stable than OLS and Ridge, but still weaker than Elastic Net / Lasso
- **Rolling OLS** is highly unstable and prone to blow-ups

---

## Regime Analysis

### Inflation regime

#### Kalman TVP
- low inflation MSE: 0.0649
- high inflation MSE: 0.0934

This is a deterioration of about **44%**.

#### Elastic Net / Lasso
Only a small degradation, around **3%**.

#### OLS / Ridge
These models actually perform slightly **better** in high inflation than in low inflation.

### Conclusion

Kalman TVP does **not** show the expected advantage in inflation regime adaptation.

This is one of the strongest arguments against treating it as the primary production model.

---

### Stress regime

#### Kalman TVP
- non-stress MSE: 0.0508
- stress MSE: 0.0925

This is roughly an **82%** increase.

#### Elastic Net / Lasso
Roughly **55%** increase.

#### OLS / Ridge
Almost no deterioration, and in some cases slightly better.

#### Rolling OLS
Very large deterioration:
- 0.0954 -> 0.3305

### Conclusion

Kalman TVP does not dominate in stress periods, and Rolling OLS is clearly not robust enough.

---

## Why Elastic Net and Lasso Win on MSE but Lose on Ranking

This behavior is explained by the coefficient paths.

### Lasso
- all slope coefficients are zero in **60.7%** of periods

### Elastic Net
- all slope coefficients are zero in **46.4%** of periods

Additional feature-level sparsity is also high:

- `value_it` zeroed in about **78.6%**
- `illiq_amihud_m` zeroed in about **75.0%**
- `value_x_inflhigh` zeroed in about **75–78.6%**
- `state_dd_stress` zeroed in about **71.4%**
- `MOM` and `size_it` zeroed in more than **64%**

### Interpretation

These models often collapse toward a very sparse structure:

- intercept
- a small number of strong macro / market variables
- heavy shrinkage on cross-sectional features

This explains the trade-off:

- **stronger MSE**
- **weaker ranking power**

They reduce variance well, but often do not produce rich cross-sectional separation.

---

## What Kalman TVP Is Actually Doing

Kalman TVP does produce meaningful time variation in coefficients.

### Largest average absolute coefficients

- `MOM`: 0.0818
- `mkt_vol`: 0.0715
- `state_dd_stress`: 0.0338
- `state_infl_high`: 0.0337
- `size_it`: 0.0300
- `illiq_amihud_m`: 0.0260

### Interpretation

The model places the strongest weight on:

- momentum
- market volatility
- regime indicators

This makes it useful for interpretation and factor dynamics analysis.

However, this adaptiveness does not convert into the best out-of-sample predictive performance.

---

## Evidence of Over-Adaptation in Kalman TVP

Kalman TVP also shows frequent sign changes:

- `size_it`: 6 sign changes
- `state_dd_stress`: 7
- `intercept`: 6
- `state_infl_high`: 4
- `value_it`: 4

### Interpretation

This suggests that part of the model’s flexibility may be reacting to noise rather than stable economic structure.

In other words:

**Kalman TVP is adaptive, but not all of that adaptation is useful.**

---

## What the Chosen `q` Values Tell Us

Three Kalman TVP state-noise settings were tested:

- `q = 1e-5`
- `q = 1e-4`
- `q = 1e-3`

### Final selected path
- `1e-5`: 13 times
- `1e-4`: 8 times
- `1e-3`: 7 times

### Validation selection
- `1e-5`: 16 times
- `1e-4`: 8 times
- `1e-3`: 5 times

### Interpretation

Most of the time, the model prefers **slow coefficient drift**.

This is a critical finding:

**The data do not support aggressively time-varying coefficients for most periods.**

That means the main bottleneck is likely not model flexibility, but rather:

- noisy features
- unstable scaling
- weak cross-sectional normalization
- fragile interaction structure

---

## OLS and Ridge

OLS and Ridge are simple, but their behavior is coherent and stable.

### Top OLS features
- `mkt_vol`: 0.0285
- `state_infl_high`: 0.0206
- `MOM`: 0.0152
- `size_it`: 0.0151
- `state_dd_stress`: 0.0139

### Top Ridge features
- `mkt_vol`: 0.0280
- `state_infl_high`: 0.0198
- `size_it`: 0.0149
- `MOM`: 0.0147
- `state_dd_stress`: 0.0136

### Interpretation

Their coefficients are much more stable than Kalman TVP and Rolling OLS.

That stability likely explains why they perform best on ranking.

---

## Rolling OLS

Rolling OLS is the weakest model in the experiment.

### Problems

- worst mean MSE
- worst MAE
- very poor tail behavior
- heavy regime sensitivity
- unstable coefficients

### Coefficient instability
Examples of sign changes:

- `state_infl_high`: 6
- `value_it`: 6
- `value_x_inflhigh`: 6
- `MOM`: 5
- `illiq_x_stress`: 5

### Conclusion

Rolling OLS behaves like a locally overfit model.

It should be removed from the production shortlist.

---

## Hyperparameter Selection Insights

### Ridge
Selection is highly bimodal:

- either almost no regularization: `alpha = 1e-4`
- or very strong regularization: `alpha = 100`

This suggests the data do not prefer moderate shrinkage.

### Lasso
Most common choice:

- `alpha = 0.1` selected 15 times

This indicates the data frequently prefer strong sparsity.

### Elastic Net
Most common settings:

- `alpha = 0.1, l1_ratio = 0.5` selected 10 times
- `alpha = 0.1, l1_ratio = 0.2` selected 6 times

This supports the idea that the best practical compromise is:

- meaningful shrinkage
- not purely L1
- a balanced mix of sparsity and stability

---

## Overall Diagnosis

### What is already clear

1. There is signal in the data.
2. That signal does not require aggressive time variation in coefficients most of the time.
3. Regularization helps more than state-space complexity.
4. The current bottleneck is more likely in the data than in the model class.

### Likely data issues

The results strongly suggest that the next performance gain should come from better feature engineering, not a more complex model.

Most likely improvement areas:

- heavy-tailed features
- unstable feature scaling
- weak cross-sectional normalization
- noisy regime interactions
- missingness handling
- transformation of illiquidity and related variables

---

## Recommended Production Stack

### If the primary objective is level prediction
Use:

- **Elastic Net**
- **Lasso**

### If the primary objective is ranking / sorting
Use:

- **Ridge**
- **OLS**

### If dynamic factor interpretation is needed
Keep:

- **Kalman TVP** as an explanatory / research model

### Remove
- **Rolling OLS**

---

## Final Conclusion

**Kalman TVP is useful as an explanatory layer, but not as the best predictive model in this experiment.**

The strongest practical performers are:

- **Elastic Net / Lasso** for forecast error
- **Ridge / OLS** for ranking quality

This means the next likely improvement will come from:

- better transformations of the input data
- more robust feature engineering
- more careful cross-sectional normalization

rather than from adding even more model complexity.

---

## Important Note on SHAP

This analysis was based on:

- model summary metrics
- regime summaries
- validation logs
- coefficient paths

SHAP results were not included in the reviewed outputs, so feature importance conclusions here are based on coefficient behavior, not SHAP decomposition.

If SHAP outputs are added later, this analysis should be extended to compare:

- coefficient-based interpretation
- SHAP-based contribution importance
- stability of feature importance across models and regimes