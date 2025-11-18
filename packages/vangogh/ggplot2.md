---
layout: default
title: ggplot2 Integration
parent: vangogh
nav_order: 2
---

# ggplot2 Integration

## Colour and Fill Scales

### scale_color_vangogh()

ggplot2 colour scale for Van Gogh palettes.

```r
scale_color_vangogh(
  palette = "StarryNight",
  type = "discrete",
  reverse = FALSE,
  ...
)
```

**Example:**

```r
library(ggplot2)

# Colour scale for categorical data
ggplot(iris, aes(x = Sepal.Length, y = Sepal.Width, color = Species)) +
  geom_point(size = 3) +
  scale_color_vangogh("StarryNight")

# Continuous colour scale
ggplot(faithfuld, aes(waiting, eruptions, color = density)) +
  geom_point() +
  scale_color_vangogh("Irises", type = "continuous")
```

### scale_fill_vangogh()

ggplot2 fill scale for Van Gogh palettes.

```r
scale_fill_vangogh(
  palette = "StarryNight",
  type = "discrete",
  reverse = FALSE,
  ...
)
```

**Example:**

```r
# Fill scale for categorical data
ggplot(mpg, aes(x = class, fill = drv)) +
  geom_bar(position = "dodge") +
  scale_fill_vangogh("CafeTerrace")

# Continuous fill scale
ggplot(faithfuld, aes(waiting, eruptions, fill = density)) +
  geom_tile() +
  scale_fill_vangogh("Irises", type = "continuous")
```

## Van Gogh Themes

Apply artistic themes to your plots.

### theme_vangogh()

```r
theme_vangogh(style = "classic")
```

**Parameters:**
- `style`: Theme variant ("classic", "light", "dark", "sketch")

**Example:**

```r
# Classic theme
ggplot(iris, aes(Sepal.Length, Sepal.Width, color = Species)) +
  geom_point(size = 4) +
  scale_color_vangogh("Irises") +
  theme_vangogh("classic")

# Light theme
ggplot(mpg, aes(x = class, fill = class)) +
  geom_bar() +
  scale_fill_vangogh("SunflowersMunich") +
  theme_vangogh("light")

# Dark theme
ggplot(diamonds, aes(x = cut, fill = clarity)) +
  geom_bar(position = "fill") +
  scale_fill_vangogh("StarryNight") +
  theme_vangogh("dark")

# Sketch theme
ggplot(mtcars, aes(wt, mpg)) +
  geom_point(size = 3, color = "#F4A460") +
  theme_vangogh("sketch")
```