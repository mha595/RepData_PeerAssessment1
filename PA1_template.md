---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
 Only obvious preprocessing at this stage is changing the date formate. 

```r
dat <- read.csv("activity.csv") 
dat$date <- as.Date(dat$date, "%Y-%m-%d")
```


## What is mean total number of steps taken per day?
The following block of code performs the following tasks:  
- Take sum of the steps by date  
- Plot a histogram of the total steps   
- Calculate the mean and median of the total steps   



```r
total_steps_by_date <- aggregate(dat$steps, by = list(dat$date), FUN=sum, na.rm=TRUE)
names(total_steps_by_date) <- c("Date", "Total steps")
hist(total_steps_by_date$`Total steps`, xlab = "Steps", main = "Steps taken in a day", breaks = 10)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
step_mean <- mean(total_steps_by_date$`Total steps`, na.rm = TRUE)
step_median <- median(total_steps_by_date$`Total steps`, na.rm = TRUE)
```
The average number of steps taken on any day are 9354,  where the median number of step are 10395.



## What is the average daily activity pattern?

Here we calculate the average number of steps for all intervel, averaged across the dates and plot the data. 


```r
average_steps_by_interval <- aggregate(dat$steps, by = list(dat$interval), FUN=mean, na.rm=TRUE)
names(average_steps_by_interval) <- c("Intereval", "Average steps")
plot(average_steps_by_interval$Intereval,average_steps_by_interval$`Average steps`, type = "l", xlab = "Interval", ylab = "Average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The intervel that contains the most number of steps is the interval number 
104 and the associated number of steps are 206.1698113.

## Imputing missing values



```r
num_of_missing_val <- length(dat[is.na(dat$steps),1])
```

There are total 2304 NA values in the data set. We can fill the NA's by using their interval number. We calculated the intervel averge across dates. We will replace NA with the average step assiciated with the respective interval id.   
We create a copy of the orignal data set named **dat_no_na**


```r
dat_no_na <- dat
na_intervals <- dat[is.na(dat$steps),3]
dat_no_na[is.na(dat_no_na)] <- average_steps_by_interval[match(na_intervals,average_steps_by_interval$Intereval), 'Average steps']

#
total_steps_no_na <- aggregate(dat_no_na$steps, by = list(dat_no_na$date), FUN=sum)
names(total_steps_no_na) <- c("Date", "Total steps")
hist(total_steps_no_na$`Total steps`, xlab = "Steps", main = "Steps taken in a day",breaks = 10)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
step_mean_nona <- mean(total_steps_no_na$`Total steps`)
step_median_nona <- median(total_steps_no_na$`Total steps`)
```

To compare the effect from removing of NA's, we will plot both histograms overlaed. 


```r
df_1 <- total_steps_by_date
df_2 <- total_steps_no_na
df_1$isna <- "nA"
df_2$isna <- "no_nA"
df <- rbind(df_1,df_2)
library(ggplot2)
ggplot(df, aes(x=`Total steps`, color=isna)) +geom_histogram(fill="white", position="dodge")+theme(legend.position="top")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

Which show that the presence of NA were resulting in a high frequency around zero. When NAs were removed, a much higher frequency appeared around 11000 steps. 

## Weekend vs weekday activity

In the following we analyse any differencesin activity between weekday and weekends.  


```r
wd_df <- data.frame(Day = weekdays(as.Date(c(1:7), origin="1899-12-31")))
cc <- factor(c("weekday", "weekday","weekday","weekday","weekday","weekend", "weekend"))
wd_df$Isweekday <- cc

dat_no_na$day <-  wd_df[match(weekdays(dat$date), wd_df$Day),2]

split_dat_no_na <- split(dat_no_na, dat_no_na$day)
mean_by_interval_weekday <- aggregate(split_dat_no_na$weekday$steps, by=list(split_dat_no_na$weekday$interval), FUN = mean)
names(mean_by_interval_weekday) <- c("Intervels", "Mean_steps")
mean_by_interval_weekday$day <- cc[1]
mean_by_interval_weekend <- aggregate(split_dat_no_na$weekend$steps, by=list(split_dat_no_na$weekend$interval), FUN = mean)
names(mean_by_interval_weekend) <- c("Intervels", "Mean_steps")
mean_by_interval_weekend$day <- cc[7]
mean_by_interval_comb <- rbind(mean_by_interval_weekday, mean_by_interval_weekend)
ggplot(mean_by_interval_comb, aes(Intervels, Mean_steps)) + geom_line() + facet_grid(day ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


From plots it appear that there is higher average number of steps in the 500 to 1000 window during weekdays. Weekends has more evenly distributed steps across the intervels. 

To make an exact comparision we also plot them on the same plot.


```r
ggplot(mean_by_interval_comb, aes(Intervels, Mean_steps, color = day)) + geom_point() 
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

