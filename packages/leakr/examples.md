---
layout: default
title: Real-World Examples
parent: leakr
nav_order: 5
---

# Real-World Examples

## Example 1: Credit Card Fraud Detection

```r
library(leakr)

# Simulate credit card transaction data
set.seed(42)
fraud_data <- data.frame(
  transaction_id = 1:1000,
  customer_id = sample(1:200, 1000, replace = TRUE),
  amount = round(rnorm(1000, 100, 50), 2),
  transaction_date = seq(as.Date("2023-01-01"), by = "day", length.out = 1000),
  merchant_category = sample(c("retail", "grocery", "online", "restaurant"), 1000, replace = TRUE),
  is_fraud = sample(0:1, 1000, replace = TRUE, prob = c(0.98, 0.02)),
  split = c(rep("train", 700), rep("test", 300))
)

# Comprehensive audit
report <- leakr_audit(
  data = fraud_data,
  target = "is_fraud",
  split = "split",
  id = "transaction_id",
  config = list(
    correlation_threshold = 0.9,
    plot_results = TRUE
  )
)

# Review findings
leakr_summarise(report, top_n = 10)

# Check for temporal issues
temporal_detector <- new_temporal_detector(
  time_col = "transaction_date",
  lookahead_window = 7  # 7-day lookahead
)

temporal_result <- run_detector(
  temporal_detector,
  fraud_data,
  split = fraud_data$split
)

# Examine temporal evidence
cat("Max train date:", as.character(temporal_result$evidence$max_train_time), "\n")
cat("Leakage count:", temporal_result$evidence$n_leak, "\n")
cat("Severity:", temporal_result$issues$severity, "\n")

# Create snapshot for reproducibility
leakr_create_snapshot(
  fraud_data,
  snapshot_name = "fraud_detection_v1",
  metadata = list(
    project = "Fraud Detection",
    version = "1.0",
    split_date = "2023-10-01"
  )
)
```

## Example 2: Customer Churn Prediction

```r
# Simulate customer data
set.seed(123)
n_customers <- 5000

customer_data <- data.frame(
  customer_id = 1:n_customers,
  tenure_months = sample(1:60, n_customers, replace = TRUE),
  monthly_charges = round(rnorm(n_customers, 70, 20), 2),
  total_charges = NA,  # Potential leakage!
  contract_type = sample(c("month-to-month", "one_year", "two_year"), 
                        n_customers, replace = TRUE),
  churned = sample(0:1, n_customers, replace = TRUE, prob = c(0.7, 0.3))
)

# Create total_charges (potential leakage!)
customer_data$total_charges <- 
  customer_data$tenure_months * customer_data$monthly_charges

# Create train/test split
train_idx <- sample(1:n_customers, 0.7 * n_customers)
customer_data$split <- "test"
customer_data$split[train_idx] <- "train"

# Import and preprocess
customer_data_clean <- leakr_import(
  customer_data,
  preprocessing = list(
    clean_column_names = TRUE,
    handle_dates = TRUE,
    remove_constant_cols = TRUE
  )
)

# Audit for leakage
report <- leakr_audit(
  data = customer_data_clean,
  target = "churned",
  split = "split",
  id = "customer_id",
  detectors = c("train_test_contamination", "target_leakage"),
  config = list(correlation_threshold = 0.85)
)

# Check for ID-based contamination
contamination_check <- new_train_test_detector(threshold = 0.05)
contamination_result <- run_detector(
  contamination_check,
  customer_data_clean,
  split = customer_data_clean$split,
  id = "customer_id"
)

# View evidence
if (length(contamination_result$evidence$overlap_ids) > 0) {
  cat("⚠ Found overlapping customer IDs:\n")
  print(head(contamination_result$evidence$overlap_ids))
} else {
  cat("✓ No customer ID overlap detected\n")
}

# Export cleaned data
leakr_export_data(
  customer_data_clean[, !names(customer_data_clean) %in% c("split")],
  "customer_churn_clean.csv"
)
```

## Example 3: Time Series Forecasting

```r
# Simulate time series data
dates <- seq(as.Date("2022-01-01"), as.Date("2023-12-31"), by = "day")
n <- length(dates)

set.seed(456)
timeseries_data <- data.frame(
  date = dates,
  sales = round(1000 + cumsum(rnorm(n, 2, 50))),
  temperature = round(rnorm(n, 20, 5), 1),
  day_of_week = weekdays(dates),
  is_holiday = sample(0:1, n, replace = TRUE, prob = c(0.95, 0.05))
)

# Add lagged features (check for leakage!)
timeseries_data$lag_1_sales <- c(NA, head(timeseries_data$sales, -1))
timeseries_data$lag_7_sales <- c(rep(NA, 7), head(timeseries_data$sales, -7))

# Create temporal split
timeseries_data$split <- ifelse(
  timeseries_data$date < as.Date("2023-10-01"),
  "train",
  "test"
)

# Import and preprocess
ts_clean <- leakr_import(
  timeseries_data,
  preprocessing = list(
    handle_dates = TRUE,
    clean_column_names = TRUE
  )
)

# Check for temporal leakage
temporal_det <- new_temporal_detector(
  time_col = "date",
  lookahead_window = 30  # 30-day lookahead
)

temporal_result <- run_detector(
  temporal_det,
  ts_clean,
  split = ts_clean$split
)

# Comprehensive audit
report <- leakr_audit(
  data = ts_clean,
  target = "sales",
  split = "split",
  config = list(
    correlation_threshold = 0.9,
    plot_results = FALSE
  )
)

# Print temporal analysis
cat("\n=== Temporal Leakage Analysis ===\n")
cat("Max train date:", as.character(temporal_result$evidence$max_train_time), "\n")
cat("Lookahead window:", temporal_result$evidence$lookahead_window, "\n")
cat("Leakage violations:", temporal_result$evidence$n_leak, "\n")
cat("Severity:", temporal_result$issues$severity, "\n")

# Create versioned snapshot
leakr_create_snapshot(
  ts_clean,
  snapshot_name = paste0("timeseries_", format(Sys.Date(), "%Y%m%d")),
  metadata = list(
    date_range = paste(range(ts_clean$date), collapse = " to "),
    features = setdiff(names(ts_clean), c("date", "sales", "split")),
    train_samples = sum(ts_clean$split == "train"),
    test_samples = sum(ts_clean$split == "test")
  )
)
```

## Example 4: Multi-format Data Pipeline

```r
# Complete data pipeline with leakr

# 1. Import from various sources
csv_data <- leakr_import("data/raw/customers.csv")
excel_data <- leakr_import("data/raw/transactions.xlsx", sheet = "2023")
json_data <- leakr_import("data/raw/metadata.json")

# 2. Merge datasets
merged_data <- merge(csv_data, excel_data, by = "customer_id")

# 3. Preprocess with full tracking
clean_data <- leakr_import(
  merged_data,
  preprocessing = list(
    remove_empty_rows = TRUE,
    remove_empty_cols = TRUE,
    clean_column_names = TRUE,
    handle_dates = TRUE,
    remove_constant_cols = TRUE,
    convert_character_to_factor = TRUE,
    max_factor_levels = 50
  )
)

# Check provenance
provenance <- attr(clean_data, "leakr_provenance")
cat("Removed empty rows:", provenance$removed_empty_rows, "\n")
cat("Removed empty cols:", provenance$removed_empty_cols, "\n")

# 4. Create train/test split
set.seed(42)
clean_data$split <- sample(
  c("train", "test"),
  nrow(clean_data),
  replace = TRUE,
  prob = c(0.8, 0.2)
)

# 5. Audit for leakage
report <- leakr_audit(
  data = clean_data,
  target = "target_variable",
  split = "split",
  id = "customer_id",
  config = list(
    correlation_threshold = 0.85,
    contamination_threshold = 0.05,
    plot_results = TRUE
  )
)

# 6. Create snapshot before modeling
snapshot_path <- leakr_create_snapshot(
  clean_data,
  snapshot_name = "model_ready_data",
  metadata = list(
    source_files = c("customers.csv", "transactions.xlsx"),
    preprocessing_steps = names(provenance),
    audit_timestamp = report$meta$timestamp,
    total_issues = report$meta$n_issues
  )
)

# 7. Export in multiple formats for downstream use
leakr_export_data(clean_data, "output/model_data.csv", format = "csv")
leakr_export_data(clean_data, "output/model_data.rds", format = "rds")
leakr_export_data(clean_data, "output/model_data.parquet", format = "parquet")

# 8. List all snapshots for audit trail
snapshots <- leakr_list_snapshots()
print(snapshots)
```