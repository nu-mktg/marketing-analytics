# Lecture 2: Price Elasticity of Demand
## How Price Changes Affect Revenue — and How to Measure It

---

### Overview

**Business question:** If you raise prices by 10%, will revenue go up or down? By how much?

**What you will be able to do after this lecture:**
- Derive the OLS slope formula using only addition and multiplication
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

> **If y = ln(x), then a 1-unit increase in x from its current level is approximately a 100% proportional change.**

More practically: in a log-log regression, the slope coefficient equals the *percentage* change in y for a 1% change in x — which is the definition of elasticity.

**Key fact:** ln(AB) = ln(A) + ln(B). Multiplying becomes adding in log space.

> 🔍 **Deep Dive:** Why ln specifically (and not log base 10)?
> The natural log has a unique property: d/dx[ln(x)] = 1/x. This means the slope of ln(x) at any point equals the proportional change rate. It is the only log function where a 1-unit increase in ln(x) exactly corresponds to a 100% increase in x. This is why economists and statisticians use ln for elasticity models.

---

### Section 1.2 — Business Motivation

---

#### The Pricing Decision

You manage a grocery chain. A category manager wants to raise the price of a product. Before changing the price, you need to know: will revenue go up or down?

The answer depends on **price elasticity of demand** — how sensitive customers are to price changes.

| Elasticity |ε| | Type | 10% price increase → |
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

**The endogeneity problem (preview):** If you raise prices when demand is high and lower them when demand is slow, then observed prices and observed demand are correlated in a way that biases the regression estimate. This is one of the most important practical problems in pricing analytics — we will return to it in Section 1.4.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: OLS in Plain Algebra

We have n observations of (price, units sold). We want to find the line ŷ = β₀ + β₁x that best fits the data — where "best" means minimizing the sum of squared errors.

The formula for the slope is:

$$\hat{\beta}_1 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^{n}(x_i - \bar{x})^2}$$

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
> OLS minimizes the sum of squared residuals: SSR = Σ(yᵢ − β₀ − β₁xᵢ)². To find the minimum, we take the derivative with respect to β₁ and set it equal to zero:
>
> ∂SSR/∂β₁ = −2Σ(yᵢ − β₀ − β₁xᵢ)xᵢ = 0
>
> Substituting β₀ = ȳ − β₁x̄ (from the other first-order condition) and rearranging gives:
>
> Σ(xᵢ − x̄)(yᵢ − ȳ) = β₁ × Σ(xᵢ − x̄)²
>
> Dividing both sides by Σ(xᵢ − x̄)² yields the formula above. The key insight: OLS is the unique linear estimator that is unbiased under the assumption that errors are uncorrelated with the predictor.

---

#### Part B: The Log-Log Model for Elasticity

The OLS above gives "units fall by 27 for every $1 price increase" — a constant slope that does not make economic sense for large price changes. The log-log model fixes this by working in percentage terms:

$$\ln(\text{units}) = \beta_0 + \beta_1 \ln(\text{price}) + \varepsilon$$

**Why log-log?** In this model, β₁ has a specific interpretation:

> A 1% increase in price is associated with a β₁% change in units sold.

This is the definition of price elasticity. β₁ is the elasticity directly — no additional calculation needed.

**The revenue-maximizing price:** Revenue = Price × Quantity. Revenue is maximized where elasticity |β₁| = 1. When |β₁| > 1 (elastic), revenue increases as you lower price. When |β₁| < 1 (inelastic), revenue increases as you raise price.

> ### 🔍 Deep Dive: Why |ε| = 1 Maximizes Revenue
> *Skip this if the result makes intuitive sense. Read it if you want the algebraic proof.*
>
> Revenue R = P × Q. Taking the derivative with respect to P and setting it to zero:
>
> dR/dP = Q + P × (dQ/dP) = Q(1 + P/Q × dQ/dP) = Q(1 + ε) = 0
>
> This equals zero when ε = −1. Since Q > 0 always, the condition reduces to 1 + ε = 0, or ε = −1 (equivalently |ε| = 1).

---

#### Part C: The Endogeneity Problem

If prices are set partly in response to demand conditions (managers lower prices during slow periods, raise them during busy ones), then price and the error term in the regression are correlated. OLS requires that predictors are uncorrelated with errors. When this fails, the estimate is **biased** — typically toward zero, making demand appear less elastic than it truly is.

**What this means practically:** If your regression returns elasticity = −0.3 but you know managers are cutting prices when sales are slow, the true elasticity is probably more negative. You are underestimating how much customers care about price.

**The fix (preview):** An instrumental variable — something that affects price but not demand directly — can break the correlation. Store display promotions (which change price but do not reflect underlying demand conditions) are a common instrument in retail pricing research.

---

### Section 1.5 — Interpretation Guide

**Elasticity = −1.84: What does this mean?**
- Demand is elastic (|1.84| > 1)
- A 1% price increase → 1.84% decrease in units sold
- A 10% price increase → 18.4% decrease in units sold
- Revenue is currently ABOVE the revenue-maximizing price — lowering price would increase revenue

**R² = 0.87: What does this mean?**
- 87% of the variation in log(units) is explained by the regression predictors
- This is R-squared for explained variation, NOT causation
- High R² does not mean the elasticity estimate is unbiased

**Cross-price elasticity = +0.52: What does this mean?**
- A 1% increase in the competitor's price → 0.52% increase in your units
- Positive cross-price elasticity = the products are substitutes
- Customers switch to you when the competitor gets more expensive

---

### Part 1 Checkpoint

1. You estimate β₁ = −1.84 in a log-log regression. A colleague says "demand is elastic." Are they right?

2. Using the table below, calculate β₁ manually. Prices: [8, 9, 10, 11, 12]. Units: [120, 100, 90, 75, 65].

3. Your elasticity estimate is −1.84. What is the approximate revenue-maximizing price relative to the current price? (Higher, lower, or current price is already optimal?)

4. A store manager lowers prices every Saturday because traffic is lower on weekends. If you run OLS on this store's data, will the elasticity estimate be too large (more negative) or too small (less negative) than the true value?

5. Cross-price elasticity = +0.62. Should you be concerned about a competitor's planned 15% price increase? Why?

---

### Checkpoint Answer Key

**Q1.** Yes. Elastic demand means |ε| > 1. Since |−1.84| = 1.84 > 1, demand is elastic. A price increase reduces revenue because the percentage quantity drop outweighs the percentage price rise.

*Common wrong answer:* "Elastic means demand responds a lot, so −1.84 must be inelastic since it's less than 1." The elasticity is compared to 1 in absolute value, not as a raw number. ε = −1.84 means |ε| = 1.84 > 1 → elastic.

**Q2.** x̄ = 10, ȳ = 90. Deviations: (−2, −1, 0, +1, +2) and (+30, +10, 0, −15, −25).
Numerator = (−2)(30) + (−1)(10) + (0)(0) + (1)(−15) + (2)(−25) = −60 − 10 + 0 − 15 − 50 = −135.
Denominator = 4 + 1 + 0 + 1 + 4 = 10. β₁ = −135/10 = **−13.5 units per dollar**.

*Common wrong answer:* Forgetting to compute deviations first, using raw values instead of (xᵢ − x̄) and (yᵢ − ȳ).

**Q3.** Revenue is maximized where |ε| = 1. Current |ε| = 1.84 > 1 means we are on the elastic portion of the demand curve. Lowering price would increase revenue — current price is **too high** relative to the revenue-maximizing price.

*Common wrong answer:* "Elastic means customers are sensitive, so raising prices will hurt us — we should lower prices to the minimum." Elasticity tells you the direction for revenue maximization, not how much to move.

**Q4.** Too small (less negative) — biased toward zero. When the manager lowers prices during slow periods, observed low prices coincide with low demand (not because of the price cut, but because it's Saturday). The regression incorrectly attributes low demand to the low price, making demand appear less sensitive to price than it truly is.

*Common wrong answer:* "The estimate will be more negative because cutting prices on slow days shows prices matter." The direction of endogeneity bias is counterintuitive — it depends on the correlation between the instrument (price) and the omitted variable (day-of-week demand), not on the direction of the price cut.

**Q5.** Yes — this is a good thing for you. Cross-price elasticity of +0.62 means a 15% competitor price increase → approximately 15 × 0.62 = 9.3% increase in your units. You could consider whether to maintain your price (capturing volume) or raise yours slightly (capturing some of the competitor's pricing power). Do not be alarmed — positive cross-price elasticity means the competitor's higher price benefits you.

*Common wrong answer:* "Positive cross-price elasticity means our products are complements, so if the competitor raises prices, demand for our product will also fall." Positive cross-price elasticity means substitutes (customers switch to us), negative means complements. Complements fall together in demand; substitutes rise when the other's price rises.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Steps 1–3 first, then reveal full solution)

**[INSTRUCTOR NOTE]** Distribute the problem. Give students 10 minutes to attempt Parts (a)–(d) individually. These are all hand-calculations using the formulas from Part 1. Then work through the solution together.

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
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Setup:** Open the GitHub Classroom assignment for Lecture 2. The repository contains:
- `data/pricing_data.csv` — weekly price and sales data for three products across 50 stores and 104 weeks
- `homework_02.ipynb` — the notebook you will complete and push

---

#### Part A: Math Questions (no agent required)

Use this small dataset for all math questions:

| Week | Price ($) | Units Sold |
|---|---|---|
| 1 | 5.00 | 200 |
| 2 | 6.00 | 160 |
| 3 | 7.00 | 140 |
| 4 | 8.00 | 110 |
| 5 | 9.00 | 90 |

**Q1.** Compute the mean price $\bar{x}$. Round to 2 decimal places.
```python
q1_mean_price = None
```
`[ANSWER KEY: 7.00]`

**Q2.** Compute the mean units sold $\bar{y}$. Round to 2 decimal places.
```python
q2_mean_units = None
```
`[ANSWER KEY: 140.0]`

**Q3.** Compute $\sum(x_i - \bar{x})(y_i - \bar{y})$. (Build the deviation table as in the worked example.)
```python
q3_cov_numerator = None
```
`[ANSWER KEY: -200.0]`

**Q4.** Compute $\sum(x_i - \bar{x})^2$.
```python
q4_var_denominator = None
```
`[ANSWER KEY: 10.0]`

**Q5.** Compute the OLS slope β_1.
```python
q5_beta1 = None
```
`[ANSWER KEY: -20.0]`

**Q6.** Compute the OLS intercept β_0.
```python
q6_beta0 = None
```
`[ANSWER KEY: 140 - (-20)(7) = 140 + 140 = 280.0]`

**Q7.** Using your regression line, what is the predicted units sold when price = $7.50?
```python
q7_prediction = None
```
`[ANSWER KEY: 329 - 27(7.50) = 329 - 202.5 = 126.5]`

**Q8.** The natural log of 5.00 is approximately 1.6094, and the natural log of 200 is approximately 5.2983. In the log-log model, which of these would be used as the x-variable (predictor)?
```python
# Enter "log_price" or "log_units"
q8_log_predictor = None
```
`[ANSWER KEY: "log_price"]`

**Q9.** In a log-log regression of log(units) on log(price), you estimate β_1 = −1.4. A 1% increase in price is associated with what percentage change in units sold?
```python
# Enter the percentage change (e.g., enter -2.3 for a 2.3% decrease)
q9_pct_change = None
```
`[ANSWER KEY: -1.4]`

**Q10.** With an elasticity of −1.4, is demand elastic or inelastic?
```python
# Enter "elastic" or "inelastic"
q10_elastic_or_not = None
```
`[ANSWER KEY: "elastic"]`

**Q11.** With an elasticity of −1.4, if you raise price by 5%, what happens to total revenue?
```python
# Enter "increases", "decreases", or "stays the same"
q11_revenue_effect = None
```
`[ANSWER KEY: "decreases"]`

---

#### Part B: Agent Questions

**Step 1:** Paste the following **Context Prompt** into your agent:

```
I am a pricing analyst studying price elasticity of demand using weekly
sales data from a grocery chain.

The dataset pricing_data.csv contains the following columns:
- week: week number (1 to 104, representing 2 years)
- store_id: store identifier (50 stores)
- product_id: product identifier (3 products)
- units_sold: weekly units sold
- price: price per unit in dollars
- competitor_price: competitor's price for the same product
- display_flag: 1 if the product was on in-store display this week, 0 otherwise

I want to estimate price elasticity using a log-log regression model:
log(units_sold) = beta_0 + beta_1 * log(price) + beta_2 * log(competitor_price)
                + beta_3 * display_flag + store_fixed_effects + product_fixed_effects

Please:
1. Load the data and show the first 5 rows
2. Create log-transformed variables: log_units, log_price, log_comp_price
3. Fit the log-log OLS regression using statsmodels, including store and
   product fixed effects as dummy variables
4. Print the full regression summary table
5. State the estimated own-price elasticity and cross-price elasticity
   clearly, and interpret each in one sentence
6. Plot predicted log(units) versus actual log(units) as a scatter plot
   to visualize model fit
```

**Step 2:** Run the agent and fill in:

**Q12.** What is the estimated own-price elasticity (coefficient on log_price)? Round to 2 decimal places.
```python
q12_own_price_elasticity = None
```
`[ANSWER KEY: approximately −1.80 to −2.20 depending on dataset; provide exact value after running]`

**Q13.** Is the own-price elasticity statistically significant? (Look at the p-value for log_price. Enter True if p-value < 0.05, False otherwise.)
```python
q13_significant = None
```
`[ANSWER KEY: True]`

**Q14.** What is the cross-price elasticity (coefficient on log_comp_price)? Round to 2 decimal places.
```python
q14_cross_price_elasticity = None
```
`[ANSWER KEY: approximately +0.40 to +0.90; positive because a higher competitor price shifts demand to our product]`

**Q15.** What is the R-squared of the regression? Round to 2 decimal places.
```python
q15_r_squared = None
```
`[ANSWER KEY: approximately 0.75 to 0.90]`

---

#### Part C: Interpretation Questions

**Q16.** The cross-price elasticity you found in Q14 is positive. What does a positive cross-price elasticity mean about the relationship between the focal product and the competitor product?
- (a) The two products are complements — when the competitor's price rises, people buy less of both products
- (b) The two products are substitutes — when the competitor's price rises, people switch to our product
- (c) The two products are unrelated — the competitor's price has no effect on our sales
- (d) The two products are identical — customers do not distinguish between them
```python
q16_cross_price_meaning = None  # Enter "a", "b", "c", or "d"
```
`[ANSWER KEY: "b"]`

**Q17.** A manager argues: "Our regression proves that raising price causes sales to drop." Is this statement correct?
- (a) Yes — a negative regression coefficient on price proves a causal relationship
- (b) No — regression shows correlation, not causation; prices may be set lower when demand is expected to be low, creating a confounding effect
- (c) No — regression cannot detect any relationship between price and sales
- (d) Yes — because R² is high, we can conclude causation
```python
q17_causation = None  # Enter "a", "b", "c", or "d"
```
`[ANSWER KEY: "b"]`

---

**[INSTRUCTOR NOTE]** The exact values for Q12–Q15 depend on the synthetic dataset. Generate the dataset with a fixed seed before class, run the agent yourself once to record exact outputs, and update the answer key accordingly. Acceptable range is ±0.15 for elasticities and ±0.05 for R-squared. Use `abs(student_answer - key_answer) < tolerance` in the autograder.

---

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
| Creates log_price and log_units columns | Section 1.4D — log transformation |
| Fits OLS regression | Section 1.4A — minimizing sum of squared residuals |
| Prints coefficient on log_price | Section 1.4D — elasticity = β_1 in log-log model |
| Prints p-values | Section 2.2 — statistical significance check |
| Prints R-squared | Section 1.4C — proportion of variance explained |
| Plots predicted vs. actual | Direct visualization of residuals from Section 1.3 |

**What to verify before trusting the agent's elasticity estimate:**
1. Does the coefficient on log(price) have the expected sign (negative)?
2. Is the magnitude in a plausible range? Most consumer goods: −0.5 to −3.5. Outside this range, investigate whether there is a data problem.
3. Is the model correctly specified? Fixed effects for store and product should be present if you have panel data. Without them, unobserved store-level and product-level differences will bias the elasticity estimate.
4. Do the predicted vs. actual values cluster around the 45-degree line? Large deviations indicate systematic misspecification.
