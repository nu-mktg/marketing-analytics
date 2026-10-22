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
- Define CAC and the CLV:CAC ratio, and use CLV by acquisition channel to set channel-specific CAC targets (Section 1.4D)

**Prerequisites:** The Beta distribution from Lecture 1, reused here for each customer's dropout
probability. The Gamma distribution is used only through its mean, which is given wherever it
appears. ⚠️ **Letters carry over with new meanings.** **α** was a Beta shape in Lecture 1 and the
Hill exponent in Lecture 3. Here it is the *scale* of the Gamma distribution of purchase rates,
unrelated to either. **p** means two different things inside this lecture, flagged where the
second one appears (Section 2.1's parameter table).

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Multiplying a Rate by a Value

A rate — how many times something happens — multiplied by a value — how much each one is worth — gives an expected total. Both inputs are averages, so the product is an average too: it is what you expect *on average*, not what any one customer will do.

**Example:** CLV is the expected number of future purchases × expected spend per purchase. If a customer has E[purchases] = 3.2 and E[spend] = $55:
> CLV = 3.2 × $55 = **$176**

---

#### Tool 2: Why Silence Is Evidence of Dropout

A customer who bought 12 times in 2 years and then went silent for 19 months is probably not still an active customer — they have likely dropped out. The model must distinguish between:
- A customer who buys infrequently (a low purchase rate, written λ) — could have a long gap naturally
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

This lecture's two models split the "still retained" question (P(alive), from BG/NBD) from the "how much will they spend" question (a second model, Gamma-Gamma). This is its core insight: historical cash flow and future value are different things.

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

⚠️ **x and "purchase count" are two different numbers, and the gap is exactly 1.** Dashboards,
finance decks and this lecture's own business prose all say "purchases" meaning the **total**; the
model always means the **repeats**. A customer with 12 *repeat* purchases made 13 purchases in
total, and their historical revenue is 13 × AOV, not 12 × AOV. Watch for it in both directions:
- **Reading tables:** a column headed $x$ or `frequency` is repeats; a column headed "orders" or
  "purchases" is the total.
- **Writing code:** `lifetimes`' `summary_data_from_transaction_data()` has
  `include_first_transaction=False` by **default**, which is the definition above and the one this
  course wants. Flipping it to `True` silently changes what the model is fitted on and moves the
  fitted (r, α, a, b) substantially. Leave it alone.

The reason the model is built this way: BG/NBD's dropout process is a coin flip **after each
purchase**, so the acquiring purchase cannot carry any dropout information — a customer who has
never repeated has had no opportunity to drop out at all. (That is also why such a customer's
P(alive) comes out at exactly 1.0; see the Common Misconceptions section.)

**Example:** Customer B bought 12 times over 24 months. Last purchase at month 5 (19 months before the observation end at month 24):
> x = 11, t_x = 5, T = 24

The model fits population-level parameters (r, α for purchase rate distribution; a, b for dropout probability distribution) by maximizing the likelihood across all customers. You do not need to perform this estimation by hand — the agent does it.

**Reading P(alive)** (at the fitted parameters of Section 2.1's worked example: r = 0.85, α = 8.20, a = 0.55, b = 3.80):
- Customer A: x=11, t_x=23 (recent), T=24 → P(alive) ≈ 0.95
- Customer B: x=11, t_x=5 (19 months ago), T=24 → P(alive) ≈ 0.0006

Both bought 12 times (so x = 11 repeat purchases each, per the definition above). The difference in P(alive) comes entirely from recency. Customer B bought about every two weeks for five months and then stayed silent for 19. Under these parameters that is close to conclusive evidence of dropout: roughly 1 chance in 1,500 that B is still active.

That answers the Overview's "by how much". The model expects A to make 3.86 purchases in the next 12 months and B 0.003. At the worked example's \$38 average order (Section 2.1), A is worth about **\$147** over the next year and B about **\$0.10**. They have the same purchase history, and recency alone makes A worth over a thousand times more.

For x ≥ 1 the model gives P(alive) in closed form:

$$P(\text{alive} \mid x, t_x, T) = \frac{1}{1 + \dfrac{a}{b + x - 1}\left(\dfrac{\alpha + T}{\alpha + t_x}\right)^{r + x}}$$

Recency enters through $(\alpha + T)/(\alpha + t_x)$. It equals 1 when the last purchase is today ($t_x = T$) and grows with the silence. The exponent $r + x$ is why frequency cuts both ways: the more often a customer bought, the harder the same silence counts against them. For x = 0 the formula does not apply, and P(alive) is exactly 1 (Common Misconception 3).

![P(alive) on the vertical axis against the month of the customer's last purchase on the horizontal axis, from 0 to 24, for customers observed for 24 months, at the worked example's fitted parameters. A solid curve for a customer with 11 repeat purchases stays flat at zero until about month 10, then rises steeply in an S-shape through 0.5 near month 16 to about 0.96 at month 24. Customer B sits on it at month 5 with P(alive) about 0.0006, and Customer A at month 23 with about 0.95. A dashed curve for a customer with only 2 repeat purchases rises gently from about 0.16 to about 0.90, and Priya sits on it at month 20 with about 0.86. The two curves cross near month 20: below that, the same silence costs the frequent buyer far more P(alive) than the infrequent one.](figures/lecture_08_palive_recency.png)

---

#### Part B: Simple CLV Calculation

Once the agent provides E[purchases in next 12 months] and E[order value], CLV is:

> CLV₁₂ = E[transactions] × E[order value]

Here E[transactions] is the model's prediction *averaged over whether the customer is still active*, so it already
includes P(alive). Section 2.1's Part C shows why that matters: multiplying this CLV by P(alive) a second time is a
common error.

(For more precision, apply a monthly discount rate: CLV = Σ (E[purchases in month t] × E[order value]) / (1 + discount_rate)^t)

**Example:** E[transactions] = 2.4, E[order value] = $55 → CLV = 2.4 × $55 = **$132**

**The mean is not the typical customer.** CLV₁₂ above is an *average*, and a CLV distribution is
almost never symmetric — so "the average customer" and "the typical customer" are two different
people. Quoting one CLV figure means choosing which of the two you mean, and the second one is the
**median**.

> **Convention (this course, `numpy`, `pandas`):** the **median** of a set of values is the middle
> one after sorting — half the customers sit at or below it, half at or above. With an **odd**
> count it is the middle value itself; with an **even** count it is the average of the two middle
> values. That is what `Series.median()` returns, and it is what "median CLV" means.

**Example.** Five customers with 12-month CLVs of \$25, \$40, \$60, \$85 and \$540:
- sorted, the middle of the five is the third → **median = \$60**
- the mean is (25 + 40 + 60 + 85 + 540)/5 = 750/5 = **\$150**

The mean is more than twice the median, and *no customer in the set is worth anything like the
mean*. Now make the top customer worth \$5,400 instead of \$540: the mean moves to \$1,122 and
**the median does not move at all**. The median ignores how far the upper tail reaches; the mean is
dragged along by it. That robustness is the reason to report it.

**Reading the gap between them — this is the diagnostic.**

| What you see | What the distribution is doing | What it means for CLV |
|---|---|---|
| mean > median | **right-skewed** — a long upper tail, i.e. a small number of very high-value customers | the normal case for CLV: the mean is pulled up by the top tail, so it *overstates* the typical customer |
| mean ≈ median | roughly symmetric | the mean is a fair summary of the typical customer |
| mean < median | left-skewed — a long lower tail, i.e. a cluster of very low-value customers | rare for CLV; usually a mostly-healthy base with a damaged tail |

Which way the gap points is a fact about the customer base, not about the model, and it is the
first thing to establish before quoting any single CLV figure. When the mean exceeds the median —
for CLV, the normal state of affairs — a budget set against the mean overpays for the customers who
are actually typical. Report both: quote the **median** when someone asks what "a customer" is
worth, and the **mean** when you need a total, because mean × customer count is the portfolio
value and median × count is not.

---

#### Part C: The Gamma-Gamma Independence Assumption

The Gamma-Gamma model for spend assumes that a customer's average order value is independent of their purchase frequency. If high-frequency buyers tend to make smaller purchases (as often happens in grocery retail), this assumption is violated.

**How to check:** Compute the Pearson correlation between each customer's purchase frequency and their average order value, and read it against one scale — the same one the pre-flight checklist at the end of this lecture uses:

| \|correlation\| | Verdict |
|---|---|
| < 0.2 | assumption holds well enough to use Gamma-Gamma as-is |
| 0.2 – 0.3 | grey zone — usable, but say so and check whether the CLV ranking changes if you drop the highest-frequency decile |
| ≥ 0.3 | material violation — CLV for high-frequency customers is systematically biased |

Two numbers used to appear in this lecture for this one decision (a −0.30 "material violation" line here and a 0.2 line in the checklist), which left the 0.2–0.3 range with two different answers depending on which passage you read. They are one scale, above.

**What to do if violated:** With a *negative* correlation (frequent buyers spend less per order, the usual case), CLV for high-frequency customers is systematically overestimated. With a positive one it is underestimated. Either way, add a note of caution to any recommendation based on CLV for high-frequency customer segments.

> ### 🔍 Deep Dive: The Full BG/NBD Likelihood
> The BG/NBD models each customer's purchase process as a Poisson process (events arriving randomly at rate λ) and their dropout process as a Bernoulli trial after each purchase (drop out with probability p). Both λ and p vary across customers: λ ~ Gamma(r, α) and p ~ Beta(a, b). The likelihood for a customer (x, t_x, T) involves computing the probability of observing exactly x purchases, with the last at t_x, given that the customer either dropped out sometime after t_x or is still alive at T. This likelihood has a closed-form expression involving the Beta function — which is why the model is tractable. The four parameters (r, α, a, b) are estimated by maximizing the sum of log-likelihoods across all customers.

---

#### Part D: CLV:CAC — Turning CLV Into an Acquisition Decision

A CLV number on its own does not tell you whether to spend. The Overview promised you would use CLV
by channel to set channel-specific acquisition budgets; this is the arithmetic that does it.

**Customer Acquisition Cost (CAC)** for a channel is the total spend on that channel over a period
divided by the number of customers it acquired in that period:

> CAC = (channel spend) / (customers acquired through that channel)

It is a *per-customer cost*, in the same dollars as CLV, which is what makes the two comparable.

**The CLV:CAC ratio** (also written **LTV:CAC** — same thing; "LTV" and "CLV" are interchangeable
names for lifetime value, and this course uses both) is:

$$\text{CLV:CAC} = \frac{\text{predicted CLV per acquired customer}}{\text{CAC}}$$

**CLV on top, CAC underneath** — so a *bigger* number is a *better* channel, and the ratio reads as
"dollars of lifetime value bought per dollar of acquisition spend." Inverting it is the common
error; a ratio below 1 means you are paying more to acquire a customer than they will ever be worth.

**Example:** a channel with CAC = \$40 and predicted 12-month CLV = \$120 has CLV:CAC = 120/40 =
**3.0**.

**The 3:1 benchmark, and what it is worth.** The widely quoted rule of thumb is that **CLV:CAC > 3**
is a sustainable channel and CLV:CAC < 1 is value-destroying. It is a *convention*, not a result:
the 3 is there to absorb gross margin (CLV is usually quoted on revenue, not contribution), the
delay between paying CAC today and collecting CLV over years, and the risk that the CLV prediction
is wrong. Businesses with high margins, fast payback or cheap capital defend lower ratios; the
honest version of the rule is "compare channels against each other and against your own payback
period, and treat 3 as the default only when you have nothing better."

**Two traps.**
1. **The horizon has to match.** A ratio built from a *12-month* CLV is a 12-month statement, and it
   is not comparable to one built from a lifetime CLV — the same channel scores far higher on the
   longer window. Always say which window the CLV came from, and note separately whether the payback
   period is short enough for the business to survive it.
2. **Aggregate CAC hides the decision.** One blended CAC across all channels averages a \$8 organic
   customer with a \$45 paid-search one and tells you to do nothing. Compute it per channel — that
   is the entire point of segmenting CLV by acquisition channel.

**The scalability caveat.** A spectacular ratio is not automatically a place to put more money.
Organic channels routinely show CLV:CAC above 20 and cannot simply be bought more of; a paid channel
at 3:1 that absorbs another \$500k of budget may create more total value than an organic one at 26:1
that is already saturated. Ratio ranks efficiency; it does not measure headroom.

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

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
*Common wrong answer:* $162 × (1/(1+0.01))^6 ≈ $152.6 — applying a discount rate that was not asked for. The question specifies "no discounting."

**Q3.** **No — violated.** The Gamma-Gamma model assumes spend ⊥ frequency. A correlation of −0.48 (high-frequency buyers spend less per order) is a material violation. The model will overestimate CLV for high-frequency, low-spend customers.
*Common wrong answer:* Satisfied because −0.48 is close to 0 (confusing "close to zero" with "close to negative one"). −0.48 is a moderately strong negative correlation — well past the −0.30 warning threshold.

**Q4.** Paid Social CLV:CAC = 85/35 = **2.43**. Organic CLV:CAC = 210/8 = **26.25**. Organic is dramatically more efficient. However, organic channels may not be scalable — you cannot simply buy more organic growth the way you buy more paid social.
*Common wrong answer:* Paid Social has a ratio > 1, so it is profitable and should continue at current scale. While profitable, the comparison to organic shows it generates far less value per dollar invested.

**Q5.** The $210 average CLV masks enormous variation across customer segments and acquisition channels. CLV for customers acquired through a high-intent organic search keyword may be $400; CLV for customers acquired through a broad awareness campaign may be $80. Using the average to set CAC targets means overpaying for low-CLV segments and potentially underspending on high-CLV ones.
*Common wrong answer:* Nothing is wrong — using the average is a reasonable simplification. The simplification is only reasonable if CLV does not vary by acquisition channel, which it almost always does.

#### Slide Checkpoint Answers

Numbered to match the Checkpoint slide in this lecture's deck. Where the slide asks a question above again, or the same idea on different numbers, the entry points to that answer.

**Slide Q1.** *"$E[\text{trans in 12 months}] = 3.6$, $E[\text{order value}] = \$45$. Compute CLV₁₂ (no discounting)."* — Same question as Q2 above — see that answer.

**Slide Q2.** *"Both observed for 12 months. Customer A: 20 purchases, last 3 months ago. Customer B: 2 purchases, last 2 weeks ago. Who has higher P(alive)?"* — **Customer B.** At the worked example's parameters (Section 2.1), and taking each customer's first purchase at the start of the 12 months, A has x = 19, t_x = 9, T = 12 and **P(alive) ≈ 0.62**. B has x = 1, t_x = 11.5, T = 12 and **P(alive) ≈ 0.87** (closed form in Section 1.4A). P(alive) reads a silence against the customer's *own* cadence. A bought about every two weeks while active (19 repeats in 9 months), so three quiet months is about six missed purchases. B buys about twice a year, so two quiet weeks is nothing. Unlike Q1 above, which holds frequency constant, this question varies both inputs, and the larger count *lowers* A's P(alive): the exponent r + x makes the same silence count more heavily against a heavy buyer.
*The margin depends on the window:* observed for 24 months instead, A's 20 purchases are spread twice as thin, three months is a shorter gap relative to that cadence, and the two come out almost level (0.85 vs 0.87). "Who is more likely alive" is never answered by frequency or recency alone.

**Slide Q3.** *"P(alive) = 0.12. If still active, this customer is worth \$340 over the next 12 months. What is their **expected** 12-month value?"* — P(alive) × (value if active) = 0.12 × \$340 = **\$40.80**. You only collect the \$340 if they are still active, so the expected figure weights it by that probability. Part C of the worked example uses the same rule to judge retention spend. Spending \$50 to retain this customer would be a loss even though they are worth \$340 if active.
⚠️ The question says *if still active* on purpose. A CLV₁₂ that came out of the model (E[transactions] × E[order value], Section 1.4B) is already the expected value: it already includes P(alive). Had the \$340 been that number, the answer would be \$340, and multiplying by 0.12 would count dropout twice.

**Slide Q4.** *"Frequency–spend correlation = −0.52. How does this affect CLV estimates?"* — Same reasoning as Q3 above, which uses a correlation of −0.48; the slide's is −0.52.

**Slide Q5.** *""Our average CLV is \$280, so we can spend \$280 to acquire anyone." What's wrong?"* — Same reasoning as Q5 above, which uses an average CLV of \$210; the slide's is \$280.

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
| $p$ | 6.25 | Gamma-Gamma: shape of the Gamma for one customer's spend from order to order |
| $q$ | 3.74 | Gamma-Gamma: shape of the Gamma for how customers' typical spend varies across the base |
| $v$ | 15.45 | Gamma-Gamma: scale of that across-customer Gamma; population mean spend $= p\,v/(q-1) = 6.25 \times 15.45/2.74 \approx \$35$ |

**Why "Gamma-Gamma":** it stacks two Gamma distributions. Each order a customer places is a draw from a Gamma
distribution centred on that customer's own typical spend (shape $p$). Customers' typical spends in turn vary across
the base according to a second Gamma (shape $q$, scale $v$). Two Gammas, one inside the other.

⚠️ **This $p$ is not the dropout probability $p$** from the Deep Dive in Section 1.4C, which is a probability between
0 and 1 and is drawn from Beta($a$, $b$). The two models were named separately and both use the letter. The table
keeps the Gamma-Gamma letters $p, q, v$ because they are exactly what `lifetimes`' `GammaGammaFitter.params_`
prints, and what the homework notebook calls them.

Three customers are to be evaluated:

| Customer | Repeat purchases ($x$) | Observation period ($T$, months) | Last purchase ($t_x$, months from start) |
|---|---|---|---|
| Valentina | 12 | 24 | 23.5 |
| Marcus | 12 | 24 | 6.0 |
| Priya | 2 | 24 | 20.0 |

**Part A:** Before running any model, rank these three customers by expected 12-month CLV. Explain your reasoning in one sentence for each.

**Part B:** The model produces these estimates:

| Customer | P(alive) | E[transactions, 12m] | E[transactions, 12m \| still active] | E[spend/order] | CLV if still active (12m) | Expected CLV (12m) |
|---|---|---|---|---|---|---|
| Valentina | 0.96 | 4.24 | 4.43 | $38 | $168 | $161 |
| Marcus | 0.0007 | 0.0032 | 4.43 | $38 | $168 | $0.12 |
| Priya | 0.86 | 0.86 | 1.00 | $38 | $38 | $33 |

The last column is the model's CLV₁₂ (Section 1.4B), E[transactions] × E[spend/order]. The two transaction columns are
linked by E[transactions] = P(alive) × E[transactions | still active], so Expected CLV = P(alive) × CLV if still
active (0.96 × \$168 ≈ \$161).

**Why every E[spend/order] is \$38.** Gamma-Gamma's estimate for one customer blends their own average order
with the population's \$35, leaning on their own average more the more orders they have. With these parameters
the weight on their own average is $p\,x/(p\,x + q - 1)$: 0.96 for Valentina and Marcus (x = 12) and 0.82 for Priya
(x = 2). A customer whose own average is \$38 comes out at about \$38 either way. The example gives all three
the same own average so that the comparison runs on P(alive) alone. In a real fit, this column differs from
customer to customer.

Does the ranking match your intuition from Part A? What explains each customer's position?

**Part C:** A marketing manager proposes spending $15 in retention budget per customer to prevent churn. For which customers is this spending justified? What is the minimum P(alive) that makes retention spending worthwhile, assuming that successful retention captures the full predicted CLV?

---

**Full Solution**

**Part A: Intuitive Ranking (before running model)**

**Expected ranking: Valentina > Priya > Marcus**

(Read the table's 12s and 2 as $x$ — **repeat** purchases. Valentina and Marcus each made 13
purchases in total, Priya 3.)

- **Valentina:** x = 12 repeats in 24 months (high frequency), last purchase 0.5 months ago (very recent). Almost certainly alive. High predicted CLV.
- **Priya:** Only x = 2 repeats, but recent (4 months ago). Low frequency suggests either new or slow buyer. Probably alive, moderate CLV.
- **Marcus:** x = 12 repeats — but all crammed into the first 6 months, then 18 months of complete silence. Despite high historical frequency, very likely churned. Low predicted CLV.

**Part B: Model Output Analysis**

The ranking matches the intuition: Valentina (\$161) > Priya (\$33) > Marcus (\$0.12).

**Valentina** (P(alive)=0.96): Recent purchase confirms she is almost certainly active. High frequency means high expected future transactions (4.24 over 12 months). Her own history, 12 repeats in 24 months, is one every two months, or 6 a year. The model predicts fewer, 4.43 even if she is still active, because it pulls each customer's rate toward the population's much slower one ($r/\alpha$ = 0.10 a month) and allows for her dropping out during the year.

**Marcus** (P(alive)=0.0007): Despite identical purchase counts to Valentina, his 18-month silence is overwhelming evidence of dropout. He bought twice a month for six months and then not at all, and the model puts the chance he is still active at about 1 in 1,400. **Notice that his "if still active" columns equal Valentina's exactly** (4.43 transactions, \$168). Given that a customer is alive, the model's forecast depends only on x and T, which the two share. The entire gap between them is P(alive). Historical revenue = 13 purchases (x = 12 repeats **plus** the acquiring purchase) × \$38 = \$494; expected future CLV = \$0.12. The past overstates his future value about 4,000 to 1. Note which count each side uses: the model runs on the 12, the revenue you already banked runs on the 13.

**Priya** (P(alive)=0.86): Low frequency (x = 2 repeats, so 3 purchases in 24 months) but recent. The model estimates an 86% chance she is still active, below Valentina's 0.96 despite the similar recency, because two repeats are thin evidence either way. With a low base rate, her expected 0.86 purchases in 12 months (1.00 if still active) is reasonable. Her low CLV reflects low frequency, not imminent churn.

**Part C: Retention Budget Justification**

A \$15 retention spend is justified if the expected return exceeds the cost. You only recover a customer's value if they are still active, so the expected return is P(alive) × (CLV if still active). That product is the Expected CLV column.

For each customer, compute expected return from retention spending:
- **Valentina:** Expected return = 0.96 × \$168 = \$161. Cost = \$15. Net = +\$146 ✓
- **Marcus:** Expected return = 0.0007 × \$168 = \$0.12. Cost = \$15. Net = −\$14.88 ✗ — **do not spend**
- **Priya:** Expected return = 0.86 × \$38 = \$33. Cost = \$15. Net = +\$18 ✓

⚠️ **Multiply P(alive) by the *if-still-active* CLV, never by the model's CLV.** `lifetimes`'
`conditional_expected_number_of_purchases_up_to_time` already averages over whether the customer is alive, so the
CLV built on it is already an expected value. Multiplying it by P(alive) again counts dropout twice. For Valentina
that shaves a few dollars (0.96 × \$161 = \$155). For a customer near the threshold, it can flip the decision. If all
you have is the model's CLV, the if-active value is CLV ÷ P(alive).

**Minimum P(alive) to justify $15 retention:**

If we assume the retention budget, if effective, captures the full CLV:
$\text{P(alive)} \times \text{CLV if still active} > 15$
$\text{P(alive)} > 15 / \text{CLV if still active}$

For Valentina: threshold = 15/168 = 0.089 (easily exceeded, P(alive) = 0.96)
For Priya: threshold = 15/38 = 0.395 (exceeded, P(alive) = 0.86)
For Marcus: threshold = 15/168 = 0.089, the same as Valentina's, because his if-active value is the same (not exceeded: P(alive) = 0.0007)

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**P(alive) by customer:** Examine the distribution of P(alive) across your customer base. A healthy base has many customers with P(alive) > 0.7. If the majority are below 0.3, your customer base may be churning faster than expected.

**CLV decile analysis:** Sort customers by predicted CLV and examine the top decile (top 10%) vs. the bottom decile. The ratio of top-to-bottom CLV is a measure of customer heterogeneity. Ratios above 10× are common in e-commerce.

**Mean vs median CLV:** Report both, and read the gap the way Section 1.4B's diagnostic table does. A base with a
long upper tail has mean above median, so quoting the mean alone systematically overstates what a
typical customer is worth — and the ratio between the two is itself a compact measure of how
concentrated value is in the top customers.

**"Hidden gems":** Customers with low historical CLV but high predicted CLV. These are often new or recently reactivated customers — their purchase history is short but recent. The BG/NBD identifies them as valuable future customers that historical analysis would underrank.

**Model fit check:** The lifetimes library provides a `plot_calibration_purchases_vs_holdout_purchases` function that compares predicted purchase counts to actual purchases in a holdout period. If the model systematically underpredicts for high-frequency customers, the Gamma assumption for $\lambda$ may be misspecified.

**Before trusting the agent output:**
1. Do BG/NBD parameters fall in plausible ranges? ($r$ typically 0.1–3, $\alpha$ typically 1–30 — the same band as the pre-flight checklist at the end of this lecture)
2. Are P(alive) estimates intuitive? Long-silent buyers should have P(alive) < 0.2. Recent buyers should be *high*, but how high depends on their purchase rate — a low-frequency customer who bought recently can sit noticeably lower (Priya, above, at 0.86 against Valentina's 0.96) and be perfectly well estimated, because the model has little evidence either way
3. Does mean predicted CLV across all customers seem reasonable relative to average historical order value?
4. Is the Gamma-Gamma independence assumption approximately satisfied? (Plot order value vs. frequency)

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: 23:59 ET on the due date in the course schedule | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_08_clv_bgnbd.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_08_clv_bgnbd.ipynb` |
| Dataset | `homework_datasets/transactions.csv` |
| Graded questions | **15** — Part A: 8 · Part B: 4 · Part C: 3 |

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "BG/NBD requires a subscription or contractual setting."**
The BG/NBD model is specifically designed for non-contractual settings where customers can silently churn (e-commerce, retail, app usage). Contractual settings (where churn is formally observed) use simpler models.

**2. "A customer with P(alive) = 0.10 will never buy again."**
P(alive) = 0.10 means there is a 10% chance they are still active — they may still purchase. It means you should not invest heavily in retaining them, but they are not zeroed out.

**3. "The BG/NBD model only works for frequent buyers."**
It also works for customers with 1 or 2 repeat purchases. ⚠️ **And it treats a customer with x = 0 in a way that surprises everyone the first time:** their P(alive) is **exactly 1.0**, not low. Dropout in BG/NBD can only happen *after* a repeat purchase, so a customer who never repeated cannot have dropped out — there was no opportunity. Their *expected future transactions* are still low, because the model infers a low purchase rate; low future value and certain-alive are not the same statement. Verify this yourself: `bgf.conditional_probability_alive(0, 0, T)` returns 1.0 for any T.

**4. "More purchases always mean a higher predicted CLV."**
Only when the last purchase is recent. More purchases raise the estimated purchase rate, and if the customer bought lately they also confirm the customer is alive. But a higher rate also makes a long silence more suspicious. At the worked example's parameters, with the last purchase at month 6 of 24, P(alive) falls from 0.60 at x = 1 to 0.19 at x = 4 and 0.0007 at x = 12 (Marcus). Expected purchases fall with it, from 0.40 to 0.003. Frequency and recency are read together, never one at a time.

**5. "We should maximize predicted CLV for every customer."**
CLV is an expected value — the cost of interventions must be weighed against the expected CLV improvement. Not every high-CLV customer needs retention spending (some will remain loyal without it).

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Prepares RFM summary (x, t_x, T, m) per customer | Section 1.4A — the BG/NBD inputs (x, t_x, T) |
| Fits BG/NBD via MLE, reports r, α, a, b | Section 1.4C Deep Dive — likelihood structure |
| Computes P(alive) per customer | Section 1.4A — reading P(alive), and its closed form |
| Computes E[transactions] per customer | Section 1.4B — E[transactions] as the CLV input |
| Fits Gamma-Gamma, computes E[spend] | Section 1.3 + 1.4C — Gamma-Gamma model |
| Multiplies E[transactions] × E[spend] × discount | Section 1.4B — CLV formula and discount factor |
| Plots spend vs. frequency (assumption check) | Section 2.2 — Gamma-Gamma independence |
| Summarises CLV by acquisition channel against channel CAC | Section 1.4D — CAC and the CLV:CAC ratio |

**What to verify:**
1. BG/NBD parameters in plausible ranges: $r \in [0.1, 3]$, $\alpha \in [1, 30]$, $a \in [0.1, 2]$, $b \in [1, 10]$
2. Recent buyers have high P(alive), typically > 0.80, though a recent low-frequency buyer can sit lower (Section 2.2); long-silent customers have P(alive) < 0.20
3. Mean predicted CLV is a plausible multiple of the average order value
4. |correlation(spend, frequency)| < 0.2 for Gamma-Gamma assumption to hold
