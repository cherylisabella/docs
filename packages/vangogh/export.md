---
layout: default
title: Data Export
parent: vangogh
nav_order: 5
---

# Data Export

Export palettes for use in other applications.

## vangogh_export()

```r
vangogh_export(
  file,
  format = "json",
  add_metadata = FALSE
)
```

**Parameters:**
- `file`: Output file path
- `format`: "json" or "csv"
- `add_metadata`: Include HCL metadata

**Example:**

```r
# Export to JSON
vangogh_export("my_palettes.json", format = "json", add_metadata = TRUE)

# Export to CSV
vangogh_export("my_palettes.csv", format = "csv")
```