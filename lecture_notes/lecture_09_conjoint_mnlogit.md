# Lecture 9: Conjoint Analysis
## What Do Customers Actually Value — and How Much Will They Pay?

---

### Overview

**Business question:** Should you charge more for an ad-free tier? How much more? Which product feature drives the most willingness-to-pay — and is it the one you've been building?

**What you will be able to do:**
- Explain why conjoint analysis reveals preferences that direct surveys cannot
- Compute WTP for any feature from logit model coefficients
- Run a market simulation to predict share of a new product configuration
- Explain the IIA property and when it misleads
- Distinguish average WTP from individual WTP and price-setting implications

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: The Softmax Function

Given a set of utilities U₁, U₂, ..., Uₙ for n alternatives, the probability of choosing alternative i is:

$$P(\text{choose } i) = \frac{e^{U_i}}{e^{U_1} + e^{U_2} + ... + e^{U_n}}$$

**Example:** Three products with utilities U₁ = −1.5, U₂ = 0.5, U₃ = −0.8:
> e^(−1.5) = 0.223, e^(0.5) = 1.649, e^(−0.8) = 0.449
> Sum = 0.223 + 1.649 + 0.449 = 2.321
> P(choose product 2) = 1.649/2.321 = **0.71** (71% market share)

---

#### Tool 2: Ratio of Coefficients

Willingness-to-pay for a feature = (feature coefficient) / |price coefficient|

> WTP = β_feature / |β_price|

This is derived by asking: how much higher price makes the premium version equally attractive as the standard version?

---

### Section 1.2 — Business Motivation

---

#### 20% of Features Drive 80% of WTP

Madhavan Ramanujam, who has advised over 250 companies on pricing:

> *"20% of what you build drives 80% of the willingness to pay. But the irony is that that 20% is the easiest thing to build."*

His research finding: teams that do not study willingness-to-pay before building tend to invest heavily in features customers appreciate but would not pay extra for — while neglecting the features that actually unlock pricing power.

The counterpart to his first insight comes from Naomi Ionita, who led growth at Evernote: Evernote spent years without revisiting its pricing model. Their failure was not the pricing number — it was building a single-player tool in an era when every metric improves when products go multiplayer. Conjoint analysis, by forcing respondents to make real trade-offs between feature bundles and price points, would have revealed this latent preference far earlier than any customer satisfaction survey.

---

### Section 1.3 — Conceptual Framework

---

#### Why Conjoint Beats Direct Price Surveys

If you ask a customer "how much would you pay for an ad-free version?", they will consistently overstate their WTP. Social desirability bias makes customers feel they should value things highly; hypothetical bias means there is no real cost to saying "$20/month!"

Conjoint analysis avoids this by never asking about price directly. Instead, respondents choose between complete product configurations: "Would you prefer Plan A ($12, standard content, no ads) or Plan B ($18, premium content, limited ads)?" The price sensitivity is revealed through the trade-offs they make, not through a direct question.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: Utility Model

Each product is characterized by attributes (price, content tier, ad level, plan type). The utility of a product is a linear combination:

$$U = \beta_{\text{price}} \times \text{price} + \beta_{\text{premium}} \times \mathbb{1}[\text{premium}] + \beta_{\text{no\_ads}} \times \mathbb{1}[\text{no\_ads}] + ...$$

where 𝟙[...] = 1 if the attribute level is present, 0 otherwise.

**Example coefficients:**
- β_price = −0.18 (per $1 increase in monthly price)
- β_premium_content = +1.26 (vs. standard content)
- β_no_ads = +2.00 (vs. full ads)
- β_annual_plan = +0.65 (vs. monthly)

**WTP calculation:**
> WTP for premium content = 1.26 / 0.18 = **$7.00/month**
> WTP for no-ads (vs full ads) = 2.00 / 0.18 = **$11.11/month**

**Plain English:** On average across all respondents, customers would pay $7 more per month to upgrade from standard to premium content, and $11.11 more per month to eliminate all ads.

---

#### Part B: Market Simulation

To predict market shares for a set of competing products, compute utility for each, then apply softmax:

**Products:**
- A: standard, full_ads, monthly, $12 → U_A = −0.18(12) + 0 + 0 + 0 = −2.16
- B: premium, no_ads, monthly, $18 → U_B = −0.18(18) + 1.26 + 2.00 = −3.24 + 3.26 = **+0.02**
- C: standard, limited_ads, annual, $15 → U_C = −0.18(15) + 0 + 0.80 + 0.65 = −2.70 + 1.45 = **−1.25**

> e^(−2.16) = 0.115, e^(+0.02) = 1.020, e^(−1.25) = 0.287. Sum = 1.422
> Shares: A = 8.1%, B = 71.7%, C = 20.2%

---

#### Part C: The IIA Property and Its Limitation

The multinomial logit model assumes IIA: **Independence of Irrelevant Alternatives**. Adding a new product draws share from all existing products proportionally to their current shares — regardless of how similar the new product is to existing ones.

**Why this is often wrong:** If you add Plan C' (very similar to Plan C, but $1 cheaper), IIA predicts it draws from A and B proportionally. In reality, it would draw mostly from C (the similar product). IIA cannot model this substitution pattern.

**Practical implication:** Market simulation results are more reliable when the products being compared are genuinely different. When testing very similar variants, IIA leads to overestimation of share for the new variant and underestimation of cannibalization from the most similar existing product.

> ### 🔍 Deep Dive: Why Logit Implies IIA
> The multinomial logit model arises from the assumption that unobserved utility components follow an independent Gumbel distribution. Under this assumption, P(choose i) = e^(U_i) / Σⱼ e^(U_j). For any two alternatives i and k: P(i)/P(k) = e^(U_i − U_k). This ratio depends only on i and k, not on any other alternative in the choice set. That is the IIA property: adding or removing any third option does not change the ratio of probabilities for any pair of options.

---

### Part 1 Checkpoint

1. β_price = −0.15, β_premium_content = +1.10. Compute WTP for premium content.

2. Two products: A = standard, full ads, $10 (U_A = −0.15×10 + 0 + 0 = −1.5). B = premium, no ads, $16 (U_B = −0.15×16 + 1.10 + 1.80 = −2.4 + 2.90 = 0.5). Compute market shares for both.

3. Conjoint WTP for no-ads = $12. Your current ad-supported tier costs $10. A PM recommends raising the no-ads tier to $22 immediately. What caution would you add?

4. True or False: WTP from conjoint represents the minimum price customers will pay.

5. A new product identical to Product A but $1 cheaper is launched. Under IIA, what happens to Product A's market share?

---

### Checkpoint Answer Key

**Q1.** WTP = 1.10 / 0.15 = **$7.33/month**.
*Common wrong answer:* WTP = 0.15/1.10 = $0.136 (inverted formula). Always divide the feature coefficient by |β_price|, not the reverse.

**Q2.** e^(−1.5) = 0.223, e^(0.5) = 1.649. Sum = 1.872.
Share A = 0.223/1.872 = **11.9%**. Share B = 1.649/1.872 = **88.1%**.
*Common wrong answer:* Sharing proportional to utility values directly (−1.5 and 0.5), ignoring the softmax transformation.

**Q3.** Conjoint WTP of $12 is an **average** — individual WTP varies. Some customers have WTP below $12 and would not pay $22. Raising to $22 (= $10 + $12 = current price + average WTP) would lose customers whose individual WTP falls between $12 and $22. Raising to $22 may be profitable if the revenue from those who stay exceeds revenue from those who churn, but this requires demand curve analysis — not just the average WTP estimate.
*Common wrong answer:* The PM is correct — WTP is $12 so the tier should cost current price + $12 = $22. Average WTP is not the price at which everyone buys; it is the average maximum each person would pay.

**Q4.** **False.** WTP is the **maximum** price a customer would accept, not the minimum. The correct statement: WTP is the maximum price at which a customer would still choose this option over the alternative.
*Common wrong answer:* True, because WTP represents the lowest price you should charge to maintain demand.

**Q5.** Under IIA, the new product draws share from A and B proportionally to their current shares. If A currently has 11.9% and B has 88.1%, the new product takes 11.9% of its share from A and 88.1% from B — even though the new product is identical to A. Product A's share decreases by its proportion of the total.
*Common wrong answer:* All of the new product's share comes from A (since they are identical). This is the realistic prediction — but IIA gives the proportional answer, which is why IIA is often wrong in near-substitute scenarios.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal)


---

**Problem Statement**

A streaming service estimates the following utility coefficients from a conjoint survey of 400 respondents:

| Attribute / Level | Coefficient (β) |
|---|---|
| price (per dollar increase) | −0.15 |
| Premium content (vs. standard) | +1.20 |
| Full ads (vs. no ads) | −2.00 |
| Limited ads (vs. no ads) | −0.80 |

Note: "no ads" and "standard content" serve as reference levels with coefficient 0.

**Part A:** A respondent faces two options:
- Option A: \$8/month, standard content, no ads
- Option B: \$13/month, premium content, no ads

Compute $V_A$, $V_B$, and $P(\text{choose B})$.

**Part B:** Compute the WTP for "premium content" using the WTP formula.

**Part C:** Simulate market shares for three competing products:
- Product 1: \$13/month, standard content, no ads
- Product 2: \$18/month, premium content, no ads
- Product 3: \$8/month, standard content, full ads

---

**Full Solution**

**Part A: Utilities and Choice Probability**

$$V_A = (-0.15)(8) + 0 \text{ (standard)} + 0 \text{ (no ads)} = -1.20$$

$$V_B = (-0.15)(13) + 1.20 \text{ (premium)} + 0 \text{ (no ads)} = -1.95 + 1.20 = -0.75$$

$$e^{V_A} = e^{-1.20} \approx 0.3012, \qquad e^{V_B} = e^{-0.75} \approx 0.4724$$

$$P(\text{choose B}) = \frac{0.4724}{0.3012 + 0.4724} = \frac{0.4724}{0.7736} \approx \mathbf{0.611}$$

Option B is preferred (61% probability of being chosen) despite costing \$5 more, because the premium content benefit more than compensates for the price difference.

**Part B: WTP for Premium Content**

$$\text{WTP} = \frac{\beta_{\text{premium}}}{|\beta_{\text{price}}|} = \frac{1.20}{0.15} = \mathbf{\$8 \text{ per month}}$$

Customers would, on average, pay up to \$8 more per month for premium content over standard. Since Product B charges \$5 more for premium, it is priced below the WTP — leaving consumer surplus.

**Part C: Market Simulation**

Computing utilities:

$$V_1 = (-0.15)(13) + 0 + 0 = -1.95$$

$$V_2 = (-0.15)(18) + 1.20 + 0 = -2.70 + 1.20 = -1.50$$

$$V_3 = (-0.15)(8) + 0 + (-2.00) = -1.20 - 2.00 = -3.20$$

Exponentiated utilities:

$$e^{-1.95} \approx 0.1423, \quad e^{-1.50} \approx 0.2231, \quad e^{-3.20} \approx 0.0408$$

$$\text{Sum} = 0.1423 + 0.2231 + 0.0408 = 0.4062$$

Market shares:

$$P_1 = \frac{0.1423}{0.4062} \approx \mathbf{35.0\%}, \quad P_2 = \frac{0.2231}{0.4062} \approx \mathbf{54.9\%}, \quad P_3 = \frac{0.0408}{0.4062} \approx \mathbf{10.1\%}$$

**Interpretation:** Product 2 (premium, no ads, \$18) dominates with 54.9% market share, capturing the segment that values a clean viewing experience highly. Product 3 (cheapest but full ads) captures only 10% — strong ad aversion makes the price advantage insufficient to compensate. Product 1 is squeezed in the middle.

**Business implication:** The company currently running Product 1 should seriously consider switching to Product 2 pricing — the data shows most consumers prefer premium content enough to pay \$5 more per month, and the premium position is strongly differentiated from the cheap-but-ads option.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**Part-worth utilities:** Sign and magnitude. Negative price coefficient is always expected. For other attributes, positive coefficients mean customers prefer that level. Larger absolute values mean stronger preferences.

**Standard errors and statistical significance:** Each coefficient comes with a standard error. If the standard error is large relative to the coefficient (roughly: |coefficient| < 2 × SE), the estimate is uncertain. Do not over-interpret small, statistically insignificant part-worths.

**WTP values:** Always interpret relative to the price range in the study. If products were priced \$5–\$25 and WTP for premium = \$8, this is a meaningful premium. If WTP = \$0.50, premium content has negligible commercial value.

**Market simulation:** Remember this assumes all consumers have identical preferences (average part-worths). Real customers are heterogeneous. The simulation predicts aggregate shares; it does not say every customer prefers Product 2. Mixed logit (not covered here) handles heterogeneity but requires more data.

**IIA check:** If two products in the simulation are very similar (e.g., almost identical features, slightly different prices), the logit IIA assumption may distort shares. The agent may flag this, or you can check by examining whether the ratio of their shares seems reasonable.

**Before trusting the agent output:**
1. Is the price coefficient negative?
2. Are part-worths for desirable features (no ads, premium content) positive?
3. Do WTP estimates fall within a plausible range?
4. Do market shares sum to 100%?

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/conjoint_data.csv` + `homework_09.ipynb`

---

#### Part A: Math Questions (no agent required)

Use these utility parameters for all calculations:
- $\beta_{\text{price}} = -0.20$ per dollar
- $\beta_{\text{premium content}} = +1.50$ (vs. standard)
- $\beta_{\text{full ads}} = -1.80$ (vs. no ads)
- $\beta_{\text{limited ads}} = -0.60$ (vs. no ads)

**Q1.** Compute $V_A$ for the product: \$10/month, standard content, no ads.
```python
q1_Va = None
```

**Q2.** Compute $V_B$ for the product: \$15/month, premium content, no ads.
```python
q2_Vb = None
```

**Q3.** Compute $e^{V_A}$. Use the approximation $e^{-2.00} \approx 0.1353$.
```python
q3_exp_Va = None
```

**Q4.** Compute $e^{V_B}$. Use $e^{-1.50} \approx 0.2231$.
```python
q4_exp_Vb = None
```

**Q5.** What is $P(\text{choose B})$ in a choice between only A and B?
```python
q5_prob_B = None
```

**Q6.** Compute the WTP for "premium content."
```python
q6_wtp_premium = None
```

**Q7.** Now a third option C is added: \$8/month, standard content, limited ads. Compute $V_C$.
```python
q7_Vc = None
```

**Q8.** With $e^{-2.20} \approx 0.1108$, compute $P(\text{choose A})$ from the 3-option set {A, B, C}.
```python
q8_prob_A_3opt = None
```

**Q9.** Compare $P(\text{choose A})$ in the 2-option set (Q5, complement = 1 − 0.6226 = 0.3774) vs. the 3-option set (Q8, 0.2884). Did adding option C take share from A, B, or both?
```python
# Enter "A_only", "B_only", or "both"
q9_share_loss = None
```

**Q10.** True or False: The WTP formula $\beta_{\text{attr}} / |\beta_{\text{price}}|$ requires that price must have a negative coefficient.
```python
q10_wtp_requires_neg_price = None  # True or False
```

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I am analyzing a conjoint survey for a streaming subscription service.

conjoint_data.csv has one row per choice task:
- respondent_id: unique respondent identifier
- task_id: choice task number (each respondent sees 8 tasks)
- chosen: 1 if this option was chosen, 0 otherwise
- price: monthly subscription price ($8, $13, or $18)
- content: "standard" or "premium"
- ads: "none", "limited", or "full"

Please:
1. Prepare the data in long format: each row is one option in one task.
2. Fit a Multinomial Logit (MNLogit) model using statsmodels or pylogit.
   Use dummy coding with "standard" and "none" (no ads) as reference levels.
3. Print the full model summary with coefficients, standard errors, p-values.
4. Compute WTP for premium content and for "full ads" (how much would
   customers need to be compensated to accept full ads).
5. Simulate market shares for three products:
   - Tier 1: $10/month, standard, no ads
   - Tier 2: $16/month, premium, no ads
   - Tier 3: $7/month, standard, full ads
6. Identify the attribute with the highest WTP magnitude.
Use random seed 42.
```

**Q11.** What is the estimated price coefficient? Round to 3 decimal places.
```python
q11_beta_price = None
```

**Q12.** What is the WTP for premium content? Round to 2 decimal places (in dollars per month).
```python
q12_wtp_premium = None
```

**Q13.** What is the simulated market share for Tier 2 (premium, \$16)? Round to 1 decimal place (as a %).
```python
q13_tier2_share = None
```

**Q14.** Is the price coefficient statistically significant at the 5% level?
```python
q14_price_significant = None  # True or False
```

---

#### Part C: Interpretation Questions

**Q15.** The WTP for premium content is \$7.50. Currently, Tier 1 (standard) is priced at \$10 and Tier 2 (premium) is priced at \$16. The price gap is \$6. What does the WTP tell you about this pricing?
- (a) The premium tier is overpriced — customers would only pay \$7.50 more, not \$6
- (b) Wait, \$6 < \$7.50, so the premium tier is underpriced — customers would pay up to \$7.50 extra but only need to pay \$6 extra
- (c) The WTP and the price gap are essentially the same
- (d) WTP cannot be compared to actual prices
```python
q15 = None  # "a", "b", "c", or "d"
```

**Q16.** A marketing analyst says: "The WTP estimate of \$7.50 means every customer will pay \$7.50 more for premium." Is this correct?
- (a) Yes — WTP is the price every customer will pay
- (b) No — WTP is the average across respondents; some are willing to pay more, some less
- (c) Yes — because conjoint analysis measures exact reservation prices
- (d) No — WTP in conjoint is always understated due to hypothetical bias
```python
q16 = None  # "a", "b", "c", or "d"
```

**Q17.** The market simulation shows Tier 3 (cheapest, full ads) capturing only 8% of the market despite being the cheapest option. What does this tell you about the customer segment in this survey?
- (a) The survey has a sampling bias toward high-income respondents
- (b) Ad aversion is very strong in this sample — the disutility from full ads outweighs the price savings
- (c) The logit model is always biased against low-price options
- (d) The price range used in the survey is too narrow
```python
q17 = None  # "a", "b", "c", or "d"
```

---

### Common Misconceptions

**1. "Conjoint analysis reveals true willingness-to-pay."**
Conjoint WTP is a statistical estimate from hypothetical choices. Real purchase behavior may differ due to hypothetical bias (people claim to value things more than they actually do when spending real money). Conjoint WTP should be treated as an upper bound and validated against revealed-preference data where possible.

**2. "A higher part-worth means more customers prefer that level."**
Part-worth utility is an average across all respondents. It does not mean all customers prefer the higher-utility level — it means the average customer does. With customer heterogeneity, some segments may strongly prefer the lower-utility option.

**3. "The IIA assumption is always a problem for conjoint."**
IIA is a concern only when close substitutes are included in the same choice set. For product configurations that are genuinely distinct (no ads vs. full ads), IIA is usually reasonable.

**4. "Market simulation predicts actual sales."**
Conjoint market simulation predicts relative choice shares among the specific alternatives presented. It does not predict absolute sales volume (which depends on market size, awareness, distribution, and many other factors outside the model).

**5. "A negative coefficient for a feature means customers dislike that feature."**
A negative coefficient means customers prefer the reference level over the coded level. Whether this is a dislike depends on what the reference level is. If "no ads" is the reference, a negative coefficient for "full ads" means customers prefer no ads — expected and correct.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Fits MNLogit via MLE | Section 1.1C — maximizing log-likelihood |
| Reports β coefficients + standard errors | Section 1.3 — part-worth utilities |
| Computes WTP = β_attr / |β_price| | Section 1.4A — indifference derivation |
| Runs market simulation using softmax | Section 1.4B — softmax formula |
| Reports significance (p-values) | Section 1.1C — MLE uncertainty |

**What to verify before trusting the output:**
1. Price coefficient is negative
2. Desirable features (no ads, premium) have positive coefficients
3. WTP is in a plausible range relative to actual prices
4. Market shares sum to 100%
5. Price coefficient is statistically significant (p < 0.05)
