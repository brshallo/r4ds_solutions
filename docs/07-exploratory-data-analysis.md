*Make sure the following packages are installed:*  



# ch. 7: Data exploration

* `cut_width`: specify binsize of each cut (often use with `geom_boxplot`)
* `cut_number`:  specify number of groups to make, allowing for variable binsize (often use with `geom_boxplot`)
* `geom_histogram`: ...
* `geom_freqpoly`: for if you want to have overlapping histograms (so outputs lines instead)
    + can set y as `..density..` to equalize scale of each (similar to how `geom_density` does).
* `geom_boxplot`: ...
* `geom_violin`: Creates double sided histograms for each factor of x
* `geom_bin2d`: scatter plot of x and y values, but use shading to determine count/density in each point
* `geom_hex`: same as `geom_bin2d` but hexagon instead of square shapes are shaded in
* `reorder`: arg1 = variable to reorder, arg2 = variable to reorder it by arg3 = function to reorder by (e.g. median, mean, max...)
* `coord_cartesian`: adjust x,y window w/o filtering out values that are excluded from view
* `xlim`; `ylim`: adjust window and filter out values not within window (same method as `scale_x(/y)_continuous`)
    + these v. `coord_cartesian` is important for geoms like `geom_smooth` that aggregate as they visualize
* `ifelse`: vectorized if else (not to be confused with `if` and `else` functions)
    + `dplyr::if_else` is more strict alternative
* `case_when`: create new variable that relies on complex combination of existing variables
    + often use when you have complex or multiple `ifelse` statements accruing

## 7.3. Variation

### 7.3.4.

*1. Explore the distribution of each of the x, y, and z variables in diamonds. What do you learn? Think about a diamond and how you might decide which dimension is the length, width, and depth.*  

x has some 0s which signifies a data colletion error, y and z have extreme outliers (z more so).


```r
x_hist <- ggplot(diamonds)+
  geom_histogram(aes(x = x), binwidth = 0.1)

y_hist <- ggplot(diamonds)+
  geom_histogram(aes(x = y), binwidth = 0.1)

z_hist <- ggplot(diamonds)+
  geom_histogram(aes(x = z), binwidth = 0.1)


gridExtra::grid.arrange(x_hist, y_hist, z_hist, ncol = 1)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-1-1.png)<!-- -->

Below you see that all three have peaks and troughs on even points. X and y have more similar distributions than z.

```r
ggplot(diamonds)+
  geom_histogram(aes(x=x),binwidth=.1)+
  coord_cartesian(xlim = c(3,11))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-2-1.png)<!-- -->

```r
ggplot(diamonds)+
  geom_histogram(aes(x=y),binwidth=.1)+
  coord_cartesian(xlim = c(3,11))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-2-2.png)<!-- -->

```r
ggplot(diamonds)+
  geom_histogram(aes(x=z),binwidth=.1)+
  coord_cartesian(xlim = c(3,11))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-2-3.png)<!-- -->

I would say that x and y are likely length and width and z depth because diamonds are typically circular on the face so will have the same ratio of length and width and we see this is the case for the x and y dimensions 

```r
diamonds %>% 
  sample_n(1000) %>% 
  ggplot()+
  geom_point(aes(x, y))+
  coord_fixed()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-3-1.png)<!-- -->

Whereas z tends to be more shallow.


```r
diamonds %>% 
  sample_n(1000) %>% 
  ggplot()+
  geom_point(aes(x, z))+
  coord_fixed()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-4-1.png)<!-- -->


*2. Explore the distribution of price. Do you discover anything unusual or surprising? (Hint: Carefully think about the binwidth and make sure you try a wide range of values.)*  


```r
ggplot(diamonds)+
  geom_histogram(aes(x = price), binwidth=10)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-5-1.png)<!-- -->

Price is right skewed.

Also notice that from 1400 to 1600 there are diamonds.


```r
ggplot(diamonds)+
  geom_histogram(aes(x = price), binwidth = 5)+coord_cartesian(xlim = c(1400,1600))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-6-1.png)<!-- -->


*3. How many diamonds are 0.99 carat? How many are 1 carat? What do you think is the cause of the difference?*  


```r
filter(diamonds, carat == 0.99) %>% 
  count()
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1    23
```

```r
filter(diamonds, carat == 1) %>% 
  count()
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1  1558
```

For visual scale.

```r
ggplot(diamonds)+
  geom_histogram(aes(x=carat), binwidth=.01)+
  coord_cartesian(xlim=c(.99,1))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-8-1.png)<!-- -->

The difference may be caused by jewlers rounding-up because people want to buy '1' carat diamonds not 0.99 carat diamonds. It could also be that some listings are simpoly only in integers^[not necessarily rounding one way or the other.].

*4.Compare and contrast `coord_cartesian()` vs `xlim()` or `ylim()` when zooming in on a histogram. What happens if you leave binwidth unset? What happens if you try and zoom so only half a bar shows?*  

`coord_cartesian` does not change data ust window view where as `xlim` and `ylim` will get rid of data outside of domain^[This is especially important when building things like boxplots whose graphs depend on all points in the graph.].


## 7.4. Missing values

### 7.4.1.

*1. What happens to missing values in a histogram? What happens to missing values in a bar chart? Why is there a difference?*  
With numeric data they both filter out NAs, though for categorical / character variables the `barplot` will create a seperate olumn with the category. This is because `NA` can just be thought of as another category though it is difficulty to place it within a distribution of values.

Treats these the same.

```r
mutate(diamonds, carattest=ifelse(carat<1.5 & carat>.7, NA, carat)) %>% 
  ggplot() +
  geom_histogram(aes(x=carattest))
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 20543 rows containing non-finite values (stat_bin).
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-9-1.png)<!-- -->

```r
mutate(diamonds, carattest=ifelse(carat<1.5 & carat>.7, NA, color)) %>% 
  ggplot() +
  geom_bar(aes(x=carattest))
```

```
## Warning: Removed 20543 rows containing non-finite values (stat_count).
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-9-2.png)<!-- -->

For character than it creates a new bar for `NA`s

```r
mutate(diamonds, carattest=ifelse(carat<1.5 & carat>.7, NA, color)) %>% 
  ggplot() +
  geom_bar(aes(x = as.character(carattest)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-10-1.png)<!-- -->


*2. What does na.rm = TRUE do in `mean()` and `sum()`?*  
  
Filters it out of the vector of values.  

## 7.5. Covariation

### 7.5.1.1.

*1. Use what you've learned to improve the visualisation of the departure times of cancelled vs. non-cancelled flights.*  
Looks like while non-cancelled flights happen at similar frequency in mornings and evenings, cancelled flights happen at a greater frequency in the evenings.


```r
nycflights13::flights %>% 
  mutate(
    cancelled = is.na(dep_time),
    sched_hour = sched_dep_time %/% 100,
    sched_min = sched_dep_time %% 100,
    sched_dep_time = sched_hour + sched_min / 60
  ) %>% 
  ggplot(mapping = aes(x=sched_dep_time, y=..density..)) + 
  geom_freqpoly(mapping = aes(colour = cancelled), binwidth = .25)+
  xlim(c(5,25))
```

```
## Warning: Removed 1 rows containing non-finite values (stat_bin).
```

```
## Warning: Removed 4 rows containing missing values (geom_path).
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-11-1.png)<!-- -->

Let's look at the same plot but smooth the distributions to make the pattern easier to see.

```r
nycflights13::flights %>% 
  mutate(
    cancelled = is.na(dep_time),
    sched_hour = sched_dep_time %/% 100,
    sched_min = sched_dep_time %% 100,
    sched_dep_time = sched_hour + sched_min / 60
  ) %>% 
  ggplot(mapping = aes(x=sched_dep_time)) + 
  geom_density(mapping = aes(fill = cancelled), alpha = 0.30)+
  xlim(c(5,25))
```

```
## Warning: Removed 1 rows containing non-finite values (stat_density).
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-12-1.png)<!-- -->


*2.What variable in the diamonds dataset is most important for predicting the price of a diamond? How is that variable correlated with cut? Why does the combination of those two relationships lead to lower quality diamonds being more expensive?*  
  
carat is the most important for predicting price.


```r
cor(diamonds$price, select(diamonds, carat, depth, table, x, y, z))
```

```
##          carat      depth     table         x         y         z
## [1,] 0.9215913 -0.0106474 0.1271339 0.8844352 0.8654209 0.8612494
```

fair `cut`s seem to associate with a higher `carat` thus while lower quality diamonds may be selling for more that is being driven by the `carat` of the diamond (the most important factor in `price`) and the quality simply cannot offset this.


```r
ggplot(data=diamonds, aes(x=cut, y=carat))+
  geom_boxplot()+
  coord_flip()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-14-1.png)<!-- -->


*3.Install the ggstance package, and create a horizontal boxplot. How does this compare to using coord_flip()?*  


```r
ggplot(diamonds)+
  ggstance::geom_boxploth(aes(x = carat, y = cut))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-15-1.png)<!-- -->

Looks like it does the same thing.

*4. One problem with boxplots is that they were developed in an era of much smaller datasets and tend to display a prohibitively large number of "outlying values". One approach to remedy this problem is the letter value plot. Install the lvplot package, and try using geom_lv() to display the distribution of price vs cut. What do you learn? How do you interpret the plots?*  

I found this helpful: https://stats.stackexchange.com/questions/301159/understanding-and-interpreting-letter-value-boxplots 

This produces a 'letter-value' boxplot which means that in the first box you have the middle ~1/2 of data, then in the adoining boxes the next ~1/4, so within the middle 3 boxes you have the middle ~3/4 of data, next two boxes is ~7/8ths, then ~15/16th etc.


```r
set.seed(1234)
a <- diamonds %>% 
  ggplot()+
  lvplot::geom_lv(aes(x = cut, y = price))

set.seed(1234)
b <- diamonds %>% 
  ggplot()+
  geom_boxplot(aes(x = cut, y = price))
```

Perhaps a helpful way to understand this is to see what it looks like at different specified 'k' values (which)

You can see the letters when you add `fill = ..LV..` to the aesthetic.


```r
diamonds %>% 
  ggplot()+
  lvplot::geom_lv(aes(x = cut, y = price, alpha = ..LV..), fill = "blue")+
  scale_alpha_discrete(range = c(0.7, 0))
```

```
## Warning: Using alpha for a discrete variable is not advised.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-17-1.png)<!-- -->

```r
diamonds %>% 
  ggplot()+
  lvplot::geom_lv(aes(x = cut, y = price, fill = ..LV..))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-17-2.png)<!-- -->

Letters represent 'median', 'fourths', 'eights'...


*5. Compare and contrast geom_violin() with a facetted geom_histogram(), or a coloured geom_freqpoly(). What are the pros and cons of each method?*  


```r
ggplot(diamonds,aes(x = cut, y = carat))+
  geom_violin()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-18-1.png)<!-- -->

```r
ggplot(diamonds,aes(colour = cut, x = carat, y = ..density..))+
  geom_freqpoly()
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-18-2.png)<!-- -->

```r
ggplot(diamonds, aes(x = carat, y = ..density..))+
  geom_histogram()+
  facet_wrap(~cut)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-18-3.png)<!-- -->

I like how `geom_freqpoly` has points directly overlaying but it can also be tough to read some, and the lines can overlap and be tough to tell apart, you also have to specify `density` for this and `geom_histogram` whereas for `geom_violin` it is the default. The tails in `geom_violin` can be easy to read but they also pull these for each of the of the values whereas by faceting `geomo_histogram` and setting `scales = "free"` you can have independent scales. I think the biggest advantage of the histogram is that it is the most familiar so people will know what you're looking at.


*6. If you have a small dataset, it's sometimes useful to use geom_jitter() to see the relationship between a continuous and categorical variable. The ggbeeswarm package provides a number of methods similar to geom_jitter(). List them and briefly describe what each one does.*  

(Come back to)


```r
ggplot(mpg, aes(x = displ, y = cty, color = drv))+
  geom_point()    
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-19-1.png)<!-- -->

```r
ggplot(mpg, aes(x = displ, y = cty, color = drv))+
  geom_jitter()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-19-2.png)<!-- -->

```r
ggplot(mpg, aes(x = displ, y = cty, color = drv))+
  geom_beeswarm()
```

```
## Warning in f(...): The default behavior of beeswarm has changed in version
## 0.6.0. In versions <0.6.0, this plot would have been dodged on the y-
## axis. In versions >=0.6.0, grouponX=FALSE must be explicitly set to group
## on y-axis. Please set grouponX=TRUE/FALSE to avoid this warning and ensure
## proper axis choice.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-19-3.png)<!-- -->

```r
ggplot(mpg, aes(x = displ, y = cty, color = drv))+
  geom_quasirandom()
```

```
## Warning in f(...): The default behavior of beeswarm has changed in version
## 0.6.0. In versions <0.6.0, this plot would have been dodged on the y-
## axis. In versions >=0.6.0, grouponX=FALSE must be explicitly set to group
## on y-axis. Please set grouponX=TRUE/FALSE to avoid this warning and ensure
## proper axis choice.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-19-4.png)<!-- -->

`geom_jitter` is similar to `geom_point` but it provides random noise to the points. You can control these with the `width` and `height` arguments. This is valuable as it allows you to better see points that may overlap one another. `geom_beeswarm` adds variation in a uniform pattern by default across only the x-axis. `geom-quasirandom` also defaults to distributing the points across the x-axis however it produces quasi-random variation, <span style="color:green">'quasi'</span> because it looks as though points follow some interrelationship^[Would need to read documentation for details.] and if you run the plot multiple times you will get the exact same plot whereas for `geom_jitter` you will get a slightly different plot each time. To see the differences between `geom_beeswarm` and geom_quasirandom` it's helpful to look at the plots above, but holding the y value constant at 1.


```r
plot_orig <- ggplot(mpg, aes(x = displ, y = cty, color = drv))+
  geom_point()

plot_bees <- ggplot(mpg, aes(x = 1, y = cty, color = drv))+
  geom_beeswarm()

plot_quasi <- ggplot(mpg, aes(x = 1, y = cty, color = drv))+
  geom_quasirandom()

gridExtra::grid.arrange(plot_orig, plot_bees, plot_quasi, ncol = 1)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-20-1.png)<!-- -->

### 7.5.2.1.

*1. How could you rescale the count dataset above to more clearly show the distribution of cut within colour, or colour within cut?*  
  
Proportion cut in color:  
(change `group_by` to `group_by(cut, color)` to set-up the converse)

```r
cut_in_color_graph <- diamonds %>% 
  group_by(color, cut) %>% 
  summarise(n = n()) %>% 
  mutate(proportion_cut_in_color = n/sum(n)) %>%
  ggplot(aes(x = color, y = cut))+
  geom_tile(aes(fill = proportion_cut_in_color))+
  labs(fill = "proportion\ncut in color")

cut_in_color_graph
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-21-1.png)<!-- -->

This makes it clear that `ideal` cuts dominate the proportions of multiple colors, not ust G.  I though am only so-so about this visualization with `geom_tile`... curious if someone had a better way of visualizing...?


```r
library(seriation)
library(d3heatmap)
library(heatmaply)
```

Did anyone explore these?  
  
*2. Use `geom_tile()` together with dplyr to explore how average flight delays vary by destination and month of year. What makes the plot difficult to read? How could you improve it?*


I improved the original graph by adding in a filter so that only destinations that received over 10000 flights were included

```r
flights %>% 
  group_by(dest, month) %>% 
  summarise(delay_mean = mean(dep_delay, na.rm=TRUE), 
            n = n()) %>% 
  mutate(sum_n = sum(n)) %>% 
  select(dest, month, delay_mean, n, sum_n) %>% 
  as.data.frame() %>% 
  filter(dest == "ABQ") %>% 
  #the sum on n will be at the dest level here
  filter(sum_n > 30) %>% 
  ggplot(aes(x = as.factor(month), y = dest, fill = delay_mean))+
  geom_tile()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-23-1.png)<!-- -->

Another way to improve it may be to group the destinations into regions. This also will prevent you from filtering out data. We aren't given region information, but we do have lat and long points in the `airports` dataset. See appendix for ntoes.  
  

*3. Why is it slightly better to use aes(x = color, y = cut) rather than aes(x = cut, y = color) in the example above?*  

If you're comparing the proportion of cut in color and want to be looking at how the specific cut proportion is changing, it may easier to view this while looking left to right vs. down to up. Compare the two plots below.


```r
cut_in_color_graph
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-24-1.png)<!-- -->

```r
cut_in_color_graph+
  coord_flip()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-24-2.png)<!-- -->


## 7.5.3 Two continuous variables

Two-d histograms

```r
smaller <- diamonds %>% 
  filter(carat < 3)

ggplot(data = smaller) +
  geom_hex(mapping = aes(x = carat, y = price))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-25-1.png)<!-- -->

```r
#can change bin number
ggplot(data = smaller) +
  geom_bin2d(mapping = aes(x = carat, y = price), bins = c(30, 30))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-25-2.png)<!-- -->

```r
#or binwidth
ggplot(data = smaller) +
  geom_bin2d(mapping = aes(x = carat, y = price), binwidth = c(.1, 1000))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-25-3.png)<!-- -->

Binned boxplots, violins, and lvs


```r
#split by width
ggplot(smaller, aes(x = carat, y = price))+
  geom_boxplot(aes(group = cut_width(carat, 0.1)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-26-1.png)<!-- -->

```r
#split to get approximately same number in each box with cut_number()
ggplot(smaller, aes(x = carat, y = price))+
  geom_boxplot(aes(group = cut_number(carat, 20)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-26-2.png)<!-- -->

These methods don't seem to work quite as well with violin plots or letter value plots

```r
##violin
ggplot(smaller, aes(x = carat, y = price))+
  geom_violin(aes(group = cut_width(carat, 0.1)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-27-1.png)<!-- -->

```r
ggplot(smaller, aes(x = carat, y = price))+
  geom_violin(aes(group = cut_number(carat, 20)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-27-2.png)<!-- -->

```r
##letter value
ggplot(smaller, aes(x = carat, y = price))+
  lvplot::geom_lv(aes(group = cut_width(carat, 0.1)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-27-3.png)<!-- -->

```r
ggplot(smaller, aes(x = carat, y = price))+
  lvplot::geom_lv(aes(group = cut_number(carat, 10)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-27-4.png)<!-- -->


But that may be because there is not enough information in each bin, they may be more effective with smaller bin sizes... Interested if anyone found more effective methods for using things other than boxplots with this binning style?

```r
ggplot(smaller, aes(x = carat, y = price))+
  geom_violin(aes(group = cut_width(carat, .25)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-28-1.png)<!-- -->


### 7.5.3.1.

*1. Instead of summarising the conditional distribution with a boxplot, you could use a frequency polygon. What do you need to consider when using cut_width() vs cut_number()? How does that impact a visualisation of the 2d distribution of carat and price?*  
  
You should keep in mind how many lines you are going to create, they may overlap each other and look busy if you're not careful.  
  

For the visualization below i wrapped it in the funciton `ggplotly`. This funciton wraps your ggplot in html so that you can do things like hover over the points

```r
ggplot(smaller, aes(x=price))+
  geom_freqpoly(aes(colour = cut_number(carat, 10)))
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-29-1.png)<!-- -->

However, I set the code chunks such that it will only execute for html, not markdown docs.

```r
p <- ggplot(smaller, aes(x=price))+
  geom_freqpoly(aes(colour = cut_width(carat, 0.25)))


plotly::ggplotly(p)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-30-1.png)<!-- -->


  
*2. Visualise the distribution of carat, partitioned by price.*  
  

```r
ggplot(diamonds, aes(x = price, y = carat))+
  geom_violin(aes(group = cut_width(price, 2500)))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-31-1.png)<!-- -->

  
*3. How does the price distribution of very large diamonds compare to small diamonds. Is it as you expect, or does it surprise you?*  
  

```r
diamonds %>% 
  mutate(percent_rank = percent_rank(carat),
         small = percent_rank < 0.025,
         large = percent_rank > 0.975) %>% 
  filter(small | large) %>% 
  ggplot(aes(large, price)) +
  geom_violin()+
  facet_wrap(~large)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-32-1.png)<!-- -->
  
Small diamonds have a left-skewed distribution, whereas large diamonds have a right skewed distribution.  
  
*4. Combine two of the techniques you've learned to visualise the combined distribution of cut, carat, and price.*  



```r
ggplot(diamonds, aes(x = carat, y = price))+
  geom_jitter(aes(colour = cut), alpha = 0.2)+
  geom_smooth(aes(colour = cut))
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-33-1.png)<!-- -->

```r
ggplot(diamonds, aes(x = carat, y = price))+
  geom_boxplot(aes(group = cut_width(carat, 0.5), colour = cut))+
  facet_grid(. ~ cut)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-33-2.png)<!-- -->

```r
##I think this gives a better visualization, but is a little more complicated to produce, I also have the github version of ggplot and do not know whether the `preserve` arg is available in current CRAN installation.
diamonds %>% 
  mutate(carat = cut(carat, 5)) %>% 
  ggplot(aes(x = carat, y = price))+
  geom_boxplot(aes(group = interaction(cut_width(carat, 0.5), cut), fill = cut), position = position_dodge(preserve = "single"))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-33-3.png)<!-- -->
  
  
*5.Two dimensional plots reveal outliers that are not visible in one dimensional plots. For example, some points in the plot below have an unusual combination of x and y values, which makes the points outliers even though their x and y values appear normal when examined separately.*  
  

```r
ggplot(data = diamonds) +
  geom_point(mapping = aes(x = x, y = y)) +
  coord_cartesian(xlim = c(4, 11), ylim = c(4, 11))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-34-1.png)<!-- -->
  
*Why is a scatterplot a better display than a binned plot for this case?*  
Binned plots give less precise value estimates at each point (constrained by the granularity of the binning) so outliers do not show-up as clearly. They also show less precise relationships between the data. The level of variability (at least with boxplots) can also be tougher to intuit. For example, let's look at the plot below as a binned boxplot.


```r
ggplot(data = diamonds) +
  geom_boxplot(mapping = aes(x = cut_width(x, 1), y = y)) +
  coord_cartesian(xlim = c(4, 11), ylim = c(4, 11))
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-35-1.png)<!-- -->
  
  
# Appendix

## 7.5.2.1.2.

Plot below shows four regions I'll split the country into. Seems like for a few destinations the lat and long points were likely misentered (probably backwards).


```r
all_states <- map_data("state")
p <- geom_polygon( data=all_states, aes(x=long, y=lat, group = group, label = NULL), colour="white", fill="grey10" )

dest_regions <- nycflights13::airports %>% 
  mutate(lat_cut = cut(percent_rank(lat), 2, labels = c("S", "N")),
         lon_cut = cut(percent_rank(lon), 2, labels = c("W", "E")),
         quadrant = paste0(lat_cut, lon_cut)) 

point_plot <- dest_regions %>%
  ggplot(aes(lon, lat, colour = quadrant))+
  p+
  geom_point()

point_plot
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-36-1.png)<!-- -->


Now let's join our region information with our flight data and do our calculations grouping by `quadrant` rather than `dest`. Note that those `quadrant`s with `NA` (did not join with `flights`) looked to be Pueorto Rico or other non-state locations.

```r
flights %>% 
  left_join(dest_regions, by = c("dest" = "faa")) %>% 
  group_by(quadrant, month) %>% 
  summarise(delay_mean = mean(dep_delay, na.rm=TRUE), 
            n = n()) %>% 
  mutate(sum_n = sum(n)) %>% 
  #the sum on n will be at the dest level here
  # filter(sum_n > 10000) %>% 
  ggplot(aes(x = as.factor(month), y = quadrant, fill = delay_mean))+
  geom_tile()
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-37-1.png)<!-- -->

I think changing the color from light-->dark to instead blue-->red may make for a more effective visualization as well.  
  
## 7.5.3.1.4.

To get the `fill` value to vary need to iterate through and make each graph seperate, can't ust use facet.

```r
diamonds_nest <- diamonds %>%
  group_by(cut) %>% 
  tidyr::nest()


plot_free <- function(df, name){
ggplot(df)+
  geom_bin2d(aes(carat, price))+
  ggtitle(name)
}

gridExtra::grid.arrange(grobs = mutate(diamonds_nest, out = purrr::map2(data, cut, plot_free))$out)
```

![](07-exploratory-data-analysis_files/figure-epub3/unnamed-chunk-38-1.png)<!-- -->



```r
diamonds %>% 
  mutate(cut = forcats::as_factor(as.character(cut), levels = c("Fair", "Good", "Very Good", "Premium", "Ideal"))) %>%
  # with(contrasts(cut))
  lm(log(price) ~ log(carat) + cut, data = .) %>% 
  summary()
```

```
## 
## Call:
## lm(formula = log(price) ~ log(carat) + cut, data = .)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.52247 -0.16484 -0.00587  0.16087  1.38115 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   8.517337   0.001996 4267.70   <2e-16 ***
## log(carat)    1.695771   0.001910  887.68   <2e-16 ***
## cutPremium   -0.078994   0.002810  -28.11   <2e-16 ***
## cutGood      -0.153967   0.004046  -38.06   <2e-16 ***
## cutVery Good -0.076458   0.002904  -26.32   <2e-16 ***
## cutFair      -0.317212   0.006632  -47.83   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2545 on 53934 degrees of freedom
## Multiple R-squared:  0.9371,	Adjusted R-squared:  0.9371 
## F-statistic: 1.607e+05 on 5 and 53934 DF,  p-value: < 2.2e-16
```

```r
contrasts(diamonds$cut)
```

```
##              .L         .Q            .C         ^4
## [1,] -0.6324555  0.5345225 -3.162278e-01  0.1195229
## [2,] -0.3162278 -0.2672612  6.324555e-01 -0.4780914
## [3,]  0.0000000 -0.5345225 -4.095972e-16  0.7171372
## [4,]  0.3162278 -0.2672612 -6.324555e-01 -0.4780914
## [5,]  0.6324555  0.5345225  3.162278e-01  0.1195229
```

```r
count(diamonds, cut)
```

```
## # A tibble: 5 x 2
##   cut           n
##   <ord>     <int>
## 1 Fair       1610
## 2 Good       4906
## 3 Very Good 12082
## 4 Premium   13791
## 5 Ideal     21551
```

