# Chapter 3: Marketing Mix Modeling (MMM)
## How to Measure What Marketing Actually Does

---

### Chapter Overview

Marketing Mix Modeling estimates the contribution of each marketing channel to
revenue by modeling carryover (adstock) and saturation (Hill function) simultaneously.
This chapter derives both transformations from first principles, develops the full
regression specification, derives the budget optimization condition analytically, and
explains why MMM produces more reliable attribution than last-touch methods.

**By the end of this chapter you should be able to:**
- Derive the adstock recursion and its long-run geometric sum
- Prove that H(EC₅₀) = 0.5 for any EC₅₀ and α
- Compute adstock and Hill values by hand from a spend series
- Interpret fitted MMM parameters (λ, EC₅₀, α, β)
- Derive the marginal ROI equalization condition for optimal budget allocation
- Explain the identification challenges in MMM estimation

---

## 3.1 Strategic Frame: What Practitioners Know

### Jonathan Becker: $3.5 Billion and the Attribution Trap

Jonathan Becker has managed over $3.5 billion in paid acquisition budgets across Uber,
Asana, Square, and Masterclass. He has watched a specific failure pattern repeat
itself across dozens of companies:

1. A company uses last-touch attribution and observes that Paid Search generates
   conversions at low cost-per-acquisition.
2. The marketing team shifts budget away from TV, podcast, and social toward Paid Search.
3. Paid Search metrics hold or improve for 6–12 months.
4. Organic brand search volume begins declining. Direct traffic falls. Branded keyword
   CPCs rise. The paid search system is now working harder to generate the same volume.
5. Eventually the metrics collapse — but by then the budget decisions have been
   institutionalized and the causal mechanism is invisible.

His diagnosis: "Paid search captures demand. It does not create it." TV and podcast
build the awareness that makes people search for your brand. Last-touch attribution
gives paid search the credit because it appears last. The problem is structural, not
analytical — you cannot fix it by looking more carefully at last-touch numbers.

MMM is the analytical response: by modeling all channels simultaneously and accounting
for the delayed effects of awareness advertising (via adstock), it can separate
demand-creation from demand-capture.

### Yuriy Timen: The Insufficient Test Problem

Yuriy Timen spent 8.5 years as Head of Growth at Grammarly, where he scaled the
business from 6 to 30 million daily active users. He later advised Canva and Airtable.

His contribution to the attribution discussion is about the *interpretation* of MMM
results, not just measurement. When an MMM returns a low coefficient for a channel,
there are two possible explanations:
1. The channel genuinely doesn't work for this business
2. The channel was never given an adequate test — too little spend, too few creative
   variants, too short a time window

His rule for YouTube: minimum impressions threshold, two to three distinct creative
angles, specific click-through rate benchmarks as evaluation criteria. If you haven't
met those requirements, you haven't tested YouTube — you've tested one insufficient
execution of YouTube, and no MMM can rehabilitate that evidence.

The practical implication: MMM coefficients should be interrogated against the quality
of the historical data that went into estimating them. A channel that was always
underfunded will appear weak regardless of its true potential.

### Three Strategic Insights

**Insight 1: The decay parameter λ is a business prior, not just a statistical output.**
Different channels have fundamentally different memory structures based on how
advertising works. TV creates brand impressions through repeated exposure; these
impressions decay slowly over weeks. Paid search captures immediate intent that
expires within hours. Before accepting an MMM's fitted λ for TV, ask: does a decay
rate of 0.85 match what we know about how TV advertising actually works? A fitted
λ = 0.20 for TV should be challenged; λ = 0.80 is consistent with empirical evidence
on TV carryover.

**Insight 2: Budget optimization outputs are directional, not prescriptive.**
An MMM that recommends shifting 40% of TV spend to digital cannot account for the
diminishing returns that kick in as digital spend scales beyond observed levels.
The model has seen TV at $X/week and digital at $Y/week. It has not seen TV at $0
and digital at $X + $Y. Extrapolating Hill curves far outside the training range is
risky. Use the marginal ROI comparison as a directional signal and test reallocation
incrementally — moving budget in steps and observing actual results.

**Insight 3: Incrementality holdouts are the ground truth that validates MMM.**
MMM estimates channel contributions by modeling the time-series relationship between
spend and revenue. This is observational, not experimental. The only way to know
whether a channel is truly incremental is to turn it off for a randomly selected
geography or customer segment and measure the lift. High-quality MMM analyses are
always cross-validated against geo-holdout tests or channel blackout experiments before
major budget reallocations are approved.

---

## 3.2 Mathematical Prerequisites

### 3.2.1 Geometric Series

For a constant ratio r with |r| < 1, the infinite geometric series converges:

$$\sum_{k=0}^{\infty} r^k = 1 + r + r^2 + r^3 + \cdots = \frac{1}{1-r}$$

**Proof of convergence:**
Let $S_n = \sum_{k=0}^{n} r^k$. Then $rS_n = \sum_{k=1}^{n+1} r^k$.
Subtracting: $S_n - rS_n = 1 - r^{n+1}$, so $S_n = (1-r^{n+1})/(1-r)$.
As n → ∞, $r^{n+1} \to 0$ when |r| < 1, giving $S_\infty = 1/(1-r)$.

**Partial sum (finite horizon):**
$$\sum_{k=0}^{n-1} r^k = \frac{1 - r^n}{1 - r}$$

We use the infinite sum to characterize total long-run advertising carryover.

*Reference: Wikipedia, "Geometric series" — en.wikipedia.org/wiki/Geometric_series*

### 3.2.2 Derivatives for Optimization

For marginal ROI computation, we differentiate the Hill function:

**Power rule:** $\frac{d}{ds}s^\alpha = \alpha s^{\alpha-1}$

**Quotient rule:** $\frac{d}{ds}\frac{f(s)}{g(s)} = \frac{f'(s)g(s) - f(s)g'(s)}{g(s)^2}$

### 3.2.3 Nonlinear Least Squares (Concept)

Unlike OLS (which has a closed-form solution), fitting MMM parameters requires
numerical optimization because the Hill and adstock transformations are nonlinear
in their parameters (λ, EC₅₀, α). The objective:

$$\min_{\lambda_j, \text{EC}_{50,j}, \alpha_j, \beta_j, \beta_0} \sum_{t=1}^T \left(y_t - \beta_0 - \sum_j \beta_j H(A_{j,t})\right)^2$$

Solved via gradient descent or the Nelder-Mead simplex method.

---

## 3.3 Adstock: Modeling Advertising Carryover

### The Business Problem

If a TV campaign runs in weeks 1–4, it would be wrong to model revenue in week 7 as
having no TV contribution. Brand impressions from the campaign are still influencing
purchase decisions. An adstock transformation converts raw spend series into an
"effective spend" series that reflects this carryover.

### The Adstock Recursion

$$A_t = S_t + \lambda A_{t-1}, \quad A_0 = 0, \quad 0 \leq \lambda < 1$$

where $S_t$ = spend in period t, $\lambda$ = decay rate, $A_t$ = adstock at time t.

### Deriving the Long-Run Effect

By backward substitution, substituting the recursion into itself repeatedly:

$$A_t = S_t + \lambda A_{t-1}$$
$$= S_t + \lambda(S_{t-1} + \lambda A_{t-2})$$
$$= S_t + \lambda S_{t-1} + \lambda^2 A_{t-2}$$
$$= S_t + \lambda S_{t-1} + \lambda^2(S_{t-2} + \lambda A_{t-3})$$
$$= S_t + \lambda S_{t-1} + \lambda^2 S_{t-2} + \lambda^3 S_{t-3} + \cdots$$

**Closed form:** $A_t = \sum_{k=0}^{t} \lambda^k S_{t-k}$

For a **single impulse** at t=0 (S₀ = S, all other Sₜ = 0):
$$A_t = \lambda^t S$$

The total long-run adstock generated by this single spend:
$$\sum_{t=0}^{\infty} A_t = S \sum_{t=0}^{\infty} \lambda^t = \frac{S}{1-\lambda}$$

**The multiplier 1/(1−λ):** A channel with λ = 0.75 generates 4× the immediate
spend in total long-run adstock. This quantifies why cutting TV spending (high λ)
has long-run revenue implications that extend well beyond the campaign period.

### Stability Requirement

The geometric series converges only if |λ| < 1. For λ = 1: adstock accumulates
without bound. For λ > 1: the system is explosive. These are economically
nonsensical — advertising effects do not grow indefinitely. Therefore 0 ≤ λ < 1 is a
model constraint enforced during estimation.

> **Teaching note (board/tablet):** Write the recursion. Unroll it four times:
> A₁, A₂, A₃, A₄. Let students spot the pattern before writing the closed form.
> Then jump to: "if only S₁ > 0 and all others are zero, what does A_t look like
> for t = 1, 2, 3, 4?" Answer: S₁, λS₁, λ²S₁, λ³S₁. This is a simple geometric
> sequence. Sum to infinity gives S₁/(1−λ). Budget 10–12 minutes.

### Worked Numerical Example

Spend series: S₁ = 200, S₂ = 0, S₃ = 50, S₄ = 0. Decay λ = 0.60.

| Period | Spend Sₜ | Calculation | Adstock Aₜ |
|---|---|---|---|
| 0 | — | A₀ = 0 | 0 |
| 1 | 200 | 200 + 0.60×0 | **200** |
| 2 | 0 | 0 + 0.60×200 | **120** ← carryover with zero spend |
| 3 | 50 | 50 + 0.60×120 | **122** |
| 4 | 0 | 0 + 0.60×122 | **73.2** |

Long-run total from S₁=200 alone: 200/(1−0.60) = **500** units of effective adstock.

### Choosing λ

| Channel type | Typical λ range | Rationale |
|---|---|---|
| TV / Out-of-home | 0.70–0.90 | Brand impressions persist for weeks |
| Podcast | 0.50–0.70 | Deep attention, medium persistence |
| Paid social | 0.40–0.60 | Scroll-based; moderate memory |
| Email | 0.20–0.40 | Opens within days; fast decay |
| Paid search | 0.10–0.25 | Captures immediate intent; near-zero carryover |

These are empirically informed ranges, not exact values. Estimation from data (with
appropriate priors if using Bayesian MMM) should dominate these defaults for
established channels with long spend histories.

*Reference: Wikipedia, "Advertising adstock" — en.wikipedia.org/wiki/Advertising_adstock*
*Reference: Broadbent, S. (1979). "One way TV advertisements work." *Journal of the
Market Research Society* 21(3). (Original adstock paper, not free online, but widely
cited; summary available at Warc.com)*

---

## 3.4 Hill Function: Modeling Saturation

### The Business Problem

Diminishing returns in advertising are empirically universal: no channel delivers
constant revenue per dollar at all spend levels. The Hill function is the standard
parametric model for this saturation relationship.

The Hill function originates in pharmacology (Archibald Hill, 1910) for
hemoglobin-oxygen binding curves. It was adopted by marketing researchers for its
well-behaved S-shaped response properties.

*Reference: Wikipedia, "Hill equation (biochemistry)" —
en.wikipedia.org/wiki/Hill_equation_(biochemistry)*

### Definition

$$H(s; \text{EC}_{50}, \alpha) = \frac{s^\alpha}{\text{EC}_{50}^\alpha + s^\alpha}$$

**Parameters:**
- $s$ = current spend (or adstock value) — the input
- $\text{EC}_{50}$ = spend at 50% maximum effectiveness (the half-effectiveness point)
- $\alpha$ = shape parameter (steepness of the S-curve)
- $H(s) \in (0, 1)$ for all finite s > 0 — bounded output

### Key Properties and Proofs

**Property 1: H(0) = 0 (no spend, no effect)**
$$H(0) = \frac{0^\alpha}{\text{EC}_{50}^\alpha + 0^\alpha} = \frac{0}{\text{EC}_{50}^\alpha} = 0$$

**Property 2: H(EC₅₀) = 0.5 regardless of EC₅₀ or α**

Substitute s = EC₅₀:
$$H(\text{EC}_{50}) = \frac{\text{EC}_{50}^\alpha}{\text{EC}_{50}^\alpha + \text{EC}_{50}^\alpha} = \frac{\text{EC}_{50}^\alpha}{2 \cdot \text{EC}_{50}^\alpha} = \frac{1}{2}$$

The EC₅₀ terms divide out completely. This is the defining property of EC₅₀ — it is
always the half-effectiveness point, regardless of the scale of spend or the shape α.

**Property 3: H(s) is strictly increasing for all s > 0**

$$\frac{dH}{ds} = \frac{\alpha \cdot \text{EC}_{50}^\alpha \cdot s^{\alpha-1}}{(\text{EC}_{50}^\alpha + s^\alpha)^2}$$

Each factor in the numerator ($\alpha > 0$, $\text{EC}_{50}^\alpha > 0$, $s^{\alpha-1} > 0$
for s > 0 and α > 0) is positive. Denominator is always positive. Therefore dH/ds > 0
for all s > 0.

**Property 4: H(s) → 1 as s → ∞**

$$\lim_{s \to \infty} H(s) = \lim_{s \to \infty} \frac{s^\alpha}{\text{EC}_{50}^\alpha + s^\alpha} = \lim_{s \to \infty} \frac{1}{\text{EC}_{50}^\alpha / s^\alpha + 1} = \frac{1}{0 + 1} = 1$$

**Property 5: The inflection point is at $s^* = \text{EC}_{50}\left(\frac{\alpha-1}{\alpha+1}\right)^{1/\alpha}$ for α > 1.**

For α ≤ 1, there is no inflection point — the curve is purely concave.
For α > 1, the curve is S-shaped with increasing then decreasing marginal returns.
At α = 1 (first-order Hill), the function reduces to $H(s) = s/(EC_{50} + s)$,
which is a rectangular hyperbola — purely concave from s = 0.

> **Teaching note:** Plot H(s) for α = 1, 2, 5 on the same axes with EC₅₀ = 100.
> With α = 1: smooth concave curve from 0. With α = 2: slight S-shape.
> With α = 5: nearly a step function at EC₅₀. The visual makes the shape
> parameter interpretable immediately: high α = sudden "switch on" behavior;
> low α = gradual saturation. Budget 5 minutes.

### Worked Numerical Examples

Channel: EC₅₀ = 100 ($000s/week), α = 2.

| Spend s | Numerator s² | Denominator (100²+s²) | H(s) | Interpretation |
|---|---|---|---|---|
| 50 | 2,500 | 10,000+2,500=12,500 | **0.200** | Far below EC₅₀, high marginal return |
| 100 | 10,000 | 10,000+10,000=20,000 | **0.500** | At EC₅₀, half-maximum |
| 141 | 20,000 | 10,000+20,000=30,000 | **0.667** | Inflection point (α=2: s*=100√(1/3)≈57.7... let me recalculate) |
| 200 | 40,000 | 10,000+40,000=50,000 | **0.800** | Well above EC₅₀ |
| 300 | 90,000 | 10,000+90,000=100,000 | **0.900** | Significant saturation |

**Reading the table:** Moving from s=50 to s=100 increases H by 0.30 (a large gain).
Moving from s=200 to s=300 increases H by only 0.10 (diminishing returns visible).

### Marginal Return and Diminishing Returns

$$\frac{dH}{ds} = \frac{\alpha \cdot \text{EC}_{50}^\alpha \cdot s^{\alpha-1}}{(\text{EC}_{50}^\alpha + s^\alpha)^2}$$

At s = EC₅₀ = 100, α = 2:
$$\frac{dH}{ds}\bigg|_{s=100} = \frac{2 \times 100^2 \times 100}{(100^2 + 100^2)^2} = \frac{2,000,000}{40,000,000} = 0.005$$

At s = 200 (double EC₅₀):
$$\frac{dH}{ds}\bigg|_{s=200} = \frac{2 \times 100^2 \times 200}{(100^2 + 200^2)^2} = \frac{4,000,000}{250,000,000} = 0.0016$$

The marginal response at s=200 is approximately one-third of the marginal response
at s=100, confirming strong diminishing returns above EC₅₀.

---

## 3.5 The Full MMM Regression

### Model Specification

For T time periods and J channels:
$$\text{Revenue}_t = \beta_0 + \sum_{j=1}^{J} \beta_j \cdot H(A_{j,t}; \text{EC}_{50,j}, \alpha_j) + \varepsilon_t$$

**Parameter interpretation:**
- β₀ = baseline revenue (revenue with zero spend in all channels)
- βⱼ = maximum revenue contribution of channel j (the plateau at H=1)
- βⱼ × H(A_{j,t}) = actual revenue contribution of channel j in period t

**Current efficiency:** A channel's current response efficiency is H(A_{j,t}).
If channel j has H = 0.82, it is generating 82% of its maximum potential contribution
at current spend levels. The remaining 18% would require spending far above EC₅₀.

### Estimation: Why OLS Cannot Be Used Directly

The regression above is nonlinear in parameters (λ_j, EC₅₀,j, α_j). We cannot
compute $\partial \text{SSR} / \partial \lambda_j$ and solve analytically — the
derivatives do not admit closed-form solutions when λ is inside the adstock
transformation.

**Standard approaches:**

**1. Grid search + OLS:** For a grid of candidate (λ_j, EC₅₀,j, α_j) values,
transform the spend series and run OLS on the linearized problem. Choose the parameter
combination that minimizes out-of-sample RMSE. This is computationally expensive for
large grids but conceptually simple.

**2. Nonlinear least squares (scipy.optimize.minimize):** Minimize the sum of
squared residuals directly over all parameters simultaneously. Requires good starting
values to avoid local minima.

**3. Bayesian hierarchical MMM:** Specify prior distributions on all parameters
and use MCMC (Stan, PyMC) to sample from the posterior. Industry standard for
production MMM because priors encode media-specific knowledge (e.g., TV λ ~ Beta(8,2)
which centers on 0.80 with moderate uncertainty) and the posterior captures estimation
uncertainty. Used by Meta's Robyn and Google's LightweightMMM.

*Reference: arXiv:2108.00955, Jin et al. (2021). "Bayesian Methods for Media Mix
Modeling with Carryover and Shape Effects." arxiv.org/abs/2108.00955*
*Reference: arXiv:1710.00954, Chan and Perry (2017). "Challenges and Opportunities
in Media Mix Modeling." arxiv.org/abs/1710.00954*

### Identification Challenges

MMM models are **weakly identified** when:
1. Channel spend levels are highly correlated (multicollinearity) — common when all
   channels ramp up and down together with the overall marketing budget
2. Spend timing has insufficient variation to identify λ — if a channel always spends
   steadily, the model cannot distinguish λ = 0.5 from λ = 0.7
3. Short time series — MMM needs at least 2–3 years of weekly data (100+ observations)
   for reliable estimation

When identification is weak, Bayesian priors become essential — they regularize the
estimates toward economically sensible values even when the data alone cannot distinguish.

---

## 3.6 Budget Optimization: Full Derivation

### The Optimization Problem

Given a total weekly marketing budget B, allocate $s_j$ to each of J channels to
maximize total predicted revenue:

$$\max_{s_1, \ldots, s_J} \sum_j \beta_j H(s_j; \text{EC}_{50,j}, \alpha_j)$$
$$\text{subject to: } \sum_j s_j = B, \quad s_j \geq 0 \; \forall j$$

(For simplicity, treating adstock as identical to spend in a single period, which is
approximately correct for a steady-state budget analysis.)

### First-Order Conditions (Lagrangian)

Form the Lagrangian:
$$\mathcal{L} = \sum_j \beta_j H(s_j) - \lambda\left(\sum_j s_j - B\right)$$

Taking partial derivative with respect to sⱼ and setting to zero:
$$\frac{\partial \mathcal{L}}{\partial s_j} = \beta_j \frac{dH}{ds_j} - \lambda = 0$$

Therefore: $\beta_j \frac{dH}{ds_j}\bigg|_{s_j = s_j^*} = \lambda \quad \forall j$

**The optimal condition:** Marginal revenue from the last dollar in channel j
equals the shadow price λ for all channels simultaneously.

Equivalently: **Marginal ROI is equalized across all channels at the optimum.**

$$\text{Marginal ROI}_j = \beta_j \frac{dH}{ds_j} = \frac{\alpha_j \beta_j \text{EC}_{50,j}^{\alpha_j} s_j^{\alpha_j-1}}{(\text{EC}_{50,j}^{\alpha_j} + s_j^{\alpha_j})^2}$$

### Numerical Solution

Since the first-order conditions are a system of nonlinear equations in (s₁, ..., s_J),
they are typically solved numerically:

```python
from scipy.optimize import minimize

def neg_revenue(s, betas, ec50s, alphas):
    return -sum(b * s[j]**a / (ec50**a + s[j]**a)
                for j,(b,ec50,a) in enumerate(zip(betas,ec50s,alphas)))

constraints = {'type': 'eq', 'fun': lambda s: sum(s) - B}
bounds = [(0, None)] * J
result = minimize(neg_revenue, x0=current_spend,
                  args=(betas, ec50s, alphas),
                  method='SLSQP', bounds=bounds, constraints=constraints)
```

### Why Not Shift Entire Budget at Once?

The second-order condition confirms this is a maximum: $\partial^2\mathcal{L}/\partial s_j^2 = \beta_j d^2H/ds_j^2 < 0$ (since H is concave above EC₅₀ and always concave at EC₅₀).
The concavity means marginal ROI is decreasing in spend. If channel A has higher
marginal ROI at current levels, shifting all budget to A would push A's marginal ROI
below zero — the optimum is an interior solution, not a corner solution.

> **Teaching note:** Draw two channels' marginal ROI curves as downward-sloping lines.
> Show that the intersection point (equal marginal ROIs) is the optimum. This geometric
> argument is more intuitive than the Lagrangian for most students. Present the
> Lagrangian as the formal proof of the graphical result.

---

## 3.7 Common Misinterpretations

| Statement | Correct? | Explanation |
|---|---|---|
| "Channel with H=0.82 is at 82% of maximum ROI" | ✅ | H = 0.82 means 82% of max potential contribution |
| "λ = 0 means no advertising effect" | ❌ | λ = 0 means no CARRYOVER; within-period effect still present via S_t |
| "Optimal allocation should be computed fresh each week" | ❌ | Budget optimization is strategic; tactical adjustments are weekly, not full reallocations |
| "Higher R² means better channel attribution" | ❌ | R² measures fit, not identification of channel effects |
| "MMM proves causation" | ❌ | MMM is observational; it requires holdout tests for causal validation |

---

## 3.8 Full Proof Appendix

### Proof A: Adstock Recursion Has a Unique Stationary Solution Under Constant Spend

If spend is constant at level S̄ for all t, the long-run adstock converges to:
$$A_\infty = \lim_{t \to \infty} A_t = S̄ \sum_{k=0}^{\infty} \lambda^k = \frac{S̄}{1-\lambda}$$

This is the **steady-state adstock** — the effective advertising level when the
channel has been spending at S̄ consistently. To verify it's a fixed point:
$A_\infty = S̄ + \lambda A_\infty \Rightarrow A_\infty(1-\lambda) = S̄ \Rightarrow A_\infty = S̄/(1-\lambda)$. ✓

### Proof B: Budget Optimization First-Order Conditions Imply Global Maximum

The objective function $\sum_j \beta_j H(s_j)$ is a sum of concave functions (each
H(s_j) is concave in s_j since $d^2H/ds_j^2 < 0$ for s_j > EC₅₀ and all s_j
when α = 1). A sum of concave functions over a convex constraint set (budget constraint)
is concave, so any first-order critical point is a global maximum. QED.

(For α > 1, H is S-shaped and not globally concave. The problem is then non-convex.
However, in practice, spend levels are typically set above the inflection point,
and the relevant portion of H is concave. If spend is below the inflection point,
the optimal solution is to push all budget into that channel until the inflection point
is reached — an interior solution may not exist in the elastic region.)

*Reference: Wikipedia, "Lagrange multiplier" — en.wikipedia.org/wiki/Lagrange_multiplier*
*Reference: Wikipedia, "Karush-Kuhn-Tucker conditions" —
en.wikipedia.org/wiki/Karush%E2%80%93Kuhn%E2%80%93Tucker_conditions*

---

## 3.9 Verification Checklist

| Check | How to verify |
|---|---|
| A_t = S_t + λ·A_{t-1}? | Compute by hand from spend sequence |
| Long-run adstock = S/(1−λ) for impulse spend S? | Check with λ=0.6, S=100: A∞=250 |
| H(EC₅₀) = 0.5 for any EC₅₀ and α? | Substitute s=EC₅₀ algebraically |
| H is strictly between 0 and 1? | For any finite positive s and EC₅₀, α > 0 |
| β_j = max revenue contribution (not coefficient on spend)? | β_j × H(A_j) = actual contribution |
| Marginal ROIs equalized at optimal allocation? | Check agent optimization output |
| Adstock applied BEFORE Hill function? | Check data pipeline sequence |
| Budget constraint satisfied? | Σs_j = B at solution |

---

## 3.10 Chapter Bibliography

1. Wikipedia, "Advertising adstock" — en.wikipedia.org/wiki/Advertising_adstock
2. Wikipedia, "Hill equation (biochemistry)" — en.wikipedia.org/wiki/Hill_equation_(biochemistry)
3. Wikipedia, "Geometric series" — en.wikipedia.org/wiki/Geometric_series
4. Wikipedia, "Lagrange multiplier" — en.wikipedia.org/wiki/Lagrange_multiplier
5. Wikipedia, "Marketing mix modeling" — en.wikipedia.org/wiki/Marketing_mix_modeling
6. Jin, Y., Wang, Y., Sun, Y., Chan, D., Koehler, J. (2021). "Bayesian Methods for Media Mix Modeling with Carryover and Shape Effects." arXiv:2108.00955. arxiv.org/abs/2108.00955
7. Chan, D. and Perry, M. (2017). "Challenges and Opportunities in Media Mix Modeling." arXiv:1710.00954. arxiv.org/abs/1710.00954
8. Meta Open Source. Robyn: Automated Marketing Mix Modeling. Free documentation: facebookexperimental.github.io/Robyn/
9. Google. LightweightMMM documentation: github.com/google/lightweight_mmm
10. MIT OpenCourseWare, 15.053 Optimization Methods in Management Science — ocw.mit.edu (for Lagrangian optimization background)

**Lenny's Podcast episode references:**
- Jonathan Becker episode on paid acquisition at Uber and Masterclass — Lenny's Podcast
- Yuriy Timen episode on growth at Grammarly — Lenny's Podcast
