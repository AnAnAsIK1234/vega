### [LINK](https://drive.google.com/drive/u/3/folders/13TOoqHVEwvUXFq8WRz-UDyrj3q4p673t) of results (csv)

### Practical winners by objective

* **Best for error minimization:** CatBoost, Elastic Net, Lasso
* **Best for cross-sectional ranking:** Rolling OLS, OLS
* **Worst Model:** PatchTST Global

---

## Final Metric Comparison

### Mean MSE

* **CatBoost:** 0.0666
* **Elastic Net:** 0.0675
* **Lasso:** 0.0676
* **Ridge:** 0.0680
* **OLS:** 0.0684
* **Kalman TVP:** 0.0740
* **PatchTST Global:** 0.0880
* **Rolling OLS:** 0.0928

### Mean MAE

* **CatBoost:** 0.1761
* **Lasso:** 0.1762
* **Elastic Net:** 0.1774
* **Ridge:** 0.1796
* **OLS:** 0.1807
* **Kalman TVP:** 0.1929
* **PatchTST Global:** 0.2119
* **Rolling OLS:** 0.2177

### Mean Spearman

* **Rolling OLS:** 0.0197
* **OLS:** 0.0167
* **Ridge:** 0.0110
* **Lasso:** 0.0019
* **Elastic Net:** -0.0064
* **Kalman TVP:** -0.0065
* **CatBoost:** -0.0165
* **PatchTST Global:** -0.0392

---

## Key Interpretation

### 1. Best models depend on the objective

#### If the objective is forecast accuracy

CatBoost, Elastic Net, and Lasso are the strongest models.

* CatBoost improves on Kalman TVP by about **10.1% in mean MSE**
* Elastic Net improves on Kalman TVP by about **8.8% in mean MSE**
* Lasso improves on Kalman TVP by about **8.6% in mean MSE**

#### If the objective is cross-sectional stock ranking

Rolling OLS and OLS are the strongest models.

* Rolling OLS delivers the highest mean Spearman: **0.0197**
* OLS is next: **0.0167**

That means the models that minimize forecast error are not the same models that sort stocks best in the cross section.

---

## Typical Quarters vs Stress Periods

### Median MSE

* CatBoost: 0.0451
* Elastic Net: 0.0465
* Lasso: 0.0465
* PatchTST Global: 0.0466
* Ridge: 0.0489
* OLS: 0.0490
* Kalman TVP: 0.0507
* Rolling OLS: 0.0512

### Interpretation

I think this means the main differences between models do **not** come from ordinary quarters.

The real separation appears more clearly in:

* difficult quarters
* stressed regimes
* tail outcomes

CatBoost, Elastic Net, and Lasso all look very similar in a typical quarter.
The larger differences only show up once the sample moves away from the median environment.

---

## Stability in the Tails

A useful proxy for instability is:

`mean MSE / median MSE`

* Ridge: 1.39
* OLS: 1.40
* Elastic Net: 1.45
* Lasso: 1.46
* Kalman TVP: 1.46
* CatBoost: 1.48
* Rolling OLS: 1.81
* PatchTST Global: 1.89

### Interpretation

* **Ridge and OLS** have the best tail stability
* **Elastic Net, Lasso, and Kalman TVP** are still reasonably controlled
* **CatBoost** is competitive on average error, but less stable than the simpler linear baselines
* **Rolling OLS** is unstable
* **PatchTST Global** is the least stable model in the full set

So the weakest point in this experiment is no longer only Rolling OLS.
The transformer model is even less robust in the tails.

---

## Regime Analysis

### Inflation regime

#### Kalman TVP

* low inflation MSE: 0.0726
* high inflation MSE: 0.0763

This is a deterioration of about **5.1%**.

#### Elastic Net / Lasso

These models deteriorate much more:

* Elastic Net: **+29.9%**
* Lasso: **+29.0%**

#### OLS / Ridge

These models also deteriorate strongly:

* OLS: **+37.2%**
* Ridge: **+36.6%**

#### CatBoost

* low inflation MSE: 0.0617
* high inflation MSE: 0.0741

This is a deterioration of about **20.1%**.

#### PatchTST Global

* low inflation MSE: 0.0730
* high inflation MSE: 0.1112

This is a deterioration of about **52.3%**.

### Conclusion

Kalman TVP does show some relative resilience in the inflation regime.

But that resilience is not enough to overcome its weaker overall average accuracy.

CatBoost appears more robust than the sparse linear models in inflation, while PatchTST performs especially poorly.

---

### Stress regime

#### Kalman TVP

* non-stress MSE: 0.0515
* stress MSE: 0.0886

This is roughly a **72.1%** increase.

#### Elastic Net / Lasso

* Elastic Net: **+64.5%**
* Lasso: **+65.1%**

#### OLS / Ridge

* OLS: **+56.0%**
* Ridge: **+56.8%**

#### CatBoost

* non-stress MSE: 0.0491
* stress MSE: 0.0778

This is a **58.4%** increase.

#### Rolling OLS

Very large deterioration:

* 0.0589 -> 0.1147

This is about a **94.6%** increase.

#### PatchTST Global

* 0.0637 -> 0.1037

This is about a **62.8%** increase.

### Conclusion

Kalman TVP does not dominate in stress periods.

The best stress robustness in this run comes from:

* **OLS**
* **Ridge**
* and, among the stronger error-minimizers, **CatBoost**

Rolling OLS and PatchTST are clearly not robust enough.

---

## Why CatBoost, Elastic Net, and Lasso Win on MSE but Lose on Ranking

This behavior is explained by how selective or nonlinear their signal extraction becomes.

### Lasso

* all slope coefficients are zero in **46.4%** of periods

### Elastic Net

* all slope coefficients are zero in **42.9%** of periods

Additional feature-level sparsity is also high.

### Lasso: most frequently zeroed features

* `illiq_missing`: **100.0%**
* `value_cs_x_inflhigh`: **85.7%**
* `value_cs`: **82.1%**
* `value_missing`: **82.1%**
* `state_dd_stress`: **82.1%**
* `illiq_cs`: **82.1%**

### Elastic Net: most frequently zeroed features

* `illiq_missing`: **100.0%**
* `value_cs_x_inflhigh`: **85.7%**
* `value_cs`: **82.1%**
* `illiq_cs`: **82.1%**
* `value_missing`: **82.1%**
* `state_dd_stress`: **78.6%**

### Interpretation

These sparse models often collapse toward a compact structure built around a small number of stronger signals.

That helps with:

* average error minimization

But it can hurt:

* cross-sectional discrimination across many stocks at the same date

CatBoost shows a similar trade-off from a different mechanism:

* better regression fit
* weaker ranking quality

---

## Largest average absolute coefficients of Kalman TVP

* `value_cs_x_inflhigh`: 0.0585
* `state_infl_high`: 0.0565
* `MOM`: 0.0532
* `illiq_cs_x_stress`: 0.0490
* `value_cs`: 0.0488
* `mkt_vol`: 0.0453

### Interpretation

The model places the strongest weight on:

* regime interactions
* inflation state
* momentum
* market volatility

So Kalman TVP is using the flexible specification in the expected direction.
But that flexibility still does not translate into top overall performance.

---

## Evidence of Over-Adaptation in Kalman TVP

Kalman TVP also shows frequent sign changes:

* `value_missing`: 10 sign changes
* `illiq_cs`: 7
* `size_cs`: 6
* `state_infl_high`: 5
* `illiq_cs_x_stress`: 5
* `MOM`: 4

### Interpretation

This suggests that part of the model’s flexibility may be reacting to noise rather than stable economic structure.

The model is adaptive, but not always in a way that improves out-of-sample performance.

---

## What the chosen `q` values tell us

Three Kalman TVP state-noise settings were tested:

* `q = 1e-5`
* `q = 1e-4`
* `q = 1e-3`

### Final selected path

* `1e-5`: 14 times
* `1e-4`: 8 times
* `1e-3`: 6 times

### Validation selection

* `1e-5`: 16 times
* `1e-4`: 6 times
* `1e-3`: 7 times

### Interpretation

Most of the time, the model prefers **slow coefficient drift**.

This is again a critical finding:

**The data do not support aggressively time-varying coefficients for most periods.**

That suggests the main limitation is not lack of flexibility, but signal quality.

---

## OLS and Ridge

OLS and Ridge are simple, but their behavior is coherent and stable.

### Top OLS features

* `value_cs_x_inflhigh`: 0.0596
* `illiq_cs_x_stress`: 0.0577
* `state_infl_high`: 0.0538
* `value_cs`: 0.0452
* `MOM`: 0.0407
* `state_dd_stress`: 0.0330

### Top Ridge features

* `state_infl_high`: 0.0445
* `MOM`: 0.0393
* `illiq_cs_x_stress`: 0.0322
* `state_dd_stress`: 0.0295
* `value_cs_x_inflhigh`: 0.0202
* `mkt_vol`: 0.0200

### Interpretation

Their coefficients are much more stable than those of Kalman TVP and Rolling OLS.

That stability likely explains why they perform best on ranking.

---

## CatBoost

CatBoost is the strongest nonlinear tabular benchmark in this experiment.

### Main result

* mean MSE: **0.0666**
* mean MAE: **0.1761**
* mean Spearman: **-0.0165**

### Interpretation

CatBoost is good at:

* minimizing prediction error

But it is weak at:

* cross-sectional ranking

So CatBoost looks useful as a nonlinear regression benchmark, but not as a stock-sorting signal in its current form.

### Largest average feature importances

* `mkt_vol__lag2`: 0.0256
* `MOM`: 0.0184
* `mkt_vol__lag4`: 0.0098
* `mkt_vol`: 0.0089
* `MOM__lag1`: 0.0089
* `state_infl_high`: 0.0049

### Interpretation

CatBoost relies heavily on:

* lagged volatility
* current and lagged momentum
* inflation-state information

---

## PatchTST Global

PatchTST Global is the weakest model in the experiment.

### Main result

* mean MSE: **0.0880**
* mean MAE: **0.2119**
* mean Spearman: **-0.0392**

### Interpretation

The sequence model is not extracting a useful signal in its current specification.

This suggests that one or more of the following is true:

* the quarterly sample is too short for this model class
* the available feature history does not support a transformer advantage

So PatchTST does **not** justify its added complexity in this run.
