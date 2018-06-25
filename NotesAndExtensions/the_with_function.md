explanation of the `with` function
================
Bryan Shalloway
Last updated: 2018-06-25

``` r
library(tidyverse)
library(nycflights13)
```

The `with` function takes a named list or dataframe as its first argument. The second argument can be an an expression you want evaluated. `with` then allows you to type column names to insert the associated vector into that location. The ability to evaluate expressions by just referencing the column names is called "non-standard evaluation" very similar to how dplyr facilitates this (also using "non-standard evaluation").

For example, say I want to output the column `Sepal.Width` from the `iris` dataset as a vector.

``` r
iris <- as_tibble(iris)
```

``` r
# base R methods
iris$Sepal.Width
```

    ##   [1] 3.5 3.0 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 3.7 3.4 3.0 3.0 4.0 4.4 3.9
    ##  [18] 3.5 3.8 3.8 3.4 3.7 3.6 3.3 3.4 3.0 3.4 3.5 3.4 3.2 3.1 3.4 4.1 4.2
    ##  [35] 3.1 3.2 3.5 3.6 3.0 3.4 3.5 2.3 3.2 3.5 3.8 3.0 3.8 3.2 3.7 3.3 3.2
    ##  [52] 3.2 3.1 2.3 2.8 2.8 3.3 2.4 2.9 2.7 2.0 3.0 2.2 2.9 2.9 3.1 3.0 2.7
    ##  [69] 2.2 2.5 3.2 2.8 2.5 2.8 2.9 3.0 2.8 3.0 2.9 2.6 2.4 2.4 2.7 2.7 3.0
    ##  [86] 3.4 3.1 2.3 3.0 2.5 2.6 3.0 2.6 2.3 2.7 3.0 2.9 2.9 2.5 2.8 3.3 2.7
    ## [103] 3.0 2.9 3.0 3.0 2.5 2.9 2.5 3.6 3.2 2.7 3.0 2.5 2.8 3.2 3.0 3.8 2.6
    ## [120] 2.2 3.2 2.8 2.8 2.7 3.3 3.2 2.8 3.0 2.8 3.0 2.8 3.8 2.8 2.8 2.6 3.0
    ## [137] 3.4 3.1 3.0 3.1 3.1 3.1 2.7 3.2 3.3 3.0 2.5 3.0 3.4 3.0

``` r
iris[["Sepal.Width"]]
```

    ##   [1] 3.5 3.0 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 3.7 3.4 3.0 3.0 4.0 4.4 3.9
    ##  [18] 3.5 3.8 3.8 3.4 3.7 3.6 3.3 3.4 3.0 3.4 3.5 3.4 3.2 3.1 3.4 4.1 4.2
    ##  [35] 3.1 3.2 3.5 3.6 3.0 3.4 3.5 2.3 3.2 3.5 3.8 3.0 3.8 3.2 3.7 3.3 3.2
    ##  [52] 3.2 3.1 2.3 2.8 2.8 3.3 2.4 2.9 2.7 2.0 3.0 2.2 2.9 2.9 3.1 3.0 2.7
    ##  [69] 2.2 2.5 3.2 2.8 2.5 2.8 2.9 3.0 2.8 3.0 2.9 2.6 2.4 2.4 2.7 2.7 3.0
    ##  [86] 3.4 3.1 2.3 3.0 2.5 2.6 3.0 2.6 2.3 2.7 3.0 2.9 2.9 2.5 2.8 3.3 2.7
    ## [103] 3.0 2.9 3.0 3.0 2.5 2.9 2.5 3.6 3.2 2.7 3.0 2.5 2.8 3.2 3.0 3.8 2.6
    ## [120] 2.2 3.2 2.8 2.8 2.7 3.3 3.2 2.8 3.0 2.8 3.0 2.8 3.8 2.8 2.8 2.6 3.0
    ## [137] 3.4 3.1 3.0 3.1 3.1 3.1 2.7 3.2 3.3 3.0 2.5 3.0 3.4 3.0

``` r
# using the with function
with(iris, Sepal.Width)
```

    ##   [1] 3.5 3.0 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 3.7 3.4 3.0 3.0 4.0 4.4 3.9
    ##  [18] 3.5 3.8 3.8 3.4 3.7 3.6 3.3 3.4 3.0 3.4 3.5 3.4 3.2 3.1 3.4 4.1 4.2
    ##  [35] 3.1 3.2 3.5 3.6 3.0 3.4 3.5 2.3 3.2 3.5 3.8 3.0 3.8 3.2 3.7 3.3 3.2
    ##  [52] 3.2 3.1 2.3 2.8 2.8 3.3 2.4 2.9 2.7 2.0 3.0 2.2 2.9 2.9 3.1 3.0 2.7
    ##  [69] 2.2 2.5 3.2 2.8 2.5 2.8 2.9 3.0 2.8 3.0 2.9 2.6 2.4 2.4 2.7 2.7 3.0
    ##  [86] 3.4 3.1 2.3 3.0 2.5 2.6 3.0 2.6 2.3 2.7 3.0 2.9 2.9 2.5 2.8 3.3 2.7
    ## [103] 3.0 2.9 3.0 3.0 2.5 2.9 2.5 3.6 3.2 2.7 3.0 2.5 2.8 3.2 3.0 3.8 2.6
    ## [120] 2.2 3.2 2.8 2.8 2.7 3.3 3.2 2.8 3.0 2.8 3.0 2.8 3.8 2.8 2.8 2.6 3.0
    ## [137] 3.4 3.1 3.0 3.1 3.1 3.1 2.7 3.2 3.3 3.0 2.5 3.0 3.4 3.0

``` r
# the above are the same, just different methods

# note that the below dplyr function gives you something different
# the output is a one-column dataframe as opposed to an atomic vector
select(iris, Sepal.Width)
```

    ## # A tibble: 150 x 1
    ##    Sepal.Width
    ##          <dbl>
    ##  1        3.50
    ##  2        3.00
    ##  3        3.20
    ##  4        3.10
    ##  5        3.60
    ##  6        3.90
    ##  7        3.40
    ##  8        3.40
    ##  9        2.90
    ## 10        3.10
    ## # ... with 140 more rows

Say I want to take the `sum` of `Sepal.Width` and ouptut it as a vector. `with` makes it easier to keep this in a pipe format.

``` r
iris %>% 
  with(sum(Sepal.Width))
```

    ## [1] 458.6

You can also reference more than one column vector in an expression. E.g. lets look at using it with the `table` function (`table` essentially works like a pivot-table with default count option).

``` r
flights %>% 
  with(table(carrier, origin))
```

    ##        origin
    ## carrier   EWR   JFK   LGA
    ##      9E  1268 14651  2541
    ##      AA  3487 13783 15459
    ##      AS   714     0     0
    ##      B6  6557 42076  6002
    ##      DL  4342 20701 23067
    ##      EV 43939  1408  8826
    ##      F9     0     0   685
    ##      FL     0     0  3260
    ##      HA     0   342     0
    ##      MQ  2276  7193 16928
    ##      OO     6     0    26
    ##      UA 46087  4534  8044
    ##      US  4405  2995 13136
    ##      VX  1566  3596     0
    ##      WN  6188     0  6087
    ##      YV     0     0   601

Or maybe I want to use `table` and then pass the outputs into `cor` (to check the correlation of flights by carrier between the three airports in NYC).

``` r
 flights %>% 
  with(table(carrier, origin)) %>% 
  cor()
```

    ##             EWR         JFK       LGA
    ## EWR  1.00000000 -0.04054738 0.1868113
    ## JFK -0.04054738  1.00000000 0.3775083
    ## LGA  0.18681128  0.37750831 1.0000000

-   LGA and JFK seem to have at least some correlation of flights and carrier

Essentially, I use `with` when I want to do operations in a "non-standard evaluation" format similar to that used in dplyr while also facilitating the use of the pipe, but when I want to do operations outside of the context of dataframes. (I mostly keep my data in dataframes though so I don't use `with` that frequently - though it can be useful.)
