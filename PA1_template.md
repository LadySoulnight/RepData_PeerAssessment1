---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data


```r
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 3.5.3
```

```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.5.3
```

Show any code that is needed to

* Load the data (i.e. `read.csv()`)


```r
data <- read.csv("activity.csv", colClasses=c("integer", "Date", "integer"))
```

* Process/transform the data (if necessary) into a format suitable for your analysis

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

* Calculate the total number of steps taken per day


```r
steps_per_day <- as.data.frame(data %>%
  group_by(date) %>%
  summarise(steps = sum(steps)))
```

* Make a histogram of the total number of steps taken each day


```r
hist(steps_per_day$steps, main = "Total number of steps taken each day", xlab = "Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Calculate and report the **mean** and **median** of the total number of steps taken per day


```r
mean(steps_per_day$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(steps_per_day$steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

* Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
avg_steps_per_day <- as.data.frame(data %>%
  group_by(interval) %>%
  summarise(steps = mean(steps, na.rm = TRUE)))

plot(avg_steps_per_day$interval, avg_steps_per_day$steps, type="l", xlab = "Interval", ylab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
avg_steps_per_day[which.max(avg_steps_per_day$steps), "interval"]
```

```
## [1] 835
```

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as `NA`). The presence of missing days may introduce bias into some calculations or summaries of the data.

* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

    The missing values will be filled with the average number of steps over all days.

* Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
interpolated_data <- data
interpolated_data[is.na(interpolated_data$steps), "steps"] <- mean(data$steps, na.rm = TRUE)
```

* Make a histogram of the total number of steps taken each day. Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
interpolated_steps_per_day <- as.data.frame(interpolated_data %>%
  group_by(date) %>%
  summarise(steps = sum(steps)))
hist(interpolated_steps_per_day$steps, main = "Total number of steps taken each day", xlab = "Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
mean(interpolated_steps_per_day$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(interpolated_steps_per_day$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
mean(interpolated_steps_per_day$steps, na.rm = TRUE) / mean(steps_per_day$steps, na.rm = TRUE) - 1
```

```
## [1] 0
```

```r
median(interpolated_steps_per_day$steps, na.rm = TRUE) / median(steps_per_day$steps, na.rm = TRUE) - 1
```

```
## [1] 0.0001104207
```

Because the mean is used to fill the NAs there is no difference to the previous one and only a small difference when looking at the median. 

## Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use the dataset with the filled-in missing values for this part.

* Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
weekday_data <- data.frame(
  interpolated_data,
  weekday = as.factor(ifelse(as.POSIXlt(interpolated_data$date)$wday %in% 1:5, 'weekday', 'weekend'))
)
```

* Make a panel plot containing a time series plot (i.e. `type="l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
avg_steps_per_weekday <- as.data.frame(weekday_data %>%
  group_by(interval, weekday) %>%
  summarise(steps = mean(steps, na.rm = TRUE)))

ggplot(avg_steps_per_weekday, aes(interval, steps)) + 
  geom_line() + 
  facet_grid(weekday ~ .) +
  xlab("5-minute interval") + 
  ylab("avarage number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
