# Chapter 10: Demand Forecasting with Prophet
## Comprehensive Reference Guide

---

### Chapter Overview

Prophet (Taylor and Letham, 2018) decomposes a time series into trend, seasonality,
and holiday effects. This chapter derives the full model specification — including
the Fourier series representation of seasonality, the piecewise linear trend with
changepoints, and the Laplace prior that enforces sparsity — and develops the full
evaluation framework including cross-validation and uncertainty quantification.

**By the end of this chapter you should be able to:**
- Define a Fourier series and explain why it can represent any smooth periodic function
- Interpret each component of the Prophet decomposition (trend, seasonality, holidays)
- Explain what a changepoint is and how the Laplace prior selects them
- Compute MAPE by hand and assess whether a given MAPE is actionable
- Explain how prediction intervals are generated and why they widen with horizon
- Identify when the model needs a manual changepoint and how to add one

---

## 10.1 Strategic Frame: What Practitioners Know

### Jess Lachs: Analytics as Impact, Not Service

Jessica Lachs built the data science and analytics organization at DoorDash into one
of the largest and most respected in tech. She articulates the purpose of demand
forecasting more precisely than any technical framework:

"Analytics is a business impact driving function and not purely a service function.
Not just answering the why, but answering the 'What do we do now that we know this?'"

A demand forecast is the most direct operationalization of this principle. It does
not tell you what happened (retrospective analytics) or why (diagnostic analytics).
It tells you what will happen if nothing changes — which creates the space to ask
whether something should change, and if so, how soon and by how much.

Her specific insight on retention leading indicators: "Retention is a terrible thing
to goal on. It's almost impossible to drive in a meaningful way in the short term.
Ultimately, you want to find a short-term metric you can measure that drives a
long-term output." Demand forecasting is exactly this framework applied to revenue:
by decomposing future revenue into trend, seasonal, and holiday components, you
identify which short-term observable patterns drive the long-term revenue trajectory.

A supply chain manager with a Prophet model showing a 40% Q4 spike can act 8 weeks
before the holiday season. Without the model, they act when stockouts appear — 2 weeks
in, after the opportunity has passed.

### Sri Batchu: Channel Efficiency and Demand-Driven Allocation

Sri Batchu led growth at Instacart and Ramp. His channel allocation principle:
"Make all channels more efficient, but also allocate more to channels that are more
efficient as long as they're scalable."

Demand forecasting enables this precision. The decomposition separates:
- **Organic trend** (weeks when demand is genuinely growing due to product and brand)
  from
- **Seasonal amplitude** (weeks when demand appears high because it always does)

Marketing spend is most productive when amplifying organic demand. Spending heavily
on paid acquisition during a seasonal peak means you are paying to reach customers
who would have come anyway. Spending during genuine organic demand growth means your
paid spend is building on compounding organic momentum.

A Prophet decomposition that correctly separates trend from seasonality makes this
allocation decision quantitative.

### Three Strategic Insights

**Insight 1: The decomposition output is more valuable than the point forecast.**

The point forecast answers "what will revenue be in week 26?" The decomposition
answers: "what fraction of that revenue is organic trend growth, what fraction is
seasonal, and what fraction is attributable to holiday effects?" These are different
decisions with different actions:
- Trend component declining: urgent product/retention intervention needed
- Seasonal component as expected: proceed with standard playbook
- Holiday effect underperforming vs. prior year: promotional strategy review needed

**Insight 2: Uncertainty intervals should drive decisions, not be ignored.**

When a forecast shows a 95% prediction interval of [$800k, $1,200k] at week 26,
a manager who ignores the interval and plans to the point forecast of $1,000k
is taking an unnecessary risk. The economically correct approach: identify the
cost asymmetry (cost of having too little inventory vs. too much) and plan to
the appropriate percentile of the interval.

If underage cost > overage cost: plan to the 70th or 80th percentile (high side).
If overage cost > underage cost: plan to the 30th or 40th percentile (low side).
If costs are symmetric: plan to the median (50th percentile, approximately the
point forecast).

**Insight 3: Manual changepoints are not "cheating" — they are required.**

A model trained on data that includes a major promotional campaign will try to
extrapolate that campaign's growth as part of the long-run trend. If the campaign
has ended, the model is wrong. Adding a changepoint at the campaign's end is not
interfering with the model — it is encoding knowledge the model cannot infer from
the data pattern alone. Models serve human judgment; they do not replace it.

---

## 10.2 Mathematical Prerequisites

### 10.2.1 Fourier Series

A **Fourier series** represents a periodic function as a sum of sinusoidal components.
For a function f(t) with period P:

$$f(t) = \frac{a_0}{2} + \sum_{n=1}^{\infty} \left[a_n \cos\left(\frac{2\pi n t}{P}\right) + b_n \sin\left(\frac{2\pi n t}{P}\right)\right]$$

**Fourier convergence theorem:** If f(t) is periodic with period P, absolutely
integrable over one period, and has at most finitely many discontinuities, the
Fourier series converges to f(t) at every point of continuity and to the average of
left and right limits at discontinuities.

**Practical implication:** Any smooth seasonal pattern can be approximated
arbitrarily well by a finite Fourier series. With N terms (pairs of sine and cosine),
you can capture N distinct "frequencies" in the seasonal pattern.

*Reference: Wikipedia, "Fourier series" — en.wikipedia.org/wiki/Fourier_series*
*Reference: Wikipedia, "Fourier analysis" — en.wikipedia.org/wiki/Fourier_analysis*
*Reference: Wikipedia, "Convergence of Fourier series" —
en.wikipedia.org/wiki/Convergence_of_Fourier_series*

### 10.2.2 Why Sine and Cosine Are the Building Blocks

Any smooth repeating wave can be decomposed into:
- **Fundamental frequency:** One complete oscillation per year
- **Second harmonic:** Two complete oscillations per year (captures mid-year pattern)
- **Third harmonic:** Three oscillations per year (captures quarterly pattern)
- etc.

Each harmonic needs both a sine and cosine term to capture phase (timing of peaks
and troughs) as well as amplitude:
$$a_n \cos(\omega_n t) + b_n \sin(\omega_n t) = R_n \cos(\omega_n t - \phi_n)$$

where $R_n = \sqrt{a_n^2 + b_n^2}$ is the amplitude and $\phi_n = \arctan(b_n/a_n)$
is the phase. The coefficients a_n and b_n are estimated from data; you never need
to specify the shape of the seasonal pattern in advance.

> **Teaching note:** This is the mathematical content of the Deep Dive box from the
> v2 course materials. For the course itself, explain the concept without the formula:
> "Think of seasonal patterns like music. Any repeating melody can be analyzed as
> a sum of pure tones at different frequencies. Prophet finds the combination of
> 'tones' that best fits your seasonal pattern. You don't need to specify the melody
> in advance — Prophet finds it automatically." Then reveal the formula for those
> who want the details. Budget 10 minutes.

*Reference: Wikipedia, "Fourier series" — en.wikipedia.org/wiki/Fourier_series*

### 10.2.3 Laplace Distribution

The Laplace distribution $\delta \sim \text{Laplace}(0, \tau)$ has density:
$$f(\delta) = \frac{1}{2\tau} e^{-|\delta|/\tau}$$

It has a sharp peak at 0 and heavier tails than the Normal. This creates a **sparse
prior**: most δ values are shrunk toward 0 (the peak), but a few are allowed to be
large (the heavy tails). Equivalent to L1 regularization (Lasso) in the frequentist
framework.

*Reference: Wikipedia, "Laplace distribution" — en.wikipedia.org/wiki/Laplace_distribution*
*Reference: Wikipedia, "Lasso (statistics)" — en.wikipedia.org/wiki/Lasso_(statistics)
(for L1 / Laplace connection)*

---

## 10.3 Trend Model

### Piecewise Linear Trend

The trend component allows the growth rate to change at specific times:

$$g(t) = (k + \mathbf{a}(t)^\top \boldsymbol{\delta}) t + (m + \mathbf{a}(t)^\top \boldsymbol{\gamma})$$

where:
- $k$ = initial growth rate (slope)
- $\boldsymbol{\delta} = [\delta_1, \ldots, \delta_S]$ = vector of slope changes at S changepoints
- $m$ = initial offset (intercept)
- $\boldsymbol{\gamma}$ = offset adjustments ensuring continuity at changepoints
- $\mathbf{a}(t) = [a_1(t), \ldots, a_S(t)]$ with $a_j(t) = \mathbb{1}[t \geq s_j]$
  (indicator that changepoint j has been passed)

**Continuity requirement:** At each changepoint $s_j$, the trend must be continuous:
$$g(s_j^-) = g(s_j^+)$$

This gives: $\gamma_j = (s_j - m - \mathbf{a}(s_j)^\top \boldsymbol{\gamma})(-\sum_{k<j}\delta_k) / k$
(simplified form in Taylor and Letham 2018).

### Changepoint Selection: Laplace Prior

Prophet places S potential changepoints uniformly throughout the training data.
The prior on each slope change:
$$\delta_j \sim \text{Laplace}(0, \tau)$$

**Effect of τ:**
- Small τ (e.g., 0.01): most δ_j ≈ 0, trend is nearly linear (smooth)
- Large τ (e.g., 10): many large δ_j, trend is highly flexible (risk of overfitting)

Prophet default: τ = 0.05. Tunable via `changepoint_prior_scale` parameter.

This is equivalent to Lasso regularization on the changepoint magnitudes: the L1
penalty (|δ_j|/τ) shrinks most changepoints to zero while allowing a few to be large.

### Automatic vs. Manual Changepoints

**Automatic:** Prophet places S = 25 potential changepoints in the first 80% of the
training data and selects significant ones via the Laplace prior.

**Manual:** When you know a structural break occurred (major campaign ended, competitor
entered, COVID lockdown), specify it explicitly:
```python
m = Prophet(changepoints=['2020-03-15', '2022-01-01'])
```

Manual changepoints encode domain knowledge the model cannot infer from data alone.

*Reference: Taylor, S. J. and Letham, B. (2018). "Forecasting at Scale."
The American Statistician 72(1):37–45. Free preprint: peerj.com/preprints/3190/*

---

## 10.4 Seasonality: Fourier Series Implementation

### The Fourier Feature Matrix

For yearly seasonality (period P = 365.25 days) with N Fourier terms:
$$s(t) = \sum_{n=1}^{N}\left[a_n \cos\left(\frac{2\pi n t}{P}\right) + b_n \sin\left(\frac{2\pi n t}{P}\right)\right] = \mathbf{X}(t)^\top \boldsymbol{\beta}$$

where $\mathbf{X}(t)$ is the 2N-dimensional Fourier feature vector at time t, and
$\boldsymbol{\beta} = [a_1, b_1, a_2, b_2, \ldots, a_N, b_N]^\top$ are estimated from data.

**Choosing N:**
- N=10 (Prophet default): captures seasonal patterns with periods as short as 36 days
- N=3: smooth, gentle seasonality only
- N=20: very flexible; risk of overfitting on short time series

**Prior:** $\boldsymbol{\beta} \sim \text{Normal}(0, \sigma_s^2 I)$. The parameter
`seasonality_prior_scale` = σ_s controls seasonal flexibility.

### Worked Illustration

For N=1 (one Fourier pair), yearly seasonality becomes:
$$s(t) = a_1 \cos\left(\frac{2\pi t}{365.25}\right) + b_1 \sin\left(\frac{2\pi t}{365.25}\right)$$

This produces a single smooth wave peaking once per year. The timing of the peak
is determined by the ratio b₁/a₁, and the amplitude by $\sqrt{a_1^2 + b_1^2}$.
With N=10, the model can represent patterns with peaks at multiple points per year,
asymmetric shapes (sharper rise than fall), and secondary seasonal effects.

### Multiple Seasonality Components

Prophet supports multiple seasonality components simultaneously:
- Yearly (P = 365.25, N = 10 default)
- Weekly (P = 7, N = 3 default)
- Custom (e.g., monthly P = 30.5, quarterly P = 91.25)

The total seasonality: $s(t) = s_{yearly}(t) + s_{weekly}(t) + \ldots$

---

## 10.5 Holiday Effects

### Specification

For holiday event e with associated dates $D_e$:
$$h(t) = \sum_{e=1}^{E} \kappa_e \mathbf{1}[t \in D_e]$$

Each κ_e is estimated with prior $\kappa_e \sim \text{Normal}(0, v^2)$, where
v = `holidays_prior_scale` (default 10, allowing large holiday effects).

### Window Effects

Real holidays have multi-day effects. Black Friday's influence begins Wednesday
(online deals) and extends through Cyber Monday. Prophet supports windows:
```python
holidays = pd.DataFrame({
  'holiday': 'black_friday',
  'ds': pd.to_datetime(['2020-11-27', '2021-11-26', '2022-11-25']),
  'lower_window': -2,   # effect starts 2 days before
  'upper_window': 3,    # effect lasts 3 days after
})
```

Each day in the window gets its own effect coefficient, enabling the model to capture
lead-up and hangover effects separately.

---

## 10.6 Model Fitting and Full Specification

### Complete Prophet Model

$$y(t) = g(t) + s(t) + h(t) + \varepsilon_t, \quad \varepsilon_t \sim \text{Normal}(0, \sigma^2)$$

**Bayesian estimation via Stan:** Prophet estimates all parameters (k, m, δ, β,
κ, σ²) using Stan, a probabilistic programming framework that implements
Hamiltonian Monte Carlo (HMC) sampling. The posterior over all parameters is:

$$P(k, m, \boldsymbol{\delta}, \boldsymbol{\beta}, \boldsymbol{\kappa}, \sigma^2 \mid \mathbf{y}) \propto P(\mathbf{y} \mid \ldots) \cdot P(\boldsymbol{\delta} \mid \tau) \cdot P(\boldsymbol{\beta} \mid \sigma_s) \cdot P(\boldsymbol{\kappa} \mid v) \cdot \ldots$$

In practice, Prophet can use MAP estimation (much faster, no posterior samples) or
full MCMC (slower, provides posterior uncertainty).

*Reference: arXiv:1111.4246, Stan Development Team. "Stan: A probabilistic programming language." arxiv.org/abs/1111.4246*

---

## 10.7 Forecast Accuracy Evaluation

### MAPE

$$\text{MAPE} = \frac{1}{n}\sum_{t=1}^n \frac{|y_t - \hat{y}_t|}{|y_t|} \times 100\%$$

**Limitations:**
- Undefined when $y_t = 0$
- Asymmetric: overforecasting and underforecasting are treated differently
- Not robust to small values (a 5-unit error on a 10-unit actual = 50% MAPE)

**Alternative:** SMAPE (symmetric MAPE):
$$\text{SMAPE} = \frac{100\%}{n}\sum_{t=1}^n \frac{|y_t - \hat{y}_t|}{(|y_t| + |\hat{y}_t|)/2}$$

Bounded in [0%, 200%] and treats over/underforecasting symmetrically.

*Reference: Wikipedia, "Mean absolute percentage error" —
en.wikipedia.org/wiki/Mean_absolute_percentage_error*
*Reference: Wikipedia, "Symmetric mean absolute percentage error" —
en.wikipedia.org/wiki/Symmetric_mean_absolute_percentage_error*

### MAPE Benchmarks by Context

| MAPE | Context-dependent assessment |
|---|---|
| < 5% | Excellent for weekly consumer demand |
| 5–10% | Good for most business planning |
| 10–20% | Acceptable for rough planning; check if structural improvements are possible |
| 20–30% | Marginal; may be irreducible noise or model misconfiguration |
| > 30% | Poor; investigate for changepoints, missing regressors, or data quality issues |

**Critical caveat:** These benchmarks are context-dependent. MAPE = 15% for fashion
retail (inherently unpredictable) may be excellent. MAPE = 3% for electricity demand
(highly predictable) may indicate a poorly specified model is fitting noise.

### Cross-Validation for Time Series

Standard k-fold cross-validation is inappropriate for time series (it leaks future
information into training). Correct approach: **rolling origin forecasting**:

1. Train on observations [1, T₀]
2. Forecast [T₀+1, T₀+H] (horizon H)
3. Advance T₀ by period P, retrain, forecast again
4. Aggregate MAPE across all forecasting origins

Prophet implements this via `cross_validation()`:
```python
from prophet.diagnostics import cross_validation, performance_metrics
df_cv = cross_validation(m,
    initial='730 days',   # minimum training period
    period='30 days',     # how much to advance the cutoff each time
    horizon='90 days'     # forecast horizon to evaluate
)
df_perf = performance_metrics(df_cv)
```

*Reference: Hyndman, R. J. and Athanasopoulos, G. *Forecasting: Principles and
Practice*, 3rd ed. Chapter 5 (Cross-validation). Free: otexts.com/fpp3/tscv.html*

---

## 10.8 Uncertainty Quantification

### Sources of Prediction Interval Width

Prophet generates intervals from three sources:

**1. Trend uncertainty:** Future changepoints may occur. Prophet samples future
slope changes from the same Laplace distribution (using the historical rate of
changepoint occurrence as a rate parameter). This is the dominant source of
uncertainty at long horizons.

**2. Observation noise:** $\sigma^2$ estimated from residuals during fitting.
Adds a constant floor to the uncertainty interval.

**3. Parameter uncertainty (full MCMC only):** If using MCMC sampling, uncertainty
in the estimated coefficients (β, κ, δ) is propagated through the forecast.

### Why Intervals Widen With Horizon

At horizon 1, the trend is well-constrained by recent observations. At horizon 26,
multiple potential future changepoints could have occurred, each shifting the trend.
The variance of the trend estimate grows approximately linearly with horizon:

$$\text{Var}[\text{trend}(T+h)] \approx \text{Var}[\text{trend}(T)] + h \cdot \sigma^2_{\text{changepoint}}$$

This is normal, expected, and honest. A forecast with constant-width intervals at all
horizons is either wrong or underestimating uncertainty.

### Decision-Making Under Uncertainty

Given prediction interval [L, U] at horizon h:

| Decision type | Use |
|---|---|
| Symmetric costs | Point forecast (median) |
| Underage >> Overage (stockout costly) | 75th–90th percentile |
| Overage >> Underage (waste costly) | 25th–40th percentile |
| Safety stock calculation | U - point_forecast (range of excess demand) |

---

## 10.9 Full Proof Appendix

### Proof: Fourier Series Represents Any Smooth Periodic Function (Summary)

For a periodic function f with period P satisfying the Dirichlet conditions:
1. f is absolutely integrable: $\int_0^P |f(t)| \, dt < \infty$
2. f has at most finitely many discontinuities in [0, P]
3. f has at most finitely many maxima and minima in [0, P]

Then the Fourier series $S_N(t) = \frac{a_0}{2} + \sum_{n=1}^N [a_n\cos(\frac{2\pi nt}{P}) + b_n\sin(\frac{2\pi nt}{P})]$ satisfies:

$$\lim_{N \to \infty} \int_0^P |f(t) - S_N(t)|^2 \, dt = 0 \quad \text{(L² convergence)}$$

For functions with more regularity (continuous derivative), convergence is uniform.
Seasonal demand patterns are smooth and periodic (the same season recurs each year),
so Fourier approximation converges rapidly. N=10 terms is sufficient for virtually
all business time series.

*Reference: Wikipedia, "Convergence of Fourier series" —
en.wikipedia.org/wiki/Convergence_of_Fourier_series*

### Proof: Laplace Prior Induces Sparse Changepoints (L1 Regularization Connection)

The MAP (maximum a posteriori) estimate under a Laplace prior $\delta_j \sim \text{Laplace}(0, \tau)$:

$$\hat{\boldsymbol{\delta}} = \arg\min_{\boldsymbol{\delta}} \left[-\log L(\boldsymbol{\delta} \mid y) + \frac{1}{\tau}\sum_j |\delta_j|\right]$$

The second term is exactly the Lasso penalty. Lasso is known to produce sparse
solutions — many $\hat{\delta}_j = 0$ exactly, with only significant changepoints
receiving nonzero values. The sparsity level is controlled by 1/τ: smaller τ (stronger
prior) means more sparsity (fewer changepoints); larger τ means more flexibility.

*Reference: Wikipedia, "Lasso (statistics)" — en.wikipedia.org/wiki/Lasso_(statistics)*

---

## 10.10 Verification Checklist

| Check | How to verify |
|---|---|
| MAPE computed on held-out test period, not training? | Check time split |
| Decomposition components sum to observed values? | $g(t) + s(t) + h(t) \approx y(t)$ |
| Prediction interval widens with horizon? | Expected; if it doesn't, check settings |
| Holiday dates correctly specified (day-of, not day-after)? | Verify with calendar |
| Seasonal multipliers compound? | 38% × 5% growth = 1.38 × 1.05 = 1.449, not 1.43 |
| changepoint_prior_scale appropriate? | Default 0.05; increase if underfitting trend |
| Cross-validation horizon matches decision horizon? | 90-day horizon for quarterly planning |
| Manual changepoints added for known structural breaks? | Check campaign/event dates |

---

## 10.11 Chapter Bibliography

1. Wikipedia, "Fourier series" — en.wikipedia.org/wiki/Fourier_series
2. Wikipedia, "Convergence of Fourier series" — en.wikipedia.org/wiki/Convergence_of_Fourier_series
3. Wikipedia, "Decomposition of time series" — en.wikipedia.org/wiki/Decomposition_of_time_series
4. Wikipedia, "Laplace distribution" — en.wikipedia.org/wiki/Laplace_distribution
5. Wikipedia, "Lasso (statistics)" — en.wikipedia.org/wiki/Lasso_(statistics)
6. Wikipedia, "Mean absolute percentage error" — en.wikipedia.org/wiki/Mean_absolute_percentage_error
7. Taylor, S. J. and Letham, B. (2018). "Forecasting at Scale." The American Statistician 72(1):37–45. Free preprint: peerj.com/preprints/3190/
8. Prophet documentation and tutorials: facebook.github.io/prophet/docs/quick_start.html
9. Hyndman, R. J. and Athanasopoulos, G. Forecasting: Principles and Practice, 3rd ed. Free: otexts.com/fpp3/
10. Hyndman, R. J. and Koehler, A. B. (2006). "Another look at measures of forecast accuracy." International Journal of Forecasting 22(4):679–688. arXiv:cs/0605103. arxiv.org/abs/cs/0605103
11. MIT OpenCourseWare, 15.075 Statistical Thinking and Data Analysis — ocw.mit.edu

**Lenny's Podcast episode references:**
- Jess Lachs episode on building data organizations at DoorDash — Lenny's Podcast
- Sri Batchu episode on growth at Instacart and Ramp — Lenny's Podcast
