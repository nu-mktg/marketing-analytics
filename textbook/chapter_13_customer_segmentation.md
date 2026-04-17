# Chapter 13: Customer Segmentation
## Discovering Latent Structure in Customer Populations

---

## 13.1 Overview

Segmentation addresses a question that prediction models cannot: *which customers are similar to each other?* CLV models rank customers by expected value. Uplift models rank them by predicted responsiveness. Neither identifies natural groupings that can guide systematic strategy — which customer types exist, how many there are, and what each group looks like.

k-means clustering is the most widely used segmentation method in marketing analytics. It partitions customers into groups that minimise within-group heterogeneity. The groups are then profiled by computing within-cluster means of CLV, treatment effect, churn probability, and other downstream model outputs.

**Estimand:** A partition $\{C_1, \ldots, C_k\}$ of the customer base that minimises total within-cluster variance in the feature space.

**Required data:** Numeric customer-level features (standardised before use).

---

## 13.2 The k-means Objective

### Problem formulation

Given $n$ customers with feature vectors $\mathbf{x}_i \in \mathbb{R}^p$, find $k$ cluster centres $\{\boldsymbol{\mu}_c\}_{c=1}^k$ and assignments $\{z_i\}_{i=1}^n$ that minimise:

$$\text{WCSS}(k) = \sum_{i=1}^n \|\mathbf{x}_i - \boldsymbol{\mu}_{z_i}\|^2 = \sum_{c=1}^k \sum_{i: z_i=c} \|\mathbf{x}_i - \boldsymbol{\mu}_c\|^2$$

This problem is NP-hard in general. **Lloyd's algorithm** finds a local minimum efficiently:

1. **Initialise** $k$ centres (k-means++ selects initial centres that are spread apart)
2. **Assign** each customer to the nearest centre: $z_i = \arg\min_c \|\mathbf{x}_i - \boldsymbol{\mu}_c\|^2$
3. **Update** centres to the mean of their assigned customers: $\boldsymbol{\mu}_c = \frac{1}{|C_c|}\sum_{i: z_i=c} \mathbf{x}_i$
4. Repeat steps 2–3 until no assignment changes

### Convergence guarantee

Lloyd's algorithm converges in finite steps because there are finitely many partitions and WCSS decreases or remains constant at each iteration. However, it converges to a *local* minimum. Running with multiple initialisations (scikit-learn default: `n_init=10`) mitigates this.

---

## 13.3 Worked Example

Three customers with two standardised features ($z_1$ = standardised spend, $z_2$ = standardised frequency). Initial centres: $\boldsymbol{\mu}_1 = (1.2, 0.8)$, $\boldsymbol{\mu}_2 = (-0.9, -1.1)$.

**Assignment step:**

$$d(A, \boldsymbol{\mu}_1) = \|(1.2-1.2, 0.8-0.8)\| = 0.000 \quad d(A, \boldsymbol{\mu}_2) = \|(1.2+0.9, 0.8+1.1)\| = 2.832 \implies z_A = 1$$

$$d(B, \boldsymbol{\mu}_1) = 2.832 \qquad d(B, \boldsymbol{\mu}_2) = 0.000 \implies z_B = 2$$

$$d(C, \boldsymbol{\mu}_1) = \|(0.7-1.2, 0.9-0.8)\| = \sqrt{0.25+0.01} = 0.510 \quad d(C, \boldsymbol{\mu}_2) = \sqrt{2.56+4.00} = 2.561 \implies z_C = 1$$

**Update step:** $\boldsymbol{\mu}_1 = \frac{(1.2+0.7)}{2}, \frac{(0.8+0.9)}{2}) = (0.95, 0.85)$. $\boldsymbol{\mu}_2 = (-0.9, -1.1)$ (unchanged).

**Convergence:** Reassigning with updated centres produces the same assignments. Algorithm terminates after one iteration.

---

## 13.4 Standardisation

### Why it is required

k-means minimises Euclidean distance. When features are measured in different units, the feature with the largest variance dominates the distance metric.

**Illustration:** `monthly_spend` has range \$0–\$2,000. `login_count` has range 0–50. A \$100 difference in spend contributes $100^2 = 10{,}000$ to WCSS. A one-unit difference in logins contributes $1^2 = 1$. Spend has 10,000× more influence on cluster assignments than logins.

**The fix:** Standardise each feature to zero mean and unit variance before clustering:

$$z_{ij} = \frac{x_{ij} - \bar{x}_j}{s_j}$$

After standardisation, a one-unit change in any feature contributes equally to the distance metric regardless of its original scale.

### What not to standardise

Do not standardise features that are outputs of other models and used only for profiling (not clustering). In the course dataset: `clv_12m` and `tau_hat` are profiling variables — standardise the five feature variables (`recency_days`, `frequency`, `avg_order_value`, `support_tickets`, `email_open_rate`) and profile clusters using the raw CLV and tau_hat values.

---

## 13.5 Choosing k

### The elbow method

Plot WCSS against $k = 2, 3, \ldots, K$. WCSS always decreases as $k$ increases — at $k=n$, each customer is its own cluster and WCSS = 0. The **elbow** is the value of $k$ beyond which the WCSS decrease becomes small relative to the decrease at lower $k$.

**Limitation:** The elbow is often ambiguous. Many datasets produce smooth WCSS curves without a clear bend.

### The silhouette score

For each customer $i$:
- $a_i$ = mean distance to all other customers in the same cluster
- $b_i$ = mean distance to all customers in the nearest other cluster

$$s_i = \frac{b_i - a_i}{\max(a_i, b_i)} \in [-1, 1]$$

$s_i$ close to 1: customer is well-matched to its cluster and poorly-matched to neighbours.  
$s_i$ close to 0: customer is near the boundary between clusters.  
$s_i$ negative: customer may have been assigned to the wrong cluster.

The **mean silhouette** $\bar{s}(k) = \frac{1}{n}\sum_i s_i$ is a scalar measure of clustering quality. Choose the $k$ that maximises $\bar{s}(k)$.

### Interpretation thresholds

| $\bar{s}$ | Interpretation |
|---|---|
| > 0.70 | Strong structure — clusters are well-separated |
| 0.50–0.70 | Reasonable structure |
| 0.25–0.50 | Weak structure — segments are directional, not definitive |
| < 0.25 | No substantial structure — reconsider features or method |

Marketing customer data typically produces silhouette scores in the 0.20–0.45 range. Weak structure is common and does not invalidate the segmentation; it means the segments should be used for directional strategy rather than precise targeting.

---

## 13.6 Connecting Segmentation to Downstream Models

### CLV + segmentation

Compute mean `clv_12m` per cluster. High-CLV clusters warrant premium retention offers and higher CAC caps for acquisition. Low-CLV clusters may be served with lower-cost channels.

### Uplift + segmentation

Compute mean `tau_hat` per cluster and compare to the targeting threshold $c/v$. Segments with mean $\bar{\tau} > c/v$ are collectively worth targeting. Note that even within a high-mean-tau segment, individual customers with $\hat{\tau}(x) < c/v$ should not be targeted — segment-level rules are less precise than individual targeting.

### Segment-specific model estimation

For heterogeneous populations, fitting separate models per segment often improves fit:
- Separate MNL conjoint models per segment reveal different WTP structures
- Separate Cox models per segment reveal different churn predictors
- Separate Markov matrices per segment reveal different transition dynamics

This is computationally straightforward — filter the dataset to the segment and re-estimate.

---

## 13.7 Scope and Limitations

**k-means assumptions:**

1. **Spherical clusters:** k-means minimises squared Euclidean distance. It performs best when clusters are approximately round in the feature space. Elongated, irregular, or nested clusters are handled poorly. Alternatives: GMM (Gaussian Mixture Models), DBSCAN.

2. **Equal cluster sizes:** k-means tends to produce clusters of similar size even when true group sizes are very different. If you have one very large "average" customer group and several small distinctive groups, k-means may merge the small groups into the large one.

3. **Hard assignment:** Every customer belongs to exactly one cluster. Customers near boundaries may be unstable — small changes in data or seed produce different assignments.

**The no-free-lunch problem:** There is no objectively "correct" segmentation. Different $k$, different features, and different algorithms produce different segments. The right segmentation is the one that produces actionable, interpretable groups that improve downstream decision-making.

---

## Practitioner Checkpoint

1. You run k-means with k=4. Customer A is assigned to Cluster 1. After updating cluster centres, is it possible for A to be assigned to Cluster 2 in the next iteration? Why or why not?

2. The elbow plot shows WCSS decreasing steadily from k=2 to k=8 with no clear elbow. What do you conclude, and what would you do?

3. Mean silhouette at k=4 is 0.19. A colleague says "the segmentation is useless." How do you respond?

4. You add `clv_12m` as a clustering feature (along with the five behavioural features). What is the likely effect on the resulting segments?

5. Cluster 3: mean CLV = \$420, mean $\hat{\tau} = 0.09$. Threshold = 0.133. Should you target Cluster 3 with the \$4 retention campaign?

---

### Answer Key

**Q1.** Yes — once the centres are updated, a customer near a boundary may become closer to a different centre. k-means is only guaranteed to converge to a state where no assignment changes are possible; it does not guarantee that any individual customer's assignment is stable across consecutive iterations.

**Q2.** A smooth WCSS decrease without an elbow suggests the data has weak cluster structure — customers are distributed relatively uniformly in the feature space without natural groupings. Examine the silhouette plot; it may still have a maximum. Consider whether the features chosen are appropriate for the segmentation goal, or whether the population genuinely lacks distinct subgroups.

**Q3.** A silhouette of 0.19 indicates weak separation — customers are not strongly concentrated in clusters. This does not make the segmentation useless; it means the cluster boundaries are soft and the labels should be treated as directional. For broad strategic decisions (allocating different marketing mixes to different groups), weak segmentation is often still useful. For individual targeting, rely on individual model scores rather than segment membership.

**Q4.** Including `clv_12m` as a clustering feature means CLV becomes one of the dimensions defining similarity. Clusters will tend to group customers by CLV as much as by behaviour. This may be appropriate if you want CLV-stratified segments, but it conflates the clustering with the profiling — CLV is typically more useful as a *profiling* variable than as a *clustering* feature.

**Q5.** No. Expected incremental profit = $0.09 \times 30 - 4 = 2.70 - 4 = -\$1.30$ per customer. The high CLV of \$420 is irrelevant to the targeting decision — CLV measures what a customer is worth, not how much they will respond to this specific campaign.

---

## Bibliography

- Hartigan, J. A., & Wong, M. A. (1979). Algorithm AS 136: A k-means clustering algorithm. *Journal of the Royal Statistical Society: Series C*, 28(1), 100–108. — The original Lloyd/k-means algorithm paper.
- Rousseeuw, P. J. (1987). Silhouettes: a graphical aid to the interpretation and validation of cluster analysis. *Journal of Computational and Applied Mathematics*, 20, 53–65. — The silhouette score.
- Wedel, M., & Kamakura, W. A. (2000). *Market Segmentation: Conceptual and Methodological Foundations* (2nd ed.). Kluwer Academic Publishers. — The standard marketing segmentation textbook.
- McCarthy, D., & Fader, P. S. (2020). Customer-based corporate valuation for publicly traded non-contractual firms. *Journal of Marketing Research*, 57(3), 521–543. — CLV-based segmentation in practice.
