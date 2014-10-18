# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
data <- read.csv("activity.csv", header=T)
```



## What is mean total number of steps taken per day?
so lets plot a histogram of the # of steps/day and take a summary:


```r
agg_date <- aggregate(steps ~ date, data=data, FUN=sum)
hist(agg_date$steps, breaks=10, col="Red", main="", xlab="# steps/day")
```

![](./PA1_template_files/figure-html/meansteps-1.png) 

```r
summary(agg_date)
```

```
##          date        steps      
##  2012-10-02: 1   Min.   :   41  
##  2012-10-03: 1   1st Qu.: 8841  
##  2012-10-04: 1   Median :10765  
##  2012-10-05: 1   Mean   :10766  
##  2012-10-06: 1   3rd Qu.:13294  
##  2012-10-07: 1   Max.   :21194  
##  (Other)   :47
```
the summary reports mean = 10766 and median = 10765



## What is the average daily activity pattern?

```r
agg_int <- aggregate(steps ~ interval, data=data, FUN=mean)
summary(agg_int)
```

```
##     interval          steps        
##  Min.   :   0.0   Min.   :  0.000  
##  1st Qu.: 588.8   1st Qu.:  2.486  
##  Median :1177.5   Median : 34.113  
##  Mean   :1177.5   Mean   : 37.383  
##  3rd Qu.:1766.2   3rd Qu.: 52.835  
##  Max.   :2355.0   Max.   :206.170
```

```r
plot(agg_int, type="l", main="Average Daily Activity Pattern", xlab="5-min interval", ylab="avg # steps/day")
```

![](./PA1_template_files/figure-html/avgactivity-1.png) 

The interval with the maximum # of steps was found by:

```r
mx <- max(agg_int$steps)
agg_int[agg_int$steps==mx,]
```

```
##     interval    steps
## 104      835 206.1698
```
indicating the average max # steps of 206.17 occurred at 8:35 am


## Imputing missing values

```r
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

From the summary above, we find 2,304 rows had NA entries for steps.

To fill these data holes, I used the mean # of steps/5-min interval (agg_int) data frame

Created a new column (wd_we) and converted the date to a weekday or weekend factor and saved it to a copy of the original data, called data_imp:


```r
#data_imp$day <- weekdays(as.Date(as.character(data_imp$date))) #tried this 1st; no workie
data_imp$wd_we <- as.factor(ifelse(!weekdays(as.Date(data_imp$date)) %in% c("Saturday", "Sunday"), "weekday","weekend"))
```

then I merged this new data frame with the agg_int data and filled all the NAs with the mean # of steps for that 5-min interval


```r
df_merge <- merge(data_imp, agg_int, by.x="interval", by.y="interval")
df_merge$steps.x[is.na(df_merge$steps.x)] <- df_merge$steps.y
```

```
## Warning in df_merge$steps.x[is.na(df_merge$steps.x)] <- df_merge$steps.y:
## number of items to replace is not a multiple of replacement length
```


## Are there differences in activity patterns between weekdays and weekends?


create a panel plot showing a time series for each set of data:


```r
library(lattice)
#xyplot(steps ~ interval | wd_we, data=data_imp, layout=c(2,1), type="l")
xyplot(steps.x ~ interval | wd_we, data=df_merge, layout=c(2,1), type="l")
```

![](./PA1_template_files/figure-html/panelplot-1.png) 


Looking at the panel plots, there are distinct differences between the measurements made during the weekday and those of the weekend. Step activity is significantly more in mornings after 5am where on the weekend, step activity doesn't really begin until after 8-9am. The data also shows a common burst of activity that begins around 3pm for both data sets. However, the weekend data shows that step activity maintained for another 2 hrs until 5pm



