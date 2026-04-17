# Chapter 1: Bayesian A/B Testing
## How to Make Decisions Under Uncertainty

---

### Chapter Overview

This chapter develops the complete mathematical foundation for Bayesian A/B testing —
the most commonly used method for measuring whether a product change genuinely
improves a key metric. We cover the Beta distribution, the conjugacy property that
makes Bayesian updating tractable, Monte Carlo estimation of posterior probabilities,
and the business judgment required to act on these numbers.

**By the end of this chapter you should be able to:**
- Derive the conjugate update rule for Beta-Binomial models from first principles
- Compute exact posterior parameters for any A/B test by hand
- Estimate P(B > A) via Monte Carlo and explain what it means precisely
- Explain the difference between a Bayesian credible interval and a frequentist confidence interval
- Construct and justify a prior based on domain knowledge
- Identify the conditions under which the test result is reliable vs. unreliable

---

## 1.1 Strategic Frame: What Practitioners Know Before the Math

### The Opening Story: Ronny Kohavi and the $100 Million Backlog Item

Ronny Kohavi spent years as Microsoft's VP of Experimentation and later led
experimentation at Airbnb. He is widely considered the world's leading practitioner
on A/B testing at scale, having overseen tens of thousands of experiments across both
organizations.

His most instructive story is not about a sophisticated multi-variant experiment with
complex statistical design. It is about a simple backlog item that nobody wanted to run.

A Bing engineer noticed an idea in the product backlog: move the second line of a search
ad title up to the first line, making the title text larger and more prominent. Nobody
rated it highly — it seemed like a cosmetic change. It sat in the backlog for months.
Someone eventually implemented it in a few days just to clear it off the list.

The result triggered an automated alarm — the kind that fires when a data pipeline
breaks and revenue metrics spike to implausible levels. The team spent days looking for
the bug. There was no bug. The change was worth approximately $100 million in
annualized revenue. It was, Kohavi has said in public talks, one of the largest
single-experiment revenue impacts in Bing's history.

His conclusion, repeated across dozens of talks and interviews: *"We are often humbled
by how bad we are at predicting the outcome of experiments."* This is not false modesty.
After reviewing thousands of A/B tests at Microsoft and Airbnb, Kohavi and colleagues
found that fewer than one-third of experiments that teams believed would improve key
metrics actually did so [[Kohavi et al., "Online Controlled Experiments at Large Scale,"
KDD 2013 — free via Microsoft Research]].

The implication: if expert teams with years of domain experience cannot predict which
changes will matter, the only reliable path is principled measurement. This is why
Bayesian inference — a framework for updating beliefs from evidence — is the right
mathematical tool. You need a way to go from "we observed these conversion rates" to
"what is the probability that B is genuinely better than A?" That is exactly the
question this chapter answers.

**The counterpoint: Lauryn Isford on when NOT to experiment**

Lauryn Isford, who led growth at Airtable after six years at Facebook, adds an
important qualifier to the "test everything" mantra. She describes a "paralysis
disease" that can afflict analytics-heavy teams: running experiments on decisions
whose outcome would not change regardless of the result.

"If the business implication of a 6% versus 7% activation rate doesn't actually change
your decision," she argues, "the precision doesn't help." An experiment is expensive:
it consumes engineering time to instrument, analyst time to design and evaluate,
calendar time to run to statistical significance, and organizational attention to act on.

The skill is not knowing how to run experiments. It is knowing which decisions are
high-stakes enough that an experiment is worth the cost — and having the statistical
literacy to design it correctly when you decide to run one.

This chapter teaches the second skill. The first is a matter of strategic judgment that
no formula can provide.

### What Practitioners Know: Three Strategic Insights

**Insight 1: The prior matters more than most teams admit**

The prior distribution over conversion rates encodes everything you believed before
running the experiment. A flat prior (Beta(1,1)) says you have no idea whether
conversion rates are 0.1% or 99%. An informative prior (Beta(2, 48)) encodes the belief
that conversion rates are around 4%, based on industry benchmarks or past experiments.

The practical implication: with a flat prior, you need far more data to reach
confidence. With a realistic prior, modest sample sizes can be sufficient. Teams that
default to flat priors because they want to "let the data speak for itself" are actually
making their inference less efficient — they are pretending not to know things they do know.
This is not intellectual honesty. It is a failure to use available information.

Choosing the prior is a strategic decision. It should be grounded in domain knowledge
and documented in the experiment design before data collection begins (to prevent
post-hoc adjustment).

**Insight 2: Guardrail metrics are as important as the primary metric**

Kohavi's most important operational rule: every experiment must specify, in advance,
which metrics must NOT worsen — not just which metric it aims to improve.

A test that lifts revenue by increasing ad density is trivially easy to run. Clicks
increase, revenue per session increases, and the test "wins." But if time-on-site,
return visits, and user satisfaction all decline, you have mortgaged long-term
engagement for short-term revenue. The A/B test framework cannot protect you from this
unless you designate these metrics as guardrails before the experiment runs.

In the Bayesian framework, this means constructing joint posteriors across multiple
metrics, or at minimum running independent Bayesian tests on each guardrail metric and
requiring all to show P(metric did not worsen) > threshold.

**Insight 3: The threshold for action is a business decision, not a statistical one**

P(B > A) = 0.91 is a number. Whether 0.91 is sufficient to ship depends entirely on
the cost of being wrong. For a button color change reversible in ten minutes, 0.80
might be sufficient. For a pricing change affecting millions of customers that requires
18 months to roll back, 0.99 might not be enough.

The Bayesian framework is particularly well-suited to this reasoning because it
produces a probability directly interpretable in terms of decision risk. The
frequentist p-value does not — it tells you the probability of your data given the null
hypothesis, which is not the probability you need for a deployment decision.

---

## 1.2 Mathematical Prerequisites

### 1.2.1 Probability as a Measure

A probability is a number in [0, 1] that measures the plausibility of an event.
Formally, a probability measure P on a sample space Ω satisfies:
1. P(∅) = 0 and P(Ω) = 1
2. For disjoint events A, B: P(A ∪ B) = P(A) + P(B) (countable additivity)
3. P(Aᶜ) = 1 − P(A) for any event A

For our purposes, we work with two types of probability:
- **Frequentist probability:** the long-run frequency of an event under repeated sampling
- **Bayesian probability:** a degree of belief, updated as evidence accumulates

Bayesian A/B testing uses the second interpretation. θ is the true conversion rate —
a fixed unknown quantity. We use a probability distribution over θ to represent our
uncertainty about it.

*Reference: Wikipedia, "Probability" and "Bayesian probability" —
en.wikipedia.org/wiki/Probability, en.wikipedia.org/wiki/Bayesian_probability*

### 1.2.2 Conditional Probability and Bayes' Theorem

The conditional probability of A given B is:
$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}, \quad P(B) > 0$$

**Bayes' Theorem** follows immediately by applying this definition twice:
$$P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)}$$

In statistical inference, we substitute:
- A → the parameter θ (the unknown)
- B → the data D (what we observed)

$$P(\theta \mid D) = \frac{P(D \mid \theta) \cdot P(\theta)}{P(D)}$$

where:
- **P(θ | D)** is the **posterior** — what we believe about θ after seeing the data
- **P(D | θ)** is the **likelihood** — how probable the data is for each θ
- **P(θ)** is the **prior** — what we believed before seeing the data
- **P(D)** is the **marginal likelihood** (normalizing constant) — P(D) = ∫ P(D|θ)P(θ)dθ

Since P(D) does not depend on θ, we can write:
$$P(\theta \mid D) \propto P(D \mid \theta) \cdot P(\theta)$$

This proportionality relationship — **Posterior ∝ Likelihood × Prior** — is the
working form of Bayes' Theorem used in all calculations below.

*Reference: Wikipedia, "Bayes' theorem" — en.wikipedia.org/wiki/Bayes%27_theorem*
*Reference: Gelman et al., "Bayesian Data Analysis" 3rd ed., Chapter 1 —
free PDF at stat.columbia.edu/~gelman/book/BDA3.pdf*

### 1.2.3 The Gamma Function

The normalizing constants in Beta distribution calculations involve the **Gamma function**:
$$\Gamma(n) = \int_0^\infty t^{n-1} e^{-t} \, dt$$

Key properties:
- **Integer values:** Γ(n) = (n−1)! for positive integers n
- **Recursion:** Γ(n+1) = n·Γ(n) for all real n > 0
- **Special value:** Γ(1/2) = √π

We use Γ primarily to write the normalizing constant of the Beta distribution. You will
not need to compute Γ by hand in course calculations — it cancels in the update rule.

*Reference: Wikipedia, "Gamma function" — en.wikipedia.org/wiki/Gamma_function*

### 1.2.4 The Exponent Multiplication Rule

For the derivation in Section 1.4, we need only one algebraic rule:
$$a^m \times a^n = a^{m+n}$$

Applied to expressions with θ:
$$\theta^{\alpha-1} \times \theta^k = \theta^{(\alpha-1)+k} = \theta^{(\alpha+k)-1}$$

This rule is the only algebra required to derive the conjugate update.

---

## 1.3 The Binomial Likelihood

### Setup

A website has a true conversion rate θ — the probability that any given visitor converts.
We cannot observe θ directly. We observe:
- n = number of visitors (fixed by our experiment design)
- k = number of conversions (random, depending on θ and random variation)

We model each visitor's behavior as an independent Bernoulli trial:
$$X_i \sim \text{Bernoulli}(\theta), \quad i = 1, \ldots, n$$

where X_i = 1 if visitor i converts and X_i = 0 otherwise.

### The Binomial Distribution

The total number of conversions k = X₁ + X₂ + ... + Xₙ follows a **Binomial distribution**:
$$P(k \mid \theta, n) = \binom{n}{k} \theta^k (1-\theta)^{n-k}$$

The **binomial coefficient** $\binom{n}{k} = \frac{n!}{k!(n-k)!}$ counts the number of
ways to arrange k successes in n trials. It does not depend on θ, so for inference:

$$P(k \mid \theta, n) \propto \theta^k (1-\theta)^{n-k}$$

**Plain English:** θ^k says θ had to "succeed" k times (once per conversion).
(1−θ)^(n−k) says (1−θ) had to "succeed" n−k times (once per non-conversion).
The likelihood is maximized at θ = k/n — the observed conversion rate.

### Worked Example

Variant B: n = 600 visitors, k = 34 conversions.
$$\text{Likelihood}(\theta) \propto \theta^{34}(1-\theta)^{566}$$

Taking the derivative with respect to θ and setting it to zero:
$$\frac{d}{d\theta}\left[\theta^{34}(1-\theta)^{566}\right] = 34\theta^{33}(1-\theta)^{566} - 566\theta^{34}(1-\theta)^{565} = 0$$
$$\theta^{33}(1-\theta)^{565}\left[34(1-\theta) - 566\theta\right] = 0$$
$$34 - 34\theta - 566\theta = 0 \implies \theta = 34/600 = 0.0567$$

The likelihood is maximized at the observed rate, as expected.

*Reference: Wikipedia, "Binomial distribution" — en.wikipedia.org/wiki/Binomial_distribution*

---

## 1.4 The Beta Prior

### Why Beta?

We need a prior distribution for θ that:
1. Lives on [0, 1] — because θ is a probability
2. Is flexible enough to represent a wide range of beliefs
3. Combines algebraically with the Binomial likelihood to produce a tractable posterior

The **Beta distribution** satisfies all three requirements. It is defined by two
positive shape parameters α and β:

$$\text{Beta}(\theta; \alpha, \beta) = \frac{1}{B(\alpha, \beta)} \theta^{\alpha-1}(1-\theta)^{\beta-1}, \quad \theta \in [0,1]$$

where $B(\alpha, \beta) = \frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha+\beta)}$ is the
**Beta function** — the normalizing constant ensuring the distribution integrates to 1.

Since the Beta function does not depend on θ, we write:
$$\text{Beta}(\theta; \alpha, \beta) \propto \theta^{\alpha-1}(1-\theta)^{\beta-1}$$

### Key Properties of Beta(α, β)

| Property | Formula | Intuition |
|---|---|---|
| Mean | α/(α+β) | Center of mass |
| Mode | (α−1)/(α+β−2) for α,β > 1 | Peak of distribution |
| Variance | αβ/[(α+β)²(α+β+1)] | Spread |
| Concentration | α+β | Higher = narrower distribution |

**The prior mean is α/(α+β).** If you want a prior centered at 4% conversion, you
need α/(α+β) = 0.04. One choice: α = 2, β = 48, giving mean = 2/50 = 0.04.

**The concentration α+β acts like a sample size.** A prior Beta(2, 48) has
concentration 50 — equivalent to having run a prior experiment with 50 observations
(2 conversions, 48 non-conversions). A flat prior Beta(1, 1) has concentration 2 —
essentially no prior information.

### Choosing the Prior

For conversion rate problems:

| Prior belief | Suggested parameterization |
|---|---|
| No information | Beta(1, 1) — uniform, all conversion rates equally plausible |
| Industry benchmark ~4%, weak confidence | Beta(2, 48) — mean=4%, concentration=50 |
| Strong historical data, conversion ≈ 6% | Beta(6, 94) — mean=6%, concentration=100 |

> **Teaching note (whiteboard):** Draw three Beta distributions on the same axes:
> Beta(1,1) is flat. Beta(2,48) is a gentle hill peaked at 4%. Beta(20,480) is a sharp
> spike at 4%. This visual makes "concentration = certainty" immediate and intuitive.
> Ask: which prior should you use if you have 2 years of historical data vs. if you're
> launching a new product type?

*Reference: Wikipedia, "Beta distribution" — en.wikipedia.org/wiki/Beta_distribution*

---

## 1.5 The Conjugate Update — Full Derivation

### What Is Conjugacy?

A prior distribution is **conjugate** to a likelihood if the posterior is in the same
family as the prior. For the Binomial likelihood, the Beta distribution is the
conjugate prior. This means:

> **If Prior ~ Beta(α, β) and the likelihood is Binomial(n, θ), then Posterior ~ Beta(α + k, β + n − k)**

This is not just convenient — it is provably the only update rule consistent with
Bayesian probability. Below is the full derivation.

### Derivation

**Step 1:** Write the posterior as proportional to likelihood × prior.

$$P(\theta \mid k, n) \propto P(k \mid \theta, n) \cdot P(\theta)$$

$$\propto \underbrace{\theta^k (1-\theta)^{n-k}}_{\text{Binomial likelihood}} \times \underbrace{\theta^{\alpha-1}(1-\theta)^{\beta-1}}_{\text{Beta prior}}$$

**Step 2:** Multiply the two expressions, grouping by base (θ and 1−θ separately).

$$\propto \theta^k \cdot \theta^{\alpha-1} \times (1-\theta)^{n-k} \cdot (1-\theta)^{\beta-1}$$

**Step 3:** Apply the exponent multiplication rule: $a^m \cdot a^n = a^{m+n}$.

$$\propto \theta^{(k + \alpha - 1)} \times (1-\theta)^{(n-k+\beta-1)}$$

$$= \theta^{(\alpha+k)-1} \times (1-\theta)^{(\beta+n-k)-1}$$

**Step 4:** Recognize the kernel.

This expression $\theta^{(\alpha+k)-1}(1-\theta)^{(\beta+n-k)-1}$ is the kernel of a
Beta distribution with parameters:
$$\alpha' = \alpha + k, \qquad \beta' = \beta + (n-k)$$

Therefore: **Posterior = Beta(α + k, β + (n − k))**

The normalizing constant B(α', β') is determined automatically by the Beta function —
we never need to compute it.

### The Update Rule in Words

| Component | Update |
|---|---|
| Prior successes (α) | Add observed conversions k |
| Prior failures (β) | Add observed non-conversions (n − k) |
| Posterior mean | (α + k) / (α + k + β + n − k) = (α + k) / (α + β + n) |

The posterior mean is a **weighted average** of the prior mean and the observed rate:
$$\text{Posterior mean} = \frac{\alpha + k}{\alpha + \beta + n} = \underbrace{\frac{\alpha + \beta}{\alpha + \beta + n}}_{\text{weight on prior}} \cdot \underbrace{\frac{\alpha}{\alpha+\beta}}_{\text{prior mean}} + \underbrace{\frac{n}{\alpha + \beta + n}}_{\text{weight on data}} \cdot \underbrace{\frac{k}{n}}_{\text{observed rate}}$$

As n → ∞, the weight on the prior → 0 and the posterior mean → observed rate.
As n → 0, the posterior mean → prior mean. The prior's influence is proportional to
its concentration (α+β) relative to the sample size n.

### Worked Example

Prior: Beta(2, 48). Observed: 600 visitors, 34 conversions.

| | α | β | Mean |
|---|---|---|---|
| Prior | 2 | 48 | 2/50 = **4.00%** |
| Update | +34 | +(600−34)=+566 | |
| Posterior | 2+34=**36** | 48+566=**614** | 36/650 = **5.54%** |

The posterior mean (5.54%) is pulled slightly below the observed rate (5.67%) by the
prior (4%). With n = 600, the data dominates but the prior still exerts a small pull.
This is correct behavior — we should be slightly skeptical of exactly 5.67%.

> **Teaching note (whiteboard/iPad):** Write the three steps in sequence, leaving
> space for each algebra step. The key moment is Step 3 — write
> θ^k × θ^(α−1) = θ^(k+α−1) = θ^(α+k)−1 explicitly. Students who see this once
> almost always remember the update rule forever. Budget 8–10 minutes for this derivation.

*Reference: Wikipedia, "Conjugate prior" — en.wikipedia.org/wiki/Conjugate_prior*
*Reference: Gelman et al., BDA3, Section 2.4 — free PDF as above*
*Reference: Wikipedia, "Beta-binomial distribution" — en.wikipedia.org/wiki/Beta-binomial_distribution*

---

## 1.6 Computing P(B > A)

### Why We Cannot Use a Formula

We want P(θ_B > θ_A) — the probability that B's true conversion rate exceeds A's.
Both θ_A and θ_B are random variables following Beta distributions:
$$\theta_A \sim \text{Beta}(\alpha_A, \beta_A), \quad \theta_B \sim \text{Beta}(\alpha_B, \beta_B)$$

The exact expression for P(θ_B > θ_A) involves an integral of the product of two
Beta densities over the region {(θ_A, θ_B) : θ_B > θ_A}:

$$P(\theta_B > \theta_A) = \int_0^1 \int_0^{\theta_B} f_A(\theta_A) f_B(\theta_B) \, d\theta_A \, d\theta_B$$

This integral has a closed form involving the regularized incomplete Beta function, but
it is computationally cumbersome to evaluate by hand. The standard practitioner
approach is Monte Carlo simulation.

### Full Derivation: Why This Integral Has No Simple Closed Form

The joint density of (θ_A, θ_B) is:
$$f(\theta_A, \theta_B) = f_A(\theta_A) \cdot f_B(\theta_B) \propto \theta_A^{\alpha_A-1}(1-\theta_A)^{\beta_A-1} \cdot \theta_B^{\alpha_B-1}(1-\theta_B)^{\beta_B-1}$$

(The two parameters are independent — their posteriors are computed from separate
variant datasets.)

$$P(\theta_B > \theta_A) = \int_0^1 F_A(\theta_B) f_B(\theta_B) \, d\theta_B$$

where $F_A(\theta_B) = \int_0^{\theta_B} f_A(\theta_A) d\theta_A$ is the CDF of θ_A
evaluated at θ_B. This is the regularized incomplete Beta function $I_{\theta_B}(\alpha_A, \beta_A)$,
which has no elementary closed form for general non-integer parameters.

*Reference: Wikipedia, "Beta function" and "Regularized incomplete beta function" —
en.wikipedia.org/wiki/Beta_function*

### Monte Carlo Estimation

Monte Carlo estimation replaces the integral with a sample average:

**Algorithm:**
1. Draw S samples from Posterior_A: $\tilde{\theta}_A^{(1)}, \ldots, \tilde{\theta}_A^{(S)} \sim \text{Beta}(\alpha_A, \beta_A)$
2. Draw S samples from Posterior_B: $\tilde{\theta}_B^{(1)}, \ldots, \tilde{\theta}_B^{(S)} \sim \text{Beta}(\alpha_B, \beta_B)$
3. Estimate: $\hat{P}(\theta_B > \theta_A) = \frac{1}{S}\sum_{s=1}^S \mathbf{1}[\tilde{\theta}_B^{(s)} > \tilde{\theta}_A^{(s)}]$

By the **Law of Large Numbers**, $\hat{P} \to P(\theta_B > \theta_A)$ as S → ∞.
With S = 100,000, the Monte Carlo standard error is:
$$\text{SE} = \sqrt{\frac{\hat{P}(1-\hat{P})}{S}} \approx \sqrt{\frac{0.97 \times 0.03}{100{,}000}} \approx 0.0005$$

This is negligible — the simulation is accurate to 3 decimal places.

### Expected Lift and Credible Intervals

**Expected lift:**
$$\hat{E}\left[\frac{\theta_B - \theta_A}{\theta_A}\right] = \frac{1}{S}\sum_{s=1}^S \frac{\tilde{\theta}_B^{(s)} - \tilde{\theta}_A^{(s)}}{\tilde{\theta}_A^{(s)}}$$

**95% credible interval on lift:**
Let $L^{(s)} = (\tilde{\theta}_B^{(s)} - \tilde{\theta}_A^{(s)}) / \tilde{\theta}_A^{(s)}$. The 95% CI is [P₂.₅(L), P₉₇.₅(L)] — the 2.5th and 97.5th percentiles of the sampled lift distribution.

### Worked Example (Continuing from Section 1.5)

Posterior_A = Beta(26, 624). Posterior_B = Beta(36, 614).

```python
import numpy as np
np.random.seed(42)
S = 100_000
theta_A = np.random.beta(26, 624, S)
theta_B = np.random.beta(36, 614, S)

prob_B_greater = np.mean(theta_B > theta_A)
lift = (theta_B - theta_A) / theta_A
expected_lift = np.mean(lift) * 100
ci_lower = np.percentile(lift, 2.5) * 100
ci_upper = np.percentile(lift, 97.5) * 100

# Results:
# P(B > A) ≈ 0.967
# Expected lift ≈ 38.5%
# 95% CI ≈ [8.2%, 75.1%]
```

**Interpreting these numbers:**
- P(B > A) = 0.967: There is a 96.7% posterior probability that B's true conversion rate exceeds A's.
- Expected lift = 38.5%: On average across all plausible true rates, B converts 38.5% more visitors.
- 95% CI = [8.2%, 75.1%]: The data is consistent with B being anywhere from 8% to 75% better than A. This wide interval indicates genuine uncertainty — the sample size is modest.

*Reference: Wikipedia, "Monte Carlo method" — en.wikipedia.org/wiki/Monte_Carlo_method*
*Reference: Wikipedia, "Credible interval" — en.wikipedia.org/wiki/Credible_interval*
*Reference: arXiv:1602.05549, Deng, Lu, Litz — "Continuous monitoring of A/B tests without pain"*

---

## 1.7 Interpreting Results: Bayesian vs. Frequentist

### What P(B > A) Is — and Is Not

| Statement | Correct? | Explanation |
|---|---|---|
| "97% probability B's true rate exceeds A's" | ✅ | This is the direct Bayesian interpretation |
| "B would win 97% of repeated experiments" | ❌ | This is a frequentist statement about repeated sampling |
| "We can be 97% confident our data shows B is better" | Approximately ✅ | Informal but not technically wrong |
| "p-value = 1 − 0.97 = 0.03" | ❌ | P(B>A) and p-values answer different questions |

### The Frequentist Alternative: p-values and Their Limitation

A p-value is P(observing data as extreme as ours | H₀: θ_A = θ_B). It answers: "how unusual would our data be if there were truly no difference?"

The limitations for A/B testing decisions:
1. A p-value does not give P(B is genuinely better) — the probability a decision-maker needs
2. "p < 0.05" is an arbitrary threshold with no direct connection to business value
3. p-values cannot be accumulated across sequential tests without correction (the "peeking problem")
4. p-values cannot quantify the magnitude of the effect, only its statistical significance

Bayesian A/B testing directly answers "what is the probability B is better?" — making
the decision threshold explicit and business-meaningful.

*Reference: Wikipedia, "P-value" — en.wikipedia.org/wiki/P-value*
*Reference: arXiv:1411.5018, Deng et al., "Objective Bayesian Two Sample Hypothesis Testing for Online Controlled Experiments"*

### Bayesian vs. Frequentist Credible/Confidence Intervals

A **Bayesian 95% credible interval** [L, U] means: given the data and prior, there is
a 95% posterior probability that the true parameter lies in [L, U].

A **frequentist 95% confidence interval** means: if you repeated the experiment
infinitely many times, 95% of the resulting intervals would contain the true parameter.
It does NOT mean there is a 95% probability the true parameter is in any specific
computed interval.

For most business users, the Bayesian credible interval has the more natural and useful
interpretation.

---

## 1.8 Sample Size and Stopping Rules

### The Peeking Problem

A common mistake: running an A/B test, checking the results repeatedly, and stopping
as soon as p < 0.05. This dramatically inflates the false positive rate. If you check
results at 5 different points during the test and stop when any check shows p < 0.05,
the true false positive rate is approximately 19%, not 5% [[Johari et al., arXiv:1512.04922]].

### Bayesian Sequential Testing

In the Bayesian framework, you can update the posterior at any point without inflating
error rates — the posterior is always a valid summary of current belief given all data
seen. However, stopping "as soon as P(B > A) > 0.95" still inflates the false positive
rate because you are selecting on a threshold crossing [[Deng et al., arXiv:1602.05549]].

The correct approach: either (a) determine sample size in advance and do not stop
early, or (b) use a proper sequential testing framework (e.g., always-valid p-values,
or Bayesian hypothesis testing with explicit stopping rules).

### Minimum Sample Size Calculation

For frequentist testing, the required sample size per variant to detect a minimum
detectable effect (MDE) at significance level α and power 1−β is:

$$n = \frac{(z_{1-\alpha/2} + z_{1-\beta})^2 [\theta_A(1-\theta_A) + \theta_B(1-\theta_B)]}{(\theta_B - \theta_A)^2}$$

For a two-sided test with α = 0.05, power = 0.80:
- z₀.₉₇₅ = 1.96, z₀.₈₀ = 0.84
- θ_A = 0.04 (baseline), θ_B = 0.05 (5% lift from 4% to 4.2%):
$$n = \frac{(1.96 + 0.84)^2 [0.04(0.96) + 0.042(0.958)]}{(0.042 - 0.04)^2} \approx 28{,}000 \text{ per variant}$$

*Reference: arXiv:1512.04922, Johari et al., "Peeking at A/B Tests"*

---

## 1.9 Full Proof Appendix

### Proof A: Beta(α, β) Integrates to 1

The Beta function is defined as:
$$B(\alpha, \beta) = \int_0^1 \theta^{\alpha-1}(1-\theta)^{\beta-1} \, d\theta$$

We need to show $\int_0^1 \frac{1}{B(\alpha,\beta)}\theta^{\alpha-1}(1-\theta)^{\beta-1} d\theta = 1$, which is true by construction (dividing by the integral of the kernel). The identity $B(\alpha,\beta) = \Gamma(\alpha)\Gamma(\beta)/\Gamma(\alpha+\beta)$ follows from computing the integral via Laplace's method or the Gamma-function recursion.

### Proof B: The Posterior Mean is a Weighted Average of Prior Mean and Observed Rate

Posterior: Beta(α + k, β + n − k). Posterior mean:
$$\mu_{post} = \frac{\alpha+k}{\alpha+\beta+n}$$

Rewrite by splitting the numerator:
$$= \frac{\alpha+\beta}{\alpha+\beta+n} \cdot \frac{\alpha}{\alpha+\beta} + \frac{n}{\alpha+\beta+n} \cdot \frac{k}{n}$$

$$= w_{prior} \cdot \mu_{prior} + w_{data} \cdot \hat{\theta}$$

where $w_{prior} = (\alpha+\beta)/(\alpha+\beta+n)$ and $w_{data} = n/(\alpha+\beta+n)$.

Since $w_{prior} + w_{data} = 1$, this is indeed a weighted average with weights
proportional to prior concentration (α+β) vs. observed sample size (n). QED.

### Proof C: Why Conjugacy Holds Only for the Beta-Binomial Pair

A prior p(θ) is conjugate to likelihood p(D|θ) if and only if the prior and likelihood
belong to the **exponential family** with the same sufficient statistics. The Binomial
has sufficient statistic (k, n) and natural parameter log(θ/(1−θ)). The Beta
distribution is the natural conjugate prior for this parameterization — its log-density
is linear in log(θ) and log(1−θ), matching the Binomial's natural parameters. This
algebraic match is exactly why the posterior update reduces to simple addition.

*Reference: Wikipedia, "Exponential family" and "Conjugate prior" —
en.wikipedia.org/wiki/Exponential_family, en.wikipedia.org/wiki/Conjugate_prior*

---

## 1.10 Verification Checklist

Use this checklist to verify agent-generated A/B test outputs:

| Check | How to verify |
|---|---|
| Posterior α = prior_α + conversions? | Compute by hand and compare |
| Posterior β = prior_β + (visitors − conversions)? | Compute by hand and compare |
| Posterior mean = α' / (α' + β')? | Compute by hand |
| Posterior mean closer to prior than observed rate? | Should be if n is modest |
| P(A) + P(B) ≈ 1.0? | Sum should be within 0.01 (Monte Carlo noise) |
| Expected lift = (mean_B − mean_A) / mean_A? | Approximately, since the posterior means are point estimates |
| 95% CI width reasonable? | Wider CI = less data, more uncertainty |
| CI consistent with P(B > A)? | If P(B>A) = 0.97, the lower CI bound should be positive |

---

## 1.11 Chapter Bibliography

**Free primary references:**

1. Wikipedia, "Beta distribution" — en.wikipedia.org/wiki/Beta_distribution
2. Wikipedia, "Conjugate prior" — en.wikipedia.org/wiki/Conjugate_prior
3. Wikipedia, "Binomial distribution" — en.wikipedia.org/wiki/Binomial_distribution
4. Wikipedia, "Bayes' theorem" — en.wikipedia.org/wiki/Bayes%27_theorem
5. Wikipedia, "Credible interval" — en.wikipedia.org/wiki/Credible_interval
6. Wikipedia, "P-value" — en.wikipedia.org/wiki/P-value
7. Gelman, A., Carlin, J. B., Stern, H. S., et al. *Bayesian Data Analysis*, 3rd ed. Chapters 1–3. Free PDF: stat.columbia.edu/~gelman/book/BDA3.pdf
8. Johari, R., Pekelis, L., and Walsh, D. J. (2015). "Peeking at A/B Tests: Why It Matters and What to Do About It." arXiv:1512.04922. arxiv.org/abs/1512.04922
9. Deng, A., Lu, J., and Litz, S. (2016). "Continuous monitoring of A/B tests without pain: Optional stopping in Bayesian testing." arXiv:1602.05549. arxiv.org/abs/1602.05549
10. Kohavi, R., Longbotham, R., Sommerfield, D., and Henne, R. M. (2009). "Controlled experiments on the web: Survey and practical guide." *Data Mining and Knowledge Discovery* 18(1):140–181. Free via Microsoft Research: exp-platform.com/Documents/controlledExperimentsOnTheWeb.pdf
11. VanderPlas, J. (2014). "Frequentism and Bayesianism" (5-part blog series). jakevdp.github.io/blog/2014/03/11/frequentism-and-bayesianism-a-practical-intro/

**Lenny's Podcast episode references:**
- Ronny Kohavi episode: "A/B testing and why it's so hard to do well" — Lenny's Podcast
- Lauryn Isford episode: "Finding and setting up your growth model" — Lenny's Podcast

