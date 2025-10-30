lecture 17 simulation
================
2025-10-30

### Simulation: Mean and SD for one *n*

The function from lecture 16, which simulates data from a normal
distribution and returns estimates of the mean and sd for that data:

``` r
sim_mean_sd = function(n, mu = 2, sigma = 3) {
  
  sim_data = tibble(
    x = rnorm(n, mean = mu, sd = sigma),
  )
  
  sim_data %>% 
    summarize(
      mu_hat = mean(x),
      sigma_hat = sd(x)
    )
}
```

IRL, repeated sampling is difficult and annoying. On a computer, it’s
pretty easy, using simulations. E.g., running the above function 100
times, to see the effective randomness of *x* and on the mean and sd
estimates:

``` r
output = vector("list", 100)

for (i in 1:100) {
  output[[i]] = sim_mean_sd(30)
}

sim_results = bind_rows(output)

sim_results
```

    ## # A tibble: 100 × 2
    ##    mu_hat sigma_hat
    ##     <dbl>     <dbl>
    ##  1   2.25      2.77
    ##  2   2.40      2.39
    ##  3   2.33      2.88
    ##  4   2.34      2.65
    ##  5   1.01      2.77
    ##  6   2.71      3.17
    ##  7   2.20      3.25
    ##  8   1.29      3.04
    ##  9   2.07      2.79
    ## 10   2.41      3.09
    ## # ℹ 90 more rows

Replicating the above in an alternative form:

- Iterate 100 times with a fixed sample size of 30
- map `sim_mean_sd` over `sample_size` column to replicate the previous
  simulation

``` r
sim_results_df = 
  expand_grid(
    sample_size = 30,
    iter = 1:100) %>% 
  mutate(
    estimate_df = map(sample_size, sim_mean_sd)) %>% 
  unnest(estimate_df)

sim_results_df
```

    ## # A tibble: 100 × 4
    ##    sample_size  iter mu_hat sigma_hat
    ##          <dbl> <int>  <dbl>     <dbl>
    ##  1          30     1   1.79      3.83
    ##  2          30     2   2.47      3.11
    ##  3          30     3   3.39      3.22
    ##  4          30     4   2.24      2.97
    ##  5          30     5   2.61      3.72
    ##  6          30     6   1.37      3.06
    ##  7          30     7   3.25      2.67
    ##  8          30     8   2.06      3.09
    ##  9          30     9   2.27      2.99
    ## 10          30    10   2.46      3.57
    ## # ℹ 90 more rows

The end result is a dataframe that can then be manipulated:

``` r
sim_results_df %>% 
  ggplot(aes(x = mu_hat)) + 
  geom_density() +
  theme_minimal()
```

![](lecture-17-simulation_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
sim_results_df %>% 
  pivot_longer(
    mu_hat:sigma_hat,
    names_to = "parameter", 
    values_to = "estimate") %>% 
  group_by(parameter) %>% 
  summarize(
    emp_mean = mean(estimate),
    emp_sd = sd(estimate)) %>% 
  knitr::kable(digits = 3)
```

| parameter | emp_mean | emp_sd |
|:----------|---------:|-------:|
| mu_hat    |    1.985 |  0.567 |
| sigma_hat |    2.979 |  0.384 |

This provides us information on the distribution in a theoretical
scenario, which can be compared against empirical models, and also
provides a way to test statistical procedures with repeated sampling in
a way that wouldn’t be possible with single data sets.

In cases where function input doesn’t change, using an “anonymous”
function with the syntax `(i)` which defines a function with the input
`i`, lets you rerun the function multiple times while keeping the input
the same:

``` r
sim_results_df =   
  map(1:100, \(i) sim_mean_sd(30, 2, 3)) %>% 
  bind_rows()

sim_results_df
```

    ## # A tibble: 100 × 2
    ##    mu_hat sigma_hat
    ##     <dbl>     <dbl>
    ##  1   1.82      2.97
    ##  2   2.62      3.43
    ##  3   1.58      2.56
    ##  4   2.08      3.97
    ##  5   1.86      3.47
    ##  6   1.64      2.82
    ##  7   2.81      3.04
    ##  8   1.33      3.25
    ##  9   1.10      3.33
    ## 10   2.12      3.76
    ## # ℹ 90 more rows

### Simulation: Mean for several *n*s

An example of iterating several times with a fixed *set* of sample
sizes:

``` r
sim_results_df = 
  expand_grid(
    sample_size = c(30, 60, 120, 240),
    iter = 1:1000) %>% 
  mutate(
    estimate_df = map(sample_size, sim_mean_sd)) %>% 
  unnest(estimate_df)

sim_results_df
```

    ## # A tibble: 4,000 × 4
    ##    sample_size  iter mu_hat sigma_hat
    ##          <dbl> <int>  <dbl>     <dbl>
    ##  1          30     1   2.63      2.85
    ##  2          30     2   1.71      2.82
    ##  3          30     3   2.16      2.70
    ##  4          30     4   1.61      2.59
    ##  5          30     5   1.87      2.65
    ##  6          30     6   2.24      2.90
    ##  7          30     7   1.72      3.50
    ##  8          30     8   2.43      2.85
    ##  9          30     9   2.33      3.13
    ## 10          30    10   2.30      2.79
    ## # ℹ 3,990 more rows

We can then manipulate the above data to see what it looks like:

``` r
sim_results_df %>% 
  mutate(
    sample_size = str_c("n = ", sample_size),
    sample_size = fct_inorder(sample_size)) %>% 
  ggplot(aes(x = sample_size, y = mu_hat, fill = sample_size)) + 
  geom_violin() +
  theme_minimal()
```

![](lecture-17-simulation_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
sim_results_df %>% 
  pivot_longer(
    mu_hat:sigma_hat,
    names_to = "parameter", 
    values_to = "estimate") %>% 
  group_by(parameter, sample_size) %>% 
  summarize(
    emp_mean = mean(estimate),
    emp_var = var(estimate)) %>% 
  knitr::kable(digits = 3)
```

| parameter | sample_size | emp_mean | emp_var |
|:----------|------------:|---------:|--------:|
| mu_hat    |          30 |    2.001 |   0.289 |
| mu_hat    |          60 |    1.992 |   0.147 |
| mu_hat    |         120 |    2.005 |   0.079 |
| mu_hat    |         240 |    1.999 |   0.038 |
| sigma_hat |          30 |    2.974 |   0.154 |
| sigma_hat |          60 |    2.999 |   0.068 |
| sigma_hat |         120 |    2.996 |   0.039 |
| sigma_hat |         240 |    2.994 |   0.018 |

### Simulation: SLR for one *n*

A function estimating intercept and beta-1 for a simulated simple linear
regression:

``` r
sim_regression = function(n, beta0 = 2, beta1 = 3) {
  
  sim_data = 
    tibble(
      x = rnorm(n, mean = 1, sd = 1),
      y = beta0 + beta1 * x + rnorm(n, 0, 1)
    )
  
  ls_fit = lm(y ~ x, data = sim_data)
  
  tibble(
    beta0_hat = coef(ls_fit)[1],
    beta1_hat = coef(ls_fit)[2]
  )
}
```

Which we can then run 500 times to show the effect of randomness on
intercepts and beta-1s:

``` r
sim_results_df = 
  expand_grid(
    sample_size = 30,
    iter = 1:500) %>% 
  mutate(
    estimate_df = map(sample_size, sim_regression)) %>% 
  unnest(estimate_df)

sim_results_df
```

    ## # A tibble: 500 × 4
    ##    sample_size  iter beta0_hat beta1_hat
    ##          <dbl> <int>     <dbl>     <dbl>
    ##  1          30     1      2.07      2.98
    ##  2          30     2      2.26      3.07
    ##  3          30     3      2.05      2.87
    ##  4          30     4      1.84      3.11
    ##  5          30     5      1.56      3.29
    ##  6          30     6      2.17      2.80
    ##  7          30     7      2.30      2.82
    ##  8          30     8      2.20      2.84
    ##  9          30     9      2.04      2.84
    ## 10          30    10      1.56      3.13
    ## # ℹ 490 more rows

We can then plot these estimates against each other on a plot, and see
the overall trend in the way they are effected by randomness:

``` r
sim_results_df %>% 
  ggplot(aes(x = beta0_hat, y = beta1_hat)) + 
  geom_point() +
  theme_minimal()
```

![](lecture-17-simulation_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

### Varying two simulation parameters

An example of iterating several times, where two different parameters
have multiple options:

``` r
sim_results_df = 
  expand_grid(
    sample_size = c(30, 60, 120, 240),
    true_sd = c(6, 3),
    iter = 1:1000) %>% 
  mutate(
    estimate_df = 
      map2(sample_size, true_sd, \(n, sd) sim_mean_sd(n = n, sigma = sd))) %>% 
  unnest(estimate_df)

sim_results_df
```

    ## # A tibble: 8,000 × 5
    ##    sample_size true_sd  iter mu_hat sigma_hat
    ##          <dbl>   <dbl> <int>  <dbl>     <dbl>
    ##  1          30       6     1  2.23       4.83
    ##  2          30       6     2  2.27       5.08
    ##  3          30       6     3 -0.345      7.08
    ##  4          30       6     4  2.33       6.84
    ##  5          30       6     5  2.66       6.31
    ##  6          30       6     6  5.71       5.88
    ##  7          30       6     7  0.693      4.33
    ##  8          30       6     8  1.94       6.03
    ##  9          30       6     9  2.43       5.97
    ## 10          30       6    10  5.46       5.23
    ## # ℹ 7,990 more rows

Which we can then plot:

``` r
sim_results_df %>% 
  mutate(
    true_sd = str_c("True SD: ", true_sd),
    true_sd = fct_inorder(true_sd),
    sample_size = str_c("n = ", sample_size),
    sample_size = fct_inorder(sample_size)) %>% 
  ggplot(aes(x = sample_size, y = mu_hat, fill = sample_size)) + 
  geom_violin() + 
  facet_grid(. ~ true_sd) +
  theme_minimal()
```

![](lecture-17-simulation_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->
