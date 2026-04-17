# Chapter 7: Uplift Modeling
## Comprehensive Reference Guide

---

### Chapter Overview

Standard propensity models predict who is likely to convert. Uplift models predict
whose conversion probability *changes* because of a treatment. This chapter develops
the potential outcomes framework from first principles, derives the T-learner CATE
estimator, proves the targeting threshold formula, defines the Qini coefficient, and
explains the identifying conditions (particularly random assignment) required for
valid uplift estimation.

**By the end of this chapter you should be able to:**
- Define potential outcomes Y(1) and Y(0) and explain the fundamental problem
- Derive the unbiasedness of the ATE estimator under random assignment
- Implement the T-learner formula and explain when it fails
- Derive the profit-maximizing targeting threshold from expected profit maximization
- Explain what the Qini coefficient measures and what it does not
- Identify three ways selection bias enters the T-learner under non-random assignment

---

## 7.1 Strategic Frame: What Practitioners Know

### Jackson Shuttleworth: 600 Experiments on a Single Feature

Jackson Shuttleworth is a Group Product Manager at Duolingo leading the retention
team. Over four years, his team ran over 600 experiments on a single product feature:
the streak — the count of consecutive days a user has practiced.

The volume is remarkable, but the insight that most directly motivates uplift modeling
is qualitative: the users who responded to specific streak interventions were not the
users who were already most committed. They were users on the margin — those who had
the habit partially formed but hadn't yet made it automatic.

He describes one specific experiment: changing the CTA button from "Continue" to
"Commit to my goal" was a significant retention win for the product. But the users
who drove this improvement were specifically those who had not yet formed a consistent
streak habit. The existing power users — the ones a propensity model would rank
highest — were coming back regardless. The campaign's value came entirely from the
marginal users.

A standard propensity model would have identified the power users as "high likelihood
of continued engagement" and targeted them preferentially. An uplift model correctly
identifies them as Sure Things — the campaign is wasted on them — and redirects
resources to the marginal users where the treatment effect is positive.

Shuttleworth adds a specific warning about campaign-level thinking: "Lower activation
rate can indicate a better product." If you lower the activation threshold (easier to
qualify as "activated"), you bring in more low-intent users who inflate your
activation count while diluting your retention cohort. The real question is not "who
activated?" but "whose activation was genuinely caused by our intervention and
persisted?" Uplift modeling formalizes this distinction.

### Lauryn Isford: The Precision vs. Volume Trade-off

Lauryn Isford, who led growth at Airtable after six years at Facebook, describes the
organizational pressure that leads to propensity-based targeting:

"If the business implication of a 6% versus 7% activation rate doesn't actually
change your decision, the precision doesn't help."

But in retention campaigns, the business implication of targeting Sure Things and
Sleeping Dogs (negative-uplift customers) is very direct: you spend the campaign
budget on customers who either didn't need it (Sure Things: wasted $8/customer) or
were made worse by it (Sleeping Dogs: the campaign prompted cancellation they otherwise
wouldn't have initiated). Precision — targeting specifically the Persuadables — is
precisely what changes the financial outcome.

Her rule: every retention campaign should have a holdout group (a random sample of
the target segment that receives no campaign). This is the only way to measure actual
uplift, as opposed to conversion rate among treated customers, which confounds
baseline conversion with treatment effect.

### Three Strategic Insights

**Insight 1: Sleeping Dogs are real and their prevalence is underestimated.**

The conventional wisdom is that retention campaigns at worst don't help. In reality,
a meaningful fraction of customers — typically 5–15% in subscription businesses —
respond negatively to retention outreach. They were planning to renew; the campaign
reminded them to evaluate whether they still needed the product; they churned.

This phenomenon is measurable only with a randomized holdout: treated churn rate
higher than control churn rate for a specific segment = Sleeping Dogs.

**Insight 2: The targeting threshold depends on the cost-to-value ratio, not just positive uplift.**

Many teams using uplift models deploy the rule "target everyone with τ̂ > 0." This
is correct only when the campaign costs nothing. For any positive campaign cost c and
conversion value v, the correct threshold is τ̂ > c/v — not τ̂ > 0.

A customer with τ̂ = 0.05 and conversion value $30 generates expected incremental
revenue of 0.05 × $30 = $1.50. If the campaign costs $4, this customer has expected
net profit of $1.50 − $4 = −$2.50. Targeting them wastes $2.50 per customer.

**Insight 3: Random assignment is not optional — it is what makes the model valid.**

Without random assignment, the T-learner estimates τ̂(x) that confounds the treatment
effect with selection bias. High-value customers are more likely to be treated in
non-randomized deployments; they also have higher baseline conversion rates. The
T-learner incorrectly attributes their high conversion rate to the treatment effect.

The bias magnitude is unknown and cannot be corrected after the fact with observed
features. The only solution is to randomize treatment assignment before running the
experiment.

---

## 7.2 The Potential Outcomes Framework

### Rubin's Causal Model

For each customer i, define two **potential outcomes**:
- $Y_i(1)$ = outcome if customer i receives the treatment (campaign sent)
- $Y_i(0)$ = outcome if customer i does not receive treatment (control)

The **individual treatment effect:**
$$\tau_i = Y_i(1) - Y_i(0)$$

**The fundamental problem of causal inference (Holland, 1986):** We observe only one
of $Y_i(1)$ or $Y_i(0)$ for each customer — never both. If customer i was treated,
we observe $Y_i(1)$ but not the counterfactual $Y_i(0)$. If they were in control,
we observe $Y_i(0)$ but not $Y_i(1)$.

This is not a measurement problem solvable with more data. It is a logical impossibility:
we cannot observe what would have happened in a world that did not occur.

*Reference: Holland, P. W. (1986). "Statistics and Causal Inference." JASA 81(396):945–960.
Summary available via JSTOR.*
*Reference: Wikipedia, "Rubin causal model" — en.wikipedia.org/wiki/Rubin_causal_model*

### Average Treatment Effect

Since we cannot compute τᵢ for any individual, we estimate the population average:
$$\tau_{ATE} = E[Y(1) - Y(0)] = E[Y(1)] - E[Y(0)]$$

**Under random assignment (T ⊥ Y(0), Y(1)):**
$$E[Y(1)] = E[Y \mid T=1], \quad E[Y(0)] = E[Y \mid T=0]$$

Therefore:
$$\hat{\tau}_{ATE} = \bar{Y}_{treated} - \bar{Y}_{control}$$

The difference in observed means is an unbiased estimator of the ATE.

**Why random assignment enables this:** Without random assignment:
$$E[Y \mid T=1] = E[Y(1) \mid T=1] \neq E[Y(1)]$$

because treated customers are not a random sample — they are systematically different.
Under random assignment, T is by construction independent of Y(1) and Y(0), so
$E[Y(1)|T=1] = E[Y(1)]$.

*Reference: Wikipedia, "Average treatment effect" —
en.wikipedia.org/wiki/Average_treatment_effect*
*Reference: arXiv:1706.03461, Künzel et al. (2019). "Meta-learners for Estimating
Heterogeneous Treatment Effects." arxiv.org/abs/1706.03461*

### CATE: Conditional Average Treatment Effect

$$\tau(x) = E[Y(1) - Y(0) \mid X = x]$$

CATE estimates the treatment effect for customers with specific feature profile X = x.
This is the quantity uplift models estimate — different from ATE because different
customer segments may respond very differently to treatment.

### SUTVA: Stable Unit Treatment Value Assumption

Required for the potential outcomes framework to be well-defined:
1. **No interference:** Customer i's outcome is unaffected by whether customer j
   was treated ($Y_i(T_i, T_{-i}) = Y_i(T_i)$)
2. **No hidden versions:** There is only one version of the treatment

SUTVA is violated in referral programs (treating one customer causes them to recruit
others), in markets with competitive dynamics (treating some customers changes prices
that affect others), and in social network settings (treatment spreads through the network).

*Reference: Wikipedia, "Stable unit treatment value assumption" —
en.wikipedia.org/wiki/Rubin_causal_model#SUTVA*

---

## 7.3 The Four Customer Segments

Every customer belongs to exactly one of four segments based on their potential outcomes:

| Segment | Y(0) | Y(1) | τᵢ | Should target? |
|---|---|---|---|---|
| **Persuadable** | 0 (won't convert without) | 1 (converts with) | +1 | ✅ |
| **Sure Thing** | 1 (converts without) | 1 (converts with) | 0 | ❌ (wasted spend) |
| **Lost Cause** | 0 (won't convert) | 0 (still won't) | 0 | ❌ (cannot help) |
| **Sleeping Dog** | 1 (converts without) | 0 (campaign hurts) | −1 | ❌❌ (makes worse) |

A propensity model $\hat{P}(Y=1 \mid X)$ ranks Sure Things highest (they have
P(Y=1) close to 1 regardless of treatment). An uplift model ranks Persuadables
highest ($\tau(x)$ close to +1 for them). The difference determines campaign ROI.

*Reference: Radcliffe, N. J. and Surry, P. D. (1999). "Differential response analysis."
Free: stochasticsolutions.com*

---

## 7.4 T-Learner: Algorithm and Validity

### Algorithm

1. Partition data into treated ($T_i = 1$) and control ($T_i = 0$) groups
2. Train $\hat{\mu}_1(x)$ on treated data: regress/classify Y on X using only treated obs
3. Train $\hat{\mu}_0(x)$ on control data: regress/classify Y on X using only control obs
4. Estimate CATE: $\hat{\tau}(x) = \hat{\mu}_1(x) - \hat{\mu}_0(x)$

Any base learner can be used for steps 2–3: Random Forest, Gradient Boosted Trees,
Logistic Regression, Neural Networks.

### Unbiasedness Under Random Assignment

Under random assignment T ⊥ Y(0), Y(1):
$$E[\hat{\mu}_1(x)] = E[Y \mid X=x, T=1] = E[Y(1) \mid X=x, T=1] = E[Y(1) \mid X=x]$$

The last equality holds because T ⊥ Y(1) under randomization. Similarly:
$$E[\hat{\mu}_0(x)] = E[Y(0) \mid X=x]$$

Therefore: $E[\hat{\tau}(x)] = E[Y(1) - Y(0) \mid X=x] = \tau(x)$.

### Bias Under Non-Random Assignment

Without randomization, suppose higher-value customers (with high $Z_i$, unobserved)
are more likely to be treated: $P(T=1 \mid Z, X)$ is increasing in $Z$.

Then:
$$E[\hat{\mu}_1(x)] = E[Y \mid X=x, T=1] = E[Y(1) + \delta_Z \mid X=x, T=1]$$

where $\delta_Z$ captures the advantage of high-Z customers. Since T=1 is correlated
with high Z, $E[\hat{\mu}_1(x)]$ is inflated. The T-learner CATE estimate:
$$E[\hat{\tau}(x)] = \tau(x) + \underbrace{E[\delta_Z \mid X=x, T=1] - E[\delta_Z \mid X=x, T=0]}_{\text{selection bias} > 0}$$

The bias is positive (we overestimate τ) and depends on the unknown selection
mechanism — we cannot correct it by adding more features.

### S-Learner and X-Learner Alternatives

**S-Learner:** Train a single model $\hat{\mu}(x, t)$ on all data, including T as a feature.
$$\hat{\tau}(x) = \hat{\mu}(x, 1) - \hat{\mu}(x, 0)$$

**Limitation:** If T is uninformative in the training data (treatment is rare or
assigned systematically), the model shrinks β_T toward 0, underestimating CATE.

**X-Learner:** Three-stage procedure:
1. Estimate $\hat{\mu}_0$ and $\hat{\mu}_1$ as in T-learner
2. Impute individual treatment effects:
   - Treated: $\tilde{\tau}_i = Y_i - \hat{\mu}_0(X_i)$ (observed − predicted control)
   - Control: $\tilde{\tau}_i = \hat{\mu}_1(X_i) - Y_i$ (predicted treated − observed)
3. Regress $\tilde{\tau}$ on X separately in each group; combine with propensity weighting

X-learner is more efficient when treatment groups are highly imbalanced or when τ
varies strongly across subgroups.

*Reference: arXiv:1706.03461, Künzel et al. — cited above*
*Reference: arXiv:1510.04342, Wager and Athey (2018). "Estimation and Inference of
Heterogeneous Treatment Effects using Random Forests." arxiv.org/abs/1510.04342*

---

## 7.5 Targeting Threshold: Full Derivation

### Setup

- Campaign cost per customer: c
- Conversion value per customer: v
- Predicted CATE for customer i: τ̂(Xᵢ)

**Expected incremental revenue** from treating customer i:
$$\text{Revenue}(X_i) = \taû(X_i) \cdot v$$

**Expected profit** from treating customer i:
$$\text{Profit}(X_i) = \taû(X_i) \cdot v - c$$

**Decision rule:** Treat customer i iff $\text{Profit}(X_i) > 0$:
$$\taû(X_i) \cdot v - c > 0 \iff \taû(X_i) > \frac{c}{v}$$

### Why τ̂ > 0 Is Not Sufficient

If c = $4 and v = $30:
- Threshold = 4/30 = 0.1333
- Customer with τ̂ = 0.05: expected profit = 0.05 × $30 − $4 = $1.50 − $4 = **−$2.50**
- Customer with τ̂ = 0.20: expected profit = 0.20 × $30 − $4 = $6.00 − $4 = **+$2.00**
- Customer with τ̂ = −0.05: expected profit = −0.05 × $30 − $4 = **−$5.50**

Any customer with 0 < τ̂ < c/v generates positive incremental conversions but
negative expected profit. The τ̂ > 0 rule targets them unnecessarily.

### Optimality of the Threshold Rule

Under independence of customers, the total expected profit from targeting a set T:
$$\Pi(T) = \sum_{i \in T} [\taû(X_i) \cdot v - c]$$

This is maximized by including exactly those customers with $\taû(X_i) \cdot v - c > 0$,
i.e., $\taû(X_i) > c/v$. Excluding any customer with positive expected profit reduces
$\Pi$; including any customer with negative expected profit reduces $\Pi$.

---


### Worked Numerical Example

Dataset: 10,000 customers, random assignment (50/50 split).
Campaign: $4 cost per customer, $30 conversion value.
T-learner output (select customers):

| Customer | x₁=recency | x₂=frequency | μ₁(x) | μ₀(x) | τ̂(x) | Profit | Target? |
|---|---|---|---|---|---|---|---|
| A | 7 days | 5 | 0.42 | 0.18 | **+0.24** | 0.24×30−4=+$3.20 | ✅ |
| B | 30 days | 8 | 0.35 | 0.22 | **+0.13** | 0.13×30−4=−$0.10 | ❌ |
| C | 3 days | 12 | 0.88 | 0.79 | **+0.09** | 0.09×30−4=−$1.30 | ❌ Sure Thing |
| D | 60 days | 2 | 0.08 | 0.11 | **−0.03** | −0.03×30−4=−$4.90 | ❌ Sleeping Dog |
| E | 14 days | 6 | 0.31 | 0.16 | **+0.15** | 0.15×30−4=+$0.50 | ✅ |

Threshold = 4/30 = 0.133. Target customers A and E only.

**ATE from experiment:** (treated conversions − control conversions) = (0.24+0.35+0.88+0.08+0.31)/5 − (0.18+0.22+0.79+0.11+0.16)/5 = 0.372 − 0.292 = **0.08** (8 pp average treatment effect)

**Campaign profit if targeting τ̂ > 0 only:** Target A,B,C,E (τ̂>0). Expected profit = 3.20 + (−0.10) + (−1.30) + 0.50 = +$2.30.
**Campaign profit using threshold τ̂ > 0.133:** Target A,E only. Expected profit = 3.20 + 0.50 = **+$3.70** — 61% more profitable by applying the threshold.

## 7.6 Qini Curve and Coefficient

### Qini Curve Construction

1. Sort all customers in the test set by $\hat{\tau}(x)$ in descending order
2. Define targeting depth $\phi \in [0, 1]$ as the fraction of customers targeted
3. At each depth $\phi$, compute:
$$Q(\phi) = \frac{\text{Incremental conversions among top-}\phi\text{ fraction}}{\text{Total test set size}}$$

Incremental conversions = treated conversions − expected control conversions at this depth.

4. The **random targeting baseline**: $Q_{rand}(\phi) = \phi \cdot \tau_{ATE}$
   (distributes uplift proportionally to targeting depth)

### Qini Coefficient

$$q = \int_0^1 [Q(\phi) - Q_{rand}(\phi)] \, d\phi$$

Approximated as the trapezoidal area between the Qini curve and the random baseline.

**Interpretation:**
- q = 0: no ranking ability (curve coincides with random baseline)
- q = 1: perfect ranking (all Persuadables ranked before all other types)
- Negative q: model is worse than random (possibly from severe selection bias)

**What Qini does NOT measure:**
- The fraction of Persuadables correctly identified
- The absolute number of incremental conversions
- Model accuracy or AUC in the standard classification sense

*Reference: Radcliffe, N. J. (2007). "Using control groups to target on predicted lift:
Building and assessing uplift models." Free: stochasticsolutions.com/pdf/dma-jan-2007.pdf*

---

## 7.7 Full Proof Appendix

### Proof: ATE Estimator is Unbiased Under Random Assignment

Let T_i ∈ {0,1} be the treatment indicator, assigned by complete random assignment.
Under randomization: E[Y(t)|T] = E[Y(t)] for t ∈ {0,1} (treatment independent of potential outcomes).

$$E[\hat{\tau}_{ATE}] = E[\bar{Y}_{treated}] - E[\bar{Y}_{control}]$$

$$= E[Y|T=1] - E[Y|T=0]$$

$$= E[Y(1)|T=1] - E[Y(0)|T=0]$$

$$= E[Y(1)] - E[Y(0)] \quad \text{(by random assignment)}$$

$$= \tau_{ATE}$$

QED. Under Bernoulli randomization (each customer independently assigned), the same
argument applies since T_i ⊥ Y_i(0), Y_i(1) by construction.

### Proof: Targeting Threshold Rule Maximizes Expected Total Profit

The total expected profit from targeting set T ⊆ {1,...,n} is:
$$\Pi(T) = \sum_{i \in T} \pi_i, \quad \text{where } \pi_i = \taû(X_i) v - c$$

Since customer profits are independent, the optimal strategy is:
$$T^* = \{i : \pi_i > 0\} = \{i : \taû(X_i) > c/v\}$$

Any deviation — adding a customer with $\pi_i < 0$ or removing a customer with
$\pi_i > 0$ — strictly reduces $\Pi(T)$. QED.

---

## 7.8 Verification Checklist

| Check | How to verify |
|---|---|
| ATE = treated_rate − control_rate? | Simple difference of group means |
| T-learner requires random assignment? | Always true; violation → selection bias |
| Threshold = c/v, not c alone? | $4 campaign, $30 value → threshold = 0.133 |
| τ̂ > 0 ≠ τ̂ > c/v? | Compute expected profit: τ̂v − c |
| Sleeping dogs: τ̂ < 0 → do not target? | Even with unlimited budget |
| Qini = ranking quality, not % persuadables? | Common misinterpretation |
| P(alive)... no wait, P(T_i ⊥ Y_i) verified? | Check randomization procedure |
| Expected profit vs ATE: different quantities? | ATE aggregates; profit is per-customer |

---

## 7.9 Chapter Bibliography

1. Wikipedia, "Average treatment effect" — en.wikipedia.org/wiki/Average_treatment_effect
2. Wikipedia, "Rubin causal model" — en.wikipedia.org/wiki/Rubin_causal_model
3. Wikipedia, "Uplift modelling" — en.wikipedia.org/wiki/Uplift_modelling
4. Holland, P. W. (1986). "Statistics and Causal Inference." JASA 81(396):945–960.
5. Künzel, S. R., Sekhon, J. S., Bickel, P. J., Yu, B. (2019). "Meta-learners for
   Estimating Heterogeneous Treatment Effects using Machine Learning." PNAS.
   arXiv:1706.03461. arxiv.org/abs/1706.03461
6. Wager, S. and Athey, S. (2018). "Estimation and Inference of Heterogeneous
   Treatment Effects using Random Forests." JASA 113(523):1228–1242.
   arXiv:1510.04342. arxiv.org/abs/1510.04342
7. Radcliffe, N. J. and Surry, P. D. (1999). "Differential response analysis."
   Free: stochasticsolutions.com
8. Radcliffe, N. J. (2007). "Using control groups to target on predicted lift."
   Free PDF: stochasticsolutions.com/pdf/dma-jan-2007.pdf
9. MIT OpenCourseWare, 14.385 Nonlinear Econometric Analysis — ocw.mit.edu
10. arXiv:1707.02641, Shalit et al. "Estimating individual treatment effect:
    Generalization bounds and algorithms." arxiv.org/abs/1707.02641

**Lenny's Podcast episode references:**
- Jackson Shuttleworth episode on retention at Duolingo — Lenny's Podcast
- Lauryn Isford episode on growth at Airtable — Lenny's Podcast
