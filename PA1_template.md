# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

```r
if(!file.exists("activity.csv")) {
  unzip("activity.zip")
}
data <- read.csv("activity.csv", header=TRUE, sep = ",", na.strings = "NA", colClasses=c("numeric","character","numeric"))
data$date <- as.Date(data$date, "%Y-%m-%d")
```

## What is mean total number of steps taken per day?
histogram of the total number of steps taken each day

```r
steps <- aggregate(steps ~ date, data, sum, na.rm = TRUE)
hist(steps$steps,plot=TRUE, main="Total Steps Histogram")
```

![](./PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

the mean and median total number of steps taken per day

```r
mean(steps$steps)
```

```
## [1] 10766.19
```

```r
median(steps$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
stepsbyinterval <- aggregate(steps ~ interval, data, mean, na.rm = TRUE)
plot(stepsbyinterval$interval, stepsbyinterval$steps,type="l", main="Steps by Interval")
```

![](./PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

Which 5-minute interval, on average across all the days in the dataset,contains the maximum number of steps?

```r
stepsbyinterval[which.max(stepsbyinterval$steps),]$interval
```

```
## [1] 835
```

## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(!complete.cases(data))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Lets fill NA steps with the average steps for the interval from the previous calculations.

Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
dataFilledWithMean <- data
rowCount = nrow(dataFilledWithMean)
for (i in 1:rowCount) {
  if (is.na(dataFilledWithMean$steps[i])) {
    meanForInterval <- stepsbyinterval$steps[which(stepsbyinterval$interval == dataFilledWithMean$interval[i])]
    dataFilledWithMean$steps[i] <- meanForInterval
  }
}
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
stepsfilled <- aggregate(steps ~ date, dataFilledWithMean, sum)
hist(stepsfilled$steps,plot=TRUE, main="Total Steps by day NA filled with mean values")
```

![](./PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

```r
mean(stepsfilled$steps)
```

```
## [1] 10766.19
```

```r
median(stepsfilled$steps)
```

```
## [1] 10766.19
```

```r
## Compare values between original and filled data
mean(stepsfilled$steps) - mean(steps$steps)
```

```
## [1] 0
```

```r
median(stepsfilled$steps) - median(steps$steps)
```

```
## [1] 1.188679
```


## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
weekdays <- weekdays(dataFilledWithMean$date)
dataFilledWithMean$day <- ifelse(weekdays %in% c("lauantai", "sunnuntai"), "Weekend","Weekday")
dataFilledWithMean <- transform(dataFilledWithMean, day=factor(day))
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
library(lattice)
stepsbyintervalDay <- aggregate(steps ~ interval + day, dataFilledWithMean, mean)
xyplot(steps ~ interval | day, data = stepsbyintervalDay, type = "l", aspect=1/2, main="Comparison weekend vs weekday")
```

![](./PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

EOF
