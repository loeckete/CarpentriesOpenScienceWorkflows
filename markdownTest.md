markdownTest
================

# Script for manuscript organization

This example was developed for a carpentries workshop

1.  Prepare an R environment

2.  Download and save USGS data

3.  Tidy data

4.  Report

## 1. prepare the R environment

load libraries

``` r
knitr::opts_chunk$set(echo=T)
library(dataRetrieval)
```

    ## Warning: package 'dataRetrieval' was built under R version 4.1.3

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(zoo)
```

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(ggplot2)
```

## 2. download data

use `readNWISdv` function from the dataRetrieval package to download KS
river data

``` r
usgs_id <- "07141220"

data_raw <- readNWISdv(siteNumbers = usgs_id,
                       parameterCd = "00065",
                       startDate = "2018-10-01",
                       endDate = "2021-09-30")
#inspect data  
summary(data_raw)
```

    ##   agency_cd           site_no               Date            X_00065_00003   
    ##  Length:1089        Length:1089        Min.   :2018-10-01   Min.   : 3.760  
    ##  Class :character   Class :character   1st Qu.:2019-07-03   1st Qu.: 4.330  
    ##  Mode  :character   Mode  :character   Median :2020-03-31   Median : 4.670  
    ##                                        Mean   :2020-03-31   Mean   : 4.919  
    ##                                        3rd Qu.:2020-12-28   3rd Qu.: 4.850  
    ##                                        Max.   :2021-09-30   Max.   :14.510  
    ##  X_00065_00003_cd  
    ##  Length:1089       
    ##  Class :character  
    ##  Mode  :character  
    ##                    
    ##                    
    ## 

``` r
# save the data
write.csv(data_raw, "data/exampleStreamGaugeData.csv")
```

## Tidy data

in this section we will perform some basic

``` r
# create data frame with better colnames
data_tidy <- data_raw %>% 
  rename(stage_ft = X_00065_00003,
         stage_QAcode = X_00065_00003_cd) %>% 
  select(-agency_cd, -site_no)

# look at dataframe  
head(data_tidy)
```

    ##         Date stage_ft stage_QAcode
    ## 1 2018-10-01     4.35            A
    ## 2 2018-10-02     4.34            A
    ## 3 2018-10-03     4.32            A
    ## 4 2018-10-04     4.30            A
    ## 5 2018-10-05     4.31            A
    ## 6 2018-10-06     4.34            A

``` r
# first step is to plot data

ggplot(data=data_tidy, aes(x=Date, y=stage_ft))+
  geom_line()
```

![](markdownTest_files/figure-gfm/clean%20data-1.png)<!-- -->

``` r
# deal with missing data by compare to possible days
first_date <- min(data_tidy$Date)
last_date <- max(data_tidy$Date)

all_dates <- seq(first_date, last_date, by = "day")
length(all_dates) == length(data_tidy)
```

    ## [1] FALSE

``` r
missing_dates <- all_dates[!(all_dates %in% data_tidy$Date)]

# add missing dates

new_dates <- data.frame(Date = missing_dates, state_ft = NA, stage_QAcode = "Gapfill")
data_clean <- bind_rows(data_tidy, new_dates) %>% 
  arrange(Date)

summary(data_clean)
```

    ##       Date               stage_ft      stage_QAcode       state_ft      
    ##  Min.   :2018-10-01   Min.   : 3.760   Length:1096        Mode:logical  
    ##  1st Qu.:2019-07-01   1st Qu.: 4.330   Class :character   NA's:1096     
    ##  Median :2020-03-31   Median : 4.670   Mode  :character                 
    ##  Mean   :2020-03-31   Mean   : 4.919                                    
    ##  3rd Qu.:2020-12-30   3rd Qu.: 4.850                                    
    ##  Max.   :2021-09-30   Max.   :14.510                                    
    ##                       NA's   :7

``` r
# fill in those gaps 

data_clean$stage_ft <- na.approx(data_clean$stage_ft)

summary(data_clean)
```

    ##       Date               stage_ft      stage_QAcode       state_ft      
    ##  Min.   :2018-10-01   Min.   : 3.760   Length:1096        Mode:logical  
    ##  1st Qu.:2019-07-01   1st Qu.: 4.330   Class :character   NA's:1096     
    ##  Median :2020-03-31   Median : 4.670   Mode  :character                 
    ##  Mean   :2020-03-31   Mean   : 4.920                                    
    ##  3rd Qu.:2020-12-30   3rd Qu.: 4.853                                    
    ##  Max.   :2021-09-30   Max.   :14.510

``` r
ggplot(data=data_tidy, aes(x=Date, y=stage_ft, color=stage_QAcode))+
  geom_point()
```

![](markdownTest_files/figure-gfm/clean%20data-2.png)<!-- -->

## 4. Write up some summary stats

We analyzed 07141220. The max stage was 14.51.
