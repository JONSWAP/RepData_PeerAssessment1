---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as `NA`)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


## Loading and preprocessing the data

The data is collected by folking githut project with files "activity.zip".

Unzip the file and save to activitydata.


```r
unzip("activity.zip")
activitydata<-read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

Change date from character to date class, summary by date, and take mean.

```r
activitydata$date<-as.Date(activitydata$date,format="%Y-%m-%d")
sumbydate<-tapply(activitydata$steps,activitydata$date,sum)
meanperday<-mean(sumbydate,na.rm=T)
```

Plot the daily mean in histogram and total mean in red line for exploratory.

```r
barplot(sumbydate)
abline(h=meanperday,col="red")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## What is the average daily activity pattern?

Calculate steps mean value by interval, then combine with time vector.

```r
dailyactivity<-tapply(activitydata$steps,activitydata$interval,mean,na.rm=T)
dailypattern<-data.frame("time"=unique(activitydata$interval), 
                         "dailyactivity"=dailyactivity)
```

Plot the value by histogram.

```r
plot(dailypattern,type="l",xlim=c(0,2355))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
## Imputing missing values

Import library VIM for impute missing values by K-mean.

```r
library(VIM)
```

```
## Warning: package 'VIM' was built under R version 4.0.2
```

```
## Loading required package: colorspace
```

```
## Loading required package: grid
```

```
## VIM is ready to use.
```

```
## Suggestions and bug-reports can be submitted at: https://github.com/statistikat/VIM/issues
```

```
## 
## Attaching package: 'VIM'
```

```
## The following object is masked from 'package:datasets':
## 
##     sleep
```

```r
actdataimputed<-kNN(activitydata,variable = "steps")
```

calculate the mean value to compare with the data with missing values.

```r
sumbydate<-tapply(actdataimputed$steps,actdataimputed$date,sum)
meanperday<-mean(sumbydate)
```

## Are there differences in activity patterns between weekdays and weekends?

Import library dplyr, tidyr, ggplot2

```r
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 4.0.2
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(tidyr)
```

```
## Warning: package 'tidyr' was built under R version 4.0.2
```

```r
library(ggplot2)
```

Add column weekday

```r
actdatabyday<-mutate(actdataimputed,day=weekdays(date))
```

Create Function to check weekday or weekend

```r
checkfunction<-function(x){ifelse(startsWith(x,"S"),"Weekend","Weekday")}
```

Get vector of weekday / weekend info

```r
check<-checkfunction(actdatabyday$day)
```

Add column to check weekday or weekend

```r
actdatabyday<-cbind(actdatabyday,check)
```

Edit data by interval and Weekday/Weekend

```r
dailyactbycheck<-with(actdatabyday,tapply(steps,list(interval,check),mean))
```

Tidy dataframe for plot

```r
patternbyday<-data.frame("time"=unique(actdatabyday$interval), 
                              dailyactbycheck)
patternbycheck<-gather(patternbyday,"daycheck","step",Weekday:Weekend)
```

Plot data

```r
g<-ggplot(patternbycheck,aes(x=time,y=step,color=daycheck))
g+geom_line()+facet_grid(.~daycheck)
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->
