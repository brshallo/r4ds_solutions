



# Ch. 21: Iteration

* Common `for` loop template:

```r
output <- vector("double", ncol(df)) # common for loop style  
for (i in seq_len(length(df))){
  output[[i]] <- fun(df[[i]])
  }  
```

* Common `while` loop template:

```r
i <- 1
while (i <= length(x)){
  # body
  i <- i + 1
}  
```

* `seq_along(df)` does essentially same as `seq_len(length(df))`
* `unlist` flatten list of vectors into single vector
    + `flaten_dbl` is stricter alternative
* `dplyr::bind_rows` save output in a list of dfs and then append all at end rather than sequential `rbind`ing
* `sample(c("T", "H"), 1)`
* `sapply` is wrapper around `lapply` that automatically simplifies output -- problematic in that never know what ouptut will be
* `vapply` is safe alternative to `sapply` e.g. for logical `vapply(df, is.numeric, logical(1))`, but `map_lgl(df, is.numeric)` is more simple
* `map()`     makes a list.
  * `map_lgl()` makes a logical vector.
  * `map_int()` makes an integer vector.
  * `map_dbl()` makes a double vector.
  * `map_chr()` makes a character vector.
* shortcuts for applying functions in `map`:

```r
models <- mtcars %>% 
  split(.$cyl) %>% 
  map(function(df) lm(mpg ~ wt, data = df))

models <- mtcars %>% 
  split(.$cyl) %>% 
  map(~lm(mpg ~ wt, data = .))
```

* extracting by named elements from `map`:

```r
models %>% 
  map(summary) %>% 
  map_dbl("r.squared")
```

* extracting by positions from `map`

```r
x <- list(list(1, 2, 3), list(4, 5, 6), list(7, 8, 9))
x %>% 
  map_dbl(2)
```

* `map2` let's you iterate through two components at once
* `pmap` allows you to iterate over p components -- works well to hold inputs in a dataframe
* `safely` takes funciton returns two parts, result and error object
  + similar to `try` but more consistent
* `possibly` similar to safely, but provide it a default value to return for errors
* `quietly` is similar to safely but captures all printed output messages and warnings
* `purrr::transpose` allows you to do things like get all 2nd elements in list, e.g. show later
* `invoke_map` let's you iterate over both the functions and the parameters, have an `f` and a `param` input, e.g. 

```r
f <- c("runif", "rnorm", "rpois")
param <- list(
  list(min = -1, max = 1), 
  list(sd = 5), 
  list(lambda = 10)
)

invoke_map(f, param, n = 5) %>% str()
```

* `walk` is alternative to `map` that you call for side effects. Also have `walk2` and `pwalk` that are generally more useful 
  + all invisibly return `.x (the first argument) so can used in the middle of pipelines
* `keep` and `discard` keep or discard elements in the input based off if `TRUE` to predicate
* `some` and `every` determine if the predicte is true for any or for all of our elements
* `detect` finds the first element where the predicate is true, `detect_index` returns its position
* `head_while` and `tail_while` take elements from the start or end of a vector while a predicate is true
* `reduce` is good for applying two table rule repeatedly, e.g. joins
  * `accumulate` is similar but keeps all the interim results

## 21.2: For loops

### 21.2.1

1.  Write for loops to (think about the output, sequence, and body __before__ you start writing
    the loop):

    1. Compute the mean of every column in `mtcars`.
    
    
    ```r
    output <- vector("double", length(mtcars))
    for (i in seq_along(mtcars)){
      output[[i]] <- mean(mtcars[[i]])
    }
    output
    ```
    
    ```
    ##  [1]  20.090625   6.187500 230.721875 146.687500   3.596563   3.217250
    ##  [7]  17.848750   0.437500   0.406250   3.687500   2.812500
    ```
    
    1. Determine the type of each column in `nycflights13::flights`.
    
    
    ```r
    output <- vector("character", length(flights))
    for (i in seq_along(flights)){
      output[[i]] <- typeof(flights[[i]])
    }
    output
    ```
    
    ```
    ##  [1] "integer"   "integer"   "integer"   "integer"   "integer"  
    ##  [6] "double"    "integer"   "integer"   "double"    "character"
    ## [11] "integer"   "character" "character" "character" "double"   
    ## [16] "double"    "double"    "double"    "double"
    ```
    
    1. Compute the number of unique values in each column of `iris`.
    
    
    ```r
    output <- vector("integer", length(iris))
    for (i in seq_along(iris)){
      output[[i]] <- unique(iris[[i]]) %>% length()
    }
    output
    ```
    
    ```
    ## [1] 35 23 43 22  3
    ```

    1. Generate 10 random normals for each of $\mu = -10$, $0$, $10$, and $100$.
    
    ```r
    output <- vector("list", 4)
    input_means <- c(-10, 0, 10, 100)
    for (i in seq_along(output)){
      output[[i]] <- rnorm(10, mean = input_means[[i]])
    }
    output
    ```
    
    ```
    ## [[1]]
    ##  [1]  -8.785447 -10.293334  -9.964542 -11.961963 -10.367496  -9.966689
    ##  [7]  -9.872905 -10.122236  -9.680880 -10.624439
    ## 
    ## [[2]]
    ##  [1]  0.4470057 -0.9311754 -0.8643282 -0.3251063  0.4204115 -0.1076904
    ##  [7]  0.8208670  2.5297311 -0.4647587 -0.1917760
    ## 
    ## [[3]]
    ##  [1] 10.048968  8.442432 11.754288 10.238710 10.336530 10.343382  9.665342
    ##  [8]  8.151330  9.335686 10.920130
    ## 
    ## [[4]]
    ##  [1] 100.03146  98.89348 101.76771  97.63465  99.39744  98.84070 100.42941
    ##  [8]  98.64100 100.20938 100.31676
    ```
    

1.  Eliminate the for loop in each of the following examples by taking advantage of an existing function that works with vectors:
    
    *example:*
    
    ```r
    out <- ""
    for (x in letters) {
      out <- stringr::str_c(out, x)
    }
    out
    ```
    
    * collabse letters into length-one character vector with all characters concatenated
    
    ```r
    str_c(letters, collapse = "")
    ```
    
    ```
    ## [1] "abcdefghijklmnopqrstuvwxyz"
    ```
    
    *example:*
    
    ```r
    x <- sample(100)
    sd <- 0
    for (i in seq_along(x)) {
      sd <- sd + (x[i] - mean(x)) ^ 2
    }
    sd <- sqrt(sd / (length(x) - 1))
    sd
    ```
    
    ```
    ## [1] 29.01149
    ```
    
    * calculate standard deviaiton of x
    
    ```r
    sd(x)
    ```
    
    ```
    ## [1] 29.01149
    ```
    
    *example:*
    
    ```r
    x <- runif(100)
    out <- vector("numeric", length(x))
    out[1] <- x[1]
    for (i in 2:length(x)) {
      out[i] <- out[i - 1] + x[i]
    }
    out
    ```
    
    ```
    ##   [1]  0.1507737  0.3819632  0.6190446  0.8712783  1.6611736  2.6142845
    ##   [7]  2.7705236  2.8276411  3.3052075  3.7373471  4.1647731  4.6092924
    ##  [13]  5.0712058  5.6332756  5.6412381  6.1453597  6.9894742  7.3962568
    ##  [19]  7.9683878  7.9775943  8.5355966  9.3105778 10.0210178 10.8283078
    ##  [25] 11.6774369 12.2769223 13.1744781 13.5807259 14.2414668 14.9237279
    ##  [31] 15.0793143 15.3206245 15.6125462 15.7847346 16.4025312 16.5435509
    ##  [37] 17.5336369 17.9557732 17.9978000 18.9842538 19.0724417 19.7118555
    ##  [43] 20.1929384 21.0516417 21.7933057 22.5605165 22.5941937 22.8235250
    ##  [49] 23.0230483 23.5756079 23.6979485 23.8287348 24.0399448 24.6032246
    ##  [55] 25.4171866 26.2972757 26.5040229 27.3059268 27.8214695 28.4986734
    ##  [61] 28.7962282 29.4559282 29.7051632 29.7354231 29.8331492 30.5732860
    ##  [67] 31.4221682 32.0849093 32.9584508 33.0259334 33.7788410 33.9608278
    ##  [73] 34.4482053 34.8517571 34.9063350 35.4285265 35.9518918 36.6849530
    ##  [79] 36.7249350 37.0182335 37.5313990 38.4370851 39.4363856 39.5972196
    ##  [85] 39.9071955 40.5816351 40.6212861 41.3229752 41.5610946 42.1284802
    ##  [91] 43.0666150 43.9352343 44.1646167 44.3314841 45.2736909 46.0071531
    ##  [97] 46.4010338 47.0237101 47.3602063 47.5998132
    ```
    
    * calculate cumulative sum
    
    ```r
    cumsum(x)
    ```
    
    ```
    ##   [1]  0.1507737  0.3819632  0.6190446  0.8712783  1.6611736  2.6142845
    ##   [7]  2.7705236  2.8276411  3.3052075  3.7373471  4.1647731  4.6092924
    ##  [13]  5.0712058  5.6332756  5.6412381  6.1453597  6.9894742  7.3962568
    ##  [19]  7.9683878  7.9775943  8.5355966  9.3105778 10.0210178 10.8283078
    ##  [25] 11.6774369 12.2769223 13.1744781 13.5807259 14.2414668 14.9237279
    ##  [31] 15.0793143 15.3206245 15.6125462 15.7847346 16.4025312 16.5435509
    ##  [37] 17.5336369 17.9557732 17.9978000 18.9842538 19.0724417 19.7118555
    ##  [43] 20.1929384 21.0516417 21.7933057 22.5605165 22.5941937 22.8235250
    ##  [49] 23.0230483 23.5756079 23.6979485 23.8287348 24.0399448 24.6032246
    ##  [55] 25.4171866 26.2972757 26.5040229 27.3059268 27.8214695 28.4986734
    ##  [61] 28.7962282 29.4559282 29.7051632 29.7354231 29.8331492 30.5732860
    ##  [67] 31.4221682 32.0849093 32.9584508 33.0259334 33.7788410 33.9608278
    ##  [73] 34.4482053 34.8517571 34.9063350 35.4285265 35.9518918 36.6849530
    ##  [79] 36.7249350 37.0182335 37.5313990 38.4370851 39.4363856 39.5972196
    ##  [85] 39.9071955 40.5816351 40.6212861 41.3229752 41.5610946 42.1284802
    ##  [91] 43.0666150 43.9352343 44.1646167 44.3314841 45.2736909 46.0071531
    ##  [97] 46.4010338 47.0237101 47.3602063 47.5998132
    ```
    

1.  Combine your function writing and for loop skills:

    1. Write a for loop that `prints()` the lyrics to the children's song "Alice the camel".

    
    ```r
    num_humps <- c("five", "four", "three", "two", "one", "no")
    
    for (i in seq_along(num_humps)){
      
      paste0("Alice the camel has ", num_humps[[i]], " humps.") %>% 
        rep(3) %>% 
        writeLines()
      
      writeLines("So go, Alice, go.\n")
    }
    ```
    
    ```
    ## Alice the camel has five humps.
    ## Alice the camel has five humps.
    ## Alice the camel has five humps.
    ## So go, Alice, go.
    ## 
    ## Alice the camel has four humps.
    ## Alice the camel has four humps.
    ## Alice the camel has four humps.
    ## So go, Alice, go.
    ## 
    ## Alice the camel has three humps.
    ## Alice the camel has three humps.
    ## Alice the camel has three humps.
    ## So go, Alice, go.
    ## 
    ## Alice the camel has two humps.
    ## Alice the camel has two humps.
    ## Alice the camel has two humps.
    ## So go, Alice, go.
    ## 
    ## Alice the camel has one humps.
    ## Alice the camel has one humps.
    ## Alice the camel has one humps.
    ## So go, Alice, go.
    ## 
    ## Alice the camel has no humps.
    ## Alice the camel has no humps.
    ## Alice the camel has no humps.
    ## So go, Alice, go.
    ```

    2. Convert the nursery rhyme "ten in the bed" to a function. Generalise it to any number of people in any sleeping structure.

    
    ```r
    nursery_bed <- function(num, y) {
      output <- vector("character", num)
      for (i in seq_along(output)) {
        output[[i]] <- str_replace_all(
        'There were x in the _y\n And the little one said, \n"Roll over! Roll over!"\n So they all rolled over and\n one fell out.', c("x" = (length(output) - i + 1), "_y" = y))
      } 
      str_c(output, collapse = "\n\n") %>% 
        writeLines()
    }
    
    nursery_bed(3, "asteroid")
    ```
    
    ```
    ## There were 3 in the asteroid
    ##  And the little one said, 
    ## "Roll over! Roll over!"
    ##  So they all rolled over and
    ##  one fell out.
    ## 
    ## There were 2 in the asteroid
    ##  And the little one said, 
    ## "Roll over! Roll over!"
    ##  So they all rolled over and
    ##  one fell out.
    ## 
    ## There were 1 in the asteroid
    ##  And the little one said, 
    ## "Roll over! Roll over!"
    ##  So they all rolled over and
    ##  one fell out.
    ```
    
    3. Convert the song "99 bottles of beer on the wall" to a function. Generalise to any number of any vessel containing any liquid on any surface.  
       
    * This is a little bit of a lazy version...
       
    
    ```r
    beer_rhyme <- function(x, y, z){
      output <- vector("character", x)
      for (i in seq_along(output)){
        output[i] <-
          str_replace_all("x bottles of y on the z.\n One fell off...", c(
          "x" = (x - i + 1),
          "y" = y,
          "z" = z
          ))
      }
      output <- (str_c(output, collapse = "\n") %>% 
                   str_c("\nNo more bottles...", collapse = ""))
      writeLines(output)
    }
    
    beer_rhyme(4, "soda", "toilet")
    ```
    
    ```
    ## 4 bottles of soda on the toilet.
    ##  One fell off...
    ## 3 bottles of soda on the toilet.
    ##  One fell off...
    ## 2 bottles of soda on the toilet.
    ##  One fell off...
    ## 1 bottles of soda on the toilet.
    ##  One fell off...
    ## No more bottles...
    ```
    
1.  It's common to see for loops that don't preallocate the output and instead increase the length of a vector at each step. How does this affect performance? Design and execute an experiment.
    
    
    ```r
    preallocate <- function(){
    x <- vector("double", 100)
      for (i in seq_along(x)){
        x[i] <- rnorm(1)
      }
    }
    
    growing <- function(){
      x <- c(0)
        for (i in 1:100){
          x[i] <- rnorm(1)
      }
    }
    
    microbenchmark::microbenchmark(
      space = preallocate(),
      no_space = growing(),
      times = 20
    )  
    ```
    
    ```
    ## Unit: microseconds
    ##      expr     min       lq     mean   median       uq      max neval cld
    ##     space 127.801 134.2515 360.3713 205.9515 230.4515 3622.402    20   a
    ##  no_space 151.701 214.5010 490.7660 245.9515 296.8505 3616.800    20   a
    ```

    * see roughly 35% better performance when creating ahead of time
    * note: if you can do these operations with vectorized approach though -- they're often much faster
    
    
    ```r
    microbenchmark::microbenchmark(
      space = preallocate(),
      no_space = growing(),
      vector = rnorm(100),
      times = 20
    )
    ```
    
    ```
    ## Unit: microseconds
    ##      expr     min       lq     mean  median      uq     max neval cld
    ##     space 130.701 135.7515 160.3561 140.651 166.751 308.601    20  b 
    ##  no_space 158.302 173.6510 236.4610 213.901 293.551 386.901    20   c
    ##    vector   7.301   7.6005   9.0412   8.052   8.601  18.001    20 a
    ```
    
    * vectorized was > 10x faster

## 21.3 For loop variations

### 21.3.5

1.  Imagine you have a directory full of CSV files that you want to read in. You have their paths in a vector, `files <- dir("data/", pattern = "\\.csv$", full.names = TRUE)`, and now want to read each one with `read_csv()`. Write the for loop that will load them into a single data frame. 
    
    * To start this problem, I first created a file directory, and then wrote in 26 csvs each with the most popular name from each year since 1880 for a particular letter[^WalkExample].
    * Next I read these into a single dataframe with a for loop
        
    [^WalkExample]: 
        Below is the code that accomplished this. I used `walk2` and methods we learn later in the chapter.

    
    ```r
    dir.create("ch21_csvs_example")
    
    babynames %>% 
      mutate(first_letter = str_sub(name, 1, 1)) %>% 
      group_by(first_letter, year) %>% 
      filter(dplyr::min_rank(-prop) == 1) %>%  
      split(.$first_letter) %>% 
      # map(~select(.x, -first_letter)) %>% 
      walk2(.x = ., .y = names(.), 
            ~write_csv(.x,
                       paste0("ch21_csvs_example/", "letter_", .y, ".csv"))
            )
    ```

    
    ```r
    append_csvs <- function(dir){
      #input vector of file paths name and output appended file
      
      out <- vector("list", length(dir))
      for (i in seq_along(out)){
        out[[i]] <- read_csv(dir[[i]], col_types = cols(.default = "c"))
      }
      out <-  bind_rows(out) %>% 
        type_convert()
      out
    }
    
    dir_examp <- dir("ch21_csvs_example", 
        pattern = "csv$",
        full.names = TRUE)
    
    names_appended <- append_csvs(dir_examp)
    names_appended
    ```
    
    ```
    ## # A tibble: 3,514 x 6
    ##     year sex   name      n   prop first_letter
    ##    <dbl> <chr> <chr> <int>  <dbl> <chr>       
    ##  1  1880 F     Anna   2604 0.0267 A           
    ##  2  1881 F     Anna   2698 0.0273 A           
    ##  3  1882 F     Anna   3143 0.0272 A           
    ##  4  1883 F     Anna   3306 0.0275 A           
    ##  5  1884 F     Anna   3860 0.0281 A           
    ##  6  1885 F     Anna   3994 0.0281 A           
    ##  7  1886 F     Anna   4283 0.0279 A           
    ##  8  1887 F     Anna   4227 0.0272 A           
    ##  9  1888 F     Anna   4982 0.0263 A           
    ## 10  1889 F     Anna   5062 0.0268 A           
    ## # ... with 3,504 more rows
    ```

    * See [Using map] for example of how this could be accomplished using `map` and `map(safely(read_csv))`.

2.  *What happens if you use `for (nm in names(x))` and `x` has no names?*

    
    ```r
    x <- list(1:10, 11:18, 19:25)
    for (nm in names(x)) {
      print(x[[nm]])
    }
    ```
    
    * each iteration produces an error, so nothing is written
  
*What if only some of the elements are named? *

    
    ```r
    x <- list(a = 1:10, 11:18, c = 19:25)
    for (nm in names(x)) {
      print(x[[nm]])
    }
    ```
    
    ```
    ##  [1]  1  2  3  4  5  6  7  8  9 10
    ## NULL
    ## [1] 19 20 21 22 23 24 25
    ```
    
    * you have output for those with names and NULL for those without
  
*What if the names are not unique?*
    
    ```r
    x <- list(a = 1:10, a = 11:18, c = 19:25)
    for (nm in names(x)) {
      print(x[[nm]])
    }
    ```
    
    ```
    ##  [1]  1  2  3  4  5  6  7  8  9 10
    ##  [1]  1  2  3  4  5  6  7  8  9 10
    ## [1] 19 20 21 22 23 24 25
    ```
    
    * it prints the first position with the name
    
3.  Write a function that prints the mean of each numeric column in a data frame, along with its name. For example, `show_mean(iris)` would print:
    
    ```r
    show_mean(iris)
    #> Sepal.Length: 5.84
    #> Sepal.Width:  3.06
    #> Petal.Length: 3.76
    #> Petal.Width:  1.20
    ```

(Extra challenge: what function did I use to make sure that the numbers lined up nicely, even though the variable names had different lengths?)

    
    ```r
    show_mean <- function(df){
      # select just cols that are numeric
      out <- vector("logical", length(df))
      for (i in seq_along(df)) {
        out[[i]] <- is.numeric(df[[i]])
      } 
      df_select <- df[out]
      # keep/discard funs would have made this easy
      
      # make list of values w/ mean
      means <- vector("list", length(df_select))
      names(means) <- names(df_select)
      for (i in seq_along(df_select)){
        means[[i]] <- mean(df_select[[i]], na.rm = TRUE) %>%
          round(digits = 2)
      }
      
      # print out, use method to identify max chars for vars printed
      means_names <- names(means)
      chars_max <- (str_count(means_names) + str_count(as.character(means))) %>% max()
      chars_pad <- chars_max - (str_count(means_names) + str_count(as.character(means)))
      names(chars_pad) <- means_names
      
    
    str_c(means_names, ": ", str_dup(" ", chars_pad), means) %>% 
      writeLines()
    }
    
    show_mean(flights)
    ```
    
    ```
    ## year:              2013
    ## month:             6.55
    ## day:              15.71
    ## dep_time:       1349.11
    ## sched_dep_time: 1344.25
    ## dep_delay:        12.64
    ## arr_time:       1502.05
    ## sched_arr_time: 1536.38
    ## arr_delay:          6.9
    ## flight:         1971.92
    ## air_time:        150.69
    ## distance:       1039.91
    ## hour:             13.18
    ## minute:           26.23
    ```

4.  What does this code do? How does it work?

    
    ```r
    trans <- list( 
      disp = function(x) x * 0.0163871,
      am = function(x) {
        factor(x, labels = c("auto", "manual"))
      }
    )
    for (var in names(trans)) {
      mtcars[[var]] <- trans[[var]](mtcars[[var]])
    }
    mtcars
    ```

    * first part builds list of functions, 2nd applies those to a dataset
    * are storing the data transformations as a function and then applying this to a dataframe ^[This is a very powerful practice because it allows you to save / keep track of your manipulations and apply them at other locations, while keeping the logic very well organized -- go and use this for documenting your work / transformations]
    
## 21.4: For loops vs. functionals

### 21.4.1

1.  Read the documentation for `apply()`. In the 2d case, what two for loops does it generalise?
    
    * It allows you to input either 1 or 2 for the `MARGIN` argument, which corresponds with looping over either the rows or the columns.
  
1.  Adapt `col_summary()` so that it only applies to numeric columns You might want to start with an `is_numeric()` function that returns a logical vector that has a TRUE corresponding to each numeric column.
    
    
    ```r
    col_summary_gen <- function(df, fun, ...) {
      #find cols that are numeric
      out <- vector("logical", length(df))
      for (i in seq_along(df)) {
        out[[i]] <- is.numeric(df[[i]])
      }
      #make list of values w/ mean
      df_select <- df[out]
      output <- vector("list", length(df_select))
      names(output) <- names(df_select)
      for (nm in names(output)) {
        output[[nm]] <- fun(df_select[[nm]], ...) %>% 
          round(digits = 2)
      }
      
      as_tibble(output)
    }
    
    col_summary_gen(flights, fun = median, na.rm = TRUE) %>% 
      gather() # trick to gather all easily
    ```
    
    ```
    ## # A tibble: 14 x 2
    ##    key            value
    ##    <chr>          <dbl>
    ##  1 year            2013
    ##  2 month              7
    ##  3 day               16
    ##  4 dep_time        1401
    ##  5 sched_dep_time  1359
    ##  6 dep_delay         -2
    ##  7 arr_time        1535
    ##  8 sched_arr_time  1556
    ##  9 arr_delay         -5
    ## 10 flight          1496
    ## 11 air_time         129
    ## 12 distance         872
    ## 13 hour              13
    ## 14 minute            29
    ```
    
    * the `...` makes this so you can add arguments to the functions.    
    
## 21.5: The map functions

### 21.5.3

1.  Write code that uses one of the map functions to:

    *Compute the mean of every column in `mtcars`.*
    
    ```r
    purrr::map_dbl(mtcars, mean)
    ```
    
    ```
    ##        mpg        cyl       disp         hp       drat         wt 
    ##  20.090625   6.187500 230.721875 146.687500   3.596563   3.217250 
    ##       qsec         vs         am       gear       carb 
    ##  17.848750   0.437500   0.406250   3.687500   2.812500
    ```
    
    *Determine the type of each column in `nycflights13::flights`.*
    
    
    ```r
    purrr::map_chr(flights, typeof)
    ```
    
    ```
    ##           year          month            day       dep_time sched_dep_time 
    ##      "integer"      "integer"      "integer"      "integer"      "integer" 
    ##      dep_delay       arr_time sched_arr_time      arr_delay        carrier 
    ##       "double"      "integer"      "integer"       "double"    "character" 
    ##         flight        tailnum         origin           dest       air_time 
    ##      "integer"    "character"    "character"    "character"       "double" 
    ##       distance           hour         minute      time_hour 
    ##       "double"       "double"       "double"       "double"
    ```
    
    *Compute the number of unique values in each column of `iris`.*
    
    
    ```r
    purrr::map(iris, unique) %>% 
      map_dbl(length)
    ```
    
    ```
    ## Sepal.Length  Sepal.Width Petal.Length  Petal.Width      Species 
    ##           35           23           43           22            3
    ```
    
    *Generate 10 random normals for each of $\mu = -10$, $0$, $10$, and $100$.*
    
    
    ```r
    purrr::map_dbl(flights, ~mean(is.na(.x)))
    ```
    
    ```
    ##           year          month            day       dep_time sched_dep_time 
    ##    0.000000000    0.000000000    0.000000000    0.024511842    0.000000000 
    ##      dep_delay       arr_time sched_arr_time      arr_delay        carrier 
    ##    0.024511842    0.025871796    0.000000000    0.028000808    0.000000000 
    ##         flight        tailnum         origin           dest       air_time 
    ##    0.000000000    0.007458964    0.000000000    0.000000000    0.028000808 
    ##       distance           hour         minute      time_hour 
    ##    0.000000000    0.000000000    0.000000000    0.000000000
    ```

1.  How can you create a single vector that for each column in a data frame indicates whether or not it's a factor?
    
    
    ```r
    purrr::map_lgl(iris, is.factor) %>% 
      as_tibble() %>% 
      mutate(column_names = row.names(.))
    ```
    
    ```
    ## Warning: Calling `as_tibble()` on a vector is discouraged, because the behavior is likely to change in the future. Use `enframe(name = NULL)` instead.
    ## This warning is displayed once per session.
    ```
    
    ```
    ## # A tibble: 5 x 2
    ##   value column_names
    ##   <lgl> <chr>       
    ## 1 FALSE 1           
    ## 2 FALSE 2           
    ## 3 FALSE 3           
    ## 4 FALSE 4           
    ## 5 TRUE  5
    ```
    
    * for this example, I added in a simple extra little trick to convert to a tibble and include original column names as a field

1.  What happens when you use the map functions on vectors that aren't lists? What does `map(1:5, runif)` do? Why?
    
    
    ```r
    purrr::map(1:5, rnorm)
    ```
    
    ```
    ## [[1]]
    ## [1] 0.9776621
    ## 
    ## [[2]]
    ## [1] -0.33246063 -0.08414805
    ## 
    ## [[3]]
    ## [1]  0.4658980  0.2018112 -0.2532251
    ## 
    ## [[4]]
    ## [1]  0.07361983 -0.91885392 -0.44735643  0.04915594
    ## 
    ## [[5]]
    ## [1]  0.9836768  0.1853731  1.6546643 -0.2262159 -1.0050666
    ```
    
    * it runs on each item in the vector. In this case then it is passing the values 1, 2, 3, 4, 5 into the first argument of `rnorm`, hence pattern above.
      
1.  What does `map(-2:2, rnorm, n = 5)` do? Why?

    
    ```r
    map(-2:2, rnorm, n = 5)
    ```
    
    ```
    ## [[1]]
    ## [1] -2.649034 -3.358016 -3.175197 -2.295753 -2.134657
    ## 
    ## [[2]]
    ## [1] -2.9999616 -0.9958054  0.3877819 -0.8219817 -1.7794775
    ## 
    ## [[3]]
    ## [1] -1.6486251  0.8349202 -0.8334311 -0.1258769 -1.0915130
    ## 
    ## [[4]]
    ## [1]  1.62728540  1.91093048  2.21385852 -0.02791367 -0.09594475
    ## 
    ## [[5]]
    ## [1] 2.820463 1.305949 3.036895 2.907764 2.182459
    ```

    * It makes 5 vectors each of length 5 with the values centered at the means of -2,-1, 0, 1, 2 respectively. 
    * The reason is that the default filling of the first argument is already named by the defined input of 'n = 5', therefore, the inputs are instead going to the 2nd argument, and hence become the mean of the different rnorm calls.
    
1.  Rewrite `map(x, function(df) lm(mpg ~ wt, data = df))` to eliminate the anonymous function. 
    

```r
mtcars %>% 
  purrr::map( ~ lm(mpg ~ wt, data = .))
```
    
## 21.9 Other patterns of for loops

### 21.9.3

1.  Implement your own version of `every()` using a for loop. Compare it with `purrr::every()`. What does purrr's version do that your version doesn't?
    
    
    ```r
    every_loop <- function(x, fun, ...) {
      output <- vector("list", length(x))
      for (i in seq_along(x)) {
      output[[i]] <- fun(x[[i]])
      }
      total <- flatten_lgl(output)
      sum(total) == length(x)
    }
    
    x <- list(flights, mtcars, iris)
    every_loop(x, is.data.frame)
    ```
    
    ```
    ## [1] TRUE
    ```
    
    ```r
    every(x, is.data.frame)
    ```
    
    ```
    ## [1] TRUE
    ```
    
    * mine can't handle shortcut formulas or new functions    
    
    ```r
    z <- sample(10)
    z %>% 
      every( ~ . < 11)
    ```
    
    ```
    ## [1] TRUE
    ```
    
    ```r
    # e.g. below would fail
    # z %>%
    #   every_loop( ~ . < 11)
    ```
    

1.  Create an enhanced `col_sum()` that applies a summary function to every numeric column in a data frame.
    
    
    ```r
    col_summary_enh <- function(x,fun){
      x %>% 
        keep(is.numeric) %>% 
        purrr::map_dbl(fun)
    }
    col_summary_enh(mtcars, median)
    ```
    
    ```
    ##     mpg     cyl    disp      hp    drat      wt    qsec      vs      am 
    ##  19.200   6.000 196.300 123.000   3.695   3.325  17.710   0.000   0.000 
    ##    gear    carb 
    ##   4.000   2.000
    ```

1.  A possible base R equivalent of `col_sum()` is:
    
    ```r
    col_sum3 <- function(df, f) {
      is_num <- sapply(df, is.numeric)
      df_num <- df[, is_num]
    
      sapply(df_num, f)
    }
    ```
    
    But it has a number of bugs as illustrated with the following inputs:
    
    
    ```r
    df <- tibble(
      x = 1:3, 
      y = 3:1,
      z = c("a", "b", "c")
    )
    # OK
    col_sum3(df, mean) 
    # Has problems: don't always return numeric vector
    col_sum3(df[1:2], mean) 
    col_sum3(df[1], mean) 
    col_sum3(df[0], mean)
    ```
    
    What causes the bugs?
    
    * The vector output is not always consistent in it's output type. Also, returns error when inputting an empty list due to indexing issue.
    
## Appendix

### 21.3.5.1

#### Using map


```r
    outputted_csv <- files_example %>% 
      mutate(csv_data = map(file_paths, read_csv))
    
    outputted_csv <- files_example %>% 
      mutate(csv_data = map(file_paths, safely(read_csv)))
```

#### Plot of names

* Below is a plot of the proportion of individuals named the most popular letter in each year. This suggests that the top names by letter do not have as large of a proportion of the population ocmpared to historically.


```r
names_appended %>% 
  ggplot(aes(x = year, y = prop, colour = first_letter))+
  geom_line()
```

<img src="21-iteration_files/figure-html/unnamed-chunk-45-1.png" width="672" />

#### csv other example

This is some code I used to read csvs from a shared drive. I added on the 'file_path_pull' and 'files_example' components to add in information on the file paths and other details that were relevant... you might also add this data into a new column on the output...

```r
files_path_pull <- dir("//companydomain.com/directory/", 
                       pattern = "csv$",
                       full.names = TRUE)

files_example <- tibble(file_paths = files_path_pull[1:2]) %>% 
  extract(file_paths, into = c("path", "name"), regex = "(.*)([0-9]{4}-[0-9]{2}-[0-9]{2})", remove = FALSE)

read_dir <- function(dir){
  #input vector of file paths name and output appended file
  out <- vector("list", length(dir))
  for (i in seq_along(out)){
    out[[i]] <- read_csv(dir[[i]])
  }
  out <-  bind_rows(out)
  out
}

read_dir(files_example$file_paths)
```


### 21.3.5.2 (with purrr)

Slightly less attractive printing

```r
show_mean2 <- function(df) {
  df %>% 
    keep(is.numeric) %>% 
    map_dbl(mean, na.rm = TRUE)
}

show_mean2(flights)
```

```
##           year          month            day       dep_time sched_dep_time 
##    2013.000000       6.548510      15.710787    1349.109947    1344.254840 
##      dep_delay       arr_time sched_arr_time      arr_delay         flight 
##      12.639070    1502.054999    1536.380220       6.895377    1971.923620 
##       air_time       distance           hour         minute 
##     150.686460    1039.912604      13.180247      26.230100
```

Maybe slightly better printing and in df

```r
show_mean3 <- function(df){
  df %>% 
    keep(is.numeric) %>% 
    map_dbl(mean, na.rm = TRUE) %>% 
    as_tibble() %>% 
    mutate(names = row.names(.))
}

show_mean3(flights)
```

```
## # A tibble: 14 x 2
##      value names
##      <dbl> <chr>
##  1 2013    1    
##  2    6.55 2    
##  3   15.7  3    
##  4 1349.   4    
##  5 1344.   5    
##  6   12.6  6    
##  7 1502.   7    
##  8 1536.   8    
##  9    6.90 9    
## 10 1972.   10   
## 11  151.   11   
## 12 1040.   12   
## 13   13.2  13   
## 14   26.2  14
```

Other method is to take advantage of the gather function

```r
flights %>% 
  keep(is.numeric) %>% 
  map(mean, na.rm = TRUE) %>% 
  as_tibble() %>% 
  gather()
```

```
## # A tibble: 14 x 2
##    key              value
##    <chr>            <dbl>
##  1 year           2013   
##  2 month             6.55
##  3 day              15.7 
##  4 dep_time       1349.  
##  5 sched_dep_time 1344.  
##  6 dep_delay        12.6 
##  7 arr_time       1502.  
##  8 sched_arr_time 1536.  
##  9 arr_delay         6.90
## 10 flight         1972.  
## 11 air_time        151.  
## 12 distance       1040.  
## 13 hour             13.2 
## 14 minute           26.2
```


### 21.9 mirroring `keep`

* below is one method for passing multiple, more complex arguments through keep, though you can also use function shortcuts (`~`) in `keep` and `discard`
    
    ```r
    ##how to pass multiple functions through keep?
    #can use map to subset columns by multiple criteria and then subset at end
    flights %>%
      purrr::map(is.na) %>% 
      purrr::map_dbl(sum) %>% 
      purrr::map_lgl(~.>10) %>% 
      flights[.]
    ```
    
    ```
    ## # A tibble: 336,776 x 6
    ##    dep_time dep_delay arr_time arr_delay tailnum air_time
    ##       <int>     <dbl>    <int>     <dbl> <chr>      <dbl>
    ##  1      517         2      830        11 N14228       227
    ##  2      533         4      850        20 N24211       227
    ##  3      542         2      923        33 N619AA       160
    ##  4      544        -1     1004       -18 N804JB       183
    ##  5      554        -6      812       -25 N668DN       116
    ##  6      554        -4      740        12 N39463       150
    ##  7      555        -5      913        19 N516JB       158
    ##  8      557        -3      709       -14 N829AS        53
    ##  9      557        -3      838        -8 N593JB       140
    ## 10      558        -2      753         8 N3ALAA       138
    ## # ... with 336,766 more rows
    ```

### invoke examples

Let's change the example to be with quantile...


```r
invoke(runif, n = 10)
```

```
##  [1] 0.6521087 0.1792651 0.3638088 0.6421232 0.5798101 0.8057947 0.5781534
##  [8] 0.8617429 0.2869783 0.1032626
```

```r
list("01a", "01b") %>%
  invoke(paste, ., sep = "-")
```

```
## [1] "01a-01b"
```

```r
set.seed(123)
invoke_map(list(runif, rnorm), list(list(n = 10), list(n = 5)))
```

```
## [[1]]
##  [1] 0.2875775 0.7883051 0.4089769 0.8830174 0.9404673 0.0455565 0.5281055
##  [8] 0.8924190 0.5514350 0.4566147
## 
## [[2]]
## [1]  1.7150650  0.4609162 -1.2650612 -0.6868529 -0.4456620
```

```r
set.seed(123)
invoke_map(list(runif, rnorm), list(list(n = 10), list(5, 50)))
```

```
## [[1]]
##  [1] 0.2875775 0.7883051 0.4089769 0.8830174 0.9404673 0.0455565 0.5281055
##  [8] 0.8924190 0.5514350 0.4566147
## 
## [[2]]
## [1] 51.71506 50.46092 48.73494 49.31315 49.55434
```


```r
list(m1 = mean, m2 = median) %>% invoke_map(x = rcauchy(100))
```

```
## $m1
## [1] 0.7316016
## 
## $m2
## [1] 0.1690467
```

```r
rcauchy(100)
```

```
##   [1]   -1.99514216    1.57378677    1.44901985    0.82604308    2.30072052
##   [6]   -0.04961749    0.52626840    0.29408692    0.47790231   -1.47138470
##  [11]   -2.54305059   -0.35508248   -1.65511601   -1.08467708  -15.03813728
##  [16]   -1.82118206   -0.62669137   -0.79456204   -0.06347636    5.19179251
##  [21]    1.48851593    3.42095041    0.03289526    0.65171559   -0.53864091
##  [26]    0.88812626    0.93375555    0.24570517    0.97348569   -1.11905466
##  [31]   -0.51964526  128.72537963    2.72138263    0.97793363    0.36391811
##  [36]    2.77745450   -4.34935786    0.81096079    5.70518746    0.81669440
##  [41] -138.41947905    2.02359725   -1.96283674    2.40809060    2.04850398
##  [46]   -9.41347275   -1.06265274    0.83312509    3.55625549    1.10375978
##  [51]   -2.31140048    0.65162145   -0.45665528   -1.02179975   -1.71189590
##  [56]   -2.57239721    2.35617831  -10.63750166   -0.41538322   -3.80770683
##  [61]   -0.55070513    1.49607830   -1.30359005    1.09910916   -3.27457763
##  [66]   16.99304208    1.09921270   -4.86030197   -0.27969649   -0.31842181
##  [71]    1.16466121    1.59209243   -0.04514112   -2.52586678   -0.19951960
##  [76]    9.47599952    3.31841045   -1.82945785    0.51884667   -4.29179059
##  [81]    0.93155898   -0.11880720   -3.03333758  -21.16294537    3.16450655
##  [86]   -0.39503234    2.19801293    1.27457150    0.59413768    0.60064481
##  [91]   17.70703023    1.01880490    0.80764382   -1.63905090    0.15086898
##  [96]   -1.36865319    1.99173761    3.39988162   -0.63043489   -0.26058630
```

Let's store everything in a dataframe...


```r
set.seed(123)
tibble(funs = list(rn = "rnorm", rp = "rpois", ru = "runif"),
       params = list(list(n = 20, mean = 10), list(n = 20, lambda = 3), list(n = 20, min = -1, max = 1))) %>% 
  with(invoke_map_df(funs, params))
```

```
## # A tibble: 20 x 3
##       rn    rp      ru
##    <dbl> <int>   <dbl>
##  1  9.44     1  0.330 
##  2  9.77     2 -0.810 
##  3 11.6      2 -0.232 
##  4 10.1      2 -0.451 
##  5 10.1      1  0.629 
##  6 11.7      1 -0.103 
##  7 10.5      2  0.620 
##  8  8.73     3  0.625 
##  9  9.31     2  0.589 
## 10  9.55     5 -0.120 
## 11 11.2      0  0.509 
## 12 10.4      3  0.258 
## 13 10.4      4  0.420 
## 14 10.1      1 -0.999 
## 15  9.44     3 -0.0494
## 16 11.8      2 -0.560 
## 17 10.5      1 -0.240 
## 18  8.03     4  0.226 
## 19 10.7      5 -0.296 
## 20  9.53     2 -0.778
```


```r
map_df(iris, ~.x*2)
```

```
## Warning in Ops.factor(.x, 2): '*' not meaningful for factors
```

```
## # A tibble: 150 x 5
##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
##           <dbl>       <dbl>        <dbl>       <dbl> <lgl>  
##  1         10.2         7            2.8         0.4 NA     
##  2          9.8         6            2.8         0.4 NA     
##  3          9.4         6.4          2.6         0.4 NA     
##  4          9.2         6.2          3           0.4 NA     
##  5         10           7.2          2.8         0.4 NA     
##  6         10.8         7.8          3.4         0.8 NA     
##  7          9.2         6.8          2.8         0.6 NA     
##  8         10           6.8          3           0.4 NA     
##  9          8.8         5.8          2.8         0.4 NA     
## 10          9.8         6.2          3           0.2 NA     
## # ... with 140 more rows
```

```r
select(iris, -Species) %>% 
  flatten_dbl() %>% 
  mean()
```

```
## [1] 3.4645
```


```r
mean.and.median <- function(x){
  list(mean = mean(x, na.rm = TRUE), 
       median = median(x, na.rm = TRUE))
}
```

Difference between dfr and dfc, taken from here: https://bio304-class.github.io/bio304-fall2017/control-flow-in-R.html 

```r
iris %>%
  select(-Species) %>%
  map_dfr(mean.and.median) %>% 
  bind_cols(tibble(names = names(select(iris, -Species))))
```

```
## # A tibble: 4 x 3
##    mean median names       
##   <dbl>  <dbl> <chr>       
## 1  5.84   5.8  Sepal.Length
## 2  3.06   3    Sepal.Width 
## 3  3.76   4.35 Petal.Length
## 4  1.20   1.3  Petal.Width
```

```r
iris %>%
  select(-Species) %>%
  map_dfr(mean.and.median) %>% 
  bind_cols(tibble(names = names(select(iris, -Species))))
```

```
## # A tibble: 4 x 3
##    mean median names       
##   <dbl>  <dbl> <chr>       
## 1  5.84   5.8  Sepal.Length
## 2  3.06   3    Sepal.Width 
## 3  3.76   4.35 Petal.Length
## 4  1.20   1.3  Petal.Width
```


```r
iris %>%
  select(-Species) %>%
  map_dfc(mean.and.median)
```

```
## # A tibble: 1 x 8
##    mean median mean1 median1 mean2 median2 mean3 median3
##   <dbl>  <dbl> <dbl>   <dbl> <dbl>   <dbl> <dbl>   <dbl>
## 1  5.84    5.8  3.06       3  3.76    4.35  1.20     1.3
```

### indexing nms caution

When creating your empty list, use indexes rather than names if you are creating values, otherwise you are creating new values on the list. E.g. in the example below I the output ends up being length 6 because you have the 3 `NULL` values plus the 3 newly created named positions.

```r
x <- list(a = 1:10, b = 11:18, c = 19:25)
output <- vector("list", length(x))
for (nm in names(x)) {
  output[[nm]] <- x[[nm]] * 3
}
output
```

```
## [[1]]
## NULL
## 
## [[2]]
## NULL
## 
## [[3]]
## NULL
## 
## $a
##  [1]  3  6  9 12 15 18 21 24 27 30
## 
## $b
## [1] 33 36 39 42 45 48 51 54
## 
## $c
## [1] 57 60 63 66 69 72 75
```

### in-class notes

the `map_*` functions are essentially like running a `flatten_*` after running `map`. E.g. the two things below are equivalent


```r
map(flights, typeof) %>% 
  flatten_chr()
```

```
##  [1] "integer"   "integer"   "integer"   "integer"   "integer"  
##  [6] "double"    "integer"   "integer"   "double"    "character"
## [11] "integer"   "character" "character" "character" "double"   
## [16] "double"    "double"    "double"    "double"
```

```r
map_chr(flights, typeof)
```

```
##           year          month            day       dep_time sched_dep_time 
##      "integer"      "integer"      "integer"      "integer"      "integer" 
##      dep_delay       arr_time sched_arr_time      arr_delay        carrier 
##       "double"      "integer"      "integer"       "double"    "character" 
##         flight        tailnum         origin           dest       air_time 
##      "integer"    "character"    "character"    "character"       "double" 
##       distance           hour         minute      time_hour 
##       "double"       "double"       "double"       "double"
```


Calculate the number of unique values for each level

```r
iris %>% 
  map(unique) %>% 
  map_dbl(length)

map_int(iris, ~length(unique(.x)))
```
    
Iterate through different min and max values

```r
min_params <- c(-1, 0, -10)
max_params <- c(11:13)
map2(.x = min_params, .y = max_params, ~runif(n = 10, min = .x, max = .y))
```

```
## [[1]]
##  [1]  1.9234337  7.0166670  4.0117614  8.4583500  0.2343757  4.2187129
##  [7] 10.8194838  9.7166134  9.6376287  1.1006318
## 
## [[2]]
##  [1] 1.568348 7.837223 4.122198 7.881098 3.844479 2.252293 9.387532
##  [8] 1.123140 5.601348 6.138066
## 
## [[3]]
##  [1]  3.7997461 -2.3450586  1.2380998 11.9528980  1.1067551 10.4780551
##  [7] 11.0320783  4.0009046 -0.5541351 -6.6168221
```

When using `pmap` it's often best to keep the parameters in a dataframe

```r
min_df_params <- tibble(n = c(10, 15, 20, 50 ), 
                        min = c(-1, 0, 1, 2), 
                        max = c(0, 1, 2, 3))

pmap(min_df_params, runif)
```

```
## [[1]]
##  [1] -0.06470020 -0.69877110 -0.93927943 -0.05227306 -0.27940373
##  [6] -0.85770570 -0.45071534 -0.04590876 -0.41451665 -0.59548972
## 
## [[2]]
##  [1] 0.6478935 0.3198206 0.3077200 0.2197676 0.3694889 0.9842192 0.1542023
##  [8] 0.0910440 0.1419069 0.6900071 0.6192565 0.8913941 0.6729991 0.7370777
## [15] 0.5211357
## 
## [[3]]
##  [1] 1.659838 1.821805 1.786282 1.979822 1.439432 1.311702 1.409475
##  [8] 1.010467 1.183850 1.842729 1.231162 1.239100 1.076691 1.245724
## [15] 1.732135 1.847453 1.497527 1.387909 1.246449 1.111096
## 
## [[4]]
##  [1] 2.389994 2.571935 2.216893 2.444768 2.217991 2.502300 2.353905
##  [8] 2.649985 2.374714 2.355445 2.533688 2.740334 2.221103 2.412746
## [15] 2.265687 2.629973 2.183828 2.863644 2.746568 2.668285 2.618018
## [22] 2.372238 2.529836 2.874682 2.581750 2.839768 2.312448 2.708290
## [29] 2.265018 2.594343 2.481290 2.265033 2.564590 2.913188 2.901874
## [36] 2.274167 2.321483 2.985641 2.619993 2.937314 2.466533 2.406833
## [43] 2.659230 2.152347 2.572867 2.238726 2.962359 2.601366 2.515030
## [50] 2.402573
```


You can often use `map` a bunch of output that can then be stored in a tibble

```r
tibble(type = map_chr(mtcars, typeof),
       means = map_dbl(mtcars, mean),
       median = map_dbl(mtcars, median),
       names = names(mtcars))
```

```
## # A tibble: 11 x 4
##    type     means median names
##    <chr>    <dbl>  <dbl> <chr>
##  1 double  20.1    19.2  mpg  
##  2 double   6.19    6    cyl  
##  3 double 231.    196.   disp 
##  4 double 147.    123    hp   
##  5 double   3.60    3.70 drat 
##  6 double   3.22    3.32 wt   
##  7 double  17.8    17.7  qsec 
##  8 double   0.438   0    vs   
##  9 double   0.406   0    am   
## 10 double   3.69    4    gear 
## 11 double   2.81    2    carb
```

*Provide the number of unique values for all columns excluding columns with numeric types or date types.*


```r
num_unique <- function(df) {
  df %>% 
  keep(~is_character(.x) | is.factor(.x)) %>% 
  map(~length(unique(.x))) %>% 
  as_tibble() %>% 
  gather() %>% 
  rename(field_name = key, num_unique = value)
}

num_unique(flights)
```

```
## # A tibble: 4 x 2
##   field_name num_unique
##   <chr>           <int>
## 1 carrier            16
## 2 tailnum          4044
## 3 origin              3
## 4 dest              105
```

```r
num_unique(iris)
```

```
## # A tibble: 1 x 2
##   field_name num_unique
##   <chr>           <int>
## 1 Species             3
```

```r
num_unique(mpg)
```

```
## # A tibble: 6 x 2
##   field_name   num_unique
##   <chr>             <int>
## 1 manufacturer         15
## 2 model                38
## 3 trans                10
## 4 drv                   3
## 5 fl                    5
## 6 class                 7
```
