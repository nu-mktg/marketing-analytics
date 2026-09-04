# Lecture 3: Marketing Mix Modeling
## How to Measure What Your Marketing Actually Does

---

### Overview

**Business question:** You spend $2M/year across TV, digital, and social. Which channel is actually driving revenue — and how should you reallocate?

**What you will be able to do:**
- Apply the adstock transformation to model advertising carryover
- Evaluate a channel's current efficiency using the Hill function
- Interpret MMM outputs: decay parameters, EC50, marginal ROI
- Recommend budget reallocations from model outputs
- Identify why last-touch attribution misleads budget decisions

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Geometric Series Sum

When you multiply the same number by itself repeatedly, the sum can be calculated compactly.

If λ = 0.7 and you spent $100 in week 1 only, the carryover in weeks 2, 3, 4 is:
> Week 2: 0.7 × 100 = 70
> Week 3: 0.7 × 70 = 49
> Week 4: 0.7 × 49 = 34.3

Total long-run effect = 100 + 70 + 49 + 34.3 + ... = 100 / (1 − 0.7) = **333.3**

**Formula:** Sum = Initial / (1 − λ), valid when 0 < λ < 1.

---

#### Tool 2: Fraction Arithmetic

The Hill function produces numbers between 0 and 1. You need to compare fractions:

> H(s) = s² / (EC50² + s²)

When s = EC50: H(EC50) = EC50² / (EC50² + EC50²) = EC50² / (2 × EC50²) = **1/2 = 0.5**

This is true regardless of what EC50 equals — the numerator and denominator are always equal at s = EC50.

---

### Section 1.2 — Business Motivation

---

#### The Attribution Problem

If a customer buys after clicking a paid search ad, last-touch attribution gives 100% of the credit to paid search. But they may have first learned about your brand through a TV spot, then seen a social post three times before finally clicking the paid search ad.

Jonathan Becker, who has managed over $3.5 billion in paid acquisition:

> *"Paid search captures demand. It does not create it."*

A campaign that shifts budget from TV to paid search may look successful in the short term — paid search conversions appear to increase. But you are harvesting awareness that TV built. Over time, organic awareness declines and paid search efficiency falls.

Marketing Mix Modeling is the analytical response to this problem: it models all channels simultaneously, accounting for their different timing and saturation dynamics.

---

### Section 1.3 — Conceptual Framework

---

#### The Two Core Nonlinearities

MMM handles two effects that make simple regression inadequate:

**1. Carryover (Adstock):** Advertising effects do not disappear instantly. A TV spot seen on Monday influences purchase decisions through the week. The adstock transformation models this decay explicitly.

**2. Diminishing Returns (Saturation):** The first $10,000 spent on a channel has more impact than the $100,001st dollar. Doubling spend does not double revenue. The Hill function models this concave relationship.

Understanding both is essential for budget optimization: you want to find the allocation where marginal ROI across all channels is equalized — the point where moving one more dollar from any channel to another cannot improve total revenue.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: Adstock — Modeling Carryover

The adstock value at time t is:

$$A_t = S_t + \lambda \cdot A_{t-1}$$

where S_t is spend in period t and λ (lambda) is the decay parameter (0 < λ < 1).

**Plain English:** This week's advertising effect equals what you spent this week, plus a fraction λ of last week's effect that is still carrying over. Higher λ = slower decay = longer-lasting effects.

**Worked example** (λ = 0.6, S = [100, 0, 50]):
- A₁ = 100 + 0.6 × 0 = **100**
- A₂ = 0 + 0.6 × 100 = **60** ← no spend, but carryover remains
- A₃ = 50 + 0.6 × 60 = **86**

**Comparing channels:**
- λ_TV = 0.82 → TV effects last many weeks after a campaign ends
- λ_Paid_Search = 0.19 → Paid search effects dissipate quickly (makes sense: captures in-the-moment intent)

> ### 🔍 Deep Dive: Why Geometric Decay?
> Adstock uses geometric decay for mathematical tractability. The total long-run effect of a single-period spend S is S × 1/(1−λ). This means a channel with λ = 0.8 generates 5× the immediate spend in long-run effect (1/(1−0.8) = 5). Geometric decay also makes the adstock recursion computationally trivial — it requires only the previous period's value, not the full history.

---

#### Part B: Hill Function — Modeling Saturation

The Hill function maps spend to effectiveness on a 0-to-1 scale:

$$H(s) = \frac{s^\alpha}{EC_{50}^\alpha + s^\alpha}$$

where EC50 is the spend level at which the channel is at 50% of its maximum effectiveness, and α controls how quickly diminishing returns set in.

**Key property:** H(EC50) = 0.5 always. This is because at s = EC50, the numerator and denominator are equal (EC50^α / 2×EC50^α = 0.5).

**Interpretation:**
- H < 0.5: current spend is below EC50 — high marginal return, increasing spend is efficient
- H = 0.5: at EC50 — the channel is at half its maximum effect
- H > 0.5: above EC50 — diminishing returns have set in, marginal ROI is falling

> ### 🔍 Deep Dive: Why H(EC50) = 0.5 Regardless of α
> Substituting s = EC50: H(EC50) = EC50^α / (EC50^α + EC50^α) = EC50^α / (2 × EC50^α) = 1/2. The α terms cancel completely. This is a mathematical identity, not an approximation.

---

#### Part C: The MMM Regression

Once we compute adstock and apply the Hill function, we fit:

$$\text{Revenue}_t = \beta_0 + \beta_{TV} H(A_{TV,t}) + \beta_{Dig} H(A_{Dig,t}) + \beta_{Soc} H(A_{Soc,t}) + \varepsilon_t$$

The β coefficients capture the maximum revenue each channel can generate (the plateau). The Hill functions scale that maximum by current efficiency.

**Budget optimization:** Compute marginal ROI for each channel at current spend. The rule: shift budget from lower marginal ROI channels to higher ones, one step at a time. Stop when all marginal ROIs are equal — that is the optimal allocation.

---

### Part 1 Checkpoint

1. With λ = 0.6, spend sequence S₁=200, S₂=0, S₃=50. Compute A₁, A₂, A₃.
2. A Digital channel has H = 0.73. Is current spend above or below EC50?
3. TV marginal ROI = $4.20/dollar. Digital marginal ROI = $1.80/dollar. In which direction should budget move? Can you shift the entire digital budget to TV immediately?
4. True or False: λ = 0 means advertising has no effect on sales.
5. A model shows TV's adstock decay parameter λ = 0.82. What does this tell you about how TV campaigns should be scheduled compared to paid search (λ = 0.19)?

---

### Checkpoint Answer Key

**Q1.** A₁ = 200 + 0 = 200. A₂ = 0 + 0.6×200 = 120. A₃ = 50 + 0.6×120 = 50 + 72 = **122**.
*Common wrong answer:* A₂ = 0 (forgetting carryover when no spend occurs). The recursion carries forward λ × previous adstock regardless of current spend.

**Q2.** **Above EC50.** H = 0.73 > 0.5, and H > 0.5 if and only if current spend > EC50. The channel is on the diminishing-returns portion of the curve.
*Common wrong answer:* "H = 0.73 means we're at 73% efficiency, which is below the maximum." True, but the relevant threshold for the EC50 question is H = 0.5, not H = 1.0.

> **Also asked on the slides:** *"Channel EC₅₀ = 200, α = 2. Compute H(100), H(200), H(400)."* — Substituting into H(s) = sᵃ / (EC50ᵃ + sᵃ) with α = 2:
> H(100) = 100² / (200² + 100²) = 10,000 / 50,000 = **0.20**;
> H(200) = 200² / (200² + 200²) = **0.50** (the H(EC50) = 0.5 property, true for any EC50);
> H(400) = 400² / (200² + 400²) = 160,000 / 200,000 = **0.80**.
> Doubling spend from 100 to 200 buys +0.30 of effectiveness; doubling again from 200 to 400 buys only +0.30 → +0.80, i.e. the same 0.30 for twice the incremental spend. That is saturation, and it is why the answer to Q3 is "shift incrementally," not "shift everything."

**Q3.** Move budget from Digital (lower marginal ROI) to TV (higher). But **not the entire Digital budget at once**. As TV spend increases, TV marginal ROI falls along the Hill curve. The equalization principle says shift incrementally until marginal ROIs are equal.
*Common wrong answer:* "Shift everything to TV since TV ROI is higher." Diminishing returns mean the last dollar moved to TV has a much lower marginal ROI than the first dollar.

> **Also asked on the slides:** *"Last-touch attribution shows Digital = 70% of conversions, TV = 10%. MMM shows TV = 35% contribution. Which is right — and why might they differ?"* — Neither is "wrong"; they measure different things, and for a budget decision the MMM figure is the relevant one. Last-touch gives 100% of the credit to whatever the customer clicked last, so it systematically over-credits demand-harvesting channels and under-credits demand-creating ones: as Section 1.2 puts it: "Paid search captures demand. It does not create it." A customer may have learned about the brand from a TV spot, seen social posts, then clicked a digital ad — last touch records only the click. MMM models all channels simultaneously with their own adstock and saturation, so TV's carryover (high λ) shows up as contribution that last touch cannot see. The practical warning in Section 1.2 is exactly this case: shifting budget from TV to digital on last-touch evidence looks successful short-term while you are harvesting awareness TV built.

**Q4.** **False.** λ = 0 means instant decay — each period's advertising effect disappears completely before the next period. The channel can still have immediate in-period impact (S_t contributes fully to A_t). λ = 0 means no carryover, not no effect.
*Common wrong answer:* True. Confusing "no carryover" with "no effect."

**Q5.** TV campaigns should be **spaced further apart** than paid search campaigns. High λ = slow decay = the effects of one TV campaign persist for weeks. You don't need to run TV every week to maintain impact. Paid search (λ = 0.19) decays quickly — its effects are mostly exhausted within 1–2 weeks, so continuous presence is needed.
*Common wrong answer:* "TV should run continuously since it has a higher λ." High λ means the campaign persists longer, so you can run *less frequently* and still maintain cumulative impact.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Parts A–B first, then reveal)


---

**Problem Statement**

Weekly TV spend ($000s): $S_1 = 100$, $S_2 = 0$, $S_3 = 150$, $S_4 = 80$. Decay: $\lambda = 0.6$. Hill parameters: EC50 = 120, $\alpha = 2$.

**Part A:** Compute $A_1, A_2, A_3, A_4$.

**Part B:** Compute $H(A_3)$ and $H(A_4)$.

**Part C:** Repeat Part A with $\lambda_D = 0.2$ (Digital channel, same spend). Compare adstock values and explain the difference.

---

**Full Solution**

**Part A — TV Adstock ($\lambda = 0.6$)**

$$A_1 = 100 + 0.6 \times 0 = \mathbf{100.0}$$
$$A_2 = 0 + 0.6 \times 100 = \mathbf{60.0}$$
$$A_3 = 150 + 0.6 \times 60 = 150 + 36 = \mathbf{186.0}$$
$$A_4 = 80 + 0.6 \times 186 = 80 + 111.6 = \mathbf{191.6}$$

Even in week 2 (no spend), adstock = 60 — the memory of the prior campaign. By week 3, adstock reaches 186 though only \$150k was spent, because it carries forward week-1 and week-2 memory.

**Part B — Hill Responses**

At $A_3 = 186$, EC50 = 120, $\alpha = 2$:

$$H(186) = \frac{186^2}{120^2 + 186^2} = \frac{34{,}596}{14{,}400 + 34{,}596} = \frac{34{,}596}{48{,}996} \approx \mathbf{0.706}$$

At $A_4 = 191.6$:

$$H(191.6) = \frac{191.6^2}{120^2 + 191.6^2} = \frac{36{,}711}{14{,}400 + 36{,}711} = \frac{36{,}711}{51{,}111} \approx \mathbf{0.718}$$

Both exceed 0.5, meaning adstock is above EC50 in both weeks — TV is operating on the flattening (diminishing-returns) portion of the curve.

**Sanity check:** $H(120) = 120^2/(120^2 + 120^2) = 1/2 = 0.5$ ✓

**Part C — Digital Adstock ($\lambda_D = 0.2$)**

$$A_1^D = 100, \quad A_2^D = 0 + 0.2 \times 100 = 20, \quad A_3^D = 150 + 0.2 \times 20 = 154, \quad A_4^D = 80 + 0.2 \times 154 = 110.8$$

| Week | TV ($\lambda=0.6$) | Digital ($\lambda=0.2$) |
|---|---|---|
| 1 | 100.0 | 100.0 |
| 2 | 60.0 | 20.0 |
| 3 | 186.0 | 154.0 |
| 4 | 191.6 | 110.8 |

Digital drops far faster in week 2 (100 → 20 vs. 100 → 60 for TV). TV accumulates more memory, producing higher week-4 adstock despite identical raw spending.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**Decay parameters (λ per channel):** Plausible ranges:
- TV, radio: 0.6–0.9
- Digital display, paid social: 0.3–0.6
- Paid search: 0.1–0.3

If paid search shows $\lambda = 0.85$, investigate — this likely reflects a data or model problem.

**EC50 values:** Compare to the actual **adstock** range in your data — adstock, not raw spend, is what the Hill function takes as input, and adstock is always the larger of the two. If EC50 is far above your maximum adstock, you are far from saturation and still on the high-marginal-return part of the curve. If EC50 is below your minimum adstock, you are already past the half-saturation point.

**β coefficients:** The maximum revenue attributable to each channel. A channel with β = \$500k can contribute at most \$500k in weekly revenue (when its Hill response approaches 1).

**Budget recommendation:** Always check: is the recommended reallocation extreme? A recommendation to shift 80% of budget to one channel should trigger scrutiny. Verify that the predicted revenue improvement is meaningful and that the model fit is good before acting on it.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer - tools/render_lecture_homework.py; do not hand-edit.
     Replaces a ~150-line duplicate of the homework that had drifted into a DIFFERENT
     assignment (Task 004, 2026-08-13): for lectures 04-10 only 1-3 of ~16 question
     variables still matched the notebook, and the dataset filenames were wrong.
     The notebook is the assignment of record; answers live only in answer_keys/. -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_03_mmm.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_03_mmm.ipynb` |
| Dataset | `homework_datasets/mmm_data.csv` |
| Graded questions | **17** — Part A: 10 · Part B: 4 · Part C: 3 |
| Answer key (instructor only) | `answer_keys/hw03.json` |

Answers and tolerances are never duplicated outside `answer_keys/hwNN.json`
(rendered for instructors as `quiz/answer_key_values.md`).

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "Adstock replaces raw spend as the input — the model ignores current spend."**
No. Adstock includes current spend plus a weighted sum of all past spend. Current spend always enters adstock with full weight ($\lambda^0 = 1$). Adstock is an enrichment of the raw spend variable, not a replacement.

**2. "A higher λ always means the channel is more effective."**
λ is a decay rate, not an effectiveness measure. TV might have high λ (slow decay) and still generate less revenue than paid search with low λ (fast decay) if paid search has higher β or lower EC50.

**3. "The Hill function directly gives the revenue contribution of a channel."**
No. The Hill function returns a number between 0 and 1. To get revenue, multiply by β. The Hill function only tells you what proportion of the channel's maximum contribution is being achieved at the current adstock level.

**4. "Budget optimization tells us how much to spend in total."**
Budget optimization tells you how to allocate a fixed total budget — it does not tell you whether to spend more or less overall. That requires a separate analysis comparing the marginal ROI of the best channel to the cost of capital.

**5. "If marginal ROI is equal across channels, the model is wrong."**
Equal marginal ROI across channels is exactly the optimal condition. The model is working correctly if the optimizer converges to a point where all channels have the same marginal ROI.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Computes $A_t = S_t + \lambda A_{t-1}$ | Section 1.4A — adstock recursion |
| Applies $H(A; \text{EC50}, \alpha)$ | Section 1.4B — Hill function |
| Fits regression on transformed variables | Lecture 2 — OLS on $\beta_c \cdot H(A_{c,t})$ |
| Reports fitted $\lambda$, EC50, $\alpha$ per channel | Sections 1.3, 1.4A, 1.4B — parameter interpretation ($\lambda$ in 1.4A) |
| Computes optimal budget allocation | Section 1.4C — equalize marginal ROI |

**What to verify in the agent output:**
1. All λ values between 0 and 1
2. EC50 values in plausible range relative to actual spend
3. Predicted revenue at current spend ≈ actual average revenue (model fit sanity check)
4. Budget recommendation moves spending toward the channel with the higher **marginal ROI** — β × the Hill slope at its current adstock — and not simply toward whichever channel sits lower relative to its own EC50. A large β can outweigh being further up the saturation curve.
