# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Load required packages:

```r
library(ggplot2) 
```

Read in the data:

```r
activity<-read.csv("C:/Users/Martin/Documents/GitHub/RepData_PeerAssessment1/activity/activity.csv") 
```

Remove NAs:

```r
activity.no_na<-activity[!is.na(activity$steps),]
```

Aggregate data to mean per day and rename columns:

```r
activity.mean<-aggregate(activity.no_na$steps, by = list(activity.no_na$date), FUN = mean)
colnames(activity.mean)=c("date","mean.steps")
```

Aggregate data to sum per day and rename columns:

```r
activity.sum<-aggregate(activity.no_na$steps, by = list(activity.no_na$date), FUN = sum)
colnames(activity.sum)=c("date","total.steps")
```

## What is mean total number of steps taken per day?
Plot a histogram:

```r
ggplot(aes(total.steps), data=activity.sum)+
  geom_histogram(binwidth = max(activity.sum$total.steps)/30)+
  ggtitle("Histogram of the total number of steps taken each day")+
  theme_bw()
```

![plot of chunk figure1](figure/figure1.png) 

Mean total steps per day:

```r
mean(activity.sum$total.steps)
```

```
## [1] 10766
```

Median total steps per day:

```r
median(activity.sum$total.steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Average data:

```r
activity.average<-aggregate(activity.no_na$steps, by = list(activity.no_na$interval), FUN = mean)
colnames(activity.average)=c("interval","mean.steps")
```
Time series plot of the 5-minute interval and the average number of steps taken, averaged across all days:

```r
time.series.plot<-ggplot(aes(interval,mean.steps), data=activity.average)+
  geom_line()+
  ggtitle("Time series plot")+
  theme_bw()
print(time.series.plot)
```

![plot of chunk figure2](figure/figure2.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max.interval<-activity.average$interval[activity.average$mean.steps==max(activity.average$mean.steps)]
print(max.interval)
```

```
## [1] 835
```

## Imputing missing values
The total number of rows with NAs:

```r
nrow(activity[is.na(activity$steps),])
```

```
## [1] 2304
```

Filling in all of the missing values with average value from same interval:

```r
activity.imput<-activity
for (t in unique(activity.average$interval)){
  activity_sub<-activity[activity$interval==t,]
    activity_sub$steps[is.na(activity_sub$steps)]<-activity.average$mean.steps[activity.average$interval==t]
    activity.imput[activity.imput$interval==t,]<-activity_sub
}
```

Average data after imput:

```r
activity.average.imput<-aggregate(activity.imput$steps, by = list(activity.imput$interval), FUN = mean)
colnames(activity.average.imput)=c("interval","mean.steps")
```

Time series plot of the 5-minute interval and the average number of steps taken, averaged across all days, after the imput:

```r
time.series.plot<-ggplot(aes(interval,mean.steps), data=activity.average.imput)+
  geom_line()+
  ggtitle("Time series plot")+
  theme_bw()
print(time.series.plot)
```

![plot of chunk figure3](figure/figure3.png) 

Aggregate data to sum per day after imput and rename columns:

```r
activity.sum.imput<-aggregate(activity.imput$steps, by = list(activity.imput$date), FUN = sum)
colnames(activity.sum.imput)=c("date","total.steps")
```

Mean total steps per day:

```r
mean(activity.sum.imput$total.steps)
```

```
## [1] 10766
```

Median total steps per day:

```r
median(activity.sum.imput$total.steps)
```

```
## [1] 10766
```
Now, the median is equivalent to the mean.

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day:

```r
activity.imput$weekday<-weekdays(strptime(activity.imput$date,"%Y-%m-%d"))
activity.imput$weekday[activity.imput$weekday=="Samstag"|activity.imput$weekday=="Sonntag"]<-"weekend"
activity.imput$weekday[!activity.imput$weekday=="weekend"]<-"weekday"
activity.imput$weekday<-factor(activity.imput$weekday)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:

```r
activity.weekdays<-aggregate(activity.imput$steps,by=list(activity.imput$interval,activity.imput$weekday),mean)
colnames(activity.weekdays)<-c("interval","weekday","steps")

ggplot(aes(interval,steps),data=activity.weekdays)+
  geom_line()+
  facet_grid(weekday~.)+
  theme_bw()
```

![plot of chunk figure4](figure/figure4.png) 
