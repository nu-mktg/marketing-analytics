# v2 Math Rescoping Guide
## What changes in each lecture (v1 → v2)

---

### Global rules applied to all lectures

**REMOVE:**
- Any formula requiring integration (∫)
- Fourier series and trigonometric components (sin, cos)
- Formal limit notation (lim h→0)
- Matrix inverse or eigenvalue decomposition
- Formal proof of statistical properties (unbiasedness, consistency)
- Maximum likelihood via calculus (∂L/∂θ = 0)

**KEEP:**
- Core formula stated plainly with exponent rules and algebra only
- Numerical worked examples with real numbers
- Intuition explanations in plain English after each formula
- The result a practitioner would use (even if the derivation is simplified)

**ADD:**
- "What this means" sentence after every formula
- Deep-dive box for the removed derivation (optional, clearly marked)
- Checkpoint answer key at the end (answer + explanation + common wrong answer)

---

### Lecture-by-lecture rescoping

| Lecture | REMOVE | KEEP | DEEP DIVE |
|---|---|---|---|
| L02 Price Elasticity | Calculus FOC for OLS β; formal variance of β̂ | Manual OLS via Σ(x−x̄)(y−ȳ)/Σ(x−x̄)²; log-log interpretation; elasticity arithmetic | FOC derivation: why minimizing SSR gives that exact formula |
| L03 MMM | Bayesian hierarchical estimation; gradient descent notation | Adstock recursion; Hill function evaluation; budget optimization result | Why Hill saturates (marginal response approaching zero) |
| L04 Survival | Formal hazard function as a limit; partial likelihood derivation | KM step formula; hazard ratio interpretation; censoring logic | The partial likelihood and why it estimates β without estimating the baseline |
| L05 Markov | Eigenvalue decomposition for steady state | Transition matrix multiplication; row sums = 1; iterating to steady state numerically | Eigenvalue approach and formal proof of convergence |
| L06 Attribution | Formal Shapley axiom proofs; Markov chain removal effect derivation | n! orderings; marginal contribution arithmetic; efficiency sum | Why Shapley uniquely satisfies efficiency + symmetry + null player |
| L07 Uplift | Formal potential outcomes (SUTVA); double-robust estimator | T-learner formula; ATE; targeting threshold cost/value | Rubin causal model and why random assignment enables unbiasedness |
| L08 CLV | Full BG/NBD likelihood with gamma functions; MLE via numerical optimization | Conjugate update intuition; P(alive) direction reasoning; CLV = E[transactions] × E[spend] | The full BG/NBD likelihood and what r, α, a, b each represent |
| L09 Conjoint | Formal MNL derivation from random utility theory; IIA proof | Utility formula; WTP = β_attr/\|β_price\|; market simulation via softmax | Why logit softmax implies IIA |
| L10 Prophet | Fourier series formula; trigonometric notation | Decomposition concept (trend + seasonality + holidays); changepoints; MAPE | How Fourier terms capture seasonal patterns without trig notation |
