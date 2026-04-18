# Model Reference Card
## Marketing Analytics — 14 Lectures at a Glance

---

| # | Model | Estimand | Required Inputs | Key Assumption | What Makes It Wrong |
|---|---|---|---|---|---|
| 1 | **Bayesian A/B Testing** | $P(\theta_B > \theta_A \mid \text{data})$; posterior mean and credible interval on lift | Visitor counts $n_A, n_B$; conversion counts $k_A, k_B$; prior $\text{Beta}(\alpha, \beta)$ | Prior is correctly specified; observations are i.i.d. Bernoulli | Small sample sizes inflated by a poorly chosen prior; non-stationary conversion rates during the experiment window |
| 2 | **Price Elasticity + OLS** | $\hat{\beta}_1 = \varepsilon$ (elasticity); revenue-maximizing price | $(P_i, Q_i)$ pairs; log transformation for constant elasticity | Exogeneity: $\text{Cov}(P_i, \varepsilon_i) = 0$ | Prices set in response to demand shocks (endogeneity); data range too narrow to identify curvature |
| 3 | **Marketing Mix Modelling** | Channel contribution $\beta_j$; marginal return to spend per channel | Weekly spend per channel; outcome (revenue/sales); time series length ≥ 2 years recommended | Correct adstock specification; no multicollinearity across channels | High collinearity in channel spend; incorrect decay rate $\lambda$; model fitted to too short a time series |
| 4 | **Survival Analysis (KM + Cox)** | $\hat{S}(t)$; hazard ratios $\text{HR}_k = e^{\hat{\beta}_k}$ | Customer ID; event indicator (churned/censored); event time; covariates (Cox only) | Censoring is non-informative; proportional hazards (Cox) | Informative censoring (customers are removed when they become at risk of churning); PH assumption violated |
| 5 | **Markov Chains** | $\boldsymbol{\pi}_{t+k}$: state distribution $k$ periods ahead; steady state $\boldsymbol{\pi}^*$ | State-labelled customer data at two or more time points; transition frequency counts | Markov property (next state depends only on current); stationarity of $P$ | State boundaries are poorly defined; $P$ is non-stationary (different dynamics by season or cohort) |
| 6 | **Shapley Attribution** | $\phi_i$: marginal contribution of channel $i$ to conversion rate | Multichannel conversion path data; conversion indicator per path | $v(S)$ is correctly estimated from observed paths | Sparse paths for large coalitions; conflates marginal contribution with causal incrementality |
| 7 | **Uplift Modelling (T-learner)** | $\hat{\tau}(x) = \hat{\mu}_1(x) - \hat{\mu}_0(x)$; CATE per customer | Outcome $Y$; treatment indicator $W$; covariates $X$; randomised assignment | Random treatment assignment (unconfounded); overlap (both treated and control exist at each $x$) | Treatment was not randomised (selection bias in $\hat{\mu}_0, \hat{\mu}_1$); rare treatment or control within covariate strata |
| 8 | **CLV — BG/NBD + Gamma-Gamma** | $P(\text{alive} \mid x, t_x, T)$; $E[\text{CLV}_{12}]$ | Recency $t_x$; frequency $x$; observation window $T$; order values (GG model) | Customers die at most once; frequency $\perp$ spend (Gamma-Gamma) | Subscription products (discrete renewal differs from BG/NBD's continuous churn model); strong frequency–spend correlation |
| 9 | **Conjoint Analysis (MNL)** | $\hat{\beta}_k$ per attribute; $\text{WTP}_k = \hat{\beta}_k / \lvert\hat{\beta}_{\text{price}}\rvert$; market shares via softmax | Choice data from stated-preference survey; attribute levels per alternative | IIA (Independence of Irrelevant Alternatives); random utility | Attributes are highly correlated; IIA fails when new alternatives are close substitutes for specific existing options |
| 10 | **Prophet Forecasting** | $\hat{y}(t) = g(t) + s(t) + h(t)$; prediction intervals at horizon $h$ | Weekly or daily time series of outcome variable; holiday calendar (optional) | Piecewise linear or logistic trend; Fourier seasonality; additive error | Missing regressors for known structural breaks (competitor entry, policy changes); very short series (< 2 seasonal cycles) |
| 11 | **Synthesis & Capstone** | Decision framework integrating all prior models; which model to apply and when | Prior model outputs across customer lifecycle stages | Each constituent model's assumptions must hold within its own application | Applying a model outside its identifying assumptions; missing a better-suited estimator for the question |
| 12 | **Difference-in-Differences** | $\hat{\tau}_{\text{DiD}} = (\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}}) - (\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}})$ | Panel data: outcome, time period, treatment/control group indicator | Parallel trends: treatment and control groups would have moved together absent intervention | Pre-existing divergent trends between groups; spillover from treatment to control; differential shocks |
| 13 | **Customer Segmentation (k-means)** | Cluster assignments $z_i \in \{1, \ldots, k\}$; centroid $\boldsymbol{\mu}_c$ per cluster; WCSS | Customer feature vectors $\mathbf{x}_i \in \mathbb{R}^p$ (standardised); number of clusters $k$ | Euclidean distance meaningful after standardisation; spherical cluster shapes | Clusters are non-spherical or very differently sized; $k$ chosen arbitrarily without diagnostic checks; features not standardised |
| 14 | **CausalImpact (BSTS)** | Posterior distribution over pointwise effect $\hat{\tau}_t = y_t - \hat{y}_t^{(0)}$; cumulative effect $\hat{\tau}_{\text{cum}}$; relative effect | Outcome time series; control covariate series uncorrelated with intervention; pre-period for model fit | Control covariates are unaffected by intervention; relationship between controls and outcome is stable across periods | Controls are themselves affected by intervention; poor pre-period model fit (high MAPE); short pre-period relative to seasonality |

---

## Key Formulas

| Model | Formula |
|---|---|
| Bayesian update | $\text{Posterior} \sim \text{Beta}(\alpha + k,\; \beta + n - k)$ |
| OLS slope | $\hat{\beta}_1 = S_{xy}/S_{xx}$ where $S_{xy} = \sum(x_i-\bar{x})(y_i-\bar{y})$ |
| Revenue max condition | $\varepsilon = -1$ (unit-elastic) |
| Adstock recursion | $A_t = S_t + \lambda A_{t-1}$; long-run $= S/(1-\lambda)$ |
| Hill function | $H(s) = s^\alpha / (\text{EC}_{50}^\alpha + s^\alpha)$; always $H(\text{EC}_{50}) = 0.5$ |
| Kaplan-Meier step | $\hat{S}(t) = \prod_{j:\,t_j \leq t} (n_j - d_j)/n_j$ |
| Conditional churn | $P(\text{churn} \in [t_1, t_2] \mid T > t_1) = 1 - \hat{S}(t_2)/\hat{S}(t_1)$ |
| Cox hazard ratio | $\text{HR}_k = e^{\hat{\beta}_k}$; for $c$ units: $e^{c\hat{\beta}_k}$ |
| Markov update | $\boldsymbol{\pi}_{t+1} = \boldsymbol{\pi}_t \cdot P$ |
| Steady state | $\boldsymbol{\pi} P = \boldsymbol{\pi},\; \sum_i \pi_i = 1$ |
| Shapley value | $\phi_i = \frac{1}{n!}\sum_{\text{orderings}} [v(S \cup \{i\}) - v(S)]$ |
| Efficiency axiom | $\sum_i \phi_i = v(N)$ |
| ATE | $\widehat{\text{ATE}} = \bar{Y}_{\text{treated}} - \bar{Y}_{\text{control}}$ |
| T-learner CATE | $\hat{\tau}(x) = \hat{\mu}_1(x) - \hat{\mu}_0(x)$ |
| Targeting threshold | Target iff $\hat{\tau}(x) > c/v$ |
| Simple CLV | $E[\text{CLV}] = E[\text{transactions}] \times E[\text{spend}]$ |
| WTP | $\text{WTP}_k = \hat{\beta}_k / \lvert\hat{\beta}_{\text{price}}\rvert$ |
| Softmax share | $P(\text{choose } j) = e^{U_j}/\sum_k e^{U_k}$ |
| MNL utility | $U_j = \hat{\beta}_{\text{price}} P_j + \sum_k \hat{\beta}_k A_{kj}$ |
| Prophet decomp | $y(t) = g(t) + s(t) + h(t) + \varepsilon_t$ |
| MAPE | $\frac{1}{n}\sum \lvert y_t - \hat{y}_t\rvert / \lvert y_t\rvert \times 100\%$ |

---

## Decision Tree: Which Model for Which Question?

```
What type of question?
│
├─ "Did an intervention work?"
│   ├─ Randomised experiment + binary outcome → Bayesian A/B Testing (L1)
│   ├─ Randomised experiment + heterogeneous effects wanted → Uplift (L7)
│   ├─ Geographic holdout design (pre/post + control regions) → DiD (L12)
│   └─ National campaign, no control group → CausalImpact (L14)
│
├─ "How does outcome change with a driver?"
│   ├─ Driver = price; outcome = quantity → Price Elasticity + OLS (L2)
│   ├─ Driver = channel spend; outcome = revenue → MMM (L3)
│   └─ Driver = touchpoints in conversion path → Shapley Attribution (L6)
│
├─ "What will happen to customers over time?"
│   ├─ Continuous time, event of interest → Survival Analysis (L4)
│   ├─ Discrete states, transition dynamics → Markov Chains (L5)
│   └─ Individual future purchase value → CLV / BG/NBD (L8)
│
├─ "What do customers want or what is coming?"
│   ├─ Feature valuation and willingness-to-pay → Conjoint (L9)
│   └─ Future demand, seasonal planning → Prophet (L10)
│
└─ "How do we group or target customers?"
    └─ Discover latent structure in customer features → Segmentation (L13)
```

---

## Assumption Violation Checklist (Before Presenting Results)

| Model | Check |
|---|---|
| OLS | Plot residuals vs. fitted — no fan shape (homoskedasticity); check endogeneity |
| Cox | Log-log survival plots parallel? (PH assumption); censoring non-informative? |
| Markov | Are state labels stable over time? Is P time-invariant? |
| T-learner | Covariate balance between treated and control? (run a balance check) |
| BG/NBD | Is this a subscription product? (BG/NBD is for contractual-free settings) |
| GG model | $\text{Corr}(\text{frequency}, \text{avg\_order\_value}) \approx 0$? |
| MNL | Are any new alternatives very similar to existing ones? (IIA concern) |
| Prophet | Are there structural breaks not encoded as changepoints? |
| DiD | Do pre-trends run parallel? Run a placebo test using only pre-period data |
| k-means | Are features standardised? Is k chosen via elbow/silhouette, not arbitrary? |
| CausalImpact | Are control covariates unaffected by the intervention? Is pre-period MAPE acceptable? |
