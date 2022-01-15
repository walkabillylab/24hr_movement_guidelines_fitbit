---
title: "fitbit analysis for CSEP project"
author: "Shelby Sturrock"
date: "14/01/2022"
output: 
  html_document:
    keep_md: true
---



## processing FitBit data 

### read in data from three time periods: 


```r
  # nov 08 - nov 21
    nov08<-list.files("fitbit_data/nov08-nov21", pattern="*.csv", full.names=TRUE)
    nov08_csv<-lapply(nov08, read.csv)
    names(nov08_csv)<-paste(gsub("fitbit_data/nov08-nov21/|.csv","",nov08),".1",sep="")
    list2env(nov08_csv,envir=.GlobalEnv)
    rm(nov08,nov08_csv)
    
  # nov 22 - dec 05
    nov22<-list.files("fitbit_data/nov22-dec05", pattern="*.csv", full.names=TRUE)
    nov22_csv<-lapply(nov22, read.csv)
    names(nov22_csv)<-paste(gsub("fitbit_data/nov22-dec05/|.csv","",nov22),".2",sep="")
    list2env(nov22_csv,envir=.GlobalEnv)
    rm(nov22,nov22_csv)
  
  # dec 06 - dec 12
    dec06<-list.files("fitbit_data/dec06-dec12", pattern="*.csv", full.names=TRUE)
    dec06_csv<-lapply(dec06, read.csv)
    names(dec06_csv)<-paste(gsub("fitbit_data/dec06-dec12/|.csv","",dec06),".3",sep="")
    list2env(dec06_csv,envir=.GlobalEnv)
    rm(dec06,dec06_csv)
```

### for each unique data type/dataset, consolidate data from three time periods


```r
# generate list of unique datasets (there are 18)
    files<-unique(gsub("\\.1|\\.2|\\.3","",ls()))
    
  # for each unique type of data, rbind data from the three periods together
    for(i in 1:length(files)) {
      
      pattern<-files[i]
      X<-mget(ls(pattern=pattern)) %>% bind_rows()
      assign(gsub("_merged","",pattern),X)
      rm(list = ls()[grepl(pattern, ls())])
      rm(X)
      i<-i+1
      
    }
    
  # clean environment
    rm(i,pattern,files)
```

#### leaves a total of 18 unique dataframes


```r
    ls()
```

```
##  [1] "30secondSleepStages"     "activitylogs"           
##  [3] "battery"                 "dailyCalories"          
##  [5] "dailyIntensities"        "dailySteps"             
##  [7] "heartrate_1min"          "heartRateZones"         
##  [9] "minuteCaloriesNarrow"    "minuteIntensitiesNarrow"
## [11] "minuteMETsNarrow"        "minuteSleep"            
## [13] "minuteStepsNarrow"       "sleepDay"               
## [15] "sleepLogInfo"            "sleepStageLogInfo"      
## [17] "sleepStagesDay"          "syncEvents"
```

### explore minute-level data (note date/time variable is "ActivityMinute" in most but not all dataframes)

#### minute-level step count


```r
      str(minuteStepsNarrow) # steps per minute
```

```
## 'data.frame':	3477600 obs. of  3 variables:
##  $ Id            : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityMinute: chr  "11/8/2021 12:00:00 AM" "11/8/2021 12:01:00 AM" "11/8/2021 12:02:00 AM" "11/8/2021 12:03:00 AM" ...
##  $ Steps         : int  0 0 0 0 0 0 0 0 0 0 ...
```

#### minute-level calories


```r
      str(minuteCaloriesNarrow) # calories burned each minute
```

```
## 'data.frame':	3477600 obs. of  3 variables:
##  $ Id            : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityMinute: chr  "11/8/2021 12:00:00 AM" "11/8/2021 12:01:00 AM" "11/8/2021 12:02:00 AM" "11/8/2021 12:03:00 AM" ...
##  $ Calories      : num  1.03 1.03 1.03 1.03 1.03 ...
```

#### minute-level activity intensity


```r
      str(minuteIntensitiesNarrow) # intensity of activity during each minute -- 0, 1, 2 and 3
```

```
## 'data.frame':	3477600 obs. of  3 variables:
##  $ Id            : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityMinute: chr  "11/8/2021 12:00:00 AM" "11/8/2021 12:01:00 AM" "11/8/2021 12:02:00 AM" "11/8/2021 12:03:00 AM" ...
##  $ Intensity     : int  0 0 0 0 0 0 0 0 0 0 ...
```

#### minute-level METs


```r
      str(minuteMETsNarrow) # MET value measued each minute
```

```
## 'data.frame':	3477600 obs. of  3 variables:
##  $ Id            : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityMinute: chr  "11/8/2021 12:00:00 AM" "11/8/2021 12:01:00 AM" "11/8/2021 12:02:00 AM" "11/8/2021 12:03:00 AM" ...
##  $ METs          : int  10 10 10 10 10 10 10 10 10 10 ...
```

#### minute-level heart rate (fewer observations than above)


```r
      str(heartrate_1min) # heart rate measured each minute -- fewer measurements than above
```

```
## 'data.frame':	3116108 obs. of  3 variables:
##  $ Id   : chr  "A01" "A01" "A01" "A01" ...
##  $ Time : chr  "11/8/2021 12:00:00 AM" "11/8/2021 12:01:00 AM" "11/8/2021 12:02:00 AM" "11/8/2021 12:03:00 AM" ...
##  $ Value: int  64 67 65 65 63 67 62 62 72 66 ...
```

#### minute-level sleep stages 


```r
      str(minuteSleep) # sleep stage per minute?
```

```
## 'data.frame':	1077947 obs. of  4 variables:
##  $ Id   : chr  "A01" "A01" "A01" "A01" ...
##  $ date : chr  "11/8/2021 12:16:00 AM" "11/8/2021 12:17:00 AM" "11/8/2021 12:18:00 AM" "11/8/2021 12:19:00 AM" ...
##  $ value: int  1 1 2 1 2 2 2 2 2 2 ...
##  $ logId: num  3.45e+10 3.45e+10 3.45e+10 3.45e+10 3.45e+10 ...
```

#### 30 second sleep stages


```r
      str(`30secondSleepStages`) # sleep stages per 30 seconds? -- also fewer sleep observations
```

```
## 'data.frame':	1781368 obs. of  6 variables:
##  $ Id        : chr  "A01" "A01" "A01" "A01" ...
##  $ LogId     : num  3.45e+10 3.45e+10 3.45e+10 3.45e+10 3.45e+10 ...
##  $ Time      : chr  "11/9/2021 12:57:30 AM" "11/9/2021 12:58:00 AM" "11/9/2021 12:58:30 AM" "11/9/2021 12:59:00 AM" ...
##  $ Level     : chr  "wake" "wake" "wake" "wake" ...
##  $ ShortWakes: chr  "" "" "" "" ...
##  $ SleepStage: chr  "wake" "wake" "wake" "wake" ...
```


### process minute-level data

#### merge minute-level calories, intensities, METs and steps


```r
    minute <- minuteCaloriesNarrow %>% full_join(minuteIntensitiesNarrow, by = c("ActivityMinute","Id")) %>%
                                       full_join(minuteMETsNarrow, by = c("ActivityMinute","Id")) %>%
                                       full_join(minuteStepsNarrow, by = c("ActivityMinute","Id")) 
```

#### generate dateTime and date variables with common naming convention 


```r
    minute$dateTime<-lubridate::mdy_hms(minute$ActivityMinute)
    minute$date<-as.Date(minute$dateTime)
```



