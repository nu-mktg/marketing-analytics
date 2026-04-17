# Chapter 6: Multi-Touch Attribution and Shapley Values
## Comprehensive Reference Guide

---

### Chapter Overview

This chapter derives the Shapley value from first principles in cooperative game
theory, proves the four axioms that uniquely characterize it, enumerates the full
calculation for 3-channel attribution, develops the Markov chain removal effect as
an alternative approach, and explains precisely why Shapley attribution differs from
causal incrementality.

**By the end of this chapter you should be able to:**
- State and explain all four Shapley axioms
- Enumerate all n! orderings and compute Shapley values for n=3 channels by hand
- Prove the efficiency axiom algebraically
- Compute the Markov chain removal effect for any channel
- Explain why Shapley measures correlation, not causation
- Identify when last-touch overcredits vs. undercredits a channel and what it implies

---

## 6.1 Strategic Frame: What Practitioners Know

### Jonathan Becker: The Systematic Destruction of Awareness Budgets

Jonathan Becker has managed over $3.5 billion in paid acquisition for companies
including Uber, Asana, Square, and Masterclass. He has watched last-touch attribution
systematically destroy awareness budgets across hundreds of campaigns.

The mechanism is precise:
1. Last-touch gives 70–80% of conversion credit to the final channel before purchase
   (typically email or branded paid search)
2. Marketing leadership, seeing this data, cuts awareness budgets (TV, podcast, social)
   and reallocates to email and paid search
3. Efficiency metrics for email and paid search hold for 6–12 months because the
   awareness channels built enough equity before being cut
4. Branded search volume begins declining. Direct traffic falls. Email open rates drop
   as the audience becomes less familiar with the brand.
5. Email and paid search performance deteriorates. The attribution model says these
   channels are now "less efficient." The brand has been hollowing out its own demand.

He describes this as spending "the last dollar you have on brand as a tactic, not as
a strategy." The analytical solution is to attribute value to the entire customer
journey, not just the last touchpoint.

### Yuriy Timen: The Channel Test Quality Problem

Yuriy Timen's contribution: attribution models only measure what actually happened.
If a channel was underfunded, ran only one creative concept, or was tested for only
three weeks, the MMM or attribution model will correctly report low performance — but
that low performance reflects an insufficient test, not an inherently weak channel.

His rule for YouTube specifically: a minimum of 5 million targeted impressions, at
least two to three creative angles with different visual approaches, and specific
click-through-rate benchmarks (above 2% view-through rate for brand campaigns) before
drawing any conclusions. If you don't meet these standards, you haven't tested YouTube
— you've tested one bad execution of YouTube.

The implication for Shapley analysis: a channel with low Shapley value that was
systematically undertested should be flagged as "insufficient data for conclusion"
rather than "low-value channel."

### Three Strategic Insights

**Insight 1: The Efficiency Axiom is a mathematical guarantee, not a coincidence.**

When Shapley values for three channels sum exactly to the overall conversion rate, this
is not a remarkable property of the specific dataset. It is guaranteed by the
mathematical structure of Shapley values for any correctly computed attribution.
The efficiency axiom states this formally: Σᵢ φᵢ = v(N). Any attribution system
that does not satisfy this property (including last-touch, first-touch, and linear)
is either over-counting or under-counting total credit across the portfolio.

**Insight 2: Last-touch overcredits closers and undercredits creators.**

A channel that frequently appears as the last touchpoint before conversion captures
demand that was created by earlier channels. Under last-touch, it appears enormously
valuable. Under Shapley, its value is assessed as its average marginal contribution
across all orderings — including orderings where it was not last. If that average
marginal contribution is low, the channel is mostly closing paths it didn't open.

The strategic implication: a Shapley value substantially below last-touch credit
indicates the channel should receive less budget if budget decisions have been made
on last-touch. Conversely, a channel with Shapley value substantially above last-touch
credit has been systematically under-invested.

**Insight 3: Attribution is not incrementality.**

Shapley attribution computes marginal contribution within observed conversion paths.
It answers: "given the paths that actually occurred, what share of each path's
conversion probability should be attributed to each channel?"

Incrementality answers: "if we turned this channel off, how many conversions would
we lose?" The two are related but not identical. A channel can have high Shapley
credit (it appears on many high-converting paths) while having low incrementality
(those paths would have converted anyway without it). The classic example: branded
paid search has high last-touch and moderate Shapley credit, but very low incrementality
— customers searching for your brand by name were going to find you regardless.

---

## 6.2 Mathematical Prerequisites

### 6.2.1 Combinatorics: Counting Orderings

The number of ways to arrange n distinct objects in a sequence is:
$$n! = n \times (n-1) \times (n-2) \times \cdots \times 2 \times 1$$

Examples: 2! = 2, 3! = 6, 4! = 24, 5! = 120, 10! = 3,628,800.

The number of orderings grows factorially. For 10 channels, computing exact Shapley
values requires evaluating 3.6 million orderings — feasible computationally but
not by hand.

### 6.2.2 Cooperative Game Theory: Foundations

A **cooperative game with transferable utility** consists of:
- A player set N = {1, 2, ..., n} (channels in attribution)
- A **characteristic function** v: 2^N → ℝ assigning a value to each coalition S ⊆ N

For attribution: v(S) = the conversion rate achievable using only the channels in
coalition S. v(∅) = 0 (no channels → no conversions).

The characteristic function encodes all relevant information about channel interactions.
Computing it requires evaluating the conversion rate for 2^n − 1 possible subsets.
For n = 3: 7 subsets. For n = 10: 1,023 subsets.

*Reference: Wikipedia, "Cooperative game theory" —
en.wikipedia.org/wiki/Cooperative_game_theory*
*Reference: Wikipedia, "Transferable utility" —
en.wikipedia.org/wiki/Transferable_utility*

---

## 6.3 Shapley Values: Full Derivation

### The Four Axioms

Shapley (1953) identified four properties that a "fair" attribution system should
satisfy, then proved that exactly one allocation rule satisfies all four simultaneously.

**Axiom 1 — Efficiency:**
$$\sum_{i \in N} \phi_i(v) = v(N)$$
The sum of all values equals the grand coalition's value (the overall conversion rate).

**Axiom 2 — Symmetry:**
If v(S ∪ {i}) = v(S ∪ {j}) for all S ⊆ N \ {i, j}, then φᵢ = φⱼ.
Channels that contribute identically to every coalition receive equal credit.

**Axiom 3 — Dummy player (null player):**
If v(S ∪ {i}) = v(S) for all S ⊆ N \ {i}, then φᵢ = 0.
A channel that never adds any value to any coalition receives zero credit.

**Axiom 4 — Additivity:**
φᵢ(v + w) = φᵢ(v) + φᵢ(w) for any two games v, w.
Credits from two independent conversion paths add linearly.

### The Shapley Value Formula

$$\phi_i(v) = \sum_{S \subseteq N \setminus \{i\}} \frac{|S|!(|N|-|S|-1)!}{|N|!} \big[v(S \cup \{i\}) - v(S)\big]$$

**Interpretation of the weight $\frac{|S|!(|N|-|S|-1)!}{|N|!}$:**

This is the probability that coalition S forms before player i, when the n players
arrive in a uniformly random order. Specifically:
- |S|! = ways to arrange the |S| players who arrive before i
- (|N|−|S|−1)! = ways to arrange the |N|−|S|−1 players who arrive after i
- |N|! = total number of orderings

So φᵢ(v) = expected marginal contribution of channel i, where the expectation is
over all n! orderings of the channels.

*Reference: Wikipedia, "Shapley value" — en.wikipedia.org/wiki/Shapley_value*
*Reference: Shapley, L. S. (1953). "A value for n-person games." In Kuhn and Tucker
(eds.), *Contributions to the Theory of Games*, Vol. 2. Princeton University Press.*

### Full Enumeration for n=3 Channels

Let N = {PS, S, E} (Paid Search, Social, Email). All orderings:

| Order | Channel arriving | Coalition S before | v(S) | v(S∪{channel}) | Marginal |
|---|---|---|---|---|---|
| PS,S,E | PS arrives 1st | {} | 0 | v({PS}) | v({PS}) |
| PS,S,E | S arrives 2nd | {PS} | v({PS}) | v({PS,S}) | v({PS,S})−v({PS}) |
| PS,S,E | E arrives 3rd | {PS,S} | v({PS,S}) | v({PS,S,E}) | v(N)−v({PS,S}) |
| PS,E,S | PS arrives 1st | {} | 0 | v({PS}) | v({PS}) |
| PS,E,S | E arrives 2nd | {PS} | v({PS}) | v({PS,E}) | v({PS,E})−v({PS}) |
| PS,E,S | S arrives 3rd | {PS,E} | v({PS,E}) | v(N) | v(N)−v({PS,E}) |

(Continuing for all 6 orderings...)

**Shapley value for PS** = (1/6) × [v({PS}) + v({PS}) + (v({PS,S})-v({S})) + (v(N)-v({S,E})) + (v({PS,E})-v({E})) + (v(N)-v({S,E}))]

All six orderings' marginal contributions are averaged.

**The efficiency axiom — algebraic proof:**

Sum Shapley values across all channels:
$$\sum_{i \in N} \phi_i = \sum_{i \in N} \sum_{S \subseteq N \setminus \{i\}} w(|S|,|N|) [v(S \cup \{i\}) - v(S)]$$

Each term $v(S \cup \{i\}) - v(S)$ appears with positive weight when i ∈ coalition
and negative weight when i ∉ coalition. The telescoping sum over all orderings
collapses to v(N) − v(∅) = v(N). QED.

*Reference: arXiv:1909.07538, Sundararajan and Najmi (2019). "The many Shapley values
for model explanation." arxiv.org/abs/1909.07538*

### Worked Numerical Example

Dataset conversion rates by coalition (estimated from data):

| Coalition S | v(S) |
|---|---|
| {} | 0 |
| {PS} | 0.22 |
| {S} | 0.14 |
| {E} | 0.08 |
| {PS, S} | 0.35 |
| {PS, E} | 0.28 |
| {S, E} | 0.20 |
| {PS, S, E} = N | **0.55** |

Computing φ_PS using all 6 orderings:

| Order | PS marginal | S marginal | E marginal |
|---|---|---|---|
| PS,S,E | v({PS})=0.22 | v({PS,S})−v({PS})=0.13 | v(N)−v({PS,S})=0.20 |
| PS,E,S | v({PS})=0.22 | v(N)−v({PS,E})=0.27 | v({PS,E})−v({PS})=0.06 |
| S,PS,E | v({PS,S})−v({S})=0.21 | v({S})=0.14 | v(N)−v({PS,S})=0.20 |
| S,E,PS | v(N)−v({S,E})=0.35 | v({S})=0.14 | v({S,E})−v({S})=0.06 |
| E,PS,S | v({PS,E})−v({E})=0.20 | v(N)−v({PS,E})=0.27 | v({E})=0.08 |
| E,S,PS | v(N)−v({S,E})=0.35 | v({S,E})−v({E})=0.12 | v({E})=0.08 |

φ_PS = (0.22+0.22+0.21+0.35+0.20+0.35)/6 = **1.55/6 = 0.258**
φ_S = (0.13+0.27+0.14+0.14+0.27+0.12)/6 = **1.07/6 = 0.178**
φ_E = (0.20+0.06+0.20+0.06+0.08+0.08)/6 = **0.68/6 = 0.113**

Sum check: 0.258 + 0.178 + 0.113 = **0.549 ≈ 0.55 = v(N)** ✓ (rounding difference)

> **Teaching note:** Reproduce this table on the board. Have students compute one
> column (e.g., E marginal) themselves before revealing the answer. The pattern
> to emphasize: each ordering represents "channels arrived in this sequence, so
> this channel's marginal contribution is what it adds to everything that came before."
> Budget 15 minutes for the full 3×6 table.

---

## 6.4 Markov Chain Removal Effect

An alternative approach: fit a Markov chain on the sequence of channels visited before
conversion. States include all channels plus Conversion and Null (non-conversion).
Transition probabilities are estimated from observed paths.

The **removal effect** for channel C:
$$\text{RE}(C) = 1 - \frac{P(\text{convert} \mid C \text{ removed})}{P(\text{convert} \mid \text{full model})}$$

"Removing" channel C means setting all transition probabilities into C to zero and
redistributing them to other states proportionally.

**Interpretation:** RE(C) = 0.25 means removing channel C would reduce the overall
conversion probability by 25%. This is the channel's structural importance to the
conversion funnel.

*Reference: arXiv:1506.06597, Shao and Li (2011). "Data-Driven Multi-Touch Attribution
Models." arxiv.org/abs/1506.06597*

---

## 6.5 Shapley vs. Incrementality: A Critical Distinction

**What Shapley measures:** The average marginal contribution of a channel to the
conversion rate across all observed paths. This is a statement about path membership
and correlation, not causation.

**What incrementality measures:** The causal effect of the channel — how many
conversions would be lost if the channel were entirely eliminated. This requires a
randomized holdout experiment.

**They can diverge substantially.** Example:
- Branded paid search appears on 80% of paths (high Shapley credit)
- But 90% of those searchers would have found the brand organically if the paid link
  hadn't existed (low incrementality)

No attribution model — including Shapley — can detect this divergence without
experimental data. The right use of Shapley: relative channel comparison and budget
allocation direction. The right use of incrementality holdouts: absolute ROI
measurement and "should we spend on this channel at all" decisions.

*Reference: arXiv:2103.13012, Gordal et al. (2021). "Causal Attribution for Marketing."
arxiv.org/abs/2103.13012*

---

## 6.6 Full Proof Appendix

### Proof: Shapley Value is the Unique Allocation Satisfying All Four Axioms

**Existence:** Direct verification that the Shapley formula satisfies each axiom.
- Efficiency: shown above (telescoping sum over orderings)
- Symmetry: follows because v(S∪{i}) = v(S∪{j}) for all S implies the same
  marginal contributions in all orderings → identical averages
- Dummy: v(S∪{i}) = v(S) for all S → all marginal contributions are 0 → φᵢ = 0
- Additivity: φᵢ(v+w) = Σ w(S)[v(S∪{i})+w(S∪{i})−v(S)−w(S)] = φᵢ(v) + φᵢ(w)

**Uniqueness:** Express any game v as a linear combination of **unanimity games**:
$v = \sum_{S \subseteq N} c_S u_S$ where $u_S(T) = 1$ iff $S \subseteq T$ and $c_S = \sum_{T \subseteq S}(-1)^{|S|-|T|}v(T)$.

For a unanimity game $u_S$, efficiency requires values sum to 1 (since $u_S(N) = 1$),
symmetry requires all members of S receive equal value, and dummy requires non-members
of S receive 0. Therefore each member of S receives 1/|S|. The additivity axiom
extends this to all games as linear combinations of unanimity games. This derivation
is unique — no other allocation satisfies all four axioms. QED.

*Reference: Wikipedia, "Shapley value — Proof of uniqueness" —
en.wikipedia.org/wiki/Shapley_value*

---

## 6.7 Verification Checklist

| Check | How to verify |
|---|---|
| n! orderings for n channels? | n=3: 6; n=4: 24; verify factorial |
| Each row of marginal contributions sums to v(N)? | By construction of each ordering |
| Sum of Shapley values = v(N)? | Efficiency axiom — should hold exactly (up to rounding) |
| Shapley value uses v(∅) = 0? | Empty coalition contributes 0 |
| Marginal contribution = v(S∪{i}) − v(S)? | Not v(S∪{i}) alone |
| All v(S) values estimated from data? | Must observe conversion rates for each coalition |
| Weight for |S| = 0: |S|!(n−1)!/n! = 1/n? | First arrival always contributes v({i}) − 0 = v({i}) |
| Last-touch > Shapley → overcredited? | Channel closes more than it creates |
| Shapley > last-touch → undercredited? | Channel creates more than it appears to close |

---

## 6.8 Chapter Bibliography

1. Wikipedia, "Shapley value" — en.wikipedia.org/wiki/Shapley_value
2. Wikipedia, "Cooperative game theory" — en.wikipedia.org/wiki/Cooperative_game_theory
3. Wikipedia, "Attribution (marketing)" — en.wikipedia.org/wiki/Attribution_(marketing)
4. Shapley, L. S. (1953). "A value for n-person games." In Kuhn and Tucker (eds.),
   *Contributions to the Theory of Games*, Vol. 2. Princeton University Press.
5. Shao, X. and Li, L. (2011). "Data-Driven Multi-Touch Attribution Models."
   Proceedings of KDD Workshop. arXiv:1506.06597. arxiv.org/abs/1506.06597
6. Sundararajan, M. and Najmi, A. (2019). "The many Shapley values for model explanation."
   ICML 2020. arXiv:1909.07538. arxiv.org/abs/1909.07538
7. Gordal, D. et al. (2021). "Causal Attribution for Marketing." arXiv:2103.13012.
   arxiv.org/abs/2103.13012
8. MIT OpenCourseWare, 14.126 Game Theory — ocw.mit.edu
9. Osborne, M. J. and Rubinstein, A. *A Course in Game Theory*. Free PDF:
   arielrubinstein.tau.ac.il/books/GT.pdf

**Lenny's Podcast episode references:**
- Jonathan Becker episode on paid acquisition and attribution — Lenny's Podcast
- Yuriy Timen episode on growth at Grammarly — Lenny's Podcast
