# Exact train/test split manifests (awaiting actual experimental indices)

Each `seed_<seed>.csv` is currently **header-only**, NOT a released experimental split. Populate using the actual original preprocessing and split outputs, not a regenerated split. One row per rating interaction; `partition` is `train` or `test`. Interaction IDs must be stable and correspond to the version of the dataset used for the published tables. Never publish personal identifiers or raw location data without authorization.

The presence of these template files does **not** establish that split indices are available or reproducible.
