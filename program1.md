---
title: 'Reproducible Research: Peer Assessment 1'
output: html_document
---


##Loading and preprocessing the data

```r
activity<-read.csv("activity.csv")
```
##Process/Transform the data into a format suitable for your analysis

```r
library(plyr)
activity<-read.csv("activity.csv",colClasses=c("integer","Date","integer"))
#q1
stepsperday<-ddply(activity,c("date"),summarise,totalsteps=sum(steps,na.rm=TRUE))
stepsper5min<-ddply(activity,c("interval"),summarise,meansteps=mean(steps,na.rm = TRUE))
```

##Histogram of the total number of steps taken each day

```r
#q2
library(ggplot2)
stepshist<-ggplot(stepsperday,aes(x=totalsteps))+geom_histogram()+
  xlab("Total number of steps")+
  ggtitle("Histogram of total steps in one day")+
  theme_bw()
print(stepshist)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)
##What is mean total number of steps taken per day?

```r
#q3
mean(stepsperday$totalsteps, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(stepsperday$totalsteps)
```

```
## [1] 10395
```
##What is the average daily activity pattern?

```r
dayline<-ggplot(stepsper5min,aes(x=interval,y=meansteps))+geom_line()+
  ggtitle("Average steps for each 5-min interval")+
  ylab("Mean steps")+
  theme_bw()
print(dayline)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

##The 5-minute interval that, on average, contains the maximum number of steps

```r
stepsper5min[which(stepsper5min$meansteps==max(stepsper5min$meansteps)), "interval"]  
```

```
## [1] 835
```

```r
stepsper5min[which(stepsper5min$meansteps==max(stepsper5min$meansteps)), "meansteps"] 
```

```
## [1] 206.1698
```

##imputing missing data

```r
step_interpolation <- function(rownumber){
  prevrow=rownumber;
  nextrow=rownumber;
  while(is.na(activity$steps[prevrow])){
    prevrow=prevrow-1
    if(prevrow<1)return(mean(activity[activity$interval==activity$interval[rownumber],"steps"],na.rm=TRUE))
  }
  while(is.na(activity$steps[nextrow])){
    nextrow=nextrow+1
    if(nextrow>nrow(activity))return(mean(activity[activity$interval==activity$interval[rownumber],"steps"],na.rm=TRUE))
  }
  return(
    (activity$steps[prevrow]+activity$steps[nextrow])/2
  )
}

activity_guessNA <-activity
for(n in 1:nrow(activity)){
  if(is.na(activity$steps[n])){
    activity_guessNA$steps[n]=step_interpolation(n);
  }
}
```
#I know, this is a density plot not a histogram, but the meaning is the same and I didn't want to superimpose two histograms. The imputed dataset has (relatively) fewer zeros, the original data is peppered with lone zeros and the imputation strategy above just doesn't reproduce this pattern. Most of the imputed entries appear to have been added in the most commonly occuring range.

```r
stepsperday2<-merge(
  ddply(activity_guessNA, c("date"),summarise,
        guesstotalsteps=sum(steps,na.rm=TRUE)
  ),
  stepsperday,
  by="date"
)

guesscheck<-ggplot(stepsperday2,aes(x=totalsteps))+
  geom_density()+
  geom_density(aes(x=guesstotalsteps,color="Imputed"))+
  ggtitle("Density plot comparing raw and NA-imputed activity datasets")+
  xlab("total steps")+
  theme_bw()
print(guesscheck)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)
#Here's the histogram for my fellow pedants:

```r
forpeoplewhoreallywanttoseeahistogram<-ggplot(stepsperday2,aes(x=guesstotalsteps))+geom_histogram()+ggtitle("Histogram of total number of steps per day after missing values imputed")+
    theme_bw()
print(forpeoplewhoreallywanttoseeahistogram)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)
#The mean and median total steps are r mean(stepsperday2$totalsteps,na.rm=TRUE) and r median(stepsperday2$totalsteps,na.rm=TRUE), for the NA-imputed data the mean and median are r mean(stepsperday2$guesstotalsteps,na.rm=TRUE) and r median(stepsperday2$guesstotalsteps,na.rm=TRUE).

##Are there differences in activity patterns between weekdays and weekends?

```r
paindays= c("Monday","Tuesday","Wednesday","Thursday","Friday")

activity_guessNA$weekday<-as.factor(ifelse(weekdays(activity_guessNA$date)%in%paindays,"weekday","weekend"))

stepsperinterval.weekdaysplit<-ddply(activity_guessNA, c("interval","weekday"),summarise,
                    meansteps = mean(steps,na.rm=TRUE)
)

weekdayplot<-ggplot(stepsperinterval.weekdaysplit,aes(x=interval,y=meansteps))+
  facet_wrap(~weekday,nrow=2,ncol=1)+
  geom_line()+
  theme_bw()+
  ggtitle("Mean steps over each 5min interval split by weekday/weekend")+
  ylab("Mean steps")+
  xlab("Interval number")
print(weekdayplot)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)