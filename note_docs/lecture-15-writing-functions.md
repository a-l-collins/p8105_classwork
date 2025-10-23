lecture 15 writing functions
================
2025-10-23

### My First Function

The best way to build up a function is to start with code you’ve written
outside of a function.

This takes a sample from a normal distribution and computes the vector
of Z scores for the sample:

``` r
x_vec <- rnorm(25, mean = 5, sd = 3)

(x_vec - mean(x_vec)) / sd(x_vec)
```

    ##  [1] -0.83687228  0.01576465 -1.05703126  1.50152998  0.16928872 -1.04107494
    ##  [7]  0.33550276  0.59957343  0.42849461 -0.49894708  1.41364561  0.23279252
    ## [13] -0.83138529 -2.50852027  1.00648110 -0.22481531 -0.19456260  0.81587675
    ## [19]  0.68682298  0.44756609  0.78971253  0.64568566 -0.09904161 -2.27133861
    ## [25]  0.47485186

If I want to repeat this process a number of times, I can create a
function to do it:

``` r
z_scores = function(x) {
  # calculates the z-scores on the sample
  z = (x - mean(x)) / sd(x)
  
  # returns the result
  z
}

z_scores(x_vec)
```

    ##  [1] -0.83687228  0.01576465 -1.05703126  1.50152998  0.16928872 -1.04107494
    ##  [7]  0.33550276  0.59957343  0.42849461 -0.49894708  1.41364561  0.23279252
    ## [13] -0.83138529 -2.50852027  1.00648110 -0.22481531 -0.19456260  0.81587675
    ## [19]  0.68682298  0.44756609  0.78971253  0.64568566 -0.09904161 -2.27133861
    ## [25]  0.47485186

However, not all edge cases are caught as errors. Error checking needs
to be included:

``` r
z_scores = function(x) {
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Z scores cannot be computed for length 1 vectors")
  }
  
  z = mean(x) / sd(x)
  z
}
```

### Multiple Outputs

One option for providing multiple outputs is to store each of the values
in a named list, and to return that list:

``` r
mean_and_sd = function(x) {
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x = mean(x)
  sd_x = sd(x)

  list(mean = mean_x, 
       sd = sd_x)
}
```

Alternatively, these values can be stored in a dataframe:

``` r
mean_and_sd = function(x) {
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x = mean(x)
  sd_x = sd(x)

  tibble(
    mean = mean_x, 
    sd = sd_x
  )
}
```

Which one you should use depends on what kind of values you want to
return, and what you plan on doing with the function itself

### Multiple Inputs

A function that takes a given sample size, its true mean, and its sd,
simulates data from a normal distribution, and returns the estimated
mean and sd:

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

Calling this function would be in the format
`sim_mean_sd(n = ##, mu = ##, sd = ##)`. `n`, `mu`, and `sd` can be
listed in any order within the function call.

### Functions as Arguments

One thing you can do is pass functions as arguments into functions

### Scoping

If you create a function but don’t provide it all the values it needs,
it will go looking in the global environment for something to fill those
values with. This can result in functions that look like they’re
working, and which break in confusing ways
