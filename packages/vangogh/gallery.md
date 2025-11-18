---
layout: default
title: Visualisation Gallery
parent: vangogh
nav_order: 4
---

# Visualisation Gallery

## StarryNight - geom_bar() example

<img src="/docs/assets/images/plots/barplot_StarryNight.png" style="width: 300px;" />

```r
df1 <- data.frame(x = letters[1:5], y = sample(1:5))
ggplot(df1, aes(x, y, fill = x)) +
  geom_bar(stat = "identity") +
  scale_fill_vangogh("StarryNight") +
  theme_minimal()
```

## SunflowersMunich - geom_point() example

<img src="/docs/assets/images/plots/scatter_SunflowersMunich.png" style="width: 300px;" />

```r
df2 <- data.frame(
  x = rnorm(100),
  y = rnorm(100),
  group = sample(1:5, 100, replace = TRUE))
ggplot(df2, aes(x, y, color = factor(group))) +
  geom_point(size = 3) +
  scale_color_vangogh("SunflowersMunich") +
  theme_minimal()
```

## SelfPortrait - geom_line() example

<img src="/docs/assets/images/plots/line_SelfPortrait.png" style="width: 300px;" />

```r
df3 <- data.frame(
  x = 1:10,
  y = cumsum(rnorm(10)),
  group = rep(1:3, length.out = 10))
ggplot(df3, aes(x, y, color = factor(group))) +
  geom_line(linewidth = 1.2) +
  scale_color_vangogh("SelfPortrait") +
  theme_minimal()
```