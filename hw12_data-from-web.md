# Shiny - Homework
Andrew MacDonald and Jenny Bryan  
`r Sys.Date()`  

## Big picture

We learned three ways of collecting data from the internet:
  * Accessing data using ROpenSci packages
  * Running (basic) API queries
  * Web scraping

For the homework, we want you to either combine two existing datasets in a novel (and reproducible!) way, or create a new dataset by web scraping.

## Please just tell me what to do! 

### Combine `gapminder` and data from `geonames`

Use data from both to make plots which answer either:
  * What is the relationship between per-capita GDP and the proportion of the population which lives in urban centers?
  * Consider the following graph (a modification of Jenny's [gapminder demo](https://github.com/jennybc/gapminder)):
    

```r
library("ggplot2")
library("gapminder")

ggplot(subset(gapminder, continent != "Oceania"),
       aes(x = year, y = pop, group = country, color = country)) +
  geom_line(lwd = 1, show_guide = FALSE) + facet_wrap(~ continent) +
  scale_color_manual(values = country_colors) +
  theme_bw() + theme(strip.text = element_text(size = rel(1.1))) + scale_y_log10()
```

![plot of chunk unnamed-chunk-1](hw12_data-from-web_files/figure-html/unnamed-chunk-1.png) 

Replace population with *density*. To do this, look up the country codes in `geonames()` and obtain the area of each country.

## Combine two datasets

* `rplos` and `rebird` -- how many articles are published on a bird species? 
* `rplos` and `geonames` -- Choose a random set of countries. How many papers have been published by people from that country? In that country? how does that relate to GDP (will require expert-level regex skills)
* `rebird` and `geonames` -- Do countries with more bird species also have more languages?

## I want to aim higher!

* Go look through the RopenSci [packages list](http://ropensci.org/packages/) and/or the Ropensci [Web Services in R](https://github.com/ropensci/webservices), find some existing resources, and remix those instead.

## even HIGHER

* Consult [programmable web](), discover an API which is NOT wrapped by Ropensci (yet!) and write a function to query that.

## I am a leaf on the wind. See how I SOAR

* Find an interesting website which is a) not on Ropensci __nor__ b) has a published API. Scrape it into a lovely dataset for us, and publish this as a data package a la Gapminder 

### Due date

Your Shiny app is due Friday 05 December 2014.

## Rubric

Your peer reviewer will look at your data, and hopefully like it.

These datasets should be reproducible.  

Recall the [general homework rubric](http://stat545-ubc.github.io/peer-review01_marking-rubric.html).

## Authors

Written by [Andrew MacDoanld][] and [Jenny Bryan][].

[Julia Gustavsen]: https://twitter.com/polesasunder
[Jenny Bryan]: http://www.stat.ubc.ca/~jenny/