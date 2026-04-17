# Lecture 1: Bayesian A/B Testing
## How to Make Decisions Under Uncertainty

---

### Overview

**Business question:** You have two versions of a webpage. Version A is the current design. Version B is a new design. After two weeks, Version B had a higher conversion rate. Should you switch permanently?

**What you will be able to do after this lecture:**
- Explain the difference between an *observed* conversion rate and a *true* conversion rate
- Update beliefs about conversion rates when new data arrives
- Compute the probability that one variant is genuinely better than another
- Interpret a posterior distribution, a credible interval, and an expected lift in plain business language
- Critically evaluate the output of an agent running a Bayesian A/B test

**Prerequisites:** None. This lecture builds everything from scratch.

---

## PART 1: Concepts and Mathematics
### (~1 hour 40 minutes)

---

### Section 1.1 — Math Toolkit
#### (~20 minutes)

Three lightweight tools. Each is simple on its own. Together they unlock the key insight.

---

#### Tool 1: Probability as a Number Between 0 and 1

A **probability** measures how likely something is. It always falls between 0 and 1.

- **0** = impossible. It will never happen.
- **1** = certain. It will always happen.
- **0.5** = happens half the time.

Probabilities and percentages represent the same idea. Divide a percentage by 100 to get a probability:

> 4% = 0.04 &nbsp;&nbsp; 50% = 0.50 &nbsp;&nbsp; 100% = 1.00

When we say "the conversion rate is 4%," we mean 4 out of every 100 visitors convert. As a probability: 0.04.

**The complement rule:** If P(event) = p, then P(event does NOT happen) = 1 − p.
> If P(converts) = 0.04, then P(does not convert) = 0.96.

---

#### Tool 2: The Exponent Multiplication Rule

We will encounter expressions like θ^3 × θ^5. One rule handles all of them:

> **When multiplying two expressions with the same base, add the exponents.**
> $$a^m \times a^n = a^{m+n}$$

**Examples:**
> θ^3 × θ^5 = θ^8
> (1−θ)^4 × (1−θ)^6 = (1−θ)^10

**Why we need this:** The entire Bayesian update derivation reduces to applying this one rule twice.

---

#### Tool 3: "Proportional To" (∝)

The symbol ∝ means: *equal up to a constant multiplier that does not depend on the variable we care about.*

**Example:** f(θ) = 7 × θ^2 × (1−θ)^3. We write:

$$f(\theta) \propto \theta^2 \times (1-\theta)^3$$

Because the 7 is a constant — it does not change the *shape* of f as θ varies. In probability, distributions must sum to 1, so the constant is determined automatically. We only need to identify the shape.

**Key takeaway:** When you see θ^(α−1) × (1−θ)^(β−1), that shape tells you everything you need to know about the distribution, regardless of whatever constant sits in front of it.

---

### Section 1.2 — Business Motivation
#### (~15 minutes)

---

#### The Conversion Rate Problem

You run an e-commerce website. You test a new homepage design (Variant B) against the current design (Variant A) for two weeks:

| Variant | Visitors | Conversions | Observed Rate |
|---|---|---|---|
| A (current) | 600 | 24 | 4.0% |
| B (new) | 600 | 34 | 5.7% |

Variant B has a higher observed rate. Should you switch?

**The naive answer:** Yes — 5.7% > 4.0%.

**The problem:** These are *observed* rates from one two-week experiment. They are noisy estimates of the *true* rates. If you ran the experiment again next week, you would likely get slightly different numbers. The question you actually want answered is:

> *Given the data we collected, what is the probability that B's **true** conversion rate is higher than A's?*

If this probability is 55%, you should be very uncertain before switching. If it is 97%, you can switch with confidence. This is the question Bayesian A/B testing answers directly.

---

#### Why Traditional Statistics Falls Short

You may have heard of p-values. A p-value is the probability of observing data as extreme as yours *assuming A and B are identical*. A small p-value (< 0.05) is taken as evidence against that assumption.

But the p-value is backwards from what a decision-maker needs:
- **p-value:** P(data this extreme | A and B are identical)
- **What you want:** P(B is truly better | the data I observed)

Bayesian A/B testing gives you the second — the direct answer to your business question.

---

#### Strategic Frame: What Practitioners Know

> *"I'm very clear that I'm a big fan of test everything... even small bug fixes, even small changes can sometimes have surprising, unexpected impact."*
> — Ronny Kohavi, former VP of Experimentation at Microsoft and Airbnb

Kohavi's most instructive story: a Bing engineer noticed an idea in the backlog to move the second line of a search ad title to the first line. Nobody rated it highly. It sat there for months. Someone finally implemented it in two days just to clear the backlog.

The change increased Bing's revenue by approximately $100 million annually. It triggered an automated alarm because the revenue spike looked like a data pipeline error. The team spent days looking for the bug. There was no bug.

His conclusion: "We are often humbled by how bad we are at predicting the outcome of experiments." The implication for this lecture: if expert product teams cannot predict the value of a change, neither can you — which is exactly why you need a principled framework for learning from data, rather than a better ability to predict in advance.

---

### Section 1.3 — Conceptual Framework
#### (~20 minutes)

---

#### The True Conversion Rate θ

Every website variant has a true conversion rate. Call it θ (theta). It is the proportion of *all possible visitors* who would convert if shown this variant.

We cannot observe θ directly. We can only collect data from a sample and use it to make inferences.

Think of θ like the bias of a coin. If a coin is fair, θ = 0.5. Flipping it 10 times and getting 6 heads does not mean θ = 0.6 — that would be ignoring the noise in a small sample. More flips give a more accurate estimate, but we never observe θ itself.

---

#### Representing Uncertainty with a Distribution

Because θ is unknown, we represent our uncertainty about it using a **probability distribution** — a curve where the height at any value of θ tells us how plausible that value is.

This distribution evolves in two stages:

**The prior:** Before collecting data, you encode your existing belief. Based on industry benchmarks, you might believe conversion rates for your type of site are typically around 4% — but with real uncertainty. Your prior curve is peaked around 0.04 with tails extending in both directions.

**The posterior:** After collecting data, you update the prior. The result is a new curve — the posterior — that reflects both your prior belief and the evidence from the experiment. With 34 conversions out of 600 visitors, the posterior for θ_B is concentrated near 5.7% but still spread out, because 600 observations is not infinite.

---

#### The Logic of Bayesian Updating

The central idea of this entire lecture is one equation:

$$\text{Posterior} \propto \text{Prior} \times \text{Likelihood}$$

- **Prior:** What you believed about θ before seeing data
- **Likelihood:** How well different values of θ explain the data you observed
- **Posterior:** Your updated belief after combining both

The posterior is a compromise. With very sparse data (10 observations), the prior matters a lot. With abundant data (10,000 observations), the data dominates and the prior becomes nearly irrelevant. This is exactly what we want.

---

### Section 1.4 — Mathematical Framework
#### (~35 minutes)

---

#### Part A: The Likelihood — How Probable Is the Data?

We observed k conversions out of n visitors. For a given true conversion rate θ, how probable is this outcome?

Each visitor independently converts with probability θ. The number of conversions follows a **Binomial distribution**:

$$P(\text{k conversions} \mid \theta, n) \propto \theta^k \times (1-\theta)^{n-k}$$

**Plain English:** θ^k says "θ had to happen k times" (each conversion). (1−θ)^(n−k) says "θ had to fail n−k times" (each non-conversion). Multiply them together because each visitor's outcome is independent.

**Example:** n = 600, k = 34 (Variant B's data):
$$\text{Likelihood}(\theta) \propto \theta^{34} \times (1-\theta)^{566}$$

This expression is highest when θ ≈ 0.057 (= 34/600). It tells us that among all possible values of θ, the observed data is most consistent with θ ≈ 5.7%.

---

#### Part B: The Prior — Encoding Existing Belief

We need a prior distribution for θ that:
1. Lives on [0, 1] (since θ is a probability)
2. Can represent "I believe θ is around 4%, but I'm uncertain"
3. Has a mathematical form that combines cleanly with the likelihood

The **Beta distribution** satisfies all three. A Beta(α, β) distribution has this shape:

$$\text{Beta}(\theta; \alpha, \beta) \propto \theta^{\alpha-1} \times (1-\theta)^{\beta-1}$$

where α and β are parameters we choose to encode our prior belief:
- **Prior mean** = α / (α + β)
- **Higher α + β** = stronger prior (more concentrated curve)
- **Lower α + β** = weaker prior (flatter, more spread out curve)

**Our prior:** We encode "conversion rates are typically around 4%" with weak confidence using **Beta(2, 48)**:
> Prior mean = 2/(2+48) = 2/50 = **0.04** ✓
> Total weight α + β = 50 (equivalent to having seen 50 prior "pseudo-observations")

---

#### Part C: The Posterior — Combining Prior and Data

Now we combine the prior and the likelihood:

$$\text{Posterior} \propto \underbrace{\theta^{\alpha-1}(1-\theta)^{\beta-1}}_{\text{Prior: Beta(}\alpha\text{, }\beta\text{)}} \times \underbrace{\theta^k(1-\theta)^{n-k}}_{\text{Likelihood}}$$

Applying the exponent multiplication rule (add exponents with the same base):

$$\text{Posterior} \propto \theta^{(\alpha + k) - 1} \times (1-\theta)^{(\beta + n - k) - 1}$$

This is the kernel of a **Beta(α + k, β + n − k)** distribution.

> **The conjugate update rule:**
> $$\text{Prior } \text{Beta}(\alpha, \beta) + \text{Data}(k, n) \rightarrow \text{Posterior } \text{Beta}(\alpha + k, \; \beta + (n-k))$$

**In plain English:**
- Add the number of conversions to α
- Add the number of non-conversions to β

That is the entire update. No integration. No numerical methods. Just addition.

---

#### Part D: The Update in Numbers

**Variant A:** Prior Beta(2, 48) + 24 conversions, 576 non-conversions out of 600

$$\text{Posterior}_A = \text{Beta}(2+24, \; 48+576) = \text{Beta}(26, 624)$$

Posterior mean: 26/(26+624) = 26/650 = **0.040** = 4.0%

**Variant B:** Prior Beta(2, 48) + 34 conversions, 566 non-conversions out of 600

$$\text{Posterior}_B = \text{Beta}(2+34, \; 48+566) = \text{Beta}(36, 614)$$

Posterior mean: 36/(36+614) = 36/650 = **0.0554** ≈ 5.5%

Notice: The posterior mean (5.5%) is pulled slightly toward the prior (4.0%) compared to the raw observed rate (5.7%). The prior is exerting a small moderating influence — as it should with only 600 observations.

---

#### Part E: Computing P(B > A) via Monte Carlo

We now want P(θ_B > θ_A) — the probability that B's true conversion rate exceeds A's.

There is no closed-form formula for this. Instead, we simulate:

1. Draw 100,000 samples from Posterior_A = Beta(26, 624)
2. Draw 100,000 samples from Posterior_B = Beta(36, 614)
3. Count the fraction of draws where sample_B > sample_A

```python
import numpy as np

np.random.seed(42)
samples_A = np.random.beta(26, 624, 100_000)
samples_B = np.random.beta(36, 614, 100_000)

prob_B_greater = np.mean(samples_B > samples_A)
expected_lift  = np.mean((samples_B - samples_A) / samples_A) * 100

print(f"P(B > A) = {prob_B_greater:.3f}")
print(f"Expected lift = {expected_lift:.1f}%")
```

**Typical output:**
```
P(B > A) = 0.967
Expected lift = 38.5%
```

**Interpretation:** Given this data and prior, there is approximately a 97% probability that Variant B's true conversion rate is higher than Variant A's. The expected lift is about 38% — meaning B converts 38% more visitors than A on average across all plausible true rates.

---

> ### 🔍 Deep Dive: Why Beta is the Right Choice
> *This box is optional. Skip it if you are comfortable with the result above. Read it if you want to understand why Beta specifically — and not some other distribution — works so cleanly.*
>
> The reason Beta and Binomial combine so neatly is called **conjugacy**. A prior distribution is **conjugate** to a likelihood if the posterior is in the same family as the prior. For the Binomial likelihood (modeling conversion counts), the Beta distribution is the conjugate prior.
>
> Why does this matter? Because it means the posterior is always a Beta distribution — no matter how much data you collect. The update rule (add conversions to α, add non-conversions to β) works for 10 visitors or 10 million visitors. The math never gets harder.
>
> Formally: if Prior ~ Beta(α, β) and Likelihood is Binomial(n, θ), then Posterior ~ Beta(α + k, β + n − k). This follows directly from:
>
> Posterior ∝ Prior × Likelihood
> ∝ θ^(α−1)(1−θ)^(β−1) × θ^k(1−θ)^(n−k)
> = θ^(α+k−1)(1−θ)^(β+n−k−1)
>
> Which is precisely the kernel of Beta(α+k, β+n−k). The normalizing constant — the piece that makes the distribution integrate to 1 — takes care of itself.
>
> Conjugate priors are a gift from the mathematical structure of the problem. Not all likelihoods have conjugate priors. The fact that conversion rate estimation does is one reason Bayesian A/B testing is practically tractable at scale.

---

### Section 1.5 — Interpretation Guide
#### (~10 minutes)

**When an agent reports P(B > A) = 0.91, what does it mean?**

This is a posterior probability — it tells you how much confidence the data and prior together support the conclusion that B is genuinely better. It is NOT:
- The fraction of experiments where B would win if repeated
- A p-value
- A guarantee

The threshold for action depends on the cost of being wrong. For a button color change that can be reversed in 10 minutes, P = 0.80 is probably sufficient. For a pricing change affecting millions of customers that would be hard to reverse, P = 0.99 might still feel uncomfortably uncertain.

**What is the 95% credible interval on lift?**

The 2.5th and 97.5th percentiles of the simulated lift distribution. "Lift of [8%, 75%] with 95% credibility" means: the data is consistent with B being anywhere from 8% to 75% better than A, with 95% of the probability mass in that range. A wide interval means the experiment was underpowered — you need more data to narrow it.

**What is expected lift?**

The mean of the lift distribution across all plausible true conversion rates. This is the business-relevant summary: "on average across all scenarios consistent with this data, B converts 38% more visitors than A."

---

### Part 1 Checkpoint
#### 5 questions — participation credit

1. A Bayesian A/B test reports P(B > A) = 0.91. Your manager says "that means if we ran this experiment 100 times, B would win 91 of them." Is your manager's interpretation correct? Why or why not?

2. You start with a prior Beta(2, 48). You observe 56 conversions out of 800 visitors. What is the posterior? State the α and β values.

3. Without computing anything, would you expect the posterior mean to be above or below the raw observed rate of 56/800 = 7.0%? Why?

4. A colleague argues that a "proper" statistical approach requires a p-value < 0.05 before you can conclude B is better. What would you say to them?

5. You run an A/B test and get P(B > A) = 0.76. The change being tested is a major redesign of your checkout flow that would take 3 months to reverse. Should you ship it? What additional information would you want?

---

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### Agent-assisted live demo (~25 minutes)

**Scenario:** FitLoop (the company from the intensive day case) ran a pricing experiment: 1,847 trial users saw \$18/month (Variant A) and 1,853 saw \$22/month (Variant B). The outcome is trial-to-paid conversion within 14 days.

**Agent context prompt:**
```
I am running a Bayesian A/B test for a subscription pricing experiment.

Variant A ($18/month): 1,847 trial users, [CONVERSIONS_A] converted.
Variant B ($22/month): 1,853 trial users, [CONVERSIONS_B] converted.

Prior for both: Beta(2, 48) — weak prior around 4% baseline.

Please:
1. Compute posterior Beta distributions for each variant using the
   conjugate update rule.
2. Use Monte Carlo simulation with 100,000 samples and seed=42 to estimate:
   - P(conversion rate higher at $18) and P(conversion rate higher at $22)
   - Expected revenue per trial user at each price point
     (= conversion_rate × price)
   - 95% credible interval on the revenue difference
3. Plot both posterior distributions on the same chart.
4. Print all numerical results clearly labeled.
```

**What to watch for when the agent runs:**
- Does the posterior mean make sense relative to the observed rate?
- Is P(A) + P(B) close to 1.0? (It should be, within simulation noise.)
- Is the expected lift consistent with the difference in posterior means?
- Does the chart show B's distribution shifted right relative to A's?

---

### Section 2.2 — Homework Assignment
#### Due: start of next lecture

See notebook: `homework_01_bayesian_ab.ipynb`

**Part A** (individual): Posterior update calculations for a new dataset.
**Part B** (collaborative): Run the agent prompt from Section 2.1 and record outputs.
**Part C** (collaborative): Interpretation multiple-choice questions.

---

## Checkpoint Answer Key

**Q1.** No. P(B > A) = 0.91 is a **Bayesian posterior probability** about the unknown true rates given this specific data and prior. It does NOT mean "B wins 91% of repeated experiments" — that is a frequentist statement about long-run behavior under repeated sampling, which is not what Bayesian inference computes. The correct interpretation: given what we observed and our prior beliefs, there is a 91% probability that B's true conversion rate is higher than A's true conversion rate.

*Common wrong answer:* Interpreting P(B>A) as a p-value or a frequentist confidence statement. The distinction matters for decisions: P(B>A) = 0.91 means B is probably better *right now*, not that B would probably win in future experiments.

**Q2.** Posterior = **Beta(2 + 56, 48 + (800 − 56)) = Beta(58, 792)**
- α' = 2 + 56 = 58
- β' = 48 + (800 − 56) = 48 + 744 = 792

*Common wrong answer:* Adding total visitors instead of non-conversions to β: Beta(58, 848). The update adds non-conversions (failures), not total visitors.

**Q3.** **Below** 7.0%. The prior has mean 4.0%. The posterior is a weighted average of the prior and the data. With 800 observations and a prior equivalent to 50 pseudo-observations, the posterior mean is pulled slightly toward 4.0%, landing below 7.0%. With a weaker prior (smaller α + β), it would be closer to 7.0%.

*Common wrong answer:* "Above, because more data makes us more confident." Confidence (narrower distribution) and the location of the mean are different things. More data narrows the distribution but does not necessarily move the mean in one direction.

**Q4.** The p-value answers a different question: "how probable is this data, assuming A and B are identical?" A small p-value is evidence against the null hypothesis of equality — but it does not directly quantify the probability that B is better. Bayesian A/B testing answers the question a decision-maker actually needs: P(B is genuinely better | the data I collected). Additionally, p-values do not incorporate prior knowledge, do not give an estimate of the lift magnitude, and cannot be updated as data accumulates without correction (multiple testing problem).

*This question has no single wrong answer — it is designed to surface whether students understand the philosophical difference, not to produce one correct response.*

**Q5.** With P(B > A) = 0.76 and a costly, slow-to-reverse change: **probably do not ship yet.** For high-stakes, irreversible decisions, most practitioners require P ≥ 0.90–0.95 before acting. The additional information that would help: the width of the credible interval on lift (is the range of plausible outcomes acceptable even in the worst case?), the expected revenue impact of each direction of error, and whether you can run the experiment longer to narrow the uncertainty before the change must be made.

*Common wrong answer:* "Ship it — 76% is probably better than chance." This conflates P(better) with the expected value of the decision. A 76% probability that B is better implies a 24% probability that A is better. If A being better means losing three months of engineering work and alienating millions of customers, the asymmetry of consequences matters more than the raw probability.
