On subsetting (wk 2)
================
Bryan Shalloway
Last updated: 2018-04-23

-   [on subsetting DFs](#on-subsetting-dfs)
-   [Subsetting vectors, difference](#subsetting-vectors-difference)
-   [Functions that take dfs](#functions-that-take-dfs)
-   [Functions that take vectors](#functions-that-take-vectors)
    -   [Expecting vector inputs](#expecting-vector-inputs)
    -   [Num values outputted](#num-values-outputted)
    -   [Dplyr and outputs](#dplyr-and-outputs)
    -   [More than 1, less than all outputs](#more-than-1-less-than-all-outputs)
-   [DPLYR and non-standard evaluation](#dplyr-and-non-standard-evaluation)
    -   [Part 1](#part-1)
    -   [Part 2, subsetting in Dplyr calcs](#part-2-subsetting-in-dplyr-calcs)

<script>
    $(document).ready(function() {
      $items = $('div#TOC li');
      $items.each(function(idx) {
        num_ul = $(this).parentsUntil('#TOC').length;
        $(this).css({'text-indent': num_ul * 10, 'padding-left': 0});
      });

    });
</script>
on subsetting DFs
-----------------

Caroline asked how she would get JUST the 'flight' column in the flights data. For our purposes, I would recommend using dplyr code to do this.

``` r
select(flights, flight)
```

    ## # A tibble: 336,776 x 1
    ##    flight
    ##     <int>
    ##  1   1545
    ##  2   1714
    ##  3   1141
    ##  4    725
    ##  5    461
    ##  6   1696
    ##  7    507
    ##  8   5708
    ##  9     79
    ## 10    301
    ## # ... with 336,766 more rows

This outputs a data frame / tibble [1] with a single column, `flight`.

**WARNING: LEAVING TIDYVERSE**

In base R this is equivalent to any of the following

`flights["flight"]`
`flights[11]`
`flights[c(FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, TRUE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE)]`

(Running any of the above should give the exact same output)

Likewise, these are also equivalent:

dplyr/tidyverse approach: `select(flights, flight, dep_delay)`

base R approach: `flights[c("flight", "dep_delay")]`

*For a wide variety of reasons (improved performance, more consistent handling, more friendly with pipes, others) we will be using the tidy approach as opposed to the base R approach, so you don't need to hurt your brain by becoming an expert at both, though any familiarity with the base R subsetting is always good to know[2]*

Subsetting vectors, difference
------------------------------

This is a slight digression and not necessary for where we are right now... but in the meeting, I briefly showed how to extract the `flight` column out of the `flights` dataframe and output it as a vector (as opposed to as a one-columned dataframe) using base R's subsetting syntax. You can do this with any of the following:

`flights$flight`
`flights[["flight"]]`
`flights[[11]]`

If you run one of these, you'll notice it just returns a vector of numbers corresponding with the values that were in the `flight` column.

This is now an atomic vector and no longer a dataframe. Much of the meta-data has been stripped or changed.
*Looking at each of these with `glimpse` can show some these differences*

``` r
glimpse(flights$flight)
```

    ##  int [1:336776] 1545 1714 1141 725 461 1696 507 5708 79 301 ...

``` r
glimpse(select(flights, flight))
```

    ## Observations: 336,776
    ## Variables: 1
    ## $ flight <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4...

Functions that take dfs
-----------------------

What may be helpful to understand is that functions like `mutate`, `filter`, `select`, etc., are expecting dataframes as their first argument, so if you pass them vectors, they are going to fail.
*E.g. the following outputs a dataframe in the firstline and pipes this to the first argument in the second line so the code works:*

``` r
flights["flight"] %>% 
  select(flight)
```

    ## # A tibble: 336,776 x 1
    ##    flight
    ##     <int>
    ##  1   1545
    ##  2   1714
    ##  3   1141
    ##  4    725
    ##  5    461
    ##  6   1696
    ##  7    507
    ##  8   5708
    ##  9     79
    ## 10    301
    ## # ... with 336,766 more rows

The first line produces a one-columned dataframe. This is then piped into the second line that again selects the `flight` column giving the final output. This is also equivalent to the following:

``` r
select(flights, flight) %>% 
  select(flight)
```

    ## # A tibble: 336,776 x 1
    ##    flight
    ##     <int>
    ##  1   1545
    ##  2   1714
    ##  3   1141
    ##  4    725
    ##  5    461
    ##  6   1696
    ##  7    507
    ##  8   5708
    ##  9     79
    ## 10    301
    ## # ... with 336,766 more rows

The second line in both of these cases is actually completely redundant and you would get the same output with just `select(flights, flight)` or `flights["flight"]`.

You could even run this to get a 0 columned dataframe...

``` r
select(flights, flight) %>% 
  select(-flight)
```

    ## # A tibble: 336,776 x 0

However, the following produces a vector in it's first line and tries to pipe this into the select statement in the second line.

``` r
flights$flight %>% 
  select(flight)
```

    ## Error in UseMethod("select_"): no applicable method for 'select_' applied to an object of class "c('integer', 'numeric')"

The select statement is expecting a dataframe[3] so the code will fail[4].

Functions that take vectors
---------------------------

### Expecting vector inputs

Functions like `mean`, `sum`, `sd`, `median` etc. are expecting vectors, not dataframes.

So this works:

``` r
median(flights$flight)
```

    ## [1] 1496

But this does not.

``` r
median(select(flights, flight))
```

    ## Error in median.default(select(flights, flight)): need numeric data

Be careful though, some function may give output when they probably shouldn't notice that in this case it outputted NA and simply gave you a warning (as opposed to an error and no output).

``` r
mean(select(flights, flight))
```

    ## Warning in mean.default(select(flights, flight)): argument is not numeric
    ## or logical: returning NA

    ## [1] NA

Others may automatically convert it to a vector for you and not throw any problem which may be fine in some cases but problematic in others[5].

### Num values outputted

The functions just mentioned like `mean`, `sum` etc. take in multiple values and output just a single value.

Many functions in R though output a value for every item inputted. For example, the logical operators `+`, `-`, `!`, etc.

``` r
x <- c(1:10)

x + 2
```

    ##  [1]  3  4  5  6  7  8  9 10 11 12

``` r
y <- c(rep(TRUE, 5), rep(FALSE, 5))
!y
```

    ##  [1] FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE

As well as functions like `ifelse`

``` r
x <- c(1:10)
x
```

    ##  [1]  1  2  3  4  5  6  7  8  9 10

``` r
ifelse(x > 4, "greater than 4", "less than 4")
```

    ##  [1] "less than 4"    "less than 4"    "less than 4"    "less than 4"   
    ##  [5] "greater than 4" "greater than 4" "greater than 4" "greater than 4"
    ##  [9] "greater than 4" "greater than 4"

`ifelse` function.

### Dplyr and outputs

Dplyr's `mutate` handles functions differently depending on if the function outputs a single value for a vector of inputs or if it outputs a value for every vector of inputs. The example below shows how these differ.

``` r
mtcars %>% 
  mutate(function_giving_single_output = sum(mpg),
         function_giving_multiple_outputs = mpg + 7)
```

    ## Warning: package 'bindrcpp' was built under R version 3.4.4

    ##     mpg cyl  disp  hp drat    wt  qsec vs am gear carb
    ## 1  21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
    ## 2  21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
    ## 3  22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
    ## 4  21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
    ## 5  18.7   8 360.0 175 3.15 3.440 17.02  0  0    3    2
    ## 6  18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
    ## 7  14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
    ## 8  24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
    ## 9  22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
    ## 10 19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
    ## 11 17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
    ## 12 16.4   8 275.8 180 3.07 4.070 17.40  0  0    3    3
    ## 13 17.3   8 275.8 180 3.07 3.730 17.60  0  0    3    3
    ## 14 15.2   8 275.8 180 3.07 3.780 18.00  0  0    3    3
    ## 15 10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
    ## 16 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
    ## 17 14.7   8 440.0 230 3.23 5.345 17.42  0  0    3    4
    ## 18 32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
    ## 19 30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
    ## 20 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
    ## 21 21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
    ## 22 15.5   8 318.0 150 2.76 3.520 16.87  0  0    3    2
    ## 23 15.2   8 304.0 150 3.15 3.435 17.30  0  0    3    2
    ## 24 13.3   8 350.0 245 3.73 3.840 15.41  0  0    3    4
    ## 25 19.2   8 400.0 175 3.08 3.845 17.05  0  0    3    2
    ## 26 27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
    ## 27 26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
    ## 28 30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
    ## 29 15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4
    ## 30 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
    ## 31 15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8
    ## 32 21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
    ##    function_giving_single_output function_giving_multiple_outputs
    ## 1                          642.9                             28.0
    ## 2                          642.9                             28.0
    ## 3                          642.9                             29.8
    ## 4                          642.9                             28.4
    ## 5                          642.9                             25.7
    ## 6                          642.9                             25.1
    ## 7                          642.9                             21.3
    ## 8                          642.9                             31.4
    ## 9                          642.9                             29.8
    ## 10                         642.9                             26.2
    ## 11                         642.9                             24.8
    ## 12                         642.9                             23.4
    ## 13                         642.9                             24.3
    ## 14                         642.9                             22.2
    ## 15                         642.9                             17.4
    ## 16                         642.9                             17.4
    ## 17                         642.9                             21.7
    ## 18                         642.9                             39.4
    ## 19                         642.9                             37.4
    ## 20                         642.9                             40.9
    ## 21                         642.9                             28.5
    ## 22                         642.9                             22.5
    ## 23                         642.9                             22.2
    ## 24                         642.9                             20.3
    ## 25                         642.9                             26.2
    ## 26                         642.9                             34.3
    ## 27                         642.9                             33.0
    ## 28                         642.9                             37.4
    ## 29                         642.9                             22.8
    ## 30                         642.9                             26.7
    ## 31                         642.9                             22.0
    ## 32                         642.9                             28.4

`sum(mpg)` outputted a single value for all rows. If you had first done a `group_by` statement, you would have gotten a single value for every group.

You'll notice in the example below that the `...single_output` only has 3 unique values, each corresponding with the `cyl` varibale used in the `group_by` whereas the `...multiple_ouputs` values are the exact same as without the `group_by` as the `mpg + 7` function operates at the row level, so the grouping here doesn't affect the output[6].

``` r
mtcars %>% 
  group_by(cyl) %>%
  mutate(function_giving_single_output = sum(mpg),
         function_giving_multiple_outputs = mpg + 7)
```

    ## # A tibble: 32 x 13
    ## # Groups:   cyl [3]
    ##      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
    ##    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1  21.0    6.  160.  110.  3.90  2.62  16.5    0.    1.    4.    4.
    ##  2  21.0    6.  160.  110.  3.90  2.88  17.0    0.    1.    4.    4.
    ##  3  22.8    4.  108.   93.  3.85  2.32  18.6    1.    1.    4.    1.
    ##  4  21.4    6.  258.  110.  3.08  3.22  19.4    1.    0.    3.    1.
    ##  5  18.7    8.  360.  175.  3.15  3.44  17.0    0.    0.    3.    2.
    ##  6  18.1    6.  225.  105.  2.76  3.46  20.2    1.    0.    3.    1.
    ##  7  14.3    8.  360.  245.  3.21  3.57  15.8    0.    0.    3.    4.
    ##  8  24.4    4.  147.   62.  3.69  3.19  20.0    1.    0.    4.    2.
    ##  9  22.8    4.  141.   95.  3.92  3.15  22.9    1.    0.    4.    2.
    ## 10  19.2    6.  168.  123.  3.92  3.44  18.3    1.    0.    4.    4.
    ## # ... with 22 more rows, and 2 more variables:
    ## #   function_giving_single_output <dbl>,
    ## #   function_giving_multiple_outputs <dbl>

The `mutate` function outputs the same number of rows as are in the input dataframe, however the `summarise` funciton outputs the number of rows as there are groups. E.g.

``` r
mtcars %>% 
  group_by(cyl) %>%
  summarise(function_giving_single_output = sum(mpg))
```

    ## # A tibble: 3 x 2
    ##     cyl function_giving_single_output
    ##   <dbl>                         <dbl>
    ## 1    4.                          293.
    ## 2    6.                          138.
    ## 3    8.                          211.

This means that functions outputting a value for each input will cause an error to occur.

``` r
mtcars %>% 
  group_by(cyl) %>%
  summarise(function_giving_single_output = sum(mpg),
         function_giving_multiple_outputs = mpg + 7)
```

    ## Error in summarise_impl(.data, dots): Column `function_giving_multiple_outputs` must be length 1 (a summary value), not 11

You may be wondering why the error message reads 'must be length 1... not 11' when there are actually 32 rows in the dataframe and not 11. This is because the error happens on each individual grouped vector and the first error occurs on the vector associated with `cyl` 4, which is of length 11.

``` r
count(mtcars)
```

    ## # A tibble: 1 x 1
    ##       n
    ##   <int>
    ## 1    32

If you had no `group_by` statement you would see the error message would reference the length for the entire dataframe.

``` r
mtcars %>% 
  summarise(function_giving_single_output = sum(mpg),
         function_giving_multiple_outputs = mpg + 7)
```

    ## Error in summarise_impl(.data, dots): Column `function_giving_multiple_outputs` must be length 1 (a summary value), not 32

You can though specify mult\_output functions within aggregating functions and it work as expected. For example, the function below will show the percentage of cars with mpg greater than 20 for each `cyl` value

``` r
mtcars %>% 
  group_by(cyl) %>% 
  summarise(function_giving_single_output = mean(mpg > 20))
```

    ## # A tibble: 3 x 2
    ##     cyl function_giving_single_output
    ##   <dbl>                         <dbl>
    ## 1    4.                         1.00 
    ## 2    6.                         0.429
    ## 3    8.                         0.

### More than 1, less than all outputs

Dplyr mostly handles functions that output either a single value for each group, or multiple values for each group.

DPLYR and non-standard evaluation
---------------------------------

### Part 1

Functions like `mutate` `filter` `summarise`, etc. use 'non-standard evaluation' that allow you to specify the names of the columns in functions like `mean` and `median` that dplyr then passes as vectors parsed by w/e `group_by` or other specifications are set.

``` r
select(flights, origin, dep_delay) %>% 
  group_by(origin) %>% 
  summarise(dep_delay_mean = mean(dep_delay, na.rm = TRUE))
```

    ## # A tibble: 3 x 2
    ##   origin dep_delay_mean
    ##   <chr>           <dbl>
    ## 1 EWR              15.1
    ## 2 JFK              12.1
    ## 3 LGA              10.3

It is kind of like doing each of the following...[7]

``` r
mean(flights$dep_delay[flights$origin == "EWR"], na.rm = TRUE)
```

    ## [1] 15.10795

``` r
mean(flights$dep_delay[flights$origin == "JFK"], na.rm = TRUE)
```

    ## [1] 12.11216

``` r
mean(flights$dep_delay[flights$origin == "LGA"], na.rm = TRUE)
```

    ## [1] 10.34688

For now, I would not worry about base R's subsetting syntax too much, we'll talk about it in chapter 20 and it is something that we will touch-on sporadically throughout the book[8]. In generaly though it won't be our focus.

### Part 2, subsetting in Dplyr calcs

Vincent pointed out this method in our discussion in week 3[9]

Say your goal is to calculate the average `arr_delay` by origin and only want to include those values that are actually delayed, i.e. have an `arr_delay` value greater than 0, and you don't want to preface your `summarise` with a `filter`[10]. You can use brackets to subset to just those values that meet the condition.

``` r
select(flights, origin, arr_delay) %>% 
  group_by(origin) %>% 
  summarise(arr_delay_late_mean = mean(arr_delay[arr_delay > 0], na.rm = TRUE))
```

    ## # A tibble: 3 x 2
    ##   origin arr_delay_late_mean
    ##   <chr>                <dbl>
    ## 1 EWR                   41.8
    ## 2 JFK                   40.0
    ## 3 LGA                   38.9

You may wonder, why not just perform a filter before hand. You could do this, but in some situations you may want to have summarized values on different subsets of the values so this can become inconvenient.. For example, say you want to calculate the average arrival delay on all flights[11] but then also on just flights that had an arrival delay that was greater than zero[12]. These are represented below by `arr_delay_all_mean` and `arr_delay_only_late_mean` respecticely.

``` r
select(flights, origin, arr_delay) %>% 
  group_by(origin) %>% 
  summarise(
    arr_delay_all_mean = mean(arr_delay, na.rm = TRUE),
    arr_delay_late_mean = mean(arr_delay[arr_delay > 0], na.rm = TRUE))
```

    ## # A tibble: 3 x 3
    ##   origin arr_delay_all_mean arr_delay_late_mean
    ##   <chr>               <dbl>               <dbl>
    ## 1 EWR                  9.11                41.8
    ## 2 JFK                  5.55                40.0
    ## 3 LGA                  5.78                38.9

The brackets in this case signify that the mean is being calculated just on those rows that satisfy the condition. If the subset is not in brackets it does not become a subset but a vector of `TRUE` `FALSE` values. In the example below `summarise` is instead returning the proportion of flights that have an `arr_delay` greater than 0.

``` r
select(flights, origin, arr_delay) %>% 
  group_by(origin) %>% 
  summarise(
    arr_delay_all_mean = mean(arr_delay, na.rm = TRUE),
    arr_delay_late_mean = mean(arr_delay > 0, na.rm = TRUE))
```

    ## # A tibble: 3 x 3
    ##   origin arr_delay_all_mean arr_delay_late_mean
    ##   <chr>               <dbl>               <dbl>
    ## 1 EWR                  9.11               0.428
    ## 2 JFK                  5.55               0.393
    ## 3 LGA                  5.78               0.396

[1] Note that dataframes and tibbles are essentially the same thing, tibbles are just dataframes with a few nice properties. There are occasionally instances where they behave a little differently, but generally they are the same. We will typically use the phrase "dataframe" as a catch-all.

[2] A tip for searching stack overflow boards with questions is to include the tag "dplyr" or "tidyverse" on your searches, this will help filter to approaches that are consistent with the style we will be using. Base R stack overflow solutions can be tough to follow if you don't have a strong backgorund there.

[3] vectors don't have key things like column headers.

[4] Use the error and warning messages as helpers when testing thing out.

[5] Some of these inconsistencies in R are what have contributed to the rise of the tidyverse

[6] When there is no grouping it's as if the entire dataframe is a single group

[7] Not exactly like this though because DPLYR is actually running in C in the backend

[8] However if you have an itch see Advanced R: <http://adv-r.had.co.nz/Subsetting.html> for a thorough treatment on the subject

[9] I initially recommended using an `ifelse`, but would definitely use method mentioned below.

[10] Continue reading for why you may not want to do this.

[11] This might be thought of as the average difference in time between when the flight arrived and when it was scheduled to arrive.

[12] This can be thought of as the average amount of delay for flights that were actually delayed.
