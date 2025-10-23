lecture 9 visualization 2
================
2025-09-30

### ggplot labels

Setting up the dataset that will be used:

``` r
library(p8105.datasets)
data("weather_df")
```

Using this scatterplot:

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) +
  geom_point(aes(color = name), alpha = 0.5)
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Labels can all be controlled using `labs`

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) +
  geom_point(aes(color = name), alpha = 0.5) +
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maximum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package"
  )
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

### ggplot scales

Controlling the scale of the plot:

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) +
  geom_point(aes(color = name), alpha = 0.5) +
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maximum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package") +
  scale_x_continuous(
    breaks = c(-15, 0, 15),
    labels = c("-15º C", "0", "15")) +
  scale_y_continuous(
    trans = "sqrt",
    position = "right")
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

There are a lot of things you can do with scales, but likely you’ll need
to google how to do exactly what it is you want to do.

### ggplot themes

You can use scale to manage the coloring:

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package") + 
  scale_color_hue(h = c(100, 300))
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

The `viridis` package can be used to more easily produce a good theme:

``` r
ggp_temp_plot = 
  weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package"
  ) + 
  viridis::scale_color_viridis(
    name = "Location", 
    discrete = TRUE
  )

ggp_temp_plot
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

You can use themes to change things like legend position:

Note that `legend.position = "none"` will remove the legend, which can
be useful.

``` r
ggp_temp_plot + 
  theme(legend.position = "bottom")
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

There are some built-in themes, which control how the box looks:

``` r
ggp_temp_plot + 
  theme_bw() + 
  theme(legend.position = "bottom")
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
ggp_temp_plot + 
  theme_classic() + 
  theme(legend.position = "bottom")
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-9-2.png)<!-- -->

``` r
ggp_temp_plot + 
  ggthemes::theme_excel() + 
  theme(legend.position = "bottom")
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-9-3.png)<!-- -->

### setting options

An example of settings you can add to the beginning of a .rmd file, to
control how things look:

``` r
library(tidyverse)

knitr::opts_chunk$set(
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme_minimal() + theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis"
)

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

### data arguments in `geom_*`

You can filter by data to control the way in which each exact data
points are displayed:

``` r
central_park_df = 
  weather_df %>% 
  filter(name == "CentralPark_NY")

molokai_df = 
  weather_df %>% 
  filter(name == "Molokai_HI")

ggplot(data = molokai_df, aes(x = date, y = tmax, color = name)) + 
  geom_point() + 
  geom_line(data = central_park_df) 
```

![](lecture-9-visualization-2_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Realistically, it’s sometimes necessary to overlay data summaries on a
plot of the complete data. One way to do this is to create a “summary”
dataframe and use that when adding a new `geom` to a `ggplot` based on
the full data.

### patchwork

This note set imports the `patchwork` library.

Finish later…
