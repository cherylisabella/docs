---
layout: default
title: Essential Functions
parent: vangogh
nav_order: 1
---

# Essential Functions

## vangogh_palette()

Extract colours from a Van Gogh palette.

```r
vangogh_palette(
  name,
  n = NULL,
  type = "discrete"
)
```

**Parameters:**
- `name`: Palette name (character)
- `n`: Number of colours to return
- `type`: "discrete" or "continuous"

**Example:**

```r
# Get all colors from a palette
vangogh_palette("StarryNight")

# Get specific number of colors
vangogh_palette("Irises", n = 3)

# Generate continuous palette
vangogh_palette("SelfPortrait", type = "continuous", n = 10)

# Use with base R plotting
plot(1:10, col = vangogh_palette("SelfPortrait"), pch = 19, cex = 2)
```

## vangogh_palettes

List of all available palettes.

```r
# View palette list
vangogh_palettes

# Get palette names
names(vangogh_palettes)
```

## vangogh_colors()

Get palette data as a data frame.

```r
vangogh_colors(add_metadata = FALSE)
```

**Parameters:**
- `add_metadata`: Include HCL color space metadata

**Example:**

```r
# Basic color data
colors <- vangogh_colors()

# With HCL metadata
colors_meta <- vangogh_colors(add_metadata = TRUE)
head(colors_meta)
```

## viz_palette()

Visualize a palette.

```r
viz_palette(
  name,
  colorblind = FALSE
)
```

**Parameters:**
- `name`: Palette name
- `colorblind`: Show color vision deficiency simulation

**Example:**

```r
# Basic visualization
viz_palette("StarryNight")

# With CVD simulation
viz_palette("Irises", colorblind = TRUE)
```

## compare_palettes()

Side-by-side palette comparison.

```r
compare_palettes(
  palettes,
  colorblind = FALSE
)
```

**Parameters:**
- `palettes`: Character vector of palette names
- `colorblind`: Show CVD simulation

**Example:**

```r
# Compare multiple palettes
compare_palettes(c("StarryNight", "SelfPortrait", "Irises"))

# With CVD simulation
compare_palettes(c("StarryNight", "SelfPortrait"), colorblind = TRUE)
```

## vangogh_suggest()

Get palette suggestions based on number of colours needed.

```r
vangogh_suggest(n)
```

**Parameters:**
- `n`: Number of colors needed

**Example:**

```r
# Get suggestions for 3 colors
vangogh_suggest(n = 3)

# Get suggestions for 5 colors
vangogh_suggest(n = 5)
```