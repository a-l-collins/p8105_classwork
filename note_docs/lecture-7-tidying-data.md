lecture 7 data tidying
================
2025-09-23

This document will discuss tidying data.

### `pivot_longer` function

``` r
# importing the package initially
pulse_df <- 
  haven::read_sas("./data/data_wrangling/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()

pulse_df
```

    ## # A tibble: 1,087 × 7
    ##       id   age sex    bdi_score_bl bdi_score_01m bdi_score_06m bdi_score_12m
    ##    <dbl> <dbl> <chr>         <dbl>         <dbl>         <dbl>         <dbl>
    ##  1 10003  48.0 male              7             1             2             0
    ##  2 10015  72.5 male              6            NA            NA            NA
    ##  3 10022  58.5 male             14             3             8            NA
    ##  4 10026  72.7 male             20             6            18            16
    ##  5 10035  60.4 male              4             0             1             2
    ##  6 10050  84.7 male              2            10            12             8
    ##  7 10078  31.3 male              4             0            NA            NA
    ##  8 10088  56.9 male              5            NA             0             2
    ##  9 10091  76.0 male              0             3             4             0
    ## 10 10092  74.2 female           10             2            11             6
    ## # ℹ 1,077 more rows

use `pivot_longer` to convert data from wide to long

``` r
# make the data longer
# remove unnecessary prefix
# rename baseline indicator to something matching the rest
pulse_tidy_df <- pivot_longer(
  pulse_df,
  bdi_score_bl:bdi_score_12m,
  names_to = "visit",
  names_prefix = "bdi_score_",
  values_to = "bdi"
) %>% 
mutate (
  visit = replace(visit, visit == "bl", "00m"), 
  visit = factor(visit)
)

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit   bdi
    ##    <dbl> <dbl> <chr> <fct> <dbl>
    ##  1 10003  48.0 male  00m       7
    ##  2 10003  48.0 male  01m       1
    ##  3 10003  48.0 male  06m       2
    ##  4 10003  48.0 male  12m       0
    ##  5 10015  72.5 male  00m       6
    ##  6 10015  72.5 male  01m      NA
    ##  7 10015  72.5 male  06m      NA
    ##  8 10015  72.5 male  12m      NA
    ##  9 10022  58.5 male  00m      14
    ## 10 10022  58.5 male  01m       3
    ## # ℹ 4,338 more rows

### learning assessment

Combine the gd_weight variables into a long format:

``` r
litters_df <- 
  read_csv("./data/data_wrangling/FAS_litters.csv", na = c("NA", ".", "")) %>% 
  janitor::clean_names()

litters_df
```

    ## # A tibble: 49 × 8
    ##    group litter_number   gd0_weight gd18_weight gd_of_birth pups_born_alive
    ##    <chr> <chr>                <dbl>       <dbl>       <dbl>           <dbl>
    ##  1 Con7  #85                   19.7        34.7          20               3
    ##  2 Con7  #1/2/95/2             27          42            19               8
    ##  3 Con7  #5/5/3/83/3-3         26          41.4          19               6
    ##  4 Con7  #5/4/2/95/2           28.5        44.1          19               5
    ##  5 Con7  #4/2/95/3-3           NA          NA            20               6
    ##  6 Con7  #2/2/95/3-2           NA          NA            20               6
    ##  7 Con7  #1/5/3/83/3-3/2       NA          NA            20               9
    ##  8 Con8  #3/83/3-3             NA          NA            20               9
    ##  9 Con8  #2/95/3               NA          NA            20               8
    ## 10 Con8  #3/5/2/2/95           28.5        NA            20               8
    ## # ℹ 39 more rows
    ## # ℹ 2 more variables: pups_dead_birth <dbl>, pups_survive <dbl>

``` r
# making table long
# cleaning data values
# would be nice if there was a names_suffix function
litters_tidy_df <- pivot_longer(
  litters_df,
  gd0_weight:gd18_weight,
  names_to = "gd",
  names_prefix = "gd",
  values_to = "weight"
) %>% 
mutate(
  gd = case_match(gd,
    "0_weight" ~ 0,
    "18_weight" ~ 18
  )
)

litters_tidy_df
```

    ## # A tibble: 98 × 8
    ##    group litter_number gd_of_birth pups_born_alive pups_dead_birth pups_survive
    ##    <chr> <chr>               <dbl>           <dbl>           <dbl>        <dbl>
    ##  1 Con7  #85                    20               3               4            3
    ##  2 Con7  #85                    20               3               4            3
    ##  3 Con7  #1/2/95/2              19               8               0            7
    ##  4 Con7  #1/2/95/2              19               8               0            7
    ##  5 Con7  #5/5/3/83/3-3          19               6               0            5
    ##  6 Con7  #5/5/3/83/3-3          19               6               0            5
    ##  7 Con7  #5/4/2/95/2            19               5               1            4
    ##  8 Con7  #5/4/2/95/2            19               5               1            4
    ##  9 Con7  #4/2/95/3-3            20               6               0            6
    ## 10 Con7  #4/2/95/3-3            20               6               0            6
    ## # ℹ 88 more rows
    ## # ℹ 2 more variables: gd <dbl>, weight <dbl>

### `pivot_wider` function

“Un-tidying” data using pivot_wider

``` r
# example dataframe we can work with
analysis_result <- tibble(
  group = c("treatment", "treatment", "placebo", "placebo"),
  time = c("pre", "post", "pre", "post"),
  mean = c(4, 8, 3.5, 4)
)

analysis_result
```

    ## # A tibble: 4 × 3
    ##   group     time   mean
    ##   <chr>     <chr> <dbl>
    ## 1 treatment pre     4  
    ## 2 treatment post    8  
    ## 3 placebo   pre     3.5
    ## 4 placebo   post    4

``` r
# using pivot_wider
analysis_result_untidy <- pivot_wider(
  analysis_result,
  names_from = "time",
  values_from = "mean"
)

analysis_result_untidy
```

    ## # A tibble: 2 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

### binding rows

An example of joining three datasets via binding rows

``` r
# preparing a dataset for use
fellowship_ring <- 
  readxl::read_excel("./data/data_wrangling/LotR_Words.xlsx", range = "B3:D6") %>% 
  mutate(movie = "fellowship_ring")

fellowship_ring
```

    ## # A tibble: 3 × 4
    ##   Race   Female  Male movie          
    ##   <chr>   <dbl> <dbl> <chr>          
    ## 1 Elf      1229   971 fellowship_ring
    ## 2 Hobbit     14  3644 fellowship_ring
    ## 3 Man         0  1995 fellowship_ring

``` r
two_towers <-
  readxl::read_excel("./data/data_wrangling/LotR_Words.xlsx", range = "F3:H6") %>% 
  mutate(movie = "two_towers")

two_towers
```

    ## # A tibble: 3 × 4
    ##   Race   Female  Male movie     
    ##   <chr>   <dbl> <dbl> <chr>     
    ## 1 Elf       331   513 two_towers
    ## 2 Hobbit      0  2463 two_towers
    ## 3 Man       401  3589 two_towers

``` r
return_king <-
  readxl::read_excel("./data/data_wrangling/LotR_Words.xlsx", range = "J3:L6") %>% 
  mutate(movie = "return_king")

return_king
```

    ## # A tibble: 3 × 4
    ##   Race   Female  Male movie      
    ##   <chr>   <dbl> <dbl> <chr>      
    ## 1 Elf       183   510 return_king
    ## 2 Hobbit      2  2673 return_king
    ## 3 Man       268  2459 return_king

``` r
# pulling all three together into a single dataset
lotr <-
  bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words"
  ) %>% 
  mutate(race = str_to_lower(race)) %>% 
  select(movie, everything())

lotr
```

    ## # A tibble: 18 × 4
    ##    movie           race   gender words
    ##    <chr>           <chr>  <chr>  <dbl>
    ##  1 fellowship_ring elf    female  1229
    ##  2 fellowship_ring elf    male     971
    ##  3 fellowship_ring hobbit female    14
    ##  4 fellowship_ring hobbit male    3644
    ##  5 fellowship_ring man    female     0
    ##  6 fellowship_ring man    male    1995
    ##  7 two_towers      elf    female   331
    ##  8 two_towers      elf    male     513
    ##  9 two_towers      hobbit female     0
    ## 10 two_towers      hobbit male    2463
    ## 11 two_towers      man    female   401
    ## 12 two_towers      man    male    3589
    ## 13 return_king     elf    female   183
    ## 14 return_king     elf    male     510
    ## 15 return_king     hobbit female     2
    ## 16 return_king     hobbit male    2673
    ## 17 return_king     man    female   268
    ## 18 return_king     man    male    2459

### joining datasets

``` r
# preparing a dataset for use
pups_df <-
  read_csv("./data/data_wrangling/FAS_pups.csv", skip = 3, na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  mutate(sex = 
           case_match(
             sex,
             1 ~ "male",
             2 ~ "female"),
         sex = as.factor(sex))

pups_df
```

    ## # A tibble: 313 × 6
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #85           male        4      13        7      11
    ##  2 #85           male        4      13        7      12
    ##  3 #1/2/95/2     male        5      13        7       9
    ##  4 #1/2/95/2     male        5      13        8      10
    ##  5 #5/5/3/83/3-3 male        5      13        8      10
    ##  6 #5/5/3/83/3-3 male        5      14        6       9
    ##  7 #5/4/2/95/2   male       NA      14        5       9
    ##  8 #4/2/95/3-3   male        4      13        6       8
    ##  9 #4/2/95/3-3   male        4      13        7       9
    ## 10 #2/2/95/3-2   male        4      NA        8      10
    ## # ℹ 303 more rows

``` r
litters_df <-
  read_csv("./data/data_wrangling/FAS_litters.csv", na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  separate(group, into = c("dose", "day_of_tx"), sep = 3) %>% 
  relocate(litter_number) %>% 
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    dose = str_to_lower(dose)
  )

litters_df
```

    ## # A tibble: 49 × 10
    ##    litter_number   dose  day_of_tx gd0_weight gd18_weight gd_of_birth
    ##    <chr>           <chr> <chr>          <dbl>       <dbl>       <dbl>
    ##  1 #85             con   7               19.7        34.7          20
    ##  2 #1/2/95/2       con   7               27          42            19
    ##  3 #5/5/3/83/3-3   con   7               26          41.4          19
    ##  4 #5/4/2/95/2     con   7               28.5        44.1          19
    ##  5 #4/2/95/3-3     con   7               NA          NA            20
    ##  6 #2/2/95/3-2     con   7               NA          NA            20
    ##  7 #1/5/3/83/3-3/2 con   7               NA          NA            20
    ##  8 #3/83/3-3       con   8               NA          NA            20
    ##  9 #2/95/3         con   8               NA          NA            20
    ## 10 #3/5/2/2/95     con   8               28.5        NA            20
    ## # ℹ 39 more rows
    ## # ℹ 4 more variables: pups_born_alive <dbl>, pups_dead_birth <dbl>,
    ## #   pups_survive <dbl>, wt_gain <dbl>

#### left join

``` r
fas_df <- left_join(pups_df, litters_df, by = "litter_number")

fas_df
```

    ## # A tibble: 313 × 15
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk dose  day_of_tx
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl> <chr> <chr>    
    ##  1 #85           male        4      13        7      11 con   7        
    ##  2 #85           male        4      13        7      12 con   7        
    ##  3 #1/2/95/2     male        5      13        7       9 con   7        
    ##  4 #1/2/95/2     male        5      13        8      10 con   7        
    ##  5 #5/5/3/83/3-3 male        5      13        8      10 con   7        
    ##  6 #5/5/3/83/3-3 male        5      14        6       9 con   7        
    ##  7 #5/4/2/95/2   male       NA      14        5       9 con   7        
    ##  8 #4/2/95/3-3   male        4      13        6       8 con   7        
    ##  9 #4/2/95/3-3   male        4      13        7       9 con   7        
    ## 10 #2/2/95/3-2   male        4      NA        8      10 con   7        
    ## # ℹ 303 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>

#### right join

``` r
fas_df <- right_join(pups_df, litters_df, by = "litter_number")

fas_df
```

    ## # A tibble: 306 × 15
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk dose  day_of_tx
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl> <chr> <chr>    
    ##  1 #85           male        4      13        7      11 con   7        
    ##  2 #85           male        4      13        7      12 con   7        
    ##  3 #1/2/95/2     male        5      13        7       9 con   7        
    ##  4 #1/2/95/2     male        5      13        8      10 con   7        
    ##  5 #5/5/3/83/3-3 male        5      13        8      10 con   7        
    ##  6 #5/5/3/83/3-3 male        5      14        6       9 con   7        
    ##  7 #5/4/2/95/2   male       NA      14        5       9 con   7        
    ##  8 #4/2/95/3-3   male        4      13        6       8 con   7        
    ##  9 #4/2/95/3-3   male        4      13        7       9 con   7        
    ## 10 #2/2/95/3-2   male        4      NA        8      10 con   7        
    ## # ℹ 296 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>

#### inner join

``` r
fas_df <- inner_join(pups_df, litters_df, by = "litter_number")

fas_df
```

    ## # A tibble: 304 × 15
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk dose  day_of_tx
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl> <chr> <chr>    
    ##  1 #85           male        4      13        7      11 con   7        
    ##  2 #85           male        4      13        7      12 con   7        
    ##  3 #1/2/95/2     male        5      13        7       9 con   7        
    ##  4 #1/2/95/2     male        5      13        8      10 con   7        
    ##  5 #5/5/3/83/3-3 male        5      13        8      10 con   7        
    ##  6 #5/5/3/83/3-3 male        5      14        6       9 con   7        
    ##  7 #5/4/2/95/2   male       NA      14        5       9 con   7        
    ##  8 #4/2/95/3-3   male        4      13        6       8 con   7        
    ##  9 #4/2/95/3-3   male        4      13        7       9 con   7        
    ## 10 #2/2/95/3-2   male        4      NA        8      10 con   7        
    ## # ℹ 294 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>

#### full join

``` r
fas_df <- full_join(pups_df, litters_df, by = "litter_number")

fas_df
```

    ## # A tibble: 315 × 15
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk dose  day_of_tx
    ##    <chr>         <fct>   <dbl>   <dbl>    <dbl>   <dbl> <chr> <chr>    
    ##  1 #85           male        4      13        7      11 con   7        
    ##  2 #85           male        4      13        7      12 con   7        
    ##  3 #1/2/95/2     male        5      13        7       9 con   7        
    ##  4 #1/2/95/2     male        5      13        8      10 con   7        
    ##  5 #5/5/3/83/3-3 male        5      13        8      10 con   7        
    ##  6 #5/5/3/83/3-3 male        5      14        6       9 con   7        
    ##  7 #5/4/2/95/2   male       NA      14        5       9 con   7        
    ##  8 #4/2/95/3-3   male        4      13        6       8 con   7        
    ##  9 #4/2/95/3-3   male        4      13        7       9 con   7        
    ## 10 #2/2/95/3-2   male        4      NA        8      10 con   7        
    ## # ℹ 305 more rows
    ## # ℹ 7 more variables: gd0_weight <dbl>, gd18_weight <dbl>, gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>,
    ## #   wt_gain <dbl>
