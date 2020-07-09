---
title: "PA1_template.Rmd"
author: "NithaDuff"
date: "7/9/2020"
output: 
  html_document:
    keep_md: true
---
##Loading and preprocessing the data
Show any code that is needed to
1. Load the data (i.e. read.csv())

```r
library(dplyr)
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
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 4.0.2
```

```r
activity <- read.csv("activity.csv", sep = ",",header = T,na.strings = "NA") 
```
2. Process/transform the data (if necessary) into a format suitable for analysis

```r
activity_bydate <- activity %>% 
  group_by(date) %>% 
  summarise(daily_steps = sum(steps,na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

##Mean total number of steps taken per day
Ignore the missing values in the dataset.
1. Histogram of the total number of steps taken each day

```r
g <- ggplot(data = activity_bydate, aes(daily_steps)) +
     geom_histogram() + labs(x= "Steps per day",y = "count",title = "Daily Steps") 
print(g)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/histogram-1.png)<!-- -->
2. The mean and median total number of steps taken
per day

```r
nmean <- mean(activity_bydate$daily_steps)
nmedian <- median(activity_bydate$daily_steps)
```
The Mean is 9354.2295082 and the median is 10395   
##The average daily activity pattern
1. Time series plot (i.e. type = "l") of the 5-minute interval (x-axis)
and the average number of steps taken, averaged across all days (y-axis)

Data frame that is grouped by interval:


```r
activity_byinterval <- activity %>%
  group_by(interval) %>% 
  summarise(average_steps = mean(steps,na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```
Plot the data using intervals on the x-axis and steps on the y-axis:

```r
with(na.omit(activity_byinterval), plot(x = interval,y = average_steps, type = "l",xlab= "Interval",ylab = "Average Steps Taken",main = "Time line"))
```

![](PA1_template_files/figure-html/timeline_plot-1.png)<!-- -->
2. 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps.


```r
activity[activity$average_steps == max(activity$average_steps),1]
```

```
## Warning in max(activity$average_steps): no non-missing arguments to max;
## returning -Inf
```

```
## integer(0)
```
##Imputing missing values  

1. The total number of missing values in the dataset
(i.e. the total number of rows with NAs)

```r
sum(is.na(activity))
```

```
## [1] 2304
```
2. A new dataset that is equal to the original dataset but with the
missing data filled in.

```r
imputed_activity <- activity
missing <- which(is.na(imputed_activity$steps))
for (i in missing) { #imputed data
  imputed_activity[i,1] <- activity_byinterval[activity_byinterval$interval == imputed_activity[i,3],2]
}
```
4. A histogram of the total number of steps taken each day 

```r
imputed_activity_bydate <- imputed_activity %>% 
  group_by(date) %>% 
  summarise(daily_steps = sum(steps,na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
g <- ggplot(data = imputed_activity_bydate, aes(daily_steps)) +
  geom_histogram() + labs(x= "Steps per day",y = "count",title = "Daily Steps with imputed data") 
print(g)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/hist2-1.png)<!-- -->
The mean and median total number of steps taken per day. The impact of imputing missing data on the estimates of the total daily number of steps.

```r
imean <- mean(imputed_activity_bydate$daily_steps)
imean
```

```
## [1] 10766.19
```

```r
imedian <- median(imputed_activity_bydate$daily_steps)
imedian
```

```
## [1] 10766.19
```
The Mean is 1.0766189\times 10^{4} and the median is 1.0766189\times 10^{4}   
This plot shows the relation between the effects of mean and median values among data that has missing values and the data with imputed values.

```r
plot(rep(1,2),c(nmean,nmedian),type = "n",xlim = c(1,2.5),ylim = c(9000,max(c(imedian,imean))))
points(rep(2,2),c(imean,imedian))
segments(1,nmean,2,imean,col = "Blue")
segments(1,nmedian,2,imedian,col = "Red")
```

![](PA1_template_files/figure-html/relation_plot-1.png)<!-- -->
##Differences in activity patterns between weekdays and weekends

1. Create a new factor variable in the dataset with two levels – “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day.

```r
imputed_activity$dayOfWeek <- factor((weekdays(as.Date(imputed_activity$date)) %in% c("Saturday","Sunday")),levels=c(TRUE, FALSE), labels=c('weekend', 'weekday')) 
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken, averaged
across all weekday days or weekend days (y-axis).

```r
imputed_activity_bydateinterval <- imputed_activity %>% group_by(interval,dayOfWeek) %>% summarise(average_steps = mean(steps,na.rm = T))
```

```
## `summarise()` regrouping output by 'interval' (override with `.groups` argument)
```

```r
g <- ggplot(imputed_activity_bydateinterval,aes(x = interval,y = average_steps))+ geom_line() + facet_grid(dayOfWeek ~ .) + labs(x= "Interval",y = "Average Steps Taken",title = "Daily Steps with imputed data compared to weekends/weekdays")
print(g)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->
