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

**Example:** Without standardisation, `monthly_spend` (range \$0–\$2000) dominates `login_count` (range 0–50) because a \$100 spend difference looks 20× larger than a 1-login difference to the Euclidean distance metric.

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

### Part A: Math Questions (Individual)

```python
# Q1. Customer A = (z1=1.2, z2=0.8). Initial centres: mu1=(1.2,0.8), mu2=(-0.9,-1.1).
# Which cluster is A assigned to? (integer: 1 or 2)
q1_cluster_a = None
print(f'Q1: {q1_cluster_a}')

# Q2. Which cluster is B = (-0.9, -1.1) assigned to?
q2_cluster_b = None
print(f'Q2: {q2_cluster_b}')

# Q3. After assignment step, update mu1.
# Cluster 1 contains A and C. Updated mu1 x-coordinate = (1.2 + 0.7) / 2
q3_updated_mu1_x = None
print(f'Q3: {q3_updated_mu1_x}')

# Q4. Updated mu1 y-coordinate = (0.8 + 0.9) / 2
q4_updated_mu1_y = None
print(f'Q4: {q4_updated_mu1_y}')

# Q5. WCSS always decreases as k increases.
# Does this mean higher k is always the better model choice? (True/False)
q5_k_more_not_always = None   # False = higher k is NOT always better
print(f'Q5: {q5_k_more_not_always}')
```

### Part B: Agent Analysis (Collaboration permitted)

```
Dataset: customer_features.csv
Variables: recency_days, frequency, avg_order_value, support_tickets,
           email_open_rate (standardise these five)
           clv_12m, tau_hat (use for profiling ONLY, do not standardise)

Please:
1. Standardise the five clustering features to zero mean and unit variance.

2. Fit k-means for k=2 through 8.
   Plot WCSS vs k (elbow plot) and mean silhouette vs k.
   State your recommended k and why.

3. Fit k-means with the recommended k (use random_state=42).

4. For each cluster, compute the mean of:
   recency_days, frequency, avg_order_value, clv_12m, tau_hat.
   Assign a descriptive label to each cluster.

5. What fraction (%) of ALL customers have tau_hat > 0.133?

6. Report mean clv_12m and mean tau_hat for each cluster separately.

Print all results clearly labelled.
```

### Part C: Interpretation Questions (Collaboration permitted)

```python
# Q11. Before k-means, you MUST standardise features when:
# a) Features are measured in different units or have very different variances
# b) Only when k > 3
# c) Only for numeric features, not binary ones
# d) Standardisation is never needed for k-means
q11 = None
print(f'Q11: {q11}')

# Q12. Mean silhouette score = 0.18 for the chosen k. This means:
# a) Clustering is invalid — discard the results
# b) Cluster structure is weak; customers are not well-separated; interpret
#    segments as directional guidance rather than ground truth
# c) The distance metric is wrong — switch to Manhattan distance
# d) k is too small — always increase k until silhouette > 0.5
q12 = None
print(f'Q12: {q12}')

# Q13. Cluster 3: mean CLV = $380, mean tau_hat = 0.06 (threshold = 0.133).
# For the retention campaign, the correct decision is:
# a) Target Cluster 3 because their high CLV justifies the spend
# b) Do not target Cluster 3 with the retention campaign — expected
#    incremental profit = 0.06×$30 − $4 = −$2.20 regardless of CLV
# c) Target only the top 20% of Cluster 3 by CLV
# d) Reduce the campaign cost to $1.80 to break even on Cluster 3
q13 = None
print(f'Q13: {q13}')
```
