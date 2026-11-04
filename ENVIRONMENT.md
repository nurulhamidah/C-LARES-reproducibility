# Computing Environment

All experiments reported in the manuscript (LARES and C-LARES, five runs, four datasets) were
executed in the environment below. The same machine and the same software versions were used
for every method, so the runtimes are directly comparable.

## Hardware

| Component | Specification |
|---|---|
| CPU | Intel Xeon Processor @ 2.10 GHz |
| Cores available | 2 |
| Memory (RAM) | 8.4 GB |
| GPU | not used (CPU only) |

## Software

| Package | Version |
|---|---|
| Operating system | Linux x86-64 (glibc 2.39) |
| Python | 3.13.16 |
| NumPy | 2.5.3 |
| SciPy | 1.18.1 |
| pandas | 3.0.5 |
| scikit-learn | 1.9.1 |

Standard-library modules used: `random` (data splitting), `time` (runtime measurement with
`time.perf_counter()`), `json`, and `collections`.

The software uses scikit-learn's `KMeans` (k-means++ initialisation) and `silhouette_score`,
SciPy's sparse matrices (`scipy.sparse.csr_matrix`) for the similarity computation, and
SciPy's `wilcoxon` for the signed-rank test.

## Files

- `environment.json`: the same information in machine-readable form.
- `requirements.txt`: package versions, which can be installed with
  `pip install -r requirements.txt`.

## Randomness

All random operations are seeded, so the runs are deterministic for a given seed
(seeds 91, 1, 2, 3, and 4):

- train/test split: `random.Random(seed)`;
- validation split: `random.Random(seed + 1000)`;
- K-Means: `random_state = seed`.
