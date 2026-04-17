# Lecture 5: Customer Journey Modeling with Markov Chains
## Where Are Customers Going — and How Do You Change It?

---

### Overview

**Business question:** Of your customers currently classified as "At-Risk," what fraction will churn by month 3 if nothing changes? And if you run a re-engagement campaign that improves the At-Risk → Active transition, how does that change the long-run customer distribution?

**What you will be able to do:**
- Multiply a state vector by a transition matrix by hand
- Iterate toward the steady-state distribution
- Identify absorbing states and explain what they imply
- Compute the impact of an intervention on the steady-state
- Explain why the steady-state is not necessarily where you want to be

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Matrix–Vector Multiplication (Row × Column)

A matrix has rows and columns. To multiply a vector v by a matrix P, multiply each element of v by the corresponding element in each column of P and sum.

**Example:** v = [0.7, 0.2, 0.1], P = [[0.8, 0.1, 0.1], [0.3, 0.5, 0.2], [0, 0, 1]]

New state 1 = 0.7×0.8 + 0.2×0.3 + 0.1×0 = 0.56 + 0.06 + 0 = **0.62**
New state 2 = 0.7×0.1 + 0.2×0.5 + 0.1×0 = 0.07 + 0.10 + 0 = **0.17**
New state 3 = 0.7×0.1 + 0.2×0.2 + 0.1×1 = 0.07 + 0.04 + 0.10 = **0.21**

Result: v₁ = [0.62, 0.17, 0.21]. All three values sum to 1. ✓

---

#### Tool 2: Row Sums Must Equal 1

Every row of a valid transition matrix sums to 1. This represents the fact that a customer must be in exactly one state next period.

If any row sum ≠ 1, the matrix is incorrectly specified.

---

### Section 1.2 — Business Motivation

---

#### Shopify's Counterintuitive Approach to Churn

Archie Abrams leads a 600-person growth organization at Shopify. His most surprising insight: Shopify deliberately does not optimize to reduce churn.

Most companies treat churn reduction as a primary growth lever. Shopify reasons differently: their mission is to increase the amount of entrepreneurship on the internet. Most new businesses fail — this is a fact about entrepreneurship, not a product defect. Shopify makes it as easy as possible to start a store, accepts that many merchants will leave when their first business fails, and builds a business model (payment processing revenue, not pure subscription) that makes the successful merchants valuable enough to sustain the entire cohort economics.

This is a Markov chain insight in business language. The states are not just "active" and "churned." They include "first business failed, starting second," "growing successfully," and "enterprise tier." The transition from churned back to active — which most models treat as impossible — is, for Shopify, a designed feature of the journey.

The lesson: before building a transition matrix, you must define states that reflect how customers actually experience your product — not states that are convenient for the analyst.

---

### Section 1.3 — Conceptual Framework

---

#### States, Transitions, and Time Steps

A Markov chain models a system that moves between a fixed set of **states** at discrete **time steps**. The probability of moving from state i to state j depends only on the current state — not on the history of how you got there. This is the **Markov property**.

For customer engagement, typical states might be:
- **Active** (regular engagement)
- **Dormant** (occasional engagement, declining)
- **At-Risk** (minimal engagement, likely to churn soon)
- **Churned** (cancelled or lapsed)

The **transition matrix P** has one row per state. P[i][j] = probability of moving from state i to state j in one period.

---

#### Absorbing States

A state is **absorbing** if, once entered, you cannot leave it. In customer models, "Churned" is typically absorbing — cancelled customers do not spontaneously reactivate. This means: if customers keep flowing into Churned, the system will eventually concentrate all probability mass in Churned. The steady-state has 100% of customers in Churned.

This implies: if Churned is absorbing, retention matters because it determines how quickly the system drains into the absorbing state — not because the steady-state distribution has Active customers.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: One Period of Transition

Starting distribution v₀ = [Active=0.70, Dormant=0.20, Churned=0.10]

Transition matrix P:
| From \ To | Active | Dormant | Churned |
|---|---|---|---|
| Active | 0.80 | 0.15 | 0.05 |
| Dormant | 0.30 | 0.50 | 0.20 |
| Churned | 0.00 | 0.00 | 1.00 |

After one period (v₀ × P):
- New Active = 0.70×0.80 + 0.20×0.30 + 0.10×0 = 0.56 + 0.06 = **0.62**
- New Dormant = 0.70×0.15 + 0.20×0.50 + 0.10×0 = 0.105 + 0.10 = **0.205**
- New Churned = 0.70×0.05 + 0.20×0.20 + 0.10×1 = 0.035 + 0.04 + 0.10 = **0.175**

v₁ = [0.62, 0.205, 0.175]. Sum = 1.00 ✓

Active decreased from 70% to 62%. Churned increased from 10% to 17.5%.

---

#### Part B: The Steady-State Distribution

The steady-state distribution π is the distribution where v × P = v. In other words: after one more period, the distribution does not change. The system has reached equilibrium.

**How to find it numerically:** Repeatedly apply P to any starting distribution. With most matrices (not fully absorbing), the distribution converges.

**The critical insight:** The steady-state is where the current transition dynamics are pointing — not necessarily where you want to be. If current Active% = 75% and the steady-state Active% = 40%, then without any intervention, Active% will decline from 75% toward 40% over time.

**Interventions change the transition matrix.** If a re-engagement campaign increases Dormant → Active from 0.30 to 0.45, a new transition matrix P' applies. The new steady-state under P' will be different — and can be computed by iterating P' to convergence.

> ### 🔍 Deep Dive: Eigenvalues and the Steady State
> For a matrix without absorbing states, the steady-state distribution is the eigenvector corresponding to eigenvalue λ = 1. All Markov chains have at least one eigenvalue equal to 1. The other eigenvalues (< 1 in absolute value) determine how quickly the distribution converges to steady state — larger gaps between 1 and the second-largest eigenvalue mean faster convergence.

---

### Part 1 Checkpoint

1. Transition matrix P: Active=[0.80, 0.15, 0.05], Dormant=[0.30, 0.50, 0.20], Churned=[0, 0, 1]. Starting from v₀=[0.60, 0.30, 0.10], compute v₁.

2. Using v₁ from Q1, compute v₂ = v₁ × P.

3. Is Churned an absorbing state in this matrix? What does that imply for the long-run steady state?

4. Current Active% = 70%. Steady-state Active% = 35%. If nothing changes, will Active% increase or decrease over time?

5. A re-engagement campaign increases Dormant→Active from 0.30 to 0.45. Without computing a new steady state, which direction will the new steady-state Active% move relative to the old one?

---

### Checkpoint Answer Key

**Q1.** v₀ = [0.60, 0.30, 0.10].
New Active = 0.60×0.80 + 0.30×0.30 + 0.10×0 = 0.48 + 0.09 = **0.57**
New Dormant = 0.60×0.15 + 0.30×0.50 + 0.10×0 = 0.09 + 0.15 = **0.24**
New Churned = 0.60×0.05 + 0.30×0.20 + 0.10×1 = 0.03 + 0.06 + 0.10 = **0.19**
v₁ = [0.57, 0.24, 0.19]. Sum = 1.00 ✓

*Common wrong answer:* Multiplying row × row instead of vector × matrix. Always multiply each element of the current distribution by the corresponding entry in each column of P.

**Q2.** v₁ = [0.57, 0.24, 0.19].
New Active = 0.57×0.80 + 0.24×0.30 + 0.19×0 = 0.456 + 0.072 = **0.528**
New Dormant = 0.57×0.15 + 0.24×0.50 + 0.19×0 = 0.0855 + 0.12 = **0.2055**
New Churned = 0.57×0.05 + 0.24×0.20 + 0.19×1 = 0.0285 + 0.048 + 0.19 = **0.2665**
v₂ ≈ [0.528, 0.206, 0.267]. Sum ≈ 1.00 ✓

**Q3.** Yes, Churned is absorbing — its row is [0, 0, 1], meaning once churned, the probability of staying churned is 1. Long-run implication: all customers eventually end up in Churned. The steady-state has 100% in Churned. Retention strategy determines how slowly or quickly the distribution drains into the absorbing state.

*Common wrong answer:* "There is a stable mix of Active/Dormant/Churned in the long run." Only if Churned has non-zero exit probabilities. With an absorbing Churned state, all probability eventually concentrates there.

**Q4.** **Decrease** toward 35%. The steady state is where the dynamics converge. Since current Active (70%) is above the steady state (35%), the system is drifting downward. Without intervention, Active% will decline.

*Common wrong answer:* Increase, because 70% > 35% means we are already above the target. The steady state is not a target — it is an attractor. The system moves toward it, not away from it.

**Q5.** The new steady-state Active% will be **higher** than 35%. Increasing the Dormant→Active flow brings more customers back into Active each period, shifting the equilibrium upward. The Dormant pool replenishes Active rather than flowing mostly toward Churned.

*Common wrong answer:* The steady state does not change because Churned is still absorbing. The steady state with an absorbing Churned always converges to 100% Churned eventually — but the transition matrix change affects how quickly and through which intermediate states the distribution moves. If framing this as a finite-horizon question (e.g., steady state over 24 months), the Active% after 24 months will be higher under the new P.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work Part B together)

**[INSTRUCTOR NOTE]** Students attempt Part A (5 minutes). Reveal and discuss. Then work through Part B (steady-state derivation) together at the board.

---

**Problem Statement**

A mobile gaming company tracks player engagement monthly. The transition count matrix from 12 months of data is:

| From↓ \ To→ | Active | Dormant | Churned |
|---|---|---|---|
| Active | 560 | 140 | 100 |
| Dormant | 180 | 200 | 120 |
| Churned | 0 | 0 | 800 |

**Part A:** Build the transition probability matrix $P$ by normalizing each row. Verify all rows sum to 1.

**Part B:** Compute the Active row of $P^2$: the 2-month transition probabilities starting from Active.

**Part C:** Is Churned an absorbing state? What is the long-run steady-state distribution? (Note: since Churned is absorbing, this is trivially $\pi_{Churned} = 1$. Instead, compute: starting from Active, what fraction of customers are still in a non-Churned state after 1 month? After 2 months? What pattern do you observe?)

---

**Full Solution**

**Part A: Transition Probability Matrix**

Row sums: Active = 800, Dormant = 500, Churned = 800.

$$P = \begin{bmatrix} 560/800 & 140/800 & 100/800 \\ 180/500 & 200/500 & 120/500 \\ 0/800 & 0/800 & 800/800 \end{bmatrix} = \begin{bmatrix} 0.700 & 0.175 & 0.125 \\ 0.360 & 0.400 & 0.240 \\ 0.000 & 0.000 & 1.000 \end{bmatrix}$$

Row sums: $0.700+0.175+0.125 = 1.000$; $0.360+0.400+0.240 = 1.000$; $0+0+1 = 1.000$ ✓

**Part B: Active Row of $P^2$**

$$(P^2)_{A,A} = 0.700 \times 0.700 + 0.175 \times 0.360 + 0.125 \times 0.000 = 0.490 + 0.063 + 0.000 = \mathbf{0.553}$$

$$(P^2)_{A,D} = 0.700 \times 0.175 + 0.175 \times 0.400 + 0.125 \times 0.000 = 0.1225 + 0.070 + 0.000 = \mathbf{0.193}$$

$$(P^2)_{A,C} = 0.700 \times 0.125 + 0.175 \times 0.240 + 0.125 \times 1.000 = 0.0875 + 0.042 + 0.125 = \mathbf{0.255}$$

Check: $0.553 + 0.193 + 0.255 = 1.001 \approx 1.000$ ✓ (rounding)

**Part C: Churn Progression from Active State**

| Months elapsed | P(Active) | P(Dormant) | P(Churned) |
|---|---|---|---|
| 0 | 1.000 | 0.000 | 0.000 |
| 1 | 0.700 | 0.175 | 0.125 |
| 2 | 0.553 | 0.193 | 0.255 |

The probability of being in a non-Churned state (Active + Dormant):
- Month 0: 1.000 (all survive)
- Month 1: 0.700 + 0.175 = 0.875
- Month 2: 0.553 + 0.193 = 0.746

This is exactly the survival function $\hat{S}(t)$ from Lecture 4, computed here via Markov chain dynamics. The connection between survival analysis and Markov chains is not a coincidence — they model the same phenomenon from different angles.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The transition matrix heatmap:** Darker cells indicate higher transition probabilities. Look for:
- Dominant diagonal: most customers stay in their current state (typical)
- Heavy off-diagonal flows toward Churned: dangerous — many customers flowing out
- Unexpected flows (e.g., Churned → Active > 0): may indicate data quality issues or intentional win-back programs

**The steady-state vector:** Represents the long-run customer mix. If the current distribution is better than steady state, expect deterioration over time. If worse, expect improvement. Use this for long-run capacity and revenue planning.

**Simulation output:** Plotting the fraction in each state over 24 simulated months starting from a realistic initial distribution. Look for:
- How quickly does the distribution converge to steady state?
- Is there a transient "danger period" when Churned fraction rises sharply?
- Does Active fraction drop below a threshold that triggers concern?

**Before trusting the agent output:**
1. Verify all rows of $P$ sum to 1 (within floating-point tolerance)
2. Verify the steady-state vector satisfies $\pi P \approx \pi$ (multiply it out and check)
3. If Churned is absorbing, verify $(P)_{Churned, j} = 0$ for all $j \neq$ Churned and $(P)_{Churned, Churned} = 1$
4. Do the transition probabilities make business sense? Active → Churned should be lower than Dormant → Churned

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/customer_states.csv` (customer-month data) + `homework_05.ipynb`

---

#### Part A: Math Questions (no agent required)

Use this transition count matrix for all calculations:

| From↓ \ To→ | New | Active | At-Risk | Churned |
|---|---|---|---|---|
| New | 0 | 350 | 100 | 50 |
| Active | 0 | 480 | 120 | 0 |
| At-Risk | 0 | 200 | 100 | 200 |
| Churned | 0 | 0 | 0 | 600 |

**Q1.** What is the row sum for Active customers?
```python
q1_active_row_sum = None
```
`[ANSWER KEY: 480 + 120 + 0 = 600]`

**Q2.** What is the transition probability $P_{Active, At-Risk}$ (Active → At-Risk)?
```python
q2_p_active_atrisk = None
```
`[ANSWER KEY: 120/600 = 0.2]`

**Q3.** What is the transition probability $P_{At-Risk, Churned}$?
```python
q3_p_atrisk_churned = None
```
`[ANSWER KEY: 200/500 = 0.4]`

**Q4.** Is Churned an absorbing state? Enter True or False.
```python
q4_churned_absorbing = None
```
`[ANSWER KEY: True — the Churned row is [0, 0, 0, 600] → P_Churned,Churned = 1.0]`

**Q5.** Starting from At-Risk, compute $(P^2)_{At-Risk, Churned}$ — the 2-step probability of reaching Churned starting from At-Risk. Use $P_{At-Risk} = [0, 0.40, 0.20, 0.40]$ and the column of Churned transitions $[0.10, 0, 0.40, 1.00]$. (Hint: dot product of the At-Risk row with the Churned column.)

```python
q5_p2_atrisk_churned = None
```
`[ANSWER KEY: 0×0.10 + 0.40×0 + 0.20×0.40 + 0.40×1.00 = 0 + 0 + 0.08 + 0.40 = 0.48]`

**Q6.** Is $(P^2)_{At-Risk, Churned}$ higher or lower than the 1-step probability $P_{At-Risk, Churned} = 0.40$?
```python
# Enter "higher" or "lower"
q6_comparison = None
```
`[ANSWER KEY: "higher" — 0.48 > 0.40, because At-Risk customers who don't immediately churn often flow back to At-Risk before churning in the next step]`

**Q7.** True or False: The steady-state distribution for this 4-state chain (with Churned absorbing) is $\pi = [0, 0, 0, 1]$.
```python
q7_steady_state = None
```
`[ANSWER KEY: True — with an absorbing state, all customers eventually end up in Churned]`

**Q8.** Given the matrix above, which transition represents the biggest "leak" — the transition with the highest probability that moves customers toward Churned?
```python
# Enter the transition as a string, e.g., "Active_to_AtRisk"
q8_biggest_leak = None
```
`[ANSWER KEY: "AtRisk_to_Churned" — probability 0.40, higher than any other transition toward Churned]`

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I have monthly customer engagement data for a subscription service.
customer_states.csv has one row per customer-month with columns:
- customer_id
- month (integer)
- state: one of "new", "active", "at_risk", "dormant", "churned"

Please:
1. Build the 5×5 transition probability matrix by counting consecutive
   month-to-month state changes and normalizing rows.
   Display as a formatted table and a heatmap.
2. Starting from "active", compute and display the state distribution
   after 1, 3, 6, and 12 months (using matrix multiplication P^n).
3. Identify the state with the highest single-period transition probability
   to "churned".
4. Simulate 500 customer journeys starting from "new" for 18 months.
   Plot the average fraction in each state over time.
Use random seed 42.
```

**Q9.** What is the transition probability from "at_risk" to "churned"? Round to 2 decimal places.
```python
q9_p_atrisk_churned = None
```
`[ANSWER KEY: pre-run to determine]`

**Q10.** After 12 months starting from "active", what fraction of customers are still active (not churned)? Round to 2 decimal places.
```python
q10_active_survival_12m = None
```
`[ANSWER KEY: pre-run to determine]`

**Q11.** Which state has the highest transition probability to "churned"?
```python
q11_highest_churn_state = None  # e.g., "dormant", "at_risk", etc.
```
`[ANSWER KEY: pre-run to determine; expected "dormant" or "at_risk"]`

---

#### Part C: Interpretation Questions

**Q12.** The steady-state distribution for a 4-state chain with an absorbing Churned state is always $\pi_{Churned} = 1$. Does this mean all customers will eventually churn? What does it actually tell you, and why is the expected time to reach Churned more useful?
- (a) Yes — it means all customers will eventually churn, which is expected for any subscription
- (b) No — it means some customers will always remain active
- (c) Yes — and this is a problem with the model that should be fixed
- (d) No — it means Churned is not truly absorbing
```python
q12 = None  # "a", "b", "c", or "d"
```
`[ANSWER KEY: "a"]`

**Q13.** The simulation shows a "spike" in the At-Risk fraction at month 3 before it decreases. What business interpretation would you give for this pattern?
- (a) The simulation has a bug — at-risk should decrease monotonically
- (b) New customers enter an at-risk phase around month 3 before either re-engaging or churning
- (c) The Markov property is violated at month 3
- (d) This only happens because the dataset is too small
```python
q13 = None  # "a", "b", "c", or "d"
```
`[ANSWER KEY: "b"]`

**Q14.** You are designing a retention campaign. Based on the transition matrix, which customers should you prioritize targeting: Active customers who might become At-Risk, or At-Risk customers who might churn?
- (a) Active customers — there are more of them
- (b) At-Risk customers — their next-step churn probability is highest, so intervention has the most immediate impact per dollar
- (c) Dormant customers — they have the lowest probability of recovery
- (d) New customers — preventing early churn is always most valuable
```python
q14 = None  # "a", "b", "c", or "d"
```
`[ANSWER KEY: "b" — though a full answer would consider both the churn probability and the cost of targeting; At-Risk has the highest marginal impact per intervention]`

---

### Common Misconceptions

**1. "The steady-state distribution tells us what will happen next month."**
The steady state is the long-run equilibrium — it describes what the distribution converges to over many periods, not what happens in one period. For short-run predictions, use $\pi_0 P^n$ where $\pi_0$ is today's distribution.

**2. "The Markov property means history does not matter at all."**
The Markov property means the transition probabilities depend only on the current state — but the current state itself summarizes the relevant history. A customer who is "At-Risk" has implicitly communicated their recent behavior through their state assignment.

**3. "A larger transition matrix is always more accurate."**
More states can capture finer distinctions but increase estimation error (fewer observations per cell), require more data to estimate reliably, and make the model harder to interpret. A 3–5 state model is often preferable to a 15-state model with sparse data.

**4. "Rows of $P^n$ are computed by multiplying each row element of $P$ by $n$."**
$P^n$ requires actual matrix multiplication ($n-1$ times), not element-wise scaling. The elements of $P^n$ are not simply $n$ times the elements of $P$.

**5. "If the steady state shows 40% Active, I can guarantee 40% Active customers tomorrow."**
The steady state is an asymptotic property. Convergence may take months or years, and if the transition matrix itself changes (due to seasonality, product changes, or campaigns), the system may never actually reach the theoretical steady state.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Counts transitions, normalizes rows → matrix $P$ | Section 1.4A — row normalization |
| Computes state distribution after $n$ months via $\pi_0 P^n$ | Section 1.4B — $P^n$ interpretation |
| Solves $\pi P = \pi$ for steady state | Section 1.4C — linear system derivation |
| Identifies absorbing states | Section 1.4D — $P_{ii} = 1$ |
| Simulation of 500 journeys | Section 2.2 — Monte Carlo approximation to $P^n$ |

**What to verify:**
1. All rows of $P$ sum to 1.0 (within 0.001 tolerance)
2. The steady-state vector satisfies $\pi P \approx \pi$ — multiply it out and check
3. If Churned is absorbing: $(P)_{Churned,Churned} = 1.0$, all other entries in that row = 0
4. Transition probabilities are directionally sensible (At-Risk → Churned > Active → Churned)
