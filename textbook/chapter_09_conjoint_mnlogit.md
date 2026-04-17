# Chapter 9: Conjoint Analysis and Willingness-to-Pay
## Comprehensive Reference Guide

---

### Chapter Overview

Conjoint analysis measures consumer preferences by observing choices between fully
described product alternatives rather than asking direct price questions. This chapter
derives random utility theory from first principles, derives the multinomial logit
model from the Gumbel error assumption, derives WTP from the indifference condition,
develops market simulation via the softmax function, and proves the IIA property and
its limitations.

**By the end of this chapter you should be able to:**
- Explain why stated preference surveys overstate WTP and how conjoint avoids this
- Derive the MNL model from random utility theory
- Derive the WTP formula from the utility indifference condition (2 lines of algebra)
- Compute market shares for a set of products using the softmax formula
- State the IIA property, prove it algebraically, and identify when it fails
- Interpret attribute importance rankings and their relationship to WTP

---

## 9.1 Strategic Frame: What Practitioners Know

### Madhavan Ramanujam: The 20%/80% WTP Rule

Madhavan Ramanujam is a senior partner at Simon-Kucher & Partners and co-author of
*Monetizing Innovation*. He has advised over 250 companies on pricing strategy,
including LinkedIn, Uber, DoorDash, Asana, and dozens of SaaS and consumer businesses.

His most provocative finding from this work: 20% of what product teams build drives
80% of customer willingness-to-pay. The irony he consistently observes: teams
systematically build the wrong 20%.

The mechanism he describes: traditional product discovery relies on customer interviews
and feature request lists. Customers are good at articulating features they want to
have — features that improve their experience, reduce friction, or give them a sense
of completeness. They are poor at predicting which features they would actually pay
more money for. These are different sets.

Conjoint analysis forces respondents into real trade-offs between feature bundles and
price points. When a respondent must choose between "Plan A: standard content, no ads,
$12/month" and "Plan B: premium content, limited ads, $18/month," they reveal their
actual preference structure under simulated cost. They cannot overstate WTP without
making a worse choice.

His specific story about AI product pricing: companies that launched AI products at
$20/month in 2023 anchored their entire customer base at a low price point without
understanding what customers would actually pay. "The winners in AI will need to master
monetization from day one," he argues, because price anchors are difficult to raise
without customer backlash. Conjoint research at launch would have revealed the true
WTP landscape before the anchor was set.

### Naomi Ionita: Evernote and the Missing Product

Naomi Ionita led growth and monetization at Evernote and is now a partner at Menlo
Ventures. Her retrospective on Evernote's pricing failure is the clearest articulation
of how missing WTP research leads to wrong product decisions, not just wrong prices.

Evernote spent years as a beloved single-player productivity tool. Its freemium model
converted a steady fraction of users to paid. But the product was "philosophically
antisocial" — built from the ground up for individual note-taking in an era when every
successful consumer product was becoming multiplayer and collaborative.

Had Evernote run conjoint studies segmented by individual vs. team use cases, they
would have discovered that teams using Evernote informally were willing to pay
enterprise-tier prices for a proper collaboration product. The team-collaboration
feature set would have shown dramatically higher WTP than any individual productivity
feature. The product roadmap would have shifted toward multiplayer years earlier.

Her actionable rule: "Think about your pricing just like you do your product roadmap.
Every 6 to 12 months, there's probably something meaningful you're launching for users
that justifies a pricing conversation." WTP is not a one-time measurement — it
changes as the product evolves.

### Three Strategic Insights

**Insight 1: Conjoint WTP is an upper bound — not a price.**

Average WTP from conjoint represents the price increment at which the average
respondent is indifferent. Half the respondents have WTP above this and half below.
Setting a price at exactly the average WTP will lose approximately half the customers
who currently pay below that price for the existing product.

In B2C, realized prices are typically 30–50% below average conjoint WTP. In B2B,
where respondents are more deliberate, the gap is smaller. Use conjoint to rank
features by their relative WTP (attribute importance) and to set price range boundaries,
not to determine exact price points.

**Insight 2: Attribute importance ranking is more actionable than individual WTP numbers.**

Knowing that WTP for ad-free is $11 and for premium content is $7 is useful. Knowing
that ad experience drives 2.4× as much choice behavior as content quality — which
the relative magnitude of the part-worth utilities reveals — shapes product strategy
more profoundly. It tells you: if forced to choose between investing in content or
reducing ads, invest in reducing ads.

**Insight 3: Market simulation is more useful than single-feature WTP for pricing decisions.**

The question that actually matters is not "what is WTP for premium content?" It is
"if we add premium content to a tier at $16 and the ad-supported tier costs $10,
what fraction of ad-supported customers upgrade, and what fraction of competitor
customers do we acquire?" The logit market simulation in Section 9.5 answers this
question directly from the estimated utility coefficients.

---

## 9.2 Mathematical Prerequisites

### 9.2.1 Exponential Function

$$e^x = \sum_{k=0}^{\infty} \frac{x^k}{k!} = 1 + x + \frac{x^2}{2} + \frac{x^3}{6} + \cdots$$

Key properties:
- $e^0 = 1$; $e^1 \approx 2.718$
- $e^{a+b} = e^a \cdot e^b$
- $e^{-a} = 1/e^a$
- $\ln(e^x) = x$ (natural log is the inverse)

### 9.2.2 The Gumbel Distribution

The **Gumbel distribution** (Type I extreme value distribution) has CDF:
$$F(x; \mu, \beta) = e^{-e^{-(x-\mu)/\beta}}$$

Standard Gumbel (μ=0, β=1): $F(x) = e^{-e^{-x}}$

**Key property:** If $\varepsilon_1, \ldots, \varepsilon_J$ are i.i.d. standard Gumbel,
then $\max_j(\varepsilon_j + V_j)$ has a Gumbel distribution, and:
$$P(\text{argmax}_j(V_j + \varepsilon_j) = k) = \frac{e^{V_k}}{\sum_j e^{V_j}}$$

This is the derivation of the softmax formula from the Gumbel error assumption.

*Reference: Wikipedia, "Gumbel distribution" — en.wikipedia.org/wiki/Gumbel_distribution*
*Reference: Wikipedia, "Discrete choice" — en.wikipedia.org/wiki/Discrete_choice*

---

## 9.3 Why Conjoint Beats Direct Price Surveys

### The Hypothetical Bias Problem

When asked "How much would you pay for an ad-free subscription?", respondents
systematically overstate their willingness to pay. Two mechanisms:

**1. Social desirability bias:** Respondents feel they should value features highly.
Admitting they would only pay $2 more for ad-free feels like admitting they don't
value their own time. They say $8.

**2. Hypothetical bias:** There is no real cost to stating a high number. In a real
purchase, spending $8 more means $8 less for something else. In a survey, it costs
nothing. Research consistently shows stated WTP exceeds revealed WTP by 30–100%.

### How Conjoint Avoids This

In choice-based conjoint, respondents choose between complete product alternatives:
"Plan A ($12, standard content, no ads) or Plan B ($18, premium content, limited ads)?"

Choosing Plan B costs the respondent $6/month in the hypothetical. The choice reveals
that the combination of (premium content + limited ads) is worth at least $6/month
more than (standard content + no ads). The price sensitivity is revealed through
observed trade-offs, not stated preferences.

**The WTP estimate is inferred from revealed choices, not stated preferences.** This
is the source of conjoint's superiority over direct price surveys.

---

## 9.4 Random Utility Theory and the MNL Model

### Random Utility Model

Consumer i facing choice set C chooses alternative j to maximize utility:
$$U_{ij} = V_{ij} + \varepsilon_{ij}$$

where $V_{ij} = \beta^\top x_{ij}$ is the **systematic utility** (observable, linear
in attributes and price) and $\varepsilon_{ij}$ is the **random component**
(unobserved factors specific to individual i's valuation of alternative j).

The consumer chooses j iff $U_{ij} > U_{ik}$ for all $k \neq j$:
$$P(\text{choose } j \mid C) = P(V_{ij} + \varepsilon_{ij} > V_{ik} + \varepsilon_{ik} \, \forall k \neq j)$$

### Derivation of Multinomial Logit from Gumbel Errors

**Assumption:** $\varepsilon_{ij}$ are i.i.d. standard Gumbel for all i, j.

Recall: if ε_{i1}, ..., ε_{iJ} are i.i.d. standard Gumbel, then:
$$P(\text{choose } j) = P(V_{ij} + \varepsilon_{ij} = \max_k(V_{ik} + \varepsilon_{ik})) = \frac{e^{V_{ij}}}{\sum_{k \in C} e^{V_{ik}}}$$

This is the **multinomial logit (MNL) choice probability**. The softmax formula
emerges directly from the Gumbel error assumption.

**Full derivation of the 2-alternative case:**

For J=2, alternatives A and B:
$$P(\text{choose B}) = P(V_{iB} + \varepsilon_{iB} > V_{iA} + \varepsilon_{iA})$$
$$= P(\varepsilon_{iA} - \varepsilon_{iB} < V_{iB} - V_{iA})$$

The difference of two i.i.d. standard Gumbels has a logistic distribution:
$$P(\varepsilon_{iA} - \varepsilon_{iB} \leq x) = \frac{e^x}{1 + e^x}$$

Therefore:
$$P(\text{choose B}) = \frac{e^{V_{iB} - V_{iA}}}{1 + e^{V_{iB} - V_{iA}}} = \frac{e^{V_{iB}}}{e^{V_{iA}} + e^{V_{iB}}}$$

which is the 2-alternative logit formula. The J-alternative generalization:
$$P(\text{choose } j) = \frac{e^{V_{ij}}}{\sum_{k=1}^J e^{V_{ik}}}$$

*Reference: McFadden, D. (1974). "Conditional Logit Analysis of Qualitative Choice
Behavior." In Zarembka (ed.), Frontiers in Econometrics. Summary at
nobelprize.org/prizes/economic-sciences/2000/mcfadden/lecture/*
*Reference: Wikipedia, "Multinomial logistic regression" —
en.wikipedia.org/wiki/Multinomial_logistic_regression*
*Reference: Train, K. E. (2009). Discrete Choice Methods with Simulation, 2nd ed.
Free PDF: eml.berkeley.edu/books/choice2.html*

---

## 9.5 WTP Derivation: Full Algebraic Proof

### Setup

Systematic utility for a product with attribute A present (+1) vs. absent (0):
$$V_{\text{with A}} = \beta_{\text{price}} P + \beta_A \cdot 1 + \text{other terms}$$
$$V_{\text{without A}} = \beta_{\text{price}} P_0 + \beta_A \cdot 0 + \text{other terms}$$

**WTP definition:** The price increment $\Delta P$ such that the consumer is
indifferent between the product with attribute A at price $P + \Delta P$ and the
product without attribute A at price $P_0$:

$$V_{\text{with A at } P + \Delta P} = V_{\text{without A at } P_0}$$

### Derivation

Setting systematic utilities equal (assuming other terms cancel):
$$\beta_{\text{price}}(P + \Delta P) + \beta_A = \beta_{\text{price}} P_0$$

For comparison at the same base price P = P_0:
$$\beta_{\text{price}} P + \beta_{\text{price}} \cdot \text{WTP} + \beta_A = \beta_{\text{price}} P$$

$$\beta_{\text{price}} \cdot \text{WTP} + \beta_A = 0$$

$$\text{WTP} = -\frac{\beta_A}{\beta_{\text{price}}} = \frac{\beta_A}{|\beta_{\text{price}}|}$$

(Since $\beta_{\text{price}} < 0$, we have $-\beta_A/\beta_{\text{price}} = \beta_A/|\beta_{\text{price}}| > 0$
for desirable attributes with $\beta_A > 0$.)

> **Teaching note (board):** Set up the indifference condition in words first:
> "WTP is the extra price at which I'm equally happy with the premium vs. basic version."
> Write V_premium = V_basic. Expand. Cancel. Two lines of algebra give the formula.
> Students who see the setup never need to memorize the formula — they can re-derive
> it from the definition in 30 seconds. Budget 8 minutes.

### Worked Example

Coefficients: β_price = −0.18, β_premium_content = +1.26, β_no_ads = +2.00.

$$\text{WTP}_{\text{premium}} = \frac{1.26}{0.18} = \$7.00/\text{month}$$

$$\text{WTP}_{\text{no\_ads}} = \frac{2.00}{0.18} = \$11.11/\text{month}$$

**Attribute importance:** |β_no_ads| = 2.00 > |β_premium| = 1.26, so no-ads
drives more choice behavior than premium content. If forced to invest in one, invest
in eliminating ads.

---

## 9.6 Market Simulation

### Setup

Three competing products. Compute utility for each, apply softmax to get shares.

| Product | Attributes | Utility V |
|---|---|---|
| A: std, full_ads, monthly, $12 | β_price×12 = −2.16 | V_A = **−2.16** |
| B: prem, no_ads, monthly, $18 | −0.18(18)+1.26+2.00 | V_B = −3.24+3.26 = **+0.02** |
| C: std, ltd_ads, annual, $15 | −0.18(15)+0+0.80+0.65 | V_C = −2.70+1.45 = **−1.25** |

$$e^{V_A} = e^{-2.16} = 0.115, \quad e^{V_B} = e^{0.02} = 1.020, \quad e^{V_C} = e^{-1.25} = 0.287$$

$$\text{Sum} = 0.115 + 1.020 + 0.287 = 1.422$$

| Product | Share |
|---|---|
| A | 0.115/1.422 = **8.1%** |
| B | 1.020/1.422 = **71.7%** |
| C | 0.287/1.422 = **20.2%** |

**What this tells us:** Product B (premium, no ads, $18) dominates despite being
the most expensive because its attribute benefits outweigh the price premium. This
is exactly what WTP analysis predicted: combined no-ads + premium WTP is $18.11 >> $6
premium over base product at $12.

---

## 9.7 IIA Property: Proof and Limitations

### Proof of IIA

For any two alternatives i and k in the MNL model:
$$\frac{P(\text{choose } i)}{P(\text{choose } k)} = \frac{e^{V_i}/\sum_j e^{V_j}}{e^{V_k}/\sum_j e^{V_j}} = \frac{e^{V_i}}{e^{V_k}} = e^{V_i - V_k}$$

This ratio depends only on V_i and V_k — not on the utilities of any other
alternative in the choice set. Therefore: adding, removing, or changing any third
alternative j does not change the ratio of choice probabilities for i and k.

This is the **Independence of Irrelevant Alternatives (IIA)** property.

### Why IIA Follows From the Gumbel Assumption

IIA emerges directly because the i.i.d. Gumbel errors are independent across
alternatives. If consumer i values A vs. B independently from how they value C,
the choice ratio A:B is unaffected by C's presence. But if A and B are similar
products (say, two versions of the same brand) and C is a different brand, consumers
who are neutral between A and B might strongly prefer either to C — introducing
correlation between ε_A and ε_B that the i.i.d. Gumbel model cannot capture.

### The Blue Bus / Red Bus Problem

Classic IIA failure: suppose a market has car and blue bus, each with 50% share.
Introduce an identical red bus. IIA predicts: car 33%, blue bus 33%, red bus 33%.

Reality: red bus and blue bus are perfect substitutes. Sensible prediction: car 50%,
blue bus 25%, red bus 25% (buses split the bus share).

IIA cannot model this because it treats red bus as equally similar to car as to blue bus.

### Solutions to IIA Violations

**1. Nested logit:** Group similar alternatives into nests. IIA holds within nests
(e.g., both buses in the same nest) but not across nests (bus nest vs. car).

**2. Mixed logit (random parameters):** Allow β to vary across consumers with a
distribution (e.g., β_price ~ Normal(μ, σ²)). Marginalized over the distribution,
the choice probabilities do not satisfy IIA.

**3. Multinomial probit:** Use multivariate Normal errors allowing arbitrary
correlation across alternatives. IIA-free but computationally expensive.

*Reference: Wikipedia, "Independence of irrelevant alternatives" —
en.wikipedia.org/wiki/Independence_of_irrelevant_alternatives*
*Reference: Wikipedia, "Nested logit" — en.wikipedia.org/wiki/Nested_logit*

---

## 9.8 MNL Estimation: Maximum Likelihood

### Log-Likelihood

For respondent n in choice task t, choosing alternative j_nt from J alternatives:
$$\log L = \sum_n \sum_t \log P(y_{nt} = j_{nt}) = \sum_n \sum_t \left[V_{nj_{nt}} - \log \sum_{k=1}^J e^{V_{nk}}\right]$$

This is the **cross-entropy loss** from machine learning applied to multinomial outcomes.

Maximized by gradient descent or Newton-Raphson. In Python (statsmodels):
```python
import statsmodels.discrete.discrete_model as dm
model = dm.MNLogit(y, X)
result = model.fit(method='newton')
```

### Standard Errors and Significance

The Hessian of the log-likelihood gives the information matrix:
$$\text{Var}(\hat{\beta}) \approx [-\nabla^2 \log L(\hat{\beta})]^{-1}$$

Standard errors from this matrix enable t-tests for each coefficient.

---

## 9.9 Full Proof Appendix

### Proof: Multinomial Logit Maximizes Entropy Subject to Moment Constraints

The maximum entropy distribution on J alternatives, subject to E[V_j] = c_j for
each j, is:
$$P(j) \propto e^{V_j / T}$$

where T is a temperature parameter. For T=1, this is the standard softmax. The
logit model is therefore the maximum-entropy distribution given the observed
expected utility constraints — it makes the weakest possible assumptions about
unmodeled heterogeneity consistent with the data.

*Reference: Wikipedia, "Maximum entropy probability distribution" —
en.wikipedia.org/wiki/Maximum_entropy_probability_distribution*

### Proof: WTP Formula Gives the Exact Compensating Variation

In welfare economics, **compensating variation** is the amount of money that
exactly compensates a consumer for a change in product attributes. For the utility
function $U = \beta_p P + \beta_A A$, the compensating variation for removing
attribute A is found by solving U(P, 1) = U(P + CV, 0):

$$\beta_p P + \beta_A = \beta_p(P + CV)$$
$$\beta_A = \beta_p \cdot CV$$
$$CV = \beta_A / \beta_p = \beta_A / |\beta_p| = \text{WTP}$$

The WTP formula therefore gives exact compensating variation for the MNL utility
specification — it is not an approximation.

---

## 9.10 Verification Checklist

| Check | How to verify |
|---|---|
| WTP = β_attribute / β_price? | Denominator is β_price, not |β_price| separately — be careful with signs |
| β_price < 0? | Required for the indifference condition to yield positive WTP |
| Softmax shares sum to 100%? | By construction |
| Log-softmax = V_j − log(Σe^{V_k})? | Alternative form for numerical stability |
| IIA: new product draws proportionally from all others? | Not just from similar products |
| WTP is average, not contract price? | Individual WTP varies; setting price at mean WTP loses ~50% |
| Attribute importance ≠ WTP magnitude alone? | Also consider attribute variance in the sample |

---

## 9.11 Chapter Bibliography

1. Wikipedia, "Conjoint analysis" — en.wikipedia.org/wiki/Conjoint_analysis
2. Wikipedia, "Multinomial logistic regression" — en.wikipedia.org/wiki/Multinomial_logistic_regression
3. Wikipedia, "Independence of irrelevant alternatives" — en.wikipedia.org/wiki/Independence_of_irrelevant_alternatives
4. Wikipedia, "Discrete choice" — en.wikipedia.org/wiki/Discrete_choice
5. Wikipedia, "Gumbel distribution" — en.wikipedia.org/wiki/Gumbel_distribution
6. McFadden, D. (2000). Nobel Prize lecture: "Economic Choices." Free: nobelprize.org/prizes/economic-sciences/2000/mcfadden/lecture/
7. Train, K. E. (2009). Discrete Choice Methods with Simulation, 2nd ed. Free PDF: eml.berkeley.edu/books/choice2.html
8. Orme, B. K. "Getting Started with Conjoint Analysis." Sawtooth Software technical papers. Free: sawtoothsoftware.com/resources/technical-papers
9. Ramanujam, M. and Tacke, G. (2016). Monetizing Innovation. Wiley. (Non-free, but widely available; relevant for practitioner framing.)
10. arXiv:1806.00882 (see Train 2009 for equivalent treatment of mixed logit)
11. MIT OpenCourseWare, 14.121 Microeconomic Theory I — ocw.mit.edu (for utility theory foundations)

**Lenny's Podcast episode references:**
- Madhavan Ramanujam episodes on pricing strategy and WTP — Lenny's Podcast
- Naomi Ionita episode on monetization and product strategy — Lenny's Podcast
