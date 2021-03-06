---
title: "Table lookup"
output: github_document
---



I try to use [dplyr joins](bit001_dplyr-cheatsheet.html) for most tasks that combine data from two tibbles. But sometimes you just need good old "table lookup". Party like it's Microsoft Excel `LOOKUP()` time!

### Load gapminder and the tidyverse


```r
library(gapminder)
library(tidyverse)
## ── Attaching packages ─────────────────────────────────────────────── tidyverse 1.2.1 ──
## ✔ ggplot2 3.0.0           ✔ purrr   0.2.5      
## ✔ tibble  1.4.99.9005     ✔ dplyr   0.7.7      
## ✔ tidyr   0.8.1           ✔ stringr 1.3.1      
## ✔ readr   1.1.1           ✔ forcats 0.3.0
## ── Conflicts ────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

### Create mini Gapminder

Work with a tiny subset of Gapminder, `mini_gap`.


```r
mini_gap <- gapminder %>% 
  filter(country %in% c("Belgium", "Canada", "United States", "Mexico"),
         year > 2000) %>% 
  select(-pop, -gdpPercap) %>% 
  droplevels()
mini_gap
## # A tibble: 8 x 4
##   country       continent  year lifeExp
##   <fct>         <fct>     <int>   <dbl>
## 1 Belgium       Europe     2002    78.3
## 2 Belgium       Europe     2007    79.4
## 3 Canada        Americas   2002    79.8
## 4 Canada        Americas   2007    80.7
## 5 Mexico        Americas   2002    74.9
## 6 Mexico        Americas   2007    76.2
## 7 United States Americas   2002    77.3
## 8 United States Americas   2007    78.2
```

### Dorky national food example.

Make a lookup table of national foods. Or at least the stereotype. Yes I have intentionally kept Mexico in mini-Gapminder and neglected to put Mexico here.


```r
food <- tribble(
        ~ country,    ~ food,
        "Belgium",  "waffle",
         "Canada", "poutine",
  "United States", "Twinkie"
)
food
## # A tibble: 3 x 2
##   country       food   
##   <chr>         <chr>  
## 1 Belgium       waffle 
## 2 Canada        poutine
## 3 United States Twinkie
```

### Lookup national food

`match(x, table)` reports where the values in the key `x` appear in the lookup variable `table`. It returns positive integers for use as indices. It assumes `x` and `table` are free-range vectors, i.e. there's no implicit data frame on the radar here.

Gapminder's `country` plays the role of the key `x`. It is replicated, i.e. non-unique, in `mini_gap`, but not in `food`, i.e. no country appears more than once `food$country`. FYI `match()` actually allows for multiple matches by only consulting the first.


```r
match(x = mini_gap$country, table = food$country)
## [1]  1  1  2  2 NA NA  3  3
```

In table lookup, there is always a value variable `y` that you plan to index with the `match(x, table)` result.  It often lives together with `table` in a data frame; they should certainly be the same length and synced up with respect to row order.

But first ...

I get `x` and `table` backwards some non-negligible percentage of the time. So I store the match indices and index the data frame where `table` lives with it. Add `x` as a column and eyeball-o-metrically assess that all is well.


```r
(indices <- match(x = mini_gap$country, table = food$country))
## [1]  1  1  2  2 NA NA  3  3
add_column(food[indices, ], x = mini_gap$country)
## # A tibble: 8 x 3
##   country       food    x            
##   <chr>         <chr>   <fct>        
## 1 Belgium       waffle  Belgium      
## 2 Belgium       waffle  Belgium      
## 3 Canada        poutine Canada       
## 4 Canada        poutine Canada       
## 5 <NA>          <NA>    Mexico       
## 6 <NA>          <NA>    Mexico       
## 7 United States Twinkie United States
## 8 United States Twinkie United States
```

Once all looks good, do the actual table lookup and, possibly, add the new info to your main table.


```r
mini_gap %>% 
  mutate(food = food$food[indices])
## # A tibble: 8 x 5
##   country       continent  year lifeExp food   
##   <fct>         <fct>     <int>   <dbl> <chr>  
## 1 Belgium       Europe     2002    78.3 waffle 
## 2 Belgium       Europe     2007    79.4 waffle 
## 3 Canada        Americas   2002    79.8 poutine
## 4 Canada        Americas   2007    80.7 poutine
## 5 Mexico        Americas   2002    74.9 <NA>   
## 6 Mexico        Americas   2007    76.2 <NA>   
## 7 United States Americas   2002    77.3 Twinkie
## 8 United States Americas   2007    78.2 Twinkie
```

Of course, if this was really our exact task, we could have used a join!


```r
mini_gap %>% 
  left_join(food)
## Joining, by = "country"
## Warning: Column `country` joining factor and character vector, coercing
## into character vector
## # A tibble: 8 x 5
##   country       continent  year lifeExp food   
##   <chr>         <fct>     <int>   <dbl> <chr>  
## 1 Belgium       Europe     2002    78.3 waffle 
## 2 Belgium       Europe     2007    79.4 waffle 
## 3 Canada        Americas   2002    79.8 poutine
## 4 Canada        Americas   2007    80.7 poutine
## 5 Mexico        Americas   2002    74.9 <NA>   
## 6 Mexico        Americas   2007    76.2 <NA>   
## 7 United States Americas   2002    77.3 Twinkie
## 8 United States Americas   2007    78.2 Twinkie
```

But sometimes you have a substantive reason (or psychological hangup) that makes you prefer the table look up interface.

### World's laziest table lookup

While I'm here, let's demo another standard R trick that's based on indexing by name.

Imagine the table you want to consult isn't even a tibble but is, instead, a named character vector.


```r
(food_vec <- setNames(food$food, food$country))
##       Belgium        Canada United States 
##      "waffle"     "poutine"     "Twinkie"
```

Another way to get the national foods for mini-Gapminder is to simply index `food_vec` with `mini_gap$country`.


```r
mini_gap %>% 
  mutate(food = food_vec[country])
## # A tibble: 8 x 5
##   country       continent  year lifeExp food   
##   <fct>         <fct>     <int>   <dbl> <chr>  
## 1 Belgium       Europe     2002    78.3 waffle 
## 2 Belgium       Europe     2007    79.4 waffle 
## 3 Canada        Americas   2002    79.8 poutine
## 4 Canada        Americas   2007    80.7 poutine
## 5 Mexico        Americas   2002    74.9 Twinkie
## 6 Mexico        Americas   2007    76.2 Twinkie
## 7 United States Americas   2002    77.3 <NA>   
## 8 United States Americas   2007    78.2 <NA>
```

HOLD ON. STOP. Twinkies aren't the national food of Mexico!?! What went wrong?

Remember `mini_gap$country` is a factor. So when we use it in an indexing context, it's integer nature is expressed. It is pure luck that we get the right foods for Belgium and Canada. Luckily the Mexico - United States situation tipped us off. Here's what we are really indexing `food_vec` by above:


```r
unclass(mini_gap$country)
## [1] 1 1 2 2 3 3 4 4
## attr(,"levels")
## [1] "Belgium"       "Canada"        "Mexico"        "United States"
```

To get our desired result, we need to explicitly coerce `mini_gap$country` to character.


```r
mini_gap %>% 
  mutate(food = food_vec[as.character(country)])
## # A tibble: 8 x 5
##   country       continent  year lifeExp food   
##   <fct>         <fct>     <int>   <dbl> <chr>  
## 1 Belgium       Europe     2002    78.3 waffle 
## 2 Belgium       Europe     2007    79.4 waffle 
## 3 Canada        Americas   2002    79.8 poutine
## 4 Canada        Americas   2007    80.7 poutine
## 5 Mexico        Americas   2002    74.9 <NA>   
## 6 Mexico        Americas   2007    76.2 <NA>   
## 7 United States Americas   2002    77.3 Twinkie
## 8 United States Americas   2007    78.2 Twinkie
```

When your key variable is character (and not a factor), you can skip this step.
