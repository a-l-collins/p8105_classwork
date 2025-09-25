lecture 8 – visualization
================
2025-09-25

Importing the data which will be used:

``` r
library(p8105.datasets)
data("weather_df")
```

### scatterplots

Basic scatterplot:

``` r
ggplot(weather_df, aes(x = tmin, y = tmax)) +
  geom_point()
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Another option is to do the following method, which allows for some
degree of pre-processing if you don’t want to save those pre-processing
steps:

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point()
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

### advanced scatterplots

Adding in a color key:

``` r
ggplot(weather_df, aes(x = tmin, y = tmax)) +
  geom_point(aes(color = name))
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Adding in a smooth curve, and making the points a bit transparent:

``` r
ggplot(weather_df, aes(x = tmin, y = tmax)) +
  geom_point(aes(color = name), alpha = 0.5) +
  geom_smooth(se = FALSE)
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Adding a facet based on name

``` r
ggplot(weather_df, aes(x = tmin, y = tmax, color = name)) +
  geom_point(aes(color = name), alpha = 0.5) +
  geom_smooth(se = FALSE) +
  facet_grid(. ~ name)
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

Changing x to be date, and only checking precipitation:

``` r
ggplot(weather_df, aes(x = date, y = tmax, color = name)) +
  geom_point(aes(size = prcp), alpha = 0.5) +
  geom_smooth(se = FALSE) +
  facet_grid(. ~ name) # format = "row ~ column"
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

### learning assessment

Using `weather_df`, filter by Central Park, compare temperatures and
convert them to Fahrenheit, and overlay a linear regression line.

``` r
weather_df %>% 
  filter(name == "CentralPark_NY") %>% 
  mutate(
    tmax_fahr = tmax * (9 / 5) + 32,
    tmin_fahr = tmin * (9 / 5) + 32) %>% 
  ggplot(aes(x = tmin_fahr, y = tmax_fahr)) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = lm, se = FALSE)
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

### univariate plots

Histograms

``` r
ggplot(weather_df, aes(x = tmax)) +
  geom_histogram()
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

You can play around with aesthetics here too

``` r
ggplot(weather_df, aes(x = tmax, fill = name)) +
  geom_histogram(position = "dodge", binwidth = 2)
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Density plots

``` r
ggplot(weather_df, aes(x = tmax, fill = name)) +
  geom_density(alpha = 0.4, adjust = 0.5, color = "blue")
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Boxplots

``` r
ggplot(weather_df, aes(x = name, y = tmax)) +
  geom_boxplot()
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

Violin plots

``` r
ggplot(weather_df, aes(x = name, y = tmax)) +
  geom_violin(aes(fill = name), alpha = 0.5) +
  stat_summary(fun = "median", color = "blue")
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

Ridge plots

``` r
ggplot(weather_df, aes(x = tmax, y = name)) +
  geom_density_ridges(scale = 0.85)
```

![](lecture-8-visualization_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->
