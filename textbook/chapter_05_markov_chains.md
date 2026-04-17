# Chapter 5: Markov Chain Customer Modeling
## Comprehensive Reference Guide

---

### Chapter Overview

Markov chains model customer journeys as movements between a fixed set of engagement
states. This chapter develops the full mathematical framework: transition matrices,
state distributions, steady-state analysis, absorbing states, the fundamental matrix
for absorbing chains, and intervention analysis. Full proofs of convergence and
uniqueness of the stationary distribution are included.

**By the end of this chapter you should be able to:**
- Multiply a state vector by a transition matrix correctly
- Prove that any ergodic chain has a unique stationary distribution
- Compute the fundamental matrix for an absorbing chain
- Interpret steady-state distributions as long-run predictions
- Model an intervention as a change to the transition matrix and quantify its impact
- Explain why state design is more important than estimation method

---

## 5.1 Strategic Frame: What Practitioners Know

### Archie Abrams: Shopify's Counterintuitive Non-Optimization

Archie Abrams leads a 600-person growth organization at Shopify. His position on
churn is genuinely counterintuitive for a subscription business: Shopify deliberately
does not try to minimize churn in the traditional sense.

His reasoning is a Markov chain argument stated without the math. Most new businesses
fail — that is a statistical fact about entrepreneurship, not a product defect. A
merchant who closes their first Shopify store and opens a second one a year later is
not "churned" in any meaningful sense; they are traversing a natural lifecycle.
Shopify's revenue model (significantly weighted toward payment processing rather than
pure subscription fees) means that successful, growing merchants are enormously more
valuable than slightly-less-churny failing merchants.

The practical implication for Markov chain design: the state space matters more than
the estimation method. A state space with only "Active" and "Churned" cannot represent
"First business failed, considering a second." Once you design a state space that
includes re-entry paths, the transition matrix reveals different strategic levers than
a simple two-state model.

His specific operational insight: instead of asking "how do we reduce Churn→Churned
transitions?", Shopify asks "how do we make the product valuable enough that returning
entrepreneurs come back?" The Markov chain analysis quantifies the long-run difference
between these two strategies.

### Hila Qu: Activation as the Critical Transition

Hila Qu built growth teams at GitLab and Acorns before becoming an advisor and
investor. Her focus is on the New User → Active transition — what she calls
"activation." Getting users to their first core value experience is, she argues, the
single highest-leverage growth investment for most products.

The Markov chain formulation makes this quantitative. The transition probability
P(New User → Active) directly determines the long-run share of the population in the
Active state. A 10% improvement in activation probability propagates through the
entire steady-state distribution — increasing the Active share, decreasing the
Dormant share, and ultimately reducing churn. Quantifying this propagation requires
computing the new steady-state distribution under the updated transition matrix.

Her specific warning: activation is frequently confused with sign-up. Sign-up is a
condition for activation but not activation itself. Activation is the moment a user
experiences the product's core value proposition for the first time. Optimizing
sign-up conversion without improving the post-signup experience simply moves the
dropout earlier in the funnel.

### Three Strategic Insights

**Insight 1: State design precedes estimation.**
The most important decision in building a Markov chain customer model is how to define
the states. States should reflect meaningful differences in customer behavior and
retention risk — not what is convenient to measure or what the database schema happens
to track. At a minimum, states should capture: recent engagement (this week), recent
non-engagement (past 1–2 weeks), extended absence (past 3–4 weeks), and churned.
Adding a fifth state for "re-activated" (returned after absence) reveals a set of
transitions most models treat as impossible.

**Insight 2: The steady state is a warning, not a target.**
When the steady-state distribution under current transition probabilities shows 60%
of customers in Churned, this does not mean "aim for 60% churn." It means: if nothing
changes, the system will drift toward 60% churn. The analysis generates urgency, not
a benchmark.

**Insight 3: Interventions change P, not the current state.**
The long-run value of a retention campaign is not measured by how many customers it
moved from At-Risk to Active this month — it is measured by how the new transition
matrix P' differs from the old P in its steady-state distribution. One month of
intervention at scale may move 1,000 customers. A persistent change to the transition
matrix affects every customer for all future months.

---

## 5.2 Mathematical Prerequisites

### 5.2.1 Matrix Operations

A **row vector** v = [v₁, v₂, ..., vₙ] represents a probability distribution over
n states. An n×n **matrix** P has entry P_{ij} in row i, column j.

**Matrix-vector multiplication (row vector × matrix):**
$$(v \cdot P)_j = \sum_{i=1}^n v_i P_{ij}$$

Component j of the result = dot product of v with column j of P.

**Matrix-matrix multiplication:**
$$(AB)_{ij} = \sum_{k=1}^n A_{ik}B_{kj}$$

**Identity matrix** I has I_{ij} = 1 if i=j, 0 otherwise. AI = IA = A.

**Matrix inverse:** $A^{-1}$ satisfies $A A^{-1} = A^{-1} A = I$. Exists iff A is
non-singular (det A ≠ 0).

### 5.2.2 Eigenvalues and Eigenvectors

For square matrix P, $\lambda$ is an **eigenvalue** and x a (right) **eigenvector** if:
$$Px = \lambda x, \quad x \neq 0$$

**Left eigenvectors:** $\pi P = \lambda \pi$ (eigenvectors of P^T).

The stationary distribution of a Markov chain is the left eigenvector of P for
eigenvalue λ = 1, normalized to sum to 1.

*Reference: Wikipedia, "Eigenvalues and eigenvectors" —
en.wikipedia.org/wiki/Eigenvalues_and_eigenvectors*

---

## 5.3 Markov Chains: Formal Definition

### Definition

A **discrete-time Markov chain** is a sequence of random variables $\{X_t\}_{t=0}^\infty$
taking values in a finite state space $S = \{1, 2, \ldots, n\}$ satisfying:

$$P(X_{t+1} = j \mid X_t = i, X_{t-1} = i_{t-1}, \ldots, X_0 = i_0) = P_{ij}$$

The **Markov property**: the future state depends only on the current state, not the
history. The **transition probabilities** $P_{ij}$ are time-homogeneous (constant).

### Transition Matrix Properties

1. $P_{ij} \geq 0$ for all i, j (probabilities)
2. $\sum_{j=1}^n P_{ij} = 1$ for all i (row stochasticity — each customer must
   be in some state next period)
3. P is called a **stochastic matrix** or **row-stochastic matrix**

*Reference: Wikipedia, "Markov chain" — en.wikipedia.org/wiki/Markov_chain*
*Reference: Grinstead, C. M. and Snell, J. L. *Introduction to Probability*, Chapter 11.
Free PDF: math.dartmouth.edu/~prob/prob/prob.pdf*

### State Distribution Dynamics

If $\pi_t = [\pi_t(1), \ldots, \pi_t(n)]$ is the distribution at time t, then:
$$\pi_{t+1} = \pi_t \cdot P$$
$$\pi_{t+k} = \pi_t \cdot P^k$$

### Worked Numerical Example

**4-state chain:** Active (A), Dormant (D), At-Risk (R), Churned (C)

$$P = \begin{pmatrix} 0.80 & 0.15 & 0.04 & 0.01 \\ 0.25 & 0.55 & 0.15 & 0.05 \\ 0.08 & 0.18 & 0.50 & 0.24 \\ 0.00 & 0.00 & 0.00 & 1.00 \end{pmatrix}$$

Row sums: 0.80+0.15+0.04+0.01 = 1.00 ✓; 0.25+0.55+0.15+0.05 = 1.00 ✓;
0.08+0.18+0.50+0.24 = 1.00 ✓; 0.00+0.00+0.00+1.00 = 1.00 ✓

Initial distribution: π₀ = [0.55, 0.28, 0.12, 0.05]

**After one period (π₁ = π₀ × P):**

$$\pi_1(A) = 0.55(0.80) + 0.28(0.25) + 0.12(0.08) + 0.05(0) = 0.440 + 0.070 + 0.010 = 0.520$$
$$\pi_1(D) = 0.55(0.15) + 0.28(0.55) + 0.12(0.18) + 0.05(0) = 0.0825 + 0.154 + 0.0216 = 0.258$$
$$\pi_1(R) = 0.55(0.04) + 0.28(0.15) + 0.12(0.50) + 0.05(0) = 0.022 + 0.042 + 0.060 = 0.124$$
$$\pi_1(C) = 0.55(0.01) + 0.28(0.05) + 0.12(0.24) + 0.05(1.00) = 0.0055 + 0.014 + 0.0288 + 0.05 = 0.098$$

Sum = 0.520 + 0.258 + 0.124 + 0.098 = 1.000 ✓

Active fell from 55% to 52%, Churned rose from 5% to 9.8%.

> **Teaching note:** Write this calculation on the board slowly, computing one
> element at a time. The most common error is doing column × column instead of
> row × column. Draw an arrow from the π₀ vector horizontally and from each
> column of P vertically, making the dot-product structure visible. Budget 12 minutes.

---

## 5.4 Classification of States

### Absorbing States

State i is **absorbing** if $P_{ii} = 1$ (once entered, cannot leave).

In customer models, "Churned" is typically absorbing. The row of P for Churned must
be [0, 0, ..., 0, 1] — the customer stays churned with probability 1.

**Implication:** With an absorbing Churned state, the long-run steady state puts 100%
of probability mass on Churned. All customers eventually churn. The interesting
quantity is not the steady state but the **time to absorption** — how quickly
customers flow through transient states into Churned.

### Transient vs. Recurrent States

- **Transient state:** With positive probability, the chain never returns after leaving.
  Active, Dormant, At-Risk are transient when Churned is absorbing.
- **Recurrent state:** The chain returns with probability 1 after every departure.
  Absorbing states are trivially recurrent.
- **Ergodic state:** Recurrent, positive (non-null), and aperiodic.

*Reference: Wikipedia, "Recurrent state" —
en.wikipedia.org/wiki/Markov_chain#Classification_of_states*

---

## 5.5 Steady-State Distribution

### Definition and Interpretation

A distribution $\pi = [\pi_1, \ldots, \pi_n]$ is a **stationary distribution** if:
$$\pi P = \pi \quad \text{and} \quad \sum_i \pi_i = 1$$

**Economic interpretation:** If the chain runs long enough in a system without
absorbing states, the distribution across states converges to π regardless of where
it started. This is the long-run equilibrium — the fraction of time each state is
occupied, equivalently the fraction of customers in each state when the system
reaches steady state.

### Existence and Uniqueness: Perron-Frobenius Theorem

For an **ergodic** chain (irreducible + aperiodic):
1. There exists a unique stationary distribution π
2. For any initial distribution π₀: $\lim_{t \to \infty} \pi_0 P^t = \pi$
3. The convergence rate is governed by the second-largest eigenvalue |λ₂|

The Perron-Frobenius theorem guarantees this: for a positive stochastic matrix P
(all entries > 0), λ = 1 is a simple eigenvalue (multiplicity 1), all other
eigenvalues satisfy |λ| < 1, and the corresponding left eigenvector (normalized to
sum to 1) is unique and has all positive entries.

*Reference: Wikipedia, "Perron-Frobenius theorem" —
en.wikipedia.org/wiki/Perron%E2%80%93Frobenius_theorem*

### Computing the Steady State

**Method 1 — Power iteration:**
Start with any π₀. Repeatedly compute πₜ₊₁ = πₜP until ||πₜ₊₁ − πₜ||₁ < ε.
Convergence guaranteed for ergodic chains.

**Method 2 — Linear system:**
The steady-state satisfies πP = π, equivalently π(P − I) = 0.
Combined with Σπᵢ = 1, this gives an (n+1) × n linear system. Solve using least
squares or by replacing one equation with the normalization constraint:

$$\begin{bmatrix} (P^\top - I) \\ \mathbf{1}^\top \end{bmatrix} \pi^\top = \begin{bmatrix} \mathbf{0} \\ 1 \end{bmatrix}$$

**Method 3 — Eigendecomposition (Python):**
```python
import numpy as np
eigenvalues, eigenvectors = np.linalg.eig(P.T)
idx = np.argmin(np.abs(eigenvalues - 1))
pi = np.real(eigenvectors[:, idx])
pi = pi / pi.sum()  # normalize
```

---

## 5.6 Absorbing Chains: Fundamental Matrix

For chains with absorbing states, we analyze time until absorption using the
**fundamental matrix**.

### Partitioning the Transition Matrix

Partition the states into **transient** (T) and **absorbing** (A):
$$P = \begin{pmatrix} Q & R \\ 0 & I \end{pmatrix}$$

where Q (|T|×|T|) = transition probabilities among transient states,
R (|T|×|A|) = transition probabilities from transient to absorbing states,
0 = (all-zero matrix, absorbing states never leave),
I = identity (absorbing states stay absorbed).

### The Fundamental Matrix N

$$N = (I - Q)^{-1} = \sum_{t=0}^{\infty} Q^t$$

Entry $N_{ij}$ = expected number of periods spent in transient state j, starting
from transient state i, before absorption.

**Why the series converges:** Since all states in Q are transient, the spectral
radius of Q is strictly less than 1 (all eigenvalues of Q have |λ| < 1). Therefore
$\sum_{t=0}^\infty Q^t = (I-Q)^{-1}$ converges.

**Expected absorption time** from state i:
$$t_i = \sum_j N_{ij} = (N \mathbf{1})_i$$

### Worked Example

3 transient states (Active, Dormant, At-Risk), 1 absorbing (Churned):

$$Q = \begin{pmatrix} 0.80 & 0.15 & 0.04 \\ 0.25 & 0.55 & 0.15 \\ 0.08 & 0.18 & 0.50 \end{pmatrix}$$

$$I - Q = \begin{pmatrix} 0.20 & -0.15 & -0.04 \\ -0.25 & 0.45 & -0.15 \\ -0.08 & -0.18 & 0.50 \end{pmatrix}$$

$(I-Q)^{-1}$ computed numerically. Row sums of N give expected months until churn
from each starting state — quantifying the difference in expected lifetime between
Active, Dormant, and At-Risk customers.

*Reference: Wikipedia, "Absorbing Markov chain" —
en.wikipedia.org/wiki/Absorbing_Markov_chain*
*Reference: Grinstead & Snell, Chapter 11 (Absorbing chains) — free PDF above*

---

## 5.7 Intervention Analysis

### How a Campaign Changes P

A re-engagement campaign targeting Dormant customers aims to increase the
Dormant → Active transition probability. Suppose P_{D→A} increases from 0.25 to 0.38.
The updated matrix P' has only this one entry changed (all row sums must still equal 1,
so another entry in the Dormant row must decrease correspondingly).

**Quantifying the long-run impact:**
1. Compute steady-state π under old P (if no absorbing states) or expected absorption
   time N under old P
2. Compute steady-state π' under new P'
3. Difference π'(Active) − π(Active) = long-run increase in Active %

**Finite-horizon impact:**
Compute πₜ = π₀ × Pᵗ and π'ₜ = π₀ × P'ᵗ for t = 1, ..., 24.
Plot the trajectory of Active % under both matrices. The area between the curves
quantifies the cumulative benefit of the campaign over the forecast horizon.

---

## 5.8 Full Proof Appendix

### Proof A: Row-Stochastic Matrix Always Has Eigenvalue 1

For row-stochastic P: $P \mathbf{1} = \mathbf{1}$ (the all-ones vector is a right
eigenvector for eigenvalue 1 by the row-sum property). Therefore λ = 1 is always
an eigenvalue of P.

### Proof B: Stationary Distribution Satisfies π = πP

From the definition $\pi P = \pi$, the stationary distribution is the left eigenvector
of P for eigenvalue 1. Every ergodic chain has exactly one such eigenvector (normalized
to sum to 1) by the Perron-Frobenius theorem.

### Proof C: Power Iteration Converges to Unique Stationary Distribution

For ergodic P, all eigenvalues except λ₁ = 1 satisfy |λₖ| < 1 for k ≥ 2.
Any distribution π₀ can be written in the eigenspace of P's transpose:
$\pi_0 = c_1 \pi + \sum_{k\geq 2} c_k v_k$ where v_k are left eigenvectors.

After t steps: $\pi_0 P^t = c_1 \pi \lambda_1^t + \sum_{k\geq 2} c_k v_k \lambda_k^t
= c_1 \pi + \sum_{k\geq 2} c_k v_k \lambda_k^t$

As t → ∞: $\lambda_k^t \to 0$ for k ≥ 2, leaving only $c_1 \pi$. Normalization
gives π₀Pᵗ → π. QED.

The **mixing time** — how many steps until the chain is close to π — is approximately
$t_{mix} \approx 1/(1 - |\lambda_2|)$ where λ₂ is the second-largest eigenvalue.

---

## 5.9 Verification Checklist

| Check | How to verify |
|---|---|
| All row sums equal 1? | Sum each row; should be exactly 1 |
| Matrix × vector: row of state vector × column of P? | Not column × column |
| Absorbing state: P_{ii}=1, all others 0? | Check the Churned row |
| Steady state satisfies πP = π? | Multiply and check equality element-wise |
| Sum of π = 1? | After normalization step |
| Fundamental matrix N = (I−Q)⁻¹? | Verify N(I−Q) = I numerically |
| Expected absorption time = row sums of N? | Check one row by hand |
| Intervention: new P has row sums = 1? | After changing one entry, verify |

---

## 5.10 Chapter Bibliography

1. Wikipedia, "Markov chain" — en.wikipedia.org/wiki/Markov_chain
2. Wikipedia, "Absorbing Markov chain" — en.wikipedia.org/wiki/Absorbing_Markov_chain
3. Wikipedia, "Perron-Frobenius theorem" — en.wikipedia.org/wiki/Perron%E2%80%93Frobenius_theorem
4. Wikipedia, "Stochastic matrix" — en.wikipedia.org/wiki/Stochastic_matrix
5. Grinstead, C. M. and Snell, J. L. *Introduction to Probability*, 2nd ed. Chapters 11–12.
   Free PDF: math.dartmouth.edu/~prob/prob/prob.pdf
6. MIT OpenCourseWare, 6.262 Discrete Stochastic Processes — ocw.mit.edu
7. Norris, J. R. *Markov Chains*. Cambridge University Press. Select chapters available online.
8. Stewart, W. J. *Introduction to the Numerical Solution of Markov Chains*. Princeton University Press.
   (For numerical methods — not free, but widely available in libraries.)

**Lenny's Podcast episode references:**
- Archie Abrams episode on growth at Shopify — Lenny's Podcast
- Hila Qu episode on activation and growth — Lenny's Podcast
