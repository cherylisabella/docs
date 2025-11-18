---
layout: default
title: Best Practices
parent: leakr
nav_order: 7
---

# Best Practices

## 1. Run Audits Early and Often

```r
# ✓ Good: Audit immediately after data splitting
data$split <- create_split(data)
report <- leakr_audit(data, target = "outcome", split = "split")

# ✗ Bad: Audit after feature engineering and model training
```

## 2. Always Specify ID Columns

```r
# ✓ Good: Use unique identifiers when available
report <- leakr_audit(
  data = customer_data,
  target = "churn",
  split = "split",
  id = "customer_id"  # Explicit ID
)

# ✗ Bad: Rely on row hashing when IDs exist
report <- leakr_audit(
  data = customer_data,
  target = "churn",
  split = "split"
  # Missing id parameter
)
```

## 3. Configure Appropriate Thresholds

```r
# For financial/healthcare: Strict thresholds
report <- leakr_audit(
  data = medical_data,
  target = "diagnosis",
  config = list(
    correlation_threshold = 0.7,
    contamination_threshold = 0.01
  )
)

# For exploratory analysis: Lenient thresholds
report <- leakr_audit(
  data = exploration_data,
  target = "outcome",
  config = list(
    correlation_threshold = 0.95,
    contamination_threshold = 0.1
  )
)
```

## 4. Review Evidence, Not Just Summaries

```r
report <- leakr_audit(data, target = "outcome", split = "split", id = "id")

# Don't just look at summary
print(report$summary)

# ✓ Examine detailed evidence
report$evidence$train_test_contamination$overlap_ids
report$evidence$temporal$leakage_indices
report$evidence$duplication_detection$exact_duplicates
```

## 5. Use Snapshots for Reproducibility

```r
# Before any analysis
snapshot_path <- leakr_create_snapshot(
  raw_data,
  snapshot_name = "raw_data_v1",
  metadata = list(source = "production_db", date = Sys.Date())
)

# After preprocessing
snapshot_path <- leakr_create_snapshot(
  clean_data,
  snapshot_name = "clean_data_v1",
  metadata = list(preprocessing = "removed nulls, normalized")
)

# Before modeling
snapshot_path <- leakr_create_snapshot(
  model_ready_data,
  snapshot_name = "model_ready_v1",
  metadata = list(audit_passed = TRUE, issues = 0)
)
```

## 6. Document Leakage Checks

```r
# Export detailed report
leakr_export_data(report$summary, "audit_report.csv")

# Save configuration for reproducibility
config_used <- report$meta$config_used
saveRDS(config_used, "audit_config.rds")

# Create audit log
audit_log <- data.frame(
  timestamp = report$meta$timestamp,
  n_detectors = report$meta$n_detectors,
  n_issues = report$meta$n_issues,
  critical_issues = sum(report$summary$severity == "critical"),
  high_issues = sum(report$summary$severity == "high"),
  data_shape = paste(report$meta$data_shape, collapse = "x")
)
write.csv(audit_log, "audit_log.csv", row.names = FALSE)
```

## 7. Integrate into CI/CD

```r
# In your testing script
test_that("no data leakage in production pipeline", {
  data <- load_production_data()
  
  report <- leakr_audit(
    data = data,
    target = "target_var",
    split = "split",
    id = "id",
    config = list(
      correlation_threshold = 0.85,
      contamination_threshold = 0.05
    )
  )
  
  # Fail test if critical issues found
  critical_count <- sum(report$summary$severity == "critical")
  expect_equal(critical_count, 0, 
               info = paste("Found", critical_count, "critical leakage issues"))
})
```

## 8. Handle Temporal Data Carefully

```r
# ✓ Good: Proper temporal validation
detector <- new_temporal_detector(
  time_col = "date",
  lookahead_window = 30  # Allow 30-day gap
)

result <- run_detector(
  detector,
  timeseries_data,
  split = timeseries_data$split
)

# Verify temporal ordering
max_train <- max(timeseries_data$date[timeseries_data$split == "train"])
min_test <- min(timeseries_data$date[timeseries_data$split == "test"])
stopifnot(min_test > max_train)
```

## Troubleshooting

### Q: Why is my audit showing no issues when I know there's leakage?

**A:** Check these common causes:

```r
# 1. Verify split labels
table(data$split)  # Should show "train" and "test"

# 2. Check if target is specified
report <- leakr_audit(data, target = "outcome")  # Don't forget target!

# 3. Lower thresholds for more sensitivity
report <- leakr_audit(
  data,
  target = "outcome",
  config = list(
    correlation_threshold = 0.7,      # Lower = more sensitive
    contamination_threshold = 0.05
  )
)

# 4. Ensure ID column is correct
names(data)  # Verify column names
report <- leakr_audit(data, id = "correct_id_column")
```

### Q: How do I interpret severity scores?

**A:** Severity levels indicate risk:

```r
# When numeric_severity = TRUE
report$summary$severity_score
# 4 = critical: Immediate action required
# 3 = high: Address before production
# 2 = medium: Investigate and monitor
# 1 = low: Minor issue or false positive

# Filter by severity
critical <- report$summary[report$summary$severity_score >= 4, ]
actionable <- report$summary[report$summary$severity_score >= 3, ]
```

### Q: Can I use custom split labels?

**A:** Currently requires "train"/"test". Convert your labels:

```r
# Convert custom labels
data$split <- ifelse(data$fold == 1, "train", "test")

# Or use a mapping
split_map <- c("training" = "train", "validation" = "test", "testing" = "test")
data$split <- split_map[data$original_split]
```

### Q: What if I get "time_col not found" errors?

**A:** Verify column existence and type:

```r
# Check column names
names(data)

# Check column type
class(data$date_column)  # Should be Date or POSIXt

# Convert if needed
data$date_column <- as.Date(data$date_column)

# Then create detector
detector <- new_temporal_detector(time_col = "date_column")
```

### Q: How do I handle large files that won't import?

**A:** Use streaming or manual chunking:

```r
# Large files are automatically streamed (first 10k rows)
data <- leakr_import("very_large_file.csv")
attr(data, "leakr_streaming")  # Check if streamed

# Or manually limit
data <- leakr_import("large_file.csv", nrows = 50000)

# For full analysis, use sampling in audit
report <- leakr_audit(
  full_data,
  config = list(sample_size = 100000)
)
```

### Q: Why do I get package dependency errors?

**A:** Install required packages for specific formats:

```r
# For Excel files
install.packages("readxl")

# For JSON
install.packages("jsonlite")

# For Parquet
install.packages("arrow")

# For better CSV performance
install.packages("data.table")

# For mlr3 integration
install.packages("mlr3")

# For tidymodels integration
install.packages("tidymodels")
```

## Advanced Configuration

### Large Dataset Handling

```r
# Automatic stratified sampling
large_report <- leakr_audit(
  data = large_dataset,  # e.g., 1M rows
  target = "outcome",
  config = list(
    sample_size = 100000,  # Sample to 100k rows
    seed = 42
  )
)

# Check if sampling occurred
if (large_report$meta$was_sampled) {
  cat("Data was sampled:\n")
  cat("  Original:", large_report$meta$original_data_shape[1], "rows\n")
  cat("  Analysed:", large_report$meta$data_shape[1], "rows\n")
}
```

### Parallel Processing

```r
# Enable parallel processing for large datasets
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  config = list(
    parallel = TRUE,
    sample_size = 200000
  )
)
```

### Custom Thresholds

```r
# Strict configuration
strict_report <- leakr_audit(
  data = my_data,
  target = "outcome",
  config = list(
    correlation_threshold = 0.7,       # More sensitive
    contamination_threshold = 0.05,    # Lower tolerance
    numeric_severity = TRUE            # Include severity scores
  )
)

# Lenient configuration (for exploratory analysis)
lenient_report <- leakr_audit(
  data = my_data,
  target = "outcome",
  config = list(
    correlation_threshold = 0.95,      # Less sensitive
    contamination_threshold = 0.2      # Higher tolerance
  )
)
```