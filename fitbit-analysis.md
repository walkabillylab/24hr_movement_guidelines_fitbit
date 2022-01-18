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

#### merge minute-level heart rate data to other minute-level activity data


```r
  heartrate_1min$ActivityMinute<-heartrate_1min$Time
  heartrate_1min<-heartrate_1min %>% select(-Time)
  minute<-merge(minute,heartrate_1min,by=c("ActivityMinute","Id"),all.x=TRUE)
  rm(heartrate_1min)
```


#### generate dateTime and date variables with common naming convention 


```r
  minute$dateTime<-lubridate::mdy_hms(minute$ActivityMinute)
  minute$date<-as.Date(minute$dateTime)
  minute<-minute %>% select(-ActivityMinute)
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

```r
  minute<-minute %>% select(-test1)
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
  minute <- minute %>% full_join(minuteSleep, by = c("dateTime","Id","date"))
  rm(minuteSleep)
```

#### rename columns in minute-level data (so it is clear which columns are minute- vs. day-level)


```r
  minute<-minute %>% rename(minuteCalories=Calories,
                            minuteHeartRate=Value,
                            minuteIntensity=Intensity,
                            minuteMETs=METs,
                            minuteSteps=Steps,
                            minuteSleepStage=value,
                            minuteSleepLogId=logId) %>%
                     relocate(dateTime,.after=Id) %>%
                     relocate(date,.after=dateTime) %>%
                     arrange(Id,dateTime)
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

```r
  # will use sleepStagesDay instead of sleepDay because it contains all of the same and more information
  rm(sleepDay)
```

### merge day-level activity data


```r
  daily<-dailyCalories %>% full_join(dailyIntensities, by = c("ActivityDay","Id")) %>%
                           full_join(dailySteps, by = c("ActivityDay","Id")) 
  rm(dailyCalories,dailyIntensities,dailySteps)
```

#### generate date variable (named to align with date column in minute-level datasets)


```r
  str(daily$ActivityDay)
```

```
##  chr [1:2415] "11/8/2021" "11/9/2021" "11/10/2021" "11/11/2021" ...
```

```r
  daily$date<-as.Date(daily$ActivityDay,"%m/%d/%Y")
  lubridate::tz(daily$date)
```

```
## [1] "UTC"
```

### merge day-level activity data and day-level sleep data 

#### generate date in the day-level sleep data 


```r
  sleepStagesDay$date<-as.Date(lubridate::mdy_hms(sleepStagesDay$SleepDay))
```

#### merge day-level activity data and day-level sleep data


```r
  daily<-daily %>% full_join(sleepStagesDay, by = c("date","Id"))
  daily<-daily %>% select(-ActivityDay,-SleepDay)
  rm(sleepStagesDay)
```

#### rename columns in day-level data (so it is clear which columns are minute- vs. day-level)


```r
  daily<-daily %>% rename(TotalCalories=Calories) %>%
                   relocate(date,.after=Id) 
  names(daily)
```

```
##  [1] "Id"                       "date"                    
##  [3] "TotalCalories"            "SedentaryMinutes"        
##  [5] "LightlyActiveMinutes"     "FairlyActiveMinutes"     
##  [7] "VeryActiveMinutes"        "SedentaryActiveDistance" 
##  [9] "LightActiveDistance"      "ModeratelyActiveDistance"
## [11] "VeryActiveDistance"       "StepTotal"               
## [13] "TotalSleepRecords"        "TotalMinutesAsleep"      
## [15] "TotalTimeInBed"           "TotalTimeAwake"          
## [17] "TotalMinutesLight"        "TotalMinutesDeep"        
## [19] "TotalMinutesREM"
```

### merge minute- and day-level dataframes


```r
  data<-merge(daily,minute,by=c("Id","date"))
  rm(daily,minute)
```

## add battery (and syncEvents) data

### explore data

#### clean columns in battery data


```r
  battery$dateTime<-as.character(lubridate::mdy_hms(battery$DateTime))
  substr(battery$dateTime,18,19)<-"00"
  battery$dateTime<-as.POSIXct(battery$dateTime,tz="UTC")
  lubridate::tz(battery$dateTime)
```

```
## [1] "UTC"
```

```r
  battery$lastSync<-lubridate::mdy_hms(battery$LastSync)
  battery<-battery %>% select(Id,dateTime,lastSync,DeviceName,BatteryLevel)
```

#### sometimes there are multiple sync's per minute, but they always have the same battery level


```r
# sometimes there are multiple syncs within the same minute
  test<-battery %>% group_by(Id,dateTime) %>% mutate(length=length(Id),
                                                     unique=length(unique(Id)))
  table(test$unique)
```

```
## 
##      1 
## 120675
```

```r
  rm(test)
```


#### take list of distinct battery measurements (one per minute maximum)


```r
  battery<-distinct(battery,across(-"lastSync"))
```


#### explore syncEvents - contains no additional info above and beyond battery data


```r
  names(syncEvents)
```

```
## [1] "Id"          "DateTime"    "SyncDateUTC" "Provider"    "DeviceName"
```

```r
# will use battery data instead (has the same and more information as syncEvents)
  rm(syncEvents)
```

### merge battery data with activity and sleep data


```r
  data<-merge(data,battery,by=c("Id","dateTime"),all=TRUE)
  rm(battery)
```

## explore remaining dataframes

### sleepLogInfo


```r
# contains other information re: sleep, including the time they went to sleep, how efficient the sleep was, minutes to fall asleep, time in bed, awakeCount, AwakeDuration, RestlessCount, Restless Duration
# may not be necessary for this study? will omit for now and revisit
  str(sleepLogInfo)
```

```
## 'data.frame':	2549 obs. of  14 variables:
##  $ Id                 : chr  "A01" "A01" "A01" "A01" ...
##  $ LogId              : num  3.45e+10 3.45e+10 3.45e+10 3.45e+10 3.45e+10 ...
##  $ StartTime          : chr  "11/8/2021 12:16:00 AM" "11/9/2021 12:57:30 AM" "11/9/2021 9:09:30 PM" "11/11/2021 12:33:00 AM" ...
##  $ Duration           : int  29760000 28320000 45600000 25680000 6120000 38160000 35820000 39300000 29520000 31080000 ...
##  $ Efficiency         : int  90 97 93 99 88 99 99 98 91 93 ...
##  $ IsMainSleep        : chr  "True" "True" "True" "True" ...
##  $ MinutesAfterWakeup : int  0 6 0 0 0 0 18 0 0 0 ...
##  $ MinutesAsleep      : int  446 450 708 402 90 629 518 631 450 480 ...
##  $ MinutesToFallAsleep: int  0 0 0 20 0 2 54 14 0 0 ...
##  $ TimeInBed          : int  496 472 760 428 102 636 597 655 492 518 ...
##  $ AwakeCount         : int  1 2 2 0 1 0 0 0 1 0 ...
##  $ AwakeDuration      : int  2 5 5 0 2 0 0 0 1 0 ...
##  $ RestlessCount      : int  16 9 11 8 5 0 0 0 22 18 ...
##  $ RestlessDuration   : int  48 17 47 12 10 0 0 0 41 38 ...
```

```r
  rm(sleepLogInfo)
```

### sleepStageLogInfo


```r
# more information relating to each individual sleep bout of sleep, including the time spent in each intensity
# however, primary dataframe (data) already contains much of the high-level summary statistics re: sleep
  str(sleepStageLogInfo)
```

```
## 'data.frame':	2298 obs. of  29 variables:
##  $ Id                     : chr  "A01" "A01" "A01" "A01" ...
##  $ LogId                  : num  3.45e+10 3.45e+10 3.45e+10 3.45e+10 3.45e+10 ...
##  $ StartTime              : chr  "11/8/2021 12:16:00 AM" "11/9/2021 12:57:30 AM" "11/9/2021 9:09:30 PM" "11/11/2021 12:33:00 AM" ...
##  $ Duration               : int  29760000 28320000 45600000 25680000 6120000 38160000 35820000 39300000 29520000 31080000 ...
##  $ Efficiency             : int  90 97 93 99 88 99 99 98 91 93 ...
##  $ IsMainSleep            : chr  "True" "True" "True" "True" ...
##  $ SleepDataType          : chr  "classic" "stages" "stages" "classic" ...
##  $ MinutesAfterWakeUp     : int  0 6 0 0 0 0 18 0 0 0 ...
##  $ MinutesAsleep          : int  446 422 615 402 90 629 518 631 371 418 ...
##  $ MinutesToFallAsleep    : int  0 0 0 20 0 2 54 14 0 0 ...
##  $ TimeInBed              : int  496 472 760 428 102 636 597 655 492 518 ...
##  $ ClassicAsleepCount     : int  0 NA NA 0 0 0 0 0 NA NA ...
##  $ ClassicAsleepDuration  : int  446 NA NA 402 90 629 518 631 NA NA ...
##  $ ClassicAwakeCount      : int  1 NA NA 0 1 0 0 0 NA NA ...
##  $ ClassicAwakeDuration   : int  2 NA NA 0 2 0 0 0 NA NA ...
##  $ ClassicRestlessCount   : int  16 NA NA 8 5 0 0 0 NA NA ...
##  $ ClassicRestlessDuration: int  48 NA NA 12 10 0 0 0 NA NA ...
##  $ StagesWakeCount        : int  NA 30 36 NA NA NA NA NA 30 29 ...
##  $ StagesWakeDuration     : int  NA 50 145 NA NA NA NA NA 121 100 ...
##  $ StagesWakeThirtyDayAvg : int  NA 65 63 NA NA NA NA NA 72 78 ...
##  $ StagesLightCount       : int  NA 28 46 NA NA NA NA NA 31 30 ...
##  $ StagesLightDuration    : int  NA 269 382 NA NA NA NA NA 241 230 ...
##  $ StagesLightThirtyDayAvg: int  NA 258 260 NA NA NA NA NA 273 265 ...
##  $ StagesDeepCount        : int  NA 2 8 NA NA NA NA NA 3 6 ...
##  $ StagesDeepDuration     : int  NA 42 99 NA NA NA NA NA 66 95 ...
##  $ StagesDeepThirtyDayAvg : int  NA 72 68 NA NA NA NA NA 71 73 ...
##  $ StagesREMCount         : int  NA 11 9 NA NA NA NA NA 7 10 ...
##  $ StagesREMDuration      : int  NA 111 134 NA NA NA NA NA 64 93 ...
##  $ StagesREMThirtyDayAvg  : int  NA 89 92 NA NA NA NA NA 96 98 ...
```

```r
  rm(sleepStageLogInfo)
```

### 30 second sleep stages


```r
# already have minute-level sleep stages (incorporated into the dataframe)
  str(`30secondSleepStages`)
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

```r
  rm(`30secondSleepStages`)
```

### heartRateZones


```r
# long dataset with day-level heart rate zone information 
# four rows per person, one per each zone (out of range, fat burn, cardio, peak)
# structured this way to have a min and max column defining the heart rate range corresponding to the label (range will differ between participants)
  str(heartRateZones)
```

```
## 'data.frame':	9660 obs. of  7 variables:
##  $ Id         : chr  "A01" "A01" "A01" "A01" ...
##  $ ActivityDay: chr  "11/8/2021" "11/8/2021" "11/8/2021" "11/8/2021" ...
##  $ ZoneName   : chr  "Out of Range" "Fat Burn" "Cardio" "Peak" ...
##  $ Min        : int  30 119 146 181 30 119 147 181 30 119 ...
##  $ Max        : int  119 146 181 220 119 147 181 220 119 147 ...
##  $ Calories   : int  2109 221 33 0 2057 147 0 0 1917 131 ...
##  $ Minutes    : int  1400 35 4 0 1415 24 0 0 1418 21 ...
```

#### re-structure heartRateZones before merging


```r
# create a "range column" for each (can always parse into min/max columns if necessary?)
  heartRateZones$range<-paste(heartRateZones$Min,"-",heartRateZones$Max,sep="")

# pivot wider
  HRrange<-heartRateZones %>% select(Id,ActivityDay,ZoneName,range)
  HRrange<-pivot_wider(data=HRrange,id_cols=c("Id","ActivityDay"),names_from="ZoneName",values_from="range")
  HRrange<-HRrange %>% rename(range_outOfRange=`Out of Range`,
                              range_fatBurn=`Fat Burn`,
                              range_cardio=`Cardio`,
                              range_peak=`Peak`)
  
# generate minutes in each HR zone
  HRzoneMinutes<-heartRateZones %>% select(Id,ActivityDay,ZoneName,Minutes)
  HRzoneMinutes<-pivot_wider(data=HRzoneMinutes,id_cols=c("Id","ActivityDay"),names_from="ZoneName",values_from="Minutes")
  HRzoneMinutes<-HRzoneMinutes %>% rename(total_outOfRange=`Out of Range`,
                                          total_fatBurn=`Fat Burn`,
                                          total_cardio=`Cardio`,
                                          total_peak=`Peak`)
  
# merge back together
  HRzones<-HRrange %>% full_join(HRzoneMinutes,by=c("Id","ActivityDay"))
  
# remove dataframes I don't need
  rm(HRzoneMinutes,heartRateZones,HRrange)
```

#### generate date column in day-level heartrate zone data


```r
  HRzones$date<-as.Date(HRzones$ActivityDay,"%m/%d/%Y")
  HRzones<-HRzones %>% select(-ActivityDay)
```

#### merge with data


```r
  data<-merge(data,HRzones,by=c("Id","date"))
  rm(HRzones)
```

### activitylogs


```r
# similar to sleepLogInfo and sleepStageLogInfo, but for workouts
# one row per workout
  str(activitylogs)
```

```
## 'data.frame':	1480 obs. of  20 variables:
##  $ Id                        : chr  "A01" "A01" "A01" "A01" ...
##  $ Date                      : chr  "11/11/2021" "11/11/2021" "11/13/2021" "11/14/2021" ...
##  $ StartTime                 : chr  "11:04:28" "15:32:28" "23:46:35" "2:22:45" ...
##  $ Duration                  : int  1485000 1229000 3021000 1229000 1588000 1025000 1076000 1434000 921000 1177000 ...
##  $ Activity                  : chr  "Walk" "Walk" "Walk" "Walk" ...
##  $ ActivityType              : int  90013 90013 90013 90013 3001 90013 90013 90013 90013 90013 ...
##  $ LogType                   : chr  "auto_detected" "auto_detected" "auto_detected" "auto_detected" ...
##  $ Steps                     : int  1564 1769 3691 1930 937 1549 1532 1995 1239 1931 ...
##  $ Distance                  : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ ElevationGain             : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ Calories                  : int  128 123 283 148 139 108 81 112 71 97 ...
##  $ SedentaryMinutes          : int  0 0 2 0 0 0 0 0 0 0 ...
##  $ LightlyActiveMinutes      : int  15 1 5 0 6 0 3 4 1 2 ...
##  $ FairlyActiveMinutes       : int  1 5 21 3 13 4 7 17 14 9 ...
##  $ VeryActiveMinutes         : int  8 14 22 18 7 13 7 2 0 8 ...
##  $ AverageHeartRate          : int  111 119 117 131 124 123 101 114 114 111 ...
##  $ OutOfRangeHeartRateMinutes: int  17 9 26 2 12 9 17 18 15 14 ...
##  $ FatBurnHeartRateMinutes   : int  7 11 24 15 13 8 0 5 0 5 ...
##  $ CardioHeartRateMinutes    : int  0 0 0 3 1 0 0 0 0 0 ...
##  $ PeakHeartRateMinutes      : int  0 0 0 0 0 0 0 0 0 0 ...
```

#### can merge workout data at the start time?


```r
  activitylogs$dateTime<-paste(activitylogs$Date,activitylogs$StartTime,sep=" ")
  substr(activitylogs$dateTime,18,19)<-"00"
  activitylogs$dateTime<-lubridate::mdy_hms(activitylogs$dateTime)
```

#### select and rename columns 


```r
  names(activitylogs)
```

```
##  [1] "Id"                         "Date"                      
##  [3] "StartTime"                  "Duration"                  
##  [5] "Activity"                   "ActivityType"              
##  [7] "LogType"                    "Steps"                     
##  [9] "Distance"                   "ElevationGain"             
## [11] "Calories"                   "SedentaryMinutes"          
## [13] "LightlyActiveMinutes"       "FairlyActiveMinutes"       
## [15] "VeryActiveMinutes"          "AverageHeartRate"          
## [17] "OutOfRangeHeartRateMinutes" "FatBurnHeartRateMinutes"   
## [19] "CardioHeartRateMinutes"     "PeakHeartRateMinutes"      
## [21] "dateTime"
```

```r
  activitylogs<-activitylogs %>% select(-Date,-StartTime,-ActivityType) %>%
                                 rename(activity_duration=Duration,
                                        activity=Activity,
                                        activity_logType=LogType,
                                        activity_steps=Steps,
                                        activity_distance=Distance,
                                        activity_elevationGain=ElevationGain,
                                        activity_calories=Calories,
                                        activity_sedentaryMins=SedentaryMinutes,
                                        activity_lightlyActiveMins=LightlyActiveMinutes,
                                        activity_fairlyActiveMins=FairlyActiveMinutes,
                                        activity_veryActiveMins=VeryActiveMinutes,
                                        activity_avgHeartRate=AverageHeartRate,
                                        activity_outOfRangeMins=OutOfRangeHeartRateMinutes,
                                        activity_fatBurnMins=FatBurnHeartRateMinutes,
                                        activity_cardioMins=CardioHeartRateMinutes,
                                        activity_peakMins=PeakHeartRateMinutes)
```


#### merge with data


```r
  names(activitylogs)
```

```
##  [1] "Id"                         "activity_duration"         
##  [3] "activity"                   "activity_logType"          
##  [5] "activity_steps"             "activity_distance"         
##  [7] "activity_elevationGain"     "activity_calories"         
##  [9] "activity_sedentaryMins"     "activity_lightlyActiveMins"
## [11] "activity_fairlyActiveMins"  "activity_veryActiveMins"   
## [13] "activity_avgHeartRate"      "activity_outOfRangeMins"   
## [15] "activity_fatBurnMins"       "activity_cardioMins"       
## [17] "activity_peakMins"          "dateTime"
```

```r
  data<-merge(data,activitylogs,by=c("Id","dateTime"),all.x=TRUE)
  rm(activitylogs)
```


