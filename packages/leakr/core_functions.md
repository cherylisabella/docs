---
layout: default
title: Core Functions
parent: leakr
nav_order: 1
---

# Core Functions

## leakr_audit()

The main function for comprehensive data leakage detection.

### Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `data` | data.frame | Dataset to audit | Required |
| `target` | character | Target variable name | `NULL` |
| `split` | character/vector | Split variable for train/test division | `NULL` |
| `id` | character | Unique identifier column | `NULL` |
| `detectors` | character vector | Detector names to run | `NULL` (all) |
| `config` | list | Configuration settings | `list()` |

### Configuration Options

```r
config <- list(
  sample_size = 50000,              # Max rows to analyse
  correlation_threshold = 0.8,      # Suspicious correlation threshold
  contamination_threshold = 0.1,    # Contamination tolerance
  numeric_severity = TRUE,          # Use numeric severity scores
  plot_results = FALSE,             # Generate diagnostic plots
  parallel = FALSE,                 # Enable parallel processing
  seed = 123                        # Random seed for reproducibility
)
```

### Basic Usage

```r
# Simple audit
report <- leakr_audit(data = my_data, target = "outcome")

# With train/test split
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  split = my_data$split_column
)

# With unique identifier
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  id = "customer_id"
)

# Select specific detectors
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  detectors = c("train_test_contamination", "target_leakage")
)

# List available detectors
list_registered_detectors()
#> [1] "file_format" "train_test_contamination" 
#> [3] "target_leakage" "duplication_detection"
```

### Report Structure

```r
# Access report components
report$summary        # Issues dataframe with severity, detector, description
report$evidence       # Detector-specific evidence (overlap_ids, leakage_indices, etc.)
report$meta          # Metadata (timestamps, config, sampling info)

# Report metadata
report$meta$n_detectors          # Number of detectors run
report$meta$n_issues            # Total issues found
report$meta$data_shape          # Analysed data dimensions
report$meta$original_data_shape # Original data dimensions
report$meta$was_sampled         # Whether data was sampled
report$meta$detectors_run       # Names of detectors executed
report$meta$timestamp           # Analysis timestamp
report$meta$config_used         # Configuration used
```

---

## Built-in Detectors

### 1. Train/Test Contamination Detector

Detects overlapping observations between training and testing sets using ID columns or row hashing.

```r
# Create detector
detector <- new_train_test_detector(threshold = 0.1)

# Run detector
result <- run_detector(
  detector = detector,
  data = my_data,
  split = split_vector,
  id = "customer_id"
)

# Access results
result$issues        # Issue information (severity, description)
result$evidence      # Evidence (overlap_ids or overlap_hashes)
result$detector      # Detector name
```

**Detection Strategy:**
- Compares IDs between train and test sets if `id` provided
- Falls back to row hashing for exact duplicate detection
- Severity levels: 
  - `critical`: ≥2 overlapping observations
  - `low`: <2 overlaps

**Example Output:**
```r
# Example with contamination
result$issues
#> $severity
#> [1] "critical"
#> 
#> $description
#> [1] "Detected 5 overlapping"

result$evidence$overlap_ids
#> [1] "CUST001" "CUST045" "CUST102" "CUST203" "CUST456"
```

### 2. Temporal Leakage Detector

Identifies temporal misalignment where test data points fall before or too close to the training period.

```r
# Create temporal detector
detector <- new_temporal_detector(
  time_col = "transaction_date",
  lookahead_window = 7  # days
)

# Run detector
result <- run_detector(
  detector = detector,
  data = my_data,
  split = split_vector
)

# Access temporal evidence
result$evidence$max_train_time      # Latest training timestamp
result$evidence$lookahead_window    # Configured lookahead period
result$evidence$n_leak             # Number of leakage violations
result$evidence$leakage_indices    # Indices of problematic rows
```

**Detection Strategy:**
- Validates that test data comes after training data plus a lookahead window
- Severity levels:
  - `high`: ≥20 violations
  - `medium`: 5-19 violations
  - `low`: <5 violations
- Supports `Date` and `POSIXt` time formats

**Example Output:**
```r
result$issues
#> $severity
#> [1] "medium"
#> 
#> $description
#> [1] "Detected 12 test samples with temporal leakage"

result$evidence
#> $max_train_time
#> [1] "2023-12-31"
#> 
#> $lookahead_window
#> Time difference of 7 days
#> 
#> $n_leak
#> [1] 12
```

### 3. Target Leakage Detector

Identifies features with suspicious correlations to the target variable.

*Automatically registered when package loads.*

```r
# Included in leakr_audit automatically
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  detectors = "target_leakage"
)
```

### 4. Duplication Detector

Finds exact and near-duplicate rows within datasets.

*Automatically registered when package loads.*

```r
# Included in leakr_audit automatically
report <- leakr_audit(
  data = my_data,
  detectors = "duplication_detection"
)
```

---

## Reporting and Visualisation

### Print Report

```r
# Print formatted report (automatically shows summary)
print(report)
```

### Detailed Summary

```r
# Detailed summary with configuration
leakr_summarise(report, top_n = 20, show_config = TRUE)

# Example output:
# Leakage Audit Report
# ===================
# Data shape: 1000 x 25
# Detectors run: train_test_contamination, target_leakage, duplication_detection
# Timestamp: 2024-01-15 14:30:22
# 
# Issues Summary:
#   Critical: 2
#   High: 1
#   Medium: 3
#   Low: 0
# 
# Top Issues:
#   detector                    severity  issue_type              description
#   train_test_contamination    critical  data_contamination      Detected 5 overlapping
#   target_leakage             high      suspicious_correlation   Feature 'total_purchases' ...
```

### Plot Results

```r
# Plot detector results
detector <- new_train_test_detector()
result <- run_detector(detector, my_data, split = my_data$split, id = "customer_id")

# Default plot
plot(result)

# Custom palette
plot(result, palette = "Set2")
```

### Automatic Plotting

```r
# Generate plots with audit
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  config = list(plot_results = TRUE)
)

# Access generated plots
report$plots
```

---

## Filtering and Analysing Results

### Filter by Severity

```r
# Filter by severity
critical_issues <- report$summary[report$summary$severity == "critical", ]

serious_issues <- report$summary[
  report$summary$severity %in% c("critical", "high"), 
]
```

### Filter by Detector

```r
# Filter by detector
train_test_issues <- report$summary[
  report$summary$detector == "train_test_contamination", 
]
```

### Sort by Severity Score

```r
# Sort by severity score (if numeric_severity = TRUE)
sorted_issues <- report$summary[
  order(-report$summary$severity_score), 
]

# Get top N most severe issues
top_5_issues <- head(sorted_issues, 5)
```

### Export Issues

```r
# Export issues for documentation
leakr_export_data(
  report$summary,
  "leakage_issues_report.csv"
)
```