# Lecture 7: Uplift Modeling
## Targeting the Persuadables, Not the Sure Things

---

### Overview

**Business question:** You have a $5 retention campaign. Which customers should receive it — and which customers will actually change their behavior because of it (vs. those who would renew anyway or those who would churn regardless)?

**What you will be able to do:**
- Explain the four customer types in uplift modeling
- Compute a simple T-learner CATE estimate
- Apply the profit-maximizing targeting threshold
- Interpret Qini curves and coefficients
- Identify when treatment assignment must be randomized for valid estimates

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Expected Value

Expected value = probability × outcome, summed across all scenarios.

**Example:** Campaign profit per customer = uplift × conversion_value − campaign_cost
> If τ(X) = 0.15, value = $30, cost = $4: Expected profit = 0.15 × 30 − 4 = 4.50 − 4 = **$0.50**
> If τ(X) = 0.08, value = $30, cost = $4: Expected profit = 0.08 × 30 − 4 = 2.40 − 4 = **−$1.60** (negative!)

The breakeven threshold: τ(X) × value = cost → τ(X) = cost/value = 4/30 = **0.133**

---

### Section 1.2 — Business Motivation

---

#### Duolingo's 600 Streak Experiments

Jackson Shuttleworth is a Group Product Manager at Duolingo leading the retention team. Over four years, his team ran over 600 experiments on a single feature: the streak. That is approximately one experiment every other day.

One finding exemplifies the uplift insight: changing the call-to-action button from "Continue" to "Commit to my goal" was a massive win for retention. But the users who responded to this change were not the users most likely to continue anyway — they were users on the margin between re-engaging and quietly lapsing. The users already highly committed to their goal did not need the new framing; they were coming back regardless.

A model that predicts "who is likely to be active next week" would have ranked the committed users highest and sent them the message. Uplift modeling ranks differently: who will change their behavior because of the message? That is a fundamentally different question.

Lauryn Isford, who led growth at Airtable after years at Facebook, adds a strategic point:

> *"An activation rate that falls in a lower percentage range, maybe 5–15%, is better than one that falls in a higher percentage range — because it means there's likely much higher correlation with long-term retention."*

Targeting precision over volume is the uplift mindset.

---

### Section 1.3 — Conceptual Framework

---

#### The Four Customer Types

In any campaign, customers fall into four segments based on how they would respond to treatment:

| Type | Without campaign | With campaign | Should you target? |
|---|---|---|---|
| **Sure Thing** | Converts anyway | Converts | No — wasted spend |
| **Persuadable** | Does not convert | Converts | Yes — this is who you want |
| **Lost Cause** | Does not convert | Does not convert | No — cannot be helped |
| **Sleeping Dog** | Converts anyway | Does NOT convert | Definitely no — makes things worse |

Standard propensity models (predict conversion probability) identify Sure Things as "high priority" customers. Uplift models identify Persuadables — customers whose behavior actually changes because of your intervention.

Sleeping Dogs are real and surprisingly common in subscription renewal campaigns. Sending a "we'll miss you" message sometimes reminds customers they wanted to cancel.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: The T-Learner

The T-Learner estimates the Conditional Average Treatment Effect (CATE) by training two separate models:
- μ₁(X): predicted conversion probability if treated, trained on treated customers only
- μ₀(X): predicted conversion probability if NOT treated, trained on control customers only

**CATE estimate for customer i:** τ̂(Xᵢ) = μ₁(Xᵢ) − μ₀(Xᵢ)

**Critical requirement:** Treatment must be randomly assigned. Without randomization, treated customers systematically differ from control customers in unobserved ways. The difference μ₁(X) − μ₀(X) would capture both the treatment effect and the selection bias — confounding the estimate.

> ### 🔍 Deep Dive: The Potential Outcomes Framework
> The formal framework defines two potential outcomes for each customer: Y(1) = what they would do if treated, Y(0) = what they would do if not treated. The true individual treatment effect is Y(1) − Y(0). The fundamental problem: you can only observe one of Y(1) or Y(0) for each person — never both. Random assignment ensures that E[Y(0)|T=1] = E[Y(0)|T=0], making the control group a valid counterfactual for the treatment group. Without this, the T-learner estimate is biased by an unknown amount.

---

#### Part B: The Targeting Threshold

Targeting all customers with τ̂(X) > 0 maximizes conversions but not profit. The profit-maximizing threshold is:

> Target if τ̂(X) > **cost / value**

**Example:** Campaign cost = $4, conversion value = $30. Threshold = 4/30 = 0.133.

Customers with 0 < τ̂(X) < 0.133 generate positive incremental conversions but negative expected profit. Targeting them costs more than they return.

---

#### Part C: Qini Coefficient

The Qini coefficient measures model ranking quality — how much of the maximum possible incremental lift is captured when you target customers in rank order of predicted uplift (highest τ̂ first).

**Qini = 0:** The model has no predictive power for uplift (no better than random targeting)
**Qini = 1:** Perfect — the model ranks all Persuadables before all non-Persuadables
**Typical good range:** 0.30–0.60 in practice

**Important:** The Qini coefficient is NOT the fraction of Persuadables the model correctly identifies. It is a measure of ranking quality, similar to AUC but for uplift.

---

### Part 1 Checkpoint

1. ATE = treated_rate − control_rate. Treated rate = 0.24, control rate = 0.16. What is the ATE?

2. Campaign cost = $4, conversion value = $25. What is the minimum profitable uplift threshold?

3. A customer has τ̂(X) = 0.05 and campaign cost = $4, value = $25. Should you target them?

4. Qini coefficient = 0.42. What does this mean in plain English?

5. The treatment team forgot to randomize — instead, they sent the campaign to high-value customers (who tend to have higher renewal rates anyway). Will τ̂ estimates from the T-learner be too high, too low, or unbiased?

---

### Checkpoint Answer Key

**Q1.** ATE = 0.24 − 0.16 = **0.08** (8 percentage points).
*Common wrong answer:* 0.24/0.16 = 1.5 (this is the lift ratio, not the ATE). ATE is an absolute difference in rates, not a ratio.

**Q2.** Threshold = 4/25 = **0.16** (16 percentage points of incremental conversion needed to break even).
*Common wrong answer:* 4/25 = 0.16, but stated as "16%" without distinguishing it from a probability. The threshold is a probability (0.16 = 16 pp of incremental conversion rate).

**Q3.** **No.** Expected profit = 0.05 × $25 − $4 = $1.25 − $4 = **−$2.75**. The customer has positive predicted uplift but insufficient uplift to justify the cost. The threshold is 0.16; τ̂ = 0.05 < 0.16.
*Common wrong answer:* Yes, because τ̂ > 0 means the campaign helps them. Positive τ̂ is necessary but not sufficient for profitable targeting.

**Q4.** The uplift model captures approximately **42% of the maximum possible incremental lift** when customers are targeted in descending order of predicted τ̂. It is better than random targeting (Qini = 0) but not perfect (Qini = 1). It does not mean the model identifies 42% of persuadable customers.
*Common wrong answer:* The model correctly identifies 42% of persuadable customers in the dataset.

**Q5.** **Too high** (biased upward). High-value customers have higher baseline renewal rates (μ₀ is higher). The T-learner estimates μ₀ from the control group, which in this case consists of lower-value customers. It underestimates the control outcome for high-value treated customers, making the treatment effect appear larger than it is.
*Common wrong answer:* Unbiased, because T-learner uses separate models for treated and control. The models are estimated on non-comparable groups, so the difference μ₁(X) − μ₀(X) is biased.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work through Parts B–C together)


---

**Problem Statement**

A subscription company ran a randomized experiment: 50% of customers received a "loyalty upgrade offer" (treatment), 50% received nothing (control). Here are 12 customers from the experiment with their features, treatment assignment, and outcome:

| ID | Age | Tenure (yrs) | T | Y | Type (fill in) |
|---|---|---|---|---|---|
| C1 | 34 | 5 | 1 | 1 | ? |
| C2 | 52 | 9 | 1 | 1 | ? |
| C3 | 28 | 1 | 1 | 0 | ? |
| C4 | 45 | 7 | 1 | 1 | ? |
| C5 | 38 | 3 | 1 | 0 | ? |
| C6 | 29 | 1 | 1 | 0 | ? |
| C7 | 55 | 10 | 0 | 1 | ? |
| C8 | 33 | 2 | 0 | 0 | ? |
| C9 | 47 | 8 | 0 | 0 | ? |
| C10 | 40 | 6 | 0 | 1 | ? |
| C11 | 31 | 2 | 0 | 0 | ? |
| C12 | 50 | 7 | 0 | 0 | ? |

**Part A:** Compute the observed ATE from this data (treatment conversion rate minus control conversion rate).

**Part B:** After running the T-learner, predicted uplift scores are:

| ID | $\hat{\tau}(X)$ | T | Y |
|---|---|---|---|
| C2 | 0.55 | 1 | 1 |
| C4 | 0.48 | 1 | 1 |
| C1 | 0.42 | 1 | 1 |
| C9 | 0.35 | 0 | 0 |
| C10 | 0.28 | 0 | 1 |
| C7 | 0.18 | 0 | 1 |
| C5 | 0.12 | 1 | 0 |
| C12 | 0.08 | 0 | 0 |
| C3 | 0.02 | 1 | 0 |
| C8 | -0.05 | 0 | 0 |
| C6 | -0.08 | 1 | 0 |
| C11 | -0.12 | 0 | 0 |

What is the mean predicted uplift across all 12 customers? How does this compare to the ATE from Part A?

**Part C:** Identify each treated customer (T=1) by type: persuadable (Y=1, would have been 0 without treatment), sure thing (Y=1 regardless), sleeping dog (Y=0, would have been 1 without), or lost cause (Y=0 regardless). Note: since we cannot observe the counterfactual, use the predicted uplift as a guide.

---

**Full Solution**

**Part A: Observed ATE**

Treatment group (T=1): C1, C2, C3, C4, C5, C6 → conversions: C1, C2, C4 → rate = 3/6 = **50%**

Control group (T=0): C7, C8, C9, C10, C11, C12 → conversions: C7, C10 → rate = 2/6 = **33%**

$$\text{ATE} = 0.50 - 0.33 = \mathbf{0.17} \approx 17 \text{ percentage points}$$

**Part B: Mean Predicted Uplift**

$$\bar{\hat{\tau}} = \frac{0.55+0.48+0.42+0.35+0.28+0.18+0.12+0.08+0.02+(-0.05)+(-0.08)+(-0.12)}{12} = \frac{2.23}{12} \approx \mathbf{0.186}$$

This is close to the observed ATE of 0.17. The small difference is due to sampling variation with only 12 customers. In a large dataset, the mean predicted uplift from the T-learner should converge to the observed ATE. This is the **ATE check** — a key validation of the model.

**Part C: Customer Type Classification**

Using predicted uplift as a guide:

| Customer | T | Y | $\hat{\tau}$ | Probable Type | Reasoning |
|---|---|---|---|---|---|
| C1 | 1 | 1 | 0.42 | Persuadable | High uplift + converted |
| C2 | 1 | 1 | 0.55 | Persuadable | Highest uplift + converted |
| C3 | 1 | 0 | 0.02 | Lost Cause | Near-zero uplift, no conversion |
| C4 | 1 | 1 | 0.48 | Persuadable | High uplift + converted |
| C5 | 1 | 0 | 0.12 | Lost Cause | Low uplift, no conversion |
| C6 | 1 | 0 | −0.08 | Sleeping Dog | Negative uplift, no conversion (might have converted without offer) |

Note: C7 (T=0, Y=1, $\hat{\tau}=0.18$) converted in the control group despite moderate predicted uplift — this is consistent with being a Sure Thing who would convert regardless.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**Mean predicted uplift vs. observed ATE:** These should be close (within 2–3 percentage points for a well-fitted model on a large sample). A large divergence suggests either a model fitting problem or that the experimental randomization was imperfect.

**The Qini curve shape:**
- Steep initial rise, then plateau: most persuadables are concentrated in the top-ranked customers — excellent model
- Roughly diagonal: model barely outperforms random targeting — poor model
- Below diagonal initially: model ranks sleeping dogs (negative uplift) near the top — dangerous

**Feature importances from the T-learner:** The features that matter in $\hat{\mu}_1$ but not $\hat{\mu}_0$ (or vice versa) drive the uplift model. If age is important in $\hat{\mu}_1$ but not $\hat{\mu}_0$, then age moderates the treatment effect — some age groups respond while others do not.

**What to verify before trusting the agent output:**
1. Is mean predicted uplift close to observed ATE?
2. Is the Qini coefficient positive? If negative, the model is inverting the ranking
3. Do feature importances make business sense? High-tenure customers might be more persuadable if they are already loyal but need a push
4. Is the Qini coefficient higher for the test set than training set? If much lower, the model is overfitting

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/uplift_data.csv` + `homework_07.ipynb`

---

#### Part A: Math Questions (no agent required)

**Q1.** In an experiment, 2,000 customers were treated and 2,000 were controls. Treated conversion rate = 0.22. Control conversion rate = 0.14. What is the ATE?
```python
q1_ate = None
```

**Q2.** The treatment cost is \$2 per customer. The value of a conversion is \$30. What is the minimum uplift τ(X) that justifies targeting a customer?
```python
q2_min_uplift = None
```

**Q3.** A customer has predicted uplift $\hat{\tau}(X) = 0.12$ and the minimum uplift threshold is 0.067. Should this customer be targeted?
```python
q3_target = None  # True or False
```

**Q4.** If there are 10,000 customers with an average predicted uplift of 0.09, and you target all of them with a \$2 cost and \$30 conversion value, what is the expected net profit from the campaign? (Expected net profit = N × τ̄ × value - N × cost)
```python
q4_expected_profit = None
```

**Q5.** True or False: A customer with Y=1 who was in the treatment group must be a "persuadable."
```python
q5_treated_converter_persuadable = None  # True or False
```

**Q6.** The T-learner trains model $\hat{\mu}_0$ on control customers only. A new customer has features X_new = [age=42, tenure=5]. The model predicts $\hat{\mu}_0(X_{new}) = 0.08$. What does this represent in business terms?
```python
# Enter "a", "b", or "c"
# (a) The customer will convert with 8% probability if treated
# (b) The customer will convert with 8% probability if NOT treated (baseline conversion probability)
# (c) The uplift from treating this customer is 8%
q6_mu0_meaning = None
```

**Q7.** Under random assignment, T is independent of X. Why does this matter for the T-learner? In one sentence, explain what goes wrong if self-selected customers join the treatment group.
```python
# Free response — participation credit; no autograded answer
q7_random_assignment = "write your answer here as a string"
```

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I am running an uplift analysis for a retail bank promotion experiment.

uplift_data.csv contains one row per customer:
- customer_id
- treated: 1 if customer received the promotion, 0 if control
- converted: 1 if customer accepted a cross-sell offer, 0 otherwise
- age: customer age in years
- account_tenure_years: years as a customer
- avg_monthly_balance: average balance in dollars
- num_products: number of existing products
- last_login_days_ago: days since last login
- credit_score_band: "low", "medium", or "high"

Please:
1. Verify random assignment: compare feature means between treated and
   control groups. Report any significant differences.
2. Compute the observed ATE (treated rate - control rate).
3. Implement a T-learner using Random Forest classifiers for both
   mu_1 and mu_0. Use 80/20 train/test split with random_state=42.
4. Compute predicted uplift tau_hat = mu_1(X) - mu_0(X) for the test set.
5. Report: mean tau_hat, and verify it is close to the observed ATE.
6. Plot the Qini curve and compute the Qini coefficient.
7. Print the top 5 most important features from mu_1 and mu_0 separately.
```

**Q8.** What is the observed ATE? Round to 3 decimal places.
```python
q8_observed_ate = None
```

**Q9.** What is the mean predicted uplift (tau_hat) on the test set? Round to 3 decimal places.
```python
q9_mean_tau_hat = None
```

**Q10.** What is the Qini coefficient? Round to 2 decimal places.
```python
q10_qini_coef = None
```

**Q11.** What is the single most important feature in model mu_1 (the treatment model)?
```python
q11_top_feature_mu1 = None  # feature name as string
```

---

#### Part C: Interpretation Questions

**Q12.** The Qini coefficient is 0.31. A colleague says: "Our model only works 31% of the time." Is this correct?
- (a) Yes — 31% accuracy is poor for a marketing model
- (b) No — the Qini coefficient measures ranking quality (how well persuadables are ranked first), not prediction accuracy
- (c) Yes — any Qini below 0.5 means the model performs worse than random
- (d) No — the Qini coefficient should always be above 0.9 for marketing models
```python
q12 = None  # "a", "b", "c", or "d"
```

**Q13.** The most important feature in mu_1 (treatment model) is avg_monthly_balance, but this feature is much less important in mu_0 (control model). What does this suggest about who responds to the promotion?
- (a) High-balance customers have higher baseline conversion rates
- (b) High-balance customers respond more strongly to the promotion — the treatment effect is larger for them
- (c) Balance is not useful for predicting conversions at all
- (d) The model is overfitted and the feature importance should be ignored
```python
q13 = None  # "a", "b", "c", or "d"
```

**Q14.** You want to target the top 20% of customers by predicted uplift. Based on the Qini curve, approximately what fraction of total incremental conversions would this targeting strategy capture?
- (a) Exactly 20% — targeting 20% always captures 20% of incremental conversions
- (b) More than 20% — if the model is good, the top 20% are disproportionately persuadables
- (c) Less than 20% — the model always underperforms random
- (d) Exactly 100% — all persuadables are in the top 20%
```python
q14 = None  # "a", "b", "c", or "d"
```

---

### Common Misconceptions

**1. "Uplift modeling requires a much larger sample than a standard A/B test."**
Uplift modeling typically requires more data than a simple ATE estimate because it needs to estimate heterogeneous effects across many customer segments. As a rough guide, you need enough data in both the treatment and control groups to reliably fit separate models. For simple T-learners, 5,000+ in each group is a practical minimum.

**2. "A customer with Y=1 in the treatment group is definitely a persuadable."**
They might be a Sure Thing — someone who would have converted anyway. Without observing their counterfactual, we cannot know for certain. The T-learner estimates the probability of each type based on similar customers in the control group.

**3. "We should always target customers with τ̂(X) > 0."**
The correct threshold is τ̂(X) > cost/value, not τ̂(X) > 0. A customer with predicted uplift of 0.01 (1 percentage point) does not justify a \$5 campaign cost unless the conversion value is over \$500.

**4. "The T-learner works without an experiment if the data is rich enough."**
Without random assignment, the T-learner is biased. Observational data (where treatment assignment is not random) requires more sophisticated methods (instrumental variables, difference-in-differences, propensity score matching) to estimate causal effects. The T-learner's validity relies entirely on random assignment.

**5. "A high Qini coefficient means the campaign is profitable."**
The Qini coefficient measures ranking quality, not profitability. A model that perfectly ranks customers by uplift would have a Qini of 1.0, but if the ATE is near zero, targeting anyone may not be profitable regardless of the ranking quality.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Compares treated/control feature means | Section 1.2 — verifying random assignment |
| Computes treated rate − control rate | Section 1.4A — observed ATE |
| Fits $\hat{\mu}_1$ on treated data | Section 1.3 — T-learner Step 2 |
| Fits $\hat{\mu}_0$ on control data | Section 1.3 — T-learner Step 3 |
| Computes $\hat{\tau}(X) = \hat{\mu}_1 - \hat{\mu}_0$ | Section 1.3 — T-learner Step 4 |
| Checks mean $\hat{\tau}$ vs observed ATE | Section 1.4B — ATE check (unbiasedness) |
| Plots Qini curve, computes coefficient | Section 1.4C — Qini curve construction |

**What to verify:**
1. Mean $\hat{\tau}$ ≈ observed ATE (within 0.02 for large samples)
2. Qini coefficient positive
3. Feature importances directionally sensible
4. No large feature imbalances between treated/control (would indicate poor randomization)
