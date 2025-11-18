---
layout: default
title: vangogh
nav_order: 3
has_children: true
permalink: /packages/vangogh/
---
# `vangogh` <img src="/docs/assets/images/vangogh.png" align="right" width="155" />

[![CRAN version](https://www.r-pkg.org/badges/version/vangogh)](https://CRAN.R-project.org/package=vangogh)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Lifecycle: stable](https://img.shields.io/badge/lifecycle-stable-brightgreen.svg)](https://lifecycle.r-lib.org/articles/stages.html#stable)
[![Total Downloads](https://cranlogs.r-pkg.org/badges/grand-total/vangogh)](https://cran.r-project.org/package=vangogh)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.17511534.svg)](https://doi.org/10.5281/zenodo.17511534)

An R package for painterly colour palettes inspired by Vincent van Gogh's artworks. The `vangogh` package provides ggplot2-compatible colour palettes derived from the artist's most iconic paintings, with tools for accessibility, visualisation, and data export.

## Installation

Install the released version from CRAN:

```r
install.packages("vangogh")
```

## Quick Start

### Basic Palette Usage

```r
library(vangogh)

# View available palettes
names(vangogh_palettes)

# Preview a palette
viz_palette("StarryNight")

# Get colours from a palette
vangogh_palette("StarryNight")

# Get palette as data frame
vangogh_colors()

# Use specific number of colours
vangogh_palette("Irises", n = 3)

# Generate continuous palette
vangogh_palette("SelfPortrait", type = "continuous", n = 10)

# Use with base R plotting
plot(1:10, col = vangogh_palette("SelfPortrait"), pch = 19, cex = 2)

# Use with ggplot2
library(ggplot2)
ggplot(iris, aes(Sepal.Length, Sepal.Width, color = Species)) +
  geom_point(size = 4) +
  scale_color_vangogh("StarryNight")
```

## Available Palettes

| StarryNight | StarryRhone | SelfPortrait | CafeTerrace | Eglise |
|---|---|---|---|---|
| ![](/docs/assets/images/paintings/starrynight2.png) | ![](/docs/assets/images/paintings/rhone1.png) | ![](/docs/assets/images/paintings/selfp.png) | ![](/docs/assets/images/paintings/cafeterrace.png) | ![](/docs/assets/images/paintings/eglise.png) |

| Irises | SunflowersMunich | SunflowersLondon | Rest | Bedroom |
|---|---|---|---|---|
| ![](/docs/assets/images/paintings/irises.png) | ![](/docs/assets/images/paintings/sunflowersm.png) | ![](/docs/assets/images/paintings/sunflowersl.png) | ![](/docs/assets/images/paintings/rest.png) | ![](/docs/assets/images/paintings/bedroom.png) |

| CafeDeNuit | Chaise | Shoes | Landscape | Cypresses |
|---|---|---|---|---|
| ![](/docs/assets/images/paintings/cafedenuit.png) | ![](/docs/assets/images/paintings/chaise.png) | ![](/docs/assets/images/paintings/shoes.png) | ![](/docs/assets/images/paintings/landscape.png) | ![](/docs/assets/images/paintings/cypresses.png) |

## Table of Contents

- [Essential Functions](functions.html) - Core palette functions
- [ggplot2 Integration](ggplot2.html) - Scales and themes
- [Advanced Features](analysis.html) - CVD Accessibility and metadata
- [Visualisation Gallery](gallery.html) - Example plots
- [Data Export](export.html) - Export palettes

## Package Philosophy

Van Gogh's use of colour was revolutionary, employing vivid hues and bold contrasts to convey emotion and movement. This package aims to bring that same artistic sensibility to data visualisation, while maintaining the technical rigor required for clear, accessible communication of information.

Each palette contains five carefully selected colours derived from the original paintings, balancing aesthetic appeal with practical considerations for data visualisation.

## Getting Help

- [GitHub Issues](https://github.com/cherylisabella/vangogh/issues)
- [GitHub Repository](https://github.com/cherylisabella/vangogh)