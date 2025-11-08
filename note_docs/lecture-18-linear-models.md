lecture 18 linear models
================
2025-11-06

### Model Fitting

The code below loads and cleans the `airbnb` data, which will be used as
the primary example for fitting linear models:

``` r
data("nyc_airbnb")

nyc_airbnb <- 
  nyc_airbnb %>% 
  mutate(stars = review_scores_location / 2) %>% 
  rename(
    borough = neighbourhood_group,
    neighborhood = neighbourhood) %>% 
  filter(borough != "Staten Island") %>% 
  select(price, stars, borough, neighborhood, room_type)
```

Fitting a model that considers price as an outcome that may depend on
rating and boro:

``` r
fit = lm(price ~ stars + borough, data = nyc_airbnb)
```

Model variants:

- Intercept-only model: `outcome ~ 1`
- No-intercept model: `outcome ~ 0 + ...`
- Model using all available predictors: `outcome ~ .`

R will automatically create indicator variables for categorical
variables, and determine reference category based on factor level. As
such, you need to be careful with your factors:

``` r
nyc_airbnb <- 
  nyc_airbnb %>% 
  mutate(
    borough = fct_infreq(borough),
    room_type = fct_infreq(room_type))

fit = lm(price ~ stars + borough, data = nyc_airbnb)
```

Changing reference category won’t change model fit, it will just impact
interpretation

### Tidying Output

The output of `lm` is an object of class `lm`– a list that isn’t a
dataframe, but which can be manipulated using other functions. Some
common functions are (output excluded):

``` r
summary(fit)
summary(fit)$coef
coef(fit)
fitted.values(fit)
```

The `broom` package has quick functions for obtaining a quick summary of
the model and for cleaning up the coefficient table:

``` r
fit %>% broom::glance()
```

    ## # A tibble: 1 × 12
    ##   r.squared adj.r.squared sigma statistic   p.value    df   logLik    AIC    BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>     <dbl> <dbl>    <dbl>  <dbl>  <dbl>
    ## 1    0.0342        0.0341  182.      271. 6.73e-229     4 -202113. 4.04e5 4.04e5
    ## # ℹ 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

``` r
fit %>% broom::tidy()
```

    ## # A tibble: 5 × 5
    ##   term            estimate std.error statistic   p.value
    ##   <chr>              <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)         19.8     12.2       1.63 1.04e-  1
    ## 2 stars               32.0      2.53     12.7  1.27e- 36
    ## 3 boroughBrooklyn    -49.8      2.23    -22.3  6.32e-109
    ## 4 boroughQueens      -77.0      3.73    -20.7  2.58e- 94
    ## 5 boroughBronx       -90.3      8.57    -10.5  6.64e- 26

Both of these functions produce dataframes, which make it
straightforward to include the results in subsequent tests:

``` r
fit %>% 
  broom::tidy() %>% 
  select(term, estimate, p.value) %>% 
  mutate(term = str_replace(term, "^borough", "Borough: ")) %>% 
  knitr::kable(digits = 3)
```

| term              | estimate | p.value |
|:------------------|---------:|--------:|
| (Intercept)       |   19.839 |   0.104 |
| stars             |   31.990 |   0.000 |
| Borough: Brooklyn |  -49.754 |   0.000 |
| Borough: Queens   |  -77.048 |   0.000 |
| Borough: Bronx    |  -90.254 |   0.000 |

`broom::tidy()` works with a lot of things, including most of the
functions for model fitting that you’re likely to run into (survival,
mixed models, additive models, …)

### Diagnostics

Regression diagnostics can ID issues in model fit, esp pertaining to
failures in model assumptions. Examining residuals and fitted values are
therefore an important component of any modeling exercise.

`modelr` package can be used to add residuals and fitted values to a
dataframe:

``` r
modelr::add_residuals(nyc_airbnb, fit)
```

    ## # A tibble: 40,492 × 6
    ##    price stars borough neighborhood room_type        resid
    ##    <dbl> <dbl> <fct>   <chr>        <fct>            <dbl>
    ##  1    99   5   Bronx   City Island  Private room      9.47
    ##  2   200  NA   Bronx   City Island  Private room     NA   
    ##  3   300  NA   Bronx   City Island  Entire home/apt  NA   
    ##  4   125   5   Bronx   City Island  Entire home/apt  35.5 
    ##  5    69   5   Bronx   City Island  Private room    -20.5 
    ##  6   125   5   Bronx   City Island  Entire home/apt  35.5 
    ##  7    85   5   Bronx   City Island  Entire home/apt  -4.53
    ##  8    39   4.5 Bronx   Allerton     Private room    -34.5 
    ##  9    95   5   Bronx   Allerton     Entire home/apt   5.47
    ## 10   125   4.5 Bronx   Allerton     Entire home/apt  51.5 
    ## # ℹ 40,482 more rows

``` r
modelr::add_predictions(nyc_airbnb, fit)
```

    ## # A tibble: 40,492 × 6
    ##    price stars borough neighborhood room_type        pred
    ##    <dbl> <dbl> <fct>   <chr>        <fct>           <dbl>
    ##  1    99   5   Bronx   City Island  Private room     89.5
    ##  2   200  NA   Bronx   City Island  Private room     NA  
    ##  3   300  NA   Bronx   City Island  Entire home/apt  NA  
    ##  4   125   5   Bronx   City Island  Entire home/apt  89.5
    ##  5    69   5   Bronx   City Island  Private room     89.5
    ##  6   125   5   Bronx   City Island  Entire home/apt  89.5
    ##  7    85   5   Bronx   City Island  Entire home/apt  89.5
    ##  8    39   4.5 Bronx   Allerton     Private room     73.5
    ##  9    95   5   Bronx   Allerton     Entire home/apt  89.5
    ## 10   125   4.5 Bronx   Allerton     Entire home/apt  73.5
    ## # ℹ 40,482 more rows

This can also be used as part of a pipeline:

``` r
nyc_airbnb %>% 
  modelr::add_residuals(fit) %>% 
  ggplot(aes(x = borough, y = resid)) + geom_violin() + theme_minimal()
```

![](lecture-18-linear-models_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
nyc_airbnb %>% 
  modelr::add_residuals(fit) %>%  
  ggplot(aes(x = stars, y = resid)) + geom_point() + theme_minimal()
```

![](lecture-18-linear-models_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Using the above plots, we can ID some issues with the data– most
notably, extreme outliers present in the data. There are a number of
ways to deal with this, which are not the subject of this course.

### Hypothesis Testing

Model summaries include results of t-tests for single coefficients, and
are the standard way of assessing statistical significance.

Testing multiple coefficients is somewhat more complicated. A useful
approach is to use nested models, meaning that the terms in a simple
“null” model are a subset of the terms in a more complex “alternative”
model. The are formal tests for comparing the null and alternative
models, even when several coefficients are added in the alternative
model. Tests of this kind are required to assess the significance of a
categorical predictor with more than two levels, as in the example
below:

``` r
fit_null <- lm(price ~ stars + borough, data = nyc_airbnb)
fit_alt <- lm(price ~ stars + borough + room_type, data = nyc_airbnb)
```

The test of interest is implemented in the `anova()` function which, of
course, can be summarized using `broom::tidy()`:

``` r
anova(fit_null, fit_alt) |> 
  broom::tidy()
```

    ## # A tibble: 2 × 7
    ##   term                        df.residual    rss    df   sumsq statistic p.value
    ##   <chr>                             <dbl>  <dbl> <dbl>   <dbl>     <dbl>   <dbl>
    ## 1 price ~ stars + borough           30525 1.01e9    NA NA            NA       NA
    ## 2 price ~ stars + borough + …       30523 9.21e8     2  8.42e7     1394.       0

This only works for *nested* models. Comparing non-nested models
requires other methods

### Nesting Data

Using `nest()` to create a list column containing datasets and fit
separate models to each

A non-nested example:

- Analyzing how star ratings and room types affect price differently in
  each boro

``` r
nyc_airbnb |> 
  lm(price ~ stars * borough + room_type * borough, data = _) |> 
  broom::tidy() |> 
  knitr::kable(digits = 3)
```

| term                                  | estimate | std.error | statistic | p.value |
|:--------------------------------------|---------:|----------:|----------:|--------:|
| (Intercept)                           |   95.694 |    19.184 |     4.988 |   0.000 |
| stars                                 |   27.110 |     3.965 |     6.838 |   0.000 |
| boroughBrooklyn                       |  -26.066 |    25.080 |    -1.039 |   0.299 |
| boroughQueens                         |   -4.118 |    40.674 |    -0.101 |   0.919 |
| boroughBronx                          |   -5.627 |    77.808 |    -0.072 |   0.942 |
| room_typePrivate room                 | -124.188 |     2.996 |   -41.457 |   0.000 |
| room_typeShared room                  | -153.635 |     8.692 |   -17.676 |   0.000 |
| stars:boroughBrooklyn                 |   -6.139 |     5.237 |    -1.172 |   0.241 |
| stars:boroughQueens                   |  -17.455 |     8.539 |    -2.044 |   0.041 |
| stars:boroughBronx                    |  -22.664 |    17.099 |    -1.325 |   0.185 |
| boroughBrooklyn:room_typePrivate room |   31.965 |     4.328 |     7.386 |   0.000 |
| boroughQueens:room_typePrivate room   |   54.933 |     7.459 |     7.365 |   0.000 |
| boroughBronx:room_typePrivate room    |   71.273 |    18.002 |     3.959 |   0.000 |
| boroughBrooklyn:room_typeShared room  |   47.797 |    13.895 |     3.440 |   0.001 |
| boroughQueens:room_typeShared room    |   58.662 |    17.897 |     3.278 |   0.001 |
| boroughBronx:room_typeShared room     |   83.089 |    42.451 |     1.957 |   0.050 |

- The native R pipe (`|>`) has to be used here; the tidyverse pipe
  (`%>%`) does not work

A nested example:

- Nesting within boros
- Fit boro-specific models associating price with room and rating type

``` r
nest_lm_res =
  nyc_airbnb %>%  
  nest(data = -borough) %>% 
  mutate(
    models = map(data, \(df) lm(price ~ stars + room_type, data = df)),
    results = map(models, broom::tidy)) %>% 
  select(-data, -models) %>% 
  unnest(results)

nest_lm_res %>% 
  select(borough, term, estimate) %>% 
  mutate(term = fct_inorder(term)) %>% 
  pivot_wider(
    names_from = term, values_from = estimate) %>% 
  knitr::kable(digits = 3)
```

| borough   | (Intercept) |  stars | room_typePrivate room | room_typeShared room |
|:----------|------------:|-------:|----------------------:|---------------------:|
| Bronx     |      90.067 |  4.446 |               -52.915 |              -70.547 |
| Queens    |      91.575 |  9.654 |               -69.255 |              -94.973 |
| Brooklyn  |      69.627 | 20.971 |               -92.223 |             -105.839 |
| Manhattan |      95.694 | 27.110 |              -124.188 |             -153.635 |

Fitting models to nested datasets is a way to perform stratified
analyses. These have a tradeoff: stratified models make it easy to
interpret covariate effects in each stratum, but don’t provide a
mechanism for assessing the significance of differences across strata.

A more extreme example on the assessment of neighborhood effects in
Manhattan, using a neighborhood-specific model:

``` r
manhattan_airbnb <-
  nyc_airbnb %>% 
  filter(borough == "Manhattan")

manhattan_nest_lm_res <-
  manhattan_airbnb %>% 
  nest(data = -neighborhood) %>%  
  mutate(
    models = map(data, \(df) lm(price ~ stars + room_type, data = df)),
    results = map(models, broom::tidy)) %>% 
  select(-data, -models) %>% 
  unnest(results)

manhattan_nest_lm_res %>% 
  filter(str_detect(term, "room_type")) %>% 
  ggplot(aes(x = neighborhood, y = estimate)) + 
  geom_point() + 
  facet_wrap(~term) + 
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 80, hjust = 1))
```

![](lecture-18-linear-models_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

With this many factor levels, fitting models or main interactions to
each wouldn’t be a great idea. Instead, it’d be better to use a mixed
model, with random intercepts and slopes for each neighborhood:

``` r
manhattan_airbnb |> 
  lme4::lmer(price ~ stars + room_type + (1 + room_type | neighborhood), data = _) |> 
  broom.mixed::tidy()
```

- Tho this isn’t working for me

### Binary Outcomes

Linear models are appropriate for continuous outcomes, but logistic
regression is better for binary outcomes

Using data from the washington post on homicides in 50 large US cities:

``` r
baltimore_df <-
  read_csv("./data/homicide-data.csv") %>% 
  filter(city == "Baltimore") %>% 
  mutate(
    resolved = as.numeric(disposition == "Closed by arrest"),
    victim_age = as.numeric(victim_age),
    victim_race = fct_relevel(victim_race, "White")) %>% 
  select(resolved, victim_age, victim_race, victim_sex)
```

    ## Rows: 52179 Columns: 12
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (9): uid, victim_last, victim_first, victim_race, victim_age, victim_sex...
    ## dbl (3): reported_date, lat, lon
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Using this data, a regression model comparing victim demographics to
whether a homicide is resolved can be done using `glm`:

``` r
fit_logistic <-
  baltimore_df |> 
  glm(resolved ~ victim_age + victim_race + victim_sex, data = _, family = binomial()) 
```

This can then be tidied:

``` r
fit_logistic %>% 
  broom::tidy() %>% 
  mutate(OR = exp(estimate)) %>% 
  select(term, log_OR = estimate, OR, p.value) %>% 
  knitr::kable(digits = 3)
```

| term                | log_OR |    OR | p.value |
|:--------------------|-------:|------:|--------:|
| (Intercept)         |  1.190 | 3.287 |   0.000 |
| victim_age          | -0.007 | 0.993 |   0.027 |
| victim_raceAsian    |  0.296 | 1.345 |   0.653 |
| victim_raceBlack    | -0.842 | 0.431 |   0.000 |
| victim_raceHispanic | -0.265 | 0.767 |   0.402 |
| victim_raceOther    | -0.768 | 0.464 |   0.385 |
| victim_sexMale      | -0.880 | 0.415 |   0.000 |

We can also compare fitted values:

``` r
baltimore_df %>% 
  modelr::add_predictions(fit_logistic) %>% 
  mutate(fitted_prob = boot::inv.logit(pred))
```

    ## # A tibble: 2,827 × 6
    ##    resolved victim_age victim_race victim_sex    pred fitted_prob
    ##       <dbl>      <dbl> <fct>       <chr>        <dbl>       <dbl>
    ##  1        0         17 Black       Male       -0.654        0.342
    ##  2        0         26 Black       Male       -0.720        0.327
    ##  3        0         21 Black       Male       -0.683        0.335
    ##  4        1         61 White       Male       -0.131        0.467
    ##  5        1         46 Black       Male       -0.864        0.296
    ##  6        1         27 Black       Male       -0.727        0.326
    ##  7        1         21 Black       Male       -0.683        0.335
    ##  8        1         16 Black       Male       -0.647        0.344
    ##  9        1         21 Black       Male       -0.683        0.335
    ## 10        1         44 Black       Female      0.0297       0.507
    ## # ℹ 2,817 more rows
