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
- Describe how a choice experiment is designed and fielded
- Find the profit-maximizing price by running the market simulation at many prices

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
>
> Sum = 0.223 + 1.649 + 0.449 = 2.321
>
> P(choose product 2) = 1.649/2.321 = **0.71** (71% market share)

---

#### Tool 2: Ratio of Coefficients

Willingness-to-pay for a feature = (feature coefficient) / |price coefficient|

> WTP = β_feature / |β_price|

This is derived by asking: how much higher a price makes the premium version exactly as attractive as the standard version? Start from two versions that are identical except for the feature, at the same price. The premium version is ahead by β_feature in utility. Now raise its price by Δ dollars: its lead becomes β_feature + β_price × Δ. The customer is indifferent when that lead is zero:

> β_feature + β_price × Δ = 0  ⇒  Δ = β_feature / (−β_price) = β_feature / |β_price|

(β_price is negative, so −β_price = |β_price|.) That Δ is the WTP: the largest price premium at which the average customer still weakly prefers the premium version.

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

Conjoint analysis reduces this by never asking about price directly. Instead, respondents choose between complete product configurations: "Would you prefer Plan A ($12, standard content, no ads) or Plan B ($18, premium content, limited ads)?" The price sensitivity is revealed through the trade-offs they make, not through a direct question.

#### How a Choice Experiment Is Run

A choice experiment is a survey in which each respondent answers several choice tasks. Each task shows a few product profiles, and the respondent picks one. The analyst designs the profiles in five steps.

1. **Choose the attributes and their levels.** List the features the decision is about, plus price. The HW09 survey uses four attributes: price (\$10, \$14, \$18, \$22), content (standard, premium), ads (full, limited, none) and plan (monthly, annual). Set the price levels inside the range the firm would consider charging. The fitted model describes preferences only at the prices the survey showed.
2. **Choose the size of the survey.** HW09 shows each of 400 respondents 8 tasks with 3 profiles each, which gives 9,600 rows. Each additional task adds data and also adds fatigue, and a tired respondent stops reading the profiles.
3. **Draw the levels at random.** For each profile, draw the level of every attribute at random and independently of the others. Price is then uncorrelated with the features, and the model can estimate the effect of each attribute separately. Redraw any task in which two profiles are identical (about 7% of tasks in the design below).
4. **Decide whether to offer "none of these."** A "none" option lets a respondent leave the market when every profile is too expensive. It is needed to set prices for a whole product line (Section 1.4D). If you include it, record it as its own row in each task, and the model gets one more coefficient: a constant for "none".
5. **Field the survey and save the responses in long format.** Survey platforms such as Qualtrics, Sawtooth Software and Conjointly generate and field the tasks for you. The result is the file format used in the homework: one row per profile shown, with `chosen = 1` for the profile picked and 0 for the others.

The figure follows one real task from the HW09 data (respondent 1, task 1) from the survey screen to the rows of the data file, and then to the model and the decisions it supports.

![Diagram in five panels. Top left, DESIGN, Levels to test: price \$10, \$14, \$18, \$22; content standard or premium; ads full, limited or none; plan monthly or annual; each profile has every level drawn at random. An arrow leads to the top middle panel, SURVEY, What the respondent sees: "Task 1 of 8: which plan would you choose?" above three plan cards. Plan 1: \$18 per month, premium content, full ads, monthly. Plan 2: \$10 per month, standard content, no ads, annual, highlighted and marked picked. Plan 3: \$18 per month, standard content, full ads, annual. An arrow leads to the top right panel, DATA, One row per plan shown: a three-row table with columns resp, task, alt, price, content, ads, plan and chosen; the rows are respondent 1, task 1, alternatives 1 to 3, with chosen equal to 0, 1 and 0, and the chosen row highlighted. Below the table: times 8 tasks times 400 respondents equals 9,600 rows; chosen = 1 marks the plan picked. An arrow leads down to the bottom right panel, MODEL, Fit the MNL: ConditionalLogit with one group per respondent and task, giving β_price and one part-worth per level. An arrow leads left to the bottom left panel, DECISIONS, Use the coefficients: WTP equals β_feature divided by the absolute value of β_price, market shares from the softmax, and the profit-maximizing price from a price search.](figures/lecture_09_conjoint_pipeline.png)

A random design of the HW09 size takes a few lines of Python; the `chosen` column is added once responses come back:

```python
import numpy as np, pandas as pd
rng = np.random.default_rng(42)
levels = {"price": [10, 14, 18, 22], "content": ["standard", "premium"],
          "ads": ["full_ads", "limited_ads", "no_ads"], "plan": ["monthly", "annual"]}
rows = [{"respondent_id": r, "task_id": t, "alternative": a,
         **{attr: rng.choice(lv) for attr, lv in levels.items()}}
        for r in range(1, 401) for t in range(1, 9) for a in range(1, 4)]
design = pd.DataFrame(rows)   # 9,600 rows; redraw tasks with identical profiles
```

Specialized design software produces more efficient designs, which estimate the same coefficients with fewer respondents. For a survey of HW09's size a random design is sufficient: in the HW09 data, every coefficient's standard error is less than a tenth of the coefficient.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: Utility Model

Each product is characterized by attributes (price, content tier, ad level, plan type). The utility of a product is a linear combination:

$$U = \beta_{\text{price}} \times \text{price} + \beta_{\text{premium}} \times \mathbb{1}[\text{premium}] + \beta_{\text{no_ads}} \times \mathbb{1}[\text{no_ads}] + ...$$

where 𝟙[...] = 1 if the attribute level is present, 0 otherwise.

**Example coefficients:**
- β_price = −0.18 (per $1 increase in monthly price)
- β_premium_content = +1.26 (vs. standard content)
- β_no_ads = +2.00 (vs. full ads)
- β_limited_ads = +0.80 (vs. full ads — used in the market simulation below)
- β_annual_plan = +0.65 (vs. monthly)

All ad coefficients are measured against **full ads** as the reference level, so full ads contributes 0.

**WTP calculation:**
> WTP for premium content = 1.26 / 0.18 = **$7.00/month**
>
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
>
> Shares: A = 8.1%, B = 71.7%, C = 20.1%

---

#### Part C: The IIA Property and Its Limitation

The multinomial logit model assumes IIA: **Independence of Irrelevant Alternatives**. Adding a new product draws share from all existing products proportionally to their current shares — regardless of how similar the new product is to existing ones.

**Why this is often wrong:** If you add Plan C' (identical to Plan C, but $1 cheaper, so U_C' = −1.07), IIA predicts it draws from A, B **and C** in proportion to their current shares. Every plan loses the same fraction of its share (19.4%): the new shares are A 6.5%, B 57.8%, C 16.2%, C' 19.4%. So IIA has C' taking 13.9 points from B (premium, no ads) and only 3.9 from C, its near-twin. In reality, it would draw mostly from C (the similar product). IIA cannot model this substitution pattern.

**Practical implication:** Market simulation results are more reliable when the products being compared are genuinely different. When testing very similar variants, IIA leads to overestimation of share for the new variant and underestimation of cannibalization from the most similar existing product.

The standard fix is **nested logit**: the analyst groups close substitutes into a "nest" (here C and C'), so a new product draws more heavily from its own nest than from products outside it. It is not covered in this course; know that it exists and what problem it solves.

> ### 🔍 Deep Dive: Why Logit Implies IIA
> The multinomial logit model arises from the assumption that unobserved utility components follow an independent Gumbel distribution. Under this assumption, P(choose i) = e^(U_i) / Σⱼ e^(U_j). For any two alternatives i and k: P(i)/P(k) = e^(U_i − U_k). This ratio depends only on i and k, not on any other alternative in the choice set. That is the IIA property: adding or removing any third option does not change the ratio of probabilities for any pair of options.

---

#### Part D: Setting the Price That Maximizes Profit

WTP measures what a feature is worth to the average respondent. The price to charge comes from the market simulation. Suppose the firm sells only Product B from Part B. Products A and C belong to competitors, so the firm cannot set their prices and takes them as given: A at \$12 and C at \$15. Serving one customer on B costs the firm c dollars a month. The expected profit per customer in the market is

$$\pi(p) = (p - c) \times s(p)$$

where s(p) is B's simulated share at price p. A higher price raises the margin p − c and lowers the share s(p), and the profit-maximizing price p* balances the two.

**Search over prices.** Run the Part B simulation at a range of prices for B, with A and C at their current prices. At each trial price, recompute all three shares. A's and C's utilities stay the same, and their shares rise as B's share falls: the search counts every customer who switches away from B. Keep the price with the highest profit for B. With c = \$6:

| Price of B | Share of B | Profit per customer, (p − 6) × s |
|---|---|---|
| \$15 | 81.3% | \$7.32 |
| \$18 | 71.7% | \$8.61 |
| \$21 | 59.7% | \$8.95 |
| \$24 | 46.3% | \$8.33 |
| \$27 | 33.4% | \$7.02 |

A search in 1-cent steps gives **p\* = \$20.52**, with a 61.7% share and a profit of \$8.96 per customer. Multiply by the number of customers in the market for total profit.

The figure plots the same search at every price between \$6 and \$36. Profit rises while the gain in margin exceeds the loss in share, peaks at \$20.52, and falls after that. Reading the peak off this plot is all the profit-maximizing price requires; no calculus is needed.

![Line chart of profit per customer, (p − 6) × share, on the vertical axis from \$0 to \$10, against the monthly price of Product B on the horizontal axis from \$6 to \$36, for Section 1.4B's example with β_price = −0.18, unit cost \$6, and Products A and C at fixed prices. The curve starts at \$0 at a price of \$6, rises to a single peak, and declines slowly toward the right. The peak is marked with a dot and a dashed line down to the price axis: best price \$20.52, share 61.7%, profit \$8.96. A note at the low-price end reads "low price: high share, small margin", and a note at the high-price end reads "high price: large margin, few customers".](figures/lecture_09_profit_curve.png)

**When the result is meaningful.** The result is B's best price at the competitors' current prices. If a competitor changes its price, run the search again with the new price; predicting how competitors respond is beyond this course. The search finds a finite p\* because customers can switch to products whose prices stay fixed, here A and C. If the firm also sold A, for example as its basic tier, customers who move from B to A would still pay the firm, so the profit to maximize would be the sum of both products' (price − cost) × share. If the firm owned all three products and raised every price by the same amount, the shares would not move (Slide Practice Answer P1), and the simulation would recommend raising prices without limit. To price a whole product line, the survey needs a "none of these" option (Section 1.3), so that the model observes customers leaving the market. The profit-maximizing price also has the limits of the simulation itself: it uses average part-worths (Section 2.2), it is reliable only inside the price range the survey tested, and it is too high when hypothetical choices understate respondents' price sensitivity (Common Misconception 1).

**Choosing a feature bundle.** To compare configurations, repeat the search for each configuration with its own cost c (premium content costs more to license) and choose the configuration whose profit at its own p\* is highest.

> ### 🔍 Deep Dive (not required): A Formula That Checks the Price Search
> The price search is the method you need. This box is for readers who want to see why the peak sits where it does; it uses a derivative and is not tested.
>
> Under the logit model, B's share responds to its own price at the rate ds/dp = β_price × s × (1 − s). Setting the derivative of profit (p − c) × s(p) to zero gives s + (p − c) × β_price × s × (1 − s) = 0, and therefore
>
> p\* − c = 1 / (|β_price| × (1 − s\*))
>
> Check: 1 / (0.18 × (1 − 0.617)) = \$14.51, and \$6 + \$14.51 = \$20.51, which matches the search up to rounding. The right-hand side depends on s\*, which itself depends on p\*, so the formula checks a price found by search and cannot be solved for p\* in one step. Two consequences follow from it. The markup is never below 1/|β_price| (\$5.56 here). It also rises with the product's share, because the logit price elasticity, β_price × p × (1 − s), is smaller in size when s is large. Written with the L02 elasticity, the formula says that the margin (p − c)/p equals 1/|elasticity| at the profit-maximizing price.

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

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

**Q3.** Conjoint WTP of $12 is an **average** — individual WTP varies. Some customers have WTP below $12 and would not pay $22. Raising to $22 (= $10 + $12 = current price + average WTP) would lose every customer whose individual WTP for no ads is below the $12 premium: about half of them, if individual WTP is spread symmetrically around the average. Raising to $22 may be profitable if the revenue from those who stay exceeds revenue from those who churn. The price search in Section 1.4D finds the price that maximizes profit.
*Common wrong answer:* The PM is correct — WTP is $12 so the tier should cost current price + $12 = $22. Average WTP is not the price at which everyone buys; it is the average maximum each person would pay.

**Q4.** **False.** WTP is the **maximum** price a customer would accept, not the minimum. The correct statement: WTP is the maximum price at which a customer would still choose this option over the alternative.
*Common wrong answer:* True, because WTP represents the lowest price you should charge to maintain demand.

**Q5.** Under IIA, the new product draws share from A and B proportionally to their current shares. If A currently has 11.9% and B has 88.1%, the new product takes 11.9% of its share from A and 88.1% from B — even though the new product is identical to A. Product A's share decreases by its proportion of the total.
*Common wrong answer:* All of the new product's share comes from A (since they are identical). This is the realistic prediction — but IIA gives the proportional answer, which is why IIA is often wrong in near-substitute scenarios.

#### Slide Checkpoint Answers

Numbered to match the Checkpoint slide in this lecture's deck. Where the slide asks a question above again, or the same idea on different numbers, the entry points to that answer.

**Slide Q1.** *"β_price = −0.18, β_annual_plan = +0.65. Compute WTP for annual plan."* — Same formula, run on the example coefficients in Section 1.4: WTP = 0.65 / 0.18 = **\$3.61/month**. Read it as "the average respondent values being on an annual plan rather than monthly at about \$3.61 of monthly price". Note the units: because β_price is per \$1 of *monthly* price, the WTP is also per month, not per year.

**Slide Q2.** *"Two products with utilities $U_A = -1.5$ and $U_B = +0.5$. Compute market shares for both."* — Same question as Q2 above — see that answer.

**Slide Q3.** *"You add Option D (similar to B). Under IIA, which existing option loses the most share?"* — Same reasoning as Q5 above, which adds a product like A; the slide adds Option D, like B.

**Slide Q4.** *"WTP for "no ads" = \$10. Current base price = \$8. Optimal premium tier price = \$18? Defend or critique."* — Same reasoning as Q3 above, which uses WTP = \$12, a \$10 tier and a \$22 proposal; the slide uses \$10, \$8 and \$18.

**Slide Q5.** *"$\beta_{\text{price}} = +0.05$ from the MNL fit. What went wrong — and how do you fix it?"* — Start from what a conjoint rules out. In Lecture 2 (its Slide Q4), a positive price coefficient in sales data can be a real finding: prices respond to demand (endogeneity), so high prices and high sales move together. In a conjoint, **price is randomised by design**, so that explanation is ruled out by construction. Suspect a defect first, and check in this order:

1. **Look at the standard error first.** It is the cheapest check. If the confidence interval spans zero, +0.05 is noise: the design did not identify price sensitivity (for example, prices varied too little), and the fix is the design, not the model.
2. **Check the coding.** Two different mistakes leave different fingerprints. If price was entered as a discount or a reversed index, *only* β_price flips sign; premium content and no ads still come out positive. If the `chosen` column is flipped (1 for the options *not* picked), *every* coefficient flips: premium content and no ads come out negative too. Fix: read the design matrix and confirm price rises in dollars and `chosen` marks the pick.
3. **Check the choice sets.** The model must be fitted per choice task (ConditionalLogit grouped by respondent × task), not on pooled rows. Verify one respondent's rows by hand against the raw responses.
4. **Coded correctly and still significantly positive:** respondents are likely reading price as a signal of quality, which means the profiles leave out an attribute they care about. That is a survey-design problem; add the missing attribute and re-field.

Whichever it is, do **not** compute WTP from this fit: β_feature / |β_price| assumes a negative price coefficient, so the dollar figures it produces mean nothing (Section 2.2, "Before trusting the agent output", check 1).
*Common wrong answer:* "Customers like paying more, so raise prices." With randomised prices, a positive coefficient is almost always a data or design problem, not a discovery.

### Slide Practice Answers

Numbered to match the two "Practice: work in pairs" slides in this lecture's deck.

**P1. True.** A \$2 cut raises every plan's utility by the same amount (0.15 × 2 = 0.30 at
β_price = −0.15). Softmax shares depend only on the *differences* between utilities, since
P(i)/P(k) = e^(U_i − U_k) (Section 1.4C, Deep Dive), so the split does not move. That is a limit of
the simulation, not a claim about the market: it compares only the plans in the choice set, so it
cannot say whether the cut brings in more subscribers overall (Common Misconception 4).

**P2. B, A 48% and B 32%.** Under IIA, C draws from A and B in proportion to their current shares,
so the remaining 80% keeps the 60:40 split: A = 0.60 × 80% = 48% and B = 0.40 × 80% = 32%
(Section 1.4C).
*Common wrong answer:* A, which has C taking everything from B. That is what you would expect in
reality, and it is exactly what MNL cannot produce: IIA underestimates how much a near-copy
cannibalizes the product it copies (Section 1.4C, "Practical implication").

**P3. B, \$5.33.** Coefficients are measured against the reference level, here no ads, which
contributes 0. Going from limited ads to no ads gains 0 − (−0.80) = 0.80 of utility, and
WTP = 0.80 / 0.15 = \$5.33 (Section 1.1, Tool 2; the coding in Section 2.1).
*Common wrong answers:* A, \$13.33 = 2.00 / 0.15, is the WTP to go from *full* ads to no ads. C,
\$8.00 = (2.00 − 0.80) / 0.15, is the WTP to go from full ads to limited ads. D, \$0.19, inverts the
formula.

**P4. False.** A part-worth is an average across respondents. +1.20 says the average respondent
prefers premium content; some segments may prefer standard (Common Misconception 2; Section 2.2,
"Market simulation").

**P5. B.** A direct price question invites hypothetical and social-desirability bias, so \$20 is
likely inflated. Conjoint reveals price sensitivity through trade-offs instead (Section 1.3). The
choices are still hypothetical, though, so treat \$11 as an upper bound and validate it against
revealed-preference data (Common Misconception 1).
*Common wrong answer:* A. Stating a number costs the respondent nothing, which is the problem
conjoint is designed around.

**P6. False.** Simulated shares are relative choice shares among the alternatives presented. They
say nothing about how many households buy at all, which depends on market size, awareness,
distribution and other factors outside the model (Common Misconception 4).

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

**Interpretation:** Product 2 (\$18, premium, no ads) leads with 54.9%. Its only difference from Product 1 is premium content at \$5 more, and premium content is worth \$8 (Part B), so the average respondent prefers it. Product 3 (cheapest, full ads) gets only 10%: its \$5 discount against Product 1 is worth 0.15 × 5 = 0.75 of utility, and full ads cost 2.00. Product 1 is squeezed in the middle.

**Business implication:** The simulation says premium content at \$18 beats standard content at \$13 for the average respondent. It does not say that switching Product 1 to premium would pay. Premium content costs money to license, and 54.9% is an average respondent's choice probability, not "55% of customers prefer premium" (Section 2.2). And if Product 1 became an exact copy of Product 2, MNL would treat the two twins as unrelated products, which is the near-copy case IIA gets wrong (Section 1.4C). Compare margin × share for each configuration before switching.

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
#### (~55 minutes in class | Due: 23:59 ET on the due date in the course schedule | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_09_conjoint_analysis.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_09_conjoint_analysis.ipynb` |
| Dataset | `homework_datasets/conjoint_survey.csv` |
| Graded questions | **18** — Part A: 9 · Part B: 4 · Part C: 5 |

<!-- END GENERATED homework pointer -->

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
| Fits MNLogit via MLE | Sections 1.1 and 1.4A — the softmax choice probability and the utility model being fitted (the lecture does not derive the MLE itself) |
| Reports β coefficients + standard errors | Section 1.4A — part-worth utilities; Section 2.2 — standard errors |
| Computes WTP = β_attr / \|β_price\| | Section 1.1, Tool 2 — indifference derivation; Section 1.4A — worked WTP |
| Runs market simulation using softmax | Section 1.4B — softmax formula |
| Reports significance (p-values) | Section 2.2 — standard errors and statistical significance |
| Searches prices for the highest (p − c) × share | Section 1.4D — profit-maximizing price by price search |

**What to verify before trusting the output:**
1. Price coefficient is negative
2. Desirable features (no ads, premium) have positive coefficients
3. WTP is in a plausible range relative to actual prices
4. Market shares sum to 100%
5. Price coefficient is statistically significant (p < 0.05)

**On your own data:** the homework prompt pins its products, reference levels and seed so that
it can be graded. The version for your own survey is in the
[Agentic Prompts page, section L09](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/agentic_prompts_14_models.md):
the data it needs, the package and functions, a general prompt, and the checks to run before you
trust the output. It is not quiz material.
