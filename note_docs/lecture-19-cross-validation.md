lecture 19 cross validation
================
2025-11-11

### CV by “hand”

This code chunk imports data that is non-linear and shows increasing
variance as the predictor increases:

``` r
data("lidar")

lidar_df <- 
  lidar %>% 
  as_tibble() %>% 
  mutate(id = row_number())

lidar_df %>% 
  ggplot(aes(x = range, y = logratio)) + 
  geom_point() +
  theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Splitting this data into training and testing sets using `anti_join`:

``` r
train_df <- sample_frac(lidar_df, size = .8)
test_df <- anti_join(lidar_df, train_df, by = "id")

ggplot(train_df, aes(x = range, y = logratio)) + 
  geom_point() + 
  geom_point(data = test_df, color = "red") +
  theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Fitting three different models to the training data for future testing:

``` r
linear_mod <- lm(logratio ~ range, data = train_df)
smooth_mod <- mgcv::gam(logratio ~ s(range), data = train_df)
wiggly_mod <- mgcv::gam(logratio ~ s(range, k = 30), sp = 10e-6, data = train_df)
```

What the smooth and wiggly mods look like graphically:

``` r
train_df %>% 
  add_predictions(smooth_mod) %>% 
  ggplot(aes(x = range, y = logratio)) + 
  geom_point() + 
  geom_line(aes(y = pred), color = "red") +
  theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
train_df %>% 
  add_predictions(wiggly_mod) %>% 
  ggplot(aes(x = range, y = logratio)) + 
  geom_point() + 
  geom_line(aes(y = pred), color = "red") +
  theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

`gather_predictions` can be used to put all models into a single dataset
in an easily plottable manner:

``` r
train_df %>% 
  gather_predictions(linear_mod, smooth_mod, wiggly_mod) %>% 
  mutate(model = fct_inorder(model)) %>% 
  ggplot(aes(x = range, y = logratio)) + 
  geom_point() + 
  geom_line(aes(y = pred), color = "red") + 
  facet_wrap(~model) +
  theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Visual inspection says that the linear model is too simple, the wiggly
model is too complex, and the smooth model is about right.

- The linear model will not be able to reproduce the actual complexity
  of the data spread
- The wiggly model is chasing datapoints and will change between
  training sets, which does not create reproducibility

Computing root mean squared errors (RMSEs) for each model on the
*testing* dataset:

``` r
rmse(linear_mod, test_df)
```

    ## [1] 0.127317

``` r
rmse(smooth_mod, test_df)
```

    ## [1] 0.08302008

``` r
rmse(wiggly_mod, test_df)
```

    ## [1] 0.08848557

### CV using `modelr`

`modelr` lets us automate the process, as well as simple ways to
reiterate the whole process.

``` r
cv_df <- crossv_mc(lidar_df, 100) 
```

When using non-`lm` models (ie when using `gam`), `cross_cv` products
need to be coerced into datasets we can work with. This loses
`cross_cv`’s smart memory storage:

``` r
cv_df %>% pull(train) %>% nth(1) %>% as_tibble()
```

    ## # A tibble: 176 × 3
    ##    range logratio    id
    ##    <dbl>    <dbl> <int>
    ##  1   390  -0.0504     1
    ##  2   394  -0.0510     4
    ##  3   396  -0.0599     5
    ##  4   399  -0.0596     7
    ##  5   400  -0.0399     8
    ##  6   402  -0.0294     9
    ##  7   403  -0.0395    10
    ##  8   405  -0.0476    11
    ##  9   406  -0.0604    12
    ## 10   408  -0.0312    13
    ## # ℹ 166 more rows

``` r
cv_df %>% pull(test) %>% nth(1) %>% as_tibble()
```

    ## # A tibble: 45 × 3
    ##    range logratio    id
    ##    <dbl>    <dbl> <int>
    ##  1   391  -0.0601     2
    ##  2   393  -0.0419     3
    ##  3   397  -0.0284     6
    ##  4   412  -0.0500    16
    ##  5   421  -0.0316    22
    ##  6   424  -0.0884    24
    ##  7   426  -0.0702    25
    ##  8   427  -0.0288    26
    ##  9   436  -0.0573    32
    ## 10   445  -0.0647    38
    ## # ℹ 35 more rows

``` r
cv_df <-
  cv_df %>%  
  mutate(
    train = map(train, as_tibble),
    test = map(test, as_tibble))
```

Candidate models and RMSEs can now be fitted using `map`:

``` r
cv_df <-
  cv_df %>%  
  mutate(
    linear_mod  = map(train, \(df) lm(logratio ~ range, data = df)),
    smooth_mod  = map(train, \(df) gam(logratio ~ s(range), data = df)),
    wiggly_mod  = map(train, \(df) gam(logratio ~ s(range, k = 30), sp = 10e-6, data = df))) %>% 
  mutate(
    rmse_linear = map2_dbl(linear_mod, test, \(mod, df) rmse(model = mod, data = df)),
    rmse_smooth = map2_dbl(smooth_mod, test, \(mod, df) rmse(model = mod, data = df)),
    rmse_wiggly = map2_dbl(wiggly_mod, test, \(mod, df) rmse(model = mod, data = df)))
```

We can now plot RMSEs to compare models:

``` r
cv_df %>% 
  select(starts_with("rmse")) %>% 
  pivot_longer(
    everything(),
    names_to = "model", 
    values_to = "rmse",
    names_prefix = "rmse_") %>% 
  mutate(model = fct_inorder(model)) %>% 
  ggplot(aes(x = model, y = rmse)) + geom_violin() + theme_minimal()
```

![](lecture-19-cross-validation_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->
