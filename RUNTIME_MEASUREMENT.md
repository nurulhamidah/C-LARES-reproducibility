# Runtime Measurement

This folder documents how the computation time of LARES and C-LARES was measured and
reports the measured values for all datasets. Both methods use k selected by the Silhouette
Coefficient, and every value is averaged over five random 80/20 splits with seeds 91, 1, 2,
3, and 4.

## 1. Measurement protocol

| Item | Setting |
|---|---|
| Timer | wall-clock time, Python `time.perf_counter()` |
| Hardware / software | see `../environment/` (the same machine is used for all runs) |
| Execution | CPU only, single process; LARES and C-LARES run one after the other in the same process, on identical training data and identical test ratings |
| Repetitions | five runs (seeds 91, 1, 2, 3, 4); mean ± standard deviation is reported |
| Warm-up / caching | data are loaded and indexed before timing starts; no result is cached between methods, and every similarity matrix is computed from scratch |
| **Similarity time** (main measure) | time to compute the user–user similarity matrix (ED-JM + location similarity) from the training ratings. LARES uses all user pairs. For C-LARES, all clusters are computed in one block-diagonal sparse product, so only same-cluster pairs are formed |
| **Prediction time** | time to predict all test ratings for K = 10, 20, …, 100 |
| **Clustering time** (offline) | K-Means on user coordinates, and the Silhouette search over k = 2–19. Reported separately and **not** included in the similarity or total time, because clustering is performed once, before recommendation |
| Excluded from timing | data loading, the train/test split, the selection of α on the validation set, and the computation of the metrics |

**Theoretical reduction:** 1 − Σ_j n_j² / n², where n_j is the number of users in cluster j and
n is the total number of users. This follows from the cost O(Σ_j n_j²) of C-LARES compared
with O(n²) for LARES.

**Measured reduction:** 1 − t_sim(C-LARES) / t_sim(LARES), computed per run and then averaged.

**Computed pairs reduction:** the reduction in the number of user pairs that actually receive
a similarity value. Only pairs with at least one co-rated item receive a value, in both
methods.

## 2. Summary (`runtime_summary.csv`)

All times are in seconds, mean over five runs.

| Dataset | k (cluster sizes) | LARES similarity | C-LARES similarity | LARES total | C-LARES total | Theoretical reduction | Measured reduction |
|---|---|---|---|---|---|---|---|
| TripAdvisor 35K | 2 (9303 / 5606) | 0.552 ± 0.114 | 0.229 ± 0.017 | 0.650 | 0.318 | 46.9% | 57.1% |
| Yelp 10K | 2 (5661 / 1283) | 0.030 ± 0.001 | 0.025 ± 0.002 | 0.061 | 0.054 | 30.1% | 17.2% |
| Yelp 64K | 2 (11172 / 2496) | 2.085 ± 0.356 | 1.496 ± 0.533 | 2.676 | 2.157 | 29.8% | 26.6% |
| Booking.com 100K | 3 (14622 / 4624 / 2083) | 9.304 ± 0.794 | 3.458 ± 0.287 | 10.230 | 4.215 | 47.4% | 62.7% |

*Total = similarity + prediction.*

Clustering time (offline, mean over five runs):

| Dataset | Final K-Means (s) | Silhouette search, k = 2–19 (s) |
|---|---|---|
| TripAdvisor 35K | 0.029 | 7.7 |
| Yelp 10K | 0.014 | 20.1 |
| Yelp 64K | 0.133 | 41.9 |
| Booking.com 100K | 0.005 | 0.2 |

Users who share the same coordinates are clustered as one weighted point. The clustering time
therefore depends on the number of distinct locations rather than the number of users. Most
users in Booking.com 100K share a small number of locations, which is why its clustering is
the fastest.

## 3. Files

| File | Content |
|---|---|
| `runtime_summary.csv` | the summary above, together with the computed-pairs reduction and the prediction times |
| `per_run/runtime_<dataset>.csv` | every timed stage per seed and method: `similarity`, `prediction`, `silhouette_selection(k=2..19)`, `final_clustering`. Rows with `cluster = all` are the timed totals. Rows with a cluster number are per-cluster diagnostics: each cluster is computed on its own, and these rows are not part of the total |
| `per_run/theory_vs_measured_<dataset>.csv` | theoretical reduction, computed-pairs reduction, measured time reduction, and similarity times, per seed |

## 4. Notes for interpretation

- **Big-O ignores constants.** O(Σ_j n_j²) describes how the cost grows with the cluster
  sizes. It ignores constant factors, memory effects, and fixed overheads.
- **Measured reduction above the theoretical value.** On TripAdvisor 35K and Booking.com 100K,
  the measured reduction is larger than the theoretical one. A likely reason is that C-LARES
  works on smaller similarity blocks, which use memory more efficiently than the full n × n
  structure of LARES. This effect was not measured directly.
- **Measured reduction below the theoretical value on Yelp 10K.** The computation takes only
  about 0.03 s, so fixed overheads dominate the runtime.
