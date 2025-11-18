---
layout: default
title: Creating Custom Detectors
parent: leakr
nav_order: 6
---

# Creating Custom Detectors

Extend leakr with domain-specific detectors.

## Detector Template

```r
# Define custom detector function
my_custom_detector <- function(audit_data, config) {
  data <- audit_data$data
  
  # Initialise issues dataframe
  issues <- data.frame(
    severity = character(),
    issue_type = character(),
    description = character(),
    suggested_fix = character(),
    stringsAsFactors = FALSE
  )
  
  # Example: Check for suspicious column names
  suspicious_cols <- grep(
    "_after_|_post_|_future_|_outcome_", 
    names(data), 
    value = TRUE,
    ignore.case = TRUE
  )
  
  if (length(suspicious_cols) > 0) {
    issues <- rbind(issues, data.frame(
      severity = "high",
      issue_type = "suspicious_features",
      description = paste(
        "Found", length(suspicious_cols), 
        "columns with temporal/outcome indicators:",
        paste(head(suspicious_cols, 3), collapse = ", ")
      ),
      suggested_fix = "Review feature engineering for temporal leakage and target-derived features",
      stringsAsFactors = FALSE
    ))
  }
  
  # Check for perfect correlations (potential proxy targets)
  if (!is.null(audit_data$target) && audit_data$target %in% names(data)) {
    numeric_features <- names(data)[sapply(data, is.numeric)]
    numeric_features <- setdiff(numeric_features, audit_data$target)
    
    if (length(numeric_features) > 0) {
      target_vals <- data[[audit_data$target]]
      
      for (feat in numeric_features) {
        if (all(!is.na(data[[feat]])) && all(!is.na(target_vals))) {
          correlation <- cor(data[[feat]], as.numeric(target_vals), 
                           use = "complete.obs")
          
          if (abs(correlation) > 0.95) {
            issues <- rbind(issues, data.frame(
              severity = "critical",
              issue_type = "perfect_correlation",
              description = paste(
                "Feature", feat, 
                "has near-perfect correlation with target:",
                round(correlation, 3)
              ),
              suggested_fix = paste("Remove", feat, "or investigate its derivation"),
              stringsAsFactors = FALSE
            ))
          }
        }
      }
    }
  }
  
  # Return standardised result
  list(
    issues = issues,
    evidence = list(
      suspicious_columns = suspicious_cols,
      n_suspicious = length(suspicious_cols)
    )
  )
}
```

## Register Detector

```r
# Register the detector
register_detector(
  name = "custom_feature_check",
  fun = my_custom_detector,
  description = "Checks for suspicious feature patterns and perfect correlations"
)

# Verify registration
"custom_feature_check" %in% list_registered_detectors()
#> [1] TRUE
```

## Use Custom Detector

```r
# Use in audit
report <- leakr_audit(
  data = my_data,
  target = "outcome",
  detectors = c("train_test_contamination", "custom_feature_check")
)
```

## Detector Requirements

Your custom detector function must:

1. Accept two parameters: `audit_data` and `config`
2. Return a list with `issues` and `evidence` components
3. Use standardized severity levels: "critical", "high", "medium", "low"
4. Include issue_type, description, and suggested_fix in issues dataframe