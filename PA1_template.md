# Reproducible Research: Peer Assessment 1 Submission


## Loading and preprocessing the data

#####Assumes you have downloaded this information and unzipped the data into your working directory. 


```r
activity <- read.csv("/Users/michaelmorgan/Desktop/octproject1/activity.csv", header = TRUE, stringsAsFactors = FALSE)
```


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggvis)
library(ggplot2)
```

```
## 
## Attaching package: 'ggplot2'
## 
## The following object is masked from 'package:ggvis':
## 
##     resolution
```

```r
library(lubridate)
```


## What is mean total number of steps taken per day?

#####First, calculate the total number of steps


```r
totalsteps <- group_by(activity, date) %>% filter(!is.na(steps)) %>% summarize(totalsteps = sum(steps))
```

#####Next, here is a histogram of the total number of steps taken per day. 


```r
library(ggplot2)

firsthist <- ggplot(totalsteps, aes(totalsteps)) + geom_histogram() + theme_bw() + ggtitle("Histogram of Steps per Day (NAs removed)") + xlab("Total Steps in a Day") + ylab("Number of Days with That Many Steps")
firsthist  
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

#####Getting the mean and the median becomes fairly easy. This is the data that had NAs in it. 


```r
mean <- mean(totalsteps$totalsteps)
median <- median(totalsteps$totalsteps)
xna <- c(mean = mean, median = median)
print(xna)
```

```
##     mean   median 
## 10766.19 10765.00
```

## What is the average daily activity pattern?

#####Below is a time series graph displaying this information. 


```r
cleaned <- filter(activity, !is.na(steps)) 
cleaned <- group_by(cleaned, interval) %>% summarize(NumSteps = mean(steps))

ggplot(cleaned, aes(interval, NumSteps)) + geom_line(color = "blue") + 
        xlab("Interval") + ylab("Average Steps per Day") + theme_bw() + 
        ggtitle("Time Series Plot of Avg. Steps per Day per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 


#####The following interval, on average across all days in the data set, contains the maximum number of steps. 


```r
x <- arrange(cleaned, desc(NumSteps))
print(x[1,1])
```

```
## Source: local data frame [1 x 1]
## 
##   interval
## 1      835
```

## Imputing missing values

#####The total number of missing values is calculated below 


```r
nas <- sum(is.na(activity))
print(nas)
```

```
## [1] 2304
```

#####My strategy for filling in the NA values was to pull out all the observations with NAs into its own data frame, and then to join in the cleaned data with the average steps by interval created above. Joining by interval allowed me to replace each NA data point with the average number of steps for that interval. After correcting headers in this cleaned NA data set, I appended it to the original data set, from which NAs had been removed, to create a combined data set. 


```r
activity2 <- arrange(activity, desc(steps))
extractnas <- filter(activity2, is.na(steps))
c <- left_join(extractnas, cleaned, by = "interval")
c <- select(c, NumSteps, date, interval)
activityclean <- filter(activity, !is.na(steps))
names(c)[1] <- "steps"
activityclean <- rbind(activityclean, c)
```

#####Next, I used this clean data to create a data set and histogram comparable to the first histogram. tclean is a data set I used for to calculate the actual difference between the original data; the histogram is created by the activityclean data, piped all the way to the graph. 


```r
tclean <- group_by(activityclean, date) %>% summarize(totalsteps = sum(steps))
```

#####Histogram of cleaned data 


```r
secondhist <- ggplot(tclean, aes(totalsteps)) + geom_histogram() + theme_bw() + ggtitle("Histogram of Steps per Day (NAs replaced)") + xlab("Total Steps in a Day") + ylab("Number of Days with That Many Steps")
secondhist 
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

#####Here are the mean and median number of steps, cleaned. 


```r
cleanmean <- mean(tclean$totalsteps)
cleanmedian <- median(tclean$totalsteps)
xclean <- c(mean = cleanmean, median = cleanmedian)
print(xclean)
```

```
##     mean   median 
## 10766.19 10766.19
```

#####For the actual difference between the original mean and median, and that of the cleaned data. 


```r
xna - xclean
```

```
##      mean    median 
##  0.000000 -1.188679
```

## Are there differences in activity patterns between weekdays and weekends?

#####I used lubridate (wday, label = FALSE) to deal with the weekend or weekday issue (which, honestly, is pretty amazing -- I had to check actual calendars to make sure it worked that easily). After creating the data sets with the weekend and weekday data, I calculated the averages. I then joined the interval data back in, and used rbind to combine the two data sets. Finally, I used ggplot to create the panel graph. 


```r
days <- mutate(tclean, day = (wday(date, label = FALSE)))
days <- days %>% mutate(wkndwkd = ifelse(day == 1 | day == 7, "Weekend", "Weekday"))

joined <- left_join(activityclean, days, by = "date")
weekend <- filter(joined, wkndwkd == "Weekend")
weekday <- filter(joined, wkndwkd == "Weekday")

weekday <- group_by(weekday, interval, wkndwkd) %>% summarize(AvgSteps = mean(steps))
weekend <- group_by(weekend, interval, wkndwkd) %>% summarize(AvgSteps = mean(steps))
forgraph <- rbind(weekend, weekday)

lastgraph <- ggplot(forgraph, aes(interval, AvgSteps)) + geom_line(color = "black") + 
        xlab("Interval") + ylab("Average Steps per Day") + theme_bw() + 
        ggtitle("Time Series Plot of Avg. Steps per Day per Interval") + facet_grid(wkndwkd~.) + theme(plot.background=element_rect(fill="gray"))

print(lastgraph)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

#####Good luck, I am off to fly-fish near the top of the world. 

