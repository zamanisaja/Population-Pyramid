---
title: "Shiny Application"
author: "Sadjad Zamani"
always_allow_html: true
output: 
  html_document:
    keep_md: true
---



## Introduction
In this demo I want to plot the age demographics of different countries around the world.
I'm getting the data from [The World Bank](https://www.worldbank.org/en/home) using this [api](https://github.com/gshs-ornl/wbstats)


## Packages

```r
library(wbstats)
library(tidyverse)
library(ggplot2)
wb_search("population ages.*male")[1:5, ]
```

```
## # A tibble: 5 × 3
##   indicator_id         indicator                                  indicator_desc
##   <chr>                <chr>                                      <chr>         
## 1 SH.CON.1524.FE.ZS    Condom use, population ages 15-24, female… Condom use, f…
## 2 SH.CON.1524.MA.ZS    Condom use, population ages 15-24, male (… Condom use, m…
## 3 SL.TLF.CACT.FM.NE.ZS Ratio of female to male labor force parti… Labor force p…
## 4 SL.TLF.CACT.FM.ZS    Ratio of female to male labor force parti… Labor force p…
## 5 SP.POP.0004.FE       Population ages 00-04, female              Female popula…
```


```r
# array of ["SP.POP.0004.FE" .. "SP.POP.2024.MA" .. ]
indicators <- paste(rep(
    c(
        "SP.POP.0004", "SP.POP.0509",
        paste("SP.POP.", 5 * c(2:15), 5 * c(2:15) + 4, sep = ""),
        "SP.POP.80UP"
    ), 2
), c(".FE", ".MA"), sep = "")

# to get the orginal data I used this command
# df <- wb_data(indicator = indicators, start_date = 2001, end_date = 2020)
df <- read_csv2("data.csv", show_col_types = FALSE)

df <- left_join(
    df,
    wb_countries()[, c("iso2c", "region")],
    by = c("iso2c")
)

df <- df %>%
    ##
    mutate(region = if_else(country == "Namibia", "Sub-Saharan Africa", region))

df <- df %>%
    relocate(region, .after = "country") %>%
    select(-c("iso2c", "iso3c")) %>%
    rename(year = date)
```


```r
df <- df %>%
    pivot_longer(
        cols = indicators,
        names_to = c("age", "gender"),
        names_pattern = "SP.POP.(.*).(..)",
        values_to = "population"
    ) %>%
    mutate(gender = if_else(gender == "FE", "Female", "Male")) %>%
    mutate(gender = as.factor(gender)) %>%
    mutate(age = as.factor(age))
```
