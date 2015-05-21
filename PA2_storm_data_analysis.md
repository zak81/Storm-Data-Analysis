# Storm events impacting health and economy

## Synopsis:
#####In my analysis of storm events data, I used NOAA Storm Database obtained from (https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2).  I used RStudio to create all documents, including a preprocessed data.  My goal is to obtain a lean data with key variables that I need for answering a question - "What is the most hazardous events impacting health and economy?"  Key variable I focused on were fatality counts, injury counts, property damage estimate and crop damage estimate.  Since not all event names are entered as one of 48 storm event name given in [Storm Data Event Table 2.1.1](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf), I need to fix and change event names to match it.  In the process, I made some assumptions when the name was ambiguous.  I deleted events titled "Other" or other names that didn't belong to any of the 48 events.  There will be sections below for Data Processing, which shows the codes and description of the process.  Results are shown at the end of my report.

## Data Processing:

#### Loading the data
##### Load the R-packages dplyr, ggplot2 and reshape2

```r
suppressPackageStartupMessages(library(dplyr))
library(ggplot2)
library(reshape2)
```

##### The data comes in the form of comma-separated-value file compressed via the bzip2 algorithm.  Download, extract and read into a new data frame.

```r
inputdata_url <- "http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
inputdata_file <- "repdata-data-StormData.csv.bz2"

if(!file.exists("repdata-data-StormData.csv.bz2")) {
        download.file(inputdata_url, inputdata_file)
}

StormData <- read.csv(bzfile(inputdata_file))
```
#### Preprocessing the data
##### Look at the summary statistics and dimension. Look at first 6 lines.

```r
summary(StormData)
```

```
##     STATE__                  BGN_DATE             BGN_TIME     
##  Min.   : 1.0   5/25/2011 0:00:00:  1202   12:00:00 AM: 10163  
##  1st Qu.:19.0   4/27/2011 0:00:00:  1193   06:00:00 PM:  7350  
##  Median :30.0   6/9/2011 0:00:00 :  1030   04:00:00 PM:  7261  
##  Mean   :31.2   5/30/2004 0:00:00:  1016   05:00:00 PM:  6891  
##  3rd Qu.:45.0   4/4/2011 0:00:00 :  1009   12:00:00 PM:  6703  
##  Max.   :95.0   4/2/2006 0:00:00 :   981   03:00:00 PM:  6700  
##                 (Other)          :895866   (Other)    :857229  
##    TIME_ZONE          COUNTY           COUNTYNAME         STATE       
##  CST    :547493   Min.   :  0.0   JEFFERSON :  7840   TX     : 83728  
##  EST    :245558   1st Qu.: 31.0   WASHINGTON:  7603   KS     : 53440  
##  MST    : 68390   Median : 75.0   JACKSON   :  6660   OK     : 46802  
##  PST    : 28302   Mean   :100.6   FRANKLIN  :  6256   MO     : 35648  
##  AST    :  6360   3rd Qu.:131.0   LINCOLN   :  5937   IA     : 31069  
##  HST    :  2563   Max.   :873.0   MADISON   :  5632   NE     : 30271  
##  (Other):  3631                   (Other)   :862369   (Other):621339  
##                EVTYPE         BGN_RANGE           BGN_AZI      
##  HAIL             :288661   Min.   :   0.000          :547332  
##  TSTM WIND        :219940   1st Qu.:   0.000   N      : 86752  
##  THUNDERSTORM WIND: 82563   Median :   0.000   W      : 38446  
##  TORNADO          : 60652   Mean   :   1.484   S      : 37558  
##  FLASH FLOOD      : 54277   3rd Qu.:   1.000   E      : 33178  
##  FLOOD            : 25326   Max.   :3749.000   NW     : 24041  
##  (Other)          :170878                      (Other):134990  
##          BGN_LOCATI                  END_DATE             END_TIME     
##               :287743                    :243411              :238978  
##  COUNTYWIDE   : 19680   4/27/2011 0:00:00:  1214   06:00:00 PM:  9802  
##  Countywide   :   993   5/25/2011 0:00:00:  1196   05:00:00 PM:  8314  
##  SPRINGFIELD  :   843   6/9/2011 0:00:00 :  1021   04:00:00 PM:  8104  
##  SOUTH PORTION:   810   4/4/2011 0:00:00 :  1007   12:00:00 PM:  7483  
##  NORTH PORTION:   784   5/30/2004 0:00:00:   998   11:59:00 PM:  7184  
##  (Other)      :591444   (Other)          :653450   (Other)    :622432  
##    COUNTY_END COUNTYENDN       END_RANGE           END_AZI      
##  Min.   :0    Mode:logical   Min.   :  0.0000          :724837  
##  1st Qu.:0    NA's:902297    1st Qu.:  0.0000   N      : 28082  
##  Median :0                   Median :  0.0000   S      : 22510  
##  Mean   :0                   Mean   :  0.9862   W      : 20119  
##  3rd Qu.:0                   3rd Qu.:  0.0000   E      : 20047  
##  Max.   :0                   Max.   :925.0000   NE     : 14606  
##                                                 (Other): 72096  
##            END_LOCATI         LENGTH              WIDTH         
##                 :499225   Min.   :   0.0000   Min.   :   0.000  
##  COUNTYWIDE     : 19731   1st Qu.:   0.0000   1st Qu.:   0.000  
##  SOUTH PORTION  :   833   Median :   0.0000   Median :   0.000  
##  NORTH PORTION  :   780   Mean   :   0.2301   Mean   :   7.503  
##  CENTRAL PORTION:   617   3rd Qu.:   0.0000   3rd Qu.:   0.000  
##  SPRINGFIELD    :   575   Max.   :2315.0000   Max.   :4400.000  
##  (Other)        :380536                                         
##        F               MAG            FATALITIES          INJURIES        
##  Min.   :0.0      Min.   :    0.0   Min.   :  0.0000   Min.   :   0.0000  
##  1st Qu.:0.0      1st Qu.:    0.0   1st Qu.:  0.0000   1st Qu.:   0.0000  
##  Median :1.0      Median :   50.0   Median :  0.0000   Median :   0.0000  
##  Mean   :0.9      Mean   :   46.9   Mean   :  0.0168   Mean   :   0.1557  
##  3rd Qu.:1.0      3rd Qu.:   75.0   3rd Qu.:  0.0000   3rd Qu.:   0.0000  
##  Max.   :5.0      Max.   :22000.0   Max.   :583.0000   Max.   :1700.0000  
##  NA's   :843563                                                           
##     PROPDMG          PROPDMGEXP        CROPDMG          CROPDMGEXP    
##  Min.   :   0.00          :465934   Min.   :  0.000          :618413  
##  1st Qu.:   0.00   K      :424665   1st Qu.:  0.000   K      :281832  
##  Median :   0.00   M      : 11330   Median :  0.000   M      :  1994  
##  Mean   :  12.06   0      :   216   Mean   :  1.527   k      :    21  
##  3rd Qu.:   0.50   B      :    40   3rd Qu.:  0.000   0      :    19  
##  Max.   :5000.00   5      :    28   Max.   :990.000   B      :     9  
##                    (Other):    84                     (Other):     9  
##       WFO                                       STATEOFFIC    
##         :142069                                      :248769  
##  OUN    : 17393   TEXAS, North                       : 12193  
##  JAN    : 13889   ARKANSAS, Central and North Central: 11738  
##  LWX    : 13174   IOWA, Central                      : 11345  
##  PHI    : 12551   KANSAS, Southwest                  : 11212  
##  TSA    : 12483   GEORGIA, North and Central         : 11120  
##  (Other):690738   (Other)                            :595920  
##                                                                                                                                                                                                     ZONENAMES     
##                                                                                                                                                                                                          :594029  
##                                                                                                                                                                                                          :205988  
##  GREATER RENO / CARSON CITY / M - GREATER RENO / CARSON CITY / M                                                                                                                                         :   639  
##  GREATER LAKE TAHOE AREA - GREATER LAKE TAHOE AREA                                                                                                                                                       :   592  
##  JEFFERSON - JEFFERSON                                                                                                                                                                                   :   303  
##  MADISON - MADISON                                                                                                                                                                                       :   302  
##  (Other)                                                                                                                                                                                                 :100444  
##     LATITUDE      LONGITUDE        LATITUDE_E     LONGITUDE_    
##  Min.   :   0   Min.   :-14451   Min.   :   0   Min.   :-14455  
##  1st Qu.:2802   1st Qu.:  7247   1st Qu.:   0   1st Qu.:     0  
##  Median :3540   Median :  8707   Median :   0   Median :     0  
##  Mean   :2875   Mean   :  6940   Mean   :1452   Mean   :  3509  
##  3rd Qu.:4019   3rd Qu.:  9605   3rd Qu.:3549   3rd Qu.:  8735  
##  Max.   :9706   Max.   : 17124   Max.   :9706   Max.   :106220  
##  NA's   :47                      NA's   :40                     
##                                            REMARKS           REFNUM      
##                                                :287433   Min.   :     1  
##                                                : 24013   1st Qu.:225575  
##  Trees down.\n                                 :  1110   Median :451149  
##  Several trees were blown down.\n              :   568   Mean   :451149  
##  Trees were downed.\n                          :   446   3rd Qu.:676723  
##  Large trees and power lines were blown down.\n:   432   Max.   :902297  
##  (Other)                                       :588295
```

```r
dim(StormData)
```

```
## [1] 902297     37
```

```r
head(StormData)
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1 TORNADO         0                                               0
## 2 TORNADO         0                                               0
## 3 TORNADO         0                                               0
## 4 TORNADO         0                                               0
## 5 TORNADO         0                                               0
## 6 TORNADO         0                                               0
##   COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1         NA         0                      14.0   100 3   0          0
## 2         NA         0                       2.0   150 2   0          0
## 3         NA         0                       0.1   123 2   0          0
## 4         NA         0                       0.0   100 2   0          0
## 5         NA         0                       0.0   150 2   0          0
## 6         NA         0                       1.5   177 2   0          0
##   INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC ZONENAMES
## 1       15    25.0          K       0                                    
## 2        0     2.5          K       0                                    
## 3        2    25.0          K       0                                    
## 4        2     2.5          K       0                                    
## 5        2     2.5          K       0                                    
## 6        6     2.5          K       0                                    
##   LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1     3040      8812       3051       8806              1
## 2     3042      8755          0          0              2
## 3     3340      8742          0          0              3
## 4     3458      8626          0          0              4
## 5     3412      8642          0          0              5
## 6     3450      8748          0          0              6
```
##### Look at the number of types of event names in EVTYPE. There are 985 events in this data. 

```r
length(levels(StormData$EVTYPE))
```

```
## [1] 985
```
##### Out of 37 variables, I picked out EVTYPE, FATALITIES, INJURIES, PROPDMG, PROPDMGEXP, CROPDMG and CROPDMGEXP for my analysis. EVTYPE defines the type of storm event, FATALITIES and INJURIES define number of fatalities and injuries caused as a result of an event, PROPDMG and PROPDMGEXP define cost incurred from property damage, and CROPDMG AND CROPDMGEXP define cost incurred from crop damage.
##### Check for NA values. No NA found in FATALITIES, INJURIES, PROPDMG and CROPDMG

```r
sum(is.na(StormData$FATALITIES))
```

```
## [1] 0
```

```r
sum(is.na(StormData$INJURIES))
```

```
## [1] 0
```

```r
sum(is.na(StormData$PROPDMG))
```

```
## [1] 0
```

```r
sum(is.na(StormData$CROPDMG))
```

```
## [1] 0
```
##### Check for units in PROPDMGEXP and CROPDMGEXP. Values of the damage are reported in dollar and those values with units are in hundreds(H), thousands(K), millions(M) and billions(B). 

```r
levels(StormData$PROPDMGEXP[StormData$PROPDMGEXP])
```

```
##  [1] ""  "-" "?" "+" "0" "1" "2" "3" "4" "5" "6" "7" "8" "B" "h" "H" "K"
## [18] "m" "M"
```

```r
levels(StormData$CROPDMGEXP[StormData$CROPDMGEXP])
```

```
## [1] ""  "?" "0" "2" "B" "k" "K" "m" "M"
```
##### I can check to see if events with undefined damage units have values. Event with "?" as a unit doesn't have a property damage value recorded. Question can be asked if the value should be $3, $3K, $3M or $3B? Since it's difficult to make assumption without having additional information about the events in question, I'm going to leave all undefined units as "1".

```r
head(filter(StormData, PROPDMGEXP=="" & PROPDMG!=0), 2)
```

```
##   STATE__         BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       6 3/9/1995 0:00:00     0301       PST     41      MARIN    CA
## 2      12 1/8/1993 0:00:00     1130       EST    125      UNION    FL
##              EVTYPE BGN_RANGE BGN_AZI  BGN_LOCATI          END_DATE
## 1 FLASH FLOOD WINDS         0                     3/10/1995 0:00:00
## 2           TORNADO        24       N Gainesville                  
##   END_TIME COUNTY_END COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH
## 1  0400PST          0         NA         0                       0.0     0
## 2                   0         NA         0                       0.1    10
##    F MAG FATALITIES INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO
## 1 NA   0          0        0    0.41                  0          ?    
## 2  0   0          0        0    3.00                  0               
##   STATEOFFIC ZONENAMES LATITUDE LONGITUDE LATITUDE_E LONGITUDE_
## 1                             0         0          0          0
## 2                          3003      8219          0          0
##                                                                                                                              REMARKS
## 1 Wind gusts to 96 mph Mt Tamalpias and 89 mph at the Golden Gate Bridge Petaluma river at Petuluma went 1.6 feet over flood stage. 
## 2               A small tornado touched down at the North Florida Prison Reception Center damaging the building before dissipating. 
##   REFNUM
## 1 192440
## 2 196182
```

```r
head(filter(StormData, PROPDMGEXP=="?" & PROPDMG!=0), 2)
```

```
##  [1] STATE__    BGN_DATE   BGN_TIME   TIME_ZONE  COUNTY     COUNTYNAME
##  [7] STATE      EVTYPE     BGN_RANGE  BGN_AZI    BGN_LOCATI END_DATE  
## [13] END_TIME   COUNTY_END COUNTYENDN END_RANGE  END_AZI    END_LOCATI
## [19] LENGTH     WIDTH      F          MAG        FATALITIES INJURIES  
## [25] PROPDMG    PROPDMGEXP CROPDMG    CROPDMGEXP WFO        STATEOFFIC
## [31] ZONENAMES  LATITUDE   LONGITUDE  LATITUDE_E LONGITUDE_ REMARKS   
## [37] REFNUM    
## <0 rows> (or 0-length row.names)
```

```r
head(filter(StormData, CROPDMGEXP=="" & CROPDMG!=0), 2)
```

```
##   STATE__         BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1      38 7/4/1994 0:00:00     0400       CST     93   STUTSMAN    ND
## 2      48 4/5/1994 0:00:00     1700       CST    209       HAYS    TX
##               EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME
## 1               HAIL         0          Jamestown                  
## 2 THUNDERSTORM WINDS         0         San Marcos                  
##   COUNTY_END COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH  F MAG
## 1          0         NA         0                         0     0 NA 175
## 2          0         NA         0                         0     0 NA  52
##   FATALITIES INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC
## 1          0        0       5          K       3                          
## 2          0        0       5          M       4                          
##   ZONENAMES LATITUDE LONGITUDE LATITUDE_E LONGITUDE_
## 1                  0         0          0          0
## 2                  0         0          0          0
##                                                                                                                                                                                                                                                                                                              REMARKS
## 1                                                                                                                                                                                                                                                                                                                   
## 2 Thunderstorms produced widespread large hail, accompanied by damaging winds that knocked down tree limbs, stripped leaves from trees and knocked out power and telephone communications to San Marcos for several hours.  The hailstones broke windows in homes and school as well as Southwest State University. 
##   REFNUM
## 1 221817
## 2 238757
```

```r
head(filter(StormData, CROPDMGEXP=="?" & CROPDMG!=0), 2)
```

```
##  [1] STATE__    BGN_DATE   BGN_TIME   TIME_ZONE  COUNTY     COUNTYNAME
##  [7] STATE      EVTYPE     BGN_RANGE  BGN_AZI    BGN_LOCATI END_DATE  
## [13] END_TIME   COUNTY_END COUNTYENDN END_RANGE  END_AZI    END_LOCATI
## [19] LENGTH     WIDTH      F          MAG        FATALITIES INJURIES  
## [25] PROPDMG    PROPDMGEXP CROPDMG    CROPDMGEXP WFO        STATEOFFIC
## [31] ZONENAMES  LATITUDE   LONGITUDE  LATITUDE_E LONGITUDE_ REMARKS   
## [37] REFNUM    
## <0 rows> (or 0-length row.names)
```
##### I'll convert the units into numerical values by assigning numerical values to each factor in PROPDMGEXP and CROPDMGEXP.

```r
levels(StormData$PROPDMGEXP) <- c("1","1","1","1","1","1","1","1","1","1","1","1","1","1000000000","100","100","1000","1000000","1000000")
levels(StormData$CROPDMGEXP) <- c("1","1","1","1","1000000000","1000","1000","1000000","1000000")
```
##### Create new variables to store damage estimates.

```r
StormData <- StormData %>% mutate(PROPDMGEST = PROPDMG * as.numeric(levels(StormData$PROPDMGEXP))[StormData$PROPDMGEXP])
StormData <- StormData %>% mutate(CROPDMGEST = CROPDMG * as.numeric(levels(StormData$CROPDMGEXP))[StormData$CROPDMGEXP])
```
##### Get sum of fatalities, injuries, property damage estimate, crop damage estimate for each event type. Then reorder the rows in descending order by number of fatalities, then by number of injuries.

```r
event_grouped <- StormData %>%
        group_by(EVTYPE) %>%
        summarise_each(funs(sum), FATALITIES, INJURIES, PROPDMGEST, CROPDMGEST) %>%
        arrange(desc(FATALITIES, INJURIES))
```
##### See how many events have zero values in 4 variables of interest

```r
count(filter(event_grouped, FATALITIES==0 & INJURIES==0 & PROPDMGEST==0 & CROPDMGEST==0))
```

```
## Source: local data frame [1 x 1]
## 
##     n
## 1 497
```
##### Filter out events that result in 0 fatalities, 0 injuries, no property damage estimate and no crop damage estimate. 497 events without fatality, injury, property damage and crop damage are removed from 985 events.

```r
event_filter <- filter(event_grouped, FATALITIES!=0 | INJURIES!=0 | PROPDMGEST!=0 | CROPDMGEST!=0)
```
##### Cleaning of the dataset is performed. Refer to [Storm Data Documention](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf) for 48 storm events that I'll end up with as a result of combining 497 events names.

```r
event_clean <- event_filter
event_clean$EVTYPE <- gsub("TORNDAO", "TORNADO", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("tornado", event_clean$EVTYPE, ignore.case=TRUE)] <- "TORNADO"
event_clean$EVTYPE[grep("excessive.*heat|heat.*excessive|heat wave|extreme heat", event_clean$EVTYPE, ignore.case=TRUE)] <- "EXCESSIVE HEAT"
event_clean$EVTYPE[grep("flash.*flood|flood.*flash", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLASH FLOOD"
event_clean$EVTYPE[grep("^heat$", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "HEAT"
```

```r
event_clean$EVTYPE <- gsub("LIGNTNING", "LIGHTNING", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("lightning", event_clean$EVTYPE, ignore.case=TRUE)] <- "LIGHTNING"
event_clean$EVTYPE <- gsub("TSTM", "THUNDERSTORM", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE <- gsub("TUNDERSTORM", "THUNDERSTORM", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("^thunderstorm|^thunderstormw|^severe thunderstorm", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE <- gsub("CSTL", "COASTAL", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("^flood|river.*flood|minor.*flood|urban.*flood|breakup.*flood|rain.*flood|wind.*flood|surf.*flood|ice.*flood|major.*flood|rural.*flood|stream.*flood|snow.*flood|tidal.*flood", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLOOD"
event_clean$EVTYPE[grep("rip|current", event_clean$EVTYPE, ignore.case=TRUE)] <- "RIP CURRENTS"
event_clean$EVTYPE[grep("^high wind|hurricane.*wind|winter.*wind|dust.*wind|snow.*high.*wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH WIND"
event_clean$EVTYPE <- gsub("AVALANCE", "AVALANCHE", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("avalanche", event_clean$EVTYPE, ignore.case=TRUE)] <- "AVALANCHE"
event_clean$EVTYPE[grep("winter storm", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER STORM"
event_clean$EVTYPE <- gsub("WINDCHILL", "WIND CHILL", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("extreme.*cold|cold.*extreme|extreme.*chill|record.*cold", event_clean$EVTYPE, ignore.case=TRUE)] <- "EXTREME COLD/WIND CHILL"
event_clean$EVTYPE[grep("heavy.*snow|snow.*heavy", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY SNOW"
event_clean$EVTYPE[grep("^strong.*wind|wind.*strong|^ice.*wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "STRONG WIND"
event_clean$EVTYPE[grep("blizzard", event_clean$EVTYPE, ignore.case=TRUE)] <- "BLIZZARD"
event_clean$EVTYPE[grep("high.*surf|surf.*high", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH SURF"
event_clean$EVTYPE[grep("heavy.*rain|rain.*heavy", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY RAIN"
event_clean$EVTYPE[grep("ice.*storm|storm.*ice", event_clean$EVTYPE, ignore.case=TRUE)] <- "ICE STORM"
event_clean$EVTYPE[grep("wild.*fire|fire.*wild|wildfire", event_clean$EVTYPE, ignore.case=TRUE)] <- "WILDFIRE"
event_clean$EVTYPE[grep("hurricane.*typhoon|typhoon.*hurricane|hurricane|typhoon", event_clean$EVTYPE, ignore.case=TRUE)] <- "HURRICANE (TYPHOON)"
event_clean$EVTYPE[grep("^fog|dense.*fog", event_clean$EVTYPE, ignore.case=TRUE)] <- "DENSE FOG"
event_clean$EVTYPE[grep("tropical.*storm|storm.*tropical", event_clean$EVTYPE, ignore.case=TRUE)] <- "TROPICAL STORM"
event_clean$EVTYPE[grep("landslide|land.*slide|slide.*land|debris.*flow|flow.*debris", event_clean$EVTYPE, ignore.case=TRUE)] <- "DEBRIS FLOW"
event_clean$EVTYPE[grep("^cold|snow.*cold|unseasonabl.*cold|extended.*cold", event_clean$EVTYPE, ignore.case=TRUE)] <- "COLD/WIND CHILL"
event_clean$EVTYPE[grep("tsunami", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "TSUNAMI"
```

```r
event_clean$EVTYPE[grep("winter.*weather|weather.*winter", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE[grep("warm.*dry|dry.*warm|^heat", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAT"
event_clean$EVTYPE <- gsub("FLD", "FLOOD", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("urban.*flood", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLOOD"
event_clean$EVTYPE[grep("^wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH WIND"
event_clean$EVTYPE[grep("^hail|wind.*hail|small.*hail", event_clean$EVTYPE, ignore.case=TRUE)] <- "HAIL"
event_clean$EVTYPE[grep("dust.*storm|storm.*dust", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "DUST STORM"
```

```r
event_clean$EVTYPE[grep("marine.*strong.*wind", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "MARINE STRONG WIND"
```

```r
event_clean$EVTYPE[grep("storm.*surge|high.*tide", event_clean$EVTYPE, ignore.case=TRUE)] <- "STORM SURGE/TIDE"
event_clean$EVTYPE[grep("unseasonabl.*warm", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAT"
event_clean$EVTYPE[grep("marin.*thunderstorm.*wind", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "MARINE THUNDERSTORM WIND" "MARINE THUNDERSTORM WIND"
```

```r
event_clean$EVTYPE[grep("rough.*sea", event_clean$EVTYPE, ignore.case=TRUE)] <- "MARINE THUNDERSTORM WIND"
event_clean$EVTYPE[grep("freez.*rain|sleet", event_clean$EVTYPE, ignore.case=TRUE)] <- "SLEET"
event_clean$EVTYPE[grep("glaze", event_clean$EVTYPE, ignore.case=TRUE)] <- "FROST/FREEZE"
event_clean$EVTYPE[grep("heavy.*surf|surf.*heavy", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH SURF"
event_clean$EVTYPE[grep("low.*temperature|temperature.*low", event_clean$EVTYPE, ignore.case=TRUE)] <- "COLD/WIND CHILL"
event_clean$EVTYPE[grep("marine.*mishap|mishap.*marine", event_clean$EVTYPE, ignore.case=TRUE)] <- "MARINE THUNDERSTORM WIND"
event_clean$EVTYPE[grep("high.*seas|seas.*high", event_clean$EVTYPE, ignore.case=TRUE)] <- "STORM SURGE/TIDE"
event_clean$EVTYPE[grep("icy.*road|road.*icy", event_clean$EVTYPE, ignore.case=TRUE)] <- "FROST/FREEZE"
event_clean$EVTYPE[grep("rip.*current|current.*rip", event_clean$EVTYPE, ignore.case=TRUE)] <- "RIP CURRENT"
event_clean$EVTYPE[grep("^snow|blowing.*snow|ice.*snow|light.*snow|falling.*snow|thundersnow|late.*snow|rain.*snow", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE[grep("gusty.*wind|wind.*gusty", event_clean$EVTYPE, ignore.case=TRUE)] <- "STRONG WIND"
event_clean$EVTYPE[grep("hypothermia|exposure", event_clean$EVTYPE, ignore.case=TRUE)] <- "EXCESSIVE HEAT"
event_clean$EVTYPE[grep("mudslide|mud.*slide|slide.*mud", event_clean$EVTYPE, ignore.case=TRUE)] <- "DEBRIS FLOW"
event_clean$EVTYPE[grep("rough.*surf|hazardous.*surf", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH SURF"
event_clean$EVTYPE[grep("dry.*microburst|microburst.*dry", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("heavy.*sea|sea.*heavy", event_clean$EVTYPE, ignore.case=TRUE)] <- "MARINE THUNDERSTORM WIND"
event_clean$EVTYPE[grep("high.*water|water.*high", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLASH FLOOD"
event_clean$EVTYPE[grep("waterspout|water.*spout", event_clean$EVTYPE, ignore.case=TRUE)] <- "WATERSPOUT"
event_clean$EVTYPE[grep("coast.*flood|flood.*coast", event_clean$EVTYPE, ignore.case=TRUE)] <- "COASTAL FLOOD"
event_clean$EVTYPE[grep("coast.*storm|storm.*coast", event_clean$EVTYPE, ignore.case=TRUE)] <- "TROPICAL STORM"
event_clean$EVTYPE[grep("excess.*rain|rain.*excess", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY RAIN"
event_clean$EVTYPE[grep("freez.*drizzle|drizzle.*freez", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE[grep("mixed.*precip|precip", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE[grep("record.*heat|heat.*record", event_clean$EVTYPE, ignore.case=TRUE)] <- "EXCESSIVE HEAT"
event_clean$EVTYPE[grep("^ice$|^ice [FJOR]|black.*ice", event_clean$EVTYPE, ignore.case=TRUE)] <- "FROST/FREEZE"
event_clean$EVTYPE[grep("freeze", event_clean$EVTYPE, ignore.case=TRUE)] <- "FROST/FREEZE"
event_clean$EVTYPE[grep("drown", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH SURF"
event_clean$EVTYPE[grep("freez.*spray", event_clean$EVTYPE, ignore.case=TRUE)] <- "FREEZING FOG"
event_clean$EVTYPE[grep("frost|freeze", event_clean$EVTYPE, ignore.case=TRUE)] <- "FROST/FREEZE"
event_clean$EVTYPE[grep("marine.*accident", event_clean$EVTYPE, ignore.case=TRUE)] <- "MARINE THUNDERSTORM WIND"
event_clean$EVTYPE[grep("ris.*water", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLASH FLOOD"
event_clean$EVTYPE[grep("whirl.*wind|whirl", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("wintry", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE[grep("^ thunderstorm", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"

event_clean$EVTYPE[grep("erosion", event_clean$EVTYPE, ignore.case=TRUE)] <- "COASTAL FLOOD"
event_clean$EVTYPE[grep("blowing.*dust", event_clean$EVTYPE, ignore.case=TRUE)] <- "DUST STORM"
event_clean$EVTYPE[grep("fire", event_clean$EVTYPE, ignore.case=TRUE)] <- "WILDFIRE"
event_clean$EVTYPE[grep("coast.*surge|surge.*coast", event_clean$EVTYPE, ignore.case=TRUE)] <- "COASTAL FLOOD"
event_clean$EVTYPE[grep("cool.*wet|excess.*wet", event_clean$EVTYPE, ignore.case=TRUE)] <- "WINTER WEATHER"
event_clean$EVTYPE <- gsub("MIRCOBURST", "MICROBURST", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("microburst", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("dam.*break", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLOOD"
event_clean$EVTYPE[grep("dust.*devil", event_clean$EVTYPE, ignore.case=TRUE)] <- "DUST DEVIL"
event_clean$EVTYPE[grep("excess.*snow", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY SNOW"
event_clean$EVTYPE[grep("downburst", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("funnel.*cloud", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "FUNNEL CLOUD"
```

```r
event_clean$EVTYPE[grep("gradient.*wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "TROPICAL DEPRESSION"
event_clean$EVTYPE[grep("gustnado", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("heavy.*mix", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY SNOW"
event_clean$EVTYPE[grep("heavy.*shower", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY RAIN"
event_clean$EVTYPE[grep("heavy.*swell|swell", event_clean$EVTYPE, ignore.case=TRUE)] <- "MARINE HIGH WIND"

event_clean$EVTYPE[grep("high  wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH WIND"
event_clean$EVTYPE[grep("hvy", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY RAIN"
event_clean$EVTYPE[grep("lake.*snow", event_clean$EVTYPE, ignore.case=TRUE)] <- "LAKE-EFFECT SNOW"
event_clean$EVTYPE[grep("land.*slump", event_clean$EVTYPE, ignore.case=TRUE)] <- "DEBRIS FLOW"
event_clean$EVTYPE[grep("land.*spout", event_clean$EVTYPE, ignore.case=TRUE)] <- "DUST DEVIL"
event_clean$EVTYPE <- gsub("LIGHTING", "LIGHTNING", ignore.case=TRUE, event_clean$EVTYPE)
event_clean$EVTYPE[grep("non.*wind", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH WIND"

event_clean$EVTYPE[grep("rain", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY RAIN"
event_clean$EVTYPE[grep("record.*snow", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAVY SNOW"
event_clean$EVTYPE[grep("rock.*slide", event_clean$EVTYPE, ignore.case=TRUE)] <- "DEBRIS FLOW"
event_clean$EVTYPE[grep("rogue.*wave|wave", event_clean$EVTYPE, ignore.case=TRUE)] <- "HIGH SURF"
event_clean$EVTYPE[grep("seiche", event_clean$EVTYPE, ignore.case=TRUE)]
```

```
## [1] "SEICHE"
```

```r
event_clean$EVTYPE[grep("turbulence", event_clean$EVTYPE, ignore.case=TRUE)] <- "FLOOD"
event_clean$EVTYPE[grep("^storm.*wind|^thunde|^thud|^thun", event_clean$EVTYPE, ignore.case=TRUE)] <- "THUNDERSTORM WIND"
event_clean$EVTYPE[grep("urban", event_clean$EVTYPE, ignore.case=TRUE)] <-"FLASH FLOOD"
event_clean$EVTYPE[grep("warm", event_clean$EVTYPE, ignore.case=TRUE)] <- "HEAT"
event_clean$EVTYPE[grep("lake.*flood", event_clean$EVTYPE, ignore.case=TRUE)] <- "LAKESHORE FLOOD"
```
##### Now I look at the cleaned list and look for any events that are not categorized.
##### Row 173 is labeled "?". Label is trivial.

```r
event_clean <- event_clean[-173,]
```
##### Row 174 "Apache County" doesn't have a category.

```r
event_clean <- event_clean[-174,]
```
##### Row 290 is labeled "High". Label is ambiguous.

```r
event_clean <- event_clean[-289,]
```
##### Row 351 is labeled "Other". It doesn't belong in any category.

```r
event_clean <- event_clean[-350,]
```
##### Row 351 is labeled "Other". It doesn't belong in any category.

```r
event_clean <- event_clean[-350,]
```

##### Get the sum of fatalities, injuries, property damage estimate and crop damage estimate grouped by event type.

```r
event_clean_grouped <- event_clean %>%
        group_by(EVTYPE) %>%
        summarise_each(funs(sum), FATALITIES, INJURIES, PROPDMGEST, CROPDMGEST)
```
#### Sort events in descending order by fatality and injury counts.

```r
event_fi_sorted <- event_clean_grouped %>%
        rename() %>%
        mutate(FI = FATALITIES + INJURIES) %>%
        arrange(desc(FI))
```
#### Sort events in descending order by property damage and crop damage estimate.

```r
event_pc_sorted <- event_clean_grouped %>%
        rename() %>%
        mutate(PC = PROPDMGEST + CROPDMGEST) %>%
        arrange(desc(PC))
```
## Results:

```r
# Bar graph showing number of fatalities and injuries with respect to storm events
event_pop_health <- melt(event_fi_sorted[1:10,], id="EVTYPE", measure=c("FATALITIES", "INJURIES"))
g <- ggplot(event_pop_health, aes(EVTYPE, value, fill=variable))
g + geom_bar(stat="identity", position="dodge") +
        theme(axis.text.x  = element_text(angle=90, size=10)) +
        theme(title = element_text(size=10)) +
        xlab("Storm event") +
        ylab("Counts") +
        ggtitle("Number of fatalities and injuries in storm events during 1950-2011")
```

![plot of chunk unnamed-chunk-22](figure/unnamed-chunk-22-1.png) 

#### Looking at the bar graph, a red bar shows fatalities and a blue bar shows injuries.  It's apparent from the graph that Tornado is #1 in both the fatality and injury counts.  5,661 fatality cases and 91,407 injury counts are the most in both categories.  Other notable storm events with high fatality counts are Thunderstorm Wind, Excessive Heat and Flood.  Excessive Heat has the second most injury counts.  My analysis shows that Tornado is the most harmful event with respect to population health.

```r
# Bar graph showing property damage estimate and crop damage estimate with respect to storm events
event_crop_dmg <- melt(event_pc_sorted[1:10,], id="EVTYPE", measure=c("PROPDMGEST", "CROPDMGEST"))
h <- ggplot(event_crop_dmg, aes(EVTYPE, value, fill=variable))
h + geom_bar(stat="identity", position="dodge") +
        theme(axis.text.x = element_text(angle=90, size=10)) +
        theme(title = element_text(size=10)) +
        xlab("Storm event") +
        ylab("Estimate ($)") +
        ggtitle("Estimate of property damage and crop damage in storm events during 1950-2011")
```

![plot of chunk unnamed-chunk-23](figure/unnamed-chunk-23-1.png) 

#### Looking at the bar graph, a red bar shows property damage estimate and a blue bar shows crop damage estimate.  Flood is the #1 in property damage cause with $150,229,813 and drought is the #1 in crop damage cause with $10,856,314.  Other notable events with high property damage is Hurricane (Typhoon), Storm Surge/Tide and Tornado.  Flood has the second highest crop damage estimate.  My analysis shows that Flood has the greatest economic consequences.
