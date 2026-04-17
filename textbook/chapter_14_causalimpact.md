# Chapter 14: CausalImpact
## Measuring Marketing Effects on Time Series Outcomes

---

## 14.1 Overview

Difference-in-differences (Chapter 12) solves the causal identification problem when a control group exists — geographic regions, customer cohorts, or product lines not exposed to the intervention. But many marketing interventions affect all observable units simultaneously: a national brand campaign, a platform-wide feature change, an economy-wide pricing adjustment. In these settings, DiD is inapplicable.

CausalImpact (Brodersen, Gallusser, Koehler, Remy, and Scott, 2015) solves this problem using a different type of control: *correlated time series* rather than control units. The method learns the historical relationship between a set of control covariates and the outcome variable, uses that relationship to project a counterfactual trajectory for the post-intervention period, and estimates the causal effect as the difference between what was observed and what was predicted.

**Estimand:** The average and cumulative causal effect of an intervention on a time series outcome during a specified post-period.

**Required data:** Time series of the outcome; time series of control covariates not affected by the intervention; a known intervention date.

---

## 14.2 The Model

### Bayesian Structural Time Series

CausalImpact fits a state-space model to the pre-intervention data:

$$y_t = Z_t^\top \alpha_t + \varepsilon_t, \qquad \varepsilon_t \sim \mathcal{N}(0, \sigma_\varepsilon^2)$$

$$\alpha_{t+1} = T_t \alpha_t + R_t \eta_t, \qquad \eta_t \sim \mathcal{N}(0, Q_t)$$

The state vector $\alpha_t$ contains:
- A **local level** component that tracks a slowly-drifting baseline
- An optional **slope** component that allows trend growth rates to change
- **Regression coefficients** on the control covariates $\mathbf{x}_t$

The Bayesian framework places spike-and-slab priors on the regression coefficients, performing automatic covariate selection — only covariates that genuinely predict the outcome in the pre-period receive non-zero weights.

### Counterfactual construction

In the pre-period (weeks 1–$T_0$): the model is fitted to the data.

In the post-period (weeks $T_0+1$ through $T$): the fitted model is conditioned on the observed control covariates $\mathbf{x}_t$ to produce a posterior predictive distribution over what $y_t$ *would have been* absent the intervention:

$$p(\hat{y}_t^{(0)} \mid \mathbf{y}_{1:T_0}, \mathbf{x}_{1:T})$$

This is a full distribution — not a point estimate. The width of this distribution reflects how uncertain the model is about the counterfactual.

---

## 14.3 The Three Causal Quantities

**Pointwise causal effect:**
$$\hat{\tau}_t = y_t - \hat{y}_t^{(0)}, \quad t = T_0+1, \ldots, T$$

Each $\hat{\tau}_t$ is a posterior distribution. The 95% credible band reflects combined uncertainty from the model fit and the extrapolation to the post-period.

**Cumulative causal effect:**
$$\hat{\tau}_{\text{cum}} = \sum_{t=T_0+1}^{T} \hat{\tau}_t$$

This is the total effect over the post-period — the most decision-relevant quantity. A 95% CI that excludes zero provides credible evidence that the cumulative effect is non-zero.

**Relative causal effect:**
$$\hat{\tau}_{\text{rel}} = \frac{\hat{\tau}_{\text{cum}}}{\sum_{t=T_0+1}^T \hat{y}_t^{(0)}}$$

The fractional lift relative to the predicted counterfactual. Useful for communicating effect size to non-technical audiences.

**Posterior probability:**
$$P(\hat{\tau}_{\text{cum}} > 0 \mid \mathbf{y}_{1:T_0}, \mathbf{x}_{1:T})$$

Analogous to $P(B > A)$ in Bayesian A/B testing: the probability, given the data and model, that the intervention had a positive causal effect.

---

## 14.4 The Identifying Assumption

The entire analysis rests on one assumption:

> **In the post-period, the control covariates $\mathbf{x}_t$ predict the counterfactual outcome $y_t^{(0)}$ using the same relationship estimated in the pre-period.**

This assumption fails in several common circumstances:

### Contaminated covariates

If the intervention *also affects the control covariate*, the covariate is contaminated. In the post-period, the covariate rises partly because of the campaign. The model predicts a high counterfactual (because covariates are high), underestimating the campaign effect.

**Example:** TV advertising generates social media discussion. Using social mentions as a control covariate when measuring a TV campaign will attenuate the estimate.

**Diagnosis:** Plot the control covariate over time. Does it show a visible change at the intervention date?

### Shifted covariate-outcome relationship

If a competitor enters the market at the same time as your campaign, the relationship between branded search and sessions may change — customers who search your brand in the post-period may have lower intent due to competitive alternatives. The pre-period $\boldsymbol{\beta}$ no longer applies.

### No informative covariate

If no time series is correlated with the outcome in the pre-period, the model cannot construct a reliable counterfactual. The result will be very wide credible intervals — honest but not informative. In this setting, CausalImpact is not the right tool.

---

## 14.5 Pre-Period Fit Assessment

The pre-period model fit determines the reliability of the post-period counterfactual. A model that cannot explain the outcome in the pre-period cannot reliably predict it in the post-period.

**MAPE in the pre-period:**
$$\text{MAPE}_{\text{pre}} = \frac{1}{T_0} \sum_{t=1}^{T_0} \frac{|y_t - \hat{y}_t^{(0)}|}{|y_t|} \times 100\%$$

| MAPE | Interpretation |
|---|---|
| < 5% | Excellent — covariates explain the outcome well |
| 5–10% | Good — counterfactual is reliable |
| 10–20% | Moderate — CIs will be wide but the estimate is usable |
| > 20% | Poor — reconsider covariates before interpreting results |

**From the dataset:** Using `branded_search_impressions` as the primary covariate, pre-period MAPE ≈ 3.3%. The covariate explains the pre-period outcome well.

---

## 14.6 CausalImpact as MMM Validation

CausalImpact can validate MMM channel contribution estimates without geographic randomisation:

1. **Hold out one channel** for a defined period (reduce spend to zero or a low fixed level)
2. **Fit CausalImpact** with other channel spend as control covariates
3. **Compare** the CausalImpact estimate to the MMM's prediction for that channel's contribution

If the MMM says TV contributes +\$180k/week and CausalImpact estimates +\$160k/week from the TV holdout, the agreement is close. If CausalImpact estimates +\$80k/week, the MMM's TV coefficient is likely confounded.

**Advantage over DiD validation:** CausalImpact can be applied even when a clean geographic holdout was not designed in advance, using historical variation in channel spend.

---

## 14.7 Scope and Limitations

**Assumes stationarity of the covariate relationship.** If the relationship between covariates and outcome changes over time (seasonality, competitive dynamics, audience composition shifts), the pre-period model will not extrapolate well.

**Does not handle multiple simultaneous interventions well.** If the firm simultaneously launched a TV campaign and changed its homepage design, CausalImpact cannot separate the two effects.

**Requires a sufficient pre-period.** The BSTS model needs enough pre-period observations to estimate its parameters. A minimum of 60 observations (roughly 15 months of weekly data) is typically recommended. Shorter pre-periods produce wider and less reliable counterfactuals.

**Sensitive to covariate selection.** Different choices of control covariates can produce meaningfully different estimates. Always check whether the estimate is robust to reasonable alternative covariate choices.

---

## Practitioner Checkpoint

1. A firm launched a product on Amazon and wants to measure the sales lift attributable to a sponsored product advertising campaign. Sales increased by 40% in the month after launch. Can you use CausalImpact? What covariates would you use?

2. CausalImpact reports: pointwise effects are consistently positive in weeks 21–30, with all 10 individual credible intervals excluding zero. Is this stronger or weaker evidence than a single cumulative CI excluding zero?

3. Pre-period MAPE = 4.2%, but in weeks 18–20 (just before the campaign), the actual outcome diverges from the fitted values by 15% each week. What does this suggest?

4. The single-covariate model (branded search only) gives cumulative = +11,200. The two-covariate model (branded search + social mentions) gives cumulative = +18,400. How do you choose between them, and what does the large discrepancy suggest?

5. MMM estimated the campaign effect at +\$95k/week. CausalImpact estimated +\$130k/week. Which do you report, and how?

---

### Answer Key

**Q1.** CausalImpact is applicable. Good control covariate candidates: (a) sales of a similar competing product not running advertising — if it tracks your pre-campaign sales, it serves as a synthetic control; (b) category-level search volume (searches for the product category, not your brand specifically) — correlated with demand but not directly affected by your sponsored ad. The assumption: these series continue to predict your counterfactual sales for demand-driven reasons, and the advertising only adds incremental sales on top.

**Q2.** Having all 10 individual weekly CIs exclude zero is stronger evidence than a single cumulative CI excluding zero. If each weekly estimate individually shows a positive effect, the cumulative estimate is robust to any single-week anomaly. However, the individual intervals are correlated (they come from the same model and overlapping credible bands), so you should not treat them as 10 independent tests.

**Q3.** The divergence in weeks 18–20 is an anticipation effect warning. If the firm began preparing for the campaign in weeks 18–20 (pre-announcing, seeding influencers, launching teaser content), the outcome may have already started increasing before the formal launch date. The parallel trends analogue for CausalImpact: the model should fit well in the entire pre-period, not just most of it. Consider using week 17 as the intervention date to see whether the divergence disappears.

**Q4.** Compare the two models' pre-period MAPE. The model with lower MAPE is more reliable. If they have similar pre-period fit, the large discrepancy in estimates (+\$7,200 or 64%) strongly suggests that social mentions were contaminated by the campaign — they rose *because of* the campaign, inflating the two-covariate estimate's counterfactual and therefore the effect estimate. The single-covariate model is likely more reliable if branded search was not itself affected.

**Q5.** Report both, along with their assumptions. "Our MMM estimate of +\$95k/week assumes the channel contribution relationship is stable and unconfounded. Our CausalImpact estimate of +\$130k/week assumes that branded search impressions predict our counterfactual sessions for demand-driven reasons unrelated to the campaign. The estimates suggest the campaign had a meaningful positive effect; the discrepancy may reflect confounding in the MMM or covariate contamination in CausalImpact. A geo-holdout DiD would provide a cleaner causal estimate."

---

## Bibliography

- Brodersen, K. H., Gallusser, F., Koehler, J., Remy, N., & Scott, S. L. (2015). Inferring causal impact using Bayesian structural time-series models. *Annals of Applied Statistics*, 9(1), 247–274. — The original CausalImpact paper.
- Scott, S. L., & Varian, H. R. (2014). Predicting the present with Bayesian structural time series. *International Journal of Mathematical Modelling and Numerical Optimisation*, 5(1/2), 4–23. — BSTS foundations.
- Larsen, K. (2016). *Sorry ARIMA, but I'm Going Bayesian*. Stitch Fix Technology Blog. — Accessible practical introduction to BSTS in marketing.
- Google CausalImpact Python package: `pip install causalimpact`. Documentation at `github.com/dafiti/causalimpact`.
