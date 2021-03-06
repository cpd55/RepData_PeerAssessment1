---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
setwd('c:/GIT/RepData_PeerAssessment1/')
activities <- read.csv('activity.csv')
activities$date  <- as.Date(activities$date)
head(activities)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day


```r
library(plyr)
sum_steps <- ddply(activities, c("date"), summarise, number.of.steps=sum(steps), daily.mean=mean(steps), daily.median=median(steps, na.rm = FALSE))
hist(sum_steps$number.of.steps,main = "Histogram of total number of steps taken",xlab='Total number of steps taken',breaks=15)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

2. Reporting the mean and median total number of steps taken per day

a. Mean:


```r
mean(sum_steps$number.of.steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

b. Median:


```r
median(sum_steps$number.of.steps, na.rm = TRUE)
```

```
## [1] 10765
```
 
## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
activities_avg=mean(activities$steps, na.rm=TRUE)
plot(x=activities$steps,type="l",xlab="time intervalls", ylab="number of steps", main="Steps per time intervalls" )
abline(h=activities_avg,col="red")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
#The daily maximum steps
daily_max_steps <- ddply(activities, c("date"), summarise, daily.max=max(steps),.drop=FALSE)
#Maerging back to activity to get the max activity intervalls 
daily_max_intervals <- merge(x=daily_max_steps[!is.na(daily_max_steps$daily.max),], y=activities[!is.na(activities$steps),],by.x=c("date","daily.max"),by.y=c("date","steps"),all=FALSE)
hist(daily_max_intervals$interval,main = "Histogram of the most active intervals",xlab='Interval no.',breaks=15)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

The most active intervalls are around 800-900.   

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs) The total number of activities are  The total number of NA steps are 0.

2. The strategy, for filling in all of the missing values in the dataset, is to get the all times mean for all the data and impute it into the the missing data.

3. The new dataset that is equal to the original dataset but with the missing data filled in


```r
activities_nona = activities
activities_nona[is.na(activities_nona$steps),"steps"]=mean(activities$steps,na.rm=TRUE)
```

4. What is mean total number of steps taken per day on the imputed data.?


```r
library(plyr)
sum_steps <- ddply(activities_nona, c("date"), summarise, number.of.steps=sum(steps), daily.mean=mean(steps), daily.median=median(steps, na.rm = FALSE))
hist(sum_steps$number.of.steps,main = "Histogram of total number of steps taken",xlab='Total number of steps taken',breaks=15)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

Reporting the mean and median total number of steps taken per day on the imputed data

a. Mean:

```r
mean(sum_steps$number.of.steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
b. Median:

```r
median(sum_steps$number.of.steps, na.rm = TRUE)
```

```
## [1] 10766.19
```


## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
week_or_weekend <- function(x) {
  if(as.Date(x)){
        if (weekdays(x) %in% c("Saturday", "Sunday")) {
            "weekend"
        } else {
            "weekday"
        }
    }
}
activities_nona$week.or.weekend <- as.factor(sapply(activities_nona$date, week_or_weekend))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
activity_avg <- ddply(activities_nona, c("week.or.weekend", "interval"), summarise, steps_avg=mean(steps))


library(ggplot2)
plot <- ggplot(activity_avg) + 
  geom_line(aes(x=interval, y=steps_avg)) +
  facet_wrap(~week.or.weekend, nrow=2, scales="free") +
  labs(y="Number of steps", x="Interval", title="Number of steps in the given time period") 
plot
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
