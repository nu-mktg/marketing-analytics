# Chapter 2: Price Elasticity of Demand and OLS Regression
## Measuring How Price Changes Affect Revenue

---

### Chapter Overview

This chapter derives ordinary least squares (OLS) regression from first principles,
develops the log-log demand model that yields price elasticity directly as a
coefficient, and works through the endogeneity problem that makes naive OLS estimates
unreliable in pricing contexts.

**By the end of this chapter you should be able to:**
- Derive the OLS normal equations by minimizing sum of squared residuals
- Apply the OLS formula by hand using deviation scores
- Interpret elasticity and classify demand as elastic, inelastic, or unit-elastic
- Find the revenue-maximizing price analytically
- Identify endogeneity bias and its direction in pricing regressions
- Explain why R² measures fit, not causation

---

## 2.1 Strategic Frame: Pricing as Value Measurement

### The Opening Story: Madhavan Ramanujam and the Pricing Paradox

Madhavan Ramanujam is a senior partner at Simon-Kucher & Partners, the world's largest
pricing strategy consultancy, and co-author of *Monetizing Innovation*. He has advised
over 250 companies on pricing, including LinkedIn, Uber, DoorDash, and Asana.

His central provocation: most companies treat pricing as a cost-plus calculation. They
determine what it costs to build the product, add a target margin, and call that the
price. The result is that pricing happens after the product is built — as an afterthought,
not a strategy.

His alternative sequencing: understand what customers value, what they are willing to
pay, and what price points create viable business models *before* writing a line of code.
"Price is a measure of value," he says — just as a liter is a measure of volume. If
you have accurately measured what the product is worth to customers, the price follows.
If you have not, no amount of post-launch optimization will fix a fundamentally mispriced product.

**The Evernote failure mode: Naomi Ionita**

Naomi Ionita led monetization at Evernote and is now a partner at Menlo Ventures.
Her retrospective on Evernote is a masterclass in what happens when pricing strategy
is not revisited regularly.

Evernote spent years as one of the most beloved productivity tools in tech. It had high
retention among its most engaged users and a freemium model that converted a steady
fraction to paid. What it lacked was a systematic process for reassessing whether the
product's value had changed — and whether the pricing reflected that change.

The deeper diagnosis she offers: Evernote was "philosophically antisocial" — built
from the start as a single-player note-taking tool in an era when every successful
consumer product was becoming multiplayer. Teams using Evernote informally were
willing to pay enterprise-grade prices for a team collaboration product. Evernote never
built that product. The pricing analysis that would have revealed this gap — conjoint
research on team vs. individual value, segmented elasticity by use case — was never run.

Her rule, stated plainly: "Think about your pricing just like you do your product
roadmap. Every 6 to 12 months, there's probably something meaningful you're launching
for users that justifies a pricing conversation."

### What Practitioners Know: Three Strategic Insights

**Insight 1: Elasticity is not one number — it varies by segment, channel, and tier**

A log-log OLS regression on aggregate revenue data returns a single elasticity
coefficient. This is useful as a baseline but misleading as a decision tool.

In practice, enterprise customers are typically much less price-sensitive than SMB
customers. Annual subscribers are less elastic than monthly subscribers (they have
already demonstrated commitment). Customers who found you through organic search may
be less elastic than those who responded to a promotion.

The business implication: segmented pricing — offering different price points to
different segments — is the practical response to heterogeneous elasticity. A single
price optimized for the average elasticity will be too low for inelastic segments (leaving
revenue on the table) and too high for elastic segments (driving churn).

**Insight 2: Revenue-maximizing price is not profit-maximizing price**

The formula derived in Section 2.6 shows that revenue is maximized where |ε| = 1.
But if a 10% price increase causes some customers to churn, the long-run effect on
customer lifetime value is not captured in a simple demand curve. Acquisition costs
make churned customers expensive to replace. Support costs may change with customer
mix. The profit-maximizing price is typically higher than the revenue-maximizing price
— which is already higher than where most companies price.

**Insight 3: Willingness-to-pay research precedes, not follows, the pricing decision**

Ramanujam's most consistent finding across 250+ companies: teams that do willingness-to-pay
research before building generate more revenue from new products than teams that price
after launch. This is not primarily because they find higher prices. It is because they
build different features — the ones customers actually value, not the ones customers
articulate most loudly in user interviews.

---

## 2.2 Mathematical Prerequisites

### 2.2.1 Summation Notation

We use the symbol Σ to denote summation:
$$\sum_{i=1}^n x_i = x_1 + x_2 + \cdots + x_n$$

Key summation identities used in OLS derivations:
$$\sum_{i=1}^n (x_i - \bar{x}) = 0 \quad \text{(deviations from mean sum to zero)}$$
$$\sum_{i=1}^n (x_i - \bar{x})^2 = \sum_{i=1}^n x_i^2 - n\bar{x}^2 \quad \text{(computational formula for variance)}$$
$$\sum_{i=1}^n (x_i - \bar{x})(y_i - \bar{y}) = \sum_{i=1}^n x_i y_i - n\bar{x}\bar{y} \quad \text{(computational formula for covariance)}$$

### 2.2.2 Calculus: Derivatives for Optimization

The OLS derivation requires setting derivatives to zero. Key rules:

$$\frac{d}{d\beta_1}\left[\sum (y_i - \beta_0 - \beta_1 x_i)^2\right]$$

By the chain rule: $\frac{d}{d\beta_1}(y_i - \beta_0 - \beta_1 x_i)^2 = 2(y_i - \beta_0 - \beta_1 x_i)(-x_i)$

Setting the sum to zero: $-2\sum x_i(y_i - \beta_0 - \beta_1 x_i) = 0$

This is the **first-order condition** for OLS.

### 2.2.3 Natural Logarithm

The natural logarithm ln(x) has two properties essential for elasticity modeling:

**Property 1 (log of product):** $\ln(AB) = \ln A + \ln B$

**Property 2 (derivative):** $\frac{d}{dx}\ln(x) = \frac{1}{x}$, equivalently $d\ln(x) = \frac{dx}{x}$

This second property means: a small change in ln(x) equals a proportional change in x.
If ln(price) increases by 0.01, price increased by approximately 1%. This is the
mathematical foundation for the elasticity interpretation of log-log regression.

*Reference: Wikipedia, "Natural logarithm" — en.wikipedia.org/wiki/Natural_logarithm*

---

## 2.3 Demand Functions and Elasticity

### The Linear Demand Function

The simplest demand model posits a linear relationship between price and quantity:
$$Q = \beta_0 + \beta_1 P + \varepsilon$$

where Q is quantity demanded, P is price, β₁ < 0 (demand falls as price rises),
and ε is an error term capturing factors outside the model.

**Limitation:** A linear model implies constant slope: a $1 price increase always
reduces demand by |β₁| units. This is unrealistic — a $1 increase from $5 to $6
(20% increase) has a very different impact than a $1 increase from $500 to $501 (0.2%).

### Price Elasticity of Demand

**Price elasticity of demand** (ε) measures the percentage change in quantity demanded
per one percent change in price:
$$\varepsilon = \frac{\%\Delta Q}{\%\Delta P} = \frac{dQ/Q}{dP/P} = \frac{dQ}{dP} \cdot \frac{P}{Q}$$

For a linear demand curve Q = β₀ + β₁P: $\varepsilon = \beta_1 \cdot P/Q$

This varies with P and Q — elasticity changes along the demand curve. At the top of
the curve (high P, low Q), elasticity is large in magnitude (elastic). At the bottom
(low P, high Q), elasticity is small (inelastic).

**Elasticity classification:**

| |ε| | Type | Price increase → Revenue |
|---|---|---|
| > 1 | Elastic | Revenue decreases |
| = 1 | Unit-elastic | Revenue unchanged |
| < 1 | Inelastic | Revenue increases |

*Reference: Wikipedia, "Price elasticity of demand" —
en.wikipedia.org/wiki/Price_elasticity_of_demand*

---

## 2.4 OLS Derivation — The Normal Equations

### Setup

We have n observations (x₁, y₁), ..., (xₙ, yₙ). We want to find the linear function
$\hat{y} = \hat{\beta}_0 + \hat{\beta}_1 x$ that minimizes the **sum of squared residuals**:

$$\text{SSR}(\beta_0, \beta_1) = \sum_{i=1}^n (y_i - \beta_0 - \beta_1 x_i)^2$$

### Full Derivation

**Step 1:** Partial derivative with respect to β₀.

$$\frac{\partial \text{SSR}}{\partial \beta_0} = \sum_{i=1}^n 2(y_i - \beta_0 - \beta_1 x_i)(-1) = 0$$

$$\Rightarrow \sum_{i=1}^n y_i = n\beta_0 + \beta_1 \sum_{i=1}^n x_i$$

$$\Rightarrow \bar{y} = \beta_0 + \beta_1 \bar{x} \quad \Rightarrow \quad \boxed{\hat{\beta}_0 = \bar{y} - \hat{\beta}_1 \bar{x}}$$

**Step 2:** Partial derivative with respect to β₁.

$$\frac{\partial \text{SSR}}{\partial \beta_1} = \sum_{i=1}^n 2(y_i - \beta_0 - \beta_1 x_i)(-x_i) = 0$$

$$\Rightarrow \sum_{i=1}^n x_i y_i = \beta_0 \sum_{i=1}^n x_i + \beta_1 \sum_{i=1}^n x_i^2$$

**Step 3:** Substitute $\beta_0 = \bar{y} - \beta_1 \bar{x}$ into the equation from Step 2.

$$\sum x_i y_i = (\bar{y} - \beta_1 \bar{x})\sum x_i + \beta_1 \sum x_i^2$$

$$\sum x_i y_i = \bar{y}\sum x_i - \beta_1 \bar{x}\sum x_i + \beta_1 \sum x_i^2$$

$$\sum x_i y_i - \bar{y}\sum x_i = \beta_1\left(\sum x_i^2 - \bar{x}\sum x_i\right)$$

**Step 4:** Use the identity $\sum (x_i - \bar{x})(y_i - \bar{y}) = \sum x_i y_i - n\bar{x}\bar{y}$ and
$\sum (x_i - \bar{x})^2 = \sum x_i^2 - n\bar{x}^2$.

$$\boxed{\hat{\beta}_1 = \frac{\sum_{i=1}^n (x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^n (x_i - \bar{x})^2} = \frac{S_{xy}}{S_{xx}}}$$

This is the **OLS slope estimator**. The denominator $S_{xx} > 0$ as long as the
x-values are not all identical.

> **Teaching note (iPad/whiteboard):** This derivation takes 15–20 minutes on the board.
> The most important pedagogical moment is Step 3 → Step 4: show students that
> the numerator simplifies to Σ(xᵢ−x̄)(yᵢ−ȳ), which they can interpret verbally as
> "how much x and y move together." Draw the scatter plot alongside. When x is above
> average and y is below average, the product (xᵢ−x̄)(yᵢ−ȳ) is negative. This is why
> a demand curve (higher price → lower quantity) gives a negative slope.

*Reference: Wikipedia, "Ordinary least squares" — en.wikipedia.org/wiki/Ordinary_least_squares*
*Reference: MIT OpenCourseWare, 14.381 Statistical Methods in Economics, Lecture 3 —
ocw.mit.edu/courses/14-381-statistical-method-in-economics-fall-2013/*

### Worked Numerical Example

Five stores in one week. x = price, y = units sold.

| i | xᵢ | yᵢ | xᵢ−x̄ | yᵢ−ȳ | (xᵢ−x̄)(yᵢ−ȳ) | (xᵢ−x̄)² |
|---|---|---|---|---|---|---|
| 1 | 5 | 200 | −2 | +60 | −120 | 4 |
| 2 | 6 | 160 | −1 | +20 | −20 | 1 |
| 3 | 7 | 140 | 0 | 0 | 0 | 0 |
| 4 | 8 | 110 | +1 | −30 | −30 | 1 |
| 5 | 9 | 90 | +2 | −50 | −100 | 4 |
| | **x̄=7** | **ȳ=140** | | | **Σ=−270** | **Σ=10** |

$$\hat{\beta}_1 = \frac{-270}{10} = -27 \text{ units per dollar}$$

$$\hat{\beta}_0 = 140 - (-27)(7) = 140 + 189 = 329$$

**Prediction:** At price = $7.50: $\hat{y} = 329 - 27(7.50) = 329 - 202.5 = 126.5$ units.

### R-Squared

$$R^2 = 1 - \frac{\text{SSR}}{\text{SST}} = 1 - \frac{\sum(y_i - \hat{y}_i)^2}{\sum(y_i - \bar{y})^2}$$

R² measures the proportion of variance in y explained by the regression. R² = 0.87
means 87% of variation in quantity is accounted for by price variation in our model.

**Critical warning:** R² measures goodness of fit — how well the model explains
variation in the data. It does NOT:
- Prove causation
- Indicate the elasticity estimate is unbiased
- Guarantee the model will predict well out-of-sample

A regression with R² = 0.95 can still have severely biased coefficients due to
omitted variables or endogeneity.

---

## 2.5 The Log-Log Demand Model

### Why Log-Log?

The linear model Q = β₀ + β₁P estimates a constant-slope demand curve. The log-log
model transforms both variables:
$$\ln Q = \beta_0 + \beta_1 \ln P + \varepsilon$$

**Why this gives constant elasticity:**

From the definition of elasticity and the log derivative:
$$\varepsilon = \frac{dQ/Q}{dP/P} = \frac{d\ln Q}{d\ln P} = \beta_1$$

The slope of the log-log regression **is** the price elasticity — everywhere on the
demand curve, not just at a specific point. This makes the model far more useful for
elasticity estimation.

### Derivation of the Elasticity Interpretation

From $\ln Q = \beta_0 + \beta_1 \ln P$, differentiate with respect to P:
$$\frac{1}{Q}\frac{dQ}{dP} = \beta_1 \cdot \frac{1}{P}$$

$$\frac{dQ}{dP} \cdot \frac{P}{Q} = \beta_1$$

But the left side is exactly the definition of price elasticity ε. Therefore β₁ = ε. QED.

### Worked Numerical Example

Transform the data from Section 2.4:

| i | xᵢ = ln(price) | yᵢ = ln(units) |
|---|---|---|
| 1 | ln(5)=1.609 | ln(200)=5.298 |
| 2 | ln(6)=1.792 | ln(160)=5.075 |
| 3 | ln(7)=1.946 | ln(140)=4.942 |
| 4 | ln(8)=2.079 | ln(110)=4.700 |
| 5 | ln(9)=2.197 | ln(90)=4.500 |

x̄ = 1.925, ȳ = 4.903

Computing:
- Σ(xᵢ−x̄)(yᵢ−ȳ) ≈ −0.2986
- Σ(xᵢ−x̄)² ≈ 0.1540
- β̂₁ = −0.2986/0.1540 ≈ **−1.94** (price elasticity ≈ −1.94)

**Interpretation:** A 1% increase in price is associated with a 1.94% decrease in
quantity. Demand is elastic (|ε| = 1.94 > 1).

*Reference: Wikipedia, "Log-log plot" and "Elasticity (economics)" —
en.wikipedia.org/wiki/Log-log_plot, en.wikipedia.org/wiki/Elasticity_(economics)*

---

## 2.6 Revenue Maximization

### The Revenue Function

Revenue R = P × Q. Using the log-log model $Q = e^{\beta_0} P^{\beta_1}$:
$$R(P) = P \cdot e^{\beta_0} P^{\beta_1} = e^{\beta_0} P^{1+\beta_1}$$

### Full Derivation of the Revenue-Maximizing Condition

Differentiate with respect to P and set to zero:
$$\frac{dR}{dP} = e^{\beta_0}(1+\beta_1)P^{\beta_1} = 0$$

Since $e^{\beta_0} > 0$ and $P^{\beta_1} \neq 0$:
$$1 + \beta_1 = 0 \implies \beta_1 = -1 \implies \varepsilon = -1$$

**Result:** Revenue is maximized at the price where price elasticity equals −1
(unit-elastic demand). This is a classic result in microeconomics [[Wikipedia,
"Profit maximization" — en.wikipedia.org/wiki/Profit_maximization]].

**Corollary:** Using the second derivative:
$$\frac{d^2R}{dP^2} = e^{\beta_0}\beta_1(1+\beta_1)P^{\beta_1-1}$$

For this to be negative (confirming a maximum), we need $\beta_1(1+\beta_1) < 0$,
which holds when −1 < β₁ < 0. For highly elastic demand (β₁ < −1) or inelastic
demand (−1 < β₁ < 0), the sign of dR/dP determines whether raising or lowering
price increases revenue.

### Revenue Implications by Elasticity Region

| Current elasticity | Revenue effect of price increase |
|---|---|
| |ε| > 1 (elastic) | Revenue decreases → lower price to increase revenue |
| |ε| = 1 (unit-elastic) | Revenue unchanged → at revenue maximum |
| |ε| < 1 (inelastic) | Revenue increases → raise price to increase revenue |

**Worked example:** β₁ = −1.94, current price P₀ = $7.

Current revenue: R₀ = $7 × 126.5 = $885.50 (from linear model approximation)

Since |ε| = 1.94 > 1, revenue is currently above the revenue-maximizing price.
Revenue would increase by lowering price. The revenue-maximizing price P* satisfies:
$$P* = e^{\beta_0/(1+\beta_1)} \cdot e^{-\ln P_0 \cdot \beta_1/(1+\beta_1)}$$

(solved numerically for specific β₀ and β₁ values)

---

## 2.7 Endogeneity and Omitted Variable Bias

### The Problem

OLS requires that the error term ε is uncorrelated with the predictor x:
$$\text{Cov}(x_i, \varepsilon_i) = 0 \quad \text{(exogeneity assumption)}$$

In pricing regressions, this assumption frequently fails. Consider: if a store manager
lowers prices when demand is expected to be slow (e.g., on weekdays) and raises them
when demand is expected to be high (e.g., before holidays), then price P is negatively
correlated with the demand shock ε.

This is **endogeneity** — the predictor is "inside" the model in a way that violates
the independence assumption. The omitted variable (expected demand conditions) affects
both the predictor (price) and the outcome (quantity).

### Bias Direction

Under endogeneity due to a negative price-demand correlation:
$$\hat{\beta}_1 \xrightarrow{p} \beta_1 + \frac{\text{Cov}(P, \varepsilon)}{\text{Var}(P)}$$

If managers lower prices when demand is low: Cov(P, ε) < 0. The second term is
negative. Since β₁ < 0 (demand curve slopes down), the OLS estimate is **more
negative** than the true elasticity — demand appears *more* elastic than it actually is.

Conversely, if managers raise prices when demand is high (common in seasonal
businesses): Cov(P, ε) > 0. The second term is positive. OLS gives an estimate closer
to zero — demand appears *less* elastic (more inelastic) than reality.

**Example:** A grocery store that cuts prices every Saturday because Saturday traffic
is lower will show a very negative price coefficient — but much of the negative
correlation is because Saturdays have lower demand regardless of price.

### The Instrumental Variables Solution

An **instrument** Z is a variable that:
1. Affects P (relevance: Cov(Z, P) ≠ 0)
2. Does not directly affect Q, except through P (exclusion: Cov(Z, ε) = 0)

The instrumental variables (IV) estimator:
$$\hat{\beta}_{1,IV} = \frac{\text{Cov}(Z, y)}{\text{Cov}(Z, x)} = \frac{S_{Zy}}{S_{Zx}}$$

Common instruments in retail pricing:
- **Display promotions:** Store display allocations are planned weeks in advance by supply
  chains, not in response to current demand shocks. They shift price (promotions
  typically lower prices) but are uncorrelated with the demand error.
- **Cost-based price changes:** Input cost shocks (raw materials, energy) shift prices
  without responding to current demand.
- **Competitor price changes in different geographic markets** (under parallel trends assumption).

*Reference: Wikipedia, "Instrumental variables estimation" —
en.wikipedia.org/wiki/Instrumental_variables_estimation*
*Reference: MIT OpenCourseWare 14.382 Econometrics — free lecture notes at ocw.mit.edu*

---

## 2.8 Multivariate Extension

### Adding Controls

In practice, quantity demanded depends on more than price alone. The multivariate
log-log model:
$$\ln Q = \beta_0 + \beta_1 \ln P_{own} + \beta_2 \ln P_{comp} + \beta_3 D_{display} + \varepsilon$$

where:
- β₁ = own-price elasticity (negative for a normal good)
- β₂ = cross-price elasticity with competitor's product
- β₃ = effect of in-store display promotion (log-linear, so β₃ ≈ % lift from display)

**Interpreting cross-price elasticity:**
- β₂ > 0: substitutes — competitor price increase shifts demand toward our product
- β₂ < 0: complements — competitor price increase reduces demand for our product
- β₂ = 0: unrelated products

**OLS in matrix form:** For p predictors plus intercept, OLS minimizes:
$$\text{SSR} = (\mathbf{y} - \mathbf{X}\boldsymbol{\beta})^\top(\mathbf{y} - \mathbf{X}\boldsymbol{\beta})$$

Setting gradient to zero: $\mathbf{X}^\top\mathbf{X}\hat{\boldsymbol{\beta}} = \mathbf{X}^\top\mathbf{y}$

These are the **normal equations**. The solution:
$$\hat{\boldsymbol{\beta}} = (\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y}$$

provided X'X is invertible (no perfect multicollinearity).

*Reference: Wikipedia, "Ordinary least squares — Matrix formulation" —
en.wikipedia.org/wiki/Ordinary_least_squares#Matrix/vector_formulation*

---

## 2.9 Full Proof Appendix

### Proof A: OLS Estimator is Unbiased (Under Gauss-Markov Assumptions)

The Gauss-Markov assumptions:
1. Linearity: y = Xβ + ε
2. Full rank: X'X is invertible
3. Exogeneity: E[ε|X] = 0
4. Homoskedasticity: Var(ε|X) = σ²I
5. No serial correlation: Cov(εᵢ, εⱼ|X) = 0 for i ≠ j

Under these assumptions, $\hat{\beta} = (X'X)^{-1}X'y$:
$$E[\hat{\beta}|X] = E[(X'X)^{-1}X'(X\beta + \varepsilon)|X]$$
$$= \beta + (X'X)^{-1}X' E[\varepsilon|X] = \beta + 0 = \beta$$

**The Gauss-Markov theorem** states that under assumptions 1–5, OLS is the Best Linear
Unbiased Estimator (BLUE): it has the smallest variance among all linear unbiased
estimators. This is why OLS is the default method.

*Reference: Wikipedia, "Gauss-Markov theorem" — en.wikipedia.org/wiki/Gauss%E2%80%93Markov_theorem*

### Proof B: Revenue is Maximized Where |ε| = 1 (General Case)

For any differentiable demand function Q(P):
$$R(P) = P \cdot Q(P)$$

$$\frac{dR}{dP} = Q(P) + P \frac{dQ}{dP} = Q(P)\left[1 + \frac{P}{Q}\frac{dQ}{dP}\right] = Q(P)[1 + \varepsilon]$$

Setting dR/dP = 0 and noting Q(P) > 0:
$$1 + \varepsilon = 0 \implies \varepsilon = -1$$

This holds for any demand function Q(P), not just the log-log model. QED.

---

## 2.10 Verification Checklist

| Check | How to verify |
|---|---|
| β̂₁ = Σ(xᵢ−x̄)(yᵢ−ȳ) / Σ(xᵢ−x̄)²? | Compute deviations table by hand |
| β̂₀ = ȳ − β̂₁x̄? | Plug in values |
| Log-log β₁ = elasticity directly? | No additional computation needed |
| |β₁| > 1 implies elastic demand? | Revenue decreases when price rises |
| Revenue-maximizing condition: |ε| = 1? | Compute analytically or numerically |
| Endogeneity concern identified? | Ask: does anyone adjust prices based on current demand? |
| Cross-price elasticity sign: positive = substitutes? | Competitor price up → your demand up |

---

## 2.11 Chapter Bibliography

1. Wikipedia, "Ordinary least squares" — en.wikipedia.org/wiki/Ordinary_least_squares
2. Wikipedia, "Price elasticity of demand" — en.wikipedia.org/wiki/Price_elasticity_of_demand
3. Wikipedia, "Gauss-Markov theorem" — en.wikipedia.org/wiki/Gauss%E2%80%93Markov_theorem
4. Wikipedia, "Instrumental variables estimation" — en.wikipedia.org/wiki/Instrumental_variables_estimation
5. Wikipedia, "Endogeneity (econometrics)" — en.wikipedia.org/wiki/Endogeneity_(econometrics)
6. MIT OpenCourseWare, 14.381 Statistical Method in Economics — ocw.mit.edu/courses/14-381-statistical-method-in-economics-fall-2013/
7. MIT OpenCourseWare, 14.382 Econometrics — ocw.mit.edu/courses/14-382-econometrics-spring-2017/
8. Wooldridge, J. M. Lecture notes on introductory econometrics — free slides at msu.edu/~ec/faculty/wooldridge/current%20textbook%20materials.htm
9. Varian, H. R. (1992). *Microeconomic Analysis*, 3rd ed. Chapter 15 (Profit Maximization). Widely cited; publisher preview available.
10. Berry, S., Levinsohn, J., Pakes, A. (1995). "Automobile Prices in Market Equilibrium." *Econometrica* 63(4):841–890. arXiv preprint version available via NBER.

**Lenny's Podcast episode references:**
- Madhavan Ramanujam episodes on pricing strategy — Lenny's Podcast
- Naomi Ionita episode on growth and monetization — Lenny's Podcast

