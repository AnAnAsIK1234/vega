### [LINK](https://drive.google.com/drive/u/3/folders/13TOoqHVEwvUXFq8WRz-UDyrj3q4p673t) of results (csv)

The following columns are added to `predictions.csv`:

| Column | Meaning |
|---|---|
| `abs_error` | Absolute forecast error for the current observation. |
| `median_j_e_t_m` | Cross-sectional median absolute error for model `m` at date `t`. |
| `e_rolling_i_t_m` | Lagged rolling mean absolute error for asset `i` and model `m`. This is based only on past errors. |
| `e_rolling_model_t_m` | Model-level lagged rolling mean absolute error, used as fallback when asset-level history is too short. |
| `e_reliability_i_t_m` | Error measure actually used for reliability scaling. It uses `e_rolling_i_t_m` when available, otherwise `e_rolling_model_t_m`. |
| `median_j_e_rolling_t_m` | Cross-sectional median of the lagged rolling error measure for model `m` at date `t`. |
| `s_reliability` | Relative instability score. Higher values mean the recent forecast error is high compared with the cross-section. |
| `q_reliability` | Reliability score in `[0, 1]`. Lower values shrink the forecast more strongly. |
| `mu_tilde_i_t_m` | Reliability-scaled forecast used as the final adjusted expected return. |
| `prediction_scaled` | Same as `mu_tilde_i_t_m`, kept as a convenient alias. |

### Reliability scaling formula

For each model \(m\), asset \(i\), and date \(t\), I first compute the absolute forecast error:

```math
e_{i,t}^{(m)} =
\left| r_{i,t} - \hat{\mu}_{i,t}^{(m)} \right|
```

The reliability score is not based on the current error. Instead, I use a lagged rolling mean of past errors:

```math
\bar{e}_{i,t}^{(m)}
=
\frac{1}{W}
\sum_{\tau=t-W}^{t-1}
e_{i,\tau}^{(m)}
```

Then I normalise this error by the cross-sectional median rolling error for the same model and date:

```math
s_{i,t}^{(m)}
=
\frac{
\bar{e}_{i,t}^{(m)}
}{
\mathrm{median}_{j}
\left(
\bar{e}_{j,t}^{(m)}
\right)
}
```

This gives a relative instability score. If \(s_{i,t}^{(m)} > 1\), the model has recently been less reliable for this asset than for the median asset in the cross-section.

The reliability coefficient is then computed as:

```math
q_{i,t}^{(m)}
=
\frac{1}{1 + \lambda s_{i,t}^{(m)}},
\qquad
q_{i,t}^{(m)} \in [0,1]
```

Finally, the raw forecast is scaled before portfolio optimisation:

```math
\tilde{\mu}_{i,t}^{(m)}
=
q_{i,t}^{(m)}
\cdot
\hat{\mu}_{i,t}^{(m)}
```

Here, \(W\) is the rolling window length and \(\lambda\) controls how strongly unreliable forecasts are shrunk. In the baseline setup, I use \(W=4\) and \(\lambda=0.5\). Since the rolling error is shifted by one period, the reliability score at date \(t\) only uses information available before \(t\).
