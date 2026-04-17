# Chapter 4: Survival Analysis and Churn Prediction
## Comprehensive Reference Guide

---

### Chapter Overview

Survival analysis is the set of statistical methods for modeling the time until an
event — in our context, churn. It differs from standard regression in one crucial
way: **censoring**. Many customers are still active when you run your analysis.
Standard regression cannot handle this correctly; survival methods do. This chapter
derives the Kaplan-Meier estimator, the Cox Proportional Hazards model, and the
partial likelihood that makes Cox estimation tractable without knowing the baseline
hazard function.

**By the end of this chapter you should be able to:**
- Define the hazard function and derive its relationship to the survival function
- Construct a Kaplan-Meier table step by step and compute conditional probabilities
- Explain why censored observations cannot simply be dropped or treated as events
- Interpret Cox model hazard ratios including their multiplicative compounding
- Test the proportional hazards assumption and recognize violations
- Derive the partial likelihood that identifies β without estimating h₀(t)

---

## 4.1 Strategic Frame: What Practitioners Know

### Gibson Biddle: Retention as the North Star

Gibson Biddle served as Vice President of Product at Netflix from 2005 to 2010.
During his tenure, the company's monthly churn rate fell from approximately 10% to
approximately 4.5% — a reduction that fundamentally changed Netflix's unit economics.

The numbers illustrate why: at 10% monthly churn, average subscriber lifetime is
$1/0.10 = 10$ months. At 4.5% monthly churn, average lifetime is $1/0.045 \approx 22$ months.
The lifetime more than doubles. This change directly determines how much Netflix can
spend to acquire each new subscriber:

$$\text{Maximum sustainable CAC} = \text{Monthly contribution} \times \text{Average lifetime}$$

Biddle's management principle: every product decision should be evaluated against its
effect on retention. Not engagement rate, not NPS, not app store rating — retention,
because it is the metric most directly tied to the lifetime value calculation above.

He describes the challenge: "It's very difficult to move retention in the short term.
The metrics that move weekly or monthly are often leading indicators." This is exactly
the problem Cox regression addresses — identifying which behavioral features measured
today predict churn timing months or years later.

### Jess Lachs: Short-Term Metrics for Long-Term Outcomes

Jessica Lachs, who built the data science organization at DoorDash, articulates the
strategic frame for survival modeling from the operational side:

"Retention is a terrible thing to goal on. It's almost impossible to drive in a
meaningful way in the short term. Ultimately, you want to find a short-term metric
you can measure that drives a long-term output."

The Cox hazard ratios derived in Section 4.6 are exactly this: they identify which
observable behaviors in the current week predict churn timing over the next 6–24 months.
A feature with HR = 0.74 (each additional workout per week reduces churn hazard 26%)
gives the product team a specific, actionable target that can be measured immediately
and whose long-run effect is quantified.

### Three Strategic Insights

**Insight 1: The survival curve is more informative than a single churn rate.**

Monthly churn rate = 3% is consistent with radically different customer trajectories.
It could mean rapid early churn followed by a stable loyal core (early-period churn
curves steeply, then flattens). Or it could mean a steady 3% monthly attrition at
every tenure level. These have completely different implications for product investment,
cohort analysis, and revenue forecasting. A Kaplan-Meier curve distinguishes them.
A single monthly churn number does not.

**Insight 2: The censoring problem is the central methodological challenge.**

The most common mistake in churn analysis: using logistic regression on a 90-day window
and treating still-active customers as "did not churn." This approach:
- Treats customers who churned at month 4 (outside the window) as "loyal" non-churners
- Treats long-tenure active customers as structurally identical to brand-new customers
  who happen not to have churned yet
- Produces an estimate of P(churn in 90 days), not a survival model

Kaplan-Meier and Cox handle censoring correctly by construction. They treat still-active
customers as "survived at least until the observation date" — providing partial
information without inventing a churn event.

**Insight 3: The proportional hazards assumption needs testing, not assuming.**

The Cox model assumes that the ratio of hazard rates between any two customers is
constant over time. In practice, this is violated by:
- Features that are highly protective early but less so later (e.g., "activated in
  first week" matters a lot at 3 months, less at 36 months)
- Features correlated with tenure itself (older customers look different from new
  customers on engagement metrics regardless of churn risk)

Standard implementations of Cox (lifelines, survival in R) provide Schoenfeld residual
tests. Always run them. A significant test result requires either stratification or
time-varying coefficients.

---

## 4.2 Mathematical Prerequisites

### 4.2.1 The Survival Function

The **survival function** S(t) gives the probability that the event time T exceeds t:
$$S(t) = P(T > t), \quad t \geq 0$$

Properties:
- S(0) = 1 (all subjects survive at time 0)
- S is non-increasing: S(t₁) ≥ S(t₂) for t₁ < t₂
- $\lim_{t \to \infty} S(t) = 0$ (eventually all subjects experience the event)

**Relationship to CDF:** $F(t) = P(T \leq t) = 1 - S(t)$

**Relationship to density:** $f(t) = \frac{d}{dt}F(t) = -\frac{d}{dt}S(t)$

*Reference: Wikipedia, "Survival function" — en.wikipedia.org/wiki/Survival_function*

### 4.2.2 The Hazard Function

The **hazard function** $h(t)$ is the instantaneous rate of failure at time t,
conditional on surviving to t:

$$h(t) = \lim_{\Delta t \to 0} \frac{P(t \leq T < t + \Delta t \mid T \geq t)}{\Delta t}$$

Using conditional probability:
$$P(t \leq T < t + \Delta t \mid T \geq t) = \frac{P(t \leq T < t + \Delta t)}{P(T \geq t)} = \frac{f(t)\Delta t}{S(t)}$$

Therefore: $h(t) = \frac{f(t)}{S(t)}$

Since $f(t) = -S'(t)$: $h(t) = -\frac{S'(t)}{S(t)} = -\frac{d}{dt}\ln S(t)$

**Cumulative hazard:** $H(t) = \int_0^t h(u)\,du = -\ln S(t)$

**Recovery:** $S(t) = \exp(-H(t)) = \exp\left(-\int_0^t h(u)\,du\right)$

These relationships mean: knowing any one of S(t), h(t), H(t), or f(t) determines
all the others. The choice of which to model is mathematical convenience.

*Reference: Wikipedia, "Hazard ratio" — en.wikipedia.org/wiki/Hazard_ratio*
*Reference: Wikipedia, "Survival analysis" — en.wikipedia.org/wiki/Survival_analysis*

---

## 4.3 Censoring: Types and Mechanics

### Right Censoring (Dominant Case in Churn Analysis)

An observation is **right-censored** at time c if we know the event had not occurred
by time c but we do not observe the event time. For a subscription business:

- **Administrative censoring:** Customer is still active at the analysis date.
  We know T > current_date − signup_date.
- **Loss to follow-up:** Customer is unreachable after some point (e.g., no longer
  responding to any touchpoints).
- **Competing event:** Something else happened (company shutdown, acquired).

### Why Censoring Cannot Be Ignored

**Scenario:** 200 customers signed up one year ago. 50 have churned (observed event
times t₁, ..., t₅₀). 150 are still active.

**Wrong approach 1 (drop censored):** Analyze only the 50 churners. Estimate
average churn time as mean(t₁,...,t₅₀). Problem: the 150 survivors are not a random
sample — they are systematically the customers who are most likely to stay long. This
selection bias makes churn appear faster than it is.

**Wrong approach 2 (treat censored as churned at today):** Assign churn time = 12
months to all 150 active customers. Problem: invents churn events. Many of these
customers will stay another 2–5 years; assigning them churn at month 12 is wrong.

**Correct approach (KM/Cox):** The 150 censored customers contribute to the risk set
at all months before censoring (providing partial information: "they survived at least
this long") but do not contribute event counts (we do not know their true churn time).

### Independent Censoring Assumption

Both KM and Cox require:
$$T \perp C \mid X$$

The censoring time C is independent of the event time T given covariates X. Informally:
knowing that a customer was censored gives no information about when they would have
churned if observed longer.

**Potential violation:** If customers who are about to churn tend to become
unreachable (stop opening emails, stop logging in — and become administratively
censored if the study ends soon after), then censoring is informative about T.
This is called **informative censoring** and violates the KM/Cox assumptions.

*Reference: Wikipedia, "Censoring (statistics)" —
en.wikipedia.org/wiki/Censoring_(statistics)*

---

## 4.4 Kaplan-Meier Estimator: Full Derivation

### The Product-Limit Approach

Let $t_1 < t_2 < \cdots < t_k$ be the distinct observed event times.

At event time $t_j$:
- $n_j$ = number at risk (not yet churned or censored immediately before $t_j$)
- $d_j$ = number of events (churns) at $t_j$
- $c_j$ = number censored in the interval $[t_{j-1}, t_j)$

The risk set decreases as: $n_{j+1} = n_j - d_j - c_j$

**Conditional survival** probability in the interval $[t_{j-1}, t_j)$:
$$\hat{p}_j = P(T > t_j \mid T \geq t_j) = \frac{n_j - d_j}{n_j} = 1 - \frac{d_j}{n_j}$$

**Kaplan-Meier estimator** — the product of all conditional survival probabilities up to t:
$$\hat{S}(t) = \prod_{j: t_j \leq t} \frac{n_j - d_j}{n_j}$$

This is called the **product-limit estimator** because it multiplies conditional
survival probabilities over all observed event times up to t.

> **Teaching note (board/tablet):** Build the table column by column. Write the
> column headers first: t, d_j, n_j, (n_j−d_j)/n_j, Ŝ(t). Fill the first row.
> Walk through three event times step by step. Emphasize two things: (1) when
> a censoring occurs between two event times, it reduces n_j at the next event
> time but does NOT appear as d_j; (2) Ŝ(t) is a running product — multiply,
> don't add. Budget 12 minutes for this construction.

### Derivation: KM is the Nonparametric MLE of S(t)

The likelihood for observed data is:
$$L = \prod_{i: \text{event}} f(t_i) \times \prod_{i: \text{censored}} S(c_i)$$

For a nonparametric estimator, S(t) can jump only at observed event times.
Let $p_j = P(T = t_j \mid T \geq t_j)$ be the discrete hazard at $t_j$. Then:
$$S(t) = \prod_{j: t_j \leq t} (1 - p_j)$$

Substituting into the likelihood and maximizing with respect to each $p_j$
independently gives: $\hat{p}_j = d_j / n_j$ (observed fraction of at-risk customers
who churned at each event time). This recovers the KM formula. QED.

*Reference: Kaplan, E. L. and Meier, P. (1958). "Nonparametric estimation from
incomplete observations." JASA 53(282):457–481.*
*Reference: Wikipedia, "Kaplan-Meier estimator" —
en.wikipedia.org/wiki/Kaplan%E2%80%93Meier_estimator*

### Worked Numerical Example

| Month t | Events dⱼ | At risk nⱼ | Step (nⱼ−dⱼ)/nⱼ | Ŝ(t) |
|---|---|---|---|---|
| 0 | — | 200 | — | 1.000 |
| 2 | 8 | 200 | 192/200 = 0.960 | **0.960** |
| 5 | 12 | 185 | 173/185 = 0.9351 | 0.960 × 0.9351 = **0.8977** |
| 9 | 20 | 160 | 140/160 = 0.875 | 0.8977 × 0.875 = **0.7855** |
| 14 | 15 | 130 | 115/130 = 0.885 | 0.785 × 0.885 = **0.695** |
| 20 | 10 | 105 | 95/105 = 0.905 | 0.695 × 0.905 = **0.629** |

**Note on risk set:** Between months 5 and 9, some customers were censored (still
active). The risk set at month 9 is 160, not 185 − 12 = 173 — 25 additional
customers were censored in the interval [5, 9).

**Unconditional churn by month 9:** P(T ≤ 9) = 1 − Ŝ(9) = 1 − 0.785 = **21.5%**

**Conditional churn between months 5 and 9, given survival to month 5:**
$$P(5 < T \leq 9 \mid T > 5) = 1 - \frac{\hat{S}(9)}{\hat{S}(5)} = 1 - \frac{0.785}{0.897} = **12.5%**$$

This is meaningfully different from the unconditional 21.5% — it describes the
experience of customers who have already demonstrated they can survive to month 5.

### Greenwood's Formula: Variance of KM Estimator

$$\widehat{\text{Var}}(\hat{S}(t)) = \hat{S}(t)^2 \sum_{j: t_j \leq t} \frac{d_j}{n_j(n_j - d_j)}$$

Used to construct confidence bands around the KM curve.

*Reference: Wikipedia, "Kaplan-Meier estimator — Confidence intervals" —
en.wikipedia.org/wiki/Kaplan%E2%80%93Meier_estimator*

---

## 4.5 Log-Rank Test: Comparing Survival Curves

To test whether two groups (e.g., monthly vs. annual subscribers) have the same
underlying survival function:

H₀: $S_1(t) = S_2(t)$ for all t.

At each event time $t_j$:

| | Events | Survivors | At risk |
|---|---|---|---|
| Group 1 | O₁ⱼ | | n₁ⱼ |
| Group 2 | O₂ⱼ | | n₂ⱼ |
| Total | dⱼ | | nⱼ |

Under H₀, expected events in group 1: $E_{1j} = n_{1j} \cdot d_j / n_j$

Test statistic:
$$\chi^2 = \frac{\left(\sum_j (O_{1j} - E_{1j})\right)^2}{\text{Var}\left(\sum_j O_{1j}\right)} \sim \chi^2_1 \text{ under H}_0$$

where the variance term follows from hypergeometric distribution theory at each
event time.

*Reference: Wikipedia, "Log-rank test" — en.wikipedia.org/wiki/Log-rank_test*

---

## 4.6 Cox Proportional Hazards Model

### Model Specification

$$h(t \mid \mathbf{x}_i) = h_0(t) \cdot \exp(\boldsymbol{\beta}^\top \mathbf{x}_i)$$

**Key features:**
- $h_0(t)$: baseline hazard — unspecified nonparametric function
- $\exp(\boldsymbol{\beta}^\top \mathbf{x}_i)$: multiplicative adjustment from covariates
- The ratio of any two subjects' hazards: $h(t|x_i)/h(t|x_j) = \exp(\beta^\top(x_i - x_j))$ — constant over time (the PH assumption)

The log-hazard is linear in covariates:
$$\ln h(t \mid \mathbf{x}_i) = \ln h_0(t) + \boldsymbol{\beta}^\top \mathbf{x}_i$$

### The Partial Likelihood: Deriving β Without Estimating h₀(t)

**Cox's insight (1972):** At each event time $t_j$, we observe that exactly one person
in the risk set $R(t_j)$ experienced the event. The conditional probability that it
was individual $i(j)$ (given that exactly one person churned from $R(t_j)$):

$$\ell_j(\boldsymbol{\beta}) = P(\text{person } i(j) \text{ fails at } t_j \mid \text{one failure from } R(t_j))$$

Using the proportional hazards structure:
$$= \frac{h(t_j \mid x_{i(j)})}{\sum_{l \in R(t_j)} h(t_j \mid x_l)} = \frac{h_0(t_j)\exp(\beta^\top x_{i(j)})}{\sum_{l \in R(t_j)} h_0(t_j)\exp(\beta^\top x_l)} = \frac{\exp(\beta^\top x_{i(j)})}{\sum_{l \in R(t_j)} \exp(\beta^\top x_l)}$$

$h_0(t_j)$ cancels from numerator and denominator. The partial likelihood:
$$L_P(\boldsymbol{\beta}) = \prod_{j=1}^{D} \frac{\exp(\boldsymbol{\beta}^\top \mathbf{x}_{i(j)})}{\sum_{l \in R(t_j)} \exp(\boldsymbol{\beta}^\top \mathbf{x}_l)}$$

$$\log L_P(\boldsymbol{\beta}) = \sum_{j=1}^{D} \left[\boldsymbol{\beta}^\top \mathbf{x}_{i(j)} - \log\sum_{l \in R(t_j)} \exp(\boldsymbol{\beta}^\top \mathbf{x}_l)\right]$$

This log-partial-likelihood is maximized numerically (Newton-Raphson or gradient descent)
to obtain the Cox estimates $\hat{\beta}$.

> **Teaching note:** Present the partial likelihood conceptually first: "At each
> churn event, we know someone churned. We ask: given the risk set, which person
> would the model say is most likely to have churned? The partial likelihood scores
> the model's answer against what actually happened." Then write the formula as
> making that statement precise. The cancellation of h₀(t) is the magic trick —
> show it explicitly in two lines. Budget 12 minutes.

*Reference: Cox, D. R. (1972). "Regression Models and Life-Tables." JRSS Series B
34(2):187–220. Widely available via academic libraries.*
*Reference: Wikipedia, "Proportional hazards model" —
en.wikipedia.org/wiki/Proportional_hazards_model*

### Hazard Ratios: Interpretation and Compounding

The hazard ratio for a one-unit increase in covariate $x_k$:
$$\text{HR}_k = \frac{h(t \mid x_k + 1, \text{others fixed})}{h(t \mid x_k, \text{others fixed})} = e^{\beta_k}$$

For a c-unit increase: $\text{HR} = e^{c\beta_k}$. Hazard ratios compound multiplicatively.

| Predictor | β̂ | HR = e^β̂ | Interpretation |
|---|---|---|---|
| avg_workouts_per_week | −0.30 | 0.74 | Each additional workout/week reduces hazard by 26% |
| support_tickets_30d | +0.35 | 1.42 | Each additional ticket increases hazard by 42% |
| annual_plan (vs monthly) | −0.90 | 0.41 | Annual subscribers have 59% lower hazard |

**Compounding example:** A customer with 3 additional support tickets has relative
hazard $1.42^3 = 2.86$ — nearly 3× higher churn risk. The exponentiation is crucial:
$3 \times 1.42 = 4.26$ (wrong); $1.42^3 = 2.86$ (correct).

### The Concordance Index (C-index)

The C-index measures the model's ability to correctly rank pairs of subjects by their
predicted survival time:
$$C = \frac{\text{number of concordant pairs}}{\text{number of comparable pairs}}$$

- C = 0.5: model ranks no better than random
- C = 1.0: model perfectly ranks all pairs
- Typical good range in churn: 0.70–0.85

### Testing the Proportional Hazards Assumption

**Schoenfeld residuals:** For each subject i who experiences the event at $t_j$:
$$r_{kj} = x_{ki(j)} - \bar{x}_{k,j}$$

where $\bar{x}_{k,j}$ is the weighted average of covariate k in the risk set at $t_j$.

Under PH, Schoenfeld residuals should be uncorrelated with event times. A significant
correlation (tested via regression of $r_{kj}$ on $t_j$) indicates PH violation for
covariate k.

In Python (lifelines):
```python
from lifelines.statistics import proportional_hazard_test
results = proportional_hazard_test(cph, df, time_transform='rank')
results.print_summary()
```

*Reference: Schoenfeld, D. (1982). "Partial residuals for the proportional hazards
regression model." Biometrika 69(1):239–241.*

---

## 4.7 Full Proof Appendix

### Proof: KM is the Nonparametric Maximum Likelihood Estimator

A fully rigorous proof proceeds as follows. Let S(t) be any cadlag nonparametric
survival function. The likelihood for the observed data is:

$$L(S) = \prod_{i: \text{event at } t_i} [S(t_i^-) - S(t_i)] \times \prod_{i: \text{censored at } c_i} S(c_i)$$

where $S(t^-)$ is the left limit. A nonparametric MLE can only place mass at observed
event times $\{t_j\}$. At each $t_j$, let $q_j = S(t_{j-1}^-) - S(t_j)$ be the
probability mass placed there. The likelihood factors into terms involving each $q_j$
separately. Maximizing each term independently gives $q_j = S(t_{j-1})d_j/n_j$,
which yields $S(t_j) = S(t_{j-1})(1 - d_j/n_j)$ — exactly the KM recursion. QED.

### Proof: Partial Likelihood is Valid for Inference

Cox showed that the partial likelihood has the same asymptotic properties as a full
likelihood: the score equations are unbiased, the information matrix is correct, and
the MLE from the partial likelihood achieves the Cramér-Rao bound within the class
of estimators based only on the event orderings. The full proof is in Cox (1975)
"Partial likelihood," Biometrika 62:269–276.

---

## 4.8 Verification Checklist

| Check | How to verify |
|---|---|
| KM step at each event = (nⱼ−dⱼ)/nⱼ? | Compute by hand |
| Ŝ(t) = running product of all steps up to t? | Multiply all factors |
| Censored obs reduce nⱼ but not dⱼ? | They leave the risk set but are not events |
| Ŝ(t) is non-increasing? | Each step multiplies by ≤ 1 |
| Conditional churn = 1 − Ŝ(t₂)/Ŝ(t₁)? | Not Ŝ(t₂) alone |
| HR = exp(β), not β itself? | Cox output is log-HR; exponentiate |
| HR compounds: k-unit increase = HRᵏ? | Not k × HR |
| C-index range [0.5, 1.0]? | 0.5 = random, 1.0 = perfect |
| PH assumption tested? | Schoenfeld residuals p-value |

---

## 4.9 Chapter Bibliography

1. Wikipedia, "Kaplan-Meier estimator" — en.wikipedia.org/wiki/Kaplan%E2%80%93Meier_estimator
2. Wikipedia, "Proportional hazards model" — en.wikipedia.org/wiki/Proportional_hazards_model
3. Wikipedia, "Survival analysis" — en.wikipedia.org/wiki/Survival_analysis
4. Wikipedia, "Censoring (statistics)" — en.wikipedia.org/wiki/Censoring_(statistics)
5. Wikipedia, "Log-rank test" — en.wikipedia.org/wiki/Log-rank_test
6. Wikipedia, "Hazard ratio" — en.wikipedia.org/wiki/Hazard_ratio
7. Kaplan, E. L. and Meier, P. (1958). "Nonparametric estimation from incomplete observations." JASA 53(282):457–481.
8. Cox, D. R. (1972). "Regression Models and Life-Tables." JRSS Series B 34(2):187–220.
9. Cox, D. R. (1975). "Partial likelihood." Biometrika 62(2):269–276.
10. Davidson-Pilon, C. lifelines documentation: lifelines.readthedocs.io
11. MIT OpenCourseWare, 18.650 Statistics for Applications — ocw.mit.edu

**Lenny's Podcast episode references:**
- Gibson Biddle episode on product strategy at Netflix — Lenny's Podcast
- Jess Lachs episode on building data organizations at DoorDash — Lenny's Podcast
