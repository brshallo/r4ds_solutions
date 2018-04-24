On variable names in character vectors (wk 2)
================
Bryan Shalloway
Last updated: 2018-04-24

-   [Reason 1: single place to change](#reason-1-single-place-to-change)
-   [Reason 2, dynamic list of vars](#reason-2-dynamic-list-of-vars)
-   [Other reasons...](#other-reasons...)

``` r
knitr::opts_chunk$set(echo = TRUE)

library(dplyr)
library(ggplot2)
library(Lahman)
library(nycflights13)
```

vincent asked about the utlitiy of keeping the names of your columns in a seperate character vector and when it may be useful to do this versus just having the names within the funciton itself.
I.e.:
`select(flights, dep_delay, arr_delay)`

VS

`vars <- c("dep_delay", "arr_delay")`
`select(flights, vars)`

What/when does this extra step get you something...? I answered this briefly but wanted to restate this and give a little more thorough explanation on when this capability may be useful.

Reason 1: single place to change
--------------------------------

If you have multiple operations on the same set of fields, it may be helpful to keep them in a character vector in a single location. Then, if you want to change your list of variables, you only need to change one location.

``` r
vars <- c("air_time", "dep_delay")


##Operation 1
flights %>% 
  select(dest, vars) %>% 
  group_by(dest) %>% 
  filter(n() > 100) %>% 
  summarise_all(mean, na.rm = TRUE)
```

    ## Warning: package 'bindrcpp' was built under R version 3.4.4

    ## # A tibble: 93 x 3
    ##    dest  air_time dep_delay
    ##    <chr>    <dbl>     <dbl>
    ##  1 ABQ      249.      13.7 
    ##  2 ACK       42.1      6.46
    ##  3 ALB       31.8     23.6 
    ##  4 ATL      113.      12.5 
    ##  5 AUS      213.      13.0 
    ##  6 AVL       89.9      8.19
    ##  7 BDL       25.5     17.7 
    ##  8 BGR       54.1     19.5 
    ##  9 BHM      123.      29.7 
    ## 10 BNA      114.      16.0 
    ## # ... with 83 more rows

``` r
##Operation 2
flights %>% 
  select(dest, vars) %>% 
  group_by(dest) %>% 
  filter(n() > 100) %>% 
  summarise_all(median, na.rm = TRUE)
```

    ## # A tibble: 93 x 3
    ##    dest  air_time dep_delay
    ##    <chr>    <dbl>     <dbl>
    ##  1 ABQ       246.        0.
    ##  2 ACK        41.       -3.
    ##  3 ALB        31.        1.
    ##  4 ATL       112.       -2.
    ##  5 AUS       211.       -1.
    ##  6 AVL        89.       -3.
    ##  7 BDL        25.       -1.
    ##  8 BGR        54.       -2.
    ##  9 BHM       121.        1.
    ## 10 BNA       113.       -1.
    ## # ... with 83 more rows

``` r
##Operation 3
##Don't worry if you don't understand this function yet
flights %>% 
  select(vars) %>% 
  mutate_at(vars, funs(in_hours = . / 60))
```

    ## # A tibble: 336,776 x 4
    ##    air_time dep_delay air_time_in_hours dep_delay_in_hours
    ##       <dbl>     <dbl>             <dbl>              <dbl>
    ##  1     227.        2.             3.78              0.0333
    ##  2     227.        4.             3.78              0.0667
    ##  3     160.        2.             2.67              0.0333
    ##  4     183.       -1.             3.05             -0.0167
    ##  5     116.       -6.             1.93             -0.100 
    ##  6     150.       -4.             2.50             -0.0667
    ##  7     158.       -5.             2.63             -0.0833
    ##  8      53.       -3.             0.883            -0.0500
    ##  9     140.       -3.             2.33             -0.0500
    ## 10     138.       -2.             2.30             -0.0333
    ## # ... with 336,766 more rows

In the above example you have three different operations all stemming from the same set of variables. Let's say you want to change your `vars` to `c("distance", "arr_delay")`, rather than needing to change the vars in each operation individually, if they all connect back to a single vector you can change this vector in just one location and all operations will be changed accordingly.

This goal of having parameters in as few places as possible and avoiding duplication is a key principle that we'll talk about when we get into functional programming and helps make it easier to make changes to your code in the future.

Reason 2, dynamic list of vars
------------------------------

**We are not here yet**, but a very common reason you may want to input your variable names as a text string is when building functions (again, we will learn about these in the future). In this case, rather than keeping your var names in one location, you are keeping your logic in one location (within the funtion). But it's often easier to build quick funcitons when you pass the arguments in as character strings.

Let's first look at an operation you may want to think about building a new function for.

``` r
flights %>% 
  group_by(dest) %>% 
  summarise_at(c("dep_delay", "arr_delay"), mean, na.rm = TRUE)
```

    ## # A tibble: 105 x 3
    ##    dest  dep_delay arr_delay
    ##    <chr>     <dbl>     <dbl>
    ##  1 ABQ       13.7       4.38
    ##  2 ACK        6.46      4.85
    ##  3 ALB       23.6      14.4 
    ##  4 ANC       12.9      -2.50
    ##  5 ATL       12.5      11.3 
    ##  6 AUS       13.0       6.02
    ##  7 AVL        8.19      8.00
    ##  8 BDL       17.7       7.05
    ##  9 BGR       19.5       8.03
    ## 10 BHM       29.7      16.9 
    ## # ... with 95 more rows

Maybe you find yourself doing this a lot where you have a dataframe, you group by some variable, and then you want to do some aggregation upon this.

If this is the case, you could put this into a funciton. Let's just take the above and replace our inputs with generic values. So `flights` is replaced with `df`, `dest` with `var_group`, `c("dep_delay", "arr_delay")` with `vars` and `mean` with `fun`. Then wrap this in a function call providing place holders for the inputs.

``` r
GroupAndSummariseAt <- function(df, var_group, vars, fun){
  df %>% 
    group_by_(var_group) %>% 
    summarise_at(vars, fun, na.rm = TRUE)
}
```

The variables exist within the dataframe and for this function must be passed into the function as text strings[1] Note also the slight change in the `group_by_` function[2].) This new function also allows you to change the method used for aggregation.

``` r
GroupAndSummariseAt(df = flights, var_group = "dest", vars = c("dep_delay", "arr_delay"), fun = "mean")
```

    ## # A tibble: 105 x 3
    ##    dest  dep_delay arr_delay
    ##    <chr>     <dbl>     <dbl>
    ##  1 ABQ       13.7       4.38
    ##  2 ACK        6.46      4.85
    ##  3 ALB       23.6      14.4 
    ##  4 ANC       12.9      -2.50
    ##  5 ATL       12.5      11.3 
    ##  6 AUS       13.0       6.02
    ##  7 AVL        8.19      8.00
    ##  8 BDL       17.7       7.05
    ##  9 BGR       19.5       8.03
    ## 10 BHM       29.7      16.9 
    ## # ... with 95 more rows

``` r
GroupAndSummariseAt(flights, "origin", c("air_time", "distance"), "sd")
```

    ## # A tibble: 3 x 3
    ##   origin air_time distance
    ##   <chr>     <dbl>    <dbl>
    ## 1 EWR        93.3     730.
    ## 2 JFK       114.      896.
    ## 3 LGA        49.4     372.

``` r
GroupAndSummariseAt(Lahman::Batting, "yearID", c("H", "HR", "AB"), "sum")
```

    ## # A tibble: 146 x 4
    ##    yearID     H    HR    AB
    ##     <int> <int> <int> <int>
    ##  1   1871  3101    47 10822
    ##  2   1872  4487    35 15679
    ##  3   1873  4923    46 16974
    ##  4   1874  5226    40 19104
    ##  5   1875  6812    40 26833
    ##  6   1876  5338    40 20121
    ##  7   1877  3705    24 13667
    ##  8   1878  3539    23 13644
    ##  9   1879  6171    58 24155
    ## 10   1880  5946    62 24301
    ## # ... with 136 more rows

Other reasons...
----------------

There are other dynamic processes you may have for generating character vectors to be fed into dplyr functions, but hopefully these provide a couple examples of possible motivations...

[1] If you tried inputting the variable names without text strings the function would assume there was some value associated with the name and would send an error once it didn't find any corresponding object in your environment.

[2] Note that for this example `group_by` has to be changed to `group_by_` which is a version of the function designed for receiving a quoted string to specify the variables. `mutate` and `select` have `*_` variants as well, I believe `select` works either way though.
