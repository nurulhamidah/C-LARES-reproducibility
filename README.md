# C-LARES – Reproducibility Package

Supplementary material for the manuscript
*"Improving the Scalability of Location-Aware Tourism Recommendation Using a Clustering-Based Approach"*.

This package contains the configuration, random seeds, exact split indices, cluster assignments,
per-rating predictions, per-run results, runtime measurements, and the software/hardware
environment of the final experiments (LARES and C-LARES, number of clusters selected with the
Silhouette Coefficient, five random 80/20 splits).

**Not included:**

- **Implementation source code.** The method is fully specified in Section 3 of the manuscript.
- **Raw datasets.** The TripAdvisor, Yelp, and Booking.com data are subject to the platforms'
  terms of use. The files refer to ratings and users by **index only**: they contain no user
  names, item names, or rating values.

---

## 1. Folder structure

```
C-LARES_reproducibility/
├── README.md
├── environment.json                 hardware and software used for all runs
├── config/<dataset>.json            experimental configuration per dataset
└── <dataset>/                       tripadvisor35k, yelp10k, yelp64k, booking100k
    ├── dataset_statistics.json      ratings, users, items, sparsity after preprocessing
    ├── splits/split_seed<s>.csv     split label of every rating
    ├── clusters/clusters_seed<s>.csv  K-Means cluster of every user
    ├── predictions/predictions_seed<s>.csv  per-rating predictions of LARES and C-LARES
    └── results/                     per-run metrics, runtime, alpha and Silhouette curves
```

The seeds are 91, 1, 2, 3, and 4 (Run 1–Run 5).

## 2. Index definitions (preprocessing)

The indices can be rebuilt from the original data files as follows.

1. **Load ratings.** Read the user–item rating file (`{user: {item: rating}}`) and list all
   ratings in file order: users in file order, and items in file order within each user.
2. **Remove zero ratings.** Ratings equal to 0 are treated as missing and removed. For
   example, 948 ratings were removed from TripAdvisor 35K, leaving 23,510 ratings.
3. **Rescale ratings to 1–5.**
   - Booking.com 100K: all ratings are on a 1–10 scale and are rescaled as r' = 1 + 4(r − 1)/9.
   - TripAdvisor 35K: the file mixes both scales. Integer ratings 1–5 are kept as they are, and
     every other value (greater than 5 or non-integer) is treated as a 10-point score and
     rescaled with the same formula.
   - Yelp: ratings are already on a 1–5 scale and are not changed.
4. **Assign indices.**
   - `rating_index` is the 0-based position of a rating in the list obtained after steps 1–3.
   - `user_index` is the 0-based position of a user in the alphabetically sorted list of user
     IDs (Python string order).

All users have coordinates (lat, lon) in the location file. No user or item filtering is
applied (`MIN_RATINGS_PER_USER = 1`).

## 3. File descriptions

### `splits/split_seed<s>.csv`

Columns: `rating_index`, `split`.

| Label | Meaning |
|---|---|
| `train_inner` | training rating used to fit the model during the selection of α |
| `validation` | training rating held out (10%) to select α |
| `validation_excluded` | validation rating whose user has no `train_inner` rating, so it cannot be predicted and is not used |
| `test` | test rating that is evaluated (20% split) |
| `test_excluded` | test rating whose user has **no** training rating (cold-start user). It cannot be predicted by either method and is excluded from evaluation |

The **training set** used for the final similarity computation and prediction is the union
of `train_inner`, `validation`, and `validation_excluded`. Test ratings never enter the
training set, the clustering, or the selection of α.

Test ratings per seed: evaluated / excluded.

| Dataset | 91 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| TripAdvisor 35K | 2029 / 2673 | 2057 / 2645 | 2015 / 2687 | 2021 / 2681 | 1984 / 2718 |
| Yelp 10K | 904 / 1096 | 879 / 1121 | 866 / 1134 | 923 / 1077 | 896 / 1104 |
| Yelp 64K | 12759 / 167 | 12749 / 177 | 12731 / 195 | 12736 / 190 | 12719 / 207 |
| Booking.com 100K | 12234 / 2850 | 12265 / 2819 | 12258 / 2826 | 12248 / 2836 | 12282 / 2802 |

LARES and C-LARES are evaluated on **exactly the same** `test` ratings in each run.

### `clusters/clusters_seed<s>.csv`

Columns: `user_index`, `cluster`.

- Clustering uses user coordinates only (no rating information).
- Cluster 0 is the largest cluster.

### `predictions/predictions_seed<s>.csv`

One row per evaluated test rating.

| Column | Meaning |
|---|---|
| `rating_index` | rating, as defined in Section 2 |
| `cluster` | C-LARES cluster of the user |
| `fallback_LARES`, `fallback_C-LARES` | True if no neighbour was available and the item mean (training data) was used |
| `LARES_K<K>`, `C-LARES_K<K>` | predicted rating for K = 10, 20, …, 100 neighbours |

Together with the original data, these files are enough to recompute every MAE and RMSE value,
the per-cluster values, and the Wilcoxon signed-rank tests.

### `results/`

| File | Content |
|---|---|
| `results_per_seed_K.csv` | MAE, RMSE, Precision, Recall, F1 per method, seed, and K. Rows `wilcoxon_p(LARES vs C-LARES)` hold the p-value of the Wilcoxon signed-rank test on paired absolute errors (column `MAE`) |
| `results_per_cluster.csv` | MAE and RMSE per cluster: number of users, number of test ratings (`n_test`) |
| `summary_average_all_K.csv`, `summary_global.csv` | mean ± std over the five runs |
| `runtime_seconds.csv` | similarity and prediction time per method and seed (the `all` rows are the timed totals; per-cluster rows are diagnostics); number of similarity pairs |
| `runtime_theory_vs_measured.csv` | theoretical reduction 1 − Σ n_j² / n², reduction in computed pairs, measured time reduction |
| `alpha_validation.csv` | validation MAE for every α in the grid (`alpha_code` and `alpha_manuscript`, see Section 4) |
| `silhouette_scores.csv` | Silhouette score for k = 2–19 |
| `run_details_per_seed.json` | selected k, cluster sizes, selected α, and training and test sizes per seed |

## 4. Configuration (`config/<dataset>.json`)

| Parameter | Value |
|---|---|
| Split | random per-rating 80/20 (`TEST_RATIO` = 0.2); seeds 91, 1, 2, 3, 4; Python `random.Random(seed).shuffle` |
| Validation split | 10% of the training ratings (`VALIDATION_RATIO` = 0.1), shuffled with `random.Random(seed + 1000)` |
| α grid | 0.0, 0.1, …, 1.0; selected by minimum validation MAE averaged over K = 10–100 |
| μ (`TAU_KM`) | maximum distance between users, d_max (km): TripAdvisor 19,941.1; Yelp 10K 3,990.1; Yelp 64K 3,998.5; Booking 19,901.9 |
| ED-JM | as in LARES; normalised-ζ variant `cosine` |
| Prediction | mean-centred weighted average over the K most similar users who rated the target item and have similarity > 0 (Eq. 7), clipped to [1, 5]; for C-LARES only users of the same cluster are candidates |
| Fallback | if no candidate neighbour exists: item mean of the training data (global training mean if the item has no training rating) |
| Neighbours K | 10, 20, …, 100 |
| Clustering | K-Means on (lat, lon); `init = k-means++`, `n_init = 10`, `max_iter = 300`, `tol = 1e-4` (scikit-learn default), `random_state = seed`; users with identical coordinates are clustered as one weighted point |
| Choice of k | maximum Silhouette Coefficient over k = 2–19 (exact when there are ≤ 5000 distinct coordinates, otherwise a sample of 10,000 users) |
| Metrics | MAE and RMSE over all M evaluated test ratings, pooled across clusters (Eqs. 8–9) |

**Note on α.** In the code and in the result files, α weights the ED-JM term:
sim = α · ED-JM + (1 − α) · sim_loc. The manuscript uses the opposite convention, so
α_manuscript = 1 − α_code. Both values are given in `alpha_validation.csv`.

Selected k (identical for all seeds): TripAdvisor 35K, Yelp 10K, and Yelp 64K use k = 2;
Booking.com 100K uses k = 3.

## 5. Environment (`environment.json`)

- **Software:** Python 3.13.16, NumPy 2.5.3, pandas 3.0.5, scikit-learn 1.9.1, SciPy 1.18.1.
- **Hardware:** Linux x86-64, Intel Xeon @ 2.10 GHz (2 cores), 8.4 GB RAM. CPU only, no GPU.

**Runtime protocol:**

- Wall-clock time is measured with `time.perf_counter()`.
- LARES and C-LARES run on the same machine, in the same process, on the same training data.
- The reported time covers the computation of the user–user similarity matrix (all clusters of
  C-LARES are computed in one block-diagonal sparse product) and the prediction phase.
- K-Means and the Silhouette search are timed separately and reported as an offline step.
- Values are averaged over the five runs.
