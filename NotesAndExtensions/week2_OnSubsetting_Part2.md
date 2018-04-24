subsetting vectors 2
================
Bryan Shalloway
Last updated: 2018-04-23

``` r
knitr::opts_chunk$set(echo = TRUE, cache = TRUE)

library(ggplot2)
library(dplyr)
library(nycflights13)
library(Lahman)
library(gapminder)
library(tidyr)
```

I just discovered how to use the `with` function and it is fantastic. It is a much better way of piping into base R functions.

``` r
mtcars %>% 
  na.omit() %>% 
  with(table(gear, cyl))
```

    ##     cyl
    ## gear  4  6  8
    ##    3  1  2 12
    ##    4  8  4  0
    ##    5  2  1  2

``` r
mtcars %>% 
  na.omit() %>% 
  with(pairwise.t.test(mpg, cyl))
```

    ## 
    ##  Pairwise comparisons using t tests with pooled SD 
    ## 
    ## data:  mpg and cyl 
    ## 
    ##   4       6      
    ## 6 0.00024 -      
    ## 8 2.6e-09 0.00415
    ## 
    ## P value adjustment method: holm
