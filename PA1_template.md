
# Reproducible Research Project 1


```r
knitr::opts_chunk$set(echo = TRUE)
```

## Loading and preprocessing the data

```r
activity <- read.csv("activity.csv")
activity$day <- weekdays(as.Date(activity$date))
activity$DateTime<- as.POSIXct(activity$date, format="%Y-%m-%d")
```

## What is mean total number of steps taken per day?
### Calculate the total number of steps taken per day

```r
clean <- activity[!is.na(activity$steps),]
#Calculate the total number of steps taken per day
sumTable <- aggregate(clean$steps ~ clean$date, FUN=sum, )
colnames(sumTable)<- c("Date", "Steps")
```

### Make a histogram of the total number of steps taken each day

```r
hist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

### Calculate and report the mean and median of the total number of steps taken per day

```r
## Mean of Steps
mean <- as.integer(mean(sumTable$Steps))
## Median of Steps
median <- as.integer(median(sumTable$Steps))
```
The average number of steps taken each day was 10766 steps.
The median number of steps taken each day was 10765 steps.

## What is the average daily activity pattern?
### Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
library(plyr)
library(ggplot2)
##pulling data without nas
clean <- activity[!is.na(activity$steps),]

##create average number of steps per interval
intervalTable <- ddply(clean, .(interval), summarize, Avg = mean(steps))

##Create line plot of average number of steps per interval
p <- ggplot(intervalTable, aes(x=interval, y=Avg), xlab = "Interval", ylab="Average Number of Steps")
p + geom_line()+xlab("Interval")+ylab("Average Number of Steps")+ggtitle("Average Number of Steps per Interval")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)

### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
##Maximum steps by interval
maxSteps <- max(intervalTable$Avg)
##Which interval contains the maximum average number of steps
maxInterval<- intervalTable[intervalTable$Avg==maxSteps,1]
```

The 5-minute interval which had the maximum number (206.1698113) of steps was the 835 interval.

## Imputing missing values
### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
##Number of NAs in original data set
N.A<- nrow(activity[is.na(activity$steps),])
```
The total number of rows with steps = ‘NA’ is 2304.

### filling in NAs and substitute the missing steps with the average 5-minute interval based on the day of the week.

```r
## Create the average number of steps per weekday and interval
avgTable <- ddply(clean, .(interval, day), summarize, Avg = mean(steps))

## Create dataset with all NAs for substitution
nadata<- activity[is.na(activity$steps),]
## Merge NA data with average weekday interval for substitution
newdata<-merge(nadata, avgTable, by=c("interval", "day"))
```
### Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
## Reorder the new substituded data in the same format as clean data set
newdata2<- newdata[,c(6,4,1,2,5)]
colnames(newdata2)<- c("steps", "date", "interval", "day", "DateTime")

##Merge the NA averages and non NA data together
mergeData <- rbind(clean, newdata2)
```
### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
##Create sum of steps per date to compare with step 1
sumTable2 <- aggregate(mergeData$steps ~ mergeData$date, FUN=sum, )
colnames(sumTable2)<- c("Date", "Steps")

## Mean of Steps with NA data taken care of
NewMean <- as.integer(mean(sumTable2$Steps))

## Median of Steps with NA data taken care of
NewMedian <- as.integer(median(sumTable2$Steps))

## Creating the histogram of total steps per day, categorized by data set to show impact
hist(sumTable2$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Black")
hist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with NAs Fixed", col="Grey", add=T)
legend("topright", c("Imputed Data", "Non-NA Data"), fill=c("black", "grey") )
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png)

The new mean of the imputed data is 10821 steps compared to the old mean of 10766 steps. 

The new median of the imputed data is 11015 steps compared to the old median of 10765 steps.

However, the overall shape of the distribution has not changed.

## Are there differences in activity patterns between weekdays and weekends?
### Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
## Create new category based on the days of the week
mergeData$DayCategory <- ifelse(mergeData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")
```
### Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
library(lattice) 
## Summarize data by interval and type of day
intervalTable2 <- ddply(mergeData, .(interval, DayCategory), summarize, Avg = mean(steps))

##Plot data in a panel plot
xyplot(Avg~interval|DayCategory, data=intervalTable2, type="l",  layout = c(1,2),
       main="Average Steps per Interval Based on Type of Day", 
       ylab="Average Number of Steps", xlab="Interval")
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png)





