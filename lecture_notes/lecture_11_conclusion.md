# Lecture 11: Synthesis and Capstone
## Connecting the 10 Foundational Models — and What to Do With Them

---

### Overview

This is the mid-course synthesis session. No new models. Instead: how the first 10
models fit together into a coherent analytical practice, the common failure modes that
cut across all of them, and how to develop the judgment to know when each model applies.

The session also prepares you for the Intensive Day, which follows after Lectures 12–14.

**By the end of this session you should be able to:**
- Map any marketing analytics question to the appropriate model (or combination of models)
- Identify the common failure modes shared across all 10 foundational models
- Explain why no model output should be trusted without external validation
- Articulate your analytical development trajectory so far and what remains in Lectures 12–14

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

## PART 3: Intensive Day Preparation

---

### 3.1 What the Intensive Day Requires

The Intensive Day happens after Lectures 12–14. Before it, you will receive the FitLoop
company briefing (Part A of the case scenario). Read it carefully. The model outputs
(Part B) are released at the start of the working session.

Your preparation for the Intensive Day is this course. The models in Lectures 1–10
(and the additional tools from Lectures 12–14) are the tools you will apply.

**What to review before the Intensive Day:**
- The model reference card (one-page summary of all models)
- The FitLoop company briefing distributed in the week prior
- Your own notes on any models you feel less confident about

**What to bring:** Nothing. All materials are provided on the day.

---

### 3.2 How the Individual Response Card Works

Before any group presentations begin, you will each submit an individual response card
with four questions. See the intensive day format document for full details. The key
point: the card is worth 100 of 140 total points and is submitted during working time —
not after presentations.

**The four questions (identical for every student):**
1. State one specific number from your analysis, the model it came from, and what it means for the decision.
2. Name one model you considered but did not use, and explain why it was lower priority for this case.
3. Identify one assumption in your analysis that you are least confident in, and what you would check to test it.
4. Write your personal recommendation for the executive team (may agree or differ from your group's), plus one condition that would change it.

---

## PART 3 Checkpoint

These questions synthesize across the first 10 lectures.

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

*Checkpoint answer key available from your instructor.*

---

## PART 4: What Comes Next in This Course

---

### 4.1 Lectures 12–14 Preview

The next three lectures extend the analytical toolkit beyond the first 10 models:

**Lecture 12: Difference-in-Differences**
- Establishes causation from observational data using parallel trends
- Critical for evaluating marketing interventions when randomized experiments are not feasible

**Lecture 13: Customer Segmentation**
- k-means clustering to identify behaviorally distinct customer groups
- Enables model-per-segment strategies for targeting and personalization

**Lecture 14: CausalImpact**
- Bayesian structural time series to measure the causal effect of a marketing campaign
- The most rigorous non-experimental tool for incrementality measurement

After Lecture 14, the Intensive Day applies all 14 models (or a selection of them)
to the FitLoop case.

---

### 4.2 Extensions Beyond the Course

Each model in this course has a body of advanced literature:

**Causal Inference:**
- Synthetic control, regression discontinuity
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

### 4.3 The Skills That Matter Most

Looking back over the first 10 lectures, the five capabilities that will most determine your
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
do? The marginal framework generalizes far beyond these models.

**5. The habit of external validation.** No model output should be trusted without
checking it against an independent source. This course gave you the tools to check:
the textbook formulas, the worked examples, the verification checklists. Use them.

---

### Mid-Course Reflection Questions

These are not graded. They are for your own synthesis at the halfway point.

1. Which of the first 10 models do you expect to use most in your career, and why?

2. Which model's assumptions are most likely to be violated in the domain you are
   planning to work in? What would you do about it?

3. The course philosophy is that analytical judgment matters more than model execution.
   Give a specific example from the first 10 lectures where judgment — not calculation — was the
   hardest and most important step.

4. What is one analytical question you are now equipped to answer that you could not
   answer before this course? What additional tools from Lectures 12–14 might further
   strengthen your answer?

---

*Three more lectures remain: Difference-in-Differences (L12), Customer Segmentation (L13), and CausalImpact (L14), followed by the Intensive Day.*

