### [LINK](https://drive.google.com/drive/folders/1Q5IuJg-39hQrupCrNbEWFTMlQEDMjOii?usp=drive_link) of results (csv)

### A. Comparison with `always_transformer` after 2020

The post-2020 sample contains 19 quarters:

| Group | Number of quarters | selector_rule_based | always_transformer | Difference |
|---|---:|---:|---:|---:|
| All quarters after 2020 | 19 | 0.0568 | 0.0652 | -0.0085 |
| Identical to transformer | 15 | 0.0692 | 0.0692 | 0.0000 |
| Switch away from transformer | 4 | 0.0102 | 0.0504 | -0.0402 |

The result shows that after 2020 `selector_rule_based` does not outperform `always_transformer`. In 15 out of 19 quarters the two strategies are identical. In the remaining 4 quarters, where the selector switches away from PatchTST to CatBoost, the selector underperforms the transformer baseline.

Therefore, interpretation is:

> After 2020, `selector_rule_based` mostly replicates the transformer strategy. The few switches away from the transformer don't improve performance in this subsample. 
---

### B. Ex ante status of the rule

I've chosen rule for selector_rule_based regardless of the results. It was just my assumption.

### C. Turnover and transaction concentration

#### Turnover diagnostics

| Strategy | Mean turnover, full sample | Median turnover, full sample | Mean turnover after 2020 | Median turnover after 2020 |
|---|---:|---:|---:|---:|
| `selector_rule_based` | 0.805 | 0.850 | 0.855 | 0.882 |
| `always_transformer` | 0.884 | 0.945 | 0.865 | 0.906 |
| `always_linear` | 0.606 | 0.605 | 0.560 | 0.566 |

The selector has high turnover. Its turnover is close to the transformer strategy and noticeably higher than the linear strategy.

#### Concentration in PatchTST-selected quarters

| Metric | Value |
|---|---:|
| Mean number of active assets | 8.73 |
| Mean top-5 weight share | 0.733 |
| Mean top-5 absolute contribution share | 0.871 |
| Quarters where top-5 contribution share exceeds 80% | 14 / 15 |
| Mean top-1 absolute contribution share | 0.344 |
| Maximum top-1 absolute contribution share | 0.645 |

This means that in PatchTST-selected quarters, a small group of positions explains a large part of the portfolio result.

At the asset level, the contribution is also concentrated:

| Asset group | Share of absolute contribution |
|---|---:|
| Top 5 assets | 33.9% |
| Top 10 assets | 54.1% |
| Top 20 assets | 74.8% |

#### Interpretation

The selector result may be partly driven by a limited number of assets and several strong quarters.

Interpretation is:

> Additional diagnostics show that PatchTST-selected quarters are characterized by a noticeable concentration of contributions. With an average number of assets of about 8.7, the top 5 positions give an average of 73.3% of the weight and 87.1% of the absolute contribution to profitability. Consequently, the outcome of the PatchTST regime partly depends on a limited set of securities and individual strong quarters.


### D. Bootstrap confidence bands

The question is:

> Do the confidence intervals of `selector_rule_based` materially dominate the confidence intervals of `always_linear` and `always_transformer`?

#### Selector vs transformer

| Metric | `selector_rule_based` CI | `always_transformer` CI | Heavy overlap |
|---|---:|---:|---:|
| Mean return | [0.0162; 0.1465] | [0.0114; 0.1448] | True |
| Annualized return | [0.0425; 0.6769] | [0.0251; 0.6639] | True |
| Sharpe ratio | [0.2047; 2.2563] | [0.1167; 2.1307] | True |
| Max drawdown | [-0.4011; -0.0610] | [-0.3957; -0.0868] | True |
| Calmar ratio | [0.1315; 8.6834] | [0.0687; 6.1021] | True |

The diagnostic result is:

The bootstrap intervals overlap heavily. Therefore, even if the point estimates of `selector_rule_based` are higher in the full sample, the bootstrap results do not support a strong claim that the selector materially dominates.
