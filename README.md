### Practical winners by objective

- **Best for error minimization:** Elastic Net, Lasso
- **Best for cross-sectional ranking:** Ridge, OLS
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

---

## Typical Quarters vs Stress Periods

### Median MSE

- Elastic Net: 0.0465
- Lasso: 0.0465
- Ridge: 0.0486
- OLS: 0.0492
- Kalman TVP: 0.0494
- Rolling OLS: 0.0500

### Interpretation

I think this means the main differences between models do **not** come from ordinary periods.

The real separation obviously appears in:

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

## Largest average absolute coefficients of Kalman TVP

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

If SHAP outputs are added later, this analysis should be extended to compare:

- coefficient-based interpretation
- SHAP-based contribution importance
- stability of feature importance across models and regimes
