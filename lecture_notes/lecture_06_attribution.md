# Lecture 6: Multi-Touch Attribution
## Which Marketing Channels Actually Deserve Credit?

---

### Overview

**Business question:** Your customer touched Paid Search, Social, and Email before converting. How much credit does each channel deserve — and does it match how your budget is currently allocated?

**What you will be able to do:**
- Enumerate all orderings for a 3-channel model, and count them for 4 or more
- Compute Shapley values by hand for a simple example
- Explain why last-touch attribution systematically misleads budget decisions
- Interpret the Shapley efficiency axiom and what it guarantees
- Distinguish between Shapley attribution and true incrementality measurement

**Prerequisites:** Markov chains from Lecture 5 (transition matrix, rows that sum to 1, absorbing
states), used in Section 1.4B. Factorials and the Shapley value are built here.

---

## PART 1: Concepts and Mathematics

---

### Section 1.1 — Math Toolkit

---

#### Tool 1: Factorial Notation (n!)

n! (read "n factorial") means multiply all integers from 1 to n together.

> 3! = 3 × 2 × 1 = **6**
>
> 4! = 4 × 3 × 2 × 1 = **24**
>
> 2! = 2 × 1 = **2**

**Why we need this:** A 3-channel attribution model has 3! = 6 possible orderings to enumerate for Shapley values. A 4-channel model has 4! = 24.

---

#### Tool 2: Marginal Contribution

**Notation.** A *coalition* is a set of channels, written $T$ in this lecture. (We do not use $S$,
because $S$ is the Social channel. The homework prompt writes the coalition as $S$ — same object.)

The marginal contribution of channel C to a coalition T that does not contain C is how much the value
goes up when C joins T:

> MC(C, T) = v(T ∪ {C}) − v(T)

**v is the *value function*** (some texts, and this course's slides, say *characteristic function* —
same object, two names). It takes a coalition T and returns a number: the conversion value the
business gets when only the channels in T are available. Everything in this lecture is built on it,
so it has to be pinned down exactly:

> $$v(T) = \frac{\text{number of converting paths whose channels all lie inside } T}{\text{total number of paths}}$$

**Read the denominator carefully: it is *all* paths, always — it does not change with T.** Only the
numerator moves. This is the definition the homework pins, and it is not optional:

- **It makes v monotone.** Enlarging T can only let more paths into the numerator, so
  $v(T \cup \{C\}) \ge v(T)$ and **every marginal contribution is ≥ 0**. Shapley values then come out
  non-negative, which is what lets Section 2.2's diagnostic treat a negative φ as a red flag.
- **It makes the efficiency axiom mean something.** With this denominator $v(\text{all channels})$
  *is* the overall conversion rate across the whole dataset, so "the Shapley values sum to the
  overall conversion rate" (below) is a true statement rather than a near-miss.

⚠️ **The tempting alternative is wrong here, and it is wrong in a way that shows up in your graded
output.** "The conversion *rate among* the paths confined to T" — i.e. dividing by the *number of
paths that use only T* rather than by all paths — has a denominator that shrinks as T shrinks. That
v is not monotone on real path data: adding a channel can *lower* it, marginal contributions go
negative, and $v(\text{all})$ is no longer the overall conversion rate. If your Shapley values come
back with a negative entry, this is the first thing to check.

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

The Shapley value from cooperative game theory asks: if each channel joins a "coalition" of channels in a random order, how much does each channel contribute on average when it joins? We write channel C's Shapley value as $\phi_C$ ("phi").

**The efficiency axiom:** Shapley values sum to $v(\text{all channels})$, which with Tool 2's v is the overall conversion rate. The reason takes one line. Fix any ordering and add up the marginal contributions along it: $[v(\{C_1\}) - v(\varnothing)] + [v(\{C_1,C_2\}) - v(\{C_1\})] + \dots$ Every middle term cancels, leaving $v(\text{all}) - v(\varnothing)$. Since $v(\varnothing) = 0$ (no channels, no converting paths), every ordering sums to $v(\text{all})$, and so does their average. This holds for any v with $v(\varnothing) = 0$, monotone or not.

**What Shapley does with path data.** With Tool 2's v, the Shapley value has a simple form: **each conversion is split equally among the distinct channels on its path, whatever their order.** Here is why. A converting path counts toward v(T) only once *all* of its channels are in T. So in any ordering, that path's value goes entirely to whichever of its channels joins last. Each of its k channels is the last one in 1/k of the orderings, so each gets 1/k of that conversion. Three consequences:

- **Position does not matter.** A channel that often *opens* converting paths earns credit even if it never appears last. A channel that mostly *closes* paths other channels opened earns less than its last-touch share suggests, because it now shares each conversion.
- **For a single path, Shapley = linear credit:** a TV → Instagram → Search conversion gives each channel one third.
- **Non-converting paths never affect φ.** A channel that sits on many converting paths gets a lot of credit **whether or not it was needed** — Shapley cannot tell a channel that creates demand from one that just happens to be on the path. That question belongs to an experiment (Section 1.4C).

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

An alternative attribution approach uses Lecture 5's Markov chain. The states are a **Start** state, one state per channel, and two absorbing states: **Conversion** and **Null** (no conversion). Each path is read as a sequence of moves, for example Start → Social → Email → Conversion. Counting those moves gives the transition matrix, and each row sums to 1. P(convert | full model) is the probability that a journey from Start ends in Conversion.

To **remove** channel C, send every transition *into* C to Null instead, and recompute the probability of reaching Conversion. The removal effect for channel C is the *relative* drop:

> RE(C) = 1 − P(convert | channel C removed) / P(convert | full model)

A high removal effect means many of the chain's converting journeys pass through C.

⚠️ **"Removed" here means deleted from the fitted model, not switched off in the world.** The construction assumes that every journey through C would have ended without converting. A real customer who never saw C might still have converted through another channel. So the removal effect is not an incrementality estimate either (Part C).

> ### 🔍 Deep Dive: Why Shapley Values Are Uniquely Fair
> Shapley values are the unique attribution that satisfies four axioms simultaneously:
> 1. **Efficiency:** Values sum to the total outcome
> 2. **Symmetry:** Channels that contribute equally receive equal credit
> 3. **Dummy:** A channel that never adds value receives zero credit
> 4. **Additivity:** For two value functions on the same channels, the Shapley values of their sum are the sums of their Shapley values
> No other attribution method satisfies all four. Last-touch fails **symmetry** (position, not contribution, decides credit) and **dummy** (a channel that adds nothing still takes 100% whenever it happens to be last). ⚠️ Note what it does **not** fail: last-touch *does* satisfy **efficiency**, because efficiency only requires the credits to sum to v(N) — and giving one channel 100% of the total still sums to the total. So do first-touch and linear credit. **Efficiency alone is therefore not what makes Shapley fair; it is the only rule satisfying all four at once.**

---

#### Part C: Shapley ≠ Incrementality

Shapley values measure marginal contribution within observed paths. They do NOT measure what would have happened if the channel were eliminated entirely (true incrementality).

A customer who would have converted through paid search even without seeing the social ad contributes to Social's Shapley credit (because Social appeared in the path). But if you turned off Social entirely, this customer's behavior would be unchanged.

**True incrementality requires a holdout experiment:** turn the channel off for a randomly selected geographic or customer segment. Only a designed experiment measures causal incrementality. Attribution models (including Shapley) are correlation-based.

---

### Part 1 Checkpoint

> **These questions are not submitted and not graded.** Attempt them before class; we
> then work through them together, and you will be asked to **explain your reasoning, not
> just state the answer.** Using Copilot or Claude to reach the answer is expected — what
> you cannot outsource is the explanation.

1. How many orderings must be enumerated for a 4-channel Shapley attribution?

2. Attribution results: Email last-touch = 65%, Email Shapley = 25%. What does this divergence tell you about Email's role in the conversion journey?

3. Shapley values: PS = 0.275, Social = 0.200, Email = 0.075. What is the overall conversion rate in this dataset?

4. True or False: Shapley attribution can tell you whether paid social caused incremental conversions.

5. Paid Search receives 8% of last-touch credit but 50% of Shapley credit. What does this imply for budget allocation if budgets were set based on last-touch?

---

### Checkpoint Answer Key

**Q1.** 4! = 4 × 3 × 2 × 1 = **24** orderings.
*Common wrong answer:* 4² = 16 (squaring instead of factorial) or 4 × 3 = 12 (omitting the last two terms). 16 is also 2⁴, the number of *coalitions* of 4 channels — a different count (Misconception 4).

**Q2.** Email is **overcredited** under last-touch. It frequently appears as the final touchpoint before conversion, but most of those paths also contain other channels, so across all orderings Email shares each of those conversions with them. Email mostly closes paths that other channels opened. Implication: budgets built on last-touch have probably over-invested in Email.
*Common wrong answer:* Email is the most effective channel because it gets the most last-touch credit. High last-touch credit + low Shapley credit specifically indicates closing without creating.

**Q3.** By the efficiency axiom, Shapley values sum to the overall conversion rate. 0.275 + 0.200 + 0.075 = **0.550** = 55%.
*Common wrong answer:* Need additional information to compute the conversion rate. No — the sum of Shapley values IS the conversion rate, guaranteed by the efficiency axiom.

**Q4.** **False.** Shapley measures marginal contribution within observed paths — it is correlation-based. It cannot distinguish between "Social caused this conversion" and "Social appeared on the path of customers who would have converted anyway." Causal incrementality requires a holdout experiment.
*Common wrong answer:* True, because Shapley accounts for the full path, not just the last touch. Path accounting ≠ causal identification.

**Q5.** Paid Search is **under-credited** by last-touch, so it may be **under-invested**. Last-touch assigns only 8% of credit to PS, so budgets built on last-touch would have cut PS spend. Shapley gives PS 50% of conversion credit: it sits on many converting paths but rarely appears last. That makes PS the first candidate for more budget — but not proof that more PS spend will produce more conversions. Shapley is not incrementality (Section 1.4C), so test PS with a holdout before moving money.
*Common wrong answer:* Paid Search should be cut because it only gets 8% of last-touch credit. This is precisely the error that last-touch attribution causes. *The opposite error:* "Shapley proves PS drives conversions, so raise its budget." Shapley says last-touch under-credits PS; it does not say what extra spend would cause.

#### Slide Checkpoint Answers

Numbered to match the Checkpoint slide in this lecture's deck. Where the slide asks a question above again, or the same idea on different numbers, the entry points to that answer.

**Slide Q1.** *"For 4 channels, how many orderings must you evaluate?"* — Same question as Q1 above — see that answer.

**Slide Q2.** *"Channel E has Shapley value 0.08 but last-touch credit of 40%. What does this suggest?"* — Same reasoning as Q2 above, which uses Email (65% last-touch, 25% Shapley); the slide's Channel E has 40% last-touch and a Shapley value of 0.08.

**Slide Q3.** *"Verify the efficiency axiom: if $\phi_{PS}=0.275$, $\phi_S=0.200$, $\phi_E=0.075$, what is the overall conversion rate implied?"* — Same question as Q3 above — see that answer.

**Slide Q4.** *"Paid Search: Shapley = 0.28, incrementality holdout = 0.04. What does the gap tell you?"* — Same reasoning as Q4 above: Shapley credit is not incrementality, and only a holdout measures incremental conversions. Q4 asks about Paid Social in general; the slide gives Paid Search's two numbers.

**Slide Q5.** *"Under what conditions could a Shapley value be negative?"* — When the **value function is non-monotone**: some coalition got *worse* when the channel joined, so that channel's average marginal contribution across orderings comes out negative. Section 2.2 makes this diagnostic check number 2, immediately after the efficiency check in Q3 — a negative Shapley value "can be a genuine finding or a specification artefact, so investigate the value function rather than assuming a coding bug" (`homework_06` pins a monotone v for exactly this reason). The slide goes on to ask you to *construct* a characteristic function v that produces φᵢ < 0. Attempt it before reading on. One construction uses two channels:

| Coalition | ∅ | {1} | {2} | {1, 2} |
|---|---|---|---|---|
| v | 0 | 0 | 10 | 0 |

There are 2! = 2 orderings. In (1, 2), channel 1 joins the empty coalition and adds v({1}) − v(∅) = 0. In (2, 1), it joins {2} and adds v({1, 2}) − v({2}) = 0 − 10 = −10. So $\phi_1 = \tfrac{1}{2}(0) + \tfrac{1}{2}(-10) = \mathbf{-5}$. Likewise $\phi_2 = \tfrac{1}{2}(10) + \tfrac{1}{2}(0) = +5$, and efficiency holds: $\phi_1 + \phi_2 = 0 = v(\{1, 2\})$. Channel 1 destroys channel 2's value when it joins — a channel that cannibalises or annoys. Under the homework's v this cannot happen, because adding a channel can never remove a converting path.

## PART 2: Application
### (~1 hour 40 minutes)

---

### Section 2.1 — Worked Example
#### (~30 minutes | Hybrid: attempt Part A first, then reveal)


---

**Problem Statement**

A direct-to-consumer brand has 3 channels: Paid Search (PS), Social (S), Email (E). From journey data, the value function is estimated as:

| Coalition $T$ | $v(T)$ |
|---|---|
| $\{\}$ | 0 |
| $\{PS\}$ | 0.25 |
| $\{S\}$ | 0.10 |
| $\{E\}$ | 0.08 |
| $\{PS,S\}$ | 0.40 |
| $\{PS,E\}$ | 0.35 |
| $\{S,E\}$ | 0.18 |
| $\{PS,S,E\}$ | 0.50 |

**Part A:** Compute $\phi_{PS}$ by enumerating all 6 orderings. Show each marginal contribution.

**Part B:** Compute $\phi_S$ and $\phi_E$. Verify the efficiency condition.

**Part C:** The table is built from journey data with Tool 2's v, so it also tells you which converting paths exist. (a) What share of all paths are converting paths with exactly {PS}? Exactly {PS, S}? Use differences of v. Find the share for every channel set. (b) The table does not record the order of touches. What range of last-touch credit can each channel have? (c) Compare each channel's Shapley share with its last-touch range. Why are they so close here, and what does that mean for budget decisions?

---

**Full Solution**

**Part A: Paid Search ($\phi_{PS}$)**

| Ordering | Coalition $T$ before PS | Marginal: $v(T \cup \{PS\}) - v(T)$ |
|---|---|---|
| (PS, S, E) | $\{\}$ | $0.25 - 0 = 0.25$ |
| (PS, E, S) | $\{\}$ | $0.25 - 0 = 0.25$ |
| (S, PS, E) | $\{S\}$ | $0.40 - 0.10 = 0.30$ |
| (S, E, PS) | $\{S,E\}$ | $0.50 - 0.18 = 0.32$ |
| (E, PS, S) | $\{E\}$ | $0.35 - 0.08 = 0.27$ |
| (E, S, PS) | $\{S,E\}$ | $0.50 - 0.18 = 0.32$ |

$$\phi_{PS} = \frac{0.25+0.25+0.30+0.32+0.27+0.32}{6} = \frac{1.71}{6} = \mathbf{0.285}$$

**Part B: Social ($\phi_S$) and Email ($\phi_E$)**

For Social:

| Ordering | Coalition $T$ before S | Marginal |
|---|---|---|
| (PS,S,E) | $\{PS\}$ | $0.40-0.25=0.15$ |
| (PS,E,S) | $\{PS,E\}$ | $0.50-0.35=0.15$ |
| (S,PS,E) | $\{\}$ | $0.10-0=0.10$ |
| (S,E,PS) | $\{\}$ | $0.10-0=0.10$ |
| (E,PS,S) | $\{PS,E\}$ | $0.50-0.35=0.15$ |
| (E,S,PS) | $\{E\}$ | $0.18-0.08=0.10$ |

$$\phi_S = \frac{0.15+0.15+0.10+0.10+0.15+0.10}{6} = \frac{0.75}{6} = \mathbf{0.125}$$

For Email:

| Ordering | Coalition $T$ before E | Marginal |
|---|---|---|
| (PS,S,E) | $\{PS,S\}$ | $0.50-0.40=0.10$ |
| (PS,E,S) | $\{PS\}$ | $0.35-0.25=0.10$ |
| (S,PS,E) | $\{PS,S\}$ | $0.50-0.40=0.10$ |
| (S,E,PS) | $\{S\}$ | $0.18-0.10=0.08$ |
| (E,PS,S) | $\{\}$ | $0.08-0=0.08$ |
| (E,S,PS) | $\{\}$ | $0.08-0=0.08$ |

$$\phi_E = \frac{0.10+0.10+0.10+0.08+0.08+0.08}{6} = \frac{0.54}{6} = \mathbf{0.090}$$

**Efficiency check:** $0.285 + 0.125 + 0.090 = 0.500 = v(\{PS,S,E\})$ ✓

**Part C: Last-touch vs. Shapley**

**(a) The path mix.** v({PS}) counts converting paths that use only PS, so 0.25 of all paths are PS-only conversions. v({PS, S}) also counts PS-only and S-only paths, so the paths with exactly PS and S are 0.40 − 0.25 − 0.10 = 0.05. The same steps give:

| Converting paths with exactly | Share of all paths |
|---|---|
| {PS} | 0.25 |
| {S} | 0.10 |
| {E} | 0.08 |
| {PS, S} | 0.40 − 0.25 − 0.10 = 0.05 |
| {PS, E} | 0.35 − 0.25 − 0.08 = 0.02 |
| {S, E} | 0.18 − 0.10 − 0.08 = 0.00 |
| {PS, S, E} | 0.50 − (0.25 + 0.10 + 0.08 + 0.05 + 0.02 + 0.00) = 0.00 |
| **Total** | **0.50** = v({PS, S, E}) ✓ |

Check against Section 1.3's rule (each conversion is split equally among the channels on its path): $\phi_{PS} = 0.25 + 0.05/2 + 0.02/2 = 0.285$ ✓.

**(b) Last-touch ranges.** A single-channel path always credits that channel. Only the two-channel paths depend on order. As a share of the 0.50 in conversions:

| Channel | Last-touch: fewest → most | Last-touch share | Shapley share |
|---|---|---|---|
| PS | 0.25 → 0.25 + 0.05 + 0.02 = 0.32 | 50% – 64% | 0.285 / 0.50 = **57%** |
| S | 0.10 → 0.10 + 0.05 = 0.15 | 20% – 30% | 0.125 / 0.50 = **25%** |
| E | 0.08 → 0.08 + 0.02 = 0.10 | 16% – 20% | 0.090 / 0.50 = **18%** |

**(c) Why they agree.** 0.43 of the 0.50 in conversions (86%) come from single-channel paths, and on those every method gives the same answer (Misconception 2). Here, last-touch and Shapley could not disagree by much whatever the order of touches. **Business implication:** in this data, switching from last-touch to Shapley would barely move the budget. Attribution matters when many conversions come from multi-channel paths, as in Checkpoint Q2, where Email is last on 65% of conversions but gets 25% under Shapley. Before choosing a model, check how many conversions are multi-channel. Even where the methods disagree, Shapley does not show what extra spend would *cause* (Section 1.4C); that takes a holdout.

---

### Section 2.2 — Interpretation Guide
#### (~10 minutes)

**The attribution comparison table:** The most important output. Focus on channels where Shapley diverges substantially from last-touch or first-touch. These divergences represent misallocation risks.

**Interpreting Shapley %:** A channel's Shapley share represents its average marginal contribution across all orderings. A channel with high Shapley % but low last-touch % tends to appear early in converting paths. It may be an "awareness" channel that warms up customers before a final-click channel closes the deal — or it may just ride along early in journeys that would have converted anyway. Shapley cannot tell these two apart (check 4 below).

**Interpreting the Markov removal effect:** A high removal effect (e.g., 0.45) means that deleting this channel from the fitted chain removes 45% of the chain's conversions. A removal effect close to 0 means few of the chain's converting journeys pass through the channel. Both are statements about the model: the chain assumes a journey through a removed channel never converts, so a removal effect is not a forecast of what switching the channel off would do (Section 1.4B).

**When the two methods agree:** If Shapley and Markov produce similar rankings, you can be more confident in the results. When they diverge substantially, investigate why — it often indicates that the path ordering matters a lot (Markov uses order; Shapley ignores it) or that data is sparse for some coalitions.

**Before trusting the agent output:**
1. Do Shapley values sum to approximately the observed overall conversion rate? (Efficiency axiom. It holds by construction, so it checks the *arithmetic*, never the business story.)
2. Are all Shapley values non-negative? A negative value means the **value function is non-monotone** — some coalition got *worse* when a channel joined. That can be a genuine finding or a specification artefact, so investigate the value function rather than assuming a coding bug (`homework_06` pins a monotone v for exactly this reason).
3. Are all removal effects between 0 and 1?
4. Is the channel with the highest Shapley value simply the one on the most converting paths? With this v it usually is, because each conversion is split equally among the channels on its path (Section 1.3). A high Shapley value says a channel is *present* on converting paths, not that it was *needed*: a redundant channel that rides along on many converting paths still scores high. Only a holdout separates the two.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: 23:59 ET on the due date in the course schedule | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_06_attribution.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_06_attribution.ipynb` |
| Dataset | `homework_datasets/attribution_data.csv` |
| Graded questions | **15** — Part A: 8 · Part B: 4 · Part C: 3 |

<!-- END GENERATED homework pointer -->

### Common Misconceptions

**1. "Shapley values prove which channel causes conversions."**
Shapley values measure marginal contribution in a cooperative game defined by observed conversion rates. They capture correlation structure, not causation. A channel that always appears in converting paths may receive high Shapley credit even if it is not causally necessary.

**2. "Last-touch attribution is always wrong."**
Last-touch is sometimes appropriate — for example, if every conversion path consists of exactly one touchpoint, last-touch and Shapley give identical results. The bias of last-touch grows with the complexity and length of conversion paths.

**3. "The channel with the most touchpoints deserves the most credit."**
Two touches by the same channel on one path count once: Shapley splits each conversion among the *distinct* channels on its path. And appearances on paths that did not convert earn nothing. But do not over-correct. With this course's v, a channel on many *converting* paths does earn a lot of Shapley credit, even if it was redundant filler. The dummy axiom gives zero only to a channel that never changes v, which here means one that is on no converting path at all. Appearing on converting paths is not the same as causing them (Misconception 1).

**4. "Shapley values work for any number of channels."**
Exact Shapley computation has $n!$ orderings but only $2^n$ distinct coalitions. Every ordering reuses the same coalition values, so you evaluate $v$ $2^n$ times and then average over the orderings with the weights in the slides' formula. Both grow fast. For $n = 20$ channels: over 1 million coalitions ($2^{20}$) and about $2.4 \times 10^{18}$ orderings ($20!$). In practice, approximate sampling-based Shapley methods are used for large channel sets.

**5. "Markov attribution is more accurate than Shapley."**
They model different things. Markov uses path ordering and transition probabilities; Shapley uses coalition membership and marginal contributions. Neither is universally more accurate — the better method depends on whether path ordering or coalition composition is more important for your data.

---

### From Theory to Agent

| Agent step | Corresponds to |
|---|---|
| Computes $v(S)$ for each coalition from path data (the prompt writes the coalition as $S$) | Section 1.1, Tool 2 — the $v(T)$ definition |
| Enumerates orderings, averages marginal contributions | Section 1.4 — full Shapley derivation |
| Builds Markov transition matrix from journeys | Lecture 5, Section 1.1, Tool 2 — row sums must equal 1 |
| Computes removal effect per channel | Section 1.4B — the removal effect RE(C) |
| Creates comparison table | Section 2.2 — interpretation of attribution divergence |

**What to verify before trusting the output:**
1. Shapley values sum to approximately the observed overall conversion rate (efficiency)
2. Shapley values are non-negative — or, if not, the value function is non-monotone (investigate v, not the code)
3. All removal effects are between 0 and 1
4. A high Shapley value means a channel is on many converting paths, not that it was needed — a redundant channel that rides along everywhere still scores high (Section 2.2, check 4). Efficiency (check 1) is guaranteed by construction, so it checks the *arithmetic*, never the business story
