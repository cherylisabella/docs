---
layout: default
title: Advanced Features
parent: vangogh
nav_order: 3
---

# Palette Analysis and Accessibility

## Palette Information

### vangogh_palette_info()

Get detailed palette information.

```r
vangogh_palette_info(add_metadata = FALSE)
```

**Parameters:**
- `add_metadata`: Include HCL color space data

**Example:**

```r
# Basic palette info
info <- vangogh_palette_info()

# With metadata
info_meta <- vangogh_palette_info(add_metadata = TRUE)
```

## Colour Vision Deficiency (CVD) Assessment

The `vangogh` package includes comprehensive tools to evaluate palette accessibility for users with colour blindness. All CVD simulations use the actively maintained `colorspace` package.

### Check Individual Palettes

Simulate how palettes appear under different types of colour vision deficiency:

```r
# Check a single palette with visual simulation
check_palette("StarryNight")

# Visualise with CVD simulation
viz_palette("Irises", colorblind = TRUE)

# Compare palettes with CVD simulation
compare_palettes(c("StarryNight", "SelfPortrait"), colorblind = TRUE)
```

### Comprehensive CVD Analysis

Get quantitative accessibility scores:

```r
# Analyse a single palette
check_vangogh_cvd("StarryNight")

# Batch check all palettes
cvd_results <- check_all_vangogh_cvd()

# Filter for CVD-safe palettes (score >= 70)
safe_palettes <- get_cvd_safe_palettes(threshold = 70)

# View pre-computed CVD scores (no colorspace required)
data(vangogh_cvd_scores)
head(vangogh_cvd_scores)
```

### CVD Reporting Tools

Generate accessibility reports and badges:

```r
# Get palette info with CVD scores
vangogh_palette_info_with_cvd()

# Print markdown badge for a palette
print_cvd_badge("StarryNight")

# Create summary table of all palettes
summarize_cvd_accessibility()
```

### Understanding CVD Scores

CVD assessment uses **CIELAB colour space** for perceptually uniform distance calculations:

- **Score ≥ 70**: Excellent accessibility
- **Score 50-69**: Good accessibility  
- **Score 30-49**: Fair accessibility
- **Score < 30**: Poor accessibility

The package evaluates three types of colour vision deficiency:
- **Deuteranopia** (red-green, most common)
- **Protanopia** (red-green)
- **Tritanopia** (blue-yellow, rare)

### Pre-computed Data

Access CVD scores without installing `colorspace`:

```r
# Load pre-computed CVD scores dataset
data(vangogh_cvd_scores)

# Filter by accessibility level
excellent_palettes <- subset(vangogh_cvd_scores, 
                              min_cvd_score >= 70)
```