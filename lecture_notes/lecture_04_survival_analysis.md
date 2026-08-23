# Lecture 4: Survival Analysis and Churn Prediction
## When Will Customers Leave — and Who Is Most at Risk?

---

### Overview

**Business question:** You have 50,000 subscribers. Which ones are likely to churn next month, and how quickly does the overall subscriber base decay?

**What you will be able to do:**
- Compute a Kaplan-Meier survival curve step by step
- Interpret conditional vs. unconditional churn probabilities
- Read a Cox model output and understand what hazard ratios mean
- Handle censored observations correctly
- Identify when the proportional hazards assumption may be violated

**Prerequisites:** Basic probability from Lecture 1. Everything else is built here.

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Conditional Probability

P(A | B) means "the probability of A given that B has already happened."

**Key formula:** P(A | B) = P(A and B) / P(B)

**Why this matters for churn:** "60% of customers churned by month 24" (unconditional) is a very different statement from "60% of customers who survived to month 12 churned by month 24" (conditional). The first includes early churners; the second asks about long-tenured customers specifically.

---

#### Tool 2: Product of Fractions

The Kaplan-Meier estimator multiplies fractions together step by step:

> Ŝ(t) = (surviving at t₁ / at-risk at t₁) × (surviving at t₂ / at-risk at t₂) × ...

Each fraction is slightly less than 1. Multiplying them gives the cumulative survival probability.

---

### Section 1.2 — Business Motivation

---

#### The Netflix Story

Gibson Biddle, VP of Product at Netflix (2005–2010), described the company's core metric this way:

> *"When Netflix started, 10% canceled every month. By 2005, it was about 4.5% per month."*

At 10% monthly churn, the average subscriber stayed approximately 10 months. At 4.5%, they stayed approximately 22 months. This difference translated directly into how much Netflix could spend to acquire each subscriber — and therefore how aggressively they could grow.

Every product decision at Netflix was evaluated against one question: "Will this improve retention?" Not "will this improve engagement" or "will this increase NPS" — retention, because that was the metric most directly tied to business value.

Jess Lachs, who built the data team at DoorDash, articulates the broader principle:

> *"Retention is a terrible thing to goal on. It's almost impossible to drive in a meaningful way in the short term. Find a short-term metric you can measure that drives a long-term output."*

Survival analysis gives you both: the long-run picture (the survival curve) and the short-run leading indicators (Cox model predictors of churn timing).

---

### Section 1.3 — Conceptual Framework

---

#### The Censoring Problem

Traditional regression requires observing the outcome for every observation. But with churn, many customers are still active when you run the analysis — you do not know when (or whether) they will eventually churn.

A customer who joined 18 months ago and is still active is **censored**: you know they survived at least 18 months, but not how long they will ultimately stay. Throwing away censored observations would bias your survival estimates downward (making churn appear faster than it is). Treating them as churned at the censoring point would be wrong too.

**Kaplan-Meier handles censoring correctly** by updating the risk set at each event time: a customer censored at month 18 contributes to the risk set at all earlier event times but not later ones.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: The Kaplan-Meier Estimator

At each event time (when at least one churn occurs), the survival probability is updated:

$$\hat{S}(t) = \hat{S}(t_{\text{prev}}) \times \frac{n_t - d_t}{n_t}$$

where n_t is the number of customers at risk just before time t, and d_t is the number who churn at time t.

**Worked example:**

| Month | Events (churned) | At risk | Survival step | Ŝ(t) |
|---|---|---|---|---|
| 0 | — | 200 | — | 1.0000 |
| 2 | 8 | 200 | (200−8)/200 = 0.9600 | **0.9600** |
| 5 | 12 | 185 | 0.9600 × (185−12)/185 = 0.9600 × 0.9351 | **0.8977** |
| 9 | 20 | 160 | 0.8977 × (160−20)/160 = 0.8977 × 0.8750 | **0.7855** |

**Reading the table:** Ŝ(9) = 0.7855 means 78.55% of customers survived at least 9 months. Equivalently, 21.45% churned within the first 9 months.

**Conditional churn probability** between any two times t₁ and t₂, given survival to t₁:

$$P(\text{churn between } t_1, t_2 \mid T > t_1) = 1 - \frac{\hat{S}(t_2)}{\hat{S}(t_1)}$$

**Example:** Conditional churn between months 5 and 9, given survival to month 5:
1 − Ŝ(9)/Ŝ(5) = 1 − 0.7855/0.8977 = 1 − 0.875 = **12.5%**

Note: this is much lower than the *unconditional* churn rate 1 − Ŝ(9) = 21.45%, which includes early-tenure churners.

> ### 🔍 Deep Dive: Why Multiply Rather Than Subtract?
> The product formula arises because each interval is conditioned on surviving the previous one. The probability of surviving through month 9 = P(survive to 2) × P(survive 2→5 | survived to 2) × P(survive 5→9 | survived to 5). Each conditional probability is (n_t − d_t)/n_t. Multiplying them gives the joint probability of surviving all intervals — which is Ŝ(t).

---

#### Part B: The Cox Proportional Hazards Model

While KM gives the survival curve for the whole population, Cox tells us how specific features predict individual churn timing.

The Cox model says:

$$h(t | X) = h_0(t) \times e^{\beta_1 X_1 + \beta_2 X_2 + ...}$$

**Plain English:** Each customer has a baseline churn hazard h₀(t) (shared across everyone). Their features multiply that baseline by e^(β₁X₁ + β₂X₂ + ...). The model estimates the βs from data.

**Hazard ratios:** We report e^β for each predictor, not β itself.

The two hazard ratios below are **illustrative round numbers**, not this course's fitted values — the
model fitted on `survival_data.csv` returns HR ≈ 1.47 for `support_tickets` and ≈ 0.72 for
`workouts_per_week`. (1.42 recurs elsewhere as a *given* constant in hand-calculations.)

| HR interpretation | Meaning |
|---|---|
| HR = 1.42 for support_tickets | Each additional support ticket multiplies churn hazard by 1.42 — a 42% increase in churn risk per ticket |
| HR = 0.74 for workouts_per_week | Each additional workout/week multiplies hazard by 0.74 — a 26% reduction in churn risk per workout |
| HR < 1 | The feature is protective — higher values reduce churn hazard |
| HR > 1 | The feature is a risk factor — higher values increase churn hazard |

**The proportional hazards assumption:** The ratio of hazards between any two customers stays constant over time. If customer A has twice the churn hazard as customer B at month 1, they must have twice the hazard at month 12, month 24, and every other time. To check: plot log(-log(Ŝ(t))) vs log(t) for each group — parallel lines confirm the assumption.

> ### 🔍 Deep Dive: The Partial Likelihood Trick
> The Cox model estimates β without needing to know the baseline hazard h₀(t) — a nuisance function that would be very hard to estimate. The partial likelihood uses only the ordering of event times: "given that someone churned at month 5, which customer in the risk set was it?" The probability of each observed event provides information about β without touching h₀(t). This is why Cox is called a semi-parametric model: parametric for the effect of covariates, non-parametric for the baseline.

---

### Part 1 Checkpoint

1. 500 customers at risk. At month 6: 25 churned. At month 12: 40 churned from the remaining risk set. Compute Ŝ(6) and Ŝ(12).

2. Using the values above: what fraction of customers who survived to month 6 churned by month 12?

3. A Cox model reports HR = 1.42 for support_tickets_last_30d. A customer filed 3 tickets. What is their relative hazard compared to a customer with 0 tickets?

4. A customer has been inactive for 18 months but has not formally cancelled. Are they: (a) censored, (b) churned at month 18, (c) still fully in the risk set?

5. The proportional hazards assumption says hazard ratios are constant over time. Why would this assumption fail for the "age of subscription" feature?

---

### Checkpoint Answer Key

**Q1.** Ŝ(6) = (500−25)/500 = 475/500 = **0.950**. For Ŝ(12): the risk set after month 6 = 500 − 25 = 475. Ŝ(12) = 0.950 × (475−40)/475 = 0.950 × 435/475 = 0.950 × 0.916 = **0.870**.
*Common wrong answer:* Using 500 as the risk set at month 12 (forgetting to subtract the 25 who churned at month 6 from the risk set).

> **Also asked on the slides:** *"A customer has been active for 14 months. The KM table has no event at month 14. What is Ŝ(14)?"* — Ŝ(14) equals the last value recorded before month 14. The Kaplan-Meier estimator is a product taken over **event times only**, so when no churn event occurs between the previous event and month 14 no new factor enters the product and the curve is flat: as Section 2.2 puts it, the KM plot is "a step function that drops at each churn event, with flat sections between events." On the slide's table (events at months 2, 5 and 9, with Ŝ = 0.9600, 0.8977 and 0.7855) that gives Ŝ(14) = **0.7855**. On the checkpoint table above, with events only at months 6 and 12, it would give Ŝ(14) = Ŝ(12) = 0.870. *Common wrong answer:* interpolating between event times, or assuming Ŝ must keep falling every month.

> **Also asked on the slides:** *"A customer has Ŝ(24) = 0.32. In plain English, what does this mean?"* — "About a 32% chance that this customer is still a subscriber 24 months in — equivalently, about a 68% chance they have churned by month 24." Ŝ(t) is the cumulative product of conditional survival probabilities (Section 1.4), so it is a survival *probability* for one customer, or the expected surviving *share* of a cohort. It is not a churn rate, not a retained-revenue figure, and it says nothing about *when* inside those 24 months the churn would occur.

**Q2.** Conditional churn = 1 − Ŝ(12)/Ŝ(6) = 1 − 0.870/0.950 = 1 − 0.916 = **8.4%**.
*Common wrong answer:* 1 − 0.870 = 13.0% (this is unconditional churn through month 12, not conditional on surviving to month 6).

> **Also asked on the slides:** *"Compute the conditional churn probability from month 2 to month 9."* — The same formula, applied to the slide's KM table (events at months 2, 5 and 9, with Ŝ = 0.9600, 0.8977 and 0.7855): 1 − Ŝ(9)/Ŝ(2) = 1 − 0.7855/0.9600 = 1 − 0.8182 = **18.2%**. Read it as "of the customers still subscribed at month 2, about 18% churn by month 9" — the denominator is the survivors at month 2, not the original cohort. The unconditional figure, 1 − 0.7855 = 21.5%, answers a different question.

**Q3.** HR = 1.42 per ticket. For 3 tickets: relative hazard = 1.42³ = **2.86** — nearly 3× higher churn risk.
*Common wrong answer:* 1.42 × 3 = 4.26. Hazard ratios compound multiplicatively (each ticket multiplies by 1.42), not additively.

> **Also asked on the slides:** *"'Customers who call support churn faster, so we should stop offering phone support.' What's wrong with this reasoning?"* — It reads an association as a cause, and gets the direction of causation backwards. A Cox hazard ratio is **associational**: HR > 1 says the covariate *predicts* faster churn, not that it *causes* it, and establishing cause would require randomising who receives support (Lecture 7). Support contacts are what Part B of the worked example calls "a meaningful early-warning signal of potential churn" — customers call *because* something has already gone wrong. Removing phone support removes the signal, not the problem, and plausibly makes churn worse by closing the one channel where the problem could still be fixed. The analytics response is to **use** the signal: trigger a retention intervention when a customer's ticket volume rises.

**Q4.** **(a) Censored.** If the customer has not formally cancelled, you observe them as active up to your analysis date. They are censored at 18 months — you know they survived at least 18 months but not what happens afterward.
*Common wrong answer:* Churned at month 18. Censoring means you lost track of the outcome, not that the outcome was bad. Treating censored customers as churned biases the survival curve downward.

**Q5.** New subscribers typically have high churn hazard (many try and leave quickly). Long-tenured subscribers have lower hazard (they've demonstrated loyalty). The hazard of a new subscriber relative to a 3-year subscriber is very high at month 1 but converges over time — the ratio is not constant. Any feature that is correlated with customer lifecycle stage will likely violate proportional hazards.
*Common wrong answer:* "It fails because some customers are more loyal than others." Heterogeneity in levels of loyalty does not violate the assumption — it's about the ratio of hazards being constant over time, not about absolute hazard levels.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work through Part B together)


---

**Problem Statement**

Seven subscription customers, observed from signup:

| Customer | Tenure (months) | Status |
|---|---|---|
| C1 | 2 | Churned |
| C2 | 4 | Churned |
| C3 | 5 | Still active (censored) |
| C4 | 7 | Churned |
| C5 | 8 | Still active (censored) |
| C6 | 10 | Churned |
| C7 | 12 | Still active (censored) |

**Part A:** Construct the Kaplan-Meier table. Compute $\hat{S}(t)$ at each event time as both a fraction and a decimal (3 places).

**Part B:** A Cox model fitted to a larger dataset returns:

| Predictor | Coefficient β | Hazard Ratio e^β | 95% CI |
|---|---|---|---|
| login_freq (logins/month) | −0.052 | ? | [0.903, 0.997] |
| support_tickets (per 90 days) | 0.148 | ? | [1.031, 1.323] |
| monthly_spend ($) | −0.002 | ? | [0.996, 1.001] |

Fill in the hazard ratios and interpret each in one business sentence.

---

**Full Solution**

**Part A: Kaplan-Meier Table**

Event times (churn only): t = 2, 4, 7, 10. Censored observations at t = 5, 8, 12 reduce the risk set but do not create KM steps.

| Time | Event? | At risk $n_i$ | Churns $d_i$ | Factor $(1-d_i/n_i)$ | $\hat{S}(t)$ |
|---|---|---|---|---|---|
| 0 | Start | 7 | 0 | 1 | 1.000 |
| 2 | Churn | 7 | 1 | 6/7 | 6/7 ≈ **0.857** |
| 4 | Churn | 6 | 1 | 5/6 | 6/7 × 5/6 = 5/7 ≈ **0.714** |
| 5 | Censored | — | — | — | 5/7 (unchanged) |
| 7 | Churn | 4 | 1 | 3/4 | 5/7 × 3/4 = 15/28 ≈ **0.536** |
| 8 | Censored | — | — | — | 15/28 (unchanged) |
| 10 | Churn | 2 | 1 | 1/2 | 15/28 × 1/2 = 15/56 ≈ **0.268** |
| 12 | Censored | — | — | — | 15/56 (unchanged) |

**Why does at-risk drop from 6 to 4 between t=4 and t=7?**
After t=4: C1 and C2 have churned (5 remain). C3 is censored at t=5 and leaves the risk set. So at t=7, only 4 customers remain at risk: C4, C5, C6, C7.

**Interpretation:** $\hat{S}(12) = 15/56 \approx 0.268$. About 27% of customers are estimated to still be active after 12 months. Note: with only 7 customers, the confidence intervals around this estimate would be very wide.

**Part B: Hazard Ratios**

$$e^{-0.052} \approx 0.949, \qquad e^{0.148} \approx 1.160, \qquad e^{-0.002} \approx 0.998$$

**Login frequency (HR = 0.949):** "Customers who log in one more time per month have a churn hazard approximately 5.1% lower, holding all else constant — more frequent engagement is protective."

**Support tickets (HR = 1.160):** "Customers who file one additional support ticket per 90 days have a churn hazard approximately 16.0% higher — ticket volume is a meaningful early-warning signal of potential churn."

**Monthly spend (HR = 0.998):** "Customers who spend \$1 more per month have a churn hazard approximately 0.2% lower — a small but consistent protective effect; a \$100 increase in monthly spend reduces hazard by about 18%."


---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The Kaplan-Meier plot:** A step function that drops at each churn event, with flat sections between events. Confidence bands widen over time as fewer customers remain. Steeper early drops indicate high early churn risk. A flat tail indicates loyal long-tenure customers.

**Log-rank test:** When comparing curves for two groups (e.g., basic vs. enterprise plan), the log-rank test p-value tells you whether the survival curves are significantly different. p < 0.05 indicates the groups have genuinely different survival patterns.

**Concordance index (c-index):** The Cox model's analog of AUC. It measures how well the model ranks customers by their actual churn timing — what fraction of customer pairs is correctly ranked (higher hazard for the one who churned sooner). A c-index of 0.5 is no better than chance; 1.0 is perfect. Values of 0.65–0.80 are typical for subscription churn models.

**Proportional hazards check:** The `check_assumptions()` function in `lifelines` tests whether hazard ratios are constant over time. If a predictor fails this test (p < 0.05 in the Schoenfeld residual test), the hazard ratio changes over time and the Cox coefficient should be interpreted as an average effect only.

**Before trusting the agent output, verify:**
1. KM curve starts at 1.0 and decreases monotonically
2. All hazard ratios are positive (non-negative)
3. C-index is above 0.6
4. Engagement predictors (login frequency, spend) have HR < 1; friction predictors (tickets, complaints) have HR > 1

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

<!-- BEGIN GENERATED homework pointer - tools/render_lecture_homework.py; do not hand-edit.
     Replaces a ~150-line duplicate of the homework that had drifted into a DIFFERENT
     assignment (Task 004, 2026-08-13): for lectures 04-10 only 1-3 of ~16 question
     variables still matched the notebook, and the dataset filenames were wrong.
     The notebook is the assignment of record; answers live only in answer_keys/. -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_04_survival_analysis.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_04_survival_analysis.ipynb` |
| Dataset | `homework_datasets/survival_data.csv` |
| Graded questions | **17** — Part A: 8 · Part B: 6 · Part C: 3 |
| Answer key (instructor only) | `answer_keys/hw04.json` |

Answers and tolerances are never duplicated outside `answer_keys/hwNN.json`
(rendered for instructors as `quiz/answer_key_values.md`).

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "Survival analysis is only for mortality data."**
"Survival" is a generic term for any time-to-event analysis. Churn, loan default, equipment failure, employee resignation — all are valid applications with identical mathematics.

**2. "Censored observations are missing data and should be excluded."**
Censored observations contain real information: the customer has survived at least until time $t_i$. Excluding them biases survival estimates downward.

**3. "The KM curve predicts when a specific customer will churn."**
The KM curve is a population-level estimate. It describes the distribution of survival times across a group, not a prediction for a specific individual.

**4. "A hazard ratio of 0.95 means the predictor reduces churn by 95%."**
HR = 0.95 means the hazard is reduced by 5% (not 95%). The reduction is $1 - 0.95 = 0.05 = 5\%$.

**5. "Proportional hazards means the hazard is constant over time."**
It means the *ratio* of two customers' hazards is constant over time — not that any single hazard is constant. The baseline hazard $h_0(t)$ can take any shape.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Fits and plots KM curves per group | Section 1.4A — product-limit estimator |
| Log-rank test between groups | Section 1.1, Tool 1 — conditional probability; Section 2.2 — reading the test |
| Fits Cox model, prints β and $e^\beta$ | Section 1.4B — hazard function and hazard ratio |
| Reports concordance index | Section 2.2 — model evaluation |
| Runs check_assumptions() | Section 2.2 — proportional hazards test |

**What to verify before trusting the output:**
1. KM curve starts at 1.0 and decreases monotonically
2. All hazard ratios positive
3. C-index above 0.6
4. Engagement predictors have HR < 1; friction predictors have HR > 1
