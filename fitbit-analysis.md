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

## minute-level data

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
  rm(minuteCaloriesNarrow,minuteIntensitiesNarrow,minuteMETsNarrow,minuteStepsNarrow)
```

#### generate dateTime and date variables with common naming convention 


```r
  minute$dateTime<-lubridate::mdy_hms(minute$ActivityMinute)
  minute$date<-as.Date(minute$dateTime)
  min(minute$date)
```

```
## [1] "2021-11-08"
```

```r
  max(minute$date)
```

```
## [1] "2021-12-12"
```

### clean and explore minute-level sleep data

#### issue: sleep observations can be measured on the half or full minute


```r
# sleep observations can be measured on the half or full minute (one or the other within a given bout of sleep)
  minuteSleep$test1<-ifelse(grepl(":30 ",minuteSleep$date),":30",":00")
  table(minuteSleep$test1)
```

```
## 
##    :00    :30 
## 621898 456049
```

```r
# all other minute-level data is measured on the full minute
  minute$test1<-ifelse(grepl(":30 ",minute$dateTime),":30",":00")
  table(minute$test1)
```

```
## 
##     :00 
## 3477600
```

#### solution: update the date all observations fall on the full minute (to align with other minute-level data)


```r
# since sleep is either measured at :00 or :30 within a bout of sleep,
# we can update all records to ":00 " without introducing any duplicates
# this is necessary to merge sleep records with the other minute-level data (always measured on full minute)
  minuteSleep$date<-gsub(":30 ",":00 ",minuteSleep$date)
  minuteSleep$dateTime<-lubridate::mdy_hms(minuteSleep$date)
  minuteSleep$date<-as.Date(minuteSleep$dateTime)
  minuteSleep$test2<-ifelse(grepl(":30 ",minuteSleep$date),":30",":00")
  table(minuteSleep$test2)
```

```
## 
##     :00 
## 1077947
```


#### filter minute-level sleep data to to same time range as other data types


```r
  min(minuteSleep$date) # data from 2021-11-07 but other data types start 2021-11-08
```

```
## [1] "2021-11-07"
```

```r
  max(minuteSleep$date) # data from 2021-12-13, but other data types end 2021-12-12
```

```
## [1] "2021-12-13"
```

```r
  minuteSleep<-minuteSleep[minuteSleep$date>"2021-11-08" & minuteSleep$date<"2021-12-13",]
```


#### issue: there are duplicate sleep records for most participants


```r
  test<-minuteSleep %>% select(Id,dateTime) %>% group_by(Id,dateTime) %>% mutate(length=length(Id))
  table(test$length) # this is unrelated to updating the date/time from :30 to :00 above
```

```
## 
##      1      2 
## 911957 113806
```

```r
  length(unique(test[test$length==2,]$Id)) # almost every participant has duplicate records for sleep
```

```
## [1] 66
```

#### note: all but 6 observations (3 minutes for 1 participants) are exact duplicates (same Id, date AND value)


```r
  test3<-minuteSleep %>% select(Id,dateTime,value) %>% group_by(Id,dateTime) %>% 
                                                       mutate(length=length(Id)) %>%
                                                       mutate(unique=length(unique(value)))

# however, only 6 records (3 minutes for 1 participant) have more than one unique "value" within the same minute
# the rest of the duplicate rows are exact duplicates (same "value" (sleep stage) across both observations)
  table(test3[test3$length==2,]$unique) 
```

```
## 
##      1      2 
## 113800      6
```

```r
  rm(test,test3)
```


#### solution: remove duplicate records within the minute-level sleep data


```r
# create de-duplicated version of the minuteSleep dataframe
# need to decide how to handle the 3 minutes with two unique "values"
  minuteSleep<-minuteSleep %>% select(-test1,-test2)
  minuteSleep<-distinct(minuteSleep) 
  test4<-minuteSleep %>% select(Id,dateTime,value) %>% group_by(Id,dateTime) %>% 
                                                       mutate(length=length(Id)) %>%
                                                       mutate(unique=length(unique(value)))
  table(test4$length)
```

```
## 
##      1      2 
## 968427    866
```

```r
# 1 participant has 800+ rows that are duplicates but the logId differs
  minuteSleep<-distinct(minuteSleep, across(-"logId"),.keep_all = TRUE) 
  test4<-minuteSleep %>% select(Id,dateTime,value) %>% group_by(Id,dateTime) %>% 
                                                       mutate(length=length(Id)) %>%
                                                       mutate(unique=length(unique(value)))
  table(test4$length)
```

```
## 
##      1      2 
## 968857      6
```

```r
  rm(test4)
  
# decide what to do with duplicate records with differing "values"
```


### merge minute-level sleep with other minute-level data


```r
  minute <- minute %>% full_join(minuteSleep, by = c("dateTime","Id"))
  rm(minuteSleep)
```

## day-level data

### explore day-level data

#### day-level calories


```r
  str(dailyCalories)
```

```
## 'data.frame':	2415 obs. of  3 variables:
##  $ Id         : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityDay: chr  "11/8/2021" "11/9/2021" "11/10/2021" "11/11/2021" ...
##  $ Calories   : int  2362 2203 2047 2294 2353 2416 3083 1856 2185 2329 ...
```

#### day-level intensities


```r
  str(dailyIntensities)
```

```
## 'data.frame':	2415 obs. of  10 variables:
##  $ Id                      : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityDay             : chr  "11/8/2021" "11/9/2021" "11/10/2021" "11/11/2021" ...
##  $ SedentaryMinutes        : int  216 760 509 663 1174 513 977 653 688 731 ...
##  $ LightlyActiveMinutes    : int  183 185 171 200 232 273 322 83 146 151 ...
##  $ FairlyActiveMinutes     : int  30 9 0 17 11 10 83 8 13 26 ...
##  $ VeryActiveMinutes       : int  19 14 0 30 23 8 56 13 26 40 ...
##  $ SedentaryActiveDistance : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ LightActiveDistance     : num  3.26 2.84 2.83 3.09 3.02 ...
##  $ ModeratelyActiveDistance: num  0.98 0.32 0 0.79 0.48 ...
##  $ VeryActiveDistance      : num  1.39 1.05 0 2.24 1.78 ...
```

#### day-level steps


```r
  str(dailySteps)
```

```
## 'data.frame':	2415 obs. of  3 variables:
##  $ Id         : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityDay: chr  "11/8/2021" "11/9/2021" "11/10/2021" "11/11/2021" ...
##  $ StepTotal  : int  7893 5900 3978 8597 7421 8203 12910 3927 6836 9088 ...
```

#### day-level sleep 


```r
  str(sleepDay)
```

```
## 'data.frame':	2166 obs. of  5 variables:
##  $ Id                : chr  "A01" "A01" "A01" "A01" ...
##  $ SleepDay          : chr  "11/8/2021 12:00:00 AM" "11/9/2021 12:00:00 AM" "11/10/2021 12:00:00 AM" "11/11/2021 12:00:00 AM" ...
##  $ TotalSleepRecords : int  1 1 1 2 1 1 1 1 1 2 ...
##  $ TotalMinutesAsleep: int  446 450 708 492 629 518 631 450 480 376 ...
##  $ TotalTimeInBed    : int  496 472 760 530 636 597 655 492 518 409 ...
```

#### day-level sleep stages


```r
  str(sleepStagesDay) # same variables as above (Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) plus total minutes awake and spent in each sleep stage (light/deep/REM)
```

```
## 'data.frame':	2415 obs. of  9 variables:
##  $ Id                : chr  "A01" "A01" "A01" "A01" ...
##  $ SleepDay          : chr  "11/8/2021 12:00:00 AM" "11/9/2021 12:00:00 AM" "11/10/2021 12:00:00 AM" "11/11/2021 12:00:00 AM" ...
##  $ TotalSleepRecords : int  1 1 1 2 0 1 0 1 1 1 ...
##  $ TotalMinutesAsleep: int  446 422 615 492 0 629 0 518 631 371 ...
##  $ TotalTimeInBed    : int  496 472 760 530 0 636 0 597 655 492 ...
##  $ TotalTimeAwake    : int  0 50 145 0 0 0 0 0 0 121 ...
##  $ TotalMinutesLight : int  0 269 382 0 0 0 0 0 0 241 ...
##  $ TotalMinutesDeep  : int  0 42 99 0 0 0 0 0 0 66 ...
##  $ TotalMinutesREM   : int  0 111 134 0 0 0 0 0 0 64 ...
```

### merge day-level activity 


```r
  daily<-dailyCalories %>% full_join(dailyIntensities, by = c("ActivityDay","Id")) %>%
                           full_join(dailySteps, by = c("ActivityDay","Id")) 
  rm(dailyCalories,dailyIntensities,dailySteps)
```




