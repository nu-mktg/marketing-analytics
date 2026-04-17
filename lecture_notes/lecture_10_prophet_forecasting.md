# Lecture 10: Demand Forecasting with Prophet
## Understanding Where Your Business Is Heading

---

### Overview

**Business question:** It is October. Your procurement team needs to order inventory for the holiday season. How much demand should you plan for — and how confident should you be in that plan?

**What you will be able to do:**
- Read and interpret Prophet's decomposed output (trend, seasonality, holidays)
- Use the uncertainty interval to make inventory or budget decisions
- Identify when a model assumption has changed and how to address it
- Evaluate forecast accuracy using MAPE
- Explain when forecasting cannot help — and what to do instead

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Percentages as Multipliers

A 38% seasonal lift means the weekly revenue is 1.38× the baseline.
> Baseline = $1,000 → With 38% lift: $1,000 × 1.38 = **$1,380**

A −15% seasonal dip means 0.85× the baseline.
> Baseline = $1,000 → With −15% dip: $1,000 × 0.85 = **$850**

Multipliers compound: a 38% lift on a 5% growth trend = 1.38 × 1.05 = 1.449 = 44.9% above prior-year baseline.

---

#### Tool 2: Mean Absolute Percentage Error (MAPE)

MAPE measures how far off a forecast is, on average, as a percentage of actual values.

> MAPE = (1/n) × Σ |actual − predicted| / actual × 100%

**Example:** 4 weeks of actuals vs. predictions:
| Actual | Predicted | |Actual−Pred|/Actual |
|---|---|---|
| 100 | 95 | 5.0% |
| 120 | 130 | 8.3% |
| 80 | 88 | 10.0% |
| 110 | 115 | 4.5% |
MAPE = (5.0 + 8.3 + 10.0 + 4.5)/4 = **6.9%**

---

### Section 1.2 — Business Motivation

---

#### The Leading Indicator Problem

Jess Lachs, who built the data team at DoorDash, crystallizes the challenge:

> *"Retention is a terrible thing to goal on. It's almost impossible to drive in a meaningful way in the short term. Ultimately, you want to find a short-term metric you can measure that drives a long-term output."*

Demand forecasting is this principle in practice. A Prophet model decomposes your revenue into trend + seasonality + holiday effects. The decomposition output — not the point forecast — is where the business insight lives.

Knowing that your trend is growing 12% annually, Q4 is 38% above the average, and Black Friday week adds 45% on top of that Q4 baseline lets you make better operational decisions than knowing only "expected sales next quarter = $2.3M." The components are actionable; the single number is not.

Sri Batchu, who led growth at Instacart and Ramp, describes the channel allocation analog:

> *"Our goal is to over time make all of our channels more efficient, but also allocate more to channels that are more efficient as long as they're scalable."*

A demand forecast tells you when to concentrate spend and when to conserve — which periods have genuine underlying demand growth vs. which are seasonal artifacts.

---

### Section 1.3 — Conceptual Framework

---

#### What Prophet Actually Does

Prophet decomposes a time series into three components:

$$y(t) = \text{trend}(t) + \text{seasonality}(t) + \text{holidays}(t) + \varepsilon$$

**Trend:** The underlying growth or decline direction. Prophet allows for changepoints — moments when the trend shifts. It detects these automatically or you can specify them manually.

**Seasonality:** Recurring patterns within a year (e.g., Q4 is consistently higher). Prophet models these patterns using a flexible curve that captures peaks and troughs without you specifying their exact shape. You do not need to know trigonometry — Prophet fits this automatically.

**Holidays:** Custom one-off effects for known events (Black Friday, Christmas, product launches). You specify the event dates; Prophet estimates the impact.

**Why decomposition matters:** The component plot is often more valuable than the point forecast. A business that knows TV spend peaks in Q4 (+38%), drops in Q2 (−15%), and spikes around Black Friday (+45% on top of Q4) can plan inventory, staffing, and marketing budgets with far more precision than one that has only a quarterly revenue estimate.

> ### 🔍 Deep Dive: How Prophet Models Seasonality Internally
> Internally, Prophet represents seasonal patterns using Fourier series — sums of sine and cosine curves at different frequencies. A full-year seasonal pattern might use 10 Fourier terms (5 sine + 5 cosine curves at harmonics 1/year, 2/year, 3/year, 4/year, 5/year). The coefficients on these terms are fit from the data. The resulting curve can capture any smooth repeating pattern without you specifying its shape in advance. This is why you do not see "seasonality = sin(2πt/52)" in your model output — Prophet shows you the resulting seasonal curve, not the internal Fourier components.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: Reading Forecast Output

Prophet produces three key outputs per time period:
- **yhat:** Point forecast (the expected value)
- **yhat_lower / yhat_upper:** 95% uncertainty interval (the range containing 95% of plausible outcomes)

**Reading the uncertainty interval:**
> Week 1: yhat = $980k, interval = [$890k, $1,070k]
> Week 26: yhat = $1,050k, interval = [$800k, $1,300k]

Uncertainty grows with forecast horizon because errors compound over time. A wide week-26 interval is not a model failure — it is the model being honest about irreducible uncertainty.

**Using the interval for decisions:** If ordering too little costs 2× as much as ordering too much (asymmetric cost), plan to the upper end of the interval. If the costs are symmetric, plan to the midpoint (yhat).

---

#### Part B: Changepoints

A changepoint is a moment when the trend changes direction or magnitude. Prophet detects these automatically in the training data.

**When to add manual changepoints:** If you know the data-generating process changed at a specific moment (e.g., a competitor launched aggressively, a major promotional campaign ran), you should add a changepoint at that date. A model trained without this changepoint will misattribute the trend shift and produce biased forecasts.

**The promotional campaign trap:** A model trained on 3 years of data that includes a major promotional campaign will try to extrapolate that campaign's growth forward — even if the campaign has ended. Adding a changepoint at the end of the campaign tells the model "the growth from that campaign is not part of the long-run trend."

---

#### Part C: Evaluating Forecast Accuracy

Always validate on a held-out period before using a forecast for decisions.

**Standard approach:**
1. Train on weeks 1–182 (3.5 years)
2. Forecast weeks 183–208 (6 months forward)
3. Compare forecast to actuals using MAPE

**What MAPE means for decisions:**
- MAPE = 8%: Excellent for most planning purposes. You can typically trust budget allocations and inventory orders based on this forecast.
- MAPE = 22%: Acceptable for rough planning but not for just-in-time inventory. Consider adding safety stock.
- MAPE = 45%+: The model is not reliable. Something structural is missing (probably a changepoint or missing regressor).

**MAPE usefulness depends on context:** A 20% MAPE on weekly demand for a fashion product may be excellent (fashion is inherently unpredictable). A 5% MAPE on electricity demand for a power plant may be dangerously imprecise (utilities need < 2% accuracy).

---

### Part 1 Checkpoint

1. Baseline weekly revenue = $1,200k. Seasonal component for Q4 = +42%. Black Friday adds another +38% on top of Q4. Estimate Black Friday week revenue.

2. A forecast shows MAPE = 18% on a 13-week holdout. Your manager says "That's too inaccurate to use." Is this always correct?

3. The uncertainty interval for week 1 is [$950k, $1,050k] and for week 26 is [$750k, $1,350k]. Is the week-26 interval a sign that the model is broken?

4. Your company ran a major promotional campaign in Year 2 that boosted revenue by 70%. The campaign ended. Prophet was trained on 3 years of data including Year 2. What will the model likely do with the promotional growth?

5. Two models: MAPE_A = 8%, MAPE_B = 22%. For rough annual budget planning, is Model A always preferable to Model B?

---

### Checkpoint Answer Key

**Q1.** Q4 baseline = $1,200k × 1.42 = $1,704k. Black Friday = $1,704k × 1.38 = **$2,351k**.
*Common wrong answer:* $1,200k × (1.42 + 1.38) = $1,200k × 2.80 = $3,360k. Seasonal lifts apply multiplicatively (each multiplied onto the previous result), not additively.

**Q2.** **Not always.** MAPE usefulness depends on context. 18% may be excellent for rough planning (staffing, marketing budgets) where decisions have significant uncertainty anyway. It may be unacceptable for just-in-time inventory or financial commitments with tight margins. The manager's statement ignores that the alternative to an 18% MAPE model may be gut feel (no model), which typically has much higher effective error.
*Common wrong answer:* The manager is correct — any MAPE above some threshold (like 10%) makes a model useless. There is no universal threshold; it depends on decision stakes.

**Q3.** **No.** Wider uncertainty at longer forecast horizons is mathematically expected: each period's uncertainty compounds. A week-26 interval that is wider than week-1 indicates the model is being appropriately honest about the limits of long-range forecasting — not that the model is broken.
*Common wrong answer:* Yes, because a good model should be equally accurate at all time horizons. Forecast accuracy degrades with horizon in virtually all forecasting models.

**Q4.** The model will likely **extrapolate the promotional growth as part of the long-run trend** — forecasting Year 3 and beyond as if the campaign's 70% lift continues indefinitely. This will cause systematic overestimates. The fix: add a changepoint at the end of the promotional campaign, or add the campaign as a holiday regressor with specific dates.
*Common wrong answer:* Prophet detects the campaign as a holiday effect automatically. Prophet's holiday component only works for events you specify explicitly. It cannot automatically detect that a boost in Year 2 was a one-time promotion vs. a structural change.

**Q5.** **Not necessarily for rough annual planning.** For annual budget planning, you aggregate weekly forecasts into a full-year estimate. Aggregation reduces percentage error (some weeks' overestimates offset underestimates). Both 8% and 22% MAPE may produce similar annual totals. Model B may be preferable if it is simpler to explain, update, or maintain — and the accuracy difference does not change any annual budget decision materially.
*Common wrong answer:* Yes, lower MAPE is always better. Lower MAPE is better for granular decisions (weekly inventory orders) but may not matter for aggregate annual planning.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal)


---

**Problem Statement**

A consumer goods company has quarterly sales data for 2022 and 2023:

| Quarter | Year | $t$ | Units Sold ($y_t$) |
|---|---|---|---|
| Q1 | 2022 | 1 | 120 |
| Q2 | 2022 | 2 | 95 |
| Q3 | 2022 | 3 | 110 |
| Q4 | 2022 | 4 | 180 |
| Q1 | 2023 | 5 | 130 |
| Q2 | 2023 | 6 | 100 |
| Q3 | 2023 | 7 | 120 |
| Q4 | 2023 | 8 | 200 |

**Part A:** Compute the seasonal indices by averaging each quarter across both years. Then compute the overall mean. Do the seasonal indices sum to approximately 0? (For additive seasonality, they should.)

**Part B:** A naive seasonal forecast uses last year's same quarter as the forecast. Compute MAPE for 2023 using 2022 actuals as the forecast.

**Part C:** A trend-adjusted forecast adds an estimated trend of +10 units per year (= +2.5 units per quarter) to last year's same quarter. Compute MAPE for 2023 under the trend-adjusted forecast. Compare with Part B.

---

**Full Solution**

**Part A: Seasonal Indices**

Overall mean: $(120+95+110+180+130+100+120+200)/8 = 1055/8 = 131.9$ units

Quarterly averages across both years:
$$\bar{y}_{Q1} = (120+130)/2 = 125.0$$
$$\bar{y}_{Q2} = (95+100)/2 = 97.5$$
$$\bar{y}_{Q3} = (110+120)/2 = 115.0$$
$$\bar{y}_{Q4} = (180+200)/2 = 190.0$$

Seasonal indices (quarterly average − overall mean):
$$s_{Q1} = 125.0 - 131.9 = \mathbf{-6.9}$$
$$s_{Q2} = 97.5 - 131.9 = \mathbf{-34.4}$$
$$s_{Q3} = 115.0 - 131.9 = \mathbf{-16.9}$$
$$s_{Q4} = 190.0 - 131.9 = \mathbf{+58.1}$$

Sum check: $-6.9 - 34.4 - 16.9 + 58.1 = -0.1 \approx 0$ ✓

Interpretation: Q4 is 58 units above average (+44%), Q2 is 34 units below average (−26%). This is a strong seasonal pattern — inventory planning must account for this.

**Part B: Naive Seasonal Forecast MAPE**

Forecast = last year's same quarter:

| Quarter | Forecast | Actual | Absolute Error | % Error |
|---|---|---|---|---|
| Q1 2023 | 120 | 130 | 10 | 10/130 = 7.7% |
| Q2 2023 | 95 | 100 | 5 | 5/100 = 5.0% |
| Q3 2023 | 110 | 120 | 10 | 10/120 = 8.3% |
| Q4 2023 | 180 | 200 | 20 | 20/200 = 10.0% |

$$\text{MAPE}_{\text{naive}} = (7.7 + 5.0 + 8.3 + 10.0) / 4 = \mathbf{7.75\%}$$

**Part C: Trend-Adjusted Forecast MAPE**

Forecast = last year's same quarter + 10 units (one year of trend):

| Quarter | Forecast | Actual | Absolute Error | % Error |
|---|---|---|---|---|
| Q1 2023 | 120+10=130 | 130 | 0 | 0.0% |
| Q2 2023 | 95+10=105 | 100 | 5 | 5/100 = 5.0% |
| Q3 2023 | 110+10=120 | 120 | 0 | 0.0% |
| Q4 2023 | 180+10=190 | 200 | 10 | 10/200 = 5.0% |

$$\text{MAPE}_{\text{trend}} = (0.0 + 5.0 + 0.0 + 5.0) / 4 = \mathbf{2.5\%}$$

**Comparison:** Adding the trend reduces MAPE from 7.75% to 2.5% — a 68% improvement. This demonstrates the value of even a simple trend adjustment. Prophet's full model would further improve on this by using the Fourier series to more precisely model the seasonal pattern and detecting any changepoints in the trend.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The `plot_components()` output:** Prophet automatically generates a 3-panel plot:
- **Trend panel:** Shows the fitted trend line with changepoints marked as vertical dashed lines. A sudden change in slope at a changepoint represents a detected structural shift in growth rate.
- **Yearly seasonality panel:** Shows the estimated seasonal effect for each day/week of the year. Q4 peaks should be clearly visible for holiday-driven businesses.
- **Holiday panel:** Shows the estimated effect of each specified holiday. Positive values indicate sales lift; negative values indicate drag.

**Interpreting changepoints:** Each vertical line in the trend panel marks where the trend slope changed. If a changepoint coincides with a known business event (new product launch, competitor entry), this is reassuring. If changepoints appear in the middle of otherwise stable periods, it may indicate the model is overfitting.

**Interpreting uncertainty intervals:** Uncertainty intervals in the forecast plot should widen as you forecast further ahead. A constant-width interval is a warning — the model may be underestimating long-run uncertainty. Intervals that are very narrow (much less than the historical variation) are also suspect.

**MAPE from cross-validation:** Run `performance_metrics()` after `cross_validation()`. Pay attention to how MAPE varies by forecast horizon. MAPE at horizon = 1 week is typically much lower than MAPE at horizon = 26 weeks. For inventory planning, use the MAPE at the specific horizon you care about (e.g., 8 weeks for 8-week lead times).

**Before trusting the agent output:**
1. Does the trend direction match your business knowledge? If Prophet shows a declining trend when you know the market is growing, the model is wrong.
2. Does the yearly seasonality pattern match domain knowledge? Holiday peaks, summer dips?
3. Are changepoints located near real business events, or do they appear arbitrary?
4. Is MAPE from cross-validation within acceptable bounds for your use case?

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/weekly_sales.csv` + `homework_10.ipynb`

---

#### Part A: Math Questions (no agent required)

Use the quarterly dataset from the worked example for all calculations.

**Q1.** What is the seasonal index for Q4? (Seasonal index = quarterly average − overall mean)
```python
q1_seasonal_q4 = None
```

**Q2.** What is the seasonal index for Q2?
```python
q2_seasonal_q2 = None
```

**Q3.** Do the four seasonal indices sum to approximately 0? (Enter True or False)
```python
q3_sum_to_zero = None
```

**Q4.** Using the naive seasonal forecast (last year's same quarter), what is the % error for Q4 2023? (Forecast=180, Actual=200)
```python
q4_q4_pct_error = None
```

**Q5.** Using the trend-adjusted forecast (add 10 units), what is the Q4 2023 forecast?
```python
q5_q4_trend_forecast = None
```

**Q6.** What is the % error for Q4 2023 using the trend-adjusted forecast?
```python
q6_q4_trend_error = None
```

**Q7.** With quarterly data ($P = 4$) and $N = 1$, the Fourier seasonality has how many parameters (coefficients to estimate)?
```python
q7_fourier_params = None
```

**Q8.** With $N = 2$, how many parameters does the Fourier seasonal component have?
```python
q8_fourier_params_n2 = None
```

**Q9.** Evaluate the Fourier term $\cos(2\pi \times 4 / 4)$. (This is the Q4 value of the n=1 cosine term with P=4.)
```python
q9_cos_q4 = None
```

**Q10.** True or False: For standard k-fold cross-validation on weekly time series data, it is acceptable for a training fold to contain data from weeks after the test fold.
```python
q10_kfold_acceptable = None
```

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I am building a demand forecast for a consumer goods company.

weekly_sales.csv has one row per week:
- ds: date (YYYY-MM-DD format, always a Sunday)
- y: units sold that week

I also have a list of promotional events that typically boost sales:
- Spring Promotion: runs first week of March annually
- Back-to-School: runs third week of August annually
- Black Friday: last Friday of November annually
- Holiday Season: runs December 15 through December 31 annually
  (use lower_window=-14 and upper_window=0 for this)

Please:
1. Fit a Prophet model with:
   - yearly_seasonality=True, weekly_seasonality=False (weekly data)
   - multiplicative seasonality mode (since peaks scale with trend)
   - The 4 promotional events as custom holidays
   - changepoint_prior_scale=0.05 (default)
2. Generate a 26-week forecast with uncertainty intervals.
3. Plot the forecast and the components (trend, seasonality, holidays).
4. Run temporal cross-validation: initial="104 weeks", period="13 weeks",
   horizon="13 weeks". Report MAPE at the 13-week horizon.
5. Report: the week with the highest forecasted demand, and the estimated
   % lift from the Holiday Season event.
Use random seed 42.
```

**Q11.** What is the MAPE at the 13-week forecast horizon from cross-validation? Round to 1 decimal place (as a %).
```python
q11_mape_13wk = None
```

**Q12.** According to the forecast, which week has the highest predicted demand? (Enter as a date string, e.g., "2024-12-22")
```python
q12_peak_week = None
```

**Q13.** What is the estimated % lift from the Holiday Season event (the holiday coefficient, approximately)? Round to 1 decimal place.
```python
q13_holiday_lift_pct = None
```

**Q14.** Does the trend component show an upward or downward direction over the full historical period?
```python
q14_trend_direction = None  # "upward" or "downward"
```

---

#### Part C: Interpretation Questions

**Q15.** MAPE at 13 weeks is 11.2%. A supply chain manager says this is too high for inventory planning. What are two things you could try to improve forecast accuracy?
- (a) Add more Fourier terms (increase N) and switch from multiplicative to additive seasonality
- (b) Add more historical data, validate and refine the holiday specifications, and tune changepoint_prior_scale
- (c) Use a different dataset and run the forecast again
- (d) Ignore MAPE and rely on the uncertainty intervals instead
```python
q15 = None  # "a", "b", "c", or "d"
```

**Q16.** The forecast plot shows a very steep trend upward for the next 6 months based on a recent sharp growth period. You know this growth was caused by a temporary promotion that has ended. What should you do?
- (a) Trust the model — it learned from the data
- (b) Manually add a changepoint at the end of the promotion period to prevent the model from extrapolating the temporary growth
- (c) Use a different forecasting model entirely
- (d) Nothing — Prophet will automatically detect that the trend has changed once more data arrives
```python
q16 = None  # "a", "b", "c", or "d"
```

**Q17.** Prophet's uncertainty interval at 26 weeks is ±35% of the point forecast. A manager says: "That's too wide to be useful — we need a precise forecast." What is the correct response?
- (a) Agree — a 35% interval is always a sign of a bad model
- (b) Disagree — forecast uncertainty increases with horizon; a 35% interval at 26 weeks may be honest and appropriate. Artificially narrow intervals give false confidence.
- (c) Agree — switch to a simpler model with tighter intervals
- (d) Disagree — Prophet's intervals are always too wide and can be ignored
```python
q17 = None  # "a", "b", "c", or "d"
```

---

### Common Misconceptions

**1. "Prophet is always better than simple forecasting methods."**
Prophet performs best with several years of data, clear seasonal patterns, and known holidays. For very short series (less than 2 years), or highly irregular data with no seasonal pattern, simpler methods (exponential smoothing, ARIMA) often perform comparably or better.

**2. "More Fourier terms (larger N) always improves forecasts."**
Larger N increases flexibility, which improves fit on training data but may worsen out-of-sample accuracy (overfitting). Always validate with temporal cross-validation, not training error.

**3. "The 95% uncertainty interval means 95% of future observations will fall within it."**
This would be true if the model were perfectly specified. In practice, the intervals are model-based — they reflect parameter uncertainty under the model's assumptions. Real-world uncertainty (structural breaks, unexpected events) is not captured, so actual coverage may be lower.

**4. "You should always include all known holidays."**
Only include holidays that systematically affect your specific variable. Including holidays that have no effect on your product adds noise and reduces model accuracy. If Mother's Day does not affect your B2B product, do not include it.

**5. "A low training MAPE means the model forecasts well."**
Training MAPE measures fit, not forecast accuracy. A model that memorizes the training data perfectly (MAPE = 0 in training) may forecast terribly. Always evaluate on held-out periods using temporal cross-validation.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Decomposes $y(t) = g(t) + s(t) + h(t) + \varepsilon$ | Section 1.3 — additive decomposition |
| Fits piecewise linear trend with changepoints | Section 1.3 — trend model |
| Fits Fourier series for seasonality | Section 1.4A — Fourier coefficients |
| Adds holiday indicators | Section 1.3 — holiday effects |
| `plot_components()` | Section 2.2 — interpreting trend/seasonality panels |
| `cross_validation()` | Section 1.4B — temporal cross-validation |
| `performance_metrics()` → MAPE | Section 1.1C — MAPE formula |

**What to verify before trusting the output:**
1. Trend direction matches domain knowledge
2. Seasonality pattern matches historical patterns
3. Holiday effects have correct signs (promotions positive, no unexpected negatives)
4. MAPE from cross-validation is within acceptable range for your decision horizon
5. Changepoints are located near known business events, not arbitrary periods

---

### Course Capstone Note

This is the final lecture of the course. You have now built a complete analytical toolkit covering:

| Lectures 1–2 | Probability + Regression foundations |
| Lecture 3 | Time-varying marketing effects (MMM) |
| Lecture 4 | Time-to-event analysis (Survival) |
| Lectures 5–6 | Sequential customer behavior (Markov + Attribution) |
| Lecture 7 | Causal treatment effects (Uplift) |
| Lecture 8 | Predictive customer value (CLV) |
| Lecture 9 | Customer preference structure (Conjoint) |
| Lecture 10 | Time series forecasting (Prophet) |

These ten models cover customer acquisition (Uplift, Conjoint), customer retention (CLV, Survival, Markov), marketing effectiveness (MMM, Attribution), experimentation (Bayesian A/B, Uplift), and planning (Prophet, Price Elasticity). Together they address virtually every quantitative marketing question you will encounter in practice. The agentic programming skills you have practiced throughout this course allow you to apply them to real data at any scale.
