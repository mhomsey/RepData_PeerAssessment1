Reproducible Research: Peer Assessment 1
========================================
## Loading and preprocessing the data  

The code is downloaded from via a zipped URL and contains a CSV file:

### function to retrieve raw zip file

The zip file is loaded into a subdirectory data. The directory is created if it 
doesn't already exists.  
The zip file is saved as "monitoing.zip".  

```r
getdatafile <- function() {
        if(!file.exists("./data")){dir.create("./data")}
        fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
        download.file(fileURL, destfile="./data/monitoring.zip",method="curl")
}
```
### function to extract required lines from zip file/file in zip file

The csv file is referenced and read through an unzip function.  
The file is not unzipped.  
The data.frame is returned to the caller.  

```r
readfromzip <- function() {
# extract the names line
        data <- read.csv2(unz("./data/monitoring.zip", "activity.csv"), header=T,  stringsAsFactors=F, sep=",")
  data
}
```

### download and read in the data using these functions:  


```r
getdatafile()
activity <- readfromzip()
```


## What is the average daily activity pattern?  
Calculate the sum per day of each of the steps taken. (Give the column names a suitable name.)

```r
sumbyDay <- aggregate(activity$steps, list(activity$date), sum)
names(sumbyDay) <- c("date", "totalSteps")
```
Plot the histogram of the steps taken.

```r
hist(sumbyDay$totalSteps, breaks=30, main="Histogram of the total number of steps taken each day", xlab="Total Steps", col="lightgreen", lwd=2)
```

![plot of chunk histo1](figure/histo1-1.png) 

Now calculate the meand and median, ignoring the days with NA values.

```r
meanIgnoreNA <- round(mean(sumbyDay$totalSteps, na.rm=T))
medianIgnoreNA <- median(sumbyDay$totalSteps, na.rm=T)
```

The Mean value of the steps per day is 1.0766 &times; 10<sup>4</sup> and the median value is 10765.  

## Imputing missing values  

1. Total number of NAs  

```r
numNA <- sum(is.na(activity$steps))
numRows <- nrow(activity)
```
There is a 2304 rows with NA values out of a total of 17568 rows in total.

2. NA replacement strategy.  
There are complete days which are filled with NA values.  
The days with NA values are to be replaced with the mean value of the whole dataset.  First make a copy of the data set by calculating the sum per month first and then replace the NA values.  
3. create the new dataset  
There are only NAs in the steps column, none in the interval or date columns

```r
sumbyDay2 <- aggregate(activity$steps, list(activity$date), sum)
names(sumbyDay2) <- c("date", "totalSteps")
sumbyDay2[is.na(sumbyDay2)] <- meanIgnoreNA
```
sumbyDay2 now contains the same as sumbyDay with the NA months replaced by the mean.  

4. new histogram and mean and median created.


```r
hist(sumbyDay2$totalSteps, breaks=30, main="Histogram of the total number of steps taken each day modified NAs", xlab="Total Steps", col="red", lwd=2)
```

![plot of chunk newhist](figure/newhist-1.png) 

Calculate the new mean and median:

```r
mean2 <- round(mean(sumbyDay2$totalSteps, na.rm=T))
median2 <- median(sumbyDay2$totalSteps, na.rm=T)
```
The Mean value of the steps per day is 1.0766 &times; 10<sup>4</sup> and the median value is 1.0766 &times; 10<sup>4</sup>.  

The histogram has changed.  The replacement of the NA values with the average has increased the number of days with the average value. See the column has shifted from 10 to 15.  
The average is unaffected. The median *has* been affected. More points are in the middle, so it has shifted from **10765** to **10766**.


## Are there differences in activity patterns between weekdays and weekends?  
1. new factor varibale "weekday" and "weekend".  
create a new column with the name of the week (using *weekdays*) and use gsub with regular expressions to change this to **weekend** or **weekday**.

```r
activity2 <- activity
activity2$weekday <- weekdays(as.Date(activity2$date))
activity2$weekday <- gsub("Monday|Tuesday|Wednesday|Thursday|Friday",
                         "weekday", activity2$weekday)

activity2$weekday <- gsub("Saturday|Sunday", "weekend", activity2$weekday)
```



