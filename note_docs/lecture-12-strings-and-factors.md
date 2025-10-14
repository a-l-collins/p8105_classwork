lecture 12 strings and factors
================
2025-10-14

Loading libraries and data for this particular lecture:

``` r
library(rvest)
library(p8105.datasets)
```

### Strings and regex

String options:

``` r
# setup string
string_vec <- c("my", "name", "is", "jeff")

# find cases where a match exists
str_detect(string_vec, "jeff")
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
# replace an instance of a match with something else
str_replace(string_vec, "jeff", "Jeff")
```

    ## [1] "my"   "name" "is"   "Jeff"

``` r
# designate match at the beginning of a line
str_detect(string_vec, "^m")
```

    ## [1]  TRUE FALSE FALSE FALSE

``` r
# designate match at the end of a line
str_detect(string_vec, "f$")
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
# designate a list of variables that will count as a match
string_vec <- c("Pumpkin", "pumpkin", "bumpkin")
str_detect(string_vec,"[Pp]umpkin")
```

    ## [1]  TRUE  TRUE FALSE

``` r
# provide a range of letters/numbers that count as a match
str_detect(string_vec, "^[a-zA-Z]")
```

    ## [1] TRUE TRUE TRUE

``` r
# the character . matches anything
str_detect(string_vec, ".")
```

    ## [1] TRUE TRUE TRUE

``` r
# [], (), . have to be searched using \\
string_vec <- c("[2]", "[1]", "3")
str_detect(string_vec, "\\[")
```

    ## [1]  TRUE  TRUE FALSE

### Thoughts on factors

Factors are a way to store categorical variables in R. They can take on
specific levels (e.g. male, female) which are presented as characters
but stored by R as integers. These integer values are used by functions
throughout R, but hidden behind the character string labels

``` r
vec_sex <- factor(c("male", "male", "female", "female"))
vec_sex
```

    ## [1] male   male   female female
    ## Levels: female male

``` r
# factors are secretly numbers
as.numeric(vec_sex)
```

    ## [1] 2 2 1 1

``` r
# factors have levels
vec_sex = fct_relevel(vec_sex, "male")
vec_sex
```

    ## [1] male   male   female female
    ## Levels: male female

``` r
as.numeric(vec_sex)
```

    ## [1] 1 1 2 2

### NSDUH

Revisiting a table scraped from the National Survey on Drug Use and
Health:

``` r
nsduh_url <- "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

table_marj <-
  read_html(nsduh_url) %>% 
  html_table() %>% 
  first() %>% 
  slice(-1)
```

Some steps to tidy the data:

1.  Not currently interested in the p-values
2.  Pivoting longer
3.  Removing letter superscripts from numeric entries

``` r
data_marj <-
  table_marj %>% 
  select(-contains("P Value")) %>% 
  pivot_longer(
    -State,
    names_to = "age_year", 
    values_to = "percent") %>% 
  separate(age_year, into = c("age", "year"), sep = "\\(") %>% 
  mutate(
    year = str_replace(year, "\\)", ""),
    percent = str_replace(percent, "[a-c]$", ""),
    percent = as.numeric(percent)) %>% 
  filter(!(State %in% c("Total U.S.", "Northeast", "Midwest", "South", "West")))
```

Stringr and regular expressions were used a couple of times above:

- in `separate`, we split age and year at the open parentheses using
  `"\\("`
- we stripped out the close parenthesis in `mutate`
- to remove character superscripts, we replaced any character using
  `"[a-c]$"`

Visualizing data for the 12-17 group, treating `state` as a factor and
reordering according to the median `percent` value:

``` r
data_marj %>% 
  filter(age == "12-17") %>% 
  mutate(State = fct_reorder(State, percent)) %>% 
  ggplot(aes(x = State, y = percent, color = year)) + 
    geom_point() + 
    theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![](lecture-12-strings-and-factors_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

More examples on the website
