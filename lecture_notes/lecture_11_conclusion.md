# Lecture 11: Synthesis and Capstone
## Connecting the 10 Models — and What to Do With Them

---

### Overview

This is the final class session. No new models. Instead: how the 10 models fit
together into a coherent analytical practice, the common failure modes that cut
across all of them, and how to develop the judgment to know when each model applies.

The session culminates in the Intensive Day debrief — going through the FitLoop case
together and explaining the analytical reasoning behind the recommended answers.

**By the end of this session you should be able to:**
- Map any marketing analytics question to the appropriate model (or combination of models)
- Identify the common failure modes shared across all 10 models
- Explain why no model output should be trusted without external validation
- Describe your own analytical development trajectory from Lecture 0 to now

---

## PART 1: The 10 Models as a Unified Framework

---

### 1.1 Three Questions, Ten Models

Every model in this course answers one of three fundamental questions about customers:

**Question 1: Did something work?** (Causal inference)
- Bayesian A/B Testing (L01): Did this product change improve conversion?
- Uplift Modeling (L07): Will this campaign change behavior — for which customers?

**Question 2: What will happen?** (Prediction)
- Survival Analysis (L04): When will this customer churn?
- Customer Lifetime Value (L08): How much will this customer be worth?
- Markov Chains (L05): Where will the customer distribution be in 3 months?
- Demand Forecasting (L10): What will revenue look like next quarter?

**Question 3: Why did something happen, and how do we optimize?** (Attribution and optimization)
- Price Elasticity (L02): How price-sensitive are customers, and what price maximizes revenue?
- Marketing Mix Modeling (L03): Which marketing channels drove revenue — and how should we reallocate budget?
- Multi-Touch Attribution (L06): Which touchpoints in the conversion path deserve credit?
- Conjoint Analysis (L09): What features do customers value — and how much will they pay?

**The most common mistake:** Using a model that answers one question when you need
an answer to a different one. MMM tells you about budget allocation (Q3), not whether
your marketing caused sales to increase (Q1). Survival analysis tells you when
customers will churn (Q2), not what to do about it (Q1 and Q3).

---

### 1.2 The Dependency Map

The models form a learning sequence. Later models depend on concepts from earlier ones:

```
Bayesian A/B (L01)
    └── Probability, updating beliefs from evidence
    └── Feeds into: Uplift (L07) — random assignment validates T-learner

OLS / Price Elasticity (L02)
    └── Regression as a framework
    └── Feeds into: MMM (L03) — nonlinear extension of OLS
    └── Feeds into: Conjoint (L09) — logit as a classification analog of regression

Survival Analysis (L04)
    └── Conditional probability, censoring, hazard functions
    └── Feeds into: CLV (L08) — P(alive) from BG/NBD is a survival concept

Markov Chains (L05)
    └── State transitions, probability distributions over states
    └── Feeds into: Attribution (L06) — Markov removal effect uses same framework

Uplift Modeling (L07)
    └── Potential outcomes, causal inference
    └── Completes: A/B Testing (L01) — both are about measuring causal effects
```

The order matters. You cannot understand BG/NBD's P(alive) logic without understanding
survival analysis. You cannot interpret Shapley values without understanding marginal
contributions. This is why the course is 10 lectures rather than 10 independent modules.

---

### 1.3 Cross-Cutting Themes

Five themes recur across all 10 models. Mastering these is more important than
mastering any individual model.

---

#### Theme 1: Correlation vs. Causation

Every model in this course is capable of being misinterpreted as causal when it is
correlational.

| Model | What it measures | What it cannot measure |
|---|---|---|
| OLS Price Elasticity | Association between price and quantity | Whether price CAUSED the quantity change |
| MMM | Channel contribution to revenue over time | Whether turning off a channel would reduce revenue |
| Shapley Attribution | Average marginal contribution across paths | Whether the channel caused the conversion |
| Cox Hazard Ratios | Association between features and churn timing | Whether changing the feature would change churn |
| BG/NBD CLV | Expected future purchases based on history | Whether investing in this customer would change their behavior |

The only model in this course that can support causal claims directly is Bayesian
A/B Testing — and only when assignment was randomized. Uplift modeling supports
causal claims about treatment effects, but only when the T-learner was estimated from
a randomized experiment.

Rule: If someone asks "did X cause Y?" and the only evidence is a coefficient from
a regression or an attribution model, the honest answer is: "We observed an association.
To establish causation, we need a randomized experiment."

---

#### Theme 2: The Assumption You Didn't Check

Every model has at least one critical assumption that is routinely violated in real
data and routinely not checked:

| Model | Critical assumption | How to check |
|---|---|---|
| OLS | Exogeneity (no endogeneity) | Is the predictor correlated with omitted variables? |
| MMM | Identified variation in spend | Did spend vary enough and independently enough to identify λ? |
| KM/Cox | Independent censoring | Is dropout from the study correlated with the event? |
| Cox | Proportional hazards | Schoenfeld residuals test |
| Markov | Markov property | Is history truly irrelevant? |
| Shapley | Stable conversion rates per coalition | Were conversion rates actually observed for each coalition? |
| T-learner | Random assignment | Was treatment truly randomized? |
| BG/NBD | Stationary individual rates | Did purchase rates shift during the observation period? |
| Gamma-Gamma | Spend ⊥ frequency | Pearson correlation between frequency and AOV |
| MNL/Conjoint | IIA | Are any alternatives close substitutes? |
| Prophet | No structural breaks | Did the data-generating process change during training? |

The analyst who checks these assumptions before presenting results is the one people
come back to. The one who doesn't check eventually presents a wrong result with confidence.

---

#### Theme 3: The Model Is a Simplification

All models are wrong. Some are useful. The question is not "is this model correct?"
but "is this model useful for this decision?"

A useful model is:
- Approximately right for the decision being made (not necessarily the most accurate
  possible model)
- Interpretable to the people who need to act on it
- Fast enough to run and update as new data arrives
- Robust to the assumptions you're not sure about

The BG/NBD model assumes Poisson purchase processes with geometric dropout. This is
almost certainly not exactly true. But it produces P(alive) estimates that are
directionally correct, interpretable to non-statisticians, and can be computed from
only three numbers per customer. That combination of properties makes it useful.

The habit of asking "what assumption is this simplification making, and how wrong
could it be?" is more valuable than the ability to implement any specific model.

---

#### Theme 4: Uncertainty Is Information

Every model produces uncertainty estimates. Most analysts suppress them.

- Bayesian A/B Testing: the 95% credible interval tells you the range of plausible
  lift magnitudes, not just the direction
- Prophet: the prediction interval widens with horizon — this is the model being
  honest about irreducible uncertainty
- Cox model: standard errors on hazard ratios tell you whether the feature effect
  is reliably estimated
- Uplift model: the Qini coefficient tells you how well the model ranks customers —
  not just the average effect

A recommendation delivered with appropriate uncertainty ("our best estimate is X,
and we are 80% confident it lies between Y and Z") is more trustworthy — and more
actionable — than a point estimate delivered with false precision.

---

#### Theme 5: The Model Output Is the Beginning, Not the End

Jess Lachs framed it precisely: "Not just answering the why, but answering the
'What do we do now that we know this?'"

The most important question to ask after any model output: "So what?" 

- Price elasticity is -1.84. **So what?** Revenue is currently above the
  revenue-maximizing price. Lower prices to increase revenue — but by how much,
  for which SKUs, with what competitive response risk?
- Shapley value for Paid Search is 50%, but last-touch was 8%. **So what?** Paid
  Search has been underinvested. How much additional PS budget could the business
  deploy before saturation? What is the marginal ROI at current spend?
- BG/NBD P(alive) = 0.09 for a previously-loyal customer. **So what?** This is a
  customer worth investigating. Was there a service failure? A competitive offer?
  Is this a solvable churn event or a natural lifecycle exit?

The model output generates the question. Your judgment answers it.

---

## PART 2: Common Failure Modes

---

### 2.1 The Ten Most Common Analytical Failures in Marketing

In roughly increasing order of seriousness:

**10. Using the wrong time unit.**
Running a weekly MMM but using daily spend data, or running a monthly KM curve when
monthly granularity is too coarse to detect early churn. Time unit choice is a model
specification decision.

**9. Treating the training dataset as the test population.**
Reporting R² on training data, reporting KM curves on the full dataset without
a holdout, running Qini on the training set. Out-of-sample validation is not optional.

**8. Confusing the intercept with baseline.**
In a Cox model, e^β₀ is not the baseline hazard rate — h₀(t) is the baseline hazard
function. In an MMM, β₀ is the baseline revenue when all adstock values are zero.
The intercept does not mean "no marketing" in most specifications.

**7. Forgetting that censoring is informative when it shouldn't be.**
If customers who are about to churn tend to go quiet (stop opening emails, stop
logging in) before cancelling, and your analysis date causes these customers to be
administratively censored, the censoring is informative — violating the KM/Cox
independence assumption.

**6. Using last-touch attribution to make budget decisions.**
This theme runs through Lectures 3 and 6. Last-touch systematically overcredits
closing channels and undercredits awareness channels. Any budget reallocation
informed by last-touch attribution alone will tend to defund the channels that are
creating the demand that makes the closing channels look efficient.

**5. Treating average CLV as the applicable CLV for all acquisition channels.**
This is so common it deserves its own entry. Average CLV = $210. Set CAC target =
$70 for all channels. Result: you dramatically overspend on channels that bring in
$80 CLV customers and dramatically underspend on channels that bring in $400 CLV
customers.

**4. Assuming non-random treatment assignment produces valid uplift estimates.**
If your "control group" consists of customers who didn't opt into a program, or
customers from a different time period, or customers in a different geography without
checking for parallel trends — your T-learner estimate is biased by an unknown amount.

**3. Extrapolating a model outside its training range.**
MMM estimated with TV spend between $50k and $200k/week should not be used to
optimize at $500k/week. Prophet trained on 3 years including a major promotional
campaign should not be used to forecast post-promotion without a changepoint. The
Hill function at spend levels far above EC₅₀ is extrapolating, not interpolating.

**2. Conflating statistical significance with business significance.**
A price elasticity of -1.84 with a standard error of 0.02 (highly statistically
significant) but based on data from a market where your product has 2% share
may be useless for pricing your product in a market where you have 40% share.
P-values and confidence intervals measure the reliability of the estimate given
the data. They say nothing about whether the estimate is relevant to the decision.

**1. Treating the model output as the answer.**
The model is not the answer. It is structured information that should change, sharpen,
or confirm your judgment about a decision. The Shapley value for Paid Search is not
a budget recommendation. The Cox hazard ratio for workouts is not a product specification.
The Prophet forecast is not an inventory order.

The analyst who mistakes the model output for the answer will eventually give a
recommendation that is technically correct and practically useless — or worse, harmful.
The analyst who treats the model as a thinking tool rather than a decision machine
will give recommendations that get implemented.

---

## PART 3: Intensive Day Debrief

---

### 3.1 The FitLoop Case: What the Models Said

This section is completed in class, working through the FitLoop case scenario
together. The six model outputs from the intensive day, and the analytical reasoning
behind the recommended answers:

**Model Output 1: Bayesian A/B Test (pricing experiment)**
- P(B > A) = 0.94. The $21 price is better than $18 with 94% posterior probability.
- Expected lift: +8.2% in conversion rate
- Key judgment call: 94% is below the typical 97% threshold for pricing changes.
  Is the additional data collection cost worth narrowing the uncertainty? This depends
  on the cost of being wrong — with 185,000 subscribers, a wrong pricing decision
  affects significant revenue.

**Model Output 2: Price Elasticity**
- Log-log elasticity: ε = -0.83 (inelastic demand)
- Revenue-maximizing price: above $21
- Key judgment call: inelastic demand supports the price increase, but elasticity
  was estimated from a period with specific competitive dynamics. The A/B test is
  more reliable than the elasticity model for this specific decision.

**Model Output 3: BG/NBD CLV**
- Median 12-month CLV: $162
- Top decile CLV: $480
- Key judgment call: which CLV estimate do you use to justify the $6 retention
  campaign budget? The median overstates value for low-P(alive) customers; the
  mean is pulled up by a few very high-value customers.

**Model Output 4: Survival Analysis + Cox Model**
- Median survival time: 22 months
- HR for avg_workouts: 0.74 (each additional workout reduces churn 26%)
- HR for support_tickets_30d: 1.42 (each ticket increases churn 42%)
- Key judgment call: the Cox model identifies two opposing signals. The product
  intervention (improve workout completion) is the high-leverage play; support
  ticket resolution is the damage-control play.

**Model Output 5: T-Learner Uplift**
- ATE: 10.7 percentage points
- Profit-maximizing targeting threshold: 3.7% (= $6 / $162)
- ~38% of at-risk customers should be targeted
- Key judgment call: 38% is a lot of customers to include in a retention campaign.
  Does the Sleeping Dog risk justify further segmentation? The model showed ~8% of
  at-risk customers have τ̂ < 0.

**Model Output 6: Markov Chain Engagement States**
- Current Active% = 68%, projected to drift to 41% over 18 months without intervention
- Key judgment call: the intervention that most changes the steady-state is
  increasing the Dormant→Active transition (re-engagement). The At-Risk→Active
  transition matters too but requires catching customers earlier in the cycle.

---

### 3.2 The Recommended Intensive Day Answers

**Q1 (specific number from a specific model):**
The specific number that most directly answers the decision question (raise price?)
is P(B>A) = 0.94 from the Bayesian A/B test. This directly quantifies confidence
that the $21 price generates higher conversion × revenue than $18.

**Q2 (model excluded and why):**
Strongest exclusion argument: the price elasticity model should be weighted less
than the A/B test for this specific decision because the A/B test provides experimental
evidence under current market conditions, while the elasticity model is based on
historical observational data subject to endogeneity bias.

**Q3 (assumption you would want to test):**
For the Cox model: proportional hazards assumption. The HR for workouts (0.74) may
not be constant over the full tenure distribution — new subscribers with low workout
rates may have very different churn dynamics than long-tenured subscribers with
established habits.

**Q4 (recommendation + what would change it):**
Raise price to $21 AND run the retention campaign targeting the 38% of at-risk
customers identified by the uplift model. What would change it: (a) If the Sleeping
Dog rate is higher than 8%, reduce the targeting threshold. (b) If competitive
analysis shows a rival is planning an aggressive pricing move, delay the price
increase until competitive dynamics are clearer.

---

## PART 3 Checkpoint

These questions synthesize across the full course.

1. A new analyst proposes running a Markov chain model to answer the question "did
   our re-engagement campaign cause more customers to become active?" What is wrong
   with this approach, and which model would you use instead?

2. You fit a Cox model and get HR = 0.52 (p < 0.001) for "annual_plan vs. monthly."
   Your manager says: "Great — let's push all customers toward annual plans to reduce
   churn." What question do you ask before making this recommendation?

3. An MMM returns β_TV = $3.8M and β_Digital = $2.1M. A colleague says: "TV is more
   valuable — let's cut digital and put it all in TV." Identify two things wrong with
   this reasoning.

4. Prophet forecast MAPE = 8% on a 13-week holdout. Your manager says the model is
   ready for production. What three additional checks would you run before agreeing?

5. You have been asked to build a CLV model, an uplift model, and a survival model
   for the same customer dataset. In what order would you build them, and why?

---

### Checkpoint Answer Key

**Q1.** Markov chains describe the distribution of customers across states over time —
they are predictive/descriptive models, not causal models. A Markov chain cannot
tell you whether the re-engagement campaign CAUSED more customers to become active
(it might just reflect seasonal variation, or the customers who received the campaign
might have self-selected). Use **Bayesian A/B Testing or uplift modeling** with a
randomized control group to measure causal impact.

*Common wrong answer:* "Compare Markov chain steady states before and after the
campaign." This doesn't control for anything that changed in the same period.

**Q2.** "Is the HR reflecting the causal effect of annual plans, or the selection
effect of who chooses annual plans?" Annual subscribers are not a random sample —
they self-select for higher commitment. The HR = 0.52 may primarily reflect that
customers who were already going to stay long-term are more likely to choose annual
plans, not that annual plans cause retention. Before recommending this to all customers,
you need a randomized experiment: randomly assign some customers to receive an
annual-plan discount and compare churn rates to a control group.

**Q3.** Two errors: (1) β_TV is the MAXIMUM revenue contribution plateau (when TV's
Hill function = 1.0), not the actual current contribution. What matters for reallocation
is the marginal ROI at current spend levels, not the model coefficient. (2) Moving
all digital to TV would push TV spend far outside the range where the Hill function
was calibrated, extrapolating into unknown territory where diminishing returns may be
severe. Reallocation should be incremental, with cross-validation at each step.

*Common wrong answer:* "You're right that TV is better." The β coefficients are not
comparable without normalizing for Hill function values at current spend.

**Q4.** Three additional checks: (1) Inspect the component decomposition for
plausibility — does the trend growth rate match business understanding? Does the
seasonal pattern peak at the right time of year? (2) Check for changepoints — does
the model correctly identify known structural breaks in the historical data? (3)
Compare the Prophet MAPE to a naive baseline (last year's same week, or a moving
average) — a MAPE of 8% means nothing if the naive baseline achieves 6%.

**Q5.** Build in this order: (1) **Survival analysis** first — it characterizes who
churns and when, providing the empirical basis for retention priorities. (2) **CLV**
second — once you know survival rates, you can estimate future transaction counts and
thus CLV. The BG/NBD uses survival-analysis-like logic (P(alive)). (3) **Uplift
modeling** last — you need CLV estimates to set the campaign targeting threshold
(cost/CLV = threshold). You also need the survival model's Cox hazard ratios to
identify which behavioral features predict churn, which informs the feature space
for the uplift model.

---

## PART 4: What Comes Next

---

### 4.1 Extensions and Advanced Topics

This course covered the foundational 10 models. Each has a body of advanced literature:

**Causal Inference:**
- Difference-in-differences, synthetic control, regression discontinuity (extensions
  of A/B and uplift concepts)
- Key resource: Cunningham, *Causal Inference: The Mixtape* — free at
  causalinferencethebook.com

**Bayesian Methods:**
- Hierarchical models, MCMC sampling, prior elicitation
- Key resource: Gelman et al., *Bayesian Data Analysis* — free PDF at
  stat.columbia.edu/~gelman/book/BDA3.pdf

**Time Series:**
- ARIMA, state space models, neural network forecasters (NHiTS, PatchTST)
- Key resource: Hyndman & Athanasopoulos, *Forecasting: Principles and Practice* —
  free at otexts.com/fpp3/

**Experimentation at Scale:**
- CUPED (variance reduction), sequential testing, switchback experiments
- Key resource: Kohavi et al., "Trustworthy Online Controlled Experiments" (book)

**CLV and Customer Models:**
- Contractual models (BG/BB for subscription businesses), PyMC-Marketing
- Key resource: brucehardie.com — Fader and Hardie's original papers and Excel
  implementations, all free

---

### 4.2 The Skills That Matter Most

Looking back over the course, the five capabilities that will most determine your
effectiveness as a marketing analyst:

**1. Model selection judgment.** Knowing which model answers which question — and
which questions a given model cannot answer. This is the hardest to teach and the
most valuable to have.

**2. Assumption auditing.** Before presenting any result: what are the critical
assumptions? Are they likely to hold in this specific context? What would change if
they don't?

**3. Uncertainty communication.** Reporting point estimates without uncertainty
intervals is worse than not reporting them — false precision misleads decisions.
Learning to say "our estimate is X, and here's how confident we are and why" is a
career-long practice.

**4. Incremental thinking.** Every optimization question in this course (budget
reallocation, targeting threshold, revenue-maximizing price) comes down to marginal
reasoning: what does one more dollar do? What does one more customer in this segment
do? The marginal framework generalizes far beyond these 10 models.

**5. The habit of external validation.** No model output should be trusted without
checking it against an independent source. This course gave you the tools to check:
the textbook formulas, the worked examples, the verification checklists. Use them.

---

### Final Reflection Questions

These are not graded. They are for your own synthesis.

1. Which of the 10 models do you expect to use most in your career, and why?

2. Which model's assumptions are most likely to be violated in the domain you are
   planning to work in? What would you do about it?

3. The course philosophy is that analytical judgment matters more than model execution.
   Give a specific example from the course where judgment — not calculation — was the
   hardest and most important step.

4. What is one analytical question you are now equipped to answer that you could not
   answer before this course? What would you need to learn next to answer it well?

---

**End of Course. Good luck.**

