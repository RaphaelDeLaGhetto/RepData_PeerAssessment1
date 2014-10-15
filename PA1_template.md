---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

Reproducible Research: Peer Assessment 1
========================================

This is the first of two project submissions for Coursera's Reproducible
Reasearch course. Its intent is to produce a literate statistical analysis on
data collected from a personal activity monitoring device. The data upon which
this analysis is performed was obtained by forking the course repository
on 2014-10-14.

## Loading and preprocessing the data

The data provided from the original repository was zipped. Unzip the data if
it has not been unzipped already (also, turn off scientific notation when
outputing values):


```r
if (!is.element('activity.csv', dir())) {
    unzip('activity.zip')
}
options(scipen = 1, digits = 2)
```

Load the data contained in `activity.csv`:


```r
data <- read.csv('activity.csv')
```

Since most of the inquiry pertains to daily activity, the `data` column is
converted to `R` `Date` objects:


```r
data$date <- as.Date(data$date)
```

## What is mean total number of steps taken per day?

Determine the total number of steps taken each day. These values are used
to plot a histogram marking their frequency and to calculate their mean and
median:


```r
totalSteps <- tapply(data$steps, data$date, sum)
hist(totalSteps, xlab='Steps Taken', ylim=c(0,30), 
    main='Frequency of total steps taken daily, October-November 2012');
```

![plot of chunk sumSteps](figure/sumSteps-1.png) 

```r
meanSteps <- mean(totalSteps, na.rm=TRUE)
medianSteps <- median(totalSteps, na.rm=TRUE)
```

This subject takes an **average of 10766.19** steps daily with a **median value of
10765**.

## What is the average daily activity pattern?

The average number of steps taken during each five minute interval across all
days is calculated:


```r
averageSteps <- tapply(data$steps, data$interval, mean, na.rm=TRUE)
plot(names(averageSteps), averageSteps, type='l', ylab='Average Steps',
    xlab='5 Minute Interval',
    main='Average number of steps taken at each 5 minute interval')
```

![plot of chunk timeSeries](figure/timeSeries-1.png) 

```r
averageSteps <- data.frame(interval=names(averageSteps), steps=averageSteps)
maxInterval <- averageSteps[averageSteps$steps==max(averageSteps$steps), 'interval']
```

On average, this subject takes the greatest number of steps at
**interval 835**.

## Imputing missing values

Identify missing values in the dataset:


```r
isMissing <- is.na(data$steps)
missingCount <- length(isMissing[isMissing==TRUE])
```

There are a total of **2304** missing values in this dataset.

Using the averages collected for each daily interval, fill in the missing values:


```r
imputedData <- merge(data, averageSteps, by.x='interval', by.y='interval')
imputedData$steps.x[is.na(imputedData$steps.x)] <-
    imputedData$steps.y[is.na(imputedData$steps.x)]
imputedTotalSteps <- tapply(imputedData$steps.x, imputedData$date, sum)
hist(imputedTotalSteps, xlab='Steps Taken',
    main='Frequency of total steps taken daily, October-November 2012');
```

![plot of chunk imputedData](figure/imputedData-1.png) 

```r
meanAdjustedSteps <- mean(imputedTotalSteps, na.rm=FALSE)
medianAdjustedSteps <- median(imputedTotalSteps, na.rm=FALSE)
```

According to the imputed data, this subject takes an **average of
10766.19** steps daily with a **median value of
10766.19**. This average value is identical to the original
average calculation. The median value differs by
1.19, which is negligible. Imputing missing
data in this case has no signficant impact on the daily step count estimate.

## Are there differences in activity patterns between weekdays and weekends?

Add a column to the imputed data frame and classify each row as occuring on a 
*Weekend* or *Weekday*. Split the results on the same factor values:


```r
imputedData$day <- as.factor(
    ifelse(weekdays(imputedData$date) %in% c('Saturday', 'Sunday'),
    'Weekend', 'Weekday'))
imputedData <- split(imputedData, imputedData$day)
```

Plot the average steps taken on the weekend alongside those taken on the
weekdays in five minute intervals:


```r
par(mfrow=c(2,1))
averageWeekdaySteps <-
    tapply(imputedData$Weekday$steps.x, imputedData$Weekday$interval, mean)
plot(names(averageWeekdaySteps), averageWeekdaySteps, type='l',
    ylab='Average Steps', xlab='5 Minute Intervals',
    main='Average number of steps taken at each 5 minute interval\non weekdays')

averageWeekendStepsa <- 
    tapply(imputedData$Weekend$steps.x, imputedData$Weekend$interval, mean)
plot(names(averageWeekendSteps), averageWeekendSteps, type='l',
    ylab='Average Steps', xlab='5 Minute Intervals',
    main='Average number of steps taken at each 5 minute interval\non weekends')
```

![plot of chunk weekdayPlot](figure/weekdayPlot-1.png) 

These plots suggest that the subject takes more steps on the weekend than
during weekdays, and that activity is sustained for longer periods. They also
suggest that the subject may like to sleep in a bit.

