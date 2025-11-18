---
layout: default
title: ML Framework Integration
parent: leakr
nav_order: 3
---

# ML Framework Integration

## From mlr3

```r
library(mlr3)
library(leakr)

# Create mlr3 task
task <- TaskClassif$new(
  id = "iris_task",
  backend = iris,
  target = "Species"
)

# Convert to leakr format
leakr_data <- leakr_from_mlr3(task, include_target = TRUE)

# Access components
leakr_data$data          # Full dataset
leakr_data$target        # Target variable
leakr_data$target_name   # Target column name
leakr_data$feature_names # Feature names
leakr_data$task_type     # Task type (e.g., "TaskClassif")

# Run audit
report <- leakr_audit(
  data = leakr_data$data,
  target = leakr_data$target_name
)
```

## From caret

```r
library(caret)
library(leakr)

# Train model with caret
train_obj <- train(
  Species ~ .,
  data = iris,
  method = "rf"
)

# Convert to leakr format
leakr_data <- leakr_from_caret(
  train_obj = train_obj,
  original_data = iris,
  target_name = "Species"
)

# Access components
leakr_data$data          # Training data
leakr_data$method        # Model method
leakr_data$final_model   # Trained model

# Run audit
report <- leakr_audit(
  data = leakr_data$data,
  target = leakr_data$target_name
)
```

## From tidymodels

```r
library(tidymodels)
library(leakr)

# Create workflow
workflow <- workflow() %>%
  add_formula(Species ~ .) %>%
  add_model(decision_tree() %>% set_engine("rpart") %>% set_mode("classification"))

# Convert to leakr format
leakr_data <- leakr_from_tidymodels(workflow, data = iris)

# Access components
leakr_data$data                   # Dataset
leakr_data$has_preprocessor       # Has preprocessing?
leakr_data$preprocessor_type      # Preprocessor type
leakr_data$model_spec            # Model specification

# Run audit
report <- leakr_audit(
  data = leakr_data$data,
  target = "Species"
)
```

## Pipeline Integration Example

```r
library(tidymodels)
library(leakr)

# Safe preprocessing with leakage checks
safe_preprocess <- function(data, target_var, split_var) {
  # Run leakage audit
  audit_report <- leakr_audit(
    data = data,
    target = target_var,
    split = split_var,
    config = list(correlation_threshold = 0.9)
  )
  
  # Check for critical issues
  critical_issues <- audit_report$summary[
    audit_report$summary$severity == "critical", 
  ]
  
  if (nrow(critical_issues) > 0) {
    stop("Critical data leakage detected! Fix before modeling:\n",
         paste(critical_issues$description, collapse = "\n"))
  }
  
  # Warn on high severity
  high_issues <- audit_report$summary[
    audit_report$summary$severity == "high", 
  ]
  
  if (nrow(high_issues) > 0) {
    warning("High severity leakage detected:\n",
            paste(high_issues$description, collapse = "\n"))
  }
  
  return(audit_report)
}

# Use in workflow
data$split <- sample(c("train", "test"), nrow(data), replace = TRUE, prob = c(0.8, 0.2))
leakage_check <- safe_preprocess(data, "outcome", "split")

# Proceed only if no critical issues
if (sum(leakage_check$summary$severity == "critical") == 0) {
  # Continue with modeling
  message("✓ No critical leakage detected - safe to proceed")
}
```