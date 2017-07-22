### This project is regarding

**Reproducible Research** course in Coursera's Data Science using
[FitBit](http://en.wikipedia.org/wiki/Fitbit) Data.

Data and variable definitions
-----------------------------

-   Dataset: [Activity monitoring
    data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
    \[52K\]

-   **steps**: Number of steps taking in a 5-minute interval  
-   **date**: The date on which the measurement was taken i
-   **interval**: Identifier for the 5-minute interval in which
    measurement was taken

Download file, UNZIP and Read content into data
-----------------------------------------------

    if(!file.exists("getdata-projectfiles-UCI HAR Dataset.zip")) {
            temp <- tempfile()
            download.file("http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
            unzip(temp)
            unlink(temp)
    }

    data <- read.csv("activity.csv")

Calculate mean total step taken for a day - aggregate steps by day
------------------------------------------------------------------

Generate Histogram, and determine mean and median.

    day_steps <- aggregate(steps ~ date, data, sum)
    hist(day_steps$steps, main = paste("Total Steps Each Day"), col="blue", xlab="Number of Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    rmean <- mean(day_steps$steps)
    rmedian <- median(day_steps$steps)

The `mean` is 1.076618910^{4} and the `median` is 10765.

Average daily activity pattern - Determine steps for each interval and Plot the average steps
---------------------------------------------------------------------------------------------

-   Find interval with most average steps.

<!-- -->

    interval_steps <- aggregate(steps ~ interval, data, mean)

    plot(interval_steps$interval,interval_steps$steps, type="l", xlab="Interval", ylab="Number of Steps",main="Average Number of Steps per Day by Interval")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

    max_interval <- interval_steps[which.max(interval_steps$steps),1]

The 5-minute interval, the maximum number of steps is 835.

Impute missing values. Assign average for each day for the missing data
-----------------------------------------------------------------------

    incomplete <- sum(!complete.cases(data))
    imputed_data <- transform(data, steps = ifelse(is.na(data$steps), interval_steps$steps[match(data$interval, interval_steps$interval)], data$steps))

For the first day 10-01-2012 zeros were imputed

    imputed_data[as.character(imputed_data$date) == "2012-10-01", 1] <- 0

Aggregate again after the imputed data for total steps by day and create
Histogram.

    day_steps_i <- aggregate(steps ~ date, imputed_data, sum)
    hist(day_steps_i$steps, main = paste("Total Steps Each Day"), col="blue", xlab="Number of Steps")

    #Create Histogram to show difference. 
    hist(day_steps$steps, main = paste("Total Steps Each Day"), col="red", xlab="Number of Steps", add=T)
    legend("topright", c("Imputed", "Non-imputed"), col=c("blue", "red"), lwd=10)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Determine new mean and median for imputed data.

    rmean.i <- mean(day_steps_i$steps)
    rmedian.i <- median(day_steps_i$steps)

Find the difference between imputed and non-imputed data.

    mean_diff <- rmean.i - rmean
    med_diff <- rmedian.i - rmedian

Determine the total difference.

    total_diff <- sum(day_steps_i$steps) - sum(day_steps$steps)

-   The new data mean after imputed is 1.058969410^{4}
-   The new median after imputed is 1.076618910^{4}
-   The difference in mean is -176.4948964
-   The difference in meadiun is 1.1886792
-   The difference in total number of steps between imputed and
    non-imputed data is 7.536332110^{4}.

Differenes in activity patterns between weekdays and weekends
-------------------------------------------------------------

Generate a plot to compare steps between the week and weekend. There is
a higher peak earlier on weekdays, and more overall activity on
weekends.

    weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", 
                  "Friday")
    imputed_data$dow = as.factor(ifelse(is.element(weekdays(as.Date(imputed_data$date)),weekdays), "Weekday", "Weekend"))

    interval_steps_i <- aggregate(steps ~ interval + dow, imputed_data, mean)

    library(lattice)

    xyplot(interval_steps_i$steps ~ interval_steps_i$interval|interval_steps_i$dow, main="Average Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)
