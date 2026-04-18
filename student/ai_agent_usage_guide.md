# Using AI Agents in This Course
## A Guide to Effective and Responsible Use

---

## What AI Agents Can Do

AI agents (Claude, GPT-4, Copilot, and similar tools) are genuinely capable
analytical assistants. When used well in this course, an agent can:

- **Write and execute correct estimation code** — fitting a BG/NBD model,
  computing Kaplan-Meier estimates, running a T-learner, fitting Prophet
- **Interpret model outputs** — explaining what a hazard ratio of 1.42 means,
  describing the shape of a CLV distribution, identifying which features
  drive the highest predicted uplift
- **Propose analytical recommendations** — suggesting which customers to target,
  which channel to reallocate budget from, whether a price increase is likely
  to raise revenue
- **Flag potential problems** — noticing that a correlation of −0.48 between
  frequency and spend violates the Gamma-Gamma independence assumption,
  warning that last-touch attribution will overstate Email's contribution
- **Explain derivations** — walking through the OLS slope formula step by step,
  deriving the Bayesian update rule, explaining why the softmax shares sum to 1

The agent is a capable collaborator. The Part B questions in this course are
designed to be run with agent assistance.

---

## What You Must Supply

Agents make systematic errors. The errors are not random — they follow patterns
that you can learn to detect. Your job is to have enough independent analytical
understanding to:

**1. Detect confidently-wrong outputs.**  
Agents sometimes produce outputs that are internally coherent but factually wrong.
Examples specific to this course:
- Reporting P(B>A) = 0.97 and then interpreting it as "B would win 97% of
  repeated experiments" (frequentist misinterpretation of a Bayesian quantity)
- Fitting OLS on price and quantity without checking for endogeneity, then
  reporting a positive slope as "higher price causes higher demand"
- Computing adstock recursion with the wrong direction (applying decay to the
  wrong period), producing plausible-looking numbers that are wrong
- Misidentifying the proportional hazards assumption violation when checking
  a Cox model
- Reporting Shapley values that do not sum to the overall conversion rate
  (efficiency axiom violation — the result is simply wrong)

**2. Add context the agent lacks.**  
An agent does not know your business context, your data collection process, or
the strategic question you are trying to answer. It will produce technically
correct outputs that are irrelevant to the actual decision if you do not
frame the problem carefully.

**3. Take responsibility for the conclusion.**  
When you submit an answer, you are asserting that it is correct. "The agent
told me" is not an explanation for a wrong answer on the autograder or in
a presentation. The autograder does not accept agent outputs directly — it
tests whether you can correctly record, verify, and submit the right value.

---

## Verified vs. Unverified Agent Output: The Test

Before recording any agent output in your notebook, apply this three-part check:

**Sign check:** Does the sign make sense?
- Elasticity should be negative (higher price → lower quantity)
- β_price in a conjoint model should be negative
- P(alive) should be between 0 and 1
- Hazard ratios should be positive
- Shapley values should sum to the overall conversion rate

**Order-of-magnitude check:** Is the number plausible?
- A price elasticity of −0.05 is surprisingly inelastic; −8.4 is suspiciously elastic
- A CLV of $3 or $300,000 for a $18/month subscription is both suspicious
- P(B>A) = 0.9999 with n=50 per variant is implausible

**Formula check:** Does it match what the formula predicts?
- If you computed β̂₁ = −27 by hand in Part A, the agent's OLS slope should match
- If the KM table gives Ŝ(5) = 0.8977 in Part A, the agent's survival curve
  should match at t=5
- If the threshold = c/v = 4/30 = 0.133 from Part A, agent-reported targeting
  percentages should be consistent with this threshold

If any check fails, either the agent made an error or you computed Part A wrong.
Investigate before submitting.

---

## Common Agent Errors by Model

### Bayesian A/B Testing
- **Frequentist interpretation of P(B>A):** The agent may describe P(B>A) as
  a long-run frequency ("B wins 97% of the time"). Correct interpretation:
  posterior probability given current data and prior.
- **Wrong posterior parameters:** Agents occasionally add total visitors instead
  of non-conversions to β. Check: posterior_beta = prior_beta + (n − k).

### Price Elasticity / OLS
- **Endogeneity blind spot:** Agents will fit OLS without raising endogeneity
  concerns unless you ask. Always prompt: "Are there any endogeneity concerns
  with this dataset?"
- **Log-log interpretation:** Agents sometimes report level-level coefficients
  but call them elasticities. Verify: in a log-log model, the slope coefficient
  equals the elasticity directly.

### Marketing Mix Modelling
- **Adstock direction:** The recursion is A_t = S_t + λ·A_{t−1}. Some agents
  write it backwards. Check: Week 1 adstock should equal Week 1 spend.
- **Hill function at EC50:** H(EC50) = 0.5 always. If the agent's Hill curve
  does not pass through (EC50, 0.5), there is a parameterisation error.

### Survival Analysis
- **Censoring handling:** Agents may drop censored observations instead of
  treating them correctly. Check: the at-risk count should decrease monotonically,
  but not by more than the number of events at each time point.
- **PH assumption:** Agents will not automatically check the proportional hazards
  assumption. Prompt explicitly: "Please create log-log plots to check the PH
  assumption for each covariate."

### Markov Chains
- **Row vs. column multiplication:** State evolution is π_{t+1} = π_t · P
  (row vector × matrix). Some agents transpose this. Check: each row of P
  should sum to 1.
- **v2 precision:** Computed from v1, not directly from v0. If the agent
  computes v2 = v0 · P², verify against the two-step multiplication.

### Attribution (Shapley)
- **Efficiency check:** The agent's Shapley values must sum to the overall
  conversion rate. This is non-negotiable. If they don't, there is a bug.
- **Causation claim:** Agents sometimes describe Shapley values as measuring
  "incremental impact." They measure marginal contribution within observed paths —
  not causal incrementality. Correct this if it appears in agent output.

### Uplift Modelling
- **ATE vs. CATE confusion:** ATE is the average across all treated customers.
  The T-learner produces CATEs (individual-level estimates). These are different
  things. Verify the agent correctly distinguishes them.
- **Sleeping dogs:** If the agent reports 0% sleeping dogs, this is suspicious.
  Verify by checking the fraction of customers with τ̂(x) < 0.

### CLV (BG/NBD)
- **Gamma-Gamma assumption:** The agent may fit the Gamma-Gamma model without
  checking the frequency-spend correlation. Compute it yourself and interpret.
- **P(alive) vs. loyalty:** An agent may describe a customer with many historical
  purchases as "loyal" even if P(alive) = 0.05. Recency dominates frequency.

### Conjoint / WTP
- **β_price sign:** Must be negative. If the agent returns a positive price
  coefficient, the model is misspecified or the data is incorrectly structured.
- **WTP ≠ optimal price:** Agents will often treat mean WTP as the recommended
  price. This is wrong — WTP is the upper bound of the demand curve for that
  feature, not the revenue-maximizing price point.

### Prophet
- **Multiplier vs. additive seasonality:** Prophet's default is additive
  seasonality (seasonal effect adds to trend). For the worked examples in this
  course, multiplicative is often more appropriate. Check which mode the agent used.
- **MAPE context:** Agents will sometimes evaluate MAPE against a threshold
  (e.g., "18% is too high") without stating the business context. MAPE
  acceptability is context-dependent — push back if the agent makes a threshold
  claim without justification.

---

## How to Write Good Agent Prompts

The quality of agent output depends heavily on the quality of your prompt.
For Part B, the context prompt is provided for you — use it exactly as written.
For other uses:

**Include:**
- The specific model you want fitted
- The variable names as they appear in your dataset
- Any parameters you have already established from Part A
- What output you need (specific variable values, not just "run the analysis")

**Avoid:**
- Vague requests like "analyse this data" without specifying the model
- Asking for interpretation before verifying the numbers
- Accepting the first output without cross-checking against Part A

**Example of a well-specified prompt:**
```
I am fitting a Cox proportional hazards model to predict subscription churn.
Dataset: survival_data.csv
Event: churned (1=churned, 0=censored)
Time: tenure_months
Covariates: support_tickets, contract_type (annual=1, monthly=0), usage_score

Please:
1. Fit the Cox model
2. Report the hazard ratio and 95% CI for each covariate
3. Report the p-value for each coefficient
4. Check the proportional hazards assumption using Schoenfeld residuals
5. Print all results clearly labelled
```

---

## The Part C Questions

Part C questions test whether you understand the model well enough to evaluate
an agent-produced interpretation. An agent can produce an interpretation of
these questions — and that interpretation may be correct. Your job is to know
*why* it is correct (or to catch it when it is wrong).

Submitting an answer to Part C that the agent gave you is acceptable — *if you
verified it*. Submitting an answer to Part C that the agent gave you without
understanding why it is correct means you have not achieved the learning objective,
and you will be unable to apply that reasoning in the Sprint or in
subsequent coursework.

The test is simple: could you defend your Part C answer to a classmate who chose
differently?
