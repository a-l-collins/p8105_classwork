lecture 6 – pipes and dplyr
================
2025-09-18

Import `FAS_litters` and `FAS_pups` datasets and clean them:

``` r
# import FAS_litters
litters_df = read_csv("data/data_wrangling/FAS_litters.csv", na = c("NA", ".", ""))
litters_df = janitor::clean_names(litters_df)

# import FAS_pups
# exclude first three rows
pups_df = read_csv("data/data_wrangling/FAS_pups.csv", skip = 3, na = c("NA", ".", ""))
pups_df = janitor::clean_names(pups_df)
```

### `select` function

Options for selecting variables from a dataframe:

``` r
# selecting select variables
select(litters_df, group, litter_number, gd0_weight)

# selecting a range of variables
select(litters_df, group:gd_of_birth)

# select by removal
select(litters_df, -group)

# select by prefix
# ends_with() and contains() are also options
select(litters_df, starts_with("gd"))

# rearrange variables
select(litters_df, litter_number, everything())
relocate(litters_df, litter_number)

# rename by selecting
# group will be renamed to GROUP
select(litters_df, GROUP = group, everything())
rename(litters_df, GROUP = group)

# only selecting one variable
# will give you a dataframe with one variable
select(litters_df, group)
pull(litters_df, group)

# overall, try to avoid litters_df$group... etc
```

### learning assessment 1

``` r
# select litter number, sex, and PD ears
select(pups_df, litter_number, sex, pd_ears)
```

    ## # A tibble: 313 × 3
    ##    litter_number   sex pd_ears
    ##    <chr>         <dbl>   <dbl>
    ##  1 #85               1       4
    ##  2 #85               1       4
    ##  3 #1/2/95/2         1       5
    ##  4 #1/2/95/2         1       5
    ##  5 #5/5/3/83/3-3     1       5
    ##  6 #5/5/3/83/3-3     1       5
    ##  7 #5/4/2/95/2       1      NA
    ##  8 #4/2/95/3-3       1       4
    ##  9 #4/2/95/3-3       1       4
    ## 10 #2/2/95/3-2       1       4
    ## # ℹ 303 more rows

### `filter` function

Options for filtering from a dataframe:

``` r
# equal to
filter(litters_df, gd_of_birth == 20)

# greater than or less than
filter(litters_df, pups_born_alive > 5)
filter(litters_df, pups_born_alive >= 5)
filter(litters_df, pups_born_alive < 5)
filter(litters_df, pups_born_alive <= 5)

# not equals to
filter(litters_df, pups_born_alive != 6)

# filter via character names
filter(litters_df, group %in% c("Con7", "Con8"))

# filter out rows with missing values
drop_na(litters_df) # will drop ALL NA rows
drop_na(litters_df, gd0_weight) # will only drop NA rows in gd0_weight
```

### learning assessment 2

``` r
# only pups with sex 1
filter(pups_df, sex == 1)
```

    ## # A tibble: 155 × 6
    ##    litter_number   sex pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <dbl>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #85               1       4      13        7      11
    ##  2 #85               1       4      13        7      12
    ##  3 #1/2/95/2         1       5      13        7       9
    ##  4 #1/2/95/2         1       5      13        8      10
    ##  5 #5/5/3/83/3-3     1       5      13        8      10
    ##  6 #5/5/3/83/3-3     1       5      14        6       9
    ##  7 #5/4/2/95/2       1      NA      14        5       9
    ##  8 #4/2/95/3-3       1       4      13        6       8
    ##  9 #4/2/95/3-3       1       4      13        7       9
    ## 10 #2/2/95/3-2       1       4      NA        8      10
    ## # ℹ 145 more rows

``` r
# only pups with PD walk less than 11 and sex 2
filter(pups_df, pd_walk < 11, sex == 2)
```

    ## # A tibble: 127 × 6
    ##    litter_number   sex pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <dbl>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #1/2/95/2         2       4      13        7       9
    ##  2 #1/2/95/2         2       4      13        7      10
    ##  3 #1/2/95/2         2       5      13        8      10
    ##  4 #1/2/95/2         2       5      13        8      10
    ##  5 #1/2/95/2         2       5      13        6      10
    ##  6 #5/5/3/83/3-3     2       5      13        8      10
    ##  7 #5/5/3/83/3-3     2       5      14        7      10
    ##  8 #5/5/3/83/3-3     2       5      14        8      10
    ##  9 #5/4/2/95/2       2      NA      14        7      10
    ## 10 #5/4/2/95/2       2      NA      14        7      10
    ## # ℹ 117 more rows

### `mutate` function

Mutate (create/modify variables) within a dataframe

``` r
# creating a variable
mutate(
  litters_df, 
  weight_gain = gd18_weight - gd0_weight
)

# modifying a variable
# in this case, making all values in "group" lowercase
mutate(
  litters_df,
  group = str_to_lower(group)
)
```

### `arrange` function

Arranging things within a dataframe

``` r
# rearrange to follow the order of pups_born_alive variable
arrange(litters_df, pups_born_alive)

# first arranges by group, and then within group, pups_born_alive
arrange(litters_df, group, pups_born_alive)

# arrange in descending order
arrange(litters_df, desc(pups_born_alive))
```

### `pipe` operator

Using pipes to pull things together

``` r
# the "bad" way
litters_df_2 = read_csv("data/data_wrangling/FAS_litters.csv", na = c("NA", ".", ""))
litters_df_2 = janitor::clean_names(litters_df_2)
litters_df_2 = select(litters_df_2, group, litter_number, starts_with("gd"))
litters_df_2 = drop_na(litters_df_2)
litters_df_2 = mutate(litters_df_2, weight_gain = gd18_weight - gd0_weight)

# the "good" way with pipes
litters_df_3 = 
  read_csv("data/data_wrangling/FAS_litters.csv", na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  select(group, litter_number, starts_with("gd")) %>% 
  drop_na() %>% 
  mutate(weight_gain = gd18_weight - gd0_weight)

# both of these produce the same end result
# the second is just cleaner, without constantly overwriting the same dataframe
# in settings, you can turn on default R pipe and use |> instead, but i like %>%
```

### learning assessment 3

``` r
# load pups data
# clean variable names
# filter sex == 1
# remove pd_ears
# create variable for if pd_pivot >= 7
pups_df_2 = 
  read_csv("data/data_wrangling/FAS_pups.csv", skip = 3, na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  filter(sex == 1) %>% 
  select(-pd_ears) %>% 
  mutate(pd_pivot_ge7 = pd_pivot >= 7)
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
