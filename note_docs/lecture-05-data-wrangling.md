lecture 5 – data import
================
2025-09-16

come back to this and hide unnecessary outputs…

## Importing data tables

### Importing a CSV file

This utilizes relative paths when importing data tables.

``` r
# import csv file; don't use read.csv
# requires "tidyverse" library to be loaded
litters_df = read_csv(file = "./data/data_wrangling/FAS_litters.csv")

# variable names before cleanup
names(litters_df)
```

    ## [1] "Group"             "Litter Number"     "GD0 weight"       
    ## [4] "GD18 weight"       "GD of Birth"       "Pups born alive"  
    ## [7] "Pups dead @ birth" "Pups survive"

``` r
# clean up variable names by converting them to lower snake case
litters_df = janitor::clean_names(litters_df)
names(litters_df)
```

    ## [1] "group"           "litter_number"   "gd0_weight"      "gd18_weight"    
    ## [5] "gd_of_birth"     "pups_born_alive" "pups_dead_birth" "pups_survive"

Fixing the “missing variables messing with variable types” problem:

``` r
# fix missing variables in import
# include na= function to indicate what the missing variables of this table are
litters_df = read_csv(file = "./data/data_wrangling/FAS_litters.csv", na = c("NA", ".", ""))
```

`package::function` lets you use a function from a package without
loading the whole library.

### Importing an Excel file

``` r
# this requires the "readxl" library to be loaded
mlb_df = read_excel("./data/data_wrangling/mlb11.xlsx")

# sometimes, you need to only import a specific set of cells
fotr_df = read_excel("./data/data_wrangling/LotR_Words.xlsx", range = "B3:D6")
```

### Importing a SAS file

``` r
# this requires the "haven" library to be loaded
pulse_df = read_sas("./data/data_wrangling/public_pulse_data.sas7bdat")
```

## Learning Assessment

Import FAS_pups.csv on my own.

``` r
pups_df = read_csv(file = "./data/data_wrangling/FAS_pups.csv")
```
