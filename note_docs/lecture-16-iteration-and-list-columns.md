lecture 16 iteration and list columns
================
2025-10-28

### Lists

In R, vectors are limited to a single data class. Attempting to merge
two vectors of different data classes will result in coercion. Lists,
alternatively, provide a way to store multiple vectors of different
lengths and data types in one place

``` r
# vector example
vec_numeric <- 5:8
vec_char <- c("My", "name", "is", "Jeff")
vec_logical <- c(TRUE, TRUE, TRUE, FALSE)

# list example
l <- list(
  vec_numeric <- 5:8,
  mat         <- matrix(1:8, 2, 4),
  vec_logical <- c(TRUE, FALSE),
  summary     <- summary(rnorm(1000)))

l
```

    ## [[1]]
    ## [1] 5 6 7 8
    ## 
    ## [[2]]
    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    3    5    7
    ## [2,]    2    4    6    8
    ## 
    ## [[3]]
    ## [1]  TRUE FALSE
    ## 
    ## [[4]]
    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## -3.00805 -0.69737 -0.03532 -0.01165  0.68843  3.81028

Options for accessing items within lists:

``` r
# names --> though this isn't working
l$vec_numeric
```

    ## NULL

``` r
# index
l[[1]]
```

    ## [1] 5 6 7 8

``` r
# index and internal filter
l[[1]][1:3]
```

    ## [1] 5 6 7

### `for` loops

Using the following list & function in the example:

``` r
list_norms <- 
  list(
    a <- rnorm(20, 3, 1),
    b <- rnorm(20, 0, 5),
    c <- rnorm(20, 10, .2),
    d <- rnorm(20, -3, 1)
  )

mean_and_sd <- function(x) {
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x <- mean(x)
  sd_x <- sd(x)

  tibble(
    mean <- mean_x, 
    sd <- sd_x
  )
}
```

We can apply the function to each element of the list manually by naming
each list item:

``` r
mean_and_sd(list_norms[[1]])
```

    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             2.70         1.12

``` r
mean_and_sd(list_norms[[2]])
```

    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            0.416         4.08

``` r
mean_and_sd(list_norms[[3]])
```

    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             10.1        0.191

``` r
mean_and_sd(list_norms[[4]])
```

    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            -3.43         1.18

However, it’s better to do a loop:

``` r
output <- vector("list", length = 4)

for (i in 1:4) {
  output[[i]] <- mean_and_sd(list_norms[[i]])
}

output
```

    ## [[1]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             2.70         1.12
    ## 
    ## [[2]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            0.416         4.08
    ## 
    ## [[3]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             10.1        0.191
    ## 
    ## [[4]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            -3.43         1.18

### `map`

A criticism of `for` loops is that there’s a lot of overhead. `map`
functions in `purrr` try to make the purpose of your code more clear.
note that `map` returns a list:

``` r
# argument 1 = the list/vector/dataframe that we are iterating over
# argument 2 = the function we want to apply to each element
output <- map(list_norms, mean_and_sd)

output
```

    ## [[1]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             2.70         1.12
    ## 
    ## [[2]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            0.416         4.08
    ## 
    ## [[3]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             10.1        0.191
    ## 
    ## [[4]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            -3.43         1.18

### `map` variants

`map_dbl`, `map_int`, `map_lgl` return a vector

``` r
output_map <- map(list_norms, median)
output_map
```

    ## [[1]]
    ## [1] 2.621376
    ## 
    ## [[2]]
    ## [1] 0.7210996
    ## 
    ## [[3]]
    ## [1] 10.05016
    ## 
    ## [[4]]
    ## [1] -3.521665

``` r
# important to use map_dbl instead of map_int or map_lgl, because the output of median() isn't an integer or logical
output_mapdbl <- map_dbl(list_norms, median, .id = "input")
output_mapdbl
```

    ## [1]  2.6213757  0.7210996 10.0501641 -3.5216649

`map_dfr` returns a dataframe

``` r
output <- map_dfr(list_norms, mean_and_sd, .id = "input")
output
```

    ## # A tibble: 4 × 3
    ##   input `mean <- mean_x` `sd <- sd_x`
    ##   <chr>            <dbl>        <dbl>
    ## 1 1                2.70         1.12 
    ## 2 2                0.416        4.08 
    ## 3 3               10.1          0.191
    ## 4 4               -3.43         1.18

`map2` and its variants are helpful when your function has two arguments

``` r
output <- map2(input_1, input_2, \(x,y) func(arg_1 = x, arg_2 = y))
```

### List columns and operations

``` r
listcol_df <- 
  tibble(
    name = c("a", "b", "c", "d"),
    samp = list_norms
  )
```

`name` and `samp` above are both their own unique dataframe columns.
however, using something like `pull`, you can treat that column as a
list

``` r
mean_and_sd(pull(listcol_df, samp)[[1]])
```

    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             2.70         1.12

Because `pull` produces a list, we can apply `map` to it:

``` r
map(pull(listcol_df, samp), mean_and_sd)
```

    ## [[1]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             2.70         1.12
    ## 
    ## [[2]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            0.416         4.08
    ## 
    ## [[3]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1             10.1        0.191
    ## 
    ## [[4]]
    ## # A tibble: 1 × 2
    ##   `mean <- mean_x` `sd <- sd_x`
    ##              <dbl>        <dbl>
    ## 1            -3.43         1.18

And then, because `map` returns a list, we can store the results as a
dataframe column:

``` r
listcol_df = 
  listcol_df %>% 
  mutate(summary = map(samp, mean_and_sd))

listcol_df
```

    ## # A tibble: 4 × 3
    ##   name  samp       summary         
    ##   <chr> <list>     <list>          
    ## 1 a     <dbl [20]> <tibble [1 × 2]>
    ## 2 b     <dbl [20]> <tibble [1 × 2]>
    ## 3 c     <dbl [20]> <tibble [1 × 2]>
    ## 4 d     <dbl [20]> <tibble [1 × 2]>
