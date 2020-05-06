==========================================  
Created by twestr on the 6th of May 2020

### Basic settings

    echo = TRUE  # Always make code visible
    knitr::opts_chunk$set(fig.path="figure/") # Creating directory for plots (for md-file)

**\_\_\_\_\_\_\_\_**

### Loading and processing the data

    unzip("activity.zip")
    data <- read.csv("activity.csv", colClasses = c("integer", "Date", "factor"))
    data$month <- as.numeric(format(data$date, "%m"))
    no_NA <- na.omit(data)
    rownames(no_NA) <- 1:nrow(no_NA)
    head(no_NA)

    ##   steps       date interval month
    ## 1     0 2012-10-02        0    10
    ## 2     0 2012-10-02        5    10
    ## 3     0 2012-10-02       10    10
    ## 4     0 2012-10-02       15    10
    ## 5     0 2012-10-02       20    10
    ## 6     0 2012-10-02       25    10

    dim(no_NA)

    ## [1] 15264     4

**\_\_\_\_\_\_\_\_**

### What is mean total number of steps taken per day?

For this part you can ignore the missing values in the dataset.

**I. Make a histogram of the total number of steps taken per day:**

    library(ggplot2)
    ggplot(no_NA, aes(date, steps)) + geom_bar(stat = "identity", colour = "blue", fill = "blue", width = 0.7) + facet_grid(. ~ month, scales = "free") + labs(title = "Histogram of the total number of steps taken each day", x = "Date", y = "Total number of steps")

![](figure/unnamed-chunk-3-1.png)

**II. Calculate and report the mean and median number of steps taken per
day**

**II.1 Mean number of steps taken per day:**

    total_steps <- aggregate(no_NA$steps, list(Date = no_NA$date), "sum")$x
    mean(total_steps)

    ## [1] 10766.19

**II.2 Median number of steps taken per day:**

    median(total_steps)

    ## [1] 10765

**\_\_\_\_\_\_\_\_**

### What is the average daily activity pattern?

**I. Make a time series plot (i.e. type = “l”) of the 5-minute interval
(x-axis) and the average number of steps taken, averaged across all days
(y-axis):**

    avg_steps <- aggregate(no_NA$steps, list(interval = as.numeric(as.character(no_NA$interval))), "mean")
    names(avg_steps)[2] <- "MeanOfSteps"

    ggplot(avg_steps, aes(interval, MeanOfSteps)) + geom_line(color = "blue", size = 0.8) + labs(title = "Time series plot of the average number of steps taken", x = "5-minute intervals", y = "Average number of steps taken")

![](figure/unnamed-chunk-6-1.png)

**II. Which 5-minute interval, on average across all the days in the
dataset, contains the maximum number of steps?**

    avg_steps[avg_steps$MeanOfSteps == max(avg_steps$MeanOfSteps), ]

    ##     interval MeanOfSteps
    ## 104      835    206.1698

**\_\_\_\_\_\_\_\_**

### Imputing missing values

**I. The total number of rows with NAs:**

    sum(is.na(data))

    ## [1] 2304

**II. Devise a strategy for filling in all of the missing values in the
dataset. The strategy does not need to be sophisticated. For example,
you could use the mean/median for that day, or the mean for that
5-minute interval, etc.**

–&gt; My strategy is to use the mean for that 5-minute interval to fill
each NA value in the steps column.

**III. Create a new dataset that is equal to the original dataset but
with the missing data filled in:**

    new_data <- data 
    for (i in 1:nrow(new_data)) {
            if (is.na(new_data$steps[i])) {
                    new_data$steps[i] <- avg_steps[which(new_data$interval[i] == avg_steps$interval), ]$MeanOfSteps
            }
    }

    head(new_data)

    ##       steps       date interval month
    ## 1 1.7169811 2012-10-01        0    10
    ## 2 0.3396226 2012-10-01        5    10
    ## 3 0.1320755 2012-10-01       10    10
    ## 4 0.1509434 2012-10-01       15    10
    ## 5 0.0754717 2012-10-01       20    10
    ## 6 2.0943396 2012-10-01       25    10

    sum(is.na(new_data))

    ## [1] 0

**IV. Make a histogram of the total number of steps taken each day after
missing values are imputed:**

    ggplot(new_data, aes(date, steps)) + geom_bar(stat = "identity",
                                                 colour = "blue",
                                                 fill = "blue",
                                                 width = 0.7) + facet_grid(. ~ month, scales = "free") + labs(title = "Total number of steps taken each day (no missing data)", x = "Date", y = "Total number of steps")

![](figure/unnamed-chunk-10-1.png)

**V. Do these values differ from the estimates from the first part of
the assignment? What is the impact of imputing missing data on the
estimates of the total daily number of steps?**

**V.1 Mean total number of steps taken per day:**

    new_total_steps <- aggregate(new_data$steps, 
                               list(Date = new_data$date), 
                               "sum")$x
    new_mean <- mean(new_total_steps)
    new_mean

    ## [1] 10766.19

**V.2 Median total number of steps taken per day:**

    new_median <- median(new_total_steps)
    new_median

    ## [1] 10766.19

**V.3 Compare them with the two before imputing missing data:**

    old_mean <- mean(total_steps)
    old_median <- median(total_steps)
    new_mean - old_mean

    ## [1] 0

    new_median - old_median

    ## [1] 1.188679

–&gt; After imputing the missing data, the new mean of total steps taken
per day is the same as that of the old mean.  
The new median of total steps taken per day is greater than that of the
old median.  
**\_\_\_\_\_\_\_\_**

### Are there differences in activity patterns between weekdays and weekends?

**I. Create a new factor variable in the dataset with two levels.
“Weekday” and “weekend” indicating whether a given date is a weekday or
weekend day:**

    head(new_data)

    ##       steps       date interval month
    ## 1 1.7169811 2012-10-01        0    10
    ## 2 0.3396226 2012-10-01        5    10
    ## 3 0.1320755 2012-10-01       10    10
    ## 4 0.1509434 2012-10-01       15    10
    ## 5 0.0754717 2012-10-01       20    10
    ## 6 2.0943396 2012-10-01       25    10

    new_data$weekdays <- factor(format(new_data$date, "%A"))
    levels(new_data$weekdays)

    ## [1] "Dienstag"   "Donnerstag" "Freitag"    "Mittwoch"   "Montag"    
    ## [6] "Samstag"    "Sonntag"

    levels(new_data$weekdays) <- list(Weekday = c("Montag", "Dienstag", "Mittwoch","Donnerstag", "Freitag"), Weekend = c("Samstag", "Sonntag"))
    levels(new_data$weekdays)

    ## [1] "Weekday" "Weekend"

    table(new_data$weekdays)

    ## 
    ## Weekday Weekend 
    ##   12960    4608

**II. Make a panel plot containing a time series plot (i.e. type = “l”)
of the 5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis):**

    avg_steps <- aggregate(new_data$steps, 
                          list(interval = as.numeric(as.character(new_data$interval)), 
                               weekdays = new_data$weekdays),"mean")
    names(avg_steps)[3] <- "MeanOfSteps"

    library(lattice)
    xyplot(avg_steps$MeanOfSteps ~ avg_steps$interval | avg_steps$weekdays, layout = c(1,2), type = "l", xlab = "Interval", ylab = "Number of steps")

![](figure/unnamed-chunk-15-1.png)
