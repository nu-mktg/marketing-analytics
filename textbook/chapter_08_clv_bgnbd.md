# Chapter 8: Customer Lifetime Value — BG/NBD and Gamma-Gamma
## Comprehensive Reference Guide

---

### Chapter Overview

This chapter develops the BG/NBD (Beta-Geometric/Negative Binomial Distribution)
model for predicting future customer purchases and the Gamma-Gamma model for expected
order value. Together they produce the most widely used probabilistic CLV model in
marketing science. Full derivations of the BG/NBD likelihood, P(alive) formula,
and expected future transactions are included.

**By the end of this chapter you should be able to:**
- Explain the four BG/NBD model assumptions and what each captures
- Identify the sufficient statistics (x, t_x, T) and explain why they are sufficient
- Interpret P(alive) for any customer given their transaction history
- Verify that CLV = E[transactions] × E[spend] (without discounting)
- Check the Gamma-Gamma independence assumption and interpret violations
- Set channel-specific CAC targets from CLV estimates

---

## 8.1 Strategic Frame: What Practitioners Know

### Patrick Campbell: LTV is a Hypothesis

Patrick Campbell built ProfitWell (acquired by Paddle) into the subscription analytics
platform used by thousands of SaaS and subscription companies to monitor revenue health.

His most important provocation about CLV: it is a hypothesis, not a fact. A CLV
number is only as reliable as the assumptions embedded in it — assumptions about:
- Future retention rates (will they remain at historical levels or change?)
- Future purchase frequency (will buying behavior stay the same?)
- Future average order value (will pricing or product mix change?)
- Discount rate (what is the right cost of capital to apply?)

Change any one of these assumptions by 10–15% and the CLV number can shift by 20–40%.
Companies that set acquisition budgets against a single CLV figure — without running
sensitivity analyses — are making precision decisions on genuinely imprecise inputs.

His operational recommendation: treat CLV as a band, not a point estimate. Run CLV
under optimistic, baseline, and pessimistic assumptions about each input. The range
of the resulting CLV values is honest information about the uncertainty you are
operating under.

### Albert Cheng: Retention as the Revenue Foundation

Albert Cheng led growth at Duolingo, Grammarly, and Chess.com — three of the most
widely used consumer software products in their respective categories. His synthesis
of what matters most:

"Growth as the job is to connect users to the value of your product." CLV is the
financial summary of how well that connection has been made.

His specific contribution to the CLV discussion: historical purchase behavior (the
inputs to BG/NBD: x, t_x, T) and future retention potential (what product investment
can improve) are different quantities. BG/NBD predicts future purchases from past
patterns — it cannot incorporate the impact of a product change that hasn't happened
yet or a new use case that customers are only beginning to discover.

He frames the limitation precisely: "User retention is gold for consumer subscription
companies. If you don't retain your users, then a lot of the onus is on getting them
to pay on day one." A CLV model tells you the current expected value; it cannot tell
you what CLV would be after a successful retention intervention. That requires
combining the CLV model with a survival model and an uplift model — exactly the
integration this course enables.

### Three Strategic Insights

**Insight 1: The BG/NBD model separates "is this customer still active?" from "how often do they buy?"**

Most ad-hoc CLV models compute average monthly revenue per customer and multiply by
an assumed retention period. This conflates two separate questions:
- P(alive): Is this customer still part of your addressable base?
- E[purchases | alive]: If still active, how often will they buy?

BG/NBD answers both separately. A customer with 15 purchases in 24 months (high
frequency) but last purchase 20 months ago (low recency) has very different P(alive)
from a customer with 5 purchases spread evenly over 24 months. The naive model treats
them similarly; BG/NBD captures the signal from the recent silence.

**Insight 2: CLV by acquisition channel transforms CAC decisions.**

Most marketing teams track cost-per-acquisition (CAC) by channel but not CLV by
channel. This optimizes the denominator of the ROI ratio while ignoring the numerator.

A channel with high CAC but high CLV (e.g., enterprise sales with $2,000 CAC and
$12,000 CLV) is more efficient than one with low CAC but low CLV (e.g., broad social
acquisition at $25 CAC and $80 CLV). The 3:1 LTV:CAC ratio is $12,000/$2,000 = 6.0
vs. $80/$25 = 3.2 — enterprise is the winner despite higher absolute CAC.

**Insight 3: The Gamma-Gamma independence assumption fails more often than analysts admit.**

The standard implementation of Gamma-Gamma assumes spend per transaction is
independent of purchase frequency. This holds approximately in some businesses
(e.g., monthly SaaS subscriptions where every transaction is the same amount) and
fails substantially in others (e.g., grocery retail where high-frequency shoppers
make smaller top-up purchases and low-frequency shoppers make large stock-up trips).

Always plot average order value against purchase frequency before applying Gamma-Gamma.
A correlation of magnitude > 0.30 warrants either a different model or explicit
acknowledgment of the bias direction in your CLV estimates.

---

## 8.2 Mathematical Prerequisites

### 8.2.1 Poisson Process

A **Poisson process** {N(t), t ≥ 0} counts events occurring at constant rate λ:
$$P(N(t) = k) = \frac{(\lambda t)^k e^{-\lambda t}}{k!}, \quad k = 0, 1, 2, \ldots$$

**Key properties:**
- Events are independent across non-overlapping time intervals
- Inter-event times are Exponential(λ): $f(s) = \lambda e^{-\lambda s}$
- E[N(t)] = λt (average number of events in time t)
- Var[N(t)] = λt

*Reference: Wikipedia, "Poisson process" — en.wikipedia.org/wiki/Poisson_point_process*

### 8.2.2 Gamma Distribution

$X \sim \text{Gamma}(r, \alpha)$ with shape r > 0 and rate α > 0:
$$f(x; r, \alpha) = \frac{\alpha^r}{\Gamma(r)} x^{r-1} e^{-\alpha x}, \quad x > 0$$

**Properties:**
- E[X] = r/α
- Var[X] = r/α²
- Gamma(1, α) = Exponential(α)
- Gamma is conjugate prior to Poisson rate: if λ ~ Gamma(r, α) and k | λ ~ Poisson(λT),
  then λ | k ~ Gamma(r + k, α + T)

*Reference: Wikipedia, "Gamma distribution" — en.wikipedia.org/wiki/Gamma_distribution*

### 8.2.3 Beta Distribution (Review from Chapter 1)

$p \sim \text{Beta}(a, b)$: mean = a/(a+b), conjugate to Bernoulli.
Used in BG/NBD for the distribution of dropout probabilities across customers.

### 8.2.4 The Beta Function

$$B(a, b) = \int_0^1 t^{a-1}(1-t)^{b-1} \, dt = \frac{\Gamma(a)\Gamma(b)}{\Gamma(a+b)}$$

This appears in BG/NBD normalizing constants and the closed-form likelihood.

---

## 8.3 BG/NBD Model: Assumptions and Structure

### The Four Assumptions

**Assumption 1:** While active, customer i makes purchases at Poisson rate λᵢ.
Individual purchase rate λᵢ is constant over the customer's active lifetime.

**Assumption 2:** After each purchase, customer i becomes permanently inactive
(drops out of the addressable market) with probability pᵢ. This is a Bernoulli
process independent across purchases.

**Assumption 3:** Cross-customer heterogeneity in purchase rates:
$$\lambda_i \sim \text{Gamma}(r, \alpha)$$
where r and α are population parameters estimated from data.

**Assumption 4:** Cross-customer heterogeneity in dropout probabilities:
$$p_i \sim \text{Beta}(a, b)$$
where a and b are population parameters estimated from data.

**Assumption 5 (independence):** λᵢ and pᵢ are independent across customers and
independent of each other for any individual customer.

### Why These Assumptions Are Useful

The Poisson purchase process is a well-known model for repeat purchase behavior.
The geometric dropout model (Bernoulli after each purchase) is the simplest model
that allows dropout at any point while keeping the timing unobservable to the analyst.
Population-level Gamma and Beta distributions are mathematically tractable conjugates
to Poisson and Bernoulli, enabling closed-form likelihood expressions.

The net result: a model that requires only (x, t_x, T) per customer — no demographics,
no product usage data, no browsing behavior — and produces both P(alive) and E[future
purchases] for every customer.

*Reference: Fader, P. S., Hardie, B. G. S., and Lee, K. L. (2005). "'Counting Your
Customers' the Easy Way." Marketing Science 24(2):275–284. Free PDF:
brucehardie.com/papers/018/fader_et_al_mksc_05.pdf*

---

## 8.4 Sufficient Statistics and P(alive)

### Why (x, t_x, T) Are Sufficient

For a customer observed over period [0, T]:
- **x** = number of repeat purchases (total purchases minus 1)
- **t_x** = time of most recent purchase (measured from start of observation)
- **T** = total length of observation period

These three numbers capture everything in the transaction history relevant to
estimating future behavior. The specific timing of intermediate purchases (between
t=0 and t_x) is not needed because the Poisson/geometric model's sufficient statistic
for each individual is exactly (x, t_x, T).

### Intuition for P(alive)

The longer the time since last purchase (T − t_x), the stronger the evidence that the
customer may have dropped out. The model evaluates two scenarios:

**Scenario A:** Customer is still alive. They made x purchases in [0, t_x], then
made no purchases in [t_x, T] by chance (even though they could have).

**Scenario B:** Customer dropped out at some time τ ∈ (t_x, T). They made x
purchases in [0, t_x], bought once just before dropping out, and have been inactive
since.

$$P(\text{alive} \mid x, t_x, T) = \frac{P(\text{Scenario A})}{P(\text{Scenario A}) + P(\text{Scenario B})}$$

As (T − t_x) increases relative to the expected inter-purchase time (1/λ), the
relative probability of Scenario B grows. A customer who was buying every 2 months
and hasn't bought in 18 months is almost certainly in Scenario B.

### The Full Likelihood

The joint likelihood for customer i with history (xᵢ, t_{x,i}, Tᵢ) integrates over
all possible individual parameters (λᵢ, pᵢ):

$$L_i(r, \alpha, a, b) = \int_0^\infty \int_0^1 L_i(\lambda, p) f(\lambda; r, \alpha) f(p; a, b) \, dp \, d\lambda$$

Fader et al. (2005) showed this integral has a closed form:
$$L_i = A_0 \left\{\frac{B(a, b+x)}{B(a,b)} \delta(x>0) \left[\sum_{j=0}^{x-1} \frac{B(a+1, b+j)}{B(a,b+j)} A_1^{x-j-1} A_2\right] + \frac{B(a+1, b+x)}{B(a,b)} A_1^x \right\}$$

where $A_0$, $A_1$, $A_2$ involve the Gamma function evaluated at the population
parameters. The full expression is given in Appendix B of Fader et al. (2005) and
computed automatically by the lifetimes Python library.

> **Teaching note:** Do not attempt to derive this on the board — the full likelihood
> is too complex for a class session. Instead: explain the two-scenario structure
> (alive vs. dead) qualitatively, show the P(alive) formula as a ratio of two
> probabilities, and demonstrate numerically that recency dominates frequency
> using the worked example below.

---

## 8.5 P(alive): Worked Examples

| Customer | x purchases | t_x (last purchase) | T (obs. period) | P(alive) |
|---|---|---|---|---|
| A | 8 | 11.5 months ago | T=12 | **0.93** ← very recent |
| B | 8 | 2 months ago | T=12 | **0.63** ← less recent despite same frequency |
| C | 8 | 20 months ago | T=24 | **0.09** ← long silence |
| D | 1 | 1 month ago | T=12 | **0.47** ← recent but infrequent |

**Key observations:**
- A vs B: same frequency (x=8), different recency → very different P(alive)
- A vs C: same frequency (x=8), much larger (T−t_x) for C → dramatically lower P(alive)
- A vs D: same recency, A bought much more frequently → higher P(alive) for A

The BG/NBD correctly captures that recency is more informative than frequency about
current activity status. This is the model's central insight.

---

## 8.6 Gamma-Gamma Spend Model

### Model Assumptions

The Gamma-Gamma model (Fader and Hardie, 2013) estimates expected spend per
transaction:

**Assumption 1:** Conditional on being "alive," the spend on transaction t for
customer i follows $z_{it} \sim \text{Gamma}(p, \nu_i)$ where $\nu_i$ is
customer-specific and p is a population-level shape parameter.

**Assumption 2:** The individual rate νᵢ varies across customers as $\nu_i \sim \text{Gamma}(q, \gamma)$.

**Assumption 3 (independence):** νᵢ is independent of the transaction rate λᵢ.
This means a customer's spending level per transaction is unrelated to how frequently
they buy.

### Expected Spend per Transaction

Integrating over the Gamma-Gamma hierarchy:
$$E[z \mid p, q, \gamma] = \frac{p\gamma}{q-1} \quad (q > 1)$$

For a specific customer with x purchases and average spend $\bar{z}$:
$$E[z \mid x, \bar{z}, p, q, \gamma] = \frac{p\gamma + \bar{z} \cdot x}{pq + x - 1}$$

This is a weighted average of the population mean and the customer's observed mean,
shrinking toward the population when x is small.

### The Independence Assumption

The model requires $\text{Corr}(\lambda_i, \nu_i) = 0$: purchase frequency and spend
per transaction are independent across customers.

**When this holds:** SaaS subscriptions (every transaction is the same amount),
commodity purchases (price is fixed), media subscriptions.

**When this fails:** Grocery retail (frequent small baskets vs. infrequent large hauls),
luxury retail (occasional big splurges), event-driven buying (customers buy more when
they buy and less at other times).

**Detection:** Compute Pearson correlation between customer-level purchase frequency
(x/T) and average order value ($\bar{z}$) in your dataset.
- |ρ| < 0.10: assumption approximately satisfied
- 0.10 < |ρ| < 0.30: mild violation, proceed with caution
- |ρ| > 0.30: material violation, CLV estimates biased for high/low-frequency segments

**Bias direction if violated:**
- ρ < 0 (high frequency → low spend): Gamma-Gamma overestimates CLV for high-frequency customers
- ρ > 0 (high frequency → high spend): Gamma-Gamma underestimates CLV for high-frequency customers

*Reference: Fader, P. S. and Hardie, B. G. S. (2013). "The Gamma-Gamma Model of
Monetary Value." Free notes: brucehardie.com/notes/025/*

---

## 8.7 CLV Computation

### Without Discounting

$$\text{CLV}_{12m} = E[\text{purchases in next 12m}] \times E[\text{spend per purchase}]$$

**Example:** $E[\text{purchases}] = 3.6$, $E[\text{spend}] = \$55$
$$\text{CLV} = 3.6 \times \$55 = \$198$$

### With Monthly Discounting

$$\text{CLV}_{12m} = \sum_{t=1}^{12} \frac{E[\text{purchases in month }t] \times E[\text{spend}]}{(1+\delta)^t}$$

For monthly discount rate δ = 0.01 (12% annually):
$$\text{CLV}_{12m} = E[\text{spend}] \times \sum_{t=1}^{12} \frac{E[P_t]}{(1.01)^t}$$

where $E[P_t]$ is the expected purchase count in month t (computed from the fitted
BG/NBD model).

### CLV Distribution Properties

CLV distributions in subscription businesses are typically:
- **Right-skewed:** Mean >> Median (a small fraction of very high-CLV customers
  pulls the mean up)
- **Heavy-tailed:** Top 10% of customers often have 4–6× the median CLV
- **Bimodal (sometimes):** A cluster of very short-lived customers and a cluster
  of long-term loyalists

**Practical implication:** Report median CLV for typical customer value and mean CLV
for total portfolio value calculations. Mean CLV × total customers = expected
portfolio revenue.

---

## 8.8 Setting CAC Targets by Channel

### The LTV:CAC Framework

$$\text{LTV:CAC ratio} = \frac{\text{CLV}}{\text{CAC}}$$

Industry benchmarks:
- LTV:CAC < 1: Unprofitable channel (spending more than customer is worth)
- LTV:CAC = 1–2: Marginal; likely not viable long-term
- LTV:CAC = 3: Generally considered healthy minimum for SaaS
- LTV:CAC > 5: Potentially underspending — could acquire more customers profitably

**Channel-specific targeting:**

| Channel | CLV | Max sustainable CAC (at 3:1) |
|---|---|---|
| Branded organic search | $420 | $140 |
| Content/SEO | $380 | $127 |
| Paid social | $190 | $63 |
| Cold email | $140 | $47 |

Using overall average CLV ($280 in this example) would set a single CAC target of
$93 — over-investing in cold email (sustainable CAC is $47) and under-investing in
organic channels (sustainable CAC is up to $140).

---

## 8.9 Full Proof Appendix

### Proof: E[future purchases] under BG/NBD

The expected number of transactions in (T, T+t] for a customer still alive at T
requires integrating over the Gamma and Beta distributions of λ and p. Fader et al.
(2005) show:

$$E[Y(t) \mid x, t_x, T, r, \alpha, a, b] = \frac{A_3}{1 - (A_3)} \cdot \left[1 - \left(\frac{\alpha+T}{\alpha+T+t}\right)^{r+x} \cdot \frac{B(a+1,b+x)}{B(a,b+x)}\right]$$

where $A_3$ involves the model parameters and customer history. Full derivation in
Fader et al. (2005) Appendix B, freely available at brucehardie.com.

### Proof: Gamma-Gamma Bias Direction Under Negative Correlation

If Corr(λ_i, ν_i) = ρ < 0 (high-frequency customers spend less per transaction):

Under the Gamma-Gamma model (assuming ρ = 0):
$$E[\hat{z} \mid \lambda \text{ is high}] = E[z] = p\gamma/(q-1) \quad \text{(population mean)}$$

True expected spend for high-λ customers:
$$E[z \mid \lambda \text{ is high}] < E[z] \quad \text{(since ρ < 0)}$$

Therefore: $E[\text{CLV} \mid \lambda \text{ is high, GG model}] > E[\text{CLV} \mid \lambda \text{ is high, true model}]$

The Gamma-Gamma model overestimates CLV for high-frequency customers when ρ < 0. QED.

---

## 8.10 Verification Checklist

| Check | How to verify |
|---|---|
| Sufficient statistics: only (x, t_x, T)? | No demographics or browsing data needed |
| Higher recency → higher P(alive) (same frequency)? | Compare customers A and C from table |
| P(alive) computed by lifetimes correctly? | Check against Hardie's Excel calculator at brucehardie.com |
| CLV = E[transactions] × E[spend] (no discounting)? | Simple multiplication |
| GG correlation check: |ρ| < 0.30? | Always compute before applying model |
| ρ < 0: CLV overstated for high-frequency segment? | Note bias direction in report |
| LTV:CAC computed separately per channel? | Not using average CLV |
| CLV median vs mean: right-skewed? | Mean > median confirms skew |

---

## 8.11 Chapter Bibliography

1. Wikipedia, "Customer lifetime value" — en.wikipedia.org/wiki/Customer_lifetime_value
2. Wikipedia, "Gamma distribution" — en.wikipedia.org/wiki/Gamma_distribution
3. Wikipedia, "Poisson process" — en.wikipedia.org/wiki/Poisson_point_process
4. Wikipedia, "Beta distribution" (review) — en.wikipedia.org/wiki/Beta_distribution
5. Fader, P. S., Hardie, B. G. S., and Lee, K. L. (2005). "'Counting Your Customers'
   the Easy Way: An Alternative to the Pareto/NBD Model." Marketing Science 24(2).
   Free PDF: brucehardie.com/papers/018/fader_et_al_mksc_05.pdf
6. Fader, P. S. and Hardie, B. G. S. (2013). "The Gamma-Gamma Model of Monetary Value."
   Free notes and Excel implementation: brucehardie.com/notes/025/
7. Hardie, B. G. S. BG/NBD model Excel spreadsheet. Free: brucehardie.com/notes/004/
8. Davidson-Pilon, C. lifetimes: survival analysis and customer lifetime value.
   Documentation: lifelines.readthedocs.io
9. McCarthy, D. and Fader, P. (2018). "Customer-Based Corporate Valuation for
   Publicly Traded Non-Contractual Firms." arXiv:1703.04108. arxiv.org/abs/1703.04108

**Lenny's Podcast episode references:**
- Patrick Campbell episode on subscription analytics and LTV — Lenny's Podcast
- Albert Cheng episode on consumer growth — Lenny's Podcast
