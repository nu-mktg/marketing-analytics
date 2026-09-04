# Lecture 13: Customer Segmentation
## Discovering Latent Structure in Customer Populations

---

### Overview

**Motivating Problem:** You have a database of 1,000 customers. Your CLV model gives each customer an expected future value. Your uplift model gives each a predicted treatment effect. Neither model tells you *which customers are similar* to each other, or how many meaningfully different types exist. Segmentation answers this — and the segments inform how you apply every downstream model.

**Learning Objectives:**

- State the k-means objective function and derive the assignment and update steps
- Apply the algorithm by hand on a two-dimensional example
- Choose k using the elbow method and silhouette score
- Explain why standardisation is necessary before clustering
- Connect segmentation outputs to CLV targeting and uplift campaign design

**Prerequisites:** Euclidean distance. No prior machine learning background needed.

---

## PART 1: Concepts and Mathematics

### (~1 hour 40 minutes)

---

### Section 1.1 — The Segmentation Problem

Standard models assign each customer a scalar: CLV = \$240, P(alive) = 0.73, τ̂ = 0.08.
These scalars are useful for individual targeting but do not describe *types* of customers.

**Segmentation asks:** Can we partition customers into groups such that members within a group are more similar to each other than to members in other groups?

This is **unsupervised learning** — there is no outcome variable to predict. We are discovering structure in the joint distribution of customer features.

---

### Section 1.2 — k-means: The Objective

Given $n$ customers with feature vectors $\mathbf{x}_i \in \mathbb{R}^p$ and $k$ cluster assignments $z_i \in \{1, \ldots, k\}$, find:

$$\boxed{\min_{\{z_i\}, \{\boldsymbol{\mu}_c\}} \sum_{i=1}^n \sum_{c=1}^k \mathbb{1}[z_i = c] \cdot \|\mathbf{x}_i - \boldsymbol{\mu}_c\|^2}$$

Minimise the **within-cluster sum of squared distances** (WCSS) from each customer to their cluster centre $\boldsymbol{\mu}_c$.

**Lloyd's Algorithm:**

1. **Initialise:** choose $k$ cluster centres (k-means++ is standard)
2. **Assign:** $z_i = \arg\min_c \|\mathbf{x}_i - \boldsymbol{\mu}_c\|^2$
3. **Update:** $\boldsymbol{\mu}_c = \frac{1}{|C_c|} \sum_{i: z_i = c} \mathbf{x}_i$
4. Repeat steps 2–3 until assignments do not change

---

### Section 1.3 — Worked Example

Three customers, two standardised features ($z_1$ = standardised spend, $z_2$ = standardised frequency):

| Customer | $z_1$ | $z_2$ |
|---|---|---|
| A | 1.2 | 0.8 |
| B | −0.9 | −1.1 |
| C | 0.7 | 0.9 |

**k = 2. Initial centres:** $\boldsymbol{\mu}_1 = (1.2, 0.8)$, $\boldsymbol{\mu}_2 = (-0.9, -1.1)$

**Assignment step** (compute Euclidean distance to each centre):

| Customer | Dist to $\boldsymbol{\mu}_1$ | Dist to $\boldsymbol{\mu}_2$ | Assigned cluster |
|---|---|---|---|
| A | $\sqrt{0^2 + 0^2} = 0.000$ | $\sqrt{4.41 + 3.61} = 2.832$ | **1** |
| B | $\sqrt{4.41 + 3.61} = 2.832$ | $\sqrt{0^2 + 0^2} = 0.000$ | **2** |
| C | $\sqrt{0.25 + 0.01} = 0.510$ | $\sqrt{2.56 + 4.00} = 2.561$ | **1** |

**Update step** — recompute cluster centres:

$$\boldsymbol{\mu}_1 = \frac{(1.2, 0.8) + (0.7, 0.9)}{2} = (0.95, 0.85)$$
$$\boldsymbol{\mu}_2 = \frac{(-0.9, -1.1)}{1} = (-0.9, -1.1) \text{ (unchanged)}$$

**Convergence check:** Reassign with new centres — assignments unchanged. Algorithm terminates.

**Interpretation:** Cluster 1 = high-spend, high-frequency customers (active). Cluster 2 = low-spend, low-frequency (dormant/at-risk).

---

### Section 1.4 — The Standardisation Requirement

**Problem:** Features on different scales give disproportionate weight to the feature with the largest variance.

**Example:** Without standardisation, `monthly_spend` (range \$0–\$2000) dominates `login_count` (range 0–50) — a 40× wider range. A \$100 spend difference contributes $100^2 = 10{,}000$ to the squared distance, while a 1-login difference contributes $1^2 = 1$.

**Fix:** Standardise each feature before clustering:

$$z_j = \frac{x_j - \bar{x}_j}{s_j}$$

After standardisation, all features have mean 0 and standard deviation 1. Each feature contributes equally to the distance metric.

---

### Section 1.5 — Choosing k

**Within-cluster sum of squares (WCSS):**

$$\text{WCSS}(k) = \sum_{c=1}^k \sum_{i: z_i=c} \|\mathbf{x}_i - \boldsymbol{\mu}_c\|^2$$

WCSS decreases monotonically as $k$ increases (at $k = n$, WCSS = 0). The **elbow** — the value of $k$ where additional clusters yield diminishing reduction — is a common heuristic.

**Silhouette score:**

For each customer $i$: let $a_i$ = mean distance to same-cluster members, $b_i$ = mean distance to nearest other-cluster members.

$$s_i = \frac{b_i - a_i}{\max(a_i, b_i)}, \qquad s_i \in [-1, 1]$$

$$\bar{s}(k) = \frac{1}{n} \sum_{i=1}^n s_i$$

Higher $\bar{s}$ = better cluster separation. Choose the $k$ that maximises $\bar{s}$.

---

### Section 1.6 — Connecting Segmentation to Downstream Models

Segmentation is not an end in itself — it is an input to targeted decision-making:

| Downstream use | How segments help |
|---|---|
| **CLV + segmentation** | Compute mean CLV per segment; prioritise high-CLV segments for premium offers and retention spend |
| **Uplift + segmentation** | Compute mean τ̂ per segment; some segments may be systematically above or below the targeting threshold |
| **Survival + segmentation** | Different segments may have very different churn timing; segment-specific KM curves reveal this |
| **Conjoint + segmentation** | Estimate separate MNL models per segment; different segments value attributes differently (basis for versioned products) |

---

### Part 1 Checkpoint

1. State the k-means objective function in words: what quantity is being minimised?

2. Using the worked example above, verify that Customer C is correctly assigned to Cluster 1 by computing both distances.

3. After convergence in the worked example, what are the updated cluster centres?

4. You have features: `monthly_spend` (\$0–\$2000), `login_count` (0–50/month), `support_tickets` (0–5/month). Why must you standardise before clustering?

5. At $k = 4$: WCSS = 1,840, silhouette = 0.31. At $k = 5$: WCSS = 1,690, silhouette = 0.24. Which k is better?

---

### Checkpoint Answer Key

**Q1.** k-means minimises the total within-cluster sum of squared Euclidean distances from each customer to their cluster centre. Equivalently, it minimises the total variance within clusters across all features simultaneously.

**Q2.** Distance from C = (0.7, 0.9) to $\boldsymbol{\mu}_1$ = (1.2, 0.8): $\sqrt{(0.7-1.2)^2 + (0.9-0.8)^2} = \sqrt{0.25 + 0.01} = 0.510$.  
Distance to $\boldsymbol{\mu}_2$ = (−0.9, −1.1): $\sqrt{(0.7+0.9)^2 + (0.9+1.1)^2} = \sqrt{2.56 + 4.00} = 2.561$.  
C is closer to $\boldsymbol{\mu}_1$, so assigned to Cluster 1. ✓

**Q3.** $\boldsymbol{\mu}_1 = (0.95, 0.85)$ (mean of A and C). $\boldsymbol{\mu}_2 = (-0.9, -1.1)$ (B alone, unchanged).

**Q4.** `monthly_spend` has a range 40× larger than `login_count`. Without standardisation, a \$100 difference in spend produces a distance contribution of $100^2 = 10{,}000$, while a 1-unit difference in logins contributes only $1^2 = 1$. Spend completely dominates the clustering. After standardisation, each feature contributes equally.

**Q5.** $k = 4$ is better. Although $k = 5$ has lower WCSS (expected — WCSS always decreases), it has a lower silhouette score (0.24 vs. 0.31), meaning customers are *less* well-separated into coherent groups. The silhouette score is the right criterion for comparing solutions of different $k$.

> **Also asked on the slides:** *"Cluster 2 has mean CLV = \$380 and mean τ̂ = 0.06. The campaign threshold is c/v = 4/30 = 0.133. Do you target Cluster 2 with the campaign?"* — **No.** The campaign decision is made on uplift against the threshold, not on value: mean τ̂ = 0.06 < 0.133, so the average member of Cluster 2 does not generate enough incremental conversion to cover the \$4 cost (expected profit ≈ 0.06 × \$30 − \$4 = −\$2.20 per contact, by the rule in Lecture 7). The \$380 mean CLV is the answer to a *different* question. Section 1.6 keeps the two uses separate: mean CLV per segment says which segments to prioritise for premium offers and retention spend, while mean τ̂ per segment says which segments sit above or below the targeting threshold. A high-CLV, low-uplift segment is exactly the case where those two recommendations diverge — worth protecting, not worth this campaign. Two cautions before acting on the segment mean: τ̂ varies *within* a cluster, so a sub-group above 0.133 may still be worth targeting, and the threshold moves with `c` and `v`.

---

## PART 2: Application

### Section 2.1 — Dataset

`customer_features.csv`: 1,000 customers × 8 variables.

| Variable | Description |
|---|---|
| `customer_id` | Unique identifier |
| `recency_days` | Days since last purchase |
| `frequency` | Number of purchases in past 12 months |
| `avg_order_value` | Mean order value (\$) |
| `support_tickets` | Support contacts in past 6 months |
| `email_open_rate` | Fraction of emails opened |
| `clv_12m` | Predicted 12-month CLV (\$) |
| `tau_hat` | Predicted treatment effect from T-learner |

**Do not standardise** `clv_12m`, `tau_hat`, or `customer_id` — these are outputs used for segment profiling, not clustering features.

---

### Section 2.3 — Homework Assignment
#### (~55 minutes in class | Due: start of next week's lecture | Submit by pushing to your course repository)

<!-- BEGIN GENERATED homework pointer - tools/render_lecture_homework.py; do not hand-edit.
     Replaces a ~150-line duplicate of the homework that had drifted into a DIFFERENT
     assignment (Task 004, 2026-08-13): for lectures 04-10 only 1-3 of ~16 question
     variables still matched the notebook, and the dataset filenames were wrong.
     The notebook is the assignment of record; answers live only in answer_keys/. -->

> **The assignment of record is the notebook, not this section.** Open
> `homework_notebooks/homework_13_segmentation.ipynb` — it carries the questions, the dataset
> description and the agent context prompt you will need. Nothing here restates
> them, so there is no second version to get out of step with the one you submit.

| | |
|---|---|
| Notebook | `homework_notebooks/homework_13_segmentation.ipynb` |
| Dataset | `homework_datasets/customer_features.csv` |
| Graded questions | **13** — Part A: 5 · Part B: 5 · Part C: 3 |
| Answer key (instructor only) | `answer_keys/hw13.json` |

Answers and tolerances are never duplicated outside `answer_keys/hwNN.json`
(rendered for instructors as `quiz/answer_key_values.md`).

<!-- END GENERATED homework pointer -->

