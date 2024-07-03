# CASE STUDY: Bellabeat Fitness Data Analysis 
# Author: Heena Begum

# Date: June 27, 2024

# [Tableau Dashboard](https://public.tableau.com/app/profile/heena.begum4134/viz/Bellabeat_17198883873430/Dashboard1?publish=yes)

##### [Tableau Story Presentation to Skateholders](https://public.tableau.com/app/profile/emily.liang7497/viz/BellabeatFitnessDataAnalysis-GoogleDataAnalyticsCapstone/Story1)

#

_The case study follows the six step data analysis process:_

### ❓ [Ask](#1-ask)
### 💻 [Prepare](#2-prepare)
### 🛠 [Process](#3-process)
### 📊 [Analyze](#4-analyze)
### 📋 [Share](#5-share)
### 🧗‍♀️ [Act](#6-Recommendations)

## Scenario
Founded in 2013, Bellabeat has rapidly grown into a leading tech-driven wellness company for women. The company offers five core products: the Bellabeat app, Leaf, Time, Spring, and Bellabeat Membership. While already successful, Bellabeat has the potential to expand significantly in the global smart device market. Our team has been tasked with analyzing smart device data to uncover insights into consumer usage patterns. These insights will inform and enhance the company's marketing strategy.

## 1. Ask
💡 **BUSINESS TASK: Analyze Fitbit data to gain insight and help guide marketing strategy for Bellabeat to grow as a global player.**

Primary stakeholders: Urška Sršen and Sando Mur, executive team members.

Secondary stakeholders: Bellabeat marketing analytics team.

## 2. Prepare 
Data Source: 30 participants FitBit Fitness Tracker Data from Mobius: https://www.kaggle.com/arashnic/fitbit

The dataset has 29 CSV. 

Reliability: The data was collected from 30 FitBit users who provided informed consent for the submission of their personal tracker data via Amazon Mechanical Turk.
Originality: The dataset is derived from original submissions by 30 FitBit users through Amazon Mechanical Turk.
Comprehensiveness: The dataset provides minute-level output for physical activity, heart rate, and sleep monitoring. It tracks various aspects of user activity and sleep, but the sample size is small and most data is recorded on specific days of the week.
Currency: The data was collected between March 2016 and May 2016. It is not current, and users' habits may have changed since then.
Citation: The source of the dataset is not specified. 

⛔The dataset has several limitations:

Sample Size: The data includes only 30 users. While the central limit theorem suggests that n ≥ 30 is acceptable for using the t-test for statistical reference, a larger sample size is preferred for more robust analysis.
Inconsistent User Data: Upon further investigation using n_distinct() to check for unique user IDs, it was found that the dataset includes 33 users in the daily activity data, 24 users in the sleep data, and only 8 users in the weight data. This indicates that there are 3 additional users and some users did not consistently record their daily activity and sleep data.
Weight Data: For the 8 users who recorded weight data, 5 users manually entered their weight, while 3 recorded it via a connected Wi-Fi device (e.g., Wi-Fi scale).
Data Recording Days: Most data is recorded from Tuesday to Thursday, which may not provide a comprehensive enough view to form an accurate analysis.
## 3. Process
[Back to Top](#author-emi-ly)

Examine the data, check for NA, and remove duplicates for three main tables: daily_activity, sleep_day and weight:

Step 1: Examine Data, Check for NA, and Remove Duplicates

```
# Load necessary libraries
library(dplyr)
library(ggplot2)
```
```
# Check dimensions, NA values, and duplicates for sleep_day
dim(sleep_day)
sum(is.na(sleep_day))
sum(duplicated(sleep_day))
sleep_day <- sleep_day[!duplicated(sleep_day), ]
```
```
# Check dimensions, NA values, and duplicates for daily_activity
dim(daily_activity)
sum(is.na(daily_activity))
sum(duplicated(daily_activity))
daily_activity <- daily_activity[!duplicated(daily_activity), ]
```

Convert ActivityDate into date format and add a column for day of the week:
```
daily_activity <- daily_activity %>% mutate( Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))
```


Check to see if we have 30 users using ```n_distinct()```. The dataset has 33 user data from daily activity, 24 from sleep and only 8 from weight. If there is a discrepency such as in the weight table, check to see how the data are recorded. The way the user record the data may give you insight on why there is missing data. 
```
# Check dimensions, NA values, and duplicates for weight
dim(weight)
sum(is.na(weight))
sum(duplicated(weight))
weight <- weight[!duplicated(weight), ]
 ```

 Step 2: Convert ActivityDate into Date Format and Add Day of the Week
 ```
 daily_activity <- daily_activity %>%
  mutate(ActivityDate = as.Date(ActivityDate, "%m/%d/%Y"),
         Weekday = weekdays(ActivityDate))
```

 Step 3: Check Number of Unique User
 ```
n_distinct(daily_activity$Id)
n_distinct(sleep_day$Id)
n_distinct(weight$Id)

```
 Step 4: Investigate Discrepancies in Weight Data
 ```
 weight %>%
  filter(IsManualReport == "True") %>%
  group_by(Id) %>%
  summarise("Manual Weight Report" = n()) %>%
  distinct()

weight %>%
  filter(IsManualReport == "False") %>%
  group_by(Id) %>%
  summarise("Automatic Weight Report" = n()) %>%
  distinct()

```
Step 5: Investigate Data Recording Distribution

```
ggplot(data = daily_activity, aes(x = Weekday)) +
  geom_bar(fill = "steelblue") +
  labs(title = "Daily Activity Data Recording by Day of the Week",
       x = "Day of the Week",
       y = "Count")

```

Step 6: Merge the Tables

```
merged_activity_sleep <- merge(daily_activity, sleep_day, by = c("Id", "date"), all = TRUE)


merged_data <- merge(merged_activity_sleep, weight, by = "Id", all = TRUE)
```
Step 7: Clean and Prepare Data for Analysis
```
# Further cleaning if necessary
# For example, you may want to handle NA values or specific columns

# Display the first few rows of the cleaned and merged data
head(merged_data)
```


Additional insight to be awared of is how often user record their data. We can see from the ```ggplot()``` bar graph that the data are greatest from Tuesday to Thursday. We need to investigate the data recording distribution. Monday and Friday are both weekdays, why isn't the data recordings as much as the other weekdays? 
```
ggplot(data=merged_data, aes(x=Weekday))+
  geom_bar(fill="orchid1")
```
![image](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/dailyac.png)


⛔ From weekday's total asleep minutes, we can see the graph look almost **same** as the graph above! We can confirmed that most sleep data is also recorded during Tuesday to Thursday. This raised a question "how comprehensive are the data to form an accurate analysis?"

![image](https://user-images.githubusercontent.com/62857660/136279328-61059fcf-1554-478a-9dd6-680d243df486.png)

Merge the three tables:
```
merged_data <- merge(merged_activity_sleep, weight, by = c("Id"), all=TRUE)
```

Clean the data to prepare for analysis in 4. Analyze!

## 4. Analyze


-  [Summary](#summary)
-  [Active Minutes](#active-minutes)
-  [Noticebal Day](#noticeable-day)
-  [Total Steps](#total-steps)
-  [Interesting Finds](#interesting-finds)
-  [Sleep](#sleep)


### Summary:

Graph variables of interest, check for outliers in the data
```
 summary(activity$TotalSteps)
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      0    3790    7406    7638   10727   36019
```

Check min, max, mean, median and any outliers. Avg weight is 135 pounds with BMI of 24 and burn 2050 calories. Avg steps is 10200, max is almost triple that 36000 steps. Users spend on avg 12 hours a day in sedentary minutes, 4 hours lightly active, only half hour in fairly+very active! Users also gets about 7 hour of sleep. 
```
merged_data %>%
  dplyr::select(Weekday,
         TotalSteps,
         TotalDistance,
         VeryActiveMinutes,
         FairlyActiveMinutes,
         LightlyActiveMinutes,
         SedentaryMinutes,
         Calories,
         TotalMinutesAsleep,
         TotalTimeInBed,
         WeightPounds,
         BMI
         ) %>%
  summary()
```

### Active Minutes:
[Back to Analyze](#4-analyze)

Percentage of active minutes in the four categories: very active, fairly active, lightly active and sedentary. From the pie chart, we can see that most users spent 81.3% of their daily activity in sedentary minutes and only 1.74% in very active minutes. 
```
percentage <- data.frame(
  level=c("Sedentary", "Lightly", "Fairly", "Very Active"),
  minutes=c(sedentary_percentage,lightly_percentage,fairly_percentage,active_percentage)
)

plot_ly(percentage, labels = ~level, values = ~minutes, type = 'pie',textposition = 'outside',textinfo = 'label+percent') %>%
  layout(title = 'Activity Level Minutes',
         xaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE),
         yaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE))
```

![newplot](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/pie-chart%20(1).svg)


The American Heart Association and World Health Organization recommend at least 150 minutes of moderate-intensity activity or 75 minutes of vigorous activity, or a combination of both, each week. That means it needs an daily goal of 21.4 minutes of FairlyActiveMinutes or 10.7 minutes of VeryActiveMinutes.

In our dataset, **30 users** met fairly active minutes or very active minutes.


### Noticeable Day:
[Back to Analyze](#4-analyze)

The bar graph shows that there is a jump on Saturday: user spent LESS time in sedentary minutes and take MORE steps. Users are out and about on Saturday. 

![image](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/totalsteps.png)





### Total Steps:
[Back to Analyze](#4-analyze)

Let's look at how active the users are per hourly in total steps. From 5PM to 7PM the users take the most steps. 

![image](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/totalsteps.png)


How active the users are weekly in total steps. Tuesday and Saturdays the users take the most steps. 

![image](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/hourlysteps.png))

## 5. Share 



### 🎨 [Bellabeat Data Analysis Dashboard](https://public.tableau.com/app/profile/heena.begum4134/viz/Bellabeat_17198883873430/Dashboard1?publish=yes)
![image](https://github.com/HeenaBegum/Google-Data-analytics-Bella-Beat-case-study/blob/blogheena/Dashboard%201.png)

Based on the descriptive statistical analyses and visualizations, the following trends in smart device usage were observed:

Sedentary Behavior: Sedentary minutes dominated participants' daily routines and remained consistent across the week.

Very Active Minutes: Average "very active minutes" were relatively stable throughout the week, averaging around 20 minutes per day.

Sleep Patterns: Participants tended to sleep the most on Sundays, coinciding with their lowest step count on that day.

Steps Taken: Tuesdays and Saturdays saw the highest average number of steps taken by participants.

Time of Day Steps: The least number of steps were typically taken at 3:00, while 18:00 recorded the highest average step count.

Sleep Duration: On average, participants slept approximately 390 minutes per night, equivalent to 6.5 hours.

Steps and Active Minutes: Users who took more steps per day were more likely to engage in "very active minutes."

These insights suggest opportunities for Bellabeat to tailor marketing strategies, focusing on encouraging more active behaviors and optimizing sleep patterns based on user data patterns throughout the week.

## 6. Recommendations 
How can these trends help influence Bellabeat's marketing strategy?

Weight Log Feature: Very few customers utilized the weight log feature, indicating it may not be a significant selling point. Bellabeat should focus on marketing other features such as activity, sleep, and steps tracking. Further research into enhancing the appeal of the weight log feature could potentially make it more marketable.

Activity Levels: Our data shows that participants engage the most in "light" activity and have fewer "very active" minutes each day. Bellabeat could introduce a "level up" feature where users earn points based on active minutes. Higher levels of activity could lead to greater rewards, motivating users to increase their activity levels consistently.

Sunday Activity: There is a notable 1000-step decrease on Sundays compared to other days. Bellabeat could send notifications on Sunday mornings setting a step goal for the day, coupled with rewards for achieving a 7-day streak. This approach can help bridge the gap and encourage users to utilize the device throughout the entire week.

Peak Activity Times: Data indicates peak device usage around 6 pm, suggesting users are most active after typical work hours. Bellabeat can target working adults with ads focusing on seamless step tracking during busy days. Additionally, reminder notifications around 12 pm and 8 pm could prompt users to increase their activity during lunch breaks and after dinner.

Sleep Tracking: On average, participants are getting less than the CDC recommended 7 hours of sleep per night. Bellabeat should continue marketing the device's sleep tracking feature, emphasizing its utility for users seeking to monitor and improve their sleep patterns. Collaborating with a meditation app or habit tracker could enhance the appeal further.

These recommendations leverage insights from user data to tailor Bellabeat's marketing strategy effectively, promoting engagement and satisfaction among users




