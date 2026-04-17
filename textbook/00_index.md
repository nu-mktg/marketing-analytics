# Marketing Analytics — Master Textbook
## Complete Knowledge Base for All 10 Course Modules

---

### Purpose of This Document

This textbook is the instructor's comprehensive reference for all course content.
It contains:

- **Full mathematical derivations** — including proofs and steps omitted from the
  v2 student materials (which use the two-track rescoped format)
- **Worked numerical examples** — all computations verified by hand to the level
  of detail needed to check agent-generated outputs
- **Strategic frame detail** — comprehensive summaries of what each Lenny's Podcast
  practitioner actually discussed, grounded in the original source transcripts
- **Verification checklists** — a checklist per chapter for auditing agent outputs
- **Free-access references** — Wikipedia, arXiv, and freely available lecture notes
  for every concept, for independent verification

---

### Chapter Index

| Chapter | Topic | Key math | Lines |
|---|---|---|---|
| 01 | Bayesian A/B Testing | Beta-Binomial conjugacy, Monte Carlo P(B>A) | 632 |
| 02 | Price Elasticity + OLS | OLS normal equations, log-log elasticity, IV | 522 |
| 03 | Marketing Mix Modeling | Adstock recursion, Hill function, budget opt | 342 |
| 04 | Survival Analysis | KM estimator, Cox partial likelihood, censoring | 342 |
| 05 | Markov Chains | Transition matrix, steady state, absorbing | 201 |
| 06 | Multi-Touch Attribution | Shapley value, efficiency axiom, Markov removal | 166 |
| 07 | Uplift Modeling | Potential outcomes, T-learner, Qini, threshold | 241 |
| 08 | CLV (BG/NBD) | BG/NBD likelihood, Gamma-Gamma, P(alive) | 248 |
| 09 | Conjoint Analysis | Random utility, MNL, WTP derivation, IIA | 206 |
| 10 | Prophet Forecasting | Fourier series, changepoints, MAPE | 277 |

---

### Teaching Technology Recommendation

**Recommended setup:** iPad + Apple Pencil + screen mirroring (AirPlay or HDMI).

- Use slide decks for Strategic Frame, Business Motivation, and final summary
- Switch to iPad/whiteboard for all derivations
- Export the iPad session as PDF and post to Canvas after class
- This creates a permanent record of exactly what was derived in class

**Derivation time budgets per chapter:**

| Chapter | Core derivation | Board time |
|---|---|---|
| 1 | Beta-Binomial conjugate update | 10–12 min |
| 2 | OLS slope formula from scratch | 15–20 min |
| 3 | Adstock geometric series + Hill EC₅₀ proof | 10 min |
| 4 | KM product formula step-by-step | 10 min |
| 5 | Matrix × vector multiplication, steady state | 10 min |
| 6 | Shapley enumeration for 3 channels | 12 min |
| 7 | Potential outcomes setup + T-learner formula | 8 min |
| 8 | BG/NBD sufficient statistics explanation | 8 min |
| 9 | WTP indifference condition derivation | 8 min |
| 10 | Fourier series concept (no computation) | 8 min |

---

### How to Use This Document for Verification

1. Before running any agent prompt: read the relevant chapter section so you know
   exactly what the correct output should be.
2. After the agent returns output: check the **Verification Checklist** at the end
   of the relevant chapter.
3. For any number that seems off: work through the corresponding **Worked Numerical
   Example** by hand and compare.
4. For any concept you're unsure about: follow the **Free-Access References** in the
   chapter bibliography.

---

### Global Master Bibliography

All references in this document are freely accessible online. Chapter-specific
bibliographies appear at the end of each chapter. The most important cross-chapter
references:

**Foundational statistics and probability:**
- Gelman et al. *Bayesian Data Analysis* 3rd ed — stat.columbia.edu/~gelman/book/BDA3.pdf
- Grinstead & Snell *Introduction to Probability* — math.dartmouth.edu/~prob/prob/prob.pdf
- MIT OpenCourseWare Statistics — ocw.mit.edu/search/?q=statistics

**Econometrics:**
- MIT OCW 14.381 Statistical Methods in Economics — ocw.mit.edu
- MIT OCW 14.382 Econometrics — ocw.mit.edu

**Machine learning and causal inference:**
- arXiv.org (search any model name for free preprints)
- Künzel et al. "Meta-learners for HTE" — arXiv:1706.03461

**Marketing-specific:**
- Fader & Hardie BG/NBD papers — brucehardie.com
- Prophet paper — peerj.com/preprints/3190/
- Train *Discrete Choice* — elsa.berkeley.edu/books/choice2.html
- Hyndman & Athanasopoulos *Forecasting: Principles and Practice* — otexts.com/fpp3/

**Wikipedia (key articles):**
All Wikipedia articles cited in this textbook are accurate and peer-monitored
for mathematical content. For any contested concept, cross-reference with the
arXiv or OCW source listed in the same chapter.

