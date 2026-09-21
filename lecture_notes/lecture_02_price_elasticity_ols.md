# Lecture 2: Price Elasticity of Demand
## How Price Changes Affect Revenue — and How to Measure It

---

### Overview

**Business question:** If you raise prices by 10%, will revenue go up or down? By how much?

**What you will be able to do after this lecture:**
- Compute the OLS slope and intercept by hand, using only addition and multiplication
- Fit a log-log regression that directly gives you the price elasticity coefficient
- Interpret elasticity: whether demand is elastic, inelastic, or unit-elastic
- Identify the revenue-maximizing price
- Recognize when OLS gives a biased elasticity estimate — and why

**Prerequisites:** Exponent rules from Lecture 1. Everything else is built here.

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: The Average (Mean)

The mean of a set of numbers is their sum divided by how many there are.

> x̄ = (x₁ + x₂ + ... + xₙ) / n

**Example:** prices = [5, 6, 7, 8, 9]. Mean = (5+6+7+8+9)/5 = **7.0**

---

#### Tool 2: Deviations from the Mean

A **deviation** is how far a value is from the mean: (xᵢ − x̄).

| Price (xᵢ) | x̄ | Deviation (xᵢ − x̄) |
|---|---|---|
| 5 | 7 | −2 |
| 6 | 7 | −1 |
| 7 | 7 | 0 |
| 8 | 7 | +1 |
| 9 | 7 | +2 |

**Key property:** The deviations always sum to zero. Σ(xᵢ − x̄) = 0.

---

#### Tool 3: The Natural Logarithm (ln)

The natural log converts multiplication into addition. For our purposes, one property matters:

> **If y = ln(x), then a small change in y equals the proportional change in x: Δln(x) ≈ Δx/x. A change of 0.01 in ln(x) is about a 1% change in x.**

More practically: in a log-log regression, the slope coefficient equals the *percentage* change in y for a 1% change in x — which is the definition of elasticity.

**Key fact:** ln(AB) = ln(A) + ln(B). Multiplying becomes adding in log space.

> 🔍 **Deep Dive:** Why ln specifically (and not log base 10)?
> The natural log has a unique property: d/dx[ln(x)] = 1/x. This means the slope of ln(x) at any point equals the proportional change rate. It is the only log function for which a *small* change in ln(x) equals the proportional change in x — a change of 0.01 in ln(x) is a ≈1% change in x. (A full 1-unit increase in ln(x) multiplies x by e ≈ 2.718, a ≈172% increase — not 100%.) This is why economists and statisticians use ln for elasticity models.

---

### Section 1.2 — Business Motivation

---

#### The Pricing Decision

You manage a grocery chain. A category manager wants to raise the price of a product. Before changing the price, you need to know: will revenue go up or down?

The answer depends on **price elasticity of demand** — how sensitive customers are to price changes.
Economists write the elasticity as **ε** (epsilon): the percentage change in quantity sold caused by a
1% change in price. It is normally negative — raise the price, sell fewer — so what matters for the
decision below is its size, written **|ε|**.

| Elasticity \|ε\| | Type | 10% price increase → |
|---|---|---|
| Greater than 1 | Elastic | Revenue decreases (quantity falls more than price rises) |
| Exactly 1 | Unit-elastic | Revenue unchanged |
| Less than 1 | Inelastic | Revenue increases (quantity falls less than price rises) |

A luxury handbag is inelastic. A commodity product like bottled water is elastic. Knowing your elasticity is the first step in any pricing decision.

---

#### Strategic Frame: What Practitioners Know

> *"When we talk about pricing, many people quickly gravitate to dollar figures. But price is a measure of value — like a liter is a measure of volume."*
> — Madhavan Ramanujam, Senior Partner, Simon-Kucher

Ramanujam has advised over 250 companies on pricing, including Uber, DoorDash, and LinkedIn. His core insight: most companies treat pricing as a cost-plus exercise. They build the product, then figure out what to charge. But the sequence should be reversed — understand what customers value and what they will pay, then build the product that captures that value.

The quantitative tool for understanding what customers will pay is exactly the elasticity model in this lecture. But the strategic point is that the regression output — the elasticity coefficient — is not the end of the analysis. It is the input to a conversation about value, positioning, and customer segmentation.

---

### Section 1.3 — Conceptual Framework

---

#### The Demand Curve

A demand curve describes the relationship between price and quantity sold. As price rises, demand typically falls. As price falls, demand typically rises.

**The challenge:** we cannot see the true demand curve. We only observe the prices we actually charged and the quantities actually sold. We use regression to estimate the demand curve from this data.

![The five observations of Section 1.4's worked example, with two curves fitted to the same five dots: the straight line q̂ = 329 − 27p, and a constant-elasticity curve with exponent −1.33. They agree across the observed $5–$9 range and separate outside it — the line reaches exactly zero units at $12.19, the curve never reaches zero](figures/lecture_02_demand_curve_inferred.png)

**Reading the figure.** The five dots are the entire dataset — that is all you ever see. Both
curves are consistent with them, and *nothing in the data* chooses between the two. Inside the
$5–$9 band they give nearly the same answer, so the choice barely matters; outside it they diverge,
which is why an elasticity estimated here should not be used to price at $12. (This is
Misconception 3, and Misconception 5, seen from the side.)

**The endogeneity problem (preview):** If you raise prices when demand is high and lower them when demand is slow, then observed prices and observed demand are correlated in a way that biases the regression estimate. This is one of the most important practical problems in pricing analytics — we will return to it in Section 1.4.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: OLS in Plain Algebra

We have n observations of (price, units sold), and we want the straight line that best fits them.
Before writing any formula, two pieces of notation have to mean something, because the rest of this
lecture — and all of Section 1.4C — depends on them.

> ⚠️ **The hat is not decoration. It marks the difference between what is true and what you computed.**
> Write the true relationship in the population as **y = β₀ + β₁x + u**. You never see any of those
> three things. What you compute from your n observations is an *estimate* of each, and estimates
> wear hats:
>
> | Symbol | What it is | Can you ever see it? |
> |---|---|---|
> | $\beta_1$ | the **true** slope in the population | **No.** Never. |
> | $\hat{\beta}_1$ ("beta-one-hat") | the **estimate** of $\beta_1$ that OLS computes from your data | Yes — it is a number you calculate |
> | u | the **error**: how far a real observation sits from the **true** line | **No.** Never. |
> | ŷ | the **fitted value**: what the estimated line predicts | Yes |
> | y − ŷ | the **residual**: how far an observation sits from the **estimated** line | Yes |
>
> ⚠️ **So "error" and "residual" are not two words for one thing**, even though they are often used
> loosely as if they were. The error is the invisible gap from the true line; the residual is the
> visible gap from the fitted line. OLS minimizes the sum of squared **residuals**, because those are
> the only ones it can see. Keep the pair apart: **Section 1.4C is a statement about the *error*, not
> the residual** — which is exactly why endogeneity cannot be spotted by staring at your residuals.

"Best fit" now has a precise meaning: choose $\hat{\beta}_0$ and $\hat{\beta}_1$ to make the sum of squared *residuals* as
small as possible. Doing that produces one formula for the slope:

$$\hat{\beta}_1 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^{n}(x_i - \bar{x})^2}$$

**Where this formula comes from.** It is not a definition — it is the *solution* of that minimization,
and deriving it takes calculus (set the derivative of the squared-residual sum to zero and solve).
The Deep Dive below does that in four lines, and it is optional. **Using** the formula, which is what
the rest of Part A does, needs nothing but addition, multiplication and division.

**Plain English:** The slope equals (how much x and y move together) divided by (how much x varies on its own). If high prices consistently coincide with low sales, the numerator will be negative, and the slope will be negative — as expected for a demand curve.

**Worked example** (5 data points):

| Price (x) | Units (y) | x−x̄ | y−ȳ | (x−x̄)(y−ȳ) | (x−x̄)² |
|---|---|---|---|---|---|
| 5 | 200 | −2 | +60 | −120 | 4 |
| 6 | 160 | −1 | +20 | −20 | 1 |
| 7 | 140 | 0 | 0 | 0 | 0 |
| 8 | 110 | +1 | −30 | −30 | 1 |
| 9 | 90 | +2 | −50 | −100 | 4 |
| **Mean** | **x̄=7, ȳ=140** | | | **Σ = −270** | **Σ = 10** |

β₁ = −270 / 10 = **−27 units per dollar**
β₀ = 140 − (−27 × 7) = 140 + 189 = **329**

Predicted line: Units = 329 − 27 × Price

> ### 🔍 Deep Dive: Where Does the Formula Come From?
> *Skip if you are comfortable using the formula. Read if you want to know why it is the right formula.*
>
> OLS minimizes the sum of squared residuals over the **estimates** — note the hats, since these are
> the quantities we get to choose: $\text{SSR} = \sum(y_i - \hat{\beta}_0 - \hat{\beta}_1 x_i)^2$. To find the minimum, we take the
> derivative with respect to $\hat{\beta}_1$ and set it equal to zero:
>
> $\partial \text{SSR} / \partial \hat{\beta}_1 = -2\sum(y_i - \hat{\beta}_0 - \hat{\beta}_1 x_i)x_i = 0$
>
> Substituting $\hat{\beta}_0 = \bar{y} - \hat{\beta}_1\bar{x}$ (from the other first-order condition) and rearranging gives:
>
> Σ(xᵢ − x̄)(yᵢ − ȳ) = β₁ × Σ(xᵢ − x̄)²
>
> Dividing both sides by Σ(xᵢ − x̄)² yields the formula above. The key insight: OLS is the unique linear estimator that is unbiased under the assumption that errors are uncorrelated with the predictor.

---

#### Part B: The Log-Log Model for Elasticity

The OLS above gives "units fall by 27 for every $1 price increase" — a constant slope that does not make economic sense for large price changes. The log-log model fixes this by working in percentage terms:

$$\ln(\text{units}) = \beta_0 + \beta_1 \ln(\text{price}) + u$$

> ⚠️ **Why the error is written `u` here, when most books write it `ε`.** Because **ε is already
> taken on this page.** Since Section 1.2, |ε| has meant the **elasticity** — the number you compare
> to 1 to decide whether a price rise helps or hurts revenue. The error term is a completely
> different object: the unobservable gap between a real observation and the true line. Writing both
> as ε is the single easiest way to misread this lecture, so in these notes:
> **ε is always the elasticity. `u` is always the error term.**
> Outside this course you will meet ε in the error's slot constantly — when you do, check which of
> the two an author means before you trust the sentence around it.

**Why log-log?** In this model, β₁ has a specific interpretation:

> A 1% increase in price is associated with a β₁% change in units sold.

This is the definition of price elasticity. β₁ is the elasticity directly — no additional calculation needed.

![The same five observations on two sets of axes. In levels the fitted relationship is a curve; in logs it is a straight line, and its slope — −1.33 — is the elasticity itself. A run of +0.20 in ln(price) forces a fall of 0.267 in ln(units)](figures/lecture_02_loglog_transform.png)

**Reading the figure.** Nothing is re-measured between the two panels: the same five weeks are
re-plotted on logged axes, and the curve straightens out. The slope of the right-hand line is the
number you would report as the elasticity — so "take logs" and "estimate an elasticity" are the
same instruction. The fit shown is the log-log fit of Part A's own five rows.

> ⚠️ **−1.33 here and −1.35 in the next figure are two different objects, not a rounding error.**
> **−1.33** is the slope of *this* line: the log-log fit, which is **one elasticity for every
> price** — that is what "constant elasticity" means, and it is why the line is straight.
> **−1.35** is the elasticity of the **levels** fit $\hat{\beta}_1$ = −27 **evaluated at one price**, the mean
> \$7: ε = $\hat{\beta}_1$ × P/Q = −27 × 7/140. That one **changes with price** — the next figure shows it
> running from 0.70 at \$5 to 2.83 at \$9. On this data the two happen to land 0.02 apart, which
> is a coincidence of these five rows and not a general fact. **If you ever have to choose: the
> log-log slope is a single summary of the whole price range; the levels elasticity is a local
> reading at one point on it.**

> ⚠️ **ε and β₁ are the same number.** Section 1.2 called the elasticity ε because that is the
> economics convention; Part B calls it β₁ because that is the name the regression output prints.
> In a log-log model they are one quantity with two labels: **ε = β₁.** So when Section 1.5 says
> "|ε| = 1.84" and the worked example in Part 2 reports "β₁ ≈ −0.96", those are the same kind of
> object measured on two different datasets — not two different statistics. Part 2 additionally
> writes it **β₁^log** at the moment it is computed, purely to stress that it is the slope from the
> *logged* regression and not the levels slope $\hat{\beta}_1$ = −27 from Part A. **Three notations, one
> elasticity** — and only the levels slope is a different quantity.

**The revenue-maximizing price:** Revenue = Price × Quantity. Revenue is maximized where elasticity |β₁| = 1. When |β₁| > 1 (elastic), revenue increases as you lower price. When |β₁| < 1 (inelastic), revenue increases as you raise price. **This rule describes a demand curve whose elasticity varies with price** — read the log-log estimate as a local approximation near the prices you observe (see Misconception 3).

> ### 🔍 Deep Dive: Why |ε| = 1 Maximizes Revenue
> *Skip this if the result makes intuitive sense. Read it if you want the algebraic proof.*
>
> Revenue R = P × Q. Taking the derivative with respect to P and setting it to zero:
>
> dR/dP = Q + P × (dQ/dP) = Q(1 + P/Q × dQ/dP) = Q(1 + ε) = 0
>
> This equals zero when ε = −1. Since Q > 0 always, the condition reduces to 1 + ε = 0, or ε = −1 (equivalently |ε| = 1).
>
> Note what this requires: ε must *change* with P for dR/dP to cross zero. In a strict constant-elasticity model it never does — if |ε| ≠ 1, revenue rises (or falls) with price without limit.

![Revenue and elasticity on one price axis, both from Part A's fitted line q̂ = 329 − 27p. Revenue peaks at $6.09, and |ε| crosses 1 at that same $6.09. At the average observed price of $7 the fit gives |ε| = 1.35 — elastic, so a price cut would raise revenue](figures/lecture_02_revenue_and_elasticity.png)

**Reading the figure.** The dotted vertical line is the whole argument: the peak in the top panel
and the |ε| = 1 crossing in the bottom panel are the *same price*. It is drawn on the straight-line
fit precisely because that fit's elasticity changes with price (from 0.70 at $5 to 2.83 at $9) —
the condition the Deep Dive just named. A strict log-log fit would give one elasticity at every
price and no peak to find. The **1.35** marked at \$7 is that same levels fit read at the mean
price — the local reading of the previous figure's −1.33, not a second estimate of it.

---

#### Part C: The Endogeneity Problem

If prices are set partly in response to demand conditions (managers lower prices during slow periods, raise them during busy ones), then price and the error term **u** are correlated — in symbols, Cov(price, u) ≠ 0. OLS needs that covariance to be zero. When it is not, the estimate is **biased** — typically toward zero, making demand appear less elastic than it truly is.

> ⚠️ **This is why Part A insisted that the error and the residual are different things.** The
> condition that fails here is about **u**, the gap from the *true* line — which you can never
> observe or plot. Your residuals will look perfectly well-behaved while this is happening. **There
> is no diagnostic plot for endogeneity**; you find it by knowing how the prices in your data were
> actually set. That is a question about the business, not about the regression output.

**What this means practically:** If your regression returns elasticity = −0.3 but you know managers are cutting prices when sales are slow, the true elasticity is probably more negative. You are underestimating how much customers care about price.

**The fix (preview):** An instrumental variable — something that affects price but not demand directly — can break the correlation. **Input cost changes** (a wholesale-price or commodity-cost shock) are the standard example in retail pricing research: they move what the retailer charges without themselves changing how much customers want the product.

> ⚠️ **A tempting instrument that is not one: in-store display promotions.** Displays are a *demand shifter*, not a price shifter — they lift units at an unchanged price, which is exactly what the exclusion restriction forbids. That is why `display_flag` appears on the **right-hand side as a control** in this week's homework regression, not as an instrument. A variable you have to control for cannot also be your instrument.

---

### Section 1.5 — Interpretation Guide

**Elasticity = −1.84: What does this mean?**
- Demand is elastic (|1.84| > 1)
- A 1% price increase → 1.84% decrease in units sold
- A 10% price increase → 18.4% decrease in units sold
- The current price is ABOVE the revenue-maximizing price — lowering price would increase revenue

**R² = 0.87: What does this mean?**

R² (*R-squared*) is the share of the variation in the outcome that the model's predictors account
for: 0 means they explain none of it, 1 means they explain all of it. So here:

- 87% of the variation in log(units) is explained by the regression predictors
- This is R-squared for explained variation, NOT causation
- High R² does not mean the elasticity estimate is unbiased

**Cross-price elasticity = +0.52: What does this mean?**
- A 1% increase in the competitor's price → 0.52% increase in your units
- Positive cross-price elasticity = the products are substitutes
- Customers switch to you when the competitor gets more expensive

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

1. You estimate β₁ = −1.84 in a log-log regression. A colleague says "demand is elastic." Are they right?

2. Using the table below, calculate β₁ manually. Prices: [8, 9, 10, 11, 12]. Units: [120, 100, 90, 75, 65].

3. Your elasticity estimate is −1.84. What is the approximate revenue-maximizing price relative to the current price? (Higher, lower, or current price is already optimal?)

4. A store manager lowers prices every Saturday because traffic is lower on weekends. If you run OLS on this store's data, will the elasticity estimate be too large (more negative) or too small (less negative) than the true value?

5. Cross-price elasticity = +0.62. Should you be concerned about a competitor's planned 15% price increase? Why?

---

### Checkpoint Answer Key

**Q1.** Yes. Elastic demand means |ε| > 1. Since |−1.84| = 1.84 > 1, demand is elastic. A price increase reduces revenue because the percentage quantity drop outweighs the percentage price rise.

*Common wrong answer:* "Elastic means demand responds a lot, so −1.84 must be inelastic since it's less than 1." The elasticity is compared to 1 in absolute value, not as a raw number. ε = −1.84 means |ε| = 1.84 > 1 → elastic.

> **Also asked on the slides:** *"A **different** store's log-log regression gives $\hat{\beta}_1$ = −1.42 — not the five rows above, which fit −1.33. Is demand elastic or inelastic? What happens to revenue if price rises 5%?"* — Elastic: |−1.42| = 1.42 > 1. In a log-log model β₁ **is** the elasticity, so a 5% price rise is associated with a 1.42 × 5% ≈ **7.1% fall in units**. Revenue = price × units, so revenue moves by roughly +5% − 7.1% ≈ **−2.1%** — revenue falls. Same rule as Q1: when demand is elastic, raising price reduces revenue.
>
> ⚠️ **Why the slide says a *different* store.** The deck's five-row table ([5,6,7,8,9] / [200,160,140,110,90]) has a **log-log fit of −1.33**, and the figure on that slide prints −1.33 on the curve. −1.42 is a fresh number for the student to apply the rule to, not a value derivable from the table — the wording keeps the two apart. Note also that −1.33 (the log-log slope, constant by construction) and **−1.35** (the *level-level* fit's point elasticity at the mean, −27 × 7/140) are different objects that happen to be close here; that is the constant-vs-varying elasticity point, not a rounding discrepancy.

**Q2.** x̄ = 10, ȳ = 90. Deviations: (−2, −1, 0, +1, +2) and (+30, +10, 0, −15, −25).
Numerator = (−2)(30) + (−1)(10) + (0)(0) + (1)(−15) + (2)(−25) = −60 − 10 + 0 − 15 − 50 = −135.
Denominator = 4 + 1 + 0 + 1 + 4 = 10. β₁ = −135/10 = **−13.5 units per dollar**.

*Common wrong answer:* Forgetting to compute deviations first, using raw values instead of (xᵢ − x̄) and (yᵢ − ȳ).

> **Also asked on the slides:** *"From the data above, compute $\hat{\beta}_0$ and verify it equals 329."* — The table on that slide is the Part A worked example in these notes: Prices [5, 6, 7, 8, 9], Units [200, 160, 140, 110, 90], x̄ = 7, ȳ = 140, Σ(x−x̄)(y−ȳ) = −270, Σ(x−x̄)² = 10. So β₁ = −270/10 = −27, and the intercept follows from β₀ = ȳ − β₁x̄ = 140 − (−27 × 7) = 140 + 189 = **329**, giving the fitted line Units = 329 − 27 × Price. Note that the checkpoint table in Q2 is a *different* dataset (β₁ = −13.5), so do not expect 329 from it.

> **Also asked on the slides:** *"Why does log-log give a constant elasticity but level-level gives a varying elasticity?"* — Because of what the coefficient measures. In the level-level model the slope is a constant number of **units per dollar** (here −27, or −13.5 in Q2), so the *percentage* response depends on where you are: at a base of 200 units a 27-unit drop is 13.5%, at a base of 90 units the same 27-unit drop is 30%. Elasticity therefore changes along the line. In the log-log model both sides are in logs, so β₁ is a **percentage response to a percentage change** — the same at every price by construction. That is why log-log is the specification used for elasticity, and why the level-level slope "does not make economic sense for large price changes."

**Q3.** Revenue is maximized where |ε| = 1. Current |ε| = 1.84 > 1 means we are on the elastic portion of the demand curve. Lowering price would increase revenue — current price is **too high** relative to the revenue-maximizing price.

*Common wrong answer:* "Elastic means customers are sensitive, so raising prices will hurt us — we should lower prices to the minimum." Elasticity tells you the direction for revenue maximization, not how much to move.

> **Also asked on the slides:** *"At what elasticity is revenue maximized? What does the firm do if current ε = −0.8?"* — Revenue is maximized where |ε| = 1. With ε = −0.8 we have |ε| = 0.8 < 1, so demand is **inelastic** and the firm should **raise** price: the percentage loss in units is smaller than the percentage gain in price, so revenue rises. This is the mirror image of Q3 above, where |ε| = 1.84 > 1 called for a price cut.

**Q4.** Too small (less negative) — biased toward zero. When the manager lowers prices during slow periods, observed low prices coincide with low demand (not because of the price cut, but because it's Saturday). The regression incorrectly attributes low demand to the low price, making demand appear less sensitive to price than it truly is.

*Common wrong answer:* "The estimate will be more negative because cutting prices on slow days shows prices matter." The direction of endogeneity bias is counterintuitive — it depends on the correlation between the endogenous regressor (price) and the omitted variable (day-of-week demand), not on the direction of the price cut.

> **Also asked on the slides:** *"A pricing analyst runs OLS on transaction data and gets $\hat{\beta}_1$ = +3 (positive!). What's the most likely explanation?"* — The same endogeneity as in Q4, only larger. Prices in transaction data are set *in response to* demand: they are raised when demand is strong and discounted when it is weak, so high price and high volume appear together. Section 1.4 notes the resulting bias is "typically toward zero" — but when the demand-driven component dominates, the bias carries the estimate **past** zero and the fitted coefficient comes out positive. A positive $\hat{\beta}_1$ is therefore a diagnostic of a broken identification assumption, not a discovery that customers buy more when things cost more. The fix is the one previewed in Section 1.4: an instrument that shifts price without reflecting demand (e.g. an input or wholesale cost shock — **not** a display promotion, which shifts demand directly and is therefore a control, not an instrument).

**Q5.** Yes — this is a good thing for you. Cross-price elasticity of +0.62 means a 15% competitor price increase → approximately 15 × 0.62 = 9.3% increase in your units. You could consider whether to maintain your price (capturing volume) or raise yours slightly (capturing some of the competitor's pricing power). Do not be alarmed — positive cross-price elasticity means the competitor's higher price benefits you.

*Common wrong answer:* "Positive cross-price elasticity means our products are complements, so if the competitor raises prices, demand for our product will also fall." Positive cross-price elasticity means substitutes (customers switch to us), negative means complements. Complements fall together in demand; substitutes rise when the other's price rises.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Steps 1–3 first, then reveal full solution)


---

**Problem Statement**

A beverage company has 5 weeks of data on the price and sales volume of a premium juice product:

| Week | Price (x) | Units Sold (y) |
|---|---|---|
| 1 | $2.00 | 120 |
| 2 | $2.50 | 100 |
| 3 | $3.00 | 85 |
| 4 | $3.50 | 75 |
| 5 | $4.00 | 60 |

**(a)** Compute $\bar{x}$ (mean price) and $\bar{y}$ (mean units sold).

**(b)** Compute $\sum(x_i - \bar{x})(y_i - \bar{y})$ and $\sum(x_i - \bar{x})^2$.

**(c)** Compute the OLS slope β_1 and intercept β_0.

**(d)** Interpret β_1 in business terms.

**(e)** Now transform the data. Compute log(x_i) and log(y_i) for each week.

**(f)** Repeat steps (a)–(c) using log(price) as x and log(units) as y.

**(g)** Interpret the log-log slope as a price elasticity.

**(h)** Is demand elastic or inelastic? What does this imply for revenue?

---

**Full Solution**

**(a) Means:**

$$\bar{x} = \frac{2.00 + 2.50 + 3.00 + 3.50 + 4.00}{5} = \frac{15.00}{5} = 3.00$$

$$\bar{y} = \frac{120 + 100 + 85 + 75 + 60}{5} = \frac{440}{5} = 88.0$$

**(b) Building the table of deviations:**

| Week | x_i | y_i | x_i − x̄ | y_i − ȳ | (x_i−x̄)(y_i−ȳ) | (x_i−x̄)² |
|---|---|---|---|---|---|---|
| 1 | 2.00 | 120 | −1.00 | +32 | −32.0 | 1.00 |
| 2 | 2.50 | 100 | −0.50 | +12 | −6.0 | 0.25 |
| 3 | 3.00 | 85 | 0 | −3 | 0.0 | 0.00 |
| 4 | 3.50 | 75 | +0.50 | −13 | −6.5 | 0.25 |
| 5 | 4.00 | 60 | +1.00 | −28 | −28.0 | 1.00 |
| **Sum** | | | | | **−72.5** | **2.50** |

$$\sum(x_i - \bar{x})(y_i - \bar{y}) = -72.5$$
$$\sum(x_i - \bar{x})^2 = 2.50$$

**(c) OLS coefficients:**

$$\beta_1 = \frac{-72.5}{2.50} = -29.0$$

$$\beta_0 = \bar{y} - \beta_1\bar{x} = 88.0 - (-29.0)(3.00) = 88.0 + 87.0 = 175.0$$

The fitted line is: $\hat{y} = 175.0 - 29.0x$

**(d) Interpretation of β_1:**

"A $1 increase in price is associated with 29 fewer units sold per week."

**(e) Log-transforming the data:**

Using log (natural logarithm):

| Week | x_i | y_i | log(x_i) | log(y_i) |
|---|---|---|---|---|
| 1 | 2.00 | 120 | 0.6931 | 4.7875 |
| 2 | 2.50 | 100 | 0.9163 | 4.6052 |
| 3 | 3.00 | 85 | 1.0986 | 4.4427 |
| 4 | 3.50 | 75 | 1.2528 | 4.3175 |
| 5 | 4.00 | 60 | 1.3863 | 4.0943 |

**(f) Log-log OLS:**

$$\overline{\log(x)} = \frac{0.6931 + 0.9163 + 1.0986 + 1.2528 + 1.3863}{5} = \frac{5.3471}{5} = 1.0694$$

$$\overline{\log(y)} = \frac{4.7875 + 4.6052 + 4.4427 + 4.3175 + 4.0943}{5} = \frac{22.2472}{5} = 4.4494$$

Building the deviation table:

| Week | log(x_i) | log(y_i) | log(x_i)−mean | log(y_i)−mean | product | square |
|---|---|---|---|---|---|---|
| 1 | 0.6931 | 4.7875 | −0.3763 | +0.3381 | −0.1272 | 0.1416 |
| 2 | 0.9163 | 4.6052 | −0.1531 | +0.1558 | −0.0239 | 0.0234 |
| 3 | 1.0986 | 4.4427 | +0.0292 | −0.0067 | −0.0002 | 0.0009 |
| 4 | 1.2528 | 4.3175 | +0.1834 | −0.1319 | −0.0242 | 0.0336 |
| 5 | 1.3863 | 4.0943 | +0.3169 | −0.3551 | −0.1125 | 0.1004 |
| **Sum** | | | | | **−0.2880** | **0.2999** |

$$\beta_1^{\log} = \frac{-0.2880}{0.2999} \approx -0.960$$

$$\beta_0^{\log} = 4.4494 - (-0.960)(1.0694) = 4.4494 + 1.0267 = 5.476$$

**(g) Elasticity interpretation:**

The price elasticity is β_1 ≈ **−0.96**.

"A 1% increase in price is associated with a 0.96% decrease in units sold."

**(h) Elastic or inelastic?**

|ε| = 0.96 < 1, so demand is **inelastic** — but barely. The elasticity is very close to −1, suggesting the product is near the revenue-maximizing price. Raising prices slightly would increase revenue (since |ε| < 1), but not by much.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

When the agent fits a regression model and prints the output table, here is what each row and column means.

**Coefficient column:** The estimated β values. For the log-log model, these are elasticities directly. For a linear model, they are unit-level effects.

**Standard error column:** A measure of uncertainty around the coefficient estimate. Smaller standard errors mean more precise estimates.

**t-statistic:** The coefficient divided by its standard error. Large absolute values (typically |t| > 2) suggest the coefficient is reliably different from zero.

**p-value:** The probability of observing a coefficient this far from zero by chance, assuming the true coefficient is zero. A p-value < 0.05 is conventional evidence that the predictor has a real relationship with the outcome. Note: this is a frequentist concept — it does not tell you the probability that the coefficient is positive.

**R-squared:** The proportion of variation in log(sales) explained by log(price). An R² of 0.95 means 95% of the variation in log(sales) is accounted for by price variation.

**What to check in the agent output:**
1. Is the coefficient on log(price) negative? If it is positive, either the model has a problem or the data contains unusual variation.
2. Is the magnitude between −0.3 and −4.0? Most consumer goods have elasticities in this range. Outside this range, investigate.
3. Does R² seem reasonable? Very high R² (> 0.99) with only price as a predictor is suspicious — price rarely explains nearly all sales variation.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_02_price_elasticity.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_02_price_elasticity.ipynb` |
| Dataset | `homework_datasets/pricing_data.csv` |
| Graded questions | **17** — Part A: 11 · Part B: 4 · Part C: 2 |

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "The intercept β_0 is the most important coefficient."**
In most business regression applications, the intercept has no direct interpretation (e.g., sales when price = $0). The slope coefficients carry the business insight. Do not overinterpret the intercept.

**2. "A higher R² always means a better model."**
R² can be artificially inflated by adding more variables. A model with 50 irrelevant variables will have a higher R² than a model with 1 relevant variable. What matters is whether the model generalizes to new data.

**3. "The log-log model's elasticity is constant everywhere."**
The log-log model assumes a constant elasticity across all price levels. In reality, elasticity may vary — consumers might be more sensitive to price changes at high prices than at low prices. The log-log model is a useful approximation, not a perfect description.

**4. "If the regression coefficient is negative, price causes sales to fall."**
Correlation and causation are different. A negative coefficient tells you that weeks with higher prices tend to have lower sales. It does not prove that raising price causes sales to fall — there could be reverse causation (prices are set low when demand is expected to be low) or omitted variables.

**5. "Price elasticity of −0.8 means demand is perfectly fine to price wherever we want."**
An elasticity of −0.8 (inelastic) means revenue increases when price increases. But it does not mean you should keep raising price indefinitely — beyond some point, demand will become more elastic, and at very high prices, elasticity will certainly exceed 1. Estimate the elasticity at the price points you are actually considering.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Creates log_price and log_units columns | Section 1.4B — the log-log specification |
| Fits OLS regression | Section 1.4A — minimizing sum of squared residuals |
| Prints coefficient on log_price | Section 1.4B — elasticity = β_1 in log-log model |
| Prints p-values | Section 2.2 — statistical significance check |
| Prints R-squared | Section 2.2 — proportion of variance explained |
| Plots predicted vs. actual | Section 1.4A — the residuals y − ŷ, plotted |

**What to verify before trusting the agent's elasticity estimate:**
1. Does the coefficient on log(price) have the expected sign (negative)?
2. Is the magnitude in a plausible range? Most consumer goods: −0.3 to −4.0. Outside this range, investigate whether there is a data problem.
3. Is the model correctly specified? Fixed effects for store and product should be present if you have panel data. Without them, unobserved store-level and product-level differences will bias the elasticity estimate.
4. Do the predicted vs. actual values cluster around the 45-degree line? Large deviations indicate systematic misspecification.
