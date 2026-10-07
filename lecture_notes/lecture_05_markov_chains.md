# Lecture 5: Customer Journey Modeling with Markov Chains
## Where Are Customers Going — and How Do You Change It?

---

### Overview

**Business question:** Of your customers currently classified as "At-Risk," what fraction will churn by month 3 if nothing changes? And if you run a re-engagement campaign that improves the At-Risk → Active transition, how does that change the long-run customer distribution?

**What you will be able to do:**
- Multiply a state vector by a transition matrix by hand
- Iterate toward the steady-state distribution
- Identify absorbing states and explain what they imply
- Compute the expected time to absorption from each state, and use it to price a retention campaign
- Work out what an intervention to the transition matrix does — and does not — change about the
  long run
- Explain why the steady-state is not necessarily where you want to be

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Matrix–Vector Multiplication (Row × Column)

**How to read P.** P is a stack of **row vectors**, one row per state *today*. Row *i* says where
the customers who are in state *i* now go next period, so the entry in row *i*, column *j* is
P(i → j), the probability of moving from state *i* to state *j*. Rows are "from" and columns are "to".

**How to read v.** v is a **row vector** with one entry per state: the share of customers in each
state today. Because it is a row, it multiplies P **from the left**, written v × P (or vP).

**Example:**

$$v = \begin{pmatrix} 0.5 & 0.3 & 0.2 \end{pmatrix}, \qquad
P = \begin{pmatrix} 0.6 & 0.3 & 0.1 \\ 0.2 & 0.5 & 0.3 \\ 0 & 0 & 1 \end{pmatrix}
\begin{matrix} \leftarrow \text{from state 1} \\ \leftarrow \text{from state 2} \\ \leftarrow \text{from state 3} \end{matrix}$$

Entry *j* of the result is v times **column** *j* of P, entry by entry, then summed:

$$\begin{aligned}
(vP)_1 &= 0.5 \times 0.6 + 0.3 \times 0.2 + 0.2 \times 0 = 0.30 + 0.06 + 0 = \mathbf{0.36} \\
(vP)_2 &= 0.5 \times 0.3 + 0.3 \times 0.5 + 0.2 \times 0 = 0.15 + 0.15 + 0 = \mathbf{0.30} \\
(vP)_3 &= 0.5 \times 0.1 + 0.3 \times 0.3 + 0.2 \times 1 = 0.05 + 0.09 + 0.20 = \mathbf{0.34}
\end{aligned}$$

$$v_1 = vP = \begin{pmatrix} 0.36 & 0.30 & 0.34 \end{pmatrix}$$

All three values sum to 1 ✓. The result is again a row vector: next period's share in each state.

**Why the recipe works.** It is the law of total probability. To be in state *j* next period, a
customer must be in *some* state *i* now and then move from *i* to *j*. So you add up the
"start in *i*, then move to *j*" probabilities over every *i*:
P(next = j) = Σᵢ P(now = i) · P(i → j). Each line of the calculation above is that sum for one *j*: the
entries of v are the P(now = i), and column *j* of P holds the P(i → j) for every *i*.

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
- **At-Risk** (declining engagement)
- **Churned** (cancelled or lapsed)

Some teams split the middle ground into two declining-engagement states; this lecture and the
homework use three. In the homework dataset and notebook the code label is `At_Risk`.

The **transition matrix P** has one row per state. P[i][j] = probability of moving from state i to state j in one period.

---

#### Absorbing States

A state is **absorbing** if, once entered, you cannot leave it. In customer models, "Churned" is typically absorbing — cancelled customers do not spontaneously reactivate. This means: if customers keep flowing into Churned, the system will eventually concentrate all probability mass in Churned. The steady-state has 100% of customers in Churned.

This implies: if Churned is absorbing, retention matters because it determines how quickly the system drains into the absorbing state — not because the steady-state distribution has Active customers.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: One Period of Transition

Starting distribution v₀ = [Active=0.85, At-Risk=0.10, Churned=0.05]

Transition matrix P:

| From \ To | Active | At-Risk | Churned |
|---|---|---|---|
| Active | 0.90 | 0.08 | 0.02 |
| At-Risk | 0.20 | 0.40 | 0.40 |
| Churned | 0.00 | 0.00 | 1.00 |

![State diagram of Section 1.4A's transition matrix. Three circles: Active, At-Risk, Churned. Active has a self-loop of 0.90, an arrow of 0.08 to At-Risk and an arrow of 0.02 to Churned. At-Risk has a self-loop of 0.40, an arrow of 0.20 back to Active and an arrow of 0.40 to Churned. Churned has a self-loop of 1.00 and no arrow out, marked absorbing.](figures/lecture_05_state_diagram.png)

After one period (v₀ × P):
- New Active = 0.85×0.90 + 0.10×0.20 + 0.05×0 = 0.765 + 0.02 = **0.785**
- New At-Risk = 0.85×0.08 + 0.10×0.40 + 0.05×0 = 0.068 + 0.04 = **0.108**
- New Churned = 0.85×0.02 + 0.10×0.40 + 0.05×1 = 0.017 + 0.04 + 0.05 = **0.107**

v₁ = [0.785, 0.108, 0.107]. Sum = 1.00 ✓

Active decreased from 85% to 78.5%. Churned increased from 5% to 10.7%.

**Several periods at once: $P^n$.** Two periods ahead is v₂ = v₁ × P = v₀ × P × P. The product P × P
is written $P^2$. Each of its entries is a row of P times a column of P, the same row-times-column
recipe as Tool 1, and row *i* of $P^2$ is the two-period distribution of a customer who starts in
state *i*. In general, $P^n$ is P multiplied by itself *n* times and vₙ = v₀ × $P^n$. Part D's Step 2
works one row of $P^2$ out by hand. (Multiplying the vector by P *n* times gives the same vₙ, and is
how the homework asks you to project.)

---

#### Part B: The Steady-State Distribution

The steady-state distribution π is the distribution that one more period leaves unchanged: π × P = π, written πP = π in code and textbooks. We keep the letter **v** for the distribution at a given time (v₀ today, v₁ next period) and **π** only for the steady state. The system has reached equilibrium.

**How to find it numerically:** Repeatedly apply P to any starting distribution until it stops changing. Where it settles depends on whether the chain has an **absorbing state** — a state with a 1 on its own diagonal, so nobody who enters it ever leaves.

**The critical insight:** The steady-state is where the current transition dynamics are pointing — not necessarily where you want to be. **If no state is absorbing** — say some churned customers are won back each month — the chain settles at a mix of all three states. If that mix has 40% Active while 75% are Active today, then without any intervention Active% will decline from 75% toward 40% over time. **If Churned is absorbing, as in our P,** the same logic gives a starker answer: the steady state is 100% Churned, and Active% declines toward 0. The next paragraph shows what that means for interventions.

![Line chart over 36 months starting from v0 = 0.85 Active, 0.10 At-Risk, 0.05 Churned under Section 1.4A's P. Active falls steadily from 0.85 (0.785 after one month) to about 6% at month 36; At-Risk edges up to about 0.11 then falls to about 1%; Churned rises every month from 0.05 to about 93%. A reference line at 100% is labelled steady state: 100% Churned, reached only in the limit.](figures/lecture_05_distribution_drain.png)

**Interventions change the transition matrix — but check what that actually moves.** If a
re-engagement campaign increases At-Risk → Active from 0.20 to 0.35, a new matrix P' applies, and you
find its steady state by iterating P' to convergence. P' is row 2 re-balanced — At-Risk → Active rises
0.20 → 0.35 and At-Risk → At-Risk falls 0.40 → 0.25, so the row still sums to 1. Do that and the result
is worth pausing on:

| | steady state | Active after 12 periods (from v₀ = [0.45, 0.40, 0.15]) |
|---|---|---|
| P (At-Risk→Active = 0.20) | [0, 0, 1] | 0.239 |
| P' (At-Risk→Active = 0.35) | [0, 0, 1] | 0.295 |

**The steady state does not budge.** As long as Churned is absorbing, *every* path leads there
eventually, so no change to the Active and At-Risk rows can alter the destination — it can only change
**how long the journey takes**. The campaign is worth running, and the 12-period Active share is where
you see its value; the steady state is simply the wrong instrument for measuring it.

**What would move the destination:** a *win-back* flow — making Churned non-absorbing by giving it a
non-zero Churned → Active probability. That is a qualitatively different intervention from improving a
transition among your surviving customers, and this is the distinction the steady state is actually
sensitive to.

> ### 🔍 Deep Dive: Eigenvalues and the Steady State
> For a matrix without absorbing states, the steady-state distribution is the eigenvector corresponding to eigenvalue λ = 1. All Markov chains have at least one eigenvalue equal to 1. The other eigenvalues (< 1 in absolute value) determine how quickly the distribution converges to steady state — larger gaps between 1 and the second-largest eigenvalue mean faster convergence.

---

#### Part C: Expected Time to Absorption

Part B ended on a promise it did not keep: when Churned is absorbing the steady state is always
`[0, 0, 1]`, so the only thing an intervention can move is **how long the journey takes** — and we
had no number for that. This is the number.

**Expected time to absorption** from state *i* is the average number of periods a customer starting
in *i* takes to reach the absorbing state. With Churned absorbing and months as the time step, this
is the customer's **expected remaining lifetime in months**, and it is the quantity an
absorbing-chain analysis actually reports.

**How to compute it: first-step analysis.** Write $t_i$ for the expected time to absorption starting
from state *i*. One period always elapses; after it, you are in some state *j* with probability
$P_{ij}$ and face $t_j$ more periods (with $t_{\text{Churned}} = 0$, since you have arrived). So:

$$t_i = 1 + \sum_{j \text{ not absorbing}} P_{ij}\, t_j$$

That is one equation per transient state. Two transient states → two equations → solve the pair.

**Worked example** with the lecture's P (Active = [0.90, 0.08, 0.02], At-Risk = [0.20, 0.40, 0.40]):

$$t_A = 1 + 0.90\,t_A + 0.08\,t_R \qquad\Longrightarrow\qquad 0.10\,t_A - 0.08\,t_R = 1$$
$$t_R = 1 + 0.20\,t_A + 0.40\,t_R \qquad\Longrightarrow\qquad -0.20\,t_A + 0.60\,t_R = 1$$

Solving: $t_A = 0.68 / 0.044 \approx \mathbf{15.5}$ months, $t_R = 0.30 / 0.044 \approx \mathbf{6.8}$
months. **A currently-Active customer is worth about 15.5 more months; an At-Risk one about 6.8.**
The gap is the whole argument for triaging retention effort toward the declining-engagement
state — there is less time left in which to act.

**Sanity check it every time:** every $t_i$ must be **positive**, and a healthier state must have the
**larger** $t_i$. If At-Risk comes out above Active, the fitted matrix is letting At-Risk customers
return to Active too readily, or the rows are transposed.

**What the campaign actually bought.** Re-run the same two equations on P' (At-Risk → Active = 0.35,
At-Risk → At-Risk = 0.25): $t_A = 0.83/0.047 = \mathbf{17.7}$ months and
$t_R = 0.45/0.047 = \mathbf{9.6}$ months. The destination is unchanged — it always was — but an
Active customer now takes about **2.2 months longer** to get there, and an At-Risk one takes
**2.8 months** longer. *That* is the campaign's effect, stated in a unit finance will accept, and it is invisible in
the steady state.

**One caveat before that number goes to finance.** Everything above *assumes* the campaign moves
At-Risk → Active from 0.20 to 0.35. A transition matrix fitted to past data describes what customers
did; it cannot show what a new campaign will cause them to do. To show that the campaign itself
produces the rise, compare customers who received it with a randomly held-out group who did not, the
same logic as the A/B test in Module 1.

> ### 🔍 Deep Dive: The Fundamental Matrix
> The same calculation in matrix form, which is what code will hand you. Strip the absorbing row and
> column out of P and call the remaining transient block **Q**. Rows are still "from" and columns
> "to", now over Active and At-Risk only:
>
> $$Q = \begin{pmatrix} 0.90 & 0.08 \\ 0.20 & 0.40 \end{pmatrix}$$
>
> Then the **fundamental matrix** is $N = (I - Q)^{-1}$, and the vector of expected times is
> $\mathbf{t} = N\mathbf{1}$ — row sums of N. For the P above,
>
> $$N = \begin{pmatrix} 13.64 & 1.82 \\ 4.55 & 2.27 \end{pmatrix}$$
>
> whose row sums are 15.45 and 6.82: the same two answers.
> $N_{ij}$ has its own reading — the expected number of periods spent in state *j* before absorption,
> starting from *i* — so $N_{AA} = 13.64$ says an Active customer spends about 13.6 of their 15.5
> remaining months Active. In `numpy`: `t = numpy.linalg.inv(numpy.eye(2) - Q) @ numpy.ones(2)`.
> Why the inverse appears: $\mathbf{t} = \mathbf{1} + Q\mathbf{t}$ is exactly the system above
> written in one line, and solving it for $\mathbf{t}$ gives $(I - Q)^{-1}\mathbf{1}$.

---

#### Part D: One Worked Example, Start to Finish

*For working through on your own, with every step shown. It covers each calculation the homework asks
for, on a **different** chain: copy the method, not the numbers.*

**The setting.** A meal-kit subscription, measured monthly, with three states and Churned absorbing:

| From \ To | Active | At-Risk | Churned |
|---|---|---|---|
| Active | 0.55 | 0.25 | 0.20 |
| At-Risk | 0.10 | 0.50 | 0.40 |
| Churned | 0 | 0 | 1 |

Today's distribution is v₀ = [Active 0.40, At-Risk 0.35, Churned 0.25].

**Step 0 — check that P is valid.** Row sums: 0.55 + 0.25 + 0.20 = 1.00; 0.10 + 0.50 + 0.40 = 1.00;
0 + 0 + 1 = 1.00 ✓. No entry is negative ✓.

**Step 1 — one month ahead, for a customer who is Active today.** Their distribution is [1, 0, 0].
Multiplying by P just picks out the Active row: next month they are **Active 0.55, At-Risk 0.25,
Churned 0.20**. A row of P *is* the one-month forecast for one customer in that state.

**Step 2 — two months ahead, for the same customer.** Multiply [0.55, 0.25, 0.20] by P once more:
- Active = 0.55×0.55 + 0.25×0.10 + 0.20×0 = 0.3025 + 0.025 = **0.3275**
- At-Risk = 0.55×0.25 + 0.25×0.50 + 0.20×0 = 0.1375 + 0.125 = **0.2625**
- Churned = 0.55×0.20 + 0.25×0.40 + 0.20×1 = 0.11 + 0.10 + 0.20 = **0.41**

Sum = 1.00 ✓. The two-month Active probability (0.3275) is **not** 0.55 × 0.55 = 0.3025. The extra
0.025 is customers who slip to At-Risk in month 1 and come back in month 2, and squaring the diagonal
entry misses them.

**Step 3 — find the absorbing state.** Churned's row is [0, 0, 1]: a 1 on its own diagonal and nothing
going out, so Churned is absorbing. Active and At-Risk each have a path into it (0.20 and 0.40 a month),
so the steady state is **[0, 0, 1] — 100% Churned**. That is where this P leads if it never changes. It
says nothing about *how soon*, and Steps 5 and 6 are the numbers that answer that.

**Step 4 — next month's churned fraction for the whole base.** Multiply each share of v₀ by its
probability of being Churned next month (the Churned column of P), then add:
0.40×0.20 + 0.35×0.40 + 0.25×1 = 0.08 + 0.14 + 0.25 = **0.47**. Do not drop the last term: the 25%
already churned stay churned. Only 0.08 + 0.14 = 0.22 of the 0.47 are *new* churners. The other two
entries of v₁ come from the same multiplication: Active = 0.40×0.55 + 0.35×0.10 = 0.22 + 0.035 =
**0.255**, and At-Risk = 0.40×0.25 + 0.35×0.50 = 0.10 + 0.175 = **0.275**. So v₁ = [0.255, 0.275, 0.47],
and the sum is 1.00 ✓.

**Step 5 — expected months until churn.** The counting convention is the one Part C uses (and the
homework uses): $t_i$ counts **the current month as month 1** and stops at the month the customer
enters Churned, which is not counted. First-step analysis gives one equation per transient state:

$$t_A = 1 + 0.55\,t_A + 0.25\,t_R \qquad\Longrightarrow\qquad 0.45\,t_A - 0.25\,t_R = 1$$
$$t_R = 1 + 0.10\,t_A + 0.50\,t_R \qquad\Longrightarrow\qquad -0.10\,t_A + 0.50\,t_R = 1$$

The matrix form gives the same answer and is what code returns. The transient block (rows "from",
columns "to", over Active and At-Risk) and I − Q are

$$Q = \begin{pmatrix} 0.55 & 0.25 \\ 0.10 & 0.50 \end{pmatrix}, \qquad
I - Q = \begin{pmatrix} 0.45 & -0.25 \\ -0.10 & 0.50 \end{pmatrix}$$

whose determinant is 0.45×0.50 − 0.25×0.10 = 0.225 − 0.025 = 0.200. Invert a 2×2 matrix by swapping
the diagonal, negating the off-diagonal, and dividing by the determinant:

$$N = (I - Q)^{-1} = \frac{1}{0.200}\begin{pmatrix} 0.50 & 0.25 \\ 0.10 & 0.45 \end{pmatrix}
= \begin{pmatrix} \mathbf{2.50} & \mathbf{1.25} \\ \mathbf{0.50} & \mathbf{2.25} \end{pmatrix}$$

The row sums are the expected times: $t_A$ = 2.50 + 1.25 = **3.75 months** and $t_R$ = 0.50 + 2.25 =
**2.75 months**. Sanity check: both are positive, and the healthier state has the longer time ✓. N's
entries have their own reading. $N_{AA}$ = 2.50 says a customer who is Active today spends, on average,
2.5 of their 3.75 remaining months Active and the other 1.25 At-Risk.

**Step 6 — project the whole base forward.** Repeat v_{k+1} = v_k × P. For month 2, start from v₁:
Active = 0.255×0.55 + 0.275×0.10 = 0.14025 + 0.0275 = 0.16775; At-Risk = 0.255×0.25 + 0.275×0.50 =
0.06375 + 0.1375 = 0.20125; Churned = 0.255×0.20 + 0.275×0.40 + 0.47×1 = 0.051 + 0.11 + 0.47 = 0.631.

| Month | Active | At-Risk | Churned |
|---|---|---|---|
| 0 (today) | 40.0% | 35.0% | 25.0% |
| 1 | 25.5% | 27.5% | 47.0% |
| 2 | 16.8% | 20.1% | 63.1% |
| 3 | 11.2% | 14.3% | 74.5% |
| 6 | 3.5% | 4.7% | 91.7% |
| 12 | 0.4% | 0.5% | 99.1% |

In code this is `v = v0` followed by twelve rounds of `v = v @ P`. It is deterministic, so there is
nothing to simulate and no seed to choose. The table and Step 5 are two views of one fact: a typical
customer here is gone within about four months, so by month 12 almost nobody is left. A retention
change would show up as a slower table and a larger $t_A$, and **not** in the steady state, which stays
at 100% Churned for any P whose Churned row is [0, 0, 1].

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

1. Transition matrix P: Active=[0.90, 0.08, 0.02], At-Risk=[0.20, 0.40, 0.40], Churned=[0, 0, 1]. Starting from v₀=[0.45, 0.40, 0.15], compute v₁.

2. Using v₁ from Q1, compute v₂ = v₁ × P.

3. Is Churned an absorbing state in this matrix? What does that imply for the long-run steady state?

4. A *different* company's chain has **no absorbing state** — some churned customers come back each month. Its current Active% = 70% and its steady-state Active% = 35%. If nothing changes, will Active% increase or decrease over time?

5. A re-engagement campaign increases At-Risk→Active from 0.20 to 0.35, with Churned still absorbing. What happens to the steady-state Active%, and what does the campaign actually change?

---

### Checkpoint Answer Key

**Q1.** v₀ = [0.45, 0.40, 0.15].
New Active = 0.45×0.90 + 0.40×0.20 + 0.15×0 = 0.405 + 0.08 = **0.485**
New At-Risk = 0.45×0.08 + 0.40×0.40 + 0.15×0 = 0.036 + 0.16 = **0.196**
New Churned = 0.45×0.02 + 0.40×0.40 + 0.15×1 = 0.009 + 0.16 + 0.15 = **0.319**
v₁ = [0.485, 0.196, 0.319]. Sum = 1.00 ✓

*Common wrong answer:* Multiplying row × row instead of vector × matrix. Always multiply each element of the current distribution by the corresponding entry in each column of P.

**Q2.** v₁ = [0.485, 0.196, 0.319].
New Active = 0.485×0.90 + 0.196×0.20 + 0.319×0 = 0.4365 + 0.0392 = **0.4757**
New At-Risk = 0.485×0.08 + 0.196×0.40 + 0.319×0 = 0.0388 + 0.0784 = **0.1172**
New Churned = 0.485×0.02 + 0.196×0.40 + 0.319×1 = 0.0097 + 0.0784 + 0.319 = **0.4071**
v₂ = [0.4757, 0.1172, 0.4071]. Sum = 1.00 ✓

**Q3.** Yes, Churned is absorbing — its row is [0, 0, 1], meaning once churned, the probability of staying churned is 1. Long-run implication: all customers eventually end up in Churned. The steady-state has 100% in Churned. Retention strategy determines how slowly or quickly the distribution drains into the absorbing state.

*Common wrong answer:* "There is a stable mix of Active/At-Risk/Churned in the long run." Only if Churned has non-zero exit probabilities. With an absorbing Churned state, all probability eventually concentrates there.

**Q4.** **Decrease** toward 35%. The steady state is where the dynamics converge. Since current Active (70%) is above the steady state (35%), the system is drifting downward. Without intervention, Active% will decline.

*Common wrong answer:* Increase, because 70% > 35% means we are already above the target. The steady state is not a target — it is an attractor. The system moves toward it, not away from it.

*Contrast with Q3:* our P has an absorbing Churned, so its steady state is 0% Active and Active% decreases toward 0. Same rule — move toward the steady state — different destination.

**Q5.** **It does not change — the steady state stays at 0% Active / 100% Churned.** Churned is still absorbing, so every customer still ends up there eventually; raising At-Risk→Active cannot alter the destination. What the campaign changes is the **speed**: starting from [0.45, 0.40, 0.15], Active after 12 periods is 0.239 under the original P and 0.295 under P'. That improvement is real and is worth paying for — you just have to measure it at a **finite horizon**, not in the steady state.

*Common wrong answer:* "The steady-state Active% will be higher, because more customers are returning to Active each period." This is the trap the absorbing state sets. It is true that more customers return to Active each period, and true that the distribution is healthier at every finite horizon — but the steady state answers a different question ("where does this end up?"), and with an absorbing Churned the answer is always the same. The only intervention that moves the steady state is one that makes Churned **non-absorbing** — a win-back programme with a non-zero Churned → Active probability.

#### Slide Checkpoint Answers

Numbered to match the Checkpoint slide in this lecture's deck. Where the slide asks a question above again, or the same idea on different numbers, the entry points to that answer.

**Slide Q1.** *"Verify that row 2 of $P$ sums to 1."* — Not worked in the key. The rule is Tool 2 in Part 1 ("Every row of a valid transition matrix sums to 1"); row 2 is the At-Risk row of the $P$ in Q1 above.

**Slide Q2.** *"Starting from $\mathbf{v}_0 = [0.45, 0.40, 0.15]$, compute $\mathbf{v}_1$."* — Same question as Q1 above — see that answer.

**Slide Q3.** *"What does it mean for Churned to be an absorbing state? Give a business interpretation."* — Answered by Q3 above, which says what an absorbing Churned state means and what it implies in the long run.

**Slide Q4.** *"The At-Risk → Active rate improves from 0.20 to 0.35. Without computing, in which direction does $\mathbf{v}_1$ change compared to the baseline?"* — Not worked in the key. The closest answer is Q5 above: the same 0.20 → 0.35 change, but it asks about the steady state and Active% after 12 periods, not about $\mathbf{v}_1$.

**Slide Q5.** *"A manager says "Our steady-state shows 100% churn — the model must be broken." How do you respond?"* — Answered by Q3 above ("The steady-state has 100% in Churned") together with Q5 above (what a campaign can and cannot change when Churned is absorbing).

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal; work Part B together)


---

**Problem Statement**

A mobile gaming company tracks player engagement monthly. The transition count matrix from 12 months of data is:

| From↓ \ To→ | Active | At-Risk | Churned |
|---|---|---|---|
| Active | 560 | 140 | 100 |
| At-Risk | 180 | 170 | 150 |
| Churned | 0 | 0 | 800 |

**Part A:** Build the transition probability matrix $P$ by normalizing each row. Verify all rows sum to 1.

**Part B:** Compute the Active row of $P^2$: the 2-month transition probabilities starting from Active.

**Part C:** Is Churned an absorbing state? What is the long-run steady-state distribution? (Note: since Churned is absorbing, this is trivially $\pi_{Churned} = 1$. Instead, compute: starting from Active, what fraction of customers are still in a non-Churned state after 1 month? After 2 months? What pattern do you observe?)

---

**Full Solution**

**Part A: Transition Probability Matrix**

Row sums: Active = 800, At-Risk = 500, Churned = 800.

$$P = \begin{bmatrix} 560/800 & 140/800 & 100/800 \\ 180/500 & 170/500 & 150/500 \\ 0/800 & 0/800 & 800/800 \end{bmatrix} = \begin{bmatrix} 0.700 & 0.175 & 0.125 \\ 0.360 & 0.340 & 0.300 \\ 0.000 & 0.000 & 1.000 \end{bmatrix}$$

Row sums: $0.700+0.175+0.125 = 1.000$; $0.360+0.340+0.300 = 1.000$; $0+0+1 = 1.000$ ✓

**Part B: Active Row of $P^2$**

$$(P^2)_{A,A} = 0.700 \times 0.700 + 0.175 \times 0.360 + 0.125 \times 0.000 = 0.490 + 0.063 + 0.000 = \mathbf{0.553}$$

$$(P^2)_{A,R} = 0.700 \times 0.175 + 0.175 \times 0.340 + 0.125 \times 0.000 = 0.1225 + 0.0595 + 0.000 = \mathbf{0.182}$$

$$(P^2)_{A,C} = 0.700 \times 0.125 + 0.175 \times 0.300 + 0.125 \times 1.000 = 0.0875 + 0.0525 + 0.125 = \mathbf{0.265}$$

Check: $0.553 + 0.182 + 0.265 = 1.000$ ✓

**Part C: Churn Progression from Active State**

| Months elapsed | P(Active) | P(At-Risk) | P(Churned) |
|---|---|---|---|
| 0 | 1.000 | 0.000 | 0.000 |
| 1 | 0.700 | 0.175 | 0.125 |
| 2 | 0.553 | 0.182 | 0.265 |

The probability of being in a non-Churned state (Active + At-Risk):
- Month 0: 1.000 (all survive)
- Month 1: 0.700 + 0.175 = 0.875
- Month 2: 0.553 + 0.182 = 0.735

This is exactly the survival function $\hat{S}(t)$ from Lecture 4, computed here via Markov chain dynamics. The connection between survival analysis and Markov chains is not a coincidence — they model the same phenomenon from different angles.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The transition matrix heatmap:** Darker cells indicate higher transition probabilities. Look for:
- Dominant diagonal: most customers stay in their current state (typical)
- Heavy off-diagonal flows toward Churned: dangerous — many customers flowing out
- Unexpected flows (e.g., Churned → Active > 0): may indicate data quality issues or intentional win-back programs

**The steady-state vector:** For a chain with **no** absorbing state, it is the long-run customer mix: if the current distribution is better than the steady state, expect deterioration over time; if worse, expect improvement. With an absorbing Churned, as in this lecture and the homework, it is always [0, 0, 1], so it carries no planning information. Read the expected time to absorption (Part C) and the finite-horizon projection instead.

**Simulation output:** Plotting the fraction in each state over 24 simulated months starting from a realistic initial distribution. Look for:
- How quickly does the distribution converge to steady state?
- Is there a transient "danger period" when Churned fraction rises sharply?
- Does Active fraction drop below a threshold that triggers concern?

**Before trusting the agent output:**
1. Verify all rows of $P$ sum to 1 (within floating-point tolerance)
2. Verify the steady-state vector satisfies $\pi P \approx \pi$ (multiply it out and check)
3. If Churned is absorbing, verify $(P)_{Churned, j} = 0$ for all $j \neq$ Churned and $(P)_{Churned, Churned} = 1$
4. Do the transition probabilities make business sense? Active → Churned should be lower than At-Risk → Churned

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_05_markov_chains.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_05_markov_chains.ipynb` |
| Dataset | `homework_datasets/engagement_data.csv` |
| Graded questions | **16** — Part A: 8 · Part B: 5 · Part C: 3 |

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "The steady-state distribution tells us what will happen next month."**
The steady state is the long-run equilibrium — it describes what the distribution converges to over many periods, not what happens in one period. For short-run predictions, use $v_0 P^n$ where $v_0$ is today's distribution.

**2. "The Markov property means history does not matter at all."**
The Markov property means the transition probabilities depend only on the current state — but the current state itself summarizes the relevant history. A customer who is "At-Risk" has implicitly communicated their recent behavior through their state assignment.

**3. "A larger transition matrix is always more accurate."**
More states can capture finer distinctions but increase estimation error (fewer observations per cell), require more data to estimate reliably, and make the model harder to interpret. A 3–5 state model is often preferable to a 15-state model with sparse data.

**4. "Rows of $P^n$ are computed by multiplying each row element of $P$ by $n$."**
$P^n$ requires actual matrix multiplication ($n-1$ times), not element-wise scaling. The elements of $P^n$ are not simply $n$ times the elements of $P$.

**5. "If the steady state shows 40% Active, I can guarantee 40% Active customers tomorrow."** (A 40% Active steady state needs a chain with no absorbing state, one where churned customers can return.)
The steady state is an asymptotic property. Convergence may take months or years, and if the transition matrix itself changes (due to seasonality, product changes, or campaigns), the system may never actually reach the theoretical steady state.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Counts transitions, normalizes rows → matrix $P$ | Sections 1.3 and 1.1 — building $P$; Tool 2: row sums must equal 1 |
| Computes state distribution after $n$ months via $v_0 P^n$ | Section 1.4A — $P^n$ (several periods at once) |
| Solves $\pi P = \pi$ for steady state | Section 1.4B — the steady-state distribution (found by iterating $P$) |
| Identifies absorbing states | Section 1.3 — absorbing states ($P_{ii} = 1$) |
| Computes expected months-until-absorption per starting state | Section 1.4C — first-step analysis; Deep Dive — the fundamental matrix $N = (I-Q)^{-1}$ |
| Simulation of 500 journeys | Section 2.2 — Monte Carlo approximation to $P^n$ |

**What to verify:**
1. All **rows** of $P$ sum to 1.0 (within 0.001 tolerance). If the *columns* sum to 1 instead, the agent used the transposed convention (Pv); ask it to use this lecture's: rows = current state, v @ P
2. The steady-state vector satisfies $\pi P \approx \pi$ — multiply it out and check
3. If Churned is absorbing: $(P)_{Churned,Churned} = 1.0$, all other entries in that row = 0
4. Transition probabilities are directionally sensible (At-Risk → Churned > Active → Churned)
5. Every expected time to absorption is positive, and the healthier state's is the larger of the two (Section 1.4C)
