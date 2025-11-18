---
layout: default
title: leakr
nav_order: 2
has_children: true
permalink: /packages/leakr/
---

# `leakr` : Universal Data Leakage Detector for R <img src="/docs/assets/images/leakr.png" align="right" width="175" />

[![CRAN version](https://www.r-pkg.org/badges/version/leakr)](https://CRAN.R-project.org/package=leakr)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Lifecycle: stable](https://img.shields.io/badge/lifecycle-stable-brightgreen.svg)](https://lifecycle.r-lib.org/articles/stages.html#stable)
[![Total Downloads](https://cranlogs.r-pkg.org/badges/grand-total/leakr)](https://cran.r-project.org/package=leakr)
[![Cite with Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.1343417-1073c8?)](https://doi.org/10.5281/zenodo.17511513)

A comprehensive R package for detecting and diagnosing data leakage in machine learning workflows.

## Overview

Data leakage undermines model validity by allowing information from outside the training dataset to influence model development. `leakr` provides an automated, registry-based detection system that identifies:

- **Train/test contamination**: Overlapping observations between training and testing sets
- **Target leakage**: Features with suspicious relationships to the target variable
- **Temporal leakage**: Using future information to predict the past in time-series data
- **Duplicate records**: Exact and near-duplicate rows that can inflate performance metrics

**Key Features:**
- Automated leakage detection across multiple dimensions
- Rich visualisation and reporting capabilities
- Seamless integration with tidymodels, caret, and mlr3
- Comprehensive I/O support (CSV, Excel, JSON, Parquet, RDS)
- Data snapshots for reproducibility
- Optimised for large datasets with intelligent sampling

## Installation

Install the development version from GitHub:

```r
# install.packages("devtools")
devtools::install_github("cherylisabella/leakr")
```

Install from CRAN:

```r
install.packages("leakr")
```

## Quick Start

```r
library(leakr)

# Basic audit on iris dataset
report <- leakr_audit(iris, target = "Species")
print(report)

# Audit with custom configuration
report <- leakr_audit(
  data = iris,
  target = "Species",
  config = list(
    sample_size = 50000,
    correlation_threshold = 0.8,
    contamination_threshold = 0.1,
    plot_results = TRUE
  )
)

# View detailed summary
leakr_summarise(report, top_n = 10, show_config = TRUE)
```

## Table of Contents

- [Core Functions](core-functions.html) - Main audit function and detectors
- [Data Import/Export](data-io.html) - Import/export and preprocessing
- [ML Framework Integration](ml-integration.html) - tidymodels, caret, mlr3
- [Data Snapshots](snapshots.html) - Reproducible data snapshots
- [Real-World Examples](examples.html) - Real-world use cases
- [Custom Detectors](custom-detectors.html) - Create your own detectors
- [Best Practices](best-practices.html) - Tips and troubleshooting

## Getting Help

- [GitHub Issues](https://github.com/cherylisabella/leakr/issues)
- [GitHub Repository](https://github.com/cherylisabella/leakr)