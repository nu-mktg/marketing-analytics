# Lecture 8: Customer Lifetime Value (BG/NBD)
## How Much Is a Customer Worth — and Are They Still There?

---

### Overview

**Business question:** Two customers each made 12 purchases. Customer A's last purchase was last month. Customer B's last purchase was 19 months ago. Which is more valuable for the next 12 months — and by how much?

**What you will be able to do:**
- Explain why recency matters more than frequency for predicting future behavior
- Interpret P(alive) estimates from BG/NBD output
- Compute a simple CLV estimate from model outputs
- Identify when the Gamma-Gamma independence assumption is violated
- Use CLV by acquisition channel to set channel-specific CAC targets

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Weighted Average

A weighted average assigns different importance to different values.

**Example:** CLV is the expected number of future purchases × expected spend per purchase. If a customer has E[purchases] = 3.2 and E[spend] = $55:
> CLV = 3.2 × $55 = **$176**

---

#### Tool 2: Why Silence Is Evidence of Dropout

A customer who bought 12 times in 2 years and then went silent for 19 months is probably not still an active customer — they have likely dropped out. The model must distinguish between:
- A customer who buys infrequently (low λ) — could have a long gap naturally
- A customer who has churned and will never buy again

The BG/NBD model handles this formally. The key insight: the longer the silence relative to previous purchase cadence, the stronger the evidence of dropout.

---

### Section 1.2 — Business Motivation

---

#### LTV Is a Hypothesis

Patrick Campbell built ProfitWell (later acquired by Paddle) into a subscription analytics platform used by thousands of SaaS companies. His core provocation:

> *"Treat LTV as a hypothesis to test, not a fact to execute against."*

A CLV number is only as reliable as the assumptions embedded in it — assumptions about future retention, future purchase frequency, future average order value, and a discount rate. Change any one of these by 10–15% and the CLV shifts substantially. Companies that set customer acquisition budgets against a single CLV figure are making precision decisions on imprecise inputs.

Albert Cheng, who led growth at Duolingo, Grammarly, and Chess.com, frames the same issue from the product side:

> *"User retention is gold for consumer subscription companies. If you don't retain your users, then a lot of the onus is on getting them to pay on day one."*

The BG/NBD model separates the "still retained" question (P(alive)) from the "how much will they spend" question (Gamma-Gamma). This is its core insight: historical cash flow and future value are different things.

---

### Section 1.3 — Conceptual Framework

---

#### The Two-Stage Model

BG/NBD estimates CLV in two stages:

**Stage 1 — P(alive) via BG/NBD:** Given a customer's (x, t_x, T) — purchase count, time of last purchase, observation period — what is the probability they are still an active customer?

**Stage 2 — Expected spend via Gamma-Gamma:** Given that a customer makes a future purchase, how much will they spend?

**Combined:** CLV = E[future purchases] × E[spend per purchase | makes a purchase]

The model only needs three numbers per customer: x (how many times they bought), t_x (when they last bought), and T (how long you have been observing them). No demographics, no browsing data — just transaction history.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: The BG/NBD Inputs

For each customer, compute:
- **x** = number of repeat purchases (total purchases minus 1, since the first purchase defines acquisition)
- **t_x** = time of most recent purchase (in whatever unit: months, weeks)
- **T** = total observation period length (same unit as t_x)

**Example:** Customer B bought 12 times over 24 months. Last purchase at month 5 (19 months before the observation end at month 24):
> x = 11, t_x = 5, T = 24

The model fits population-level parameters (r, α for purchase rate distribution; a, b for dropout probability distribution) by maximizing the likelihood across all customers. You do not need to perform this estimation by hand — the agent does it.

**Reading P(alive):**
- Customer A: x=12, t_x=23 (recent), T=24 → P(alive) ≈ 0.93
- Customer B: x=12, t_x=5 (19 months ago), T=24 → P(alive) ≈ 0.09

Both bought 12 times. The difference in P(alive) comes entirely from recency. Customer B's 19 months of silence is strong evidence of dropout.

---

#### Part B: Simple CLV Calculation

Once the agent provides E[purchases in next 12 months] and E[order value], CLV is:

> CLV₁₂ = E[transactions] × E[order value]

(For more precision, apply a monthly discount rate: CLV = Σ (E[purchases in month t] × E[order value]) / (1 + discount_rate)^t)

**Example:** E[transactions] = 2.4, E[order value] = $55 → CLV = 2.4 × $55 = **$132**

---

#### Part C: The Gamma-Gamma Independence Assumption

The Gamma-Gamma model for spend assumes that a customer's average order value is independent of their purchase frequency. If high-frequency buyers tend to make smaller purchases (as often happens in grocery retail), this assumption is violated.

**How to check:** Compute the Pearson correlation between each customer's purchase frequency and their average order value. A correlation of −0.30 or stronger is a material violation.

**What to do if violated:** The CLV estimates for high-frequency customers will be systematically overestimated. Add a note of caution to any recommendation based on CLV for high-frequency customer segments.

> ### 🔍 Deep Dive: The Full BG/NBD Likelihood
> The BG/NBD models each customer's purchase process as a Poisson process (events arriving randomly at rate λ) and their dropout process as a Bernoulli trial after each purchase (drop out with probability p). Both λ and p vary across customers: λ ~ Gamma(r, α) and p ~ Beta(a, b). The likelihood for a customer (x, t_x, T) involves computing the probability of observing exactly x purchases, with the last at t_x, given that the customer either dropped out sometime after t_x or is still alive at T. This likelihood has a closed-form expression involving the Beta function — which is why the model is tractable. The four parameters (r, α, a, b) are estimated by maximizing the sum of log-likelihoods across all customers.

---

### Part 1 Checkpoint

1. Three customers with identical purchase counts (x=8) but different recency. Customer A: last purchase 1 month ago (T=12). Customer B: last purchase 6 months ago (T=12). Customer C: last purchase 11 months ago (T=12). Rank from highest to lowest P(alive).

2. E[transactions in 12 months] = 3.6, E[order value] = $45. Compute CLV (no discounting).

3. Gamma-Gamma correlation = −0.48. Is the independence assumption satisfied?

4. CAC for Paid Social = $35, predicted 12m CLV = $85. CAC for Organic = $8, predicted 12m CLV = $210. Compute CLV:CAC ratio for each. Which channel is more efficient?

5. A marketing manager says "Our average CLV is $210, so we can spend up to $210 to acquire any customer." What is wrong with this reasoning?

---

### Checkpoint Answer Key

**Q1.** A > B > C. With identical purchase frequency, P(alive) is entirely determined by recency. Customer A bought 1 month ago (very recent) → highest P(alive). Customer C bought 11 months ago (long silence relative to 12-month observation period) → lowest P(alive).
*Common wrong answer:* All equal because they all made 8 purchases. Frequency and recency are both inputs to BG/NBD; when frequency is controlled, recency alone determines P(alive) differences.

**Q2.** CLV = 3.6 × $45 = **$162**.
*Common wrong answer:* $162 × (1/(1+0.01))^6 ≈ $152.7 — applying a discount rate that was not asked for. The question specifies "no discounting."

**Q3.** **No — violated.** The Gamma-Gamma model assumes spend ⊥ frequency. A correlation of −0.48 (high-frequency buyers spend less per order) is a material violation. The model will overestimate CLV for high-frequency, low-spend customers.
*Common wrong answer:* Satisfied because −0.48 is close to 0 (confusing "close to zero" with "close to negative one"). −0.48 is a moderately strong negative correlation — well past the −0.30 warning threshold.

**Q4.** Paid Social CLV:CAC = 85/35 = **2.43**. Organic CLV:CAC = 210/8 = **26.25**. Organic is dramatically more efficient. However, organic channels may not be scalable — you cannot simply buy more organic growth the way you buy more paid social.
*Common wrong answer:* Paid Social has a ratio > 1, so it is profitable and should continue at current scale. While profitable, the comparison to organic shows it generates far less value per dollar invested.

**Q5.** The $210 average CLV masks enormous variation across customer segments and acquisition channels. CLV for customers acquired through a high-intent organic search keyword may be $400; CLV for customers acquired through a broad awareness campaign may be $80. Using the average to set CAC targets means overpaying for low-CLV segments and potentially underspending on high-CLV ones.
*Common wrong answer:* Nothing is wrong — using the average is a reasonable simplification. The simplification is only reasonable if CLV does not vary by acquisition channel, which it almost always does.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work through Parts B–C together)


---

**Problem Statement**

A subscription box company has fitted a BG/NBD + Gamma-Gamma model to their customer transaction data. The fitted parameters are:

| Parameter | Value | Interpretation |
|---|---|---|
| $r$ | 0.85 | Shape of Gamma for purchase rates |
| $\alpha$ | 8.20 | Scale of Gamma for purchase rates; mean rate = $r/\alpha = 0.104$ per month |
| $a$ | 0.55 | Beta shape parameter for dropout |
| $b$ | 3.80 | Beta shape parameter; mean dropout prob per purchase = $a/(a+b) = 0.126$ |
| $p$ | 6.25 | Gamma-Gamma shape |
| $q$ | 3.74 | Gamma-Gamma shape |
| $\gamma$ | 15.45 | Gamma-Gamma scale; population mean spend ≈ $p/\gamma \times q/(q-1) \approx \$42$ |

Three customers are to be evaluated:

| Customer | Purchases ($x$) | Observation period ($T$, months) | Last purchase ($t_x$, months from start) |
|---|---|---|---|
| Valentina | 12 | 24 | 23.5 |
| Marcus | 12 | 24 | 6.0 |
| Priya | 2 | 24 | 20.0 |

**Part A:** Before running any model, rank these three customers by expected 12-month CLV. Explain your reasoning in one sentence for each.

**Part B:** The model produces these estimates:

| Customer | P(alive) | E[transactions, 12m] | E[spend/order] | CLV (12m) |
|---|---|---|---|---|
| Valentina | 0.94 | 4.8 | $38 | $182 |
| Marcus | 0.11 | 0.5 | $38 | $19 |
| Priya | 0.73 | 1.6 | $38 | $61 |

Does the ranking match your intuition from Part A? What explains each customer's position?

**Part C:** A marketing manager proposes spending $15 in retention budget per customer to prevent churn. For which customers is this spending justified? What is the minimum P(alive) that makes retention spending worthwhile, assuming that successful retention captures the full predicted CLV?

---

**Full Solution**

**Part A: Intuitive Ranking (before running model)**

**Expected ranking: Valentina > Priya > Marcus**

- **Valentina:** 12 purchases in 24 months (high frequency), last purchase 0.5 months ago (very recent). Almost certainly alive. High predicted CLV.
- **Priya:** Only 2 purchases, but recent (4 months ago). Low frequency suggests either new or slow buyer. Probably alive, moderate CLV.
- **Marcus:** 12 purchases — but all crammed into the first 6 months, then 18 months of complete silence. Despite high historical frequency, very likely churned. Low predicted CLV.

**Part B: Model Output Analysis**

The ranking matches the intuition: Valentina ($182) > Priya ($61) > Marcus ($19).

**Valentina** (P(alive)=0.94): Recent purchase confirms she is almost certainly active. High frequency means high expected future transactions (4.8 over 12 months). At 1 purchase per month historically and mostly still active, this is consistent.

**Marcus** (P(alive)=0.11): Despite identical total purchases as Valentina, his 18-month silence is overwhelming evidence of dropout. Even if the model thinks there is an 11% chance he is still alive, that translates to only 0.5 expected purchases. Historical CLV = 12 × $38 = $456; future CLV = $19. This is a 24:1 ratio — the past massively overstates his future value.

**Priya** (P(alive)=0.73): Low frequency (2 purchases in 24 months) but recent. The model estimates a 73% chance she is still active. With a low base rate, her expected 1.6 purchases in 12 months is reasonable. Her low CLV reflects low frequency, not imminent churn.

**Part C: Retention Budget Justification**

A $15 retention spend is justified if the expected return exceeds the cost. Since we only recover CLV if the customer stays, the expected return is approximately P(alive) × CLV.

For each customer, compute expected return from retention spending:
- **Valentina:** Expected return = 0.94 × $182 = $171. Cost = $15. Net = +$156 ✓
- **Marcus:** Expected return = 0.11 × $19 = $2.09. Cost = $15. Net = −$13 ✗ — **do not spend**
- **Priya:** Expected return = 0.73 × $61 = $44.5. Cost = $15. Net = +$30 ✓

**Minimum P(alive) to justify $15 retention:**

If we assume the retention budget, if effective, captures the full CLV:
$\text{P(alive)} \times \text{CLV} > 15$
$\text{P(alive)} > 15 / \text{CLV}$

For Valentina: threshold = 15/182 = 0.082 (easily exceeded)
For Priya: threshold = 15/61 = 0.246 (easily exceeded)
For Marcus: threshold = 15/19 = 0.789 (not exceeded — P(alive) = 0.11)

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**P(alive) by customer:** Examine the distribution of P(alive) across your customer base. A healthy base has many customers with P(alive) > 0.7. If the majority are below 0.3, your customer base may be churning faster than expected.

**CLV decile analysis:** Sort customers by predicted CLV and examine the top decile (top 10%) vs. the bottom decile. The ratio of top-to-bottom CLV is a measure of customer heterogeneity. Ratios above 10× are common in e-commerce.

**"Hidden gems":** Customers with low historical CLV but high predicted CLV. These are often new or recently reactivated customers — their purchase history is short but recent. The BG/NBD identifies them as valuable future customers that historical analysis would underrank.

**Model fit check:** The lifetimes library provides a `plot_calibration_purchases_vs_holdout_purchases` function that compares predicted purchase counts to actual purchases in a holdout period. If the model systematically underpredicts for high-frequency customers, the Gamma assumption for $\lambda$ may be misspecified.

**Before trusting the agent output:**
1. Do BG/NBD parameters fall in plausible ranges? ($r$ typically 0.1–3, $\alpha$ typically 1–20)
2. Are P(alive) estimates intuitive? Recent buyers should have P(alive) > 0.8; long-silent buyers should have P(alive) < 0.2
3. Does mean predicted CLV across all customers seem reasonable relative to average historical order value?
4. Is the Gamma-Gamma independence assumption approximately satisfied? (Plot order value vs. frequency)

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/transactions.csv` + `homework_08.ipynb`

---

#### Part A: Math Questions (no agent required)

**Q1.** A customer purchases at rate $\lambda = 0.4$ per month. Under the Poisson model, what is the expected number of purchases in 6 months?
```python
q1_expected_purchases = None
```

**Q2.** Under the geometric dropout model, a customer remains active after each purchase with probability $p = 0.80$. What is the probability they survive (remain active) through 4 consecutive purchase opportunities?
```python
q2_survive_4 = None
```

**Q3.** The BG/NBD model has parameters $r = 1.2$ and $\alpha = 6.0$. What is the mean purchase rate across all customers?
```python
q3_mean_rate = None
```

**Q4.** A customer has E[transactions next 12 months] = 3.2 and expected order value = $55. What is their predicted 12-month CLV (ignoring discounting)?
```python
q4_clv_no_discount = None
```

**Q5.** With a 2% monthly discount rate, what is the present value of a $100 payment received 12 months from now? Use the formula PV = 100/(1.02)^12. Note: (1.02)^12 ≈ 1.268.
```python
q5_present_value = None
```

**Q6.** Customer X made 3 purchases in 24 months, last purchase was 2 months ago. Customer Y made 3 purchases in 24 months, last purchase was 22 months ago. Without running any model, which customer has a higher probability of being alive? Explain in one sentence.
```python
# Enter "X" or "Y"
q6_higher_p_alive = None
```

**Q7.** True or False: The BG/NBD model requires detailed demographic data (age, gender, income) to compute predicted CLV.
```python
q7_needs_demographics = None
```

**Q8.** A company's Gamma-Gamma model shows that customers who purchase 10+ times per year have average order values of $20, while customers who purchase 1–2 times per year have average order values of $80. Should the Gamma-Gamma model be applied to this data?
```python
# Enter "yes" or "no" and briefly explain
q8_gamma_gamma_applicable = None  # "yes" or "no"
```

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I am computing customer lifetime value for an e-commerce company.

transactions.csv has one row per transaction:
- customer_id
- transaction_date: date of purchase (YYYY-MM-DD format)
- order_value: revenue from this transaction in dollars

Please:
1. Use the lifetimes library to prepare the RFM summary data
   (frequency, recency, T, monetary_value) for each customer.
   Observation end date = the most recent date in the dataset.
2. Fit the BG/NBD model. Print the fitted parameters r, alpha, a, b.
3. Fit the Gamma-Gamma model. Print the fitted parameters p, q, v.
4. Compute predicted CLV for each customer for the next 12 months
   using a monthly discount rate of 1%.
5. Report: mean CLV, median CLV, and CLV at the 10th, 50th, 90th,
   and 99th percentiles.
6. Find the 5 customers with the highest predicted CLV.
7. Check the Gamma-Gamma assumption: plot average order value vs.
   purchase frequency (number of transactions). Report the correlation.
Use random seed 42 where applicable.
```

**Q9.** What is the fitted BG/NBD parameter $r$? Round to 2 decimal places.
```python
q9_param_r = None
```

**Q10.** What is the fitted BG/NBD parameter $\alpha$? Round to 2 decimal places.
```python
q10_param_alpha = None
```

**Q11.** What is the median predicted 12-month CLV across all customers? Round to 2 decimal places.
```python
q11_median_clv = None
```

**Q12.** What is the 99th percentile predicted CLV? Round to 2 decimal places.
```python
q12_p99_clv = None
```

**Q13.** What is the Pearson correlation between average order value and purchase frequency? Round to 2 decimal places. (This tests the Gamma-Gamma independence assumption.)
```python
q13_spend_freq_correlation = None
```

---

#### Part C: Interpretation Questions

**Q14.** The mean predicted CLV is $85 and the median is $32. What does this large gap between mean and median tell you about the distribution of CLV in this customer base?
- (a) Most customers have CLV around $85, with a few very high-value outliers pulling the mean up
- (b) The model has a bug — mean and median should always be equal
- (c) The CLV distribution is right-skewed: most customers have modest CLV, but a small number of high-CLV customers pull the mean up significantly
- (d) The median is a better estimate of future revenue than the mean
```python
q14 = None  # "a", "b", "c", or "d"
```

**Q15.** Customer K made 15 purchases with their last purchase 3 months ago. Their predicted CLV is $310. Customer L made 2 purchases with their last purchase 1 month ago. Their predicted CLV is $45. A marketing manager proposes spending $50 on retention for Customer K and $5 for Customer L, proportional to CLV. Is this reasonable?
- (a) Yes — spending proportional to CLV is always optimal
- (b) Not necessarily — you should spend up to the point where the cost equals P(alive) × CLV, which may differ from a proportional allocation
- (c) No — you should spend nothing, since both will churn eventually
- (d) Yes — Customer K is definitely more valuable than Customer L
```python
q15 = None  # "a", "b", "c", or "d"
```

**Q16.** The Gamma-Gamma correlation is −0.47. This is a strong negative correlation between spend per order and purchase frequency. Should you use the Gamma-Gamma model as-is?
- (a) Yes — a correlation of −0.47 is always acceptable for Gamma-Gamma
- (b) No — the independence assumption is violated; the Gamma-Gamma model will overestimate CLV for low-frequency/high-spend customers and underestimate for high-frequency/low-spend customers
- (c) Yes — the Gamma-Gamma model is robust to any correlation
- (d) No — you should switch to a completely different CLV method
```python
q16 = None  # "a", "b", "c", or "d"
```

---

### Common Misconceptions

**1. "BG/NBD requires a subscription or contractual setting."**
The BG/NBD model is specifically designed for non-contractual settings where customers can silently churn (e-commerce, retail, app usage). Contractual settings (where churn is formally observed) use simpler models.

**2. "A customer with P(alive) = 0.10 will never buy again."**
P(alive) = 0.10 means there is a 10% chance they are still active — they may still purchase. It means you should not invest heavily in retaining them, but they are not zeroed out.

**3. "The BG/NBD model only works for frequent buyers."**
The model handles customers with 0 purchases in the observation period (by giving them a very low P(alive) and low expected future transactions). It also works for customers with 1 or 2 purchases.

**4. "Predicted CLV decreases as we observe more purchases."**
More purchases generally increase predicted CLV, because they provide stronger evidence the customer is alive and indicate a higher purchase rate. The recency of the most recent purchase matters enormously.

**5. "We should maximize predicted CLV for every customer."**
CLV is an expected value — the cost of interventions must be weighed against the expected CLV improvement. Not every high-CLV customer needs retention spending (some will remain loyal without it).

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Prepares RFM summary (x, t_x, T, m) per customer | Lecture 1 workshop concepts + Section 1.3 |
| Fits BG/NBD via MLE, reports r, α, a, b | Section 1.4A — likelihood structure |
| Computes P(alive) per customer | Section 1.4B — alive probability intuition |
| Computes E[transactions] per customer | Section 1.4C — expected future transactions |
| Fits Gamma-Gamma, computes E[spend] | Section 1.3 + 1.4D — Gamma-Gamma model |
| Multiplies E[transactions] × E[spend] × discount | Section 1.4D — CLV formula |
| Plots spend vs. frequency (assumption check) | Section 2.2 — Gamma-Gamma independence |

**What to verify:**
1. BG/NBD parameters in plausible ranges: $r \in [0.1, 3]$, $\alpha \in [1, 30]$, $a \in [0.1, 2]$, $b \in [1, 10]$
2. Recent buyers have P(alive) > 0.80; long-silent customers have P(alive) < 0.20
3. Mean predicted CLV is a plausible multiple of the average order value
4. |correlation(spend, frequency)| < 0.2 for Gamma-Gamma assumption to hold
