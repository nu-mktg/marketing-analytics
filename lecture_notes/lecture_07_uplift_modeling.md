# Lecture 7: Uplift Modeling
## Targeting the Persuadables, Not the Sure Things

---

### Overview

**Business question:** You have a $4 retention campaign. Which customers should receive it — and which customers will actually change their behavior because of it (vs. those who would renew anyway or those who would churn regardless)?

**What you will be able to do:**
- Explain the four customer types in uplift modeling
- Compute a simple T-learner CATE estimate
- Apply the profit-maximizing targeting threshold
- Interpret Qini curves and coefficients
- Identify when treatment assignment must be randomized for valid estimates

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Expected Value

Expected value = probability × outcome, summed across all scenarios.

**Example:** Incremental profit per contacted customer = uplift × conversion_value − campaign_cost

Uplift τ(X) is how much the offer raises a customer's probability of converting (defined in Section 1.4A). With the offer, a customer converts with probability p₁ and is worth p₁ × value − cost, because the cost is paid on every contact. Without it, the customer is worth p₀ × value. The difference is (p₁ − p₀) × value − cost = τ(X) × value − cost.
> If τ(X) = 0.15, value = $30, cost = $4: Expected profit = 0.15 × 30 − 4 = 4.50 − 4 = **$0.50**
>
> If τ(X) = 0.08, value = $30, cost = $4: Expected profit = 0.08 × 30 − 4 = 2.40 − 4 = **−$1.60** (negative!)

The breakeven threshold: τ(X) × value = cost → τ(X) = cost/value = 4/30 = **0.133**

---

### Section 1.2 — Business Motivation

---

#### Duolingo's 600 Streak Experiments

Jackson Shuttleworth is a Group Product Manager at Duolingo leading the retention team. Over four years, his team ran over 600 experiments on a single feature: the streak. That is approximately one experiment every other day.

One finding exemplifies the uplift insight: changing the call-to-action button from "Continue" to "Commit to my goal" was a massive win for retention. But the users who responded to this change were not the users most likely to continue anyway — they were users on the margin between re-engaging and quietly lapsing. The users already highly committed to their goal did not need the new framing; they were coming back regardless.

A model that predicts "who is likely to be active next week" would have ranked the committed users highest and sent them the message. Uplift modeling ranks differently: who will change their behavior because of the message? That is a fundamentally different question.

Lauryn Isford, who led growth at Airtable after years at Facebook, adds a strategic point:

> *"An activation rate that falls in a lower percentage range, maybe 5–15%, is better than one that falls in a higher percentage range — because it means there's likely much higher correlation with long-term retention."*

Targeting precision over volume is the uplift mindset.

---

### Section 1.3 — Conceptual Framework

---

#### The Four Customer Types

In any campaign, customers fall into four segments based on how they would respond to treatment:

| Type | Without campaign | With campaign | Should you target? |
|---|---|---|---|
| **Sure Thing** | Converts anyway | Converts | No — wasted spend |
| **Persuadable** | Does not convert | Converts | Yes — this is who you want |
| **Lost Cause** | Does not convert | Does not convert | No — cannot be helped |
| **Sleeping Dog** | Converts | Does NOT convert | Definitely no — makes things worse |

Standard propensity models (predict conversion probability) identify Sure Things as "high priority" customers. Uplift models identify Persuadables — customers whose behavior actually changes because of your intervention.

Sleeping Dogs are real and surprisingly common in subscription renewal campaigns. Sending a "we'll miss you" message sometimes reminds customers they wanted to cancel.

---

### Section 1.4 — Mathematical Framework

---

> **Uplift modeling starts with an A/B test.** An A/B test is a **randomized experiment**: a coin flip decides which version each customer gets. In uplift terms, version A is no campaign (the **control** group) and version B is the campaign (the **treated** group). This lecture uses "A/B test" and "randomized experiment" for the same thing. Everything below assumes the data come from one. The T-learner needs it to estimate τ(x) (Part A), and the Qini needs it to evaluate the ranking (Part C). In practice, a firm runs the campaign as an A/B test on a sample of customers, fits the uplift model on that sample, and uses it to choose whom to target among the rest.

#### Part A: Treatment Effects and the T-Learner

**Notation.** **X** is the list of features we observe about a customer before the campaign: in the worked example below, age and tenure; in `homework_07`, tenure, workouts per week, streak, support tickets and plan. **T** = 1 if the customer was treated, 0 if control. **Y** = 1 if they converted. Y(1) and Y(0) are the customer's two potential outcomes, with and without the campaign (Deep Dive below).

**Two effects, and how they are related.**
- The **conditional average treatment effect** (CATE) at features x is τ(x) = E[Y(1) − Y(0) | X = x]: the average uplift among customers who look like x. "Uplift" in this lecture always means τ(x).
- The **average treatment effect** (ATE) is the same average taken over everyone: ATE = E[Y(1) − Y(0)].
- Averaging the CATE over the customer base gives the ATE: **ATE = E[τ(X)]**. Under random assignment the ATE needs no model. It is estimated by the treated conversion rate minus the control conversion rate. This identity is the reason the **ATE check** in Section 2.1 works: the mean of a model's τ̂ over a sample should land near that sample's treated-minus-control rate.
- When τ(x) differs from one kind of customer to another, the treatment effect is **heterogeneous** (a *heterogeneous treatment effect*, or HTE). Uplift modeling exists because of heterogeneity. If τ(x) were the same for everyone, it would equal the ATE, and the A/B test alone would settle targeting: everyone or no one.

**A hat marks an estimate.** τ(x) is the true uplift, which no one observes. τ̂(x) is a model's estimate of it from data. Likewise μ and μ̂.

The T-Learner estimates the CATE by training two separate models:
- μ̂₁(X): predicted conversion probability if treated, trained on treated customers only
- μ̂₀(X): predicted conversion probability if NOT treated, trained on control customers only

**CATE estimate for customer i:** τ̂(Xᵢ) = μ̂₁(Xᵢ) − μ̂₀(Xᵢ)

**Example:** the treated-arm model gives a customer μ̂₁ = 0.30 and the control-arm model gives the same customer μ̂₀ = 0.12. Then τ̂ = 0.30 − 0.12 = **0.18**: the campaign is estimated to raise this customer's conversion probability by 18 percentage points.

**Critical requirement:** Treatment must be randomly assigned. Without randomization, treated customers systematically differ from control customers in unobserved ways. The difference μ̂₁(X) − μ̂₀(X) would capture both the treatment effect and the selection bias — confounding the estimate.

**Both models are evaluated at the same X.** τ̂(Xᵢ) asks each model for a prediction about customer i, including the model of the arm customer i was never in. That only works where both arms contain customers like i (this is called **overlap**). Randomization delivers overlap on average. In a thin corner of the feature space, such as a rare plan type or an extreme tenure, one arm may hold only a handful of similar customers, and τ̂ there is mostly noise.

> ### 🔍 Deep Dive: The Potential Outcomes Framework
> The formal framework defines two potential outcomes for each customer: Y(1) = what they would do if treated, Y(0) = what they would do if not treated. The true individual treatment effect is Y(1) − Y(0). The fundamental problem: you can only observe one of Y(1) or Y(0) for each person — never both. Random assignment ensures that E[Y(0)|T=1] = E[Y(0)|T=0], making the control group a valid counterfactual for the treatment group. Because assignment ignores X as well, the same equality holds within every group of customers with X = x. That conditional version is what the T-learner uses: it lets μ̂₀(x), fitted on control customers, stand in for the missing Y(0) of treated customers with the same x. Without random assignment, the T-learner estimate is biased by an unknown amount.

#### Reading the four types off the models

The four types in Section 1.3 describe one customer's two potential outcomes, and no one ever observes both. A model only sees groups of similar customers, so τ(x) measures a **mix**. Because the outcome here is binary (Y = 1 if the customer converts, 0 if not), Y(1) − Y(0) is +1 for a Persuadable, −1 for a Sleeping Dog and 0 for the other two types, so among customers with features x:

> τ(x) = (share of Persuadables) − (share of Sleeping Dogs)

This is the same difference in probabilities as before. Customers with x convert with the offer at rate (Sure Things + Persuadables) and without it at rate (Sure Things + Sleeping Dogs). The Sure Things cancel, and Lost Causes never convert, so neither adds anything to τ(x).

| What the model says at x | What it means about customers like x | Action |
|---|---|---|
| τ̂(x) > c/v (Part B) | Persuadables outnumber Sleeping Dogs by enough to pay for the contact | Target |
| 0 < τ̂(x) < c/v | Persuadables outnumber Sleeping Dogs, but not by enough to pay | Don't target |
| τ̂(x) < 0 | Sleeping Dogs outnumber Persuadables: the campaign does net harm here | Never target |

τ̂ alone cannot tell a Sure Thing from a Lost Cause, since both sit at τ̂ ≈ 0. μ̂₀, the conversion probability without the campaign, can: high μ̂₀ means Sure Things and low μ̂₀ means Lost Causes. When this course labels customers "sleeping dogs" from data, for example everyone with τ̂ < 0, it means the last row: a group in which the campaign is estimated to do net harm, not a group made up only of Sleeping Dogs.

---

#### Part B: The Targeting Threshold

Targeting all customers with τ̂(X) > 0 maximizes conversions but not profit. The profit-maximizing threshold is:

> Target if τ̂(X) > **cost / value**

**Example:** Campaign cost = $4, conversion value = $30. Threshold = 4/30 = 0.133.

Customers with 0 < τ̂(X) < 0.133 generate positive incremental conversions but negative expected profit. Targeting them costs more than they return.

**Why the rule uses τ̂ when Tool 1 derived it for τ.** The break-even condition τ(X) × value = cost (Section 1.1) is about the true uplift. τ is never observed, so in practice the rule is applied to the estimate τ̂. The decision is then only as good as τ̂. A customer whose τ̂ is just above 0.133 may truly be below it, and the reverse. That is why the model's ranking has to be checked on customers it has not seen (Part C) before anyone acts on it.

---

#### Part C: The Qini Curve and the Qini Coefficient

The Qini curve answers one question. If you contacted customers in order of predicted uplift, highest τ̂ first, how many extra conversions would you have bought by the time you reached the top k?

**Where the treated and control groups come from.** The curve is computed on the randomized experiment's data, ideally the held-out part. Every customer already has a treatment label from the coin flip, and nobody was contacted according to τ̂. The model's only job here is to sort them. Because assignment ignored X, every top-k slice holds both treated and control customers with similar τ̂. Each slice is a small experiment of its own.

**What a good ranking looks like.** A high τ̂ predicts a large gap between a customer's outcome with the offer and without it. It does not predict a high chance of converting. In a slice of high-τ̂ customers, the treated ones should convert well above the control ones. In a slice of sure things, the two convert at the same rate. A good ranking puts the slices with the largest gaps first, so Q(k) climbs fastest at the start.

**Building the curve.** Sort the customers by τ̂, highest first. Among the top k, count:
- N_T(k) and N_C(k): how many were in the treated group and how many in the control group
- Y_T(k) and Y_C(k): how many of those treated customers converted, and how many of those control customers converted

The **incremental conversions** among the top k are

$$Q(k) = Y_T(k) - Y_C(k) \times \frac{N_T(k)}{N_C(k)}$$

This is the treated conversions minus the conversions the same number of treated customers would have produced at the control rate. While the top k hold no control customers, the second term is taken as 0.

**Axes.** The x-axis is k, the number (or share) of customers targeted, from 0 up to everyone. The y-axis is Q(k). At k = everyone, Q equals ATE × N_T, so every model's curve ends at the same height. Only the path to that point depends on the ranking.

**Two reference lines.**
- **Random targeting:** the straight line from (0, 0) to the curve's end point. Contacting customers in random order buys incremental conversions at a constant rate.
- **Perfect ranking:** the curve you get by sorting on the outcomes themselves, with treated converters first and control converters last. It is the yardstick for the best ranking possible on these customers.

**The coefficient.** This is the definition `sklift.metrics.qini_auc_score` computes, and the one `homework_07` asks for:

$$\text{Qini} = \frac{\text{area between the model's curve and the random line}}{\text{area between the perfect curve and the random line}}$$

- **Qini = 0:** the ranking is no better than random targeting.
- **Qini = 1:** the ranking is as good as sorting on the outcomes themselves.
- **Qini < 0:** the ranking is worse than random. It puts the customers the campaign harms near the top.

**On the worked example (Section 2.1, Part B).** Sorted by τ̂, the top 3 customers are three treated converters, so Q(3) = 3 − 0 = 3. The next three are control customers, one of whom converted, so Q(6) = 3 − 1 × 3/3 = 2. Over all 12, Q(12) = 3 − 2 × 6/6 = 1, which is ATE × N_T = (1/6) × 6.

⚠️ **A Qini number means nothing until you say which customers it was computed on.** Scored on the rows the models were trained on (**in-sample**), a fully grown random forest has memorised those rows, and its Qini looks strong. Scored on **held-out** customers that the models never saw, the same forest's Qini can be about one-twentieth as large. That is `homework_07`'s situation, shown below. The held-out number is the one to report, because it is the only one that says how the ranking would do on the next campaign's customers. A small held-out Qini is not a bug in your code. It is the honest finding that this model ranks new customers no better than random.

**What the two panels are, and are not.** They show one T-learner, fitted once on 80% of the experiment, scored on two sets of customers: the 3,200 it was trained on and the 800 it never saw. The gap between them shows how much the forest memorized. It is not a comparison you would report. When you do compare Qini values, for example to choose between two models, score both on the same held-out customers. Holding customers out needs no second experiment. They are part of the same A/B test, set aside before fitting, so they still carry the coin-flip labels the Qini needs.

![Two panels from homework_07's own T-learner: two random forests of 100 trees each, seed 42, trained on an 80 percent split. Each panel plots incremental conversions Q(k) against the share of customers targeted in order of predicted uplift, with three lines: the model's Qini curve, the straight random-targeting line from zero to the curve's end point, and the perfect-ranking curve. The area between the model's curve and the random line is shaded. Panel (a), labelled held-out, uses the 800 test customers the models never saw: the model's curve stays close to the random line and the shaded area is a thin sliver. Panel (b), labelled in-sample, uses the 3,200 training customers the forests were fitted on: the model's curve rises steeply above the random line, then comes back down to the same end point, and the shaded area is about twenty times larger relative to the perfect curve.](figures/lecture_07_qini_curve.png)

**Important:** The Qini coefficient is NOT the fraction of Persuadables the model correctly identifies. It is a ratio of two areas, a measure of ranking quality similar to AUC but for uplift.

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

1. ATE = treated_rate − control_rate. Treated rate = 0.24, control rate = 0.16. What is the ATE?

2. Campaign cost = $4, conversion value = $25. What is the minimum profitable uplift threshold?

3. A customer has τ̂(X) = 0.05 and campaign cost = $4, value = $25. Should you target them?

4. Qini coefficient = 0.42. What does this mean in plain English?

5. The treatment team forgot to randomize — instead, they sent the campaign to high-value customers (who tend to have higher renewal rates anyway). Will τ̂ estimates from the T-learner be too high, too low, or unbiased?

---

### Checkpoint Answer Key

**Q1.** ATE = 0.24 − 0.16 = **0.08** (8 percentage points).
*Common wrong answer:* 0.24/0.16 = 1.5 (this is the lift ratio, not the ATE). ATE is an absolute difference in rates, not a ratio.

**Q2.** Threshold = 4/25 = **0.16** (16 percentage points of incremental conversion needed to break even).
*Common wrong answer:* reading 0.16 as a **16% relative lift** in conversions rather than **16 percentage points** of *incremental* conversion probability. A customer converting 12% of the time without the offer needs to reach 28%, not 13.9%.

**Q3.** **No.** Expected profit = 0.05 × $25 − $4 = $1.25 − $4 = **−$2.75**. The customer has positive predicted uplift but insufficient uplift to justify the cost. The threshold is 0.16; τ̂ = 0.05 < 0.16.
*Common wrong answer:* Yes, because τ̂ > 0 means the campaign helps them. Positive τ̂ is necessary but not sufficient for profitable targeting.

**Q4.** When customers are targeted in descending order of predicted τ̂, the area between the model's Qini curve and the random-targeting line is **42% of the area a perfect ranking would give** (Section 1.4C). The ranking is better than random targeting (Qini = 0) but not perfect (Qini = 1). Before reading anything into it, ask which customers it was computed on: only a held-out Qini says how the ranking does on new customers. It does not mean the model identifies 42% of persuadable customers.
*Common wrong answer:* The model correctly identifies 42% of persuadable customers in the dataset.

**Q5.** **Too high** (biased upward). High-value customers have higher baseline renewal rates (μ₀ is higher). The T-learner estimates μ₀ from the control group, which in this case consists of lower-value customers. It underestimates the control outcome for high-value treated customers, making the treatment effect appear larger than it is. (This assumes the model's features do not fully capture what made a customer "high-value". If they did, the problem would turn into one of overlap (Section 1.4A): no control customers would look like the treated ones, and μ̂₀ would have nothing to learn from.)
*Common wrong answer:* Unbiased, because T-learner uses separate models for treated and control. The models are estimated on non-comparable groups, so the difference μ₁(X) − μ₀(X) is biased.

#### Slide Checkpoint Answers

Numbered to match the Checkpoint slide in this lecture's deck. Where the slide asks a question above again, or the same idea on different numbers, the entry points to that answer.

**Slide Q1.** *"ATE = 0.108. Threshold = 0.12. Is the average customer profitable to target?"* — **No, on average.** 0.108 < 0.12, so targeting a customer whose uplift equals the average loses money, in the same way Q3's customer does. That does **not** mean "target no one". The ATE averages over customers whose τ(x) differs (Section 1.4A), and customers whose τ̂ is above 0.12 are still profitable to target. This is the reason to estimate the CATE at all. Q1 and Q2 above compute an ATE and a break-even threshold on other numbers (0.08 and 0.16).

**Slide Q2.** *"A customer has $\hat{\tau}(x) = 0.05$, $c = \$4$, $v = \$25$. Target them?"* — Same question as Q3 above — see that answer.

**Slide Q3.** *"The Qini coefficient = 0.00. What does this mean in plain language?"* — Same reasoning as Q4 above, which reads a Qini of 0.42 and says what Qini = 0 means.

**Slide Q4.** *"Why does T-learner require random assignment for unbiased CATE estimates?"* — Same reasoning as Q5 above, which works through a campaign that was not randomly assigned.

**Slide Q5.** *"Campaign reaches 10,000 customers. 12% are sleeping dogs. Campaign cost = \$4 each. What is the direct cost of targeting sleeping dogs?"* — 10,000 × 12% = 1,200 sleeping dogs, at \$4 each = **\$4,800** of campaign spend that should never have been sent. Note what "direct cost" does and does not cover: it is only the \$4 per contact. Sleeping Dogs are the segment that converts without the campaign and does **not** convert with it (Section 1.3), so the real damage is the *lost conversions* their negative uplift causes — at v = \$25 a sleeping dog with τ̂ = −0.05 destroys a further 0.05 × \$25 = \$1.25 of expected value each, on top of the \$4. The \$4,800 is therefore a floor, not the full cost.

### Slide Practice Answers

Numbered to match the two "Practice: work in pairs" slides in this lecture's deck. Commit to an answer, and to Sure or Unsure, before you read these.

**P1. False.** A propensity model ranks the customers most likely to renew at the top, and many of those would renew without the campaign: they are Sure Things. Uplift ranks customers by how much the campaign *changes* the outcome (Section 1.3).

**P2. D, 0.15.** Among customers like x, τ(x) = share of Persuadables − share of Sleeping Dogs = 0.20 − 0.05 = 0.15 (Section 1.4A, "Reading the four types off the models"). Sure Things convert either way and Lost Causes never do, so neither moves τ(x). *Common wrong answer:* A, 0.50, which is the conversion rate *with* the campaign (Sure Things + Persuadables), not the change it causes.

**P3. C, −\$1.00.** Expected incremental profit = τ̂ × v − c = 0.10 × 40 − 5 = −1.00 (Section 1.1, Tool 1). The uplift is positive but below the break-even c/v = 5/40 = 0.125. *Common wrong answer:* A, +\$4.00, which forgets that the \$5 is paid on every contact.

**P4. True.** τ̂ alone cannot separate a Sure Thing from a Lost Cause, because both sit near 0. μ̂₀, the conversion probability without the campaign, can: high μ̂₀ points to Sure Things and low μ̂₀ to Lost Causes (Section 1.4A).

**P5. B, 0.09.** Only the held-out Qini says how the ranking does on customers the model has not seen. The in-sample number is inflated because the forest memorized its training rows (Section 1.4C). *Common wrong answer:* D. Held-out customers are part of the same A/B test, set aside before fitting, so they keep their coin-flip labels and no second experiment is needed.

**P6. False.** Each arm's importances show what predicts *conversion* in that arm. A feature that ranks high in both predicts conversion with or without the campaign, which says nothing about who responds to it. A feature that matters in one model but not the other is the hint, to be checked by comparing τ̂ across its values (Section 2.2).

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work through Parts B–C together)


---

**Problem Statement**

A subscription company ran a randomized experiment: 50% of customers received a "loyalty upgrade offer" (treatment), 50% received nothing (control). Here are 12 customers from the experiment with their features, treatment assignment, and outcome:

| ID | Age | Tenure (yrs) | T | Y | Type (fill in) |
|---|---|---|---|---|---|
| C1 | 34 | 5 | 1 | 1 | ? |
| C2 | 52 | 9 | 1 | 1 | ? |
| C3 | 28 | 1 | 1 | 0 | ? |
| C4 | 45 | 7 | 1 | 1 | ? |
| C5 | 38 | 3 | 1 | 0 | ? |
| C6 | 29 | 1 | 1 | 0 | ? |
| C7 | 55 | 10 | 0 | 1 | ? |
| C8 | 33 | 2 | 0 | 0 | ? |
| C9 | 47 | 8 | 0 | 0 | ? |
| C10 | 40 | 6 | 0 | 1 | ? |
| C11 | 31 | 2 | 0 | 0 | ? |
| C12 | 50 | 7 | 0 | 0 | ? |

**Part A:** Compute the observed ATE from this data (treatment conversion rate minus control conversion rate).

**Part B:** After running the T-learner, predicted uplift scores are:

| ID | $\hat{\tau}(X)$ | T | Y |
|---|---|---|---|
| C2 | 0.55 | 1 | 1 |
| C4 | 0.48 | 1 | 1 |
| C1 | 0.42 | 1 | 1 |
| C9 | 0.35 | 0 | 0 |
| C10 | 0.28 | 0 | 1 |
| C12 | 0.08 | 0 | 0 |
| C7 | 0.07 | 0 | 1 |
| C5 | 0.06 | 1 | 0 |
| C3 | 0.02 | 1 | 0 |
| C8 | -0.05 | 0 | 0 |
| C6 | -0.08 | 1 | 0 |
| C11 | -0.12 | 0 | 0 |

What is the mean predicted uplift across all 12 customers? How does this compare to the ATE from Part A?

**Part C:** Identify each treated customer (T=1) by type: persuadable (Y=1, would have been 0 without treatment), sure thing (Y=1 regardless), sleeping dog (Y=0, would have been 1 without), or lost cause (Y=0 regardless). Note: since we cannot observe the counterfactual, use the predicted uplift as a guide.

---

**Full Solution**

**Part A: Observed ATE**

Treatment group (T=1): C1, C2, C3, C4, C5, C6 → conversions: C1, C2, C4 → rate = 3/6 = **50%**

Control group (T=0): C7, C8, C9, C10, C11, C12 → conversions: C7, C10 → rate = 2/6 = **33%**

$$\text{ATE} = 0.50 - 0.33 = \mathbf{0.17} \approx 17 \text{ percentage points}$$

**Part B: Mean Predicted Uplift**

$$\bar{\hat{\tau}} = \frac{0.55+0.48+0.42+0.35+0.28+0.08+0.07+0.06+0.02+(-0.05)+(-0.08)+(-0.12)}{12} = \frac{2.06}{12} \approx \mathbf{0.172}$$

This is close to the observed ATE of 0.17 (1/6 ≈ 0.167 before rounding). This is the **ATE check**, a key validation of the model, and it works because the ATE is the average of the CATE over customers (Section 1.4A). The small difference is sampling variation with only 12 customers. On real data, expect the two to agree in sign and order of magnitude, not to be equal (Section 2.2).

**Part C: Customer Type Classification**

Using predicted uplift as a guide:

| Customer | T | Y | $\hat{\tau}$ | Probable Type | Reasoning |
|---|---|---|---|---|---|
| C1 | 1 | 1 | 0.42 | Persuadable | High uplift + converted |
| C2 | 1 | 1 | 0.55 | Persuadable | Highest uplift + converted |
| C3 | 1 | 0 | 0.02 | Lost Cause | Near-zero uplift, no conversion |
| C4 | 1 | 1 | 0.48 | Persuadable | High uplift + converted |
| C5 | 1 | 0 | 0.06 | Lost Cause | Low uplift, no conversion |
| C6 | 1 | 0 | −0.08 | Sleeping Dog | Negative uplift, no conversion (might have converted without offer) |

Note: C7 (T=0, Y=1, $\hat{\tau}=0.07$) converted in the control group with low predicted uplift. This is consistent with being a Sure Thing who would convert regardless. As Section 1.4A notes, τ̂ alone cannot separate a Sure Thing from a Lost Cause; C7's conversion without the offer is what points to Sure Thing.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**Mean predicted uplift vs. observed ATE:** Because ATE = E[τ(X)] (Section 1.4A), the mean of τ̂ over a sample should land near that sample's treated-minus-control rate. Expect agreement in **sign and order of magnitude**, not equality: τ̂ is noisy, and the observed ATE has its own sampling error. Compare the two on the same data. In `homework_07`, Part A's ATE comes from fixed exercise numbers, not from `uplift_experiment.csv`, so check Part B's mean τ̂ against the ATE you compute from the CSV. Treat a **sign flip**, or a difference of several multiples, as the real warning sign of a fitting problem (for example τ̂ = μ̂₀ − μ̂₁ by mistake) or imperfect randomization.

**The Qini curve shape** (the figure in Section 1.4C shows the first two on real data: the in-sample panel rises steeply, and the held-out panel stays close to the diagonal):
- Steep initial rise, then plateau: most persuadables are concentrated in the top-ranked customers — excellent model
- Roughly diagonal: model barely outperforms random targeting — poor model
- Below diagonal initially: model ranks sleeping dogs (negative uplift) near the top — dangerous

**Feature importances from the T-learner:** $\hat{\mu}_1$'s importances show what predicts conversion among treated customers, and $\hat{\mu}_0$'s show the same among control customers. Neither shows what drives the uplift. A feature that ranks high in both models predicts conversion whether or not the customer is treated, which says nothing about who responds to the campaign. A feature that matters in one model but not the other is a hint that it moderates the effect. Check the hint by comparing τ̂ across that feature's values before acting on it.

**What to verify before trusting the agent output:**
1. Is mean predicted uplift close to observed ATE?
2. Is the Qini coefficient positive? If negative, the model is inverting the ranking
3. Do feature importances make business sense? Remember that they show what predicts conversion, not who is persuadable. To see who is persuadable, compare τ̂ across a feature's values
4. Which split is the Qini from? Report the **test-set** Qini. Expect it to be much lower than the training-set Qini, because a fully grown forest scores far higher in-sample. If the test-set Qini is near 0, say so plainly: the ranking does not carry over to new customers. Quoting the in-sample number instead would hide that

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: 23:59 ET on the due date in the course schedule | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_07_uplift_modeling.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_07_uplift_modeling.ipynb` |
| Dataset | `homework_datasets/uplift_experiment.csv` |
| Graded questions | **16** — Part A: 8 · Part B: 5 · Part C: 3 |

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "If the A/B test was big enough to measure the ATE, it is big enough for uplift modeling."**
It usually is not. The ATE is one number estimated from everyone. Uplift modeling estimates a different effect for every kind of customer, and each arm's model must be fitted well enough that the *difference* between two noisy predictions still carries signal. In `homework_07` the experiment is large enough to measure the ATE clearly, yet the forest's held-out ranking is no better than random (Section 1.4C). Plan the sample for the heterogeneity you want to find, not for the ATE.

**2. "A customer with Y=1 in the treatment group is definitely a persuadable."**
They might be a Sure Thing — someone who would have converted anyway. Without observing their counterfactual, we cannot know for certain. The T-learner does not estimate any one customer's type. At best, τ̂(X) estimates the share of Persuadables minus the share of Sleeping Dogs among customers like them (Section 1.4A).

**3. "We should always target customers with τ̂(X) > 0."**
The correct threshold is τ̂(X) > cost/value, not τ̂(X) > 0. A customer with predicted uplift of 0.01 (1 percentage point) does not justify a \$4 campaign cost unless the conversion value is over \$400.

**4. "The T-learner works without an experiment if the data is rich enough."**
Without random assignment, the T-learner is biased unless X contains every variable that drove both who was treated and who converted, and that can never be checked from the data. Methods built for observational data (propensity scores, instrumental variables, difference-in-differences) rest on their own assumptions that cannot be fully tested. In this course, uplift modeling means an A/B test first.

**5. "A high Qini coefficient means the campaign is profitable."**
The Qini coefficient measures ranking quality, not profitability. A model that perfectly ranks customers by uplift would have a Qini of 1.0, but if the ATE is near zero, targeting anyone may not be profitable regardless of the ranking quality.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Compares treated/control feature means | Section 1.4A — treatment must be randomly assigned |
| Computes treated rate − control rate | Section 1.4A — observed ATE |
| Fits $\hat{\mu}_1$ on treated data | Section 1.4A — $\mu_1(X)$, trained on treated customers only |
| Fits $\hat{\mu}_0$ on control data | Section 1.4A — $\mu_0(X)$, trained on control customers only |
| Computes $\hat{\tau}(X) = \hat{\mu}_1 - \hat{\mu}_0$ | Section 1.4A — the CATE estimate |
| Checks mean $\hat{\tau}$ vs observed ATE | Section 1.4A — ATE = E[τ(X)], the CATE averaged over customers |
| Plots Qini curve, computes coefficient | Section 1.4C — building the Qini curve, and the coefficient as a ratio of areas |

**What to verify:**
1. Mean $\hat{\tau}$ agrees with the observed ATE in **sign and order of magnitude** — not to within a fixed tolerance; how close depends on the learner's regularisation (see Section 2.2)
2. Qini coefficient positive
3. Feature importances directionally sensible
4. No large feature imbalances between treated/control (would indicate poor randomization)
