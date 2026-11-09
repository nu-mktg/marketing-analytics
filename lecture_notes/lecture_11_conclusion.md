# Lecture 11: Synthesis and Capstone
## How the Ten Models Work Together, and How to Answer the Questions They Raise

---

### Overview

This lecture is a take-home reading. It introduces no new models. It shows how the ten models of
the course (L01–L10) fit into one framework, lists the assumptions and failure modes that those
lectures taught, and shows how to answer the follow-up questions that a model output raises. Part 3
describes the Sprint.

About 15 minutes of class time go to this material: the decision map, one worked example that goes
from a model output to a follow-up question and its answer, and the Sprint logistics. Read the full
note before that session (MKTG 3501: Wed 11/18; MKTG 6434: Mon 11/23).

Every assumption, check and failure mode named below cites the lecture section that taught it. If
a point is unclear, the cited section explains it in full.

**By the end of this reading you should be able to:**
- Map a marketing analytics question to the model, or the combination of models, that answers it
- Name the critical assumption of each model and the check its lecture taught
- Turn a model output into a follow-up question, and answer that question with a model from the
  course or with a test design
- Explain which models the Sprint uses and how the group write-up uses the other four

---

## PART 1: The Ten Models as a Unified Framework

---

### 1.1 Three Questions, Ten Models

Every model in this course answers one of three questions about customers:

**Question 1: Did something work?** (Causal inference)
- Bayesian A/B Testing (L01): Did this product change improve conversion?
- Uplift Modeling (L07): Will this campaign change behavior, and for which customers?

**Question 2: What will happen?** (Prediction)
- Survival Analysis (L04): When will this customer churn?
- Customer Lifetime Value, CLV (L08): How much will this customer be worth?
- Markov Chains (L05): Where will the customer distribution be in 3 months?
- Demand Forecasting with Prophet (L10): What will revenue look like next quarter?

**Question 3: Why did something happen, and how do we optimize?** (Attribution and optimization)
- Price Elasticity (L02): How price-sensitive are customers, and how should the price move?
- Marketing Mix Modeling, MMM (L03): Which channels drove revenue, and how should the budget be
  reallocated?
- Multi-Touch Attribution (L06): Which touchpoints in the conversion path deserve credit?
- Conjoint Analysis (L09): Which features do customers value, how much will they pay, and which
  price maximizes profit?

A frequent mistake is to answer one question with a model built for another. MMM informs budget
allocation (Question 3). Whether marketing caused a sales increase is Question 1, and answering it
requires an experiment. Survival analysis estimates when customers will churn (Question 2); deciding
what to do about it requires CLV, an uplift model or an A/B test.

---

### 1.2 The Dependency Map

Later models reuse concepts from earlier ones:

```
Bayesian A/B (L01)
    └── Probability, updating beliefs from evidence
    └── Feeds into: Uplift (L07), built on a randomized A/B test

OLS / Price Elasticity (L02)
    └── Regression as a framework
    └── Feeds into: MMM (L03), whose β's are a least-squares fit once the curves are set
    └── Feeds into: Conjoint (L09), whose profit-maximizing margin equals 1/|elasticity|

Survival Analysis (L04)
    └── Conditional probability, censoring, hazard functions
    └── Related to: CLV (L08), whose P(alive) is the probability a customer has not dropped out

Markov Chains (L05)
    └── State transitions, probability distributions over states
    └── Feeds into: Attribution (L06), whose removal effect uses a Markov chain

Uplift Modeling (L07)
    └── Potential outcomes, causal inference
    └── Completes: A/B Testing (L01); both measure causal effects
```

Shapley values (L06) are averages of marginal contributions. The same question, "what does one more
unit add?", underlies marginal ROI in MMM (L03) and the targeting threshold in uplift (L07).

---

### 1.3 Cross-Cutting Themes

The following themes recur across the ten models.

---

#### Theme 1: Correlation and Causation

Every model in this course can be misinterpreted as causal when it measures an association.

| Model | What it measures | What it cannot measure | Taught in |
|---|---|---|---|
| OLS Price Elasticity | Association between price and quantity | Whether price CAUSED the quantity change | L02 §1.4C |
| MMM | Channel contribution to revenue over time | Whether turning off a channel would reduce revenue | L03 §2.2 |
| Shapley Attribution | Average marginal contribution within observed paths | Whether the channel caused the conversion | L06 §1.4C |
| Cox Hazard Ratios | Association between features and churn timing | Whether changing the feature would change churn | L04 §2.2 |
| BG/NBD CLV | Expected future purchases based on history | Whether spending on this customer would change their behavior | L04 §2.2, L08 §2.1 |

Of the ten models, two support causal claims directly, and both only under randomization: Bayesian
A/B testing (L01), and uplift modeling (L07) when the T-learner is estimated from a randomized
experiment. Every other model in the table above estimates an association. When a randomized test is
feasible, it provides the causal check. L03 §2.2 and L06 §1.4C both describe a geo holdout, in which
a channel is withheld or raised in randomly chosen regions and the regions are compared.

If someone asks "did X cause Y?" and the only evidence is a regression coefficient or an attribution
share, the accurate answer is: "We observed an association. To establish causation, we need a
randomized experiment."

---

#### Theme 2: The Assumption You Did Not Check

Each model has a critical assumption, and each lecture taught a way to check it:

| Model | Critical assumption | How to check | Taught in |
|---|---|---|---|
| OLS elasticity | Exogeneity: price varies for reasons unrelated to demand | How were prices chosen? Prices raised in high-demand weeks bias β₁ | L02 §1.4C |
| MMM | Non-ad drivers that move week to week (season, promotions) enter as control variables | Does spend rise in weeks when revenue would have been high anyway? | L03 §1.4C |
| KM/Cox | Non-informative censoring | Do newer cohorts churn the way older cohorts did at the same age? | L04 §1.3 |
| Cox | Proportional hazards | Schoenfeld residual test (`check_assumptions()` in `lifelines`) | L04 §1.4B, §2.2 |
| Markov | Markov property: transitions depend only on the current state | Does the current state summarize the history that matters? | L05 §1.3 |
| Shapley | Credit is measured within observed paths | A holdout experiment (channel off in randomly chosen regions) measures incrementality | L06 §1.4C |
| T-learner | Random assignment | Was treatment assigned by a coin flip? | L07 §1.4A |
| BG/NBD | The purchase and dropout processes describe the data | Calibration plot: predicted against actual purchases in a holdout period | L08 §2.2 |
| Gamma-Gamma | Spend per order is independent of purchase frequency | Pearson correlation between frequency and average order value | L08 §1.4C |
| MNL/Conjoint | IIA | Are any alternatives close substitutes? | L09 §1.4C |
| Prophet | Lasting trend changes are captured by changepoints or regressors, and temporary events are specified | Do the kept changepoints sit near known events? Was a temporary campaign specified? | L10 §1.4B |

Checking these assumptions before presenting a result is part of the analysis. A result whose
critical assumption fails can be precise and still wrong: L02 §1.4C shows endogeneity
producing a tight standard error around a biased elasticity.

---

#### Theme 3: The Model Is a Simplification

Every model is a simplification. The useful question is whether the model is good enough for the
decision at hand.

A useful model is:
- Approximately right for the decision being made
- Interpretable to the people who need to act on it
- Fast enough to run and update as new data arrive
- Robust to the assumptions you are unsure about

The BG/NBD model assumes that each active customer buys according to a Poisson process and drops out
with some probability after each purchase (L08 §1.4A, §1.4C). Real purchasing is unlikely to follow
these processes exactly. The model still produces P(alive) estimates that a manager can interpret,
that respond to recency (two customers with the same number of purchases differ in P(alive) only
through recency, L08 §1.4A), and that require only three numbers per customer. That combination of
properties makes it useful.

The habit of asking "what does this simplification assume, and how wrong could it be?" is more
valuable than the ability to implement any specific model.

---

#### Theme 4: Uncertainty Is Information

Every model in the course produces an uncertainty estimate, and each belongs next to its point
estimate:

- Bayesian A/B testing: the 95% credible interval gives the range of plausible lift values, and
  whether it excludes 0 shows the direction of the effect (L01 §1.5)
- Prophet: the uncertainty interval widens with the forecast horizon, by an amount that depends on
  how much the trend moved in the history; its coverage is checked on a holdout (L10 §2.2)
- Cox model: the 95% confidence interval on a hazard ratio shows whether a feature's effect is
  reliably estimated; an interval that includes 1.0 is not statistically significant at the 5%
  level (L04 §2.1)
- Uplift model: the Qini coefficient on held-out customers measures how well the model ranks
  customers by their response (L07 §1.4C)

A recommendation stated with its uncertainty ("our best estimate is X, and the 95% interval runs
from Y to Z") gives the decision-maker what they need to weigh the risk.

---

#### Theme 5: From Model Output to an Answered Follow-Up Question

Jess Lachs framed the goal as "Not just answering the why, but answering the 'What do we do now
that we know this?'"

A model output usually raises a follow-up question. Most of these questions can be answered with a
model from this course, and the rest with a test whose result is analyzed with one. For three
outputs from the lectures, the tables below list the follow-up questions, the model or test design
that answers each one, and what a result looks like.

**Output A: price elasticity = −1.84** (L02 §1.5). Demand is elastic at current prices: a 1% price
increase goes with a 1.84% decrease in units sold, and a small price cut would raise revenue.

| Follow-up question | Model or test that answers it | What a result looks like |
|---|---|---|
| How far should the price move? | For revenue, fit a demand curve whose elasticity varies with price (the levels model) and find its peak inside the observed price range (L02 §1.4B). For profit, use the condition that at the profit-maximizing price the margin (p − c)/p equals 1/\|elasticity\| (L09 §1.4D), where c is the unit cost. | With a constant elasticity of −1.84, the profit-maximizing margin is 1/1.84 ≈ 54% of price. Then p\* = c / (1 − 0.54) ≈ 2.19 × c. At an illustrative unit cost of \$10, p\* ≈ \$21.90. The result applies only if \$21.90 lies inside the range of prices observed in the data. |
| Is the answer the same for every SKU? | Re-estimate the log-log regression separately for each SKU or customer segment (L02 §1.4B). | One elasticity per SKU, each with its standard error. The price changes for SKUs whose elasticity is estimated precisely enough for the decision. |
| How will competitors respond? | No model in the course predicts a competitor's reaction from your own sales data. Design a test: change the price in a few randomly chosen markets, keep the other markets at the current price, and analyze the result as a Bayesian A/B test (L01). | P(test markets > control markets) on revenue per customer, and a 95% credible interval on the lift, measured over a period long enough for competitors to react. |

**Output B: Paid Search receives 50% of Shapley credit and 8% of last-touch credit** (L06
Checkpoint Q5). Last-touch under-credits Paid Search, and budgets set on last-touch may under-invest
in it.

| Follow-up question | Model or test that answers it | What a result looks like |
|---|---|---|
| How much more should Paid Search get? | Shapley credit measures contribution within observed paths (L06 §1.4C). The budget question asks what the next dollar buys: compute each channel's MMM marginal ROI at current spend, MR = β · H′(A) / (1 − λ) (L03 §1.4C). | If Paid Search has the highest marginal ROI, the MMM predicts that shifting budget toward it raises revenue. The proposed allocation is judged by its predicted revenue (L03 Common Misconception 5). |
| Will the extra spend cause more conversions? | MMM and Shapley are both fitted on observational data (L03 §2.2). Run a geo holdout: raise Paid Search spend in randomly chosen regions, keep the other regions unchanged, and analyze the result as an A/B test (L01; L03 §2.2; L06 §1.4C). | P(raised regions > control regions) on conversions, and a credible interval on the lift. |

**Output C: P(alive) = 0.09 for a customer who used to buy often** (an illustrative number).

| Follow-up question | Model or test that answers it | What a result looks like |
|---|---|---|
| Why might this customer be leaving? | A Cox model with behavior covariates such as support tickets and logins shows which behaviors go with faster churn (L04 §2.1). Its hazard ratios are associations (L04 §2.2). | In L04's worked example, HR = 1.160 per additional support ticket per 90 days: customers who file more tickets churn faster. This finding identifies the support experience as worth examining. Showing that tickets cause churn would require an experiment. |
| Is the customer worth a retention offer? | The most an offer can recover is P(alive) × CLV if still active (L08 §2.1 Part C). | With L08's if-active CLV of \$168 and a \$15 offer: 0.09 × \$168 = \$15.12, about equal to the cost. The break-even P(alive) is 15/168 = 0.089, and even the best case only breaks even. |
| Will the offer change what the customer does? | An uplift model fitted on a randomized test of the offer: target the customer only if τ̂ > c/v, the offer cost divided by the value of a conversion (L07 §1.4B). | τ̂ for customers like this one, compared with c/v. Below the threshold, the offer loses money even when it has a positive effect. |

**A method for any follow-up question.** The same four steps apply to every output in the course:

1. **Write the follow-up as a decision.** "Paid Search is under-credited" becomes "Should we move
   budget from Email to Paid Search next quarter, and how much?"
2. **Pick the model that answers that decision** (the three questions in Part 1 and the decision map in the slides).
3. **If no model answers it with the data in hand, design a test.** Randomize the change across
   customers, markets or regions, and analyze the result with a Bayesian A/B test (L01).
4. **Report the answer with its uncertainty:** the estimate, its interval, and the assumption it
   depends on (Theme 2).

The last criterion of the Sprint group rubric asks for exactly this: a follow-up analysis, with the
model, the data and how the result could change the recommendation (the rubric in Part 3).

---

## PART 2: Common Failure Modes

---

### 2.1 Common Analytical Failures in Marketing

Each failure below was taught in the lecture cited next to it.

**Evaluating a model on the data it was trained on.** A Qini coefficient computed on the training
rows of a random forest is inflated, because the forest has memorized those rows (L07 §1.4C). A
training MAPE measures fit; forecast accuracy is the MAPE on a held-out period (L10 §1.4C, Common
Misconception 5). The BG/NBD fit is checked against purchases in a holdout period (L08 §2.2).

**Ignoring informative censoring.** Kaplan-Meier assumes that a customer censored at age t would
have churned like the customers still at risk at age t (L04 §1.3). If a recent cohort was acquired
with a deep discount and churns faster, the curve's later ages, estimated only from older cohorts,
overstate that cohort's survival.

**Ranking MMM channels by β.** β is a channel's maximum weekly revenue, and β₀ is baseline revenue
from everything that is not advertising. Budget decisions rank channels by marginal ROI at current
spend (L03 §1.4C, §2.2).

**Using last-touch attribution to make budget decisions.** Last-touch systematically over-credits
closing channels and under-credits awareness channels (L03 §1.2, L06 §1.2). A budget reallocated on
last-touch alone tends to defund the channels that create the demand the closing channels capture.

**Applying average CLV to every acquisition channel.** A manager who says "Our average CLV is \$210,
so we can spend up to \$210 to acquire any customer" averages over channels that bring very different
customers (L08 Checkpoint Q5). In the same way, one blended CAC averages an \$8 organic customer with
a \$45 paid-search one. CLV and CAC are computed per channel (L08 §1.4D).

**Estimating uplift without random assignment.** If the control group consists of customers who did
not opt into a program, or customers from a different time period, the T-learner estimate mixes the
treatment effect with the differences between the groups (L07 §1.4A).

**Using a model outside the range of its data.** A revenue-maximizing price is located only inside
the range of prices observed (L02 §1.4B), and a conjoint profit-maximizing price is reliable only
inside the price range the survey tested (L09 §1.4D). A Prophet model trained through a temporary
campaign that it was not told about absorbs the campaign into the trend and biases the forecast
(L10 §1.4B).

**Applying one decision threshold regardless of the cost of being wrong.** For a button color that
can be reversed in minutes, P(B > A) = 0.80 may be enough; for a price change that is hard to
reverse, 0.99 may still be too uncertain (L01 §1.5).

**Treating the model output as the answer.** A high churn risk, a low P(alive) or a high CLV
identifies a customer worth a closer look. Spending on that customer is justified when the change
the offer causes, multiplied by what the customer is worth if kept, exceeds the offer's cost (the
retention rule, L04 §2.2 and L08 §2.1). The Shapley value for Paid Search is an input to a budget
recommendation, and the Prophet forecast is an input to an inventory order.

---

## PART 3: The Sprint

---

### 3.1 What the Sprint Covers

The Sprint case (FitLoop) includes six pre-run model outputs: Bayesian A/B testing (L01), price
elasticity (L02), Kaplan-Meier and Cox survival analysis (L04), Markov chains (L05), T-learner uplift
(L07) and BG/NBD CLV (L08). The individual quiz is built on these six outputs. MMM (L03), Shapley
attribution (L06), conjoint analysis and the profit-maximizing price (L09), and Prophet forecasting
(L10) are tested in Quiz 2 and HW11. They enter the Sprint through the group presentation and
write-up, whose last rubric criterion asks for a follow-up analysis that uses one of them
(rubric below). Quiz Q7 (model selection) can ask about any model in the course.

### 3.2 Timeline

| Date | What happens | Section |
|---|---|---|
| Mon 11/23 | FitLoop company briefing (Part A of the case) distributed. No model outputs yet. | both |
| Mon 11/30 | Model outputs released at the start of class; 60-minute individual quiz; group work for the rest of class | both |
| Wed 12/2 | Group work day | MKTG 3501 |
| Mon 12/7 | Group presentations | both |
| Wed 12/9 | Group presentations (continued) | MKTG 3501 |
| Mon 12/14 | Group write-up due (PDF, online); HW11 due | both |

Between 11/23 and 11/30 the preparation is conceptual. There are no numbers to analyze until the
model outputs are released on 11/30. Read the briefing with your group, decide which models address
each of the two business decisions, and decide which models you would leave out and why.

**What to review before 11/30:**
- The model reference card (a one-page summary of all models)
- The FitLoop company briefing distributed on 11/23
- Your own notes on any models you feel less confident about

Nothing extra is needed on 11/30. The model outputs are provided at the start of class.

---

### 3.3 How the Individual Scenario Quiz Works

On 11/30, after the model outputs are distributed, you complete a **10-question multiple-choice
Canvas quiz** (60 minutes, closes automatically). Your group's model output packet is open in front
of you: the quiz is open-book for the data, and communication with group members is not allowed.

The quiz is scored out of **100 Canvas points** and is the **Sprint — Individual 15%** of your
course grade (the group presentation and write-up are the other 15%). Canvas grades it immediately.

**The 10 question topics (one per area):**
1. **Bayesian A/B test**: interpreting posterior probability and credible intervals for a pricing decision
2. **Price elasticity**: using the elasticity coefficient alongside the A/B test for the same decision
3. **BG/NBD CLV**: P(alive) interpretation and retention campaign targeting
4. **Survival analysis**: Cox hazard ratios and the product intervention they imply
5. **Uplift model**: profit-maximizing targeting threshold and sleeping dog exclusion
6. **Markov chain**: steady-state interpretation and identifying the highest-leverage bottleneck
7. **Model selection**: which model applies to which decision (and which does not)
8. **Assumption identification**: causal and observational inference in the Cox model
9. **Integration**: identifying the most important analytical gap in a proposed recommendation
10. **Recommendation**: selecting the recommendation that best integrates uncertainty with business constraints

Each question has four answer choices, all correct or plausible statements about the FitLoop case.
The task is to identify the *best* answer to what the specific question asks. The correct answer
requires combining a specific number from your group's model outputs with the analytical reasoning
being tested.

---

### 3.4 Group Presentation and Write-Up

The presentation (12–15 minutes) and the 2-page write-up are scored on the same 40-point scorecard,
and the higher of the two scores on each criterion counts:

| Criterion | Points |
|---|---|
| Model selection: 2+ relevant models used, selection justified, at least one non-obvious exclusion reasoned | 15 |
| Output connection: recommendation explicitly tied to at least one specific number | 15 |
| Limitation and follow-up: one genuine analytical gap named, plus the follow-up analysis that would answer it, using a model from L03, L06, L09 or L10 (for example, conjoint willingness to pay for the price decision): what you would run, on which data, and how the result could change your recommendation | 10 |

The last criterion is the method of Theme 5 applied to FitLoop. The full guide, including the makeup
policy, is the Sprint Assessment Guide on the course site.

---

### Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class. The answers
> follow, and in class you may be asked to **explain your reasoning** for any of them. Using Copilot
> or Claude to reach an answer is expected; the explanation is yours to give.

These questions draw on all ten lectures.

1. A new analyst proposes running a Markov chain model to answer the question "did our
   re-engagement campaign cause more customers to become active?" What is wrong with this approach,
   and which model would you use?

2. You fit a Cox model and get HR = 0.52 (p < 0.001) for "annual_plan vs. monthly." Your manager
   says: "Great — let's push all customers toward annual plans to reduce churn." What question do
   you ask before making this recommendation?

3. An MMM returns β_TV = \$3.8M and β_Digital = \$2.1M. A colleague says: "TV is more valuable —
   let's cut digital and put it all in TV." Identify two things wrong with this reasoning.

4. A Prophet forecast has MAPE = 8% on a 13-week holdout. Your manager says the model is ready for
   production. What three additional checks would you run before agreeing?

5. You have been asked to build a CLV model, an uplift model and a survival model for the same
   customer dataset. In what order would you build them, and why?

---

### Checkpoint Answer Key

**Q1.** A Markov chain estimates transition probabilities between states from observed customer
movements (L05). Those probabilities describe all customers, including the ones who would have
become active without the campaign, and the chain has no control group. It measures where customers
moved, and the question asks what the campaign caused (Theme 1). The model for that question is a
randomized experiment: assign the campaign at random, compare the share of customers who become
active in each group with a Bayesian A/B test (L01), and, to find which customers respond, fit a
T-learner uplift model on the same experiment (L07).
*Common wrong answer:* "Compare the transition matrix before and after the campaign." The
before-after difference also contains every other change between the two periods.

**Q2.** Ask whether the plan changes behavior or whether committed customers choose the plan.
Customers select into the annual plan, and customers who commit to twelve months up front were
likely to stay on any plan (self-selection, L04 §2.2). The hazard ratio of 0.52 is an association,
and a Cox model adjusts only for the covariates that were measured. To find out whether moving a
customer to an annual plan reduces churn, offer a discount on the annual plan to a randomly chosen
subset of new customers and compare them with the controls (L04 §2.2).
*Common wrong answer:* "p < 0.001, so the effect is real." The p-value shows that the two groups
differ, and self-selection alone can produce that difference.

**Q3.** First, β is a channel's maximum weekly revenue, the plateau of its Hill curve. Budget
decisions depend on marginal ROI at current spend, MR = β · H′(A) / (1 − λ). A channel with a large
β that is close to saturation can return less from the next dollar than a channel with a smaller β
that is early on its curve (L03 §1.4C, §2.2). Second, moving the entire digital budget to TV is an
extreme reallocation, and L03 §2.2 treats any recommendation of that size as a reason for scrutiny.
MMM is fitted on observational spend, and the revenue gain it predicts for the move is an estimate
that a geo holdout could measure (L03 §2.2).
*Common wrong answer:* "TV's β is larger, so TV is more valuable." Ranking channels by β is the
misreading that L03 §2.2 warns about.

**Q4.** Any three of the following, all from L10:
- Repeat the holdout at several cutoff dates with `cross_validation()` and read the MAPE at the
  horizon the decision needs; one 13-week holdout scores one forecast (L10 §2.2).
- Check the interval's coverage on the holdout: about 95% of the holdout actuals should fall inside
  a 95% interval (L10 §2.2, Common Misconception 3).
- Check that the trend direction and the seasonal pattern match what you know about this business
  (L10 §2.2, "Before trusting the agent output").
- Check that the kept changepoints sit near known business events, and that any temporary campaign
  or permanent level shift in the history was specified as a holiday or a regressor (L10 §1.4B).
*Common wrong answer:* "8% is excellent, so no further checks are needed." A single holdout MAPE
describes one forecast, and the checks above test whether its accuracy and its interval hold up.

**Q5.** Survival analysis and CLV can be fitted on the existing transaction history; the uplift
model needs a randomized experiment, so it comes last. A defensible order:
1. **Survival analysis (L04):** which customers are at risk and how soon, and which behaviors go with
   faster churn.
2. **CLV (L08):** what each customer is worth, and whether an offer could pay off even in the best
   case, P(alive) × CLV if still active > offer cost (L08 §2.1 Part C).
3. **Uplift (L07):** run a randomized test of the offer, fit the T-learner on it, and target the
   customers whose τ̂ exceeds c/v.

This order follows the retention rule (L04 §2.2): survival analysis and CLV say whom to look at, and
only the experiment and the uplift model say whether the offer changes behavior. Building survival
analysis and CLV in the other order is equally defensible, because neither depends on the other.
*Common wrong answer:* "Uplift first, because it is the causal model." The uplift model cannot be
fitted until a randomized test has been run.

---

## PART 4: Going Further

---

### 4.1 Extensions Beyond the Course

Each model in this course has a body of advanced literature:

**Causal Inference:**
- Difference-in-differences, synthetic control, regression discontinuity
- Key resource: Cunningham, *Causal Inference: The Mixtape* — free at
  mixtape.scunning.com (the current online edition is titled *The Remix*)

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

Looking back over the ten lectures, these capabilities will most determine your effectiveness as a
marketing analyst:

**1. Model selection judgment.** Knowing which model answers which question, and which questions a
given model cannot answer.

**2. Checking assumptions.** Before presenting any result: what are the critical assumptions, are
they likely to hold in this context, and what would change if they fail?

**3. Uncertainty communication.** A point estimate reported without its interval invites false
precision. Saying "our estimate is X, and here is how confident we are and why" is a career-long
practice.

**4. Incremental thinking.** Every optimization question in this course (budget reallocation,
targeting threshold, profit-maximizing price) comes down to marginal reasoning: what does one more
dollar do, and what does one more customer in this segment do?

**5. Answering the follow-up.** A model output is the start of the next question. The four-step
method in Theme 5 turns that question into a decision, a model or a test, and an answer with its
uncertainty.

---

### Reflection Questions

These are not graded. They are for your own synthesis at the end of the course.

1. Which of the ten models do you expect to use most in your career, and why?

2. Which model's assumptions are most likely to be violated in the domain you plan to work in? What
   would you do about it?

3. The course philosophy is that analytical judgment matters more than model execution. Give a
   specific example from the ten lectures where judgment was the hardest and most important step.

4. What is one analytical question you are now equipped to answer that you could not answer before
   this course? Which follow-up analysis or test would strengthen your answer?
