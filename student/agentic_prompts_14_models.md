# Agentic Programming Prompts
## Every Marketing Analytics Model

> **The course is L00–L11.** Models 12–14 (Difference-in-Differences, Customer
> Segmentation, CausalImpact) are not taught this term and carry no homework; their
> prompts are kept here as reference.

**L01–L10 (every taught model): "On your own data".** Each section is the real-world
version of that homework's agent prompt, in six parts: the question it answers · data
needed and deal-breakers · tools (package, functions, maintained status, checked
2026-11-04) · a general prompt with `<placeholders>` to fill in · what you get back · checks
before trusting it. Each ends with the values the homework prompt pins for grading that you
should **not** copy to real data. The homework prompt is still the one to use for the
homework. This page is for your own data, and it is not assessed.

**L12–L14 (reference only)** keep the older three-prompt format: a Context Prompt to orient
the agent, a Build Prompt to generate the analysis, and a Validation Prompt to check the
output before trusting it.

---

## L01 — Bayesian A/B Testing

> **On your own data.** This section is the real-world version of Homework 1's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the counts, the prior, the
> number of draws and the seed so every answer can be checked. Use the prompt below for
> your own test, and read "What not to copy from the homework" first.

### 1. The question it answers
Given what you saw in a test, how likely is it that variant B really converts better
than A, and by how much (as a percentage lift), with an honest range around that?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 1. In practice: visitors and conversions for each variant, from the same period,
with each visitor counted once.

Stop before running if any of these is true:
- **Users were not randomly split**, or one variant ran at a different time. Then the
  difference may be the period, not the variant.
- **Visitors are counted more than once** (sessions rather than people), or conversions
  are not 0/1 per visitor. The Beta-Binomial model assumes independent 0/1 outcomes.
- **The conversion rate drifted during the test** (a promotion started half way). The
  posterior then averages two different rates.
- **Very little data.** The prior then decides the answer (L01 Section 1.3). Say so, or
  wait for more data.

### 3. Tools
Checked against PyPI on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Posterior (conjugate update) | No package: Beta(α + conversions, β + non-conversions) | n/a | n/a |
| Posterior draws | `numpy`: `rng = np.random.default_rng(seed)`, then `rng.beta(a, b, size)` for each arm | Yes (1.26.4) | Yes |
| Or | `scipy.stats.beta(a, b).rvs(size, random_state=rng)` | Yes (1.13.0) | Yes: 1.18.1 released 2026-08-21 |

One thing to know, confirmed on the course env: **if you give each arm its own
`random_state=42`, the two arms reuse the same random stream**, so their draws are
correlated (0.11 in a test case) and P(B > A) moves. Create **one** generator and draw both
arms from it.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I ran an A/B test. Variant A: <n_A> visitors, <k_A> conversions. Variant B: <n_B>
visitors, <k_B> conversions. Visitors were randomly assigned, over <dates>.
Prior for both variants: Beta(<alpha>, <beta>), chosen because <reason, e.g. past
conversion rates around 4%>.

1. Posterior for each variant: Beta(alpha + conversions, beta + non-conversions).
   Print both, with their means.
2. Draw <S, e.g. 100,000> samples from each posterior using ONE numpy Generator
   (np.random.default_rng(<seed>)) for both arms. Do not reseed per arm, and do not
   sort the draws.
3. From the paired draws report: P(theta_B > theta_A); the expected relative lift
   E[(theta_B - theta_A) / theta_A] as a %; and its 95% interval (2.5th and 97.5th
   percentiles of the same ratio).
4. Plot both posteriors on one chart.

Show the code, then a results table. Do not invent any number.
```

### 5. What you get back
- Two posterior distributions, with their means.
- P(B > A): a probability, not a p-value.
- The expected percentage lift and a 95% interval for it.

What it does **not** give you: a decision. Whether 0.90 or 0.99 is enough depends on how
costly and reversible the change is (L01 Section 1.5).

### 6. Checks before trusting it
The L01 note has no "What to verify" list. These follow its "What to watch for" and Section 2.2.
1. Each **posterior mean sits between the prior mean and the observed rate**, closer to
   the observed rate when there is more data.
2. **P(A > B) + P(B > A) ≈ 1.**
3. **Mean of the ratio is not the ratio of the means.** The expected lift is the average
   of (θ_B − θ_A)/θ_A over the draws. It is not (mean θ_B − mean θ_A) / mean θ_A. If the
   agent computed the second, ask for the first.
4. **The interval and the probability agree.** The 95% interval excludes 0 exactly when
   P(B > A) is above 0.975 or below 0.025.
5. **A wide interval is not a failed test.** It measures how sure you are, which is a
   separate question from which way the lift goes.
6. **Read the result as a probability statement** ("there is a 95% probability that B's
   rate is higher"), not as a claim about repeated experiments.

### What not to copy from the homework
- **The prior.** The homework gives it to you. On your own test, set it from past
  conversion rates, and keep it weak unless you have a reason not to.
- **`random_state=42`.** Any fixed seed makes the numbers reproducible. The value 42 has
  no meaning, and seeding each arm separately is the mistake above.
- **The visitor and conversion counts.** They are the graded scenario.

**Do keep** the same prior for both arms, the relative lift as the estimand, and around
100,000 draws, which is enough for the second decimal to be stable.

---

## L02 — Price Elasticity and OLS

> **On your own data.** This section is the real-world version of Homework 2's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset and the model
> so every answer can be checked. Use the prompt below for your own data, and read "What
> not to copy from the homework" first.

### 1. The question it answers
By what percentage do units sold change when the price changes by 1% (the elasticity),
holding other things fixed?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 2. In practice: units and price per store (or product) per period, plus anything else
that moves demand (competitor price, promotions or displays, season).

Stop, or change plan, if any of these is true:
- **The firm set prices in response to demand** (raise when busy, discount when slow).
  That is endogeneity, and it biases the elasticity, usually toward zero. More data does
  not fix it. An instrument does (L02 Section 1.4C).
- **Prices barely vary.** A narrow price range cannot identify the slope, let alone curvature.
- **Units are zero in some periods.** log(0) is undefined. Decide how to handle zeros
  before fitting; do not let the agent silently drop them.
- **You want a price far outside the observed range.** A constant elasticity is a local
  approximation, so do not extrapolate it.

### 3. Tools
Checked against PyPI on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Log-log regression with fixed effects | `statsmodels`: `smf.ols("log_units ~ log_price + ... + C(store_id)", data).fit()` | Yes (0.14.2) | Yes: 0.15.0 released 2026-08-27 |
| Standard errors clustered by store | `.fit(cov_type="cluster", cov_kwds={"groups": df["store_id"]})` | Yes | Yes |
| Instrumental variables (if you have an instrument) | `linearmodels`: `IV2SLS` | No | Yes: 7.1 released 2026-10-16 |

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have <weekly | daily> sales data in <file.csv>: columns <unit>, <time>, <units>,
<price>, and these demand shifters: <competitor_price, promo_flag, ...>. Prices were set
by <how prices were set, e.g. a national price list changed quarterly>.

1. Check before fitting: how many rows have units = 0 or price <= 0 (stop and ask me if
   any); how much price varies within each <unit>; how many levels each fixed-effect
   column has.
2. Create log_units = log(units) and log_price = log(price) (and log of any other price).
3. Fit OLS with statsmodels:
   log_units ~ log_price + <other regressors> + C(<unit>) [+ C(<time>)]
   with standard errors clustered by <unit>. Report the full summary table.
4. State the own-price elasticity (the log_price coefficient) with its 95% CI.
5. Plot predicted vs actual log_units with a 45-degree line.

Show the code, then the results. Do not invent any number.
```

### 5. What you get back
- An elasticity with a standard error, and the coefficients on the other regressors.
- A predicted-vs-actual plot.

What it does **not** give you: proof that the elasticity is causal (that depends on how
prices were set), or a revenue-maximising price. Under a constant elasticity, revenue is
monotone in price. Finding a peak needs a model whose elasticity changes with price,
fitted inside the observed range (L02 Section 1.4B).

### 6. Checks before trusting it
The first four are the L02 note's "What to verify before trusting the agent's elasticity estimate".
1. The **sign is negative**.
2. The **magnitude is plausible**: most consumer goods sit between −0.3 and −4.0. Outside
   that, look for a data problem.
3. **Fixed effects are in** if you have panel data. Without them, store-level differences bias the estimate.
4. **Predicted vs actual** clusters around the 45-degree line.
5. **R² is not the test.** A very high R² (above 0.99) is suspicious, and a high R² says
   nothing about bias.
6. **Ask how prices were set.** No regression output can show endogeneity. If prices
   followed demand, say the estimate is likely biased toward zero.

### What not to copy from the homework
- **"Use random seed 42."** OLS has no randomness, so the seed does nothing.
- **`C(product_id)`** in a file with one product. With one level it drops out. Use the
  fixed effects your data actually has.
- **Default (non-clustered) standard errors.** On panel data, cluster by store.

**Do keep** the log-log form (so the coefficient is the elasticity), the competitor price
and promotion controls, and store fixed effects.

---

## L03 — Marketing Mix Modelling

> **On your own data.** This section is the real-world version of Homework 3's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the parameter
> bounds and the optimiser's seed so every answer can be checked. Use the prompt below for
> your own data, and read "What not to copy from the homework" first.

### 1. The question it answers
How much revenue does each channel's spend bring in, counting carry-over (adstock) and
diminishing returns (saturation)? And how should a fixed total budget be split across channels?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 3. In practice: weekly spend per channel and weekly revenue, ideally two years or
more, plus the other things that move revenue (season, price, promotions, distribution).

Stop, or change plan, if any of these is true:
- **Channel spends move together** (always raised and cut at the same time). The model
  cannot separate them.
- **No controls for season or promotions.** Their effect is then credited to whatever
  channel happens to spend at the same time (L03 Section 1.4C).
- **Less than about two years of weekly data.** There are too few points for nine curve
  parameters plus the regression.
- **You need a causal answer for one channel.** An MMM is observational. A geo-holdout
  test answers that (L03 Section 2.2).

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Joint fit of adstock + Hill + regression | `scipy.optimize.differential_evolution` over all curve parameters, with the betas by `numpy.linalg.lstsq` inside each evaluation | Yes (scipy 1.13.0) | Yes: scipy 1.18.1 released 2026-08-21 |
| Budget split at a fixed total | `scipy.optimize.minimize(method="SLSQP")` with an equality constraint, **started from several points including the corners** | Yes | Yes |
| Bayesian MMM with priors and uncertainty | `pymc-marketing`: `MMM` with `GeometricAdstock`, `HillSaturation` | No: needs Python 3.12 or newer | Yes: 1.3.0 released 2026-10-22 |
| Or | Google's `google-meridian` | No | Yes: 2.1.0 released 2026-09-25 |

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have <N> weeks of data in <file.csv>: <week>, spend columns <channel_1>, <channel_2>,
... (in <unit>), revenue (in <unit>), and controls <season / price / promo columns, or
"none">.

1. Check before fitting: the correlation between every pair of spend columns (warn me
   above 0.8), the weeks with zero spend per channel, and the number of weeks.
2. Model: adstock A_t = S_t + lambda * A_{t-1} per channel; Hill
   H(A) = A^alpha / (EC50^alpha + A^alpha) per channel; revenue = beta_0 +
   sum_c beta_c * H(A_c) + <controls> + error.
3. Fit everything JOINTLY: minimise the total squared error over all lambdas, EC50s and
   alphas at once with scipy.optimize.differential_evolution, solving the betas by least
   squares inside each evaluation. Bounds: lambda in [0, 0.95], EC50 within the range of
   each channel's ADSTOCK, alpha in [0.3, 3]. Report every parameter and the in-sample R^2.
4. Marginal ROI per channel at current spend, using steady-state adstock A = S/(1 - lambda):
   MR = beta * H'(A) / (1 - lambda).
5. Budget split: hold the total at <current average | a given amount>. Maximise predicted
   revenue with SLSQP from several starting points, and also evaluate every corner
   (all budget in one channel). Report the best, the current split, and both predicted revenues.

Show the code, then one results table per step. Do not invent any number.
```

### 5. What you get back
- λ, EC50 and α for each channel, the betas, and an in-sample R².
- The marginal return on the next unit of spend, per channel.
- A recommended split at a fixed total, with predicted revenue at the current and recommended split.

What it does **not** give you: causal effects (spend may follow demand), uncertainty on
the parameters (this is a least-squares point fit), or a safe answer far outside the
spend levels you observed.

### 6. Checks before trusting it
The first four are the L03 note's "What to verify in the agent output".
1. **Every λ is between 0 and 1.** λ = 0 (no carry-over) is allowed.
2. **EC50 is plausible against the adstock series**, not against raw spend. Adstock is
   larger than spend by about 1/(1 − λ).
3. **Fitted weekly revenue tracks actual weekly revenue** (in-sample R²).
4. **The budget moves toward the higher marginal ROI**, with MR = β × Hill slope ÷ (1 − λ).
   It does not simply move toward the channel furthest below its EC50.
5. **Equal marginal ROI is not proof of the best split.** With S-shaped curves (α > 1) an
   equal-MR split can be a minimum. Compare against the corners (L03 Section 1.4C).
6. **Large reallocations deserve suspicion.** Moving most of the budget is a reason to
   test with a geo-holdout before acting.

### What not to copy from the homework
- **`seed=42` and the exact bounds** (EC50 in [1, 300]). The bounds fit the course
  dataset's scale. Set them from your own adstock range.
- **A_0 = 0.** Fine for a toy. On real data, start the recursion before the period you fit (a burn-in).
- **No control variables.** The homework has none. Real data almost always needs them.

**Do keep** the joint fit (fitting channel by channel leaves EC50 unidentified), the
steady-state adstock in the budget step, and the 1/(1 − λ) factor in marginal ROI.

---

## L04 — Survival Analysis

> **On your own data.** This section is the real-world version of Homework 4's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the encoding
> and the time points so every answer can be checked. Use the prompt below for your own
> data, and read "What not to copy from the homework" first.

### 1. The question it answers
How long do customers stay? What share is still there at month 6, 12 or 24? And which
customer traits go with a higher or lower churn hazard?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 4. In practice: one row per customer, with time observed (start to churn, or start
to today), a churned flag (1 = churned, 0 = still active, i.e. censored), and the traits
you want to compare.

Stop before fitting if any of these is true:
- **You kept only the customers who churned**, or **treated still-active customers as
  churned**. Both throw away what survival analysis exists to use (L04 Section 1.3).
- **Censoring is informative**: customers leave your data *because* they were about to
  churn (for example, accounts removed when a payment fails).
- **Newer cohorts behave differently.** Their short histories then make the long-run tail
  look better than it will be.
- **The trait was chosen by the customer** (plan type, opt-in programme). The hazard ratio
  is then an association, not the effect of moving someone to that group (L04 Section 2.2).

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Survival curves | `lifelines`: `KaplanMeierFitter().fit(durations, event_observed)`, `.survival_function_at_times([...])` | Yes (0.29.0) | Yes: 0.30.3 released 2026-03-05 |
| Compare two groups | `lifelines.statistics.logrank_test` | Yes | Yes |
| Hazard ratios | `CoxPHFitter().fit(df, duration_col=, event_col=)`, `.summary`, `.concordance_index_` | Yes | Yes |
| Proportional-hazards check | `cph.check_assumptions(df, p_value_threshold=0.05)` | Yes | Yes |

One thing to know, confirmed on the course env: **`check_assumptions` flags at p < 0.01
by default**, not 0.05. Set `p_value_threshold` yourself, or the agent's "no violations"
uses a different cutoff from the one you report.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have customer data in <file.csv>: <customer_id> (not a covariate), <duration> (in
<months>), <churned> (1 = churned, 0 = still active), and covariates <list, with their
units>. Group of interest: <e.g. plan_type>.

1. Check before fitting: the share censored, durations <= 0 (stop if any), and churn
   rate by start cohort (warn me if newer cohorts churn much faster).
2. Kaplan-Meier curves for each level of <group>, on one chart. Report S(t) at
   t = <6, 12, 24> for each group, and a log-rank test between groups.
3. Cox model with <covariates>. Encode <categorical> as 0/1 with <reference level> = 0.
   Report hazard ratios, 95% CIs, p-values and the concordance index.
4. Proportional hazards: check_assumptions with p_value_threshold=0.05. Report which
   covariates, if any, violate it.
5. Conditional churn: P(churn between <t1> and <t2> | still active at <t1>) = 1 - S(t2)/S(t1).

Show the code, then one results table per step. Do not invent any number.
```

### 5. What you get back
- Survival curves, survival at your chosen months, and a log-rank p-value.
- A hazard ratio per covariate, *per unit* of that covariate, with a CI.
- A concordance index and a list of any proportional-hazards violations.

What it does **not** give you: the probability that a given customer churns (a hazard
ratio multiplies a rate, not a probability), or the effect of moving a customer to a
different plan when customers chose their plan.

### 6. Checks before trusting it
The first five are the L04 note's "What to verify before trusting the output".
1. Each **KM curve starts at 1.0 and only steps down**.
2. **Every hazard ratio is positive.**
3. **C-index is above 0.6.**
4. **Engagement traits have HR < 1, friction traits HR > 1.** If not, check the coding.
5. **A group the customer chose is reported as an association**, not an effect.
6. **Do not rank traits by their HR.** HRs are per unit, so a covariate measured in
   different units gets a different HR. Compare effects for realistic changes (HR^k for k units).
7. **A covariate that violates proportional hazards** has an HR that is only an average over time. Say so.

### What not to copy from the homework
- **`random_state=42`.** Kaplan-Meier and Cox are deterministic fits, so the seed does nothing.
- **"No penalizer, original scale"** is a reproducibility pin. On many correlated
  covariates a small penalizer is reasonable, but then report that you used one.
- **t = 6, 12, 24 months.** Choose the time points your decision is about.

**Do keep** censored customers in the data, a single 0/1 coding with a named reference
level, and an explicit p < 0.05 threshold for the PH test.

---

## L05 — Markov Chains

> **On your own data.** This section is the real-world version of Homework 5's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the horizon and
> the rounding so every answer can be checked. Use the prompt below for your own data, and
> read "What not to copy from the homework" first.

### 1. The question it answers
Customers move between states (Active, At-Risk, Churned) from one period to the next.
Where will the customer base be in k periods? And, when Churned is permanent, how long
does a customer in each state last on average before churning?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 5. In practice: one row per customer per period with that period's state, for at
least two consecutive periods, using the same state definitions throughout.

Stop, or change plan, if any of these is true:
- **The states are fuzzy or were redefined part way.** Transition rates then mix two definitions.
- **Dynamics change over time** (season, a product change, a campaign). One pooled matrix
  then averages them, and projections can be far off.
- **Too many states for your data.** Many cells get a handful of transitions, and those
  probabilities are mostly noise.
- **Churned customers come back in your data.** Either it is a data error, or there is a
  win-back flow and Churned is not absorbing. Decide which before fitting.

### 3. Tools
No dedicated package is needed: `pandas` to count transitions and `numpy` for the matrix
algebra (both in the course Codespace). The core operations are `v @ P` (one period
ahead), repeated multiplication (k periods), and
`np.linalg.solve(np.eye(m) - Q, np.ones(m))` for the expected time to absorption, where
`Q` is P restricted to the non-absorbing states.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have <file.csv> with columns <customer_id>, <period>, <state>. States: <list>.
<Absorbing state, e.g. Churned> is permanent: <yes | no, we run win-back>.

Convention, used in every step: P[i, j] = P(next period = j | this period = i). Rows
are the CURRENT state and every ROW sums to 1. A distribution is a ROW vector, so one
period ahead is v @ P. Never P @ v, and never transpose P.

1. Estimate P by counting transitions between consecutive periods for the same customer
   and normalising each row. Print P with labels and the count behind each row.
2. Check: every row sums to 1 (within 0.001); the absorbing state's row is [0, ..., 1];
   report any customer who leaves the absorbing state.
3. Project the current distribution <k> periods ahead deterministically (v @ P, k times).
   Do not simulate individual customers. Plot each state's share over time.
4. If there is an absorbing state: expected periods until absorption for each
   non-absorbing state, t = (I - Q)^-1 * 1. If there is not: the steady state pi with
   pi @ P = pi.
5. Scenario: raise P[<from> -> <to>] by <x> points, taking it from P[<from> -> <which
   entry gives way>], so the row still sums to 1. Report the change in step 4's answer.

Show the code, then one results table per step. Do not invent any number.
```

### 5. What you get back
- A labelled transition matrix and the counts behind it.
- The state mix k periods ahead.
- Expected periods until churn from each state (absorbing chain), or the long-run mix
  (chain with no absorbing state).
- The change in that answer under your scenario.

What it does **not** give you: proof that a campaign *would* change the transition rate.
The scenario assumes the change, so measuring it needs a randomised holdout (L05 Section 1.4C).

### 6. Checks before trusting it
The first five are the L05 note's "What to verify".
1. **Every row of P sums to 1** (within 0.001). If the *columns* do, P has been transposed.
2. **The steady state satisfies πP ≈ π.** Multiply it out.
3. **An absorbing Churned row is exactly [0, 0, 1].**
4. **The transitions make sense**: At-Risk → Churned is larger than Active → Churned.
5. **Every expected time to absorption is positive, and the healthier state's is larger.**
   If At-Risk outlasts Active, suspect a transposed P.
6. **With one absorbing state, the steady state is always 100% in that state.** That is
   expected and carries no planning information, so use expected time to absorption instead (L05 Section 1.4B).

### What not to copy from the homework
- **12 months, starting from "the current observed distribution", and the rounding.**
  Use your own planning horizon and current mix.
- **"All customers start Active in month 1."** Real data has customers joining at
  different times. Count transitions, do not assume a common start.

**Do keep** the row-vector convention, deterministic projection (there is nothing to
simulate), and computing from the unrounded matrix.

---

## L06 — Multi-Touch Attribution (Shapley Values)

> **On your own data.** This section is the real-world version of Homework 6's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the channels
> and the output format so every answer can be checked. Use the prompt below for your own
> data, and read "What not to copy from the homework" first.

### 1. The question it answers
How should credit for conversions be shared among the channels a customer touched, and
how different is that from giving all credit to the last touch?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 6. In practice: one row per customer journey with the channels in touch order and a
0/1 converted flag, **including journeys that did not convert**.

Stop, or change plan, if any of these is true:
- **You only have converting paths.** Neither Shapley nor removal effects can be computed.
- **Most conversions come from single-channel paths.** Every method then gives the same
  answer, so check the share of multi-channel paths first (L06 Section 2.1).
- **You want to know what happens if you switch a channel off.** Neither method answers
  that. It needs a holdout experiment (L06 Section 1.4C).
- **Many channels.** Exact Shapley grows as n! orderings, so beyond about 10 channels it
  needs sampling, and large coalitions become sparse.

### 3. Tools
Checked against PyPI on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Last touch, Shapley (exact) | No package: `pandas` plus `itertools.permutations` over the channels | Yes | n/a |
| Markov removal effects | `numpy`: build the transition matrix (Start, channels, Conversion, Null) and solve for the conversion probability | Yes | n/a |
| Or, packaged Markov attribution | `ChannelAttribution`: `markov_model(...)` | No | Yes: 2.2.5 released 2026-06-08 |

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have journey data in <file.csv>: <journey_id>, <path> (channels in touch order,
separated by '<sep>'), <converted> (0/1). It includes non-converting journeys.
Channels: <list>.

1. Check first: number of journeys, conversion rate, and the share of conversions from
   multi-channel paths. Stop and tell me if almost all are single-channel.
2. Last touch: the percent of conversions where each channel was the final touch.
3. Shapley: enumerate all channel orderings. Use the coalition value
   v(S) = (conversions on paths whose channels all lie in S) / (all paths),
   so v({}) = 0 and v is monotone in S. Do NOT use the conversion rate within the
   paths confined to S. Average each channel's marginal contribution over orderings.
4. Markov removal effects: build the first-order transition matrix with Start,
   Conversion and Null states. For each channel, redirect every transition INTO it to
   Null, and report the RELATIVE drop (p_full - p_removed) / p_full.
5. One table: last touch %, Shapley value, Shapley %, removal effect, per channel.
   One grouped bar chart.

Show the code, then the results. Do not invent any number.
```

### 5. What you get back
- Three credit views per channel: last touch, Shapley and removal effect.
- A chart showing where they disagree.

What it does **not** give you: incrementality. Shapley and removal effects describe how
channels co-occur on converting paths. Removing a channel *from the model* is not
switching it off *in the world*.

### 6. Checks before trusting it
The first four are the L06 note's "What to verify before trusting the output".
1. **Shapley values sum to about the overall conversion rate.** This holds by
   construction, so it checks the arithmetic, not the business story.
2. **Every Shapley value is non-negative.** With the v(S) above it must be, so a negative
   one means the code used a different v.
3. **Every removal effect is between 0 and 1.**
4. **A high Shapley value means "on many converting paths", not "needed".** A channel
   that rides along everywhere still scores high.
5. **Removal effects do not sum to 1.** They are not credit shares, so do not present them as one.

### What not to copy from the homework
- **`seed=42`.** Nothing here is random, so the seed does nothing.
- **"Percent, e.g. 12.3, not 0.123"** is a grading format. Report whatever your readers expect.
- **Four named channels.** Use your own channel list, and decide how to treat repeated
  touches of the same channel (the course counts them once).

**Do keep** the all-paths v(S), the redirect-to-Null construction, and the relative drop.
These are the method, not grading pins.

---

## L07 — Uplift Modelling (T-learner)

> **On your own data.** This section is the real-world version of Homework 7's agent
> prompt. The homework prompt is a *grading* prompt: it pins the forest size, the seed,
> the exact split call and the scenario's cost and order value so every answer can be
> checked. Use the prompt below for your own experiment, and read "What not to copy from
> the homework" first.

### 1. The question it answers
In a randomised campaign, which customers does the treatment actually change? Who should
you target so that the extra conversions are worth the cost, and whom should you leave alone?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 7. In practice: one row per customer with a 0/1 treatment flag, a 0/1 outcome, and
features measured **before** treatment.

Stop, or change plan, if any of these is true:
- **Treatment was not randomised** (for example, the team picked who got the coupon). The
  two models then compare different kinds of customer, and τ̂ is biased.
- **Features measured after treatment** (anything the treatment could have changed).
- **Thin corners**: feature combinations where almost everyone was treated, or almost no
  one. τ̂ there is mostly noise.
- **An experiment sized only for the average effect.** It is usually too small to rank
  customers reliably, so expect a small held-out Qini (the L07 note says the homework's held-out ranking is close to random).

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| The two outcome models | `scikit-learn`: `RandomForestClassifier`, with `predict_proba(X)[:, 1]` | Yes (1.4.2) | Yes: 1.9.1 released 2026-09-10 |
| Train/test split | `sklearn.model_selection.train_test_split(..., stratify=...)` | Yes | Yes |
| Qini coefficient | `scikit-uplift`: `sklift.metrics.qini_auc_score`, `sklift.viz.plot_qini_curve` | Yes (0.5.1) | **Dormant**: last release August 2022, no commits since October 2023 |
| Maintained alternative | `causalml`: `causalml.metrics.qini_score`, `plot_qini` (also ready-made T-, S- and X-learners) | No (installs on Python 3.11) | Yes: 0.18.0 released 2026-10-12 |

`scikit-uplift` still works in the Codespace, but do not start a new project on it.
Qini has several incompatible definitions, so **never compare a Qini from one package
with a Qini from another**.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I ran a randomised experiment. <file.csv> has <customer_id>, <treated> (0/1),
<outcome> (0/1), and pre-treatment features <list>. Treatment = <what it was>. It costs
<c> per treated customer, and a conversion is worth <v>.

1. Balance: compare treated vs control means on every feature. Report the observed
   ATE = treated outcome rate - control outcome rate, on all the data.
2. Split 80/20 with train_test_split, stratified on treated x outcome, fixed seed.
3. T-learner: fit mu1 (RandomForestClassifier) on treated training rows only and mu0 on
   control training rows only. Use min_samples_leaf=<e.g. 50> so the forests do not
   memorise the training rows. tau_hat = mu1(X) - mu0(X).
4. On the TEST set: mean tau_hat (compare with step 1's ATE), the Qini curve, and the
   Qini coefficient with <sklift.metrics.qini_auc_score | causalml qini_score>. Also
   report the training-set Qini, labelled as in-sample.
5. Targeting: the share of test customers with tau_hat > c / v, and with tau_hat < 0.
6. For persuadability, compare mean tau_hat across values of the main features.
   Do not read mu1's feature importances as drivers of uplift.

Show the code, then one results table per step. Do not invent any number.
```

### 5. What you get back
- The observed ATE and a balance table.
- A τ̂ for every customer, and its average on the test set.
- A held-out Qini curve and coefficient (and the in-sample one for comparison).
- The share worth targeting at your cost and value, and the share the treatment may harm.

What it does **not** give you: a profitable campaign. A good Qini says the ranking works,
not that targeting pays. That is the c / v threshold's job.

### 6. Checks before trusting it
The first four combine the L07 note's two "What to verify" lists.
1. **Mean τ̂ agrees with the observed ATE in sign and order of magnitude.** It does not
   need to match to a fixed tolerance. How close it gets depends on the forests' regularisation.
2. **The test-set Qini is positive.** A negative Qini means the ranking is inverted.
3. **Report the test-set Qini, not the in-sample one.** A fully grown forest's in-sample
   Qini can be about 20 times larger (L07 Section 1.4C). A test-set Qini near 0 is an
   honest finding: the ranking does not carry over to new customers.
4. **No large imbalance between treated and control** on any feature. If there is one,
   suspect the randomisation.
5. **Feature importances show what predicts conversion, not who is persuadable.**
6. **Customers with τ̂ < 0** are a group the treatment may harm. Do not treat them as
   proof that the model works or that it fails.

### What not to copy from the homework
- **`n_estimators=100, random_state=42`, fully grown trees.** These are reproducibility
  pins. On your data, regularise (for example `min_samples_leaf`) and compare a few settings on held-out data.
- **`train_test_split(..., random_state=42)` with no `stratify`.** The homework pins the
  exact call because the test-set Qini is sensitive to which rows land in the test set.
  On your data, stratify, and check the Qini across a few different splits.
- **The cost and order value** (a fixed coupon and order value for everyone). Use your own.

**Do keep** training each model on its own arm only, and scoring the Qini on held-out customers.

---

## L08 — Customer Lifetime Value (BG/NBD + Gamma-Gamma)

> **On your own data.** This section is the real-world version of Homework 8's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the penalizers,
> the horizon and the discount rate so every answer can be checked. Use the prompt below
> for your own data, and read "What not to copy from the homework" first.

### 1. The question it answers
For each customer: how likely are they to still be a customer (P(alive)), how many more
purchases will they make, how much will each be worth, and what is that worth over the
next 12 months?

### 2. Data you need, and deal-breakers
See the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 8. In practice: a transaction log with one row per order: customer id, order date,
order value.

Stop, or change plan, if any of these is true:
- **It is a subscription (contractual) business**, where you see the moment a customer
  leaves. BG/NBD is for settings where churn is never observed. Use a contractual model
  instead, such as the survival analysis of L04, or a shifted-beta-geometric model
  (`ShiftedBetaGeoModel` in PyMC-Marketing).
- **Order value moves with purchase frequency** (frequent buyers spend more or less per
  order). Gamma-Gamma assumes they are independent. Measure it first (check 4).
- **Very short history.** With few repeat purchases, the parameters are poorly pinned down.
- **Many one-time buyers.** BG/NBD gives every customer with no repeat purchase
  P(alive) = 1 by construction, not by evidence (L08 Section 1.4C).

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| RFM summary | `lifetimes.utils.summary_data_from_transaction_data(df, "customer_id", "order_date", monetary_value_col="order_value", freq="D")` | Yes (0.11.3) | **No: archived**. Last release July 2020, repo archived June 2024 |
| BG/NBD, P(alive) | `lifetimes.BetaGeoFitter`, `.conditional_probability_alive(...)` | Yes | No (archived) |
| Gamma-Gamma, CLV | `lifetimes.GammaGammaFitter`, `.customer_lifetime_value(bgf, ..., time=12, discount_rate=...)` | Yes | No (archived) |
| The successor | `pymc-marketing`: `pymc_marketing.clv.BetaGeoModel`, `GammaGammaModel` (Bayesian, with uncertainty) | No: needs Python 3.12 or newer | Yes: 1.3.0 released 2026-10-22 |

`lifetimes`' own README names PyMC-Marketing as its successor. `lifetimes` still runs in
the Codespace, but do not start a new project on it.

Two things to know about `lifetimes`, both confirmed on the course env:
- **Gamma-Gamma refuses customers with no repeat purchase** (`ValueError` on a
  `monetary_value` of 0). Fit it on `frequency > 0` only.
- **`recency` (t_x) is the time from the first to the most recent purchase**, not the time
  since the last purchase. The silent gap is `T - recency`.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have a transaction log in <file.csv>: <customer_id>, <order_date>, <order_value>
(in <currency>), from <start> to <end>. This is a non-contractual business: customers
can stop buying without telling us.

1. RFM summary with lifetimes.utils.summary_data_from_transaction_data, calibration
   end = <date>, freq = "<D | W>", include_first_transaction=False. Report how many
   customers have frequency = 0.
2. Calibration check: refit on data before <holdout start>, predict purchases in the
   holdout, and compare predicted vs actual by frequency group.
3. BG/NBD (BetaGeoFitter, penalizer_coef=<small, e.g. 0.001>) on all data. Report r,
   alpha, a, b, and P(alive) for every customer.
4. Independence check, REPEAT buyers only (frequency > 0): correlation between frequency
   and monetary_value, and a scatter plot.
5. Gamma-Gamma (GammaGammaFitter, penalizer_coef=0) on frequency > 0 only.
6. <H>-month CLV with discount_rate=<monthly rate>. Do NOT multiply by P(alive) again:
   it is already inside the expected purchases. Report median, mean and 90th percentile
   CLV, and the top 10 customers.

Show the code, then one results table per step. Do not invent any number.
```

### 5. What you get back
- r, α, a and b, and a P(alive) for every customer.
- A holdout check of predicted vs actual purchases.
- The frequency–spend correlation among repeat buyers.
- A CLV per customer over your horizon, with the distribution and the top customers.

What it does **not** give you: who will respond to a retention offer. Low P(alive) or high
CLV is a reason to look, not a reason to spend. Who responds is an uplift question (L07).

### 6. Checks before trusting it
The first four are the L08 note's "What to verify before trusting the output".
1. **Parameters are in plausible ranges**: r in [0.1, 3], α in [1, 30], a in [0.1, 2],
   b in [1, 10]. α is a rate parameter, so it scales with the time unit you chose in step 1.
2. **Recent buyers have high P(alive)**, typically above 0.8 (a recent low-frequency
   buyer can sit lower), and long-silent buyers are below 0.2.
3. **Mean CLV is a plausible multiple of the average order value.**
4. **|correlation(spend, frequency)| < 0.2** among repeat buyers. Between 0.2 and 0.3 is a
   grey zone, and 0.3 or more is a material violation (L08 Section 1.4C).
5. **P(alive) was not applied twice.** CLV × P(alive) double-counts churn.
6. **The holdout check** (prompt step 2): predicted purchases should track actual ones
   for each frequency group, not just on average.

### What not to copy from the homework
- **`penalizer_coef=0.01` for BG/NBD.** Some ridge is common, but the value is a pin.
  Use the smallest that gives a stable fit, and report it.
- **`discount_rate=0.01` per month and the 12-month horizon.** Use your own cost of
  capital and the horizon your decision needs. Match it to the CAC horizon if you compare CLV with CAC.
- **"Use max(order_date) as calibration end" with no holdout.** On your data, hold out a
  period and check the predictions (prompt step 2).
- **`seed=42`.** The fits are deterministic maximum likelihood, so the seed does nothing.

**Do keep** `include_first_transaction=False` (BG/NBD is defined on repeat purchases), an
unpenalized Gamma-Gamma fit on repeat buyers only, and the independence check on repeat buyers only.

---

## L09 — Conjoint Analysis (MNL)

> **On your own data.** This section is the real-world version of Homework 9's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the products
> and the reference levels so every student gets the same checkable number. Use the
> prompt below for your own survey, and read "What not to copy from the homework" first.

### 1. The question it answers
Which product features do customers trade off against price, how many dollars is each
feature worth to them (willingness to pay, WTP), and what share would each product in a
set of competing products win?

### 2. Data you need, and deal-breakers
**No data yet?** Run the choice experiment first. The L09 note, Section 1.3, gives the five
design steps, and the design prompt in part 4 builds the survey's choice tasks with an agent.

What the model needs and assumes is in the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 9. In practice your file must be in **long format**: one row per alternative shown,
with a respondent id, a choice-task id, a 0/1 `chosen` column, price as a number, and one
column per attribute.

Stop before fitting if any of these is true:
- **Not exactly one `chosen = 1` per task.** Multi-select answers, or a dropped "none of
  these" option, break the model's choice-set structure.
- **Price does not vary within tasks**, or moves in lockstep with another attribute
  (premium is always the expensive one). Then β_price is not identified separately and
  every WTP is meaningless.
- **The data are real purchases at prices the firm set**, not survey choices. That is
  revealed-preference data, and prices set in response to demand bring back the
  endogeneity problem from L02.
- **You want WTP or shares outside the tested range** (prices or levels the survey never
  showed). That is extrapolation the data cannot support.

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Fit the MNL | `statsmodels`: `statsmodels.discrete.conditional_models.ConditionalLogit`, with `groups=` one id per (respondent, task) | Yes (0.14.2) | Yes: 0.15.0 released 2026-08-27 |
| Build the design matrix | `patsy.dmatrix(...)`, then drop the `Intercept` column | Yes | Yes (ships with statsmodels) |
| Design the choice tasks | No package: random draws with `numpy`, saved with `pandas` | Yes | Yes |
| WTP, market simulation, profit-maximizing price | No package: arithmetic on the fitted coefficients (softmax, then a price search) | n/a | n/a |
| Different preferences per customer (mixed logit, not covered in this course) | `biogeme` | No: needs Python 3.12 or newer | Yes: 3.3.5 released 2026-08-28 |

Two things an agent may reach for that you should not use: `pylogit` (last release
December 2020) and `xlogit` (last release July 2023, no commits since January 2024).

Two errors you will hit with `ConditionalLogit`, both confirmed on the course env:
- A formula with an intercept is **refused** ("Conditional models should not have an
  intercept"). Each choice task is its own comparison, so a constant cancels out.
- Writing `0 +` in the formula does **not** fix it: patsy then codes the first categorical
  attribute with *all* its levels, and the fit fails because the Hessian cannot be inverted.
  Build the matrix with the intercept and drop that column instead.

### 4. A general prompt
Fill in every `<placeholder>`. Paste each prompt as one message.

**Design prompt (before you field the survey).** Survey platforms such as Qualtrics,
Sawtooth Software and Conjointly can generate the design for you. Use this prompt when you
build the tasks yourself, or to check the platform's design.

```
I am designing a choice-based conjoint survey. Attributes and levels:
  price: <level_1, level_2, ...> (numeric, in <currency> per <unit>)
  <attribute_1>: <level_a, level_b, ...>
  <attribute_2>: <...>
Each respondent sees <K> tasks with <J> profiles each. I plan <N> respondents.
<Include / Do not include> a "none of these" option in every task.

1. Generate a random design with numpy and pandas, no design package: for every profile,
   draw each attribute's level independently and with equal probability. Use a fixed
   random seed and save the design file; the responses must be matched back to it.
2. Redraw any task in which two profiles are identical. Report how many you redrew.
3. Check balance: how often each level appears, and the correlation between price and
   every other attribute (the counts should be close to equal and the correlations
   close to zero).
4. Save in long format: respondent_id, task_id, alternative, one column per attribute,
   and an empty chosen column.

Show the code and the balance table. Do not invent any number.
```

**Analysis prompt (after the responses are in).**

```
I have stated-preference (conjoint) choice data in <file.csv>, in long format: one row
per alternative shown. Columns: <respondent_id>, <task_id>, <chosen> (0/1), <price>
(numeric, in <currency> per <unit, e.g. month>), and attributes <attribute_1>,
<attribute_2>, ... Each task showed <J> alternatives. Each respondent answered <K> tasks.

1. Check the data BEFORE fitting, and stop and tell me if any check fails:
   - every (respondent, task) has exactly one row with chosen = 1;
   - price varies within tasks;
   - list every attribute's levels and how often each level appears.
2. Fit a multinomial logit with statsmodels ConditionalLogit, with groups = one id per
   (respondent, task). Do NOT use a pooled binary Logit. Build the design matrix with
   patsy and drop the Intercept column. Price enters as a number, not a category.
   Reference levels: <attribute_1: level>, <attribute_2: level>, ...
   <If the survey had a "none" option: code each "none" row with price 0 and every
   attribute at its reference level, add a 0/1 column none (1 only on those rows),
   and include none as a regressor.>
3. Report every coefficient with its standard error and p-value.
4. WTP for each non-price level = beta_level / |beta_price|, in <currency> per <unit>.
5. Market simulation: compute softmax shares among exactly these products:
   <product 1: levels, price>
   <product 2: levels, price>
   <...>
   Then repeat with <one change, e.g. product 2's price + 2>.
6. Attribute importance: for each attribute, the range of its part-worths across the
   levels tested (for price: |beta_price| x (highest tested price - lowest)).
7. <Optional, if you know unit costs> Profit-maximizing price for <product>, unit cost
   <c>, all other products' prices held fixed: search prices from <low> to <high>
   (inside the tested range) in steps of <0.10>, and report the price with the highest
   (price - c) x share, its share and its profit per customer. Check the result against
   price - c = 1 / (|beta_price| x (1 - share)).

Show the code, then one results table per step. Do not invent any number. If a step
fails, say so instead of working around it.
```

### 5. What you get back
- A coefficient table: one row for price and one per non-reference level, each with a
  standard error and p-value. Every coefficient is relative to its attribute's reference
  level.
- A WTP for each non-price level, in your currency per unit.
- A share table for your products that sums to 100%, plus the same table after your change.
- An importance ranking of the attributes.
- If you ran step 7: the profit-maximizing price for your product, with its share and
  profit per customer. It is finite only because the other products keep their prices
  (L09 note, Section 1.4D). To price a whole product line, the survey needs a "none" option.

What it does **not** give you: each customer's preferences (the shares assume everyone
has the average part-worths) or predicted sales volume (shares are among the listed
products only). WTP alone sets no price: WTP is an average (see "WTP is Not a Price
Recommendation" in the L09 slides), and the price comes from step 7.

### 6. Checks before trusting it
The first five are the L09 note's "What to verify before trusting the output".
1. β_price is **negative**. If it is positive, check how price was coded before anything else.
2. β_price is **statistically significant** (p < 0.05).
3. Levels customers should want (e.g. no ads, premium content) have **positive**
   coefficients against their reference. A sign is only meaningful relative to its reference level.
4. WTP is **plausible** next to the prices you tested. WTP holds only inside that range.
5. Market shares **sum to 100%**.
6. **Recompute one WTP and one share by hand** from the coefficient table
   (L09 Section 1.1 Tool 2, and the softmax in Section 1.4B). If yours and the agent's
   disagree, the agent is wrong until shown otherwise.
7. **Close substitutes?** If two products in your simulation are near-copies, the model
   spreads the new one's share across all products in proportion (IIA, Section 1.4C), so
   it understates cannibalization of the near-twin. Say so in your write-up.
8. **The standard errors treat every task as independent**, although each respondent
   answered several. `ConditionalLogit` silently ignores `cov_type="cluster"`. When
   respondents genuinely differ, these SEs can be too small, so do not rest a decision on
   a borderline p-value. Bootstrap by respondent if it matters. (On the course dataset
   the respondent bootstrap matches the reported SEs, because its simulated respondents
   share one set of preferences.)
9. **Hypothetical choices.** People overstate what they would pay when no money changes
   hands. Treat WTP as an upper bound and check it against real purchases where you can.
   The same bias raises the profit-maximizing price from step 7.
10. **If you designed the survey yourself:** the design prompt's balance table shows every
   level appearing about equally often and price uncorrelated with the other attributes.

### What not to copy from the homework
The HW09 prompt pins these so the autograder can check one number. **Do not reuse them**
for your own data:
- **`Use seed=42`.** The fit is deterministic maximum likelihood, so the seed changes
  nothing. Its presence does not mean your analysis is reproducible.
- **Products A, B, C and their prices.** They are the graded scenario. Simulate your own
  products and your competitors' actual offers.
- **The column names, levels and reference levels** (standard, full_ads, monthly). Pick
  the reference that matches your decision, usually your current offer. Coefficients
  change with the reference, but WTP *differences* between levels do not.
- **"Rank by |coefficient|."** It compares price's per-dollar slope with whole-level
  jumps, so the ranking depends on the price unit (dollars or cents). Use the range-based
  importance in step 6 of the prompt.

**Do keep** `ConditionalLogit` grouped by (respondent, task). That is the method, not a
grading pin: a pooled binary Logit on the same rows ignores the choice set and shifts
every coefficient.

---

## L10 — Demand Forecasting (Prophet)

> **On your own data.** This section is the real-world version of Homework 10's agent
> prompt. The homework prompt is a *grading* prompt: it fixes the dataset, the split, the
> interval level and the seed so every student gets the same checkable number. Use the
> prompt below for your own series, and read "What not to copy from the homework" first.

### 1. The question it answers
How much demand should you plan for over the next weeks or months, and how uncertain is
that number? Also: how much of the history is trend, how much is the yearly season, and
how much is holidays?

### 2. Data you need, and deal-breakers
What the model needs and assumes is in the
[Model Reference Card](https://nu-mktg.github.io/marketing-analytics/md-viewer.html?file=student/model_reference_card.md),
row 10. In practice: two columns, `ds` (a date) and `y` (the number to forecast), one row
per day or per week, with the same frequency throughout.

Stop, or change plan, if any of these is true:
- **Less than two full years** of history. One year cannot separate the yearly season from
  the trend.
- **A known break you cannot explain to the model**: a competitor's entry, a pricing
  change, a lost channel, a campaign that has ended. Tell the model the dates, or the trend
  absorbs the break and bends around it (L10 Section 1.4B): a permanent level shift goes in as
  a 0/1 step regressor (`add_regressor`), and a campaign that ended goes in as a holiday
  spanning its dates. A changepoint changes only the slope, so it does not fix a level jump.
- **The break is recent.** Prophet places no changepoints in the last 20% of the history
  (`changepoint_range=0.8`), so a shift in that stretch is not picked up by default.
- **You want to forecast the effect of your own planned action** (a price cut, a new
  campaign). Prophet extrapolates the past. It does not know about actions that never
  happened in the history.
- **`y` has zeros or near-zeros** (intermittent demand). MAPE is undefined there, and
  Prophet's `performance_metrics` **drops the MAPE column** when any `y` is close to 0,
  with only an info-level log line. Use MAE, or a method built for intermittent demand.

### 3. Tools
Checked against PyPI and GitHub on **2026-11-04**.

| Step | Package and function | In the course Codespace? | Maintained? |
|---|---|---|---|
| Fit and forecast | `prophet`: `Prophet(...)`, `.fit(df)`, `.make_future_dataframe(periods=, freq=)`, `.predict(future)` | Yes (1.1.5) | Yes: 1.5.0 released 2026-10-05, commits through 2026-10-22 |
| Components and changepoints | `m.plot_components(forecast)`, `prophet.plot.add_changepoints_to_plot(...)` | Yes | Yes |
| Holdout accuracy | `prophet.diagnostics.cross_validation(m, initial=, period=, horizon=)`, then `performance_metrics(...)` | Yes | Yes |
| Simple baseline to beat | `statsmodels`: `ExponentialSmoothing`, or a seasonal-naive forecast (same week last year, no package) | Yes | Yes (statsmodels 0.15.0, 2026-08-27) |
| More forecasting models (ETS, ARIMA, many series at once) | `statsforecast` (Nixtla) | No | Yes: 2.1.1 released 2026-07-16 |

`neuralprophet` exists, but its last release was 0.9.0 in June 2024 and it supports Python
3.12 at most. Do not start a project on it.

Three things to know about `prophet`, all confirmed on the course env:
- **On weekly data, a holiday only counts if it lands exactly on a `ds` date.**
  `add_country_holidays(country_name="US")` on the course's Monday-stamped weeks gives
  Thanksgiving, New Year's Day and Veterans Day **zero** rows in four years. Only the
  Monday holidays register. Fix: build the holidays table yourself and give each holiday a
  window that covers its week (for weeks stamped on the Monday: `lower_window=-6`,
  `upper_window=0`). With that, every holiday registers in every year.
- **`m.changepoints` lists all 25 *potential* changepoints**, not the ones the fit kept.
  An agent that prints it will report 25 "detected" changepoints. Look at the slope changes
  (`m.params["delta"]`) or at `add_changepoints_to_plot`, which shows only the large ones.
- **The intervals are simulated**, from 1,000 random draws by default. A different seed
  moves the interval width slightly (a few percent on the course data). It does not move
  the forecast line.

### 4. A general prompt
Fill in every `<placeholder>`. Paste it as one message.

```
I have a <weekly | daily> time series in <file.csv> with columns ds (date) and y
(<what y is, in what unit>), from <start date> to <end date>. I need a forecast
<H> <weeks | days> ahead to decide <the decision, e.g. how much stock to order>.
Known events in the history: <date: what happened>, ... (or "none").

1. Check the data before fitting, and stop and tell me if any check fails: one row per
   <week | day> with no duplicate dates, the number of missing periods, the number of
   y values at or near zero, and how many full years of history there are.
2. Fit Prophet with yearly_seasonality=True,
   weekly_seasonality=<False for weekly data, True for daily data>,
   interval_width=<level that matches the decision, e.g. 0.90>.
   Holidays: <country, or "none">. If the data are weekly, build the holidays table with a
   window so each holiday falls inside its week (do not rely on add_country_holidays to
   match dates). Add a changepoint at each known event date listed above.
3. Accuracy: run prophet.diagnostics.cross_validation with horizon = <H> <weeks | days>,
   several cutoffs, and at least two years before the first cutoff. Report MAPE (or MAE
   if y has zeros) and interval coverage, by horizon.
4. Fit a seasonal-naive baseline (same period last year) on the same cutoffs and report
   its MAPE next to Prophet's.
5. Refit on all the data. Forecast <H> <weeks | days> ahead. Plot history, forecast and
   interval. Plot the components.
6. List the changepoints the fit actually kept (large slope change), with dates and
   direction. Do not list all the potential ones.

Set numpy's seed so the intervals can be reproduced. Show the code, then one results
table per step. Do not invent any number. If a step fails, say so instead of working
around it.
```

### 5. What you get back
- A forecast table: `yhat` with its lower and upper bounds for each future period.
- A components plot: trend, yearly season and holidays, with the size of each.
- Accuracy by horizon from the holdout cutoffs: MAPE (or MAE) and the share of actuals that
  fell inside the interval (coverage), next to a simple baseline's MAPE.
- The changepoints the fit kept, with dates.

What it does **not** give you: the effect of something that has never happened in the
history, a guaranteed coverage (the interval covers only the uncertainty the model knows
about, L10 Common Misconception 3), or a reason *why* the trend changed.

### 6. Checks before trusting it
The first five are the L10 note's "What to verify before trusting the output".
1. The **trend direction** matches what you know about the business.
2. The **yearly season** matches *this* series' own history and this business's calendar.
   Do not check it against retail's Q4 peak unless this is a retailer.
3. **Holiday effects** have sensible signs. On weekly data, first confirm the holidays
   matched any rows at all (see the gotcha in Tools).
4. **Holdout MAPE** is acceptable for the decision, at the horizon you actually need
   (L10 Section 1.4C).
5. The **changepoints the fit kept** sit near known business events, not in stable stretches.
6. **It beats a simple baseline.** If seasonal-naive is as accurate, Prophet adds nothing
   for this decision (L10 Common Misconception 1).
7. **Coverage is near the interval level you asked for.** If a 90% interval covers only 70%
   of the holdout actuals, treat the bounds as too narrow before using them for safety stock.
8. **Read the width, don't judge it.** The interval has a noise part (the same at every
   horizon) and a trend part (grows with the horizon), so on a steady series it can stay almost
   constant for months, and that is fine. Coverage (check 7) is the test, not the shape
   (L10 Section 1.4A).
9. **Training fit is not accuracy.** Ignore any in-sample MAPE the agent reports
   (L10 Common Misconception 5).

### What not to copy from the homework
The HW10 prompt pins these so the autograder can check one number. **Do not carry them
over** to your own data:
- **Train on 182 weeks, test on the last 26.** One split, sized for the course dataset.
  Use your own decision's horizon and several cutoffs (step 3 of the prompt).
- **`interval_width=0.95`.** The homework pins it because its questions ask about 95%
  intervals. Pick the level your decision needs.
- **"US holidays."** The homework adds every US holiday. Only add holidays that move *your*
  series (L10 Common Misconception 4). The week-long window the homework uses is method, not a
  pin: keep it for any weekly data.
- **"Top 3 weeks with the widest interval" and "peak seasonal week".** These are graded
  outputs, not the questions your planning decision asks.

**Do keep** `weekly_seasonality=False` if your data are weekly (a weekly series has no
within-week pattern to fit), and a fixed seed, so you can reproduce the interval you report.

---

## L12 — Difference-in-Differences

### Context Prompt
```
I am a marketing analyst measuring the causal effect of a price reduction
experiment. The firm reduced prices by 15% in four geographic regions (treatment)
while maintaining prices in four control regions. I have weekly revenue data for
all regions across 20 weeks (weeks 1–10 pre-intervention, weeks 11–20 post).
I want to estimate the causal revenue impact using DiD and validate the parallel
trends assumption.
```

### Build Prompt
```
Using the geo_experiment dataset:

1. Compute the 2×2 DiD estimate manually from the group-period means:
   DiD = (mean_treat_post - mean_treat_pre) - (mean_ctrl_post - mean_ctrl_pre)
   Also report the naive estimate (treat post - treat pre) and the common trend
   (ctrl post - ctrl pre). Show that DiD = naive - common trend.

2. Fit the DiD regression:
   revenue = β₀ + β₁·treat + β₂·post + β₃·(treat×post) + ε
   Report all four coefficients, standard errors, p-values, and 95% CI for β₃.
   β₃ is the causal treatment effect.

3. Plot: mean weekly revenue for treatment and control groups (weeks 1–20)
   with a vertical line at week 10.5 (intervention start).
   Do the pre-period trends look parallel?

4. Run a placebo test: use pre-period data only (weeks 1–10).
   Define fake_post = (week ≥ 5). Run the DiD regression on pre-period only.
   Report β₃_placebo and its p-value. A non-significant placebo supports
   parallel trends.

Print all values clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. TREATMENT EFFECT COEFFICIENT: Confirm β₃ equals the manually computed DiD estimate.
   They should be identical (the regression is just a computational form of the
   2×2 table). Report: PASS or FAIL.

2. PARALLEL TRENDS: Does the pre-period plot show treatment and control groups
   moving together? If the treatment group was already growing faster before week 11,
   the DiD estimate is biased upward. Report whether the plot looks parallel.

3. PLACEBO P-VALUE: Is the placebo β₃ statistically insignificant (p > 0.05)?
   A significant placebo means there was already a differential trend in the
   pre-period — the parallel trends assumption is likely violated.
   Report: the placebo p-value and whether it supports parallel trends.
```

---

## L13 — Customer Segmentation (k-means)

### Context Prompt
```
I am a customer analytics analyst. I have customer-level feature data including
recency (days since last purchase), frequency (purchases in last 12 months),
average order value, support ticket count, and email open rate. I want to
segment customers into meaningful groups for targeted marketing using k-means
clustering, then profile each segment using downstream model outputs (CLV and
predicted uplift).
```

### Build Prompt
```
Using the customer_features dataset:

1. Standardise the five clustering features (recency_days, frequency,
   avg_order_value, support_tickets, email_open_rate) to zero mean and unit
   variance. Do NOT standardise clv_12m or tau_hat — these are profiling variables.

2. Fit k-means for k = 2 through 8 (random_state=42).
   Plot: WCSS vs k (elbow plot) and mean silhouette score vs k.
   State your recommended k and the reasoning.

3. Fit k-means with the recommended k. For each cluster, report mean values of:
   recency_days, frequency, avg_order_value, clv_12m, tau_hat.
   Assign a descriptive label to each cluster.

4. Report: what fraction of customers in each cluster have tau_hat > 0.133
   (above the retention campaign targeting threshold)?

Print all results clearly labelled. Use random_state=42.
```

### Validation Prompt
```
Three validation checks:

1. STANDARDISATION VERIFICATION: Confirm each of the five clustering features
   has mean ≈ 0 and standard deviation ≈ 1 after standardisation.
   If any feature has std >> 1, standardisation failed.
   Report: means and stds for all five features post-standardisation.

2. SILHOUETTE INTERPRETATION: The silhouette score is the right criterion for
   comparing clustering solutions of different k (WCSS always decreases with k
   and is not a valid comparison). Confirm your k selection was based on the
   silhouette maximum, not the WCSS elbow. Report: which k has the highest
   silhouette, and what that silhouette value is.

3. TARGETING LOGIC: For any cluster with mean tau_hat < 0.133, confirm your
   output does NOT recommend targeting that cluster with the retention campaign,
   regardless of its mean CLV. High CLV does not override a below-threshold
   predicted uplift. Flag if this error appears in the output.
```

---

## L14 — CausalImpact

### Context Prompt
```
I am a marketing analyst measuring the causal effect of a brand awareness campaign
on website sessions. The campaign ran nationally (no untreated geographic regions),
so I cannot use DiD. I have weekly data on website sessions (outcome) and correlated
control series that were not affected by the campaign: branded search impressions
and social mentions. The campaign started at week 21. Pre-period: weeks 1–20.
Post-period: weeks 21–30.
```

### Build Prompt
```
Using the campaign_timeseries dataset:

1. Install and import the causalimpact library.
   Fit CausalImpact with:
   - Outcome: website_sessions
   - Control covariates: branded_search_impressions, social_mentions
   - Pre-period index: rows 0–19 (weeks 1–20)
   - Post-period index: rows 20–29 (weeks 21–30)

2. Plot all three CausalImpact panels: original (with counterfactual),
   pointwise effect, and cumulative effect.

3. Print the summary table showing:
   - Average actual vs predicted sessions in the post-period
   - Cumulative actual vs predicted
   - Average and cumulative causal effects with 95% posterior CIs
   - Posterior probability that the effect is positive

4. Compute pre-period MAPE:
   mean(|actual - fitted| / actual) × 100 over weeks 1–20.
   A MAPE < 10% indicates good pre-period fit.

Print all results clearly labelled.
```

### Validation Prompt
```
Three validation checks:

1. PRE-PERIOD FIT: Is the pre-period MAPE below 10%? If above 20%, the control
   covariates do not explain the outcome well — the post-period counterfactual
   will be unreliable. Report: the pre-period MAPE and whether it is acceptable.

2. CI INTERPRETATION: Does the 95% credible interval for the cumulative effect
   exclude zero? If zero is inside the CI, you cannot claim credible evidence of
   a positive causal effect, even if the point estimate is large and positive.
   Report: the CI bounds and whether zero is excluded.

3. COVARIATE CONTAMINATION CHECK: Could either control covariate (branded search
   impressions, social mentions) have INCREASED because of the campaign itself?
   If the campaign drove branded search or social conversation, the covariates are
   contaminated — the model will underestimate the true campaign effect.
   Flag this as a limitation if it is plausible.
```
