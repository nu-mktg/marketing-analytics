# Lecture 6: Multi-Touch Attribution
## Which Marketing Channels Actually Deserve Credit?

---

### Overview

**Business question:** Your customer touched Paid Search, Social, and Email before converting. How much credit does each channel deserve — and does it match how your budget is currently allocated?

**What you will be able to do:**
- Enumerate all orderings for a 3- or 4-channel model
- Compute Shapley values by hand for a simple example
- Explain why last-touch attribution systematically misleads budget decisions
- Interpret the Shapley efficiency axiom and what it guarantees
- Distinguish between Shapley attribution and true incrementality measurement

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Factorial Notation (n!)

n! (read "n factorial") means multiply all integers from 1 to n together.

> 3! = 3 × 2 × 1 = **6**
> 4! = 4 × 3 × 2 × 1 = **24**
> 2! = 2 × 1 = **2**

**Why we need this:** A 3-channel attribution model has 3! = 6 possible orderings to enumerate for Shapley values. A 4-channel model has 4! = 24.

---

#### Tool 2: Marginal Contribution

The marginal contribution of channel C to a coalition S is the increase in conversion probability when C joins a group that previously did not include it.

> MC(C, S) = v(S ∪ C) − v(S)

where v(S) = conversion rate when only the channels in S are present.

---

### Section 1.2 — Business Motivation

---

#### Why Last-Touch Destroys Awareness Budgets

Jonathan Becker has managed over $3.5 billion in paid acquisition for companies including Uber, Asana, and Masterclass. His consistent finding:

> *"Paid search captures demand. It does not create it."*

Here is how last-touch attribution damages budgets over time:

1. Company attributes 65% of conversions to Email (last touch before purchase)
2. Marketing team shifts budget toward Email, away from TV and Social
3. Email conversion rates hold steady for 6 months — success!
4. Organic branded search and direct traffic slowly decline
5. Email is harvesting awareness that Social and TV built — there is less and less to harvest
6. Eventually, Email performance deteriorates too

Yuriy Timen, head of growth at Grammarly for 8.5 years, adds the channel testing failure mode: "The only thing worse than a channel not working is when you prematurely and erroneously concluded it doesn't work and gave it an insufficient shot."

---

### Section 1.3 — Conceptual Framework

---

#### The Shapley Value: Fair Credit Allocation

The Shapley value from cooperative game theory asks: if each channel joins a "coalition" of channels in a random order, how much does each channel contribute on average when it joins?

A channel that frequently *creates* the purchase intent earns high Shapley credit even if it never appears last in the path. A channel that consistently *closes* paths it did not open earns lower Shapley credit than its last-touch share would suggest.

**The efficiency axiom:** Shapley values are mathematically guaranteed to sum to the overall conversion rate. This is not a coincidence — it is a property proven for any correctly computed Shapley attribution.

---

### Section 1.4 — Mathematical Framework

---

#### Part A: Enumerating Shapley Values

For 3 channels (Paid Search = PS, Social = S, Email = E), there are 3! = 6 orderings:

| Order | PS joins: v with PS − v without | S joins: v with S − v without | E joins: v with E − v without |
|---|---|---|---|
| PS, S, E | v(PS)−v(∅) | v(PS,S)−v(PS) | v(PS,S,E)−v(PS,S) |
| PS, E, S | v(PS)−v(∅) | v(PS,S,E)−v(PS,E) | v(PS,E)−v(PS) |
| S, PS, E | v(PS,S)−v(S) | v(S)−v(∅) | v(PS,S,E)−v(PS,S) |
| S, E, PS | v(PS,S,E)−v(S,E) | v(S)−v(∅) | v(S,E)−v(S) |
| E, PS, S | v(PS,E)−v(E) | v(PS,S,E)−v(PS,E) | v(E)−v(∅) |
| E, S, PS | v(PS,S,E)−v(S,E) | v(S,E)−v(E) | v(E)−v(∅) |

**Shapley value for each channel = average of its column across all 6 orderings.**

**The efficiency axiom in action:** φ_PS + φ_S + φ_E = v(PS,S,E) = overall conversion rate. This is guaranteed.

---

#### Part B: Markov Chain Removal Effects

An alternative attribution approach: fit a Markov chain on customer paths, then compute the conversion rate if each channel is removed. The removal effect for channel C is:

> RE(C) = 1 − P(convert | channel C removed) / P(convert | full model)

Channels with high removal effects are critically important to the journey.

> ### 🔍 Deep Dive: Why Shapley Values Are Uniquely Fair
> Shapley values are the unique attribution that satisfies four axioms simultaneously:
> 1. **Efficiency:** Values sum to the total outcome
> 2. **Symmetry:** Channels that contribute equally receive equal credit
> 3. **Dummy:** A channel that never adds value receives zero credit
> 4. **Additivity:** For two independent games, values add
> No other attribution method satisfies all four. Last-touch fails efficiency (100% to one channel), symmetry, and dummy player. Shapley is the unique "fair" allocation given these properties.

---

#### Part C: Shapley ≠ Incrementality

Shapley values measure marginal contribution within observed paths. They do NOT measure what would have happened if the channel were eliminated entirely (true incrementality).

A customer who would have converted through paid search even without seeing the social ad contributes to Social's Shapley credit (because Social appeared in the path). But if you turned off Social entirely, this customer's behavior would be unchanged.

**True incrementality requires a holdout experiment:** turn the channel off for a randomly selected geographic or customer segment. Only a designed experiment measures causal incrementality. Attribution models (including Shapley) are correlation-based.

---

### Part 1 Checkpoint

1. How many orderings must be enumerated for a 4-channel Shapley attribution?

2. Attribution results: Email last-touch = 65%, Email Shapley = 13.6%. What does this divergence tell you about Email's role in the conversion journey?

3. Shapley values: PS = 0.275, Social = 0.200, Email = 0.075. What is the overall conversion rate in this dataset?

4. True or False: Shapley attribution can tell you whether paid social caused incremental conversions.

5. Paid Search receives 8% of last-touch credit but 50% of Shapley credit. What does this imply for budget allocation if budgets were set based on last-touch?

---

### Checkpoint Answer Key

**Q1.** 4! = 4 × 3 × 2 × 1 = **24** orderings.
*Common wrong answer:* 4² = 16 (squaring instead of factorial) or 4 × 3 = 12 (omitting the last two terms).

**Q2.** Email is **overcredited** under last-touch. It frequently appears as the final touchpoint before conversion but generates low marginal value when evaluated across all orderings — other channels created the intent that Email closes. Implication: budgets built on last-touch have over-invested in Email.
*Common wrong answer:* Email is the most effective channel because it gets the most last-touch credit. High last-touch credit + low Shapley credit specifically indicates closing without creating.

**Q3.** By the efficiency axiom, Shapley values sum to the overall conversion rate. 0.275 + 0.200 + 0.075 = **0.550** = 55%.
*Common wrong answer:* Need additional information to compute the conversion rate. No — the sum of Shapley values IS the conversion rate, guaranteed by the efficiency axiom.

**Q4.** **False.** Shapley measures marginal contribution within observed paths — it is correlation-based. It cannot distinguish between "Social caused this conversion" and "Social appeared on the path of customers who would have converted anyway." Causal incrementality requires a holdout experiment.
*Common wrong answer:* True, because Shapley accounts for the full path, not just the last touch. Path accounting ≠ causal identification.

**Q5.** Paid Search is likely **under-invested**. Last-touch assigns only 8% of credit to PS, so budgets built on last-touch would have cut PS spend. But Shapley shows PS contributes 50% of conversion credit — it is a high-value channel that rarely appears last. Under-investment in PS means you are getting less from it than you should.
*Common wrong answer:* Paid Search should be cut because it only gets 8% of last-touch credit. This is precisely the error that last-touch attribution causes.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal)


---

**Problem Statement**

A direct-to-consumer brand has 3 channels: Paid Search (P), Social (S), Email (E). From journey data, the value function is estimated as:

| Coalition | $v(S)$ |
|---|---|
| $\{\}$ | 0 |
| $\{P\}$ | 0.25 |
| $\{S\}$ | 0.10 |
| $\{E\}$ | 0.08 |
| $\{P,S\}$ | 0.40 |
| $\{P,E\}$ | 0.35 |
| $\{S,E\}$ | 0.18 |
| $\{P,S,E\}$ | 0.50 |

**Part A:** Compute $\phi_P$ by enumerating all 6 orderings. Show each marginal contribution.

**Part B:** Compute $\phi_S$ and $\phi_E$. Verify the efficiency condition.

**Part C:** Under last-touch attribution (assume Email is always last), Email gets 100% of credit. Under Shapley, what percentage does Email actually receive? What does this difference mean for budget decisions?

---

**Full Solution**

**Part A: Paid Search ($\phi_P$)**

| Ordering | Coalition before P | Marginal: $v(S \cup \{P\}) - v(S)$ |
|---|---|---|
| (P, S, E) | $\{\}$ | $0.25 - 0 = 0.25$ |
| (P, E, S) | $\{\}$ | $0.25 - 0 = 0.25$ |
| (S, P, E) | $\{S\}$ | $0.40 - 0.10 = 0.30$ |
| (S, E, P) | $\{S,E\}$ | $0.50 - 0.18 = 0.32$ |
| (E, P, S) | $\{E\}$ | $0.35 - 0.08 = 0.27$ |
| (E, S, P) | $\{S,E\}$ | $0.50 - 0.18 = 0.32$ |

$$\phi_P = \frac{0.25+0.25+0.30+0.32+0.27+0.32}{6} = \frac{1.71}{6} = \mathbf{0.285}$$

**Part B: Social ($\phi_S$) and Email ($\phi_E$)**

For Social:

| Ordering | Coalition before S | Marginal |
|---|---|---|
| (P,S,E) | $\{P\}$ | $0.40-0.25=0.15$ |
| (P,E,S) | $\{P,E\}$ | $0.50-0.35=0.15$ |
| (S,P,E) | $\{\}$ | $0.10-0=0.10$ |
| (S,E,P) | $\{\}$ | $0.10-0=0.10$ |
| (E,P,S) | $\{P,E\}$ | $0.50-0.35=0.15$ |
| (E,S,P) | $\{E\}$ | $0.18-0.08=0.10$ |

$$\phi_S = \frac{0.15+0.15+0.10+0.10+0.15+0.10}{6} = \frac{0.75}{6} = \mathbf{0.125}$$

For Email:

| Ordering | Coalition before E | Marginal |
|---|---|---|
| (P,S,E) | $\{P,S\}$ | $0.50-0.40=0.10$ |
| (P,E,S) | $\{P\}$ | $0.35-0.25=0.10$ |
| (S,P,E) | $\{P,S\}$ | $0.50-0.40=0.10$ |
| (S,E,P) | $\{S\}$ | $0.18-0.10=0.08$ |
| (E,P,S) | $\{\}$ | $0.08-0=0.08$ |
| (E,S,P) | $\{\}$ | $0.08-0=0.08$ |

$$\phi_E = \frac{0.10+0.10+0.10+0.08+0.08+0.08}{6} = \frac{0.54}{6} = \mathbf{0.090}$$

**Efficiency check:** $0.285 + 0.125 + 0.090 = 0.500 = v(\{P,S,E\})$ ✓

**Part C: Last-touch vs. Shapley for Email**

Under last-touch (Email always last): Email receives 100% of the 0.50 total value = 0.50.

Under Shapley: Email receives 0.090 / 0.500 = **18%** of total value.

**Business implication:** Last-touch attribution would lead the marketing team to invest heavily in Email (cutting Paid Search and Social to fund more email campaigns). But Shapley reveals Paid Search generates 57% of value (0.285/0.500) — the single most important channel. A budget decision based on last-touch would significantly underinvest in the channel that actually drives the most conversions.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The attribution comparison table:** The most important output. Focus on channels where Shapley diverges substantially from last-touch or first-touch. These divergences represent misallocation risks.

**Interpreting Shapley %:** A channel's Shapley share represents its average marginal contribution across all orderings. A channel with high Shapley % but low last-touch % is typically an "awareness" channel that warms up customers before a final-click channel closes the deal.

**Interpreting the Markov removal effect:** A high removal effect (e.g., 0.45) means removing this channel would reduce conversions by 45%. Channels with low removal effects (close to 0) are redundant — the remaining channels would generate similar conversions without them.

**When the two methods agree:** If Shapley and Markov produce similar rankings, you can be more confident in the results. When they diverge substantially, investigate why — it often indicates that the path ordering matters a lot (Shapley may be more appropriate) or that data is sparse for some coalitions.

**Before trusting the agent output:**
1. Do Shapley values sum to approximately the observed overall conversion rate? (Efficiency axiom)
2. Are all Shapley values non-negative? Negative values indicate a computational error.
3. Are all removal effects between 0 and 1?
4. Does the channel with the most touchpoints have the highest Shapley value? Not necessarily — a channel in many paths but always as redundant filler should have low Shapley value.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit via GitHub Classroom)

**Repository:** `data/customer_journeys.csv` + `homework_06.ipynb`

---

#### Part A: Math Questions (no agent required)

Use the following value function for 3 channels: Display (D), Affiliate (A), Retargeting (R):

| Coalition | $v(S)$ |
|---|---|
| $\{\}$ | 0 |
| $\{D\}$ | 0.08 |
| $\{A\}$ | 0.12 |
| $\{R\}$ | 0.18 |
| $\{D,A\}$ | 0.22 |
| $\{D,R\}$ | 0.28 |
| $\{A,R\}$ | 0.35 |
| $\{D,A,R\}$ | 0.42 |

**Q1.** How many orderings of 3 channels exist? (Compute 3!)
```python
q1_num_orderings = None
```

**Q2.** In ordering (D, A, R), what is D's marginal contribution? (Coalition before D is empty.)
```python
q2_d_marginal_ordering1 = None
```

**Q3.** In ordering (A, D, R), what is D's marginal contribution? (Coalition before D is {A}.)
```python
q3_d_marginal_ordering3 = None
```

**Q4.** In ordering (A, R, D), what is D's marginal contribution? (Coalition before D is {A,R}.)
```python
q4_d_marginal_ordering4 = None
```

**Q5.** The 6 orderings and D's marginal contributions are:
(D,A,R): 0.08; (D,R,A): 0.08; (A,D,R): 0.10; (A,R,D): 0.07; (R,D,A): 0.10; (R,A,D): 0.07.
Compute $\phi_D$.
```python
q5_shapley_d = None
```

**Q6.** The Shapley values are $\phi_D = 0.083$, $\phi_A = 0.127$, $\phi_R = 0.210$. Verify the efficiency condition: do these sum to $v(\{D,A,R\})$?
```python
q6_efficiency_check = None  # True if sum ≈ 0.42, False otherwise
```

**Q7.** Under last-touch attribution (assume R is almost always last), Retargeting would receive nearly 100% of credit. Under Shapley, what percentage does Retargeting receive?
```python
# Enter as a percentage, e.g., 50.0 for 50%
q7_retargeting_shapley_pct = None
```

**Q8.** True or False: In this example, Display's Shapley value (0.083) is lower than its standalone conversion rate (0.08). Since the Shapley value is only slightly above Display's standalone rate, Display contributes relatively little incremental value when combined with other channels.
```python
q8_display_incremental = None  # True or False
```

---

#### Part B: Agent Questions

Paste the following **Context Prompt** into your agent:

```
I am a marketing analyst conducting multi-touch attribution analysis.

customer_journeys.csv has one row per touchpoint:
- customer_id
- session_date
- channel: one of "paid_search", "social", "email", "display", "organic"
- converted: 1 if this session ended in purchase, 0 otherwise

Please:
1. Compute last-touch, first-touch, linear (equal), and Shapley value
   attribution for each channel. Report as % of total conversions.
2. Compute Markov chain attribution using the removal effect method.
3. Create a comparison table: channel vs. attribution share under each method.
4. Plot a grouped bar chart comparing all methods side by side.
5. Identify the channel where Shapley and last-touch differ most.
Use the pychannel library if available; otherwise implement Shapley
via coalition enumeration with random seed 42.
```

**Q9.** What is the Shapley attribution share for paid_search? Round to 1 decimal place (as a %).
```python
q9_shapley_paid_search_pct = None
```

**Q10.** What is the last-touch attribution share for email? Round to 1 decimal place.
```python
q10_lastouch_email_pct = None
```

**Q11.** Which channel has the largest difference between Shapley and last-touch attribution?
```python
q11_biggest_divergence_channel = None  # channel name as string
```

**Q12.** What is the Markov removal effect for social? Round to 2 decimal places.
```python
q12_removal_effect_social = None
```

---

#### Part C: Interpretation Questions

**Q13.** Paid Search has Shapley share = 35% but last-touch share = 8%. What does this tell you about Paid Search's role in the customer journey?
- (a) Paid Search is overvalued by Shapley — last-touch is more accurate
- (b) Paid Search frequently appears early in paths and generates high marginal value, but rarely as the final click
- (c) Paid Search generates conversions that would have happened anyway without it
- (d) Paid Search is a dummy channel and should receive zero attribution
```python
q13 = None  # "a", "b", "c", or "d"
```

**Q14.** Email has Shapley share = 12% but last-touch share = 55%. A budget analyst proposes cutting Paid Search and tripling Email spend. Based on the attribution analysis, what is the main risk?
- (a) Email campaigns are always cheaper than Paid Search
- (b) Removing Paid Search may reduce the pool of customers that Email can later convert — the channels are complementary
- (c) Shapley values are always wrong for Email
- (d) The Markov model is more reliable than Shapley for Email
```python
q14 = None  # "a", "b", "c", or "d"
```

**Q15.** The Shapley and Markov methods give very similar channel rankings for your data. What does this convergence tell you about the reliability of the results?
- (a) Nothing — two methods agreeing is always suspicious
- (b) The rankings are likely robust — different methodological assumptions lead to the same conclusion
- (c) Both methods are wrong because they use the same data
- (d) You should use whichever method gives the highest attribution to your most expensive channel
```python
q15 = None  # "a", "b", "c", or "d"
```

---

### Common Misconceptions

**1. "Shapley values prove which channel causes conversions."**
Shapley values measure marginal contribution in a cooperative game defined by observed conversion rates. They capture correlation structure, not causation. A channel that always appears in converting paths may receive high Shapley credit even if it is not causally necessary.

**2. "Last-touch attribution is always wrong."**
Last-touch is sometimes appropriate — for example, if every conversion path consists of exactly one touchpoint, last-touch and Shapley give identical results. The bias of last-touch grows with the complexity and length of conversion paths.

**3. "The channel with the most touchpoints deserves the most credit."**
A channel that appears in many paths but never changes conversion probability has zero marginal contribution — the dummy player axiom assigns it zero Shapley value. Frequency of appearance is not the same as contribution.

**4. "Shapley values work for any number of channels."**
Exact Shapley computation requires $2^n$ coalition evaluations, growing exponentially. For $n = 20$ channels: over 1 million coalitions. In practice, approximate sampling-based Shapley methods are used for large channel sets.

**5. "Markov attribution is more accurate than Shapley."**
They model different things. Markov uses path ordering and transition probabilities; Shapley uses coalition membership and marginal contributions. Neither is universally more accurate — the better method depends on whether path ordering or coalition composition is more important for your data.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Computes $v(S)$ for each coalition from path data | Section 1.3 — value function definition |
| Enumerates orderings, averages marginal contributions | Section 1.4 — full Shapley derivation |
| Builds Markov transition matrix from journeys | Lecture 5, Section 1.4A — row normalization |
| Computes removal effect per channel | Section 1.3 — Markov attribution formula |
| Creates comparison table | Section 2.2 — interpretation of attribution divergence |

**What to verify before trusting the output:**
1. Shapley values sum to approximately the observed overall conversion rate (efficiency)
2. All Shapley values are non-negative
3. All removal effects are between 0 and 1
4. Channels in nearly every path do not automatically get the highest Shapley — verify the efficiency axiom makes business sense
