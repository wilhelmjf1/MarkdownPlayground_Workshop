Carpentries Example Streamflow Analysis
================
Jess Wilhelm

This is an example script developed during a Carpentries workshop.
Within it, we will develop a workflow to:

1.) Prepare an R environment 2.) Download and save USGS streamflow data
3.) Tidy, QAQC, and gap-fill the data 4.) Report some summary statistics

## 1\. Prepare the R environment

we will load any relevant packages and set the default for our code
visibility in the final report.

building our first code chunk\!\!

``` r
knitr::opts_chunk$set(echo = TRUE)
library(dataRetrieval) 
```

    ## Warning: package 'dataRetrieval' was built under R version 4.0.5

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 4.0.5

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

if we knit a file it runs all of the code.

## 2\. Download and save USGS streamflow data

We will use the ‘readNWISdv’ function from the ’dataRetrieval package to
download stream stage data for the USGS gage on the Arkansas River near
Larned. {r download-data}

    # gage id
    usgs_id <- "07141220"
    
    # download the data
      data_raw <-
        readNWISdv(siteNumber = usgs_id,
        parameterCd = "00065",
        startDate = "2018-10-01",
        endDate = "2021-09-30")
    
    # inspect the data
    summary(data_raw)
    
    # save the data
    write.csv(data_raw, "data/ExampleStreamStage_Raw.csv")

## 3\. Tidy, gap-fill, and QAQC data

In this section, we will perform some basic data cleaning operations to
get out data ready for further analysis. {r clean-data}

    # create a new data frame with better column names
    data_tidy <-
      data_raw %>%
      rename(stage_ft = X_00065_00003,
             stageQAcode = X_00065_00003_cd) %>%
      select(-agency_cd, -site_no)
    
    #look at the new data frame
    head(data_tidy)
    
    #first step in communing with data is to plot
    ggplot(data_tidy, aes(x = Date, y = stage_ft)) +
      geom_line()
      
      # check for missing dates by comparing all possible dates to the dates you have
      first_date <- min(data_tidy$Date)
      last_date <- max(data_tidy$Date)
      all_dates <- seq(first_date, last_date, by = "day") # make vector of all dates
      length(all_dates) == length(data_tidy$Date)

# determine missing dates

missing\_dates \<- all\_dates\[\!(all\_dates %in% data\_tidy$Date)\]

# add missing dates to data frame

new\_dates \<- data.frame(Date = missing\_dates, stage\_ft = NA,
stage\_QAcode = “Gapfill”)

data\_clean \<- bind\_rows(data\_tidy, new\_dates) %\>% arrange(Date)

summary(data\_clean)

\#fill in those gaps using linear interpolation
data\_clean\(stage_ft <- na.approx(data_clean\)stage\_ft)

summary(data\_clean)

\# plot and inspect ggplot(data\_clean, aes(x = Date, y = stage\_ft,
color = stage\_QAcode)) + geom\_point()

\#save data write.csv(data\_clean, “data/ExampleStreamStage\_Clean.csv”)

\`\`\`
