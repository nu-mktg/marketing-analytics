# Chapter 12: Difference-in-Differences
## Causal Identification of Marketing Interventions

---

## 12.1 Overview

Every model covered in Chapters 1–11 produces an associational estimate — a measure of how two quantities co-vary in observed data. Marketing Mix Models correlate channel spend with revenue. Survival models correlate customer attributes with churn timing. Attribution models correlate channel appearances with conversion. None of them, on their own, identifies whether a relationship is causal.

Difference-in-differences (DiD) is the standard quasi-experimental method for recovering a causal estimate from observational data when a clean randomised experiment is unavailable, or for validating that a randomised geo-holdout produced the expected result. It is the single most commonly used causal identification strategy in applied marketing analytics.

**Estimand:** The average treatment effect on the treated — the causal effect of the intervention on the units that received it, during the post-intervention period.

**Required data:** An outcome measured in at least two periods (pre and post), for at least two groups (treatment and control), with known assignment to treatment.

---

## 12.2 The Identification Problem

### The counterfactual

Consider a subscription firm that reduces its monthly price by 15% in four geographic regions starting in week 11. Revenue grows in those regions. The observed quantity is:

$$\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}} = \underbrace{\tau}_{\text{causal effect}} + \underbrace{\delta}_{\text{common trend}}$$

where $\tau$ is the effect we want and $\delta$ is the time trend that would have occurred regardless of the intervention. We observe their sum, not each separately.

**The fundamental problem:** We cannot observe $Y_{T,\text{post}}^{(0)}$ — the revenue that would have occurred in the treatment regions absent the price reduction. This is the counterfactual.

### The naive estimator

$$\hat{\tau}_{\text{naive}} = \bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}}$$

This is biased whenever there is a time trend in the outcome. If revenue was growing by 4 units/week regardless of the price change, the naive estimator attributes all 4 units of trend to the price change.

---

## 12.3 The DiD Estimator

### Construction

Difference-in-differences uses a **control group** — regions not exposed to the intervention — to estimate the counterfactual. The control group's time trend is used as a proxy for what would have happened in the treatment group absent treatment.

$$\boxed{\hat{\tau}_{\text{DiD}} = (\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}}) - (\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}})}$$

The first difference removes the common time trend. The second removes any pre-existing level difference between treatment and control. The double differencing is where the name comes from.

### Worked example

Dataset `geo_experiment.csv`: 8 regions, 20 weeks.

| Period | Treatment (regions 1–4) | Control (regions 5–8) |
|---|---|---|
| Pre (weeks 1–10) | 146.39 | 137.02 |
| Post (weeks 11–20) | 156.47 | 140.84 |

$$\hat{\tau}_{\text{DiD}} = (156.47 - 146.39) - (140.84 - 137.02) = 10.08 - 3.83 = \mathbf{6.25}$$

**Comparison:** The naive estimate would be 10.08. The DiD estimate removes the 3.83 units that would have occurred anyway due to the common time trend. The naive estimator overstates the causal effect by 61%.

---

## 12.4 Regression Formulation

The 2×2 DiD is a special case of ordinary least squares with an interaction term:

$$Y_{it} = \beta_0 + \beta_1 \text{Treat}_i + \beta_2 \text{Post}_t + \beta_3 (\text{Treat}_i \times \text{Post}_t) + \varepsilon_{it}$$

**Coefficient interpretation:**

$$\beta_0 = \bar{Y}_{C,\text{pre}} = 137.02 \qquad \beta_1 = \bar{Y}_{T,\text{pre}} - \bar{Y}_{C,\text{pre}} = 9.38$$

$$\beta_2 = \bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}} = 3.83 \qquad \beta_3 = \hat{\tau}_{\text{DiD}} = 6.25$$

**Critical fact:** The treatment effect is $\beta_3$, the coefficient on the interaction $\text{Treat} \times \text{Post}$. The other three coefficients describe the pre-period structure of the data and are not the treatment effect.

**From `geo_experiment.csv`:**

```python
import statsmodels.formula.api as smf
model = smf.ols("revenue ~ treat + post + treat_x_post", data=df).fit()
# beta3 = model.params['treat_x_post'] ≈ 6.25, p ≈ 0.049
```

---

## 12.5 The Parallel Trends Assumption

### Statement

DiD identifies a causal effect under one key assumption:

$$E[Y_{T,\text{post}}^{(0)} - Y_{T,\text{pre}}] = E[Y_{C,\text{post}} - Y_{C,\text{pre}}]$$

Absent the intervention, the treatment and control groups would have followed the same time trend. This is the **parallel trends assumption**.

### Testability

This assumption cannot be tested directly — we cannot observe $Y_{T,\text{post}}^{(0)}$. However, it can be made plausible through:

**1. Pre-trend visualisation:** Plot treatment and control group trajectories for several periods before the intervention. If the lines move together with similar slopes, parallel trends is more credible.

**2. Placebo test:** Define a fake intervention date within the pre-period. Run the DiD regression on pre-period data only. If the placebo coefficient $\hat{\beta}_3^{\text{placebo}} \approx 0$ and $p > 0.05$, there is no evidence of a differential pre-trend.

```python
pre_df = df[df.post == 0].copy()
pre_df['fake_post'] = (pre_df['week'] >= 5).astype(int)
pre_df['fake_txp']  = pre_df['treat'] * pre_df['fake_post']
placebo = smf.ols("revenue ~ treat + fake_post + fake_txp", data=pre_df).fit()
# From geo_experiment: placebo beta3 = 0.81, p = 0.86 → plausible
```

### When parallel trends fails

| Threat | Mechanism | Detection |
|---|---|---|
| Differential pre-trend | Treatment region had faster pre-existing growth | Visual inspection; $t$-test on pre-period slopes |
| Anticipation effects | Customers/firms in treatment region changed behaviour before formal date | Check variables in final pre-period weeks |
| Contemporaneous shock | A second event coincided with the intervention in the treatment region | Domain knowledge; check for other outcome variables |
| Spillover (SUTVA violation) | Treatment in one region affected control region outcomes | Use geographically distant controls; check border effects |

---

## 12.6 Confidence Intervals and Inference

The standard error of $\hat{\beta}_3$ reflects uncertainty due to:
- Sampling variation in revenue across regions and weeks
- The number of treatment and control regions (more regions → smaller SE)
- The number of post-period observations (more weeks → smaller SE)

**From the dataset:**
$$\hat{\beta}_3 = 6.25, \quad \text{SE} = 3.15, \quad p = 0.049, \quad 95\% \text{ CI} = [0.09,\ 12.41]$$

The wide CI reflects having only 4 regions per group. In practice, geo-holdout experiments use 10–20 matched region pairs to achieve tighter intervals. Google's `GeoX` framework (Vaver and Koehler, 2011) recommends a minimum of 10 pairs.

**Interpreting a wide CI:**

A CI of [0.09, 12.41] excludes zero — the causal effect is credibly positive. But the width reflects genuine uncertainty about the *magnitude*. The point estimate of 6.25 is the best single estimate; the CI tells us the true effect could plausibly be as small as 0.09 or as large as 12.41. More regions would narrow this.

---

## 12.7 DiD for MMM Validation

### The validation workflow

Marketing Mix Models are fitted to observational data. Because advertising spend and sales are often jointly determined by demand conditions (endogeneity), MMM coefficients may not represent causal effects.

**The standard validation procedure:**

1. **Design the geo-holdout:** Randomly assign geographic regions to treatment (change channel spend) and control (maintain status quo). Choose regions in advance; match on pre-period characteristics if possible.

2. **Run the DiD:** After the holdout period, estimate $\hat{\tau}_{\text{DiD}}$ comparing treatment and control regions before and after the change.

3. **Compare to MMM prediction:** The MMM predicts what should have happened. Agreement within ~20% is typically considered validation. Systematic divergence signals MMM endogeneity.

**From the dataset:**
- MMM predicted: +10.5 units/week
- DiD estimated: +6.25 units/week
- Discrepancy: +4.25 units (MMM overstates by 68%)

The most likely explanation: MMM's price coefficient is inflated because price reductions were historically deployed during high-demand periods, creating positive correlation between price cuts and demand shocks. OLS cannot distinguish this from causal channel effects.

---

## 12.8 Scope and Limitations

**Estimand:** The average treatment effect on the treated (ATT) — not the average treatment effect (ATE). DiD estimates the effect in the regions that were treated, not the population-wide effect of treating all regions.

**External validity:** The estimate applies to regions similar to the treatment regions during the experiment period. Effects may differ in other regions, in different seasons, or for different types of customers.

**Functional form:** The standard DiD assumes additive separability: $Y_{it} = f_i + g_t + \tau D_{it} + \varepsilon_{it}$ where $f_i$ is a unit fixed effect and $g_t$ is a time fixed effect. When the outcome is multiplicative (e.g., percentage changes), log transformation may be more appropriate.

**Dynamic treatment effects:** The standard 2×2 DiD assumes the treatment effect is constant across all post-period weeks. If the effect builds over time (e.g., advertising carryover) or fades, event study designs with week-level interaction terms are more appropriate.

---

## Practitioner Checkpoint

1. State the parallel trends assumption in your own words. Why is it not directly testable?

2. Compute $\hat{\tau}_{\text{DiD}}$ from: treatment pre = 200, post = 240; control pre = 180, post = 210.

3. Write the regression equation. Identify which coefficient is the treatment effect.

4. Your MMM estimates the causal effect of a TV campaign at +$220k/week. A geo-holdout DiD estimates +$180k/week. Is the MMM credible? What would you investigate?

5. The placebo test returns $\hat{\beta}_3^{\text{placebo}} = 4.8$ with $p = 0.02$. What does this imply?

---

### Answer Key

**Q1.** Absent the intervention, treatment and control groups would have experienced the same time trend. It is not directly testable because the counterfactual potential outcome $Y_{T,\text{post}}^{(0)}$ is never observed — we can only observe what actually happened to the treatment regions, not what would have happened.

**Q2.** $(240-200) - (210-180) = 40 - 30 = \mathbf{+10}$.

**Q3.** $Y_{it} = \beta_0 + \beta_1\text{Treat}_i + \beta_2\text{Post}_t + \beta_3(\text{Treat}_i \times \text{Post}_t) + \varepsilon_{it}$. The treatment effect is $\beta_3$.

**Q4.** The MMM estimate (+$220k) is within 22% of the DiD estimate (+$180k) — borderline acceptable. The 22% discrepancy is plausibly attributable to endogeneity: TV spend may have been higher in periods of pre-existing high demand. To investigate: examine whether historical TV spend levels correlate with demand-side predictors (weather, seasonality, competitive events). If so, consider instrumental variable estimation or more careful holdout design.

**Q5.** $p = 0.02$ for the placebo coefficient means there was already a statistically significant differential pre-trend between treatment and control regions before the intervention. The parallel trends assumption is violated. The DiD estimate from the full dataset is likely biased — the treatment and control regions were not following the same trajectory even before the intervention, so the control group is not a valid counterfactual.

---

## Bibliography

- Angrist, J. D., & Pischke, J.-S. (2009). *Mostly Harmless Econometrics*. Princeton University Press. — The standard graduate reference on DiD and causal identification.
- Card, D., & Krueger, A. B. (1994). Minimum wages and employment: A case study of the fast-food industry in New Jersey and Pennsylvania. *American Economic Review*, 84(4), 772–793. — The foundational empirical application of DiD.
- Vaver, J., & Koehler, J. (2011). *Measuring Ad Effectiveness Using Geo Experiments*. Google Inc. — The practitioner's guide to geo-holdout design for marketing measurement.
- Callaway, B., & Sant'Anna, P. H. (2021). Difference-in-differences with multiple time periods. *Journal of Econometrics*, 225(2), 200–230. — The modern extension handling staggered treatment adoption.
