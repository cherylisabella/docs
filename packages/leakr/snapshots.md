---
layout: default
title: Data Snapshots
parent: leakr
nav_order: 4
---

# Data Snapshots

Create reproducible snapshots with full provenance tracking.

## Create Snapshot

```r
# Basic snapshot
snapshot_path <- leakr_create_snapshot(
  data = my_data,
  snapshot_name = "baseline_data"
)

# Snapshot with metadata
snapshot_path <- leakr_create_snapshot(
  data = my_data,
  output_dir = "data/snapshots",
  snapshot_name = "experiment_v1",
  metadata = list(
    description = "Initial dataset for fraud detection model",
    experiment_id = "EXP001",
    notes = "Removed duplicates and handled missing values"
  )
)

# Fast hashing for large datasets
snapshot_path <- leakr_create_snapshot(
  data = large_dataset,
  sample_for_hash = TRUE  # Use sampling for faster hashing
)
```

## What's Saved

- `data.csv`: Data in CSV format
- `data.rds`: Data in R binary format (recommended)
- `metadata.json`: Complete metadata and provenance
- `README.md`: Human-readable documentation

## Metadata Includes

- Creation timestamp
- leakr and R versions
- Data dimensions and types
- Column names and types
- Data hash for integrity checking
- Preprocessing provenance
- Custom user metadata

## Load Snapshot

```r
# Load snapshot (RDS format - recommended)
data <- leakr_load_snapshot("data/snapshots/baseline_data")

# Load from CSV
data <- leakr_load_snapshot(
  "data/snapshots/baseline_data",
  format = "csv"
)

# Verify integrity
data <- leakr_load_snapshot(
  "data/snapshots/baseline_data",
  verify_integrity = TRUE  # Checks hash
)
```

## List Snapshots

```r
# List all snapshots with metadata
snapshots <- leakr_list_snapshots("data/snapshots")
print(snapshots)
#>              name             created  rows cols leakr_version size_mb
#> 1  experiment_v2 2024-01-15 14:30:22  5000   25         0.1.0    2.45
#> 2  experiment_v1 2024-01-14 09:15:00  5000   23         0.1.0    2.31
#> 3  baseline_data 2024-01-10 08:00:00  4800   20         0.1.0    2.10

# Simple listing without metadata
snapshots <- leakr_list_snapshots(
  "data/snapshots",
  include_metadata = FALSE
)
```