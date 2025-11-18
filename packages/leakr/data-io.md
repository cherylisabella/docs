---
layout: default
title: Data Import/Export
parent: leakr
nav_order: 2
---

# Data Import/Export

## Import Data

leakr provides flexible data import with automatic format detection and preprocessing.

```r
# Auto-detect format from extension
data <- leakr_import("path/to/data.csv")

# Specify format explicitly
data <- leakr_import("path/to/data.xlsx", format = "excel")

# Import with preprocessing
data <- leakr_import(
  "path/to/data.csv",
  preprocessing = list(
    remove_empty_rows = TRUE,
    remove_empty_cols = TRUE,
    clean_column_names = TRUE,
    handle_dates = TRUE,
    remove_constant_cols = TRUE,
    convert_character_to_factor = FALSE,
    max_factor_levels = 100
  )
)

# Quick import (minimal preprocessing)
data <- leakr_quick_import("path/to/data.csv")

# Import from data.frame
data <- leakr_import(iris)
```

## Supported Formats

| Format | Extensions | Required Package |
|--------|-----------|------------------|
| CSV | `.csv`, `.txt` | Base R |
| TSV | `.tsv` | Base R |
| Excel | `.xlsx`, `.xls` | `readxl` |
| RDS | `.rds` | Base R |
| JSON | `.json` | `jsonlite` |
| Parquet | `.parquet` | `arrow` |

## Advanced Import Options

```r
# CSV with specific encoding
data <- leakr_import(
  "data.csv",
  encoding = "latin1",
  verbose = TRUE
)

# Excel with specific sheet
data <- leakr_import(
  "workbook.xlsx",
  format = "excel",
  sheet = "Sheet2"  # or sheet = 2
)

# Handle large files (automatic streaming)
# Files > 100MB are automatically limited to first 10k rows
large_data <- leakr_import("very_large_file.csv")
attr(large_data, "leakr_streaming")  # TRUE if streamed
```

## Preprocessing Features

```r
# Default preprocessing (automatic)
data <- leakr_import(
  "data.csv",
  preprocessing = list(
    remove_empty_rows = TRUE,      # Remove all-NA rows
    remove_empty_cols = TRUE,      # Remove all-NA columns
    clean_column_names = TRUE,     # Standardise column names
    handle_dates = TRUE,           # Auto-detect and convert dates
    remove_constant_cols = FALSE   # Remove columns with single value
  )
)

# Check preprocessing provenance
attr(data, "leakr_provenance")
#> $clean_column_names
#> [1] TRUE
#> 
#> $removed_empty_rows
#> [1] 5
#> 
#> $removed_empty_cols
#> [1] 2
#> 
#> $handle_dates
#> [1] TRUE
```

**Date Detection**: Automatically detects and converts:
- ISO format (YYYY-MM-DD)
- UK format (DD/MM/YYYY)
- US format (MM/DD/YYYY)
- Unix timestamps (seconds and milliseconds)
- Long formats (DD Month YYYY)

## Export Data

```r
# Export to CSV
leakr_export_data(data, "output.csv", format = "csv")

# Export to Excel
leakr_export_data(data, "output.xlsx", format = "excel")

# Export to RDS (recommended for R)
leakr_export_data(data, "output.rds", format = "rds")

# Export to JSON
leakr_export_data(data, "output.json", format = "json")

# Export to Parquet
leakr_export_data(data, "output.parquet", format = "parquet")

# Silent export
leakr_export_data(data, "output.csv", verbose = FALSE)
```