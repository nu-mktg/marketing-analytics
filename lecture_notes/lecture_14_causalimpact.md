# Lecture 14: CausalImpact
## Measuring Marketing Effects on Time Series Outcomes

---

### Overview

**Motivating Problem:** A brand campaign launched on March 1st. Website traffic increased in March. Did the campaign cause the increase, or was it seasonal growth? Difference-in-differences (Lecture 12) requires a geographic control group — but this campaign ran nationally, with no untreated regions.

**Learning Objectives:**

- State the identifying assumption of CausalImpact and explain when it holds and fails
- Interpret the three panels of a CausalImpact plot: original, pointwise, and cumulative
- Read a CausalImpact summary table and extract the key inferential quantities
- Assess pre-period model fit using MAPE and connect it to counterfactual reliability
- Apply CausalImpact to validate MMM channel contribution estimates

**Prerequisites:** Bayesian A/B Testing (Lecture 1), Prophet Forecasting (Lecture 10), DiD (Lecture 12).

---

## PART 1: Concepts and Mathematics

### (~1 hour 40 minutes)

---

### Section 1.1 — The Problem CausalImpact Solves

**DiD limitation:** DiD requires a control group of units not exposed to the intervention. If the intervention affected *all* units simultaneously — a national TV campaign, a platform-wide product change, an economy-wide pricing shift — there is no control group. DiD cannot be applied.

**CausalImpact's approach** (Brodersen et al., 2015): Instead of a geographic control group, use *correlated time series* as a synthetic control. The model learns the relationship between a set of control covariates and the outcome variable in the **pre-period**, then uses that learned relationship to predict what the outcome *would have been* absent the intervention in the **post-period**.

**Control covariate requirements:**

1. Correlated with the outcome in the pre-period
2. *Not themselves affected by the intervention* in the post-period
3. Stable relationship with the outcome across both periods

---

### Section 1.2 — The Structural Time Series Model

CausalImpact fits a **Bayesian structural time series (BSTS)** model to the pre-period data:

$$y_t = \mu_t + \mathbf{x}_t^\top \boldsymbol{\beta} + \varepsilon_t, \qquad \varepsilon_t \sim \mathcal{N}(0, \sigma^2)$$

where $\mu_t$ is a local trend component, $\mathbf{x}_t$ is the vector of control covariates at time $t$, and $\boldsymbol{\beta}$ are regression coefficients estimated from the pre-period.

The model components:

| Component | Role | Key parameter |
|---|---|---|
| **Local level** $\mu_t$ | Captures slowly-drifting baseline | Level variance |
| **Local trend** | Captures trend changes over time | Slope variance |
| **Regression on $\mathbf{x}_t$** | Captures correlation with control series | $\boldsymbol{\beta}$ |
| **Seasonal** (optional) | Captures within-year patterns | Fourier components |

---

### Section 1.3 — The Three Causal Quantities

::: {.formula-box}
**Pointwise effect at time $t$:** $\hat{\tau}_t = y_t - \hat{y}_t^{(0)}$

**Cumulative effect over post-period:** $\hat{\tau}_{\text{cum}} = \sum_{t \in \text{post}} \hat{\tau}_t$

**Relative effect:** $\hat{\tau}_{\text{rel}} = \hat{\tau}_{\text{cum}} \;/\; \sum_{t \in \text{post}} \hat{y}_t^{(0)}$
:::

Each quantity is a **posterior distribution** — not a single point estimate. The BSTS model produces a full probability distribution over the counterfactual $\hat{y}_t^{(0)}$, which propagates to a distribution over the causal effect.

---

### Section 1.4 — The Three-Panel Plot

**Panel 1 — Original:**  
Solid line: observed $y_t$. Dashed line: predicted counterfactual $\hat{y}_t^{(0)}$ with 95% posterior band.
- Pre-period: lines should overlap closely (model fit check)
- Post-period: the gap between observed and predicted is the estimated causal effect

**Panel 2 — Pointwise:**  
$\hat{\tau}_t = y_t - \hat{y}_t^{(0)}$ with 95% credible band. If zero is inside the band throughout the post-period, the evidence for a positive effect is weak.

**Panel 3 — Cumulative:**  
Running sum $\sum_{s=21}^t \hat{\tau}_s$. The most decision-relevant quantity: "After 10 weeks of the campaign, we estimate it generated X additional sessions." The final cumulative value with its CI is the headline result.

---

### Section 1.5 — The Identifying Assumption

$$\boxed{\text{In the post-period, } \mathbf{x}_t \text{ predicts } y_t^{(0)} \text{ using the pre-period relationship } \boldsymbol{\beta}.}$$

This assumption fails when:

**1. The covariate itself is affected by the intervention.**  
If your control covariate is "branded search impressions" and the campaign increases branded search, then $\mathbf{x}_t$ in the post-period is contaminated by the treatment. CausalImpact will systematically *underestimate* the campaign effect — the covariate rises partly because of the campaign, so the "predicted" counterfactual is too high.

**2. The covariate-outcome relationship changes in the post-period.**  
If a competitor launches simultaneously and changes the relationship between search impressions and site visits, the pre-period $\boldsymbol{\beta}$ no longer applies.

**3. No good covariate exists.**  
If no series is correlated with the outcome, the model reduces to a time series extrapolation with very wide posterior intervals — honest but not very informative.

---

### Section 1.6 — Pre-Period Model Fit Assessment

Before trusting the post-period counterfactual, verify the pre-period fit:

$$\text{MAPE}_{\text{pre}} = \frac{1}{T_{\text{pre}}} \sum_{t=1}^{T_{\text{pre}}} \frac{|y_t - \hat{y}_t^{(0)}|}{|y_t|} \times 100\%$$

| Pre-period MAPE | Interpretation |
|---|---|
| < 5% | Excellent fit — counterfactual is reliable |
| 5–10% | Good fit |
| 10–20% | Moderate — post-period CIs will be wide |
| > 20% | Poor fit — counterfactual is unreliable; reconsider covariates |

**Dataset result:** With `branded_search_impressions` as covariate, pre-period MAPE ≈ 3.3%. This indicates the covariate explains the outcome well in the pre-period, and the post-period counterfactual is credible.

---

### Section 1.7 — CausalImpact vs. DiD

| Criterion | CausalImpact | Difference-in-Differences |
|---|---|---|
| Control group required? | No — uses control *time series* | Yes — needs untreated *units* |
| What it uses as counterfactual | Correlated pre-period time series | Other geographic regions or cohorts |
| Handles time series dynamics? | Yes — models trend and seasonality explicitly | Basic DiD assumes stable trends |
| Requires parallel trends? | No — but requires stable covariate relationship | Yes |
| Best suited for | National campaigns; product launches; pricing changes | Geo-holdouts; natural experiments with clear groups |

**The practical rule:** If you have a clean geographic or unit-level control group, use DiD. If the intervention was nationwide and you have correlated control time series, use CausalImpact.

---

### Part 1 Checkpoint

1. A firm launches a national email campaign. No regions are untreated. You have data on branded search clicks (correlated with email opens). Is CausalImpact applicable? What is the key assumption you are making?

2. The CausalImpact summary reports: cumulative effect = +14,200 with 95% CI [2,100, 26,300]. What does this mean in plain language?

3. Pre-period MAPE = 22%. What does this imply about the post-period estimate?

4. You use "display ad impressions" as a control covariate. The campaign you are measuring *also increased* display ad impressions. What is the consequence for the CausalImpact estimate?

5. DiD estimated the TV campaign effect at +\$140k/week. CausalImpact estimated +\$190k/week using social mentions as the control covariate. Which do you trust more, and what might explain the discrepancy?

---

### Checkpoint Answer Key

**Q1.** Yes — CausalImpact is applicable. The identifying assumption: branded search clicks in the post-period predict the counterfactual website sessions using the same relationship estimated in the pre-period, for reasons unrelated to the email campaign. If the email campaign *also* increased branded search (by driving recipients to search the brand name), the assumption is violated and the estimate will be attenuated.

**Q2.** Over the post-period, the model estimates that the intervention caused between 2,100 and 26,300 additional sessions (95% posterior credible interval), with a best estimate of 14,200. The interval excludes zero, providing credible evidence of a positive causal effect. The wide interval reflects uncertainty in the counterfactual — more pre-period data or better covariates would narrow it.

**Q3.** A MAPE of 22% in the pre-period means the control covariates explain only about 78% of the variance in the outcome. The post-period counterfactual is unreliable — it will have very wide credible intervals. The first step: examine whether different or additional control covariates improve the pre-period fit. If no good covariates exist, CausalImpact is not the right tool for this setting.

**Q4.** The display impressions covariate is contaminated — it rises partly *because* of the campaign. In the pre-period, the model learns that high display impressions predict high sessions. In the post-period, both display and sessions rise together. The model attributes session growth to the display increase rather than the campaign, systematically underestimating the causal effect.

**Q5.** DiD is generally more credible when a clean geographic holdout was properly randomised, because it makes fewer assumptions. CausalImpact's estimate of +\$190k could be inflated if the social mentions covariate was affected by the TV campaign (TV advertising often generates social discussion, contaminating the covariate). The discrepancy (+\$50k, or 36%) warrants investigation: check whether social mentions increased in the post-period of the TV campaign, and if so, rerun CausalImpact without that covariate.

---

## PART 2: Application

### Section 2.1 — Dataset

`campaign_timeseries.csv`: 30 weekly observations.

| Variable | Description |
|---|---|
| `week` | Week number (1–30) |
| `website_sessions` | Outcome: weekly website visits |
| `branded_search_impressions` | Control covariate 1: branded keyword search volume |
| `social_mentions` | Control covariate 2: brand mentions on social media |
| `post` | 1 = post-campaign (weeks 21–30), 0 = pre-campaign |

**Campaign:** Brand awareness campaign launched at week 21. True causal effect: +1,200 sessions/week.

---

### Part A: Math Questions (Individual)

Use the summary statistics printed at the top of the notebook.

```python
# Q1. Average weekly sessions in the post-period (weeks 21-30)
q1_post_mean_sessions = None
print(f'Q1: {q1_post_mean_sessions}')

# Q2. Total sessions in the post-period (sum, weeks 21-30)
q2_post_total_sessions = None
print(f'Q2: {q2_post_total_sessions}')

# Q3. Are mean post-period sessions higher than mean pre-period sessions? (True/False)
q3_post_gt_pre = None
print(f'Q3: {q3_post_gt_pre}')

# Q4. In which week do website_sessions reach their maximum?
q4_peak_week = None
print(f'Q4: {q4_peak_week}')

# Q5. In CausalImpact, the prediction interval for week 30 (the final post week)
# is wider than for week 21 (the first post week). Is this expected? (True/False)
q5_ci_grows_with_horizon = None
print(f'Q5: {q5_ci_grows_with_horizon}')
```

### Part B: Agent Analysis (Collaboration permitted)

```
Dataset: campaign_timeseries.csv
Campaign start: week 21.
Pre-period index: 0–19 (weeks 1–20).
Post-period index: 20–29 (weeks 21–30).

Please:
1. Install: pip install causalimpact
   Fit CausalImpact with branded_search_impressions and social_mentions as
   control covariates. Report the summary table (average and cumulative effects
   with 95% CIs, posterior probability of positive effect).

2. Plot the three panels (original, pointwise, cumulative).

3. Compute the pre-period MAPE:
   mean(|actual - fitted| / actual) × 100 over weeks 1-20.

4. Refit with only branded_search_impressions as the single covariate.
   Does the cumulative effect change by more than 10%?

Print all results clearly labelled.
```

### Part C: Interpretation Questions (Collaboration permitted)

```python
# Q11. The key identifying assumption of CausalImpact is:
# a) The control time series perfectly predicts the counterfactual
# b) The control covariates continue to predict the outcome in the post-period
#    using the pre-period relationship, for reasons unrelated to the campaign
# c) The outcome time series is stationary
# d) The campaign affects all time series simultaneously
q11 = None

# Q12. CausalImpact reports: cumulative effect = +12,400, 95% CI = [-200, 25,000].
# The correct interpretation is:
# a) The campaign worked — 12,400 sessions is a large positive effect
# b) The CI includes zero; the positive causal effect is not statistically
#    credible at the 95% level
# c) The CI is too wide — CausalImpact always overestimates uncertainty
# d) 95% CI means there is a 95% probability the effect is positive
q12 = None

# Q13. Pre-period MAPE = 28%. This indicates:
# a) The model fit is excellent — 28% is within the acceptable threshold
# b) The campaign started earlier than week 21 (anticipation effect)
# c) The control covariates do not explain the outcome well in the pre-period;
#    the post-period counterfactual will have wide and unreliable uncertainty
# d) The dataset contains a structural break in the pre-period
q13 = None
```
