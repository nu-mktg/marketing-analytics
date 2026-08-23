# Lecture 12: Difference-in-Differences
## Causal Identification of Marketing Interventions

---

### Overview

**Motivating Problem:** A subscription service ran a 15% price reduction in four geographic regions starting week 11, while four other regions maintained the original price. Revenue rose in the treatment regions. Did the price reduction cause the increase, or would revenue have risen anyway?

**Learning Objectives:**

- State the parallel trends assumption and explain when it is and is not plausible
- Compute the 2×2 DiD estimator by hand from a two-period, two-group design
- Write the DiD regression with an interaction term and interpret all four coefficients
- Run a pre-trend test and a placebo test using the pre-period data
- Connect DiD to the geo-holdout validation of MMM estimates

**Prerequisites:** OLS from Lecture 2. No prior causal inference background needed.

---

## PART 1: Concepts and Mathematics

### (~1 hour 40 minutes)

---

### Section 1.1 — The Counterfactual Problem

#### What would have happened without treatment?

A subscription firm cuts prices in the Northwest region starting March 1st. Revenue grows. Three candidate explanations:

1. The price cut caused revenue to grow (causal effect)
2. Revenue was already growing in the Northwest for unrelated reasons (time trend)
3. Both

We want to separate explanation 1 from explanations 2 and 3. To do this, we need to know what would have happened in the Northwest **absent** the price cut — the **counterfactual**. We cannot observe this directly.

**Naive estimate (wrong):**

$$\hat{\tau}_{\text{naive}} = \bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}}$$

This captures both the causal effect and any pre-existing time trend. It will be biased.

---

### Section 1.2 — The Difference-in-Differences Estimator

**Key idea:** Use the *control region's* time trend as an estimate of what the treatment region's trend would have been absent the intervention.

**Data structure:**

| Group | Before intervention | After intervention |
|---|---|---|
| Treatment | $\bar{Y}_{T,\text{pre}}$ | $\bar{Y}_{T,\text{post}}$ |
| Control | $\bar{Y}_{C,\text{pre}}$ | $\bar{Y}_{C,\text{post}}$ |

$$\boxed{\hat{\tau}_{\text{DiD}} = (\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}}) - (\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}})}$$

The first difference removes the common time trend. The second difference removes any pre-existing level difference between treatment and control. Hence: *difference-in-differences*.

---

### Section 1.3 — Worked Example (from dataset)

Using aggregate means from `geo_experiment.csv`:

| Period | Treatment regions | Control regions |
|---|---|---|
| Pre (weeks 1–10) | 146.39 | 137.02 |
| Post (weeks 11–20) | 156.47 | 140.84 |

**Treatment region change:** 156.47 − 146.39 = **+10.08**

**Control region change (common trend):** 140.84 − 137.02 = **+3.83**

$$\hat{\tau}_{\text{DiD}} = 10.08 - 3.83 = \mathbf{+6.25}$$

**Interpretation:** The price reduction caused a 6.25-unit increase in weekly revenue, after removing the 3.83-unit increase that would have occurred due to the underlying time trend alone.

**Overstatement of the naive estimate:** 10.08 − 6.25 = **3.83 units**.

> **Convention — what "overstates by" means in this course.** Overstatement is a **difference, in the outcome's own units**:
>
> $$\text{overstatement} = \hat{\tau}_{\text{naive}} - \hat{\tau}_{\text{DiD}}$$
>
> That difference *is* the common time trend $\beta_2$. The trend is exactly what the naive estimate fails to subtract, so what is left over is the trend itself.
>
> Two other quantities are true of the example above but are **not** what "overstates by" means. A **percentage**: the difference as a share of the causal effect, **61%** here, so the naive estimate is 61% larger than the truth. A **ratio**: $\hat{\tau}_{\text{naive}} / \hat{\tau}_{\text{DiD}} = 1.61$.
>
> **Part A Q3 (`q3_overstatement`) wants the difference, in units, computed from your own given constants.** Every student's constants differ, so no figure from this worked example is your answer.

---

### Section 1.4 — The Regression Formulation

The 2×2 DiD is a special case of OLS with an interaction term:

$$Y_{it} = \beta_0 + \beta_1 \cdot \text{Treat}_i + \beta_2 \cdot \text{Post}_t + \beta_3 \cdot (\text{Treat}_i \times \text{Post}_t) + \varepsilon_{it}$$

| Coefficient | Interpretation | Value from data |
|---|---|---|
| $\beta_0$ | Control region, pre-period baseline | 137.02 |
| $\beta_1$ | Pre-existing level difference, treatment vs. control | 9.38 |
| $\beta_2$ | Common time trend (control region change) | 3.83 |
| $\boldsymbol{\beta_3}$ | **Causal effect of treatment (DiD estimator)** | **6.25** |

The treatment effect is always the coefficient on the **interaction term** $\text{Treat}_i \times \text{Post}_t$.

---

### Section 1.5 — The Parallel Trends Assumption

DiD is valid only under the parallel trends assumption:

$$\boxed{E[\bar{Y}_{T,\text{post}}^{(0)} - \bar{Y}_{T,\text{pre}}] = E[\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}}]}$$

**Reading the superscript.** $\bar{Y}_{T,\text{post}}$ — no superscript — is the treatment regions' *observed* mean after the intervention: a column of numbers you can compute from `geo_experiment.csv`. The superscript $(0)$ marks a **potential outcome under no treatment**, so $\bar{Y}_{T,\text{post}}^{(0)}$ is the treatment regions' post-period mean *in the world where the price cut never happened*. That world did not occur, so **no dataset contains $\bar{Y}_{T,\text{post}}^{(0)}$** — not this one, and not a larger one. These are two different quantities for the same regions in the same weeks, and the gap between them is precisely the thing we are trying to estimate: $\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{post}}^{(0)}$. ($\bar{Y}_{T,\text{pre}}$ needs no superscript: before the intervention the treated and untreated worlds are the same world.)

**Why that makes it an assumption.** Every term in the boxed equation is observable *except* $\bar{Y}_{T,\text{post}}^{(0)}$. Parallel trends is therefore a constraint on a quantity that is missing by construction — you cannot check it against data the way you check a regression coefficient, and no sample size fixes that. What the assumption buys is a **substitution**: it licenses using the control regions' observed change $(\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}})$ in place of the treatment regions' unobservable counterfactual change. When it holds, DiD is causal; when it fails, DiD is a subtraction with no causal content, and the data look identical either way.

So the two procedures below can never *verify* parallel trends. They ask a weaker question that data can answer — in the pre-period, where *both* groups' outcomes are observed, did the two move together? — and a yes makes the substitution more credible. Plausibility is the most any evidence can buy here.

**How to assess parallel trends:**

1. **Pre-trend plot:** Plot treatment and control group trajectories for several periods *before* the intervention. If they move together, parallel trends is more credible.

2. **Placebo test:** Define a fake intervention date in the pre-period. Run the DiD regression on pre-period data only. If $\hat{\beta}_3 \approx 0$ and $p > 0.05$, there is no evidence of a differential pre-trend.

**From the dataset:** Placebo $\hat{\beta}_3 = 0.81$, $p = 0.86$. No evidence of pre-treatment differential trend. Parallel trends is plausible.

---

### Section 1.6 — Threats to Parallel Trends

| Threat | Description | Detection |
|---|---|---|
| **Differential pre-trend** | Treatment region was already growing faster | Visual inspection; placebo test |
| **Anticipation effect** | Treatment region changed behaviour before the formal intervention date | Check pre-period data near intervention |
| **Contemporaneous shock** | Something else changed in the treatment region at the same time | Domain knowledge; cross-check with other outcomes |
| **Spillover** | Control region is affected by the intervention (e.g., consumers travel between regions) | Check for effects in "pure control" sub-regions |

---

### Section 1.7 — DiD as MMM Validation

Marketing Mix Models estimate channel contributions from observational data. Because prices, budgets, and demand are often correlated (endogeneity), MMM estimates may not be causal.

**The standard validation procedure:**

1. **Design a geo-holdout:** Randomly assign regions to treatment (change the channel spend) and control (maintain existing spend). Run for a fixed period.

2. **Estimate the DiD effect:** Compare outcome in treatment vs. control regions, before and after the spend change. This $\hat{\tau}_{\text{DiD}}$ is a causal estimate.

3. **Compare to MMM prediction:** The MMM predicts what the revenue change should be. If MMM and DiD agree, the MMM estimate is credible. If they disagree substantially, the MMM is likely confounded.

**Key insight:** MMM says what *would* happen. DiD measures what *did* happen. Without geo-holdout validation, MMM channel contributions remain observational claims.

---

### Part 1 Checkpoint

1. Treatment region revenue: pre = 180, post = 200. Control region: pre = 160, post = 172. Compute $\hat{\tau}_{\text{DiD}}$.

2. In question 1, what is the naive estimate, and by how much does it overstate the causal effect?

3. Write the regression equation for DiD. Which coefficient is the treatment effect?

4. You plot weekly revenue for treatment and control regions in the 8 weeks before the intervention. The treatment regions trend at +5.2 units/week; the control regions trend at +3.1 units/week. Is parallel trends plausible, and what does your answer imply for the DiD estimate?

5. Your MMM estimates the causal effect of the price reduction at +10.5 units/week. Your geo-holdout DiD estimates +6.25 units/week. What does this discrepancy suggest about the MMM estimate?

---

### Checkpoint Answer Key

**Q1.** $\hat{\tau}_{\text{DiD}} = (200-180) - (172-160) = 20 - 12 = \mathbf{+8}$

**Q2.** Naive estimate = 200 − 180 = **20**. By §1.3's convention the overstatement is the difference, in units: 20 − 8 = **12** — which is exactly the common trend (172 − 160 = 12), as it always is. As a fraction of the causal effect that is 12/8 = 150%, and the ratio 20/8 = 2.5× is the same fact in a third form. **The answer is 12.**

**Q3.** $Y_{it} = \beta_0 + \beta_1 \text{Treat}_i + \beta_2 \text{Post}_t + \beta_3(\text{Treat}_i \times \text{Post}_t) + \varepsilon_{it}$. The treatment effect is $\beta_3$, the coefficient on the interaction term.

**Q4.** No. The two groups are already diverging before anything happens: the treatment regions gain 5.2 units/week against the control regions' 3.1, a **differential pre-trend of about 2.1 units/week**. Parallel trends says the two groups' *changes* would have matched absent the intervention, and the pre-period is the one window where that can be checked — here it visibly fails. Note that the *levels* are irrelevant to this judgement; only the slopes matter.

The consequence is directional, not just "be careful". DiD substitutes the control regions' change for what the treatment regions would have done anyway. That substitute is too small by ~2.1 units/week, so the leftover is credited to the intervention and **DiD will overstate the causal effect** — by roughly 2.1 units for each post-period week. A treatment group on a *shallower* pre-trend would bias it the other way.

Formally this is what the placebo test in §1.5 detects: run the DiD on pre-period data alone with a fake intervention date, and here $\hat{\beta}_3$ would come back significantly non-zero rather than the $\hat{\beta}_3 = 0.81$, $p = 0.86$ the real dataset gives. The response is to find better-matched control regions, or to use a method that allows differing trends — not to report the DiD estimate with a caveat attached.

**Q5.** The MMM overestimates the causal effect by 4.25 units/week (~68%). The most likely cause: the MMM's price coefficient is confounded — price reductions may have been deployed in periods when demand was already expected to increase. The DiD is a more credible causal estimate because randomisation of treatment regions eliminates the endogeneity that biases the MMM.

---

## PART 2: Application

### (~1 hour 40 minutes)

---

### Section 2.1 — Dataset

`geo_experiment.csv`: 20 weeks × 8 geographic regions = 160 observations.

| Variable | Description |
|---|---|
| `region_id` | Region identifier (1–8) |
| `week` | Week number (1–20) |
| `revenue` | Weekly revenue (indexed units) |
| `treat` | 1 = treatment region, 0 = control |
| `post` | 1 = post-intervention (weeks 11–20), 0 = pre |
| `treat_x_post` | Interaction term: treat × post |

Regions 1–4: received a 15% price reduction starting week 11.
Regions 5–8: control, no price change.

---

### Section 2.2 — Interpretation Guide

**Reading the DiD regression table:**

When you fit `revenue ~ treat + post + treat_x_post`, the output contains four rows. Only one of them is the treatment effect: the row labelled `treat_x_post`. The other three coefficients describe the pre-period structure of the data.

**The parallel trends plot:** Plot mean revenue by week, separately for treatment and control groups. The pre-period lines should be roughly parallel (same slope, possibly different levels). If they diverge in the pre-period, parallel trends is violated.

**The placebo test:** Running DiD on pre-period data with a fake intervention date tests whether there is *any* differential trend in the pre-period. A significant placebo effect ($p < 0.05$) is evidence against parallel trends.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

<!-- BEGIN GENERATED homework pointer - tools/render_lecture_homework.py; do not hand-edit.
     Replaces a ~150-line duplicate of the homework that had drifted into a DIFFERENT
     assignment (Task 004, 2026-08-13): for lectures 04-10 only 1-3 of ~16 question
     variables still matched the notebook, and the dataset filenames were wrong.
     The notebook is the assignment of record; answers live only in answer_keys/. -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_12_did.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_12_did.ipynb` |
| Dataset | `homework_datasets/geo_experiment.csv` |
| Graded questions | **13** — Part A: 5 · Part B: 5 · Part C: 3 |
| Answer key (instructor only) | `answer_keys/hw12.json` |

Answers and tolerances are never duplicated outside `answer_keys/hwNN.json`
(rendered for instructors as `quiz/answer_key_values.md`).

<!-- END GENERATED homework pointer -->

