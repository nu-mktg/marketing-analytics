# Agentic Programming Prompts
## All 14 Marketing Analytics Models

Each section contains: (1) a **Context Prompt** to orient the agent before any analysis,
(2) a **Build Prompt** to generate the analysis, and (3) a **Validation Prompt** to
critically evaluate the output before trusting it. Designed for GitHub Codespaces
(Copilot Agent or Claude Projects).

**How to use:** Paste the Context Prompt first. Confirm the agent understands the setup.
Then paste the Build Prompt. Then run the Validation Prompt before accepting any output.

---

## L01 — Bayesian A/B Testing

### Context Prompt
```
I am a growth analyst running a website conversion experiment. Variant A is the
current homepage design. Variant B is a redesigned version. I have visitor counts
and conversion counts for each variant. I want to use Bayesian analysis to estimate
the probability that Variant B has a higher true conversion rate than Variant A,
and to compute a credible interval on the expected lift.

My prior belief (before this experiment) is that conversion rates are typically around
4–6% for this type of product page, based on historical data.
```

### Build Prompt
```
Using the experimental data I will provide:

1. Encode the prior as a Beta distribution with parameters that reflect a prior mean
   of approximately 5% and the equivalent of ~50 prior observations (enough to anchor
   the prior without overwhelming the data).

2. Compute the posterior Beta distribution for each variant by adding observed
   conversions and non-conversions to the prior parameters. Report the posterior
   alpha, beta, and mean for each variant.

3. Use Monte Carlo simulation (100,000 samples) to estimate:
   a) P(B > A): the probability that Variant B's true rate exceeds Variant A's
   b) Expected lift: E[(θ_B - θ_A) / θ_A] expressed as a percentage
   c) 95% credible interval on the absolute lift (θ_B - θ_A)

4. Plot the posterior distributions for both variants on the same axis.
   Mark the posterior means with vertical lines.

Print all key values clearly labelled.
```

### Validation Prompt
```
Before I trust these results, run three checks:

1. PARAMETER CHECK: Does posterior_alpha_B = prior_alpha + conversions_B?
   Does posterior_beta_B = prior_beta + (visitors_B - conversions_B)?
   Report: PASS or FAIL with the actual numbers.

2. EFFICIENCY CHECK: Do the posterior means fall between the prior mean (5%) and
   the raw observed conversion rates? They should — the posterior is a weighted
   average of prior and data.
   Report: PASS or FAIL.

3. INTERPRETATION CHECK: Confirm that P(B > A) is a Bayesian posterior probability
   (a statement about the current state of knowledge given data and prior),
   NOT a long-run frequency ("B would win X% of repeated experiments").
   If your output description uses frequentist language, correct it.
```

---

## L02 — Price Elasticity and OLS

### Context Prompt
```
I am a pricing analyst at a consumer goods company. I have weekly sales data at
the product-store level. Each row is one store-week observation. Columns include:
units_sold, price (the focal product), competitor_price, display_flag (1 if the
product was on a promotional display), and store_id.

I want to estimate the price elasticity of demand using a log-log OLS regression,
controlling for competitor price and display effects. The elasticity coefficient
will inform a pricing recommendation.
```

### Build Prompt
```
Using the pricing dataset:

1. Fit a log-log OLS regression:
   ln(units_sold) = β₀ + β₁·ln(price) + β₂·ln(competitor_price) + β₃·display_flag + ε

2. Report: all coefficients, standard errors, p-values, and R-squared.
   Interpret β₁ as the own-price elasticity and β₂ as the cross-price elasticity.

3. Compute the predicted units sold at the current mean price and at a 10% price increase.

4. Determine whether the current price is above or below the revenue-maximising price.
   The revenue-maximising condition is |own-price elasticity| = 1. Report whether
   a price increase would increase or decrease revenue.

5. Plot log(price) vs. log(units_sold) as a scatter with the OLS line overlaid.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. SIGN CHECK: Is β₁ (own-price elasticity) negative? A positive own-price
   elasticity would indicate demand increases with price — unlikely for a normal good
   and a signal of endogeneity or specification error. Report: PASS or FAIL.

2. ELASTICITY INTERPRETATION: Does your output correctly state that |β₁| > 1 means
   elastic demand (price increase reduces revenue) and |β₁| < 1 means inelastic
   (price increase raises revenue)? Confirm the direction of your revenue recommendation
   is consistent with this.

3. ENDOGENEITY WARNING: If prices in this dataset were set by store managers in
   response to expected demand (lower prices during slow periods), the OLS estimate
   is biased toward zero. Does the regression setup use any instrument or fixed effects
   to address this? If not, flag this as a limitation.
```

---

## L03 — Marketing Mix Modelling

### Context Prompt
```
I am a marketing analyst at a consumer brand with weekly data on advertising spend
across TV, digital, and social channels and the corresponding weekly revenue. I want
to fit a Marketing Mix Model that accounts for adstock (advertising carry-over) and
diminishing returns (Hill function), then use the model to optimise budget allocation
across channels.
```

### Build Prompt
```
Using the MMM dataset:

1. For each channel, transform spend into adstock using the recursion:
   A_t = S_t + λ · A_{t-1}
   Fit λ separately for each channel as a parameter.

2. Apply the Hill transformation to each adstock series:
   H(A) = A^α / (EC50^α + A^α)
   Fit α and EC50 for each channel.

3. Fit OLS: revenue = β₀ + Σ βⱼ · H(Aⱼ) + ε

4. Report:
   a) Fitted λ and EC50 for each channel
   b) Marginal ROI at current spend for each channel (dRevenue/dSpend)
   c) Current channel contribution (% of modelled revenue)

5. Recommend a budget reallocation: which channel should receive more spend
   and which less, based on marginal ROI equalisation?

Print all results clearly labelled.
```

### Validation Prompt
```
Four validation checks:

1. ADSTOCK DIRECTION: Verify A₂ > 0 when S₂ = 0 and A₁ > 0.
   If adstock in a zero-spend week is zero, the recursion is wrong. Report: PASS/FAIL.

2. HILL FUNCTION AT EC50: For each channel, verify H(EC50) ≈ 0.50.
   This must hold for any value of α. Report: PASS/FAIL for each channel.

3. MARGINAL ROI LOGIC: Is the channel with the highest marginal ROI NOT currently
   receiving the most budget? If two channels have equal marginal ROI, the budget
   is already optimal for those two. Confirm the reallocation recommendation is
   consistent with this logic.

4. DECAY INTERPRETATION: Confirm that a higher λ means SLOWER decay (effects
   persist longer), not faster. If your output reverses this, correct it.
```

---

## L04 — Survival Analysis

### Context Prompt
```
I am a customer success analyst at a subscription SaaS company. I have a cohort
dataset where each row is one customer. Columns include: customer_id, tenure_months
(how long they have been a customer), churned (1 = churned, 0 = still active /
censored), and several behavioral covariates (support tickets, logins, integrations).

I want to: (1) estimate the survival function non-parametrically, (2) fit a Cox
proportional hazards model to identify churn predictors, and (3) use the model
to identify customers most at risk in the next 90 days.
```

### Build Prompt
```
Using the survival dataset:

1. Fit a Kaplan-Meier survival curve. Report Ŝ(t) at t = 6, 12, 18, 24 months.
   Plot the KM curve with 95% confidence bands.

2. Compute the conditional churn probability from month 12 to month 24 for
   customers who survived to month 12. Use: 1 - Ŝ(24)/Ŝ(12).

3. Fit a Cox PH model with all available covariates. Report:
   a) Hazard ratio and 95% CI for each covariate
   b) Which covariate has the largest hazard ratio (greatest churn risk)
   c) Which has the smallest (most protective)
   d) Concordance index (C-index)

4. Check the proportional hazards assumption using Schoenfeld residuals or
   log-log survival plots. Flag any covariate that violates PH.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. CENSORING HANDLING: Confirm that customers who are still active (churned = 0)
   are treated as censored, not as churned at their current tenure. If the model
   drops them or treats them as events, the survival estimate is biased downward.
   Report: PASS or FAIL.

2. KM MONOTONICITY: Confirm that Ŝ(t) is non-increasing. Ŝ(6) ≥ Ŝ(12) ≥ Ŝ(18) ≥ Ŝ(24).
   If this is violated, there is a data or implementation error.

3. HAZARD RATIO INTERPRETATION: For a covariate with HR = 1.42, confirm your output
   states the hazard is 42% HIGHER (not 42% lower, not 1.42x lower).
   For HR = 0.68, confirm: 32% lower hazard (not 68% higher).
```

---

## L05 — Markov Chains

### Context Prompt
```
I am a retention analyst. I have monthly snapshots of customer engagement states
for a subscription service. States are: Active, Dormant, and Churned (absorbing).
Each row represents a customer-month-state observation. I want to estimate the
transition probability matrix and use it to project the customer state distribution
forward and compute the steady state.
```

### Build Prompt
```
Using the engagement dataset:

1. Compute the transition matrix P by counting transitions between consecutive months
   and normalising each row to sum to 1. Report the full 3×3 matrix.

2. Starting from the current state distribution v₀ = [fraction Active, fraction Dormant,
   fraction Churned], compute v₁ and v₂ using matrix-vector multiplication:
   v_{t+1} = v_t · P

3. Compute the steady-state distribution π* by solving π·P = π, Σπᵢ = 1.
   Report what fraction of customers will eventually be in each state.

4. Show what happens to the state distribution if P[Dormant → Active] is increased
   by 15 percentage points (a retention campaign effect). Recompute v₁ and v₂.
   What is the change in the Active fraction at t = 2?

Print all results clearly labelled. Rows of P must sum to 1.
```

### Validation Prompt
```
Three validation checks:

1. ROW SUMS: Verify every row of P sums to exactly 1.00 (within floating-point tolerance).
   Report: PASS or FAIL for each row.

2. MULTIPLICATION DIRECTION: Confirm you computed v_{t+1} = v_t · P (row vector times
   matrix), NOT P · v_t (matrix times column vector). These give different results.
   Show the shape of the operation explicitly.

3. ABSORBING STATE: If Churned is an absorbing state, P[Churned → Churned] should equal
   1.0 and P[Churned → Active] = P[Churned → Dormant] = 0. Verify this holds.
   If the steady state has all customers eventually Churned, confirm this is expected
   (not a modelling error) for a chain with a single absorbing state.
```

---

## L06 — Multi-Touch Attribution (Shapley Values)

### Context Prompt
```
I am a marketing analyst at a DTC e-commerce brand. I have customer journey data
where each row is one conversion path. Columns: conversion_id, channels (pipe-separated
sequence of touchpoints), and converted (1/0). Channels include: paid_search, social,
email, display, organic. I want to attribute conversion credit to each channel using
Shapley values and compare to last-touch attribution.
```

### Build Prompt
```
Using the attribution dataset:

1. Compute last-touch attribution: for each conversion, award 100% credit to the
   final channel. Report each channel's share of total conversions.

2. Compute Shapley value attribution using marginal contribution averaging across
   all possible coalitions. The Shapley value for channel c is:
   φ_c = (1/n!) Σ [v(S ∪ {c}) - v(S)] averaged over all orderings.
   Use the channel presence/absence in conversion paths to estimate v(S).

3. Report: a comparison table of last-touch % vs. Shapley % for each channel.
   Verify that Shapley values sum to the overall conversion rate (efficiency axiom).

4. Plot: grouped bar chart comparing last-touch and Shapley attribution side by side.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. EFFICIENCY AXIOM: Do the Shapley values sum to the overall conversion rate?
   Σ φ_c should equal the fraction of paths that converted.
   Report the sum and whether it matches. Report: PASS or FAIL.

2. LAST-TOUCH BIAS CHECK: Paid search and direct channels typically appear last in
   conversion paths and therefore receive inflated last-touch credit. Does your output
   show paid_search or direct with higher last-touch than Shapley share?
   If not, verify the path data has multi-touch paths (single-touch paths make
   last-touch and Shapley identical).

3. CAUSATION WARNING: Confirm your output does NOT claim Shapley values represent
   the causal incremental effect of each channel. Shapley values measure marginal
   co-occurrence in observed paths, not whether removing the channel would cause
   fewer conversions. Flag if this distinction is absent from the output.
```

---

## L07 — Uplift Modelling (T-learner)

### Context Prompt
```
I am a growth analyst at a subscription service. I have data from a randomised
retention campaign experiment. Each row is one customer. Columns: customer_id,
treated (1 = received campaign, 0 = control), converted (1 = renewed, 0 = churned),
and several behavioral covariates (tenure, usage frequency, plan type, support tickets).
I want to estimate heterogeneous treatment effects (CATE) using a T-learner and
identify which customers to target in the next campaign.
```

### Build Prompt
```
Using the uplift experiment dataset:

1. Fit a T-learner: train a separate model (Random Forest, random_state=42) on
   treated customers predicting converted, and a separate model on control customers.
   The predicted CATE is: τ̂(x) = μ̂₁(x) - μ̂₀(x)

2. Report: ATE (mean of τ̂ across all customers), and the distribution of τ̂.

3. Given campaign cost c = $4 and subscription value v = $30, compute the
   profit-maximising targeting threshold: c/v = 0.133.
   Report: fraction of customers with τ̂ > 0.133 (should receive campaign)
           fraction with τ̂ < 0 (sleeping dogs — should NOT receive campaign)

4. Plot: Qini curve (targeting by predicted uplift vs. random targeting).
   Report the Qini coefficient.

5. Plot: distribution of τ̂ values with the threshold marked.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. ATE vs OBSERVED: The ATE from the T-learner should be approximately equal to
   the simple difference (mean converted in treated) - (mean converted in control).
   If they differ by more than 0.02, there is likely a data leak or implementation error.
   Report: both values and whether they agree within 0.02.

2. SLEEPING DOGS: Are there customers with τ̂ < 0? There should be. If τ̂ is positive
   for every customer, the model has likely overfit or the dataset has no sleeping dogs.
   Report: fraction with τ̂ < 0.

3. RANDOMISATION CHECK: Confirm that treatment assignment in the dataset appears random
   by checking covariate balance: are the means of key covariates similar between
   treated and control groups? A large imbalance (>10% difference in means) would
   suggest non-random assignment, which would bias the T-learner estimates.
```

---

## L08 — Customer Lifetime Value (BG/NBD + Gamma-Gamma)

### Context Prompt
```
I am a customer analytics manager at a non-contractual subscription service.
I have transaction data with columns: customer_id, order_date, order_value.
I want to use the BG/NBD model to estimate each customer's probability of still
being alive (P(alive)) and expected future purchases, and the Gamma-Gamma model
to estimate expected spend per transaction, then combine them into a 12-month CLV.
```

### Build Prompt
```
Using the transactions dataset:

1. Compute the RFM summary: for each customer, calculate frequency (number of
   repeat purchases), recency (time since last purchase), T (observation window),
   and mean transaction value. Use the lifetimes library.

2. Fit the BG/NBD model. Report the fitted parameters (r, α, a, b).
   For each customer, compute P(alive) and predicted purchases in the next 12 months.

3. Check the Gamma-Gamma independence assumption: compute the Pearson correlation
   between customer frequency and mean transaction value. If |correlation| > 0.1,
   flag a potential assumption violation.

4. Fit the Gamma-Gamma model. Combine with BG/NBD to compute CLV over 12 months
   for each customer (use a 10% annual discount rate).

5. Report: mean CLV, median CLV, P10 and P90, and the top-10 customers by CLV.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. P(ALIVE) RECENCY TEST: A customer with their last purchase 18+ months ago should
   have substantially lower P(alive) than a customer with the same number of purchases
   but a last purchase 1 month ago. Find two such customers and verify the direction
   is correct. Report: PASS or FAIL.

2. GAMMA-GAMMA ASSUMPTION: Report the frequency-spend correlation. If it is above 0.1
   in absolute value, flag that the Gamma-Gamma CLV estimates are biased. A positive
   correlation (heavy buyers also spend more) means CLV is underestimated by the model.

3. BG/NBD APPLICABILITY: Confirm this product is non-contractual (customers can buy
   at will, not on a fixed subscription with discrete renewal). If this is actually a
   subscription product, flag that BG/NBD is not the right model (use BG/BB instead).
```

---

## L09 — Conjoint Analysis (MNL)

### Context Prompt
```
I am a product strategy analyst. I have stated-preference conjoint survey data
where respondents chose between subscription plan alternatives with varying attributes:
price, content tier (standard/premium), and ad experience (ads/no ads). Each row
is one choice task. I want to estimate a Multinomial Logit model, compute willingness
to pay for each attribute, and simulate market shares under different pricing scenarios.
```

### Build Prompt
```
Using the conjoint survey dataset:

1. Fit a Multinomial Logit model:
   U_j = β_price · price + β_premium · I(premium) + β_no_ads · I(no_ads) + ε
   Use maximum likelihood estimation. Report all coefficients and standard errors.

2. Verify: β_price must be negative (higher price = lower utility for a normal good).
   If positive, the model is misspecified or data is incorrectly structured.

3. Compute willingness to pay (WTP) for each non-price attribute:
   WTP_k = β_k / |β_price|
   Report: WTP for premium content and WTP for no-ads.

4. Simulate market shares for three plan configurations using the softmax rule:
   P(j) = exp(U_j) / Σ exp(U_k)
   Verify shares sum to 1.0.

5. What happens to Premium's market share if the price is raised by $2?

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. SIGN CHECK: Is β_price negative? This is non-negotiable for a correctly specified
   model. Report: PASS or FAIL. If FAIL, check whether the price variable was
   accidentally coded as negative or inverted.

2. WTP FORMULA: Verify WTP_premium = β_premium / |β_price|. Compute it manually
   from the coefficients and confirm it matches the reported value.
   Report: PASS or FAIL.

3. IIA WARNING: The MNL model assumes the ratio of choice probabilities between
   any two plans is unaffected by the presence of other plans (Independence of
   Irrelevant Alternatives). If you are simulating the introduction of a new plan
   that is very similar to an existing one, note that MNL will draw proportional
   share from ALL plans — which may not reflect reality. Flag this limitation if
   the simulation involves a close substitute.
```

---

## L10 — Demand Forecasting (Prophet)

### Context Prompt
```
I am a demand planning analyst. I have weekly revenue time series data with a date
column (ds) and revenue column (y) in thousands of dollars. The series has 3–4 years
of history, clear seasonality, some promotional spikes, and a visible upward trend.
I want to decompose the series, forecast the next 26 weeks, and evaluate model
accuracy using cross-validation.
```

### Build Prompt
```
Using the revenue time series dataset:

1. Fit a Prophet model with:
   - Yearly seasonality: enabled
   - Weekly seasonality: disabled (weekly data, not daily)
   - Changepoint prior scale: 0.05 (default)

2. Plot the component decomposition: trend, yearly seasonality.
   Report: the peak seasonal week of the year and the trough.

3. Forecast the next 26 weeks. Plot: actual history + forecast + 80% and 95%
   prediction intervals.

4. Run time-series cross-validation with initial=104 weeks, horizon=26 weeks,
   period=13 weeks. Report MAPE across all forecast horizons.

5. Identify any changepoints in the trend. Report: dates and direction (up/down).

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. INTERVAL WIDENING: Do the 95% prediction intervals widen as the horizon increases?
   The interval at week 26 should be wider than at week 4. If the intervals are
   constant width, the model is not properly propagating uncertainty.
   Report: PASS or FAIL.

2. SEASONAL PEAK DIRECTION: Does the seasonal component peak in a period that makes
   business sense (e.g., Q4 for consumer goods, summer for fitness)? If the peak
   is at a counterintuitive time, check whether the data has unusual patterns.
   Report the peak week and whether it is plausible.

3. MAPE CONTEXT: A MAPE of X% is not good or bad in isolation. Report the MAPE
   alongside the decision it informs (e.g., safety stock planning, staffing, budget
   allocation) and a brief statement of whether X% is operationally acceptable
   for that use case.
```

---

## L12 — Difference-in-Differences

### Context Prompt
```
I am a marketing analyst measuring the causal effect of a price reduction
experiment. The firm reduced prices by 15% in four geographic regions (treatment)
while maintaining prices in four control regions. I have weekly revenue data for
all regions across 20 weeks (weeks 1–10 pre-intervention, weeks 11–20 post).
I want to estimate the causal revenue impact using DiD and validate the parallel
trends assumption.
```

### Build Prompt
```
Using the geo_experiment dataset:

1. Compute the 2×2 DiD estimate manually from the group-period means:
   DiD = (mean_treat_post - mean_treat_pre) - (mean_ctrl_post - mean_ctrl_pre)
   Also report the naive estimate (treat post - treat pre) and the common trend
   (ctrl post - ctrl pre). Show that DiD = naive - common trend.

2. Fit the DiD regression:
   revenue = β₀ + β₁·treat + β₂·post + β₃·(treat×post) + ε
   Report all four coefficients, standard errors, p-values, and 95% CI for β₃.
   β₃ is the causal treatment effect.

3. Plot: mean weekly revenue for treatment and control groups (weeks 1–20)
   with a vertical line at week 10.5 (intervention start).
   Do the pre-period trends look parallel?

4. Run a placebo test: use pre-period data only (weeks 1–10).
   Define fake_post = (week ≥ 5). Run the DiD regression on pre-period only.
   Report β₃_placebo and its p-value. A non-significant placebo supports
   parallel trends.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. TREATMENT EFFECT COEFFICIENT: Confirm β₃ equals the manually computed DiD estimate.
   They should be identical (the regression is just a computational form of the
   2×2 table). Report: PASS or FAIL.

2. PARALLEL TRENDS: Does the pre-period plot show treatment and control groups
   moving together? If the treatment group was already growing faster before week 11,
   the DiD estimate is biased upward. Report whether the plot looks parallel.

3. PLACEBO P-VALUE: Is the placebo β₃ statistically insignificant (p > 0.05)?
   A significant placebo means there was already a differential trend in the
   pre-period — the parallel trends assumption is likely violated.
   Report: the placebo p-value and whether it supports parallel trends.
```

---

## L13 — Customer Segmentation (k-means)

### Context Prompt
```
I am a customer analytics analyst. I have customer-level feature data including
recency (days since last purchase), frequency (purchases in last 12 months),
average order value, support ticket count, and email open rate. I want to
segment customers into meaningful groups for targeted marketing using k-means
clustering, then profile each segment using downstream model outputs (CLV and
predicted uplift).
```

### Build Prompt
```
Using the customer_features dataset:

1. Standardise the five clustering features (recency_days, frequency,
   avg_order_value, support_tickets, email_open_rate) to zero mean and unit
   variance. Do NOT standardise clv_12m or tau_hat — these are profiling variables.

2. Fit k-means for k = 2 through 8 (random_state=42).
   Plot: WCSS vs k (elbow plot) and mean silhouette score vs k.
   State your recommended k and the reasoning.

3. Fit k-means with the recommended k. For each cluster, report mean values of:
   recency_days, frequency, avg_order_value, clv_12m, tau_hat.
   Assign a descriptive label to each cluster.

4. Report: what fraction of customers in each cluster have tau_hat > 0.133
   (above the retention campaign targeting threshold)?

Print all results clearly labelled. Use random_state=42.
```

### Validation Prompt
```
Three validation checks:

1. STANDARDISATION VERIFICATION: Confirm each of the five clustering features
   has mean ≈ 0 and standard deviation ≈ 1 after standardisation.
   If any feature has std >> 1, standardisation failed.
   Report: means and stds for all five features post-standardisation.

2. SILHOUETTE INTERPRETATION: The silhouette score is the right criterion for
   comparing clustering solutions of different k (WCSS always decreases with k
   and is not a valid comparison). Confirm your k selection was based on the
   silhouette maximum, not the WCSS elbow. Report: which k has the highest
   silhouette, and what that silhouette value is.

3. TARGETING LOGIC: For any cluster with mean tau_hat < 0.133, confirm your
   output does NOT recommend targeting that cluster with the retention campaign,
   regardless of its mean CLV. High CLV does not override a below-threshold
   predicted uplift. Flag if this error appears in the output.
```

---

## L14 — CausalImpact

### Context Prompt
```
I am a marketing analyst measuring the causal effect of a brand awareness campaign
on website sessions. The campaign ran nationally (no untreated geographic regions),
so I cannot use DiD. I have weekly data on website sessions (outcome) and correlated
control series that were not affected by the campaign: branded search impressions
and social mentions. The campaign started at week 21. Pre-period: weeks 1–20.
Post-period: weeks 21–30.
```

### Build Prompt
```
Using the campaign_timeseries dataset:

1. Install and import the causalimpact library.
   Fit CausalImpact with:
   - Outcome: website_sessions
   - Control covariates: branded_search_impressions, social_mentions
   - Pre-period index: rows 0–19 (weeks 1–20)
   - Post-period index: rows 20–29 (weeks 21–30)

2. Plot all three CausalImpact panels: original (with counterfactual),
   pointwise effect, and cumulative effect.

3. Print the summary table showing:
   - Average actual vs predicted sessions in the post-period
   - Cumulative actual vs predicted
   - Average and cumulative causal effects with 95% posterior CIs
   - Posterior probability that the effect is positive

4. Compute pre-period MAPE:
   mean(|actual - fitted| / actual) × 100 over weeks 1–20.
   A MAPE < 10% indicates good pre-period fit.

Print all results clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. PRE-PERIOD FIT: Is the pre-period MAPE below 10%? If above 20%, the control
   covariates do not explain the outcome well — the post-period counterfactual
   will be unreliable. Report: the pre-period MAPE and whether it is acceptable.

2. CI INTERPRETATION: Does the 95% credible interval for the cumulative effect
   exclude zero? If zero is inside the CI, you cannot claim credible evidence of
   a positive causal effect, even if the point estimate is large and positive.
   Report: the CI bounds and whether zero is excluded.

3. COVARIATE CONTAMINATION CHECK: Could either control covariate (branded search
   impressions, social mentions) have INCREASED because of the campaign itself?
   If the campaign drove branded search or social conversation, the covariates are
   contaminated — the model will underestimate the true campaign effect.
   Flag this as a limitation if it is plausible.
```
