--- 
title: "Another set of solutions and notes to 'R for Data Science'"
author: "Bryan Shalloway"
date: "2019-05-22"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
description: "My notes and solutions to 'R for Data Science' by Garrett Grolemund and Hadley Wickham"
---

# Purpose of this book

This book contains my solutions and notes to Garrett Grolemund and Hadley Wickham's book, *R for Data Science* (R4DS). First and foremost my book is set-up as a resource and refresher for myself. 

If you are looking for a place to check your solutions for R4DS I would recommend using @Jrnold's solutions ___ as a source. @Jrnold has done a great job of getting community feedback and well vetted solutions. Though feel free to use this book as another point of reference (if you're curious to see a potentially alternative solution to a problem).

I am not actively seeking feedback, but if you find a major issue, or want to comment on one of my notes, feel free to open a github issue @brshallo.

# Features of this book

Each chapter may contain the following:
* list of functions and notes from the chapter^[When functions show up in multiple locations I typically only note them the first time they appear.]. 
* Solutions to exercises
* Appendix containing notes, or alternative solutions to problems

# Origin of this book

'R for Data Science' is my go to resource I recommend for people interested in getting started in "data science", R programming, or the "tidyverse". I've gone through the book front to back three times and regularly come back to it as a reference.

I first read and completed the exercises to R4DS in early 2017 on the tail-end of completing a Master's in Analytics program. My second time going through R4DS came in early 2018 when myself and a colleague, @StephenKimel, organized an internal study group for our colleagues.

You can see my part of an internal talk I gave plugging "tidy" data science workflows (and implicitly the R4DS study group) here: __

The study group provided me a chance to clean-up my solutions, organize my notes, and pursue a vague ambition to publish my solutions and notes online into a book. After stumbling into @Jrnold online R4DS solutions this ambition no longer felt useful... but, roughly a year later, I got around to publishing it -- and went through R4DS a third time.

# Plug for R4DS

Although reading a book three times in three years may seem excessive, R4DS is excellent on repeat engagements. After the first read, I felt comfortable doing most common types of data manipulation. On the second, functional programming methods and elegant iteration methods like those demonstrated in [Chapter 25: many models](https://r4ds.had.co.nz/many-models.html) felt easy and intuitive. 

See a lightning talk I gave on "managing many models" ___

On the third R4DS reading, various functions or options I'd forgotten, or not inuited the utility of suddenly popped-out to me as useful things I needed to add to my tool belt to replace less effective methods I'd been using (e.g. `tidyr::enframe()`, `cache.extra` chunk option, ...).

## Plug for R4DS

R4DS is very much a nuts and bolts book. It focuses more on data "tidying" and visualization than on some of the sexy topics 

My Master's program had provided me with a broad overview and strong foundation of knowledge on a wide range of analytic methods. However, having bounced between python, R, SQL, and SAS on different projects, the nuts and bolts of my coding skills had yet to mature. 

Job interviews were coming-up and I was nervous my lack of fluency in coding would come through either in code submissions or reviews.

## Acknowledgements

Of course Hadley Wickham and Garrett Grolemund for their amazing book, but also the entire team at RStudio producing superb materials. Stephen Kimel and my colleagues at NetApp for going through our study group together. The (@R4DS community)[] and @rstats communities for creating such welcoming and inspiring environments for learning R programming.
