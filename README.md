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
### 🧗‍♀️ [Act](#6-act)

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
# Bar graph for daily activity data recording by day of the week
```
ggplot(data = daily_activity, aes(x = Weekday)) +
  geom_bar(fill = "steelblue") +
  labs(title = "Daily Activity Data Recording by Day of the Week",
       x = "Day of the Week",
       y = "Count")

```

Step 6: Merge the Tables
# Merge daily_activity and sleep_day
```
merged_activity_sleep <- merge(daily_activity, sleep_day, by = c("Id", "date"), all = TRUE)

# Merge the result with weight
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
  geom_bar(fill="steelblue")
```
![image](https://user-images.githubusercontent.com/62857660/136279088-a4c39b17-990b-4d7f-9092-7307d1c9a37b.png)


⛔ From weekday's total asleep minutes, we can see the graph look almost **same** as the graph above! We can confirmed that most sleep data is also recorded during Tuesday to Thursday. This raised a question "how comprehensive are the data to form an accurate analysis?"

![image](https://user-images.githubusercontent.com/62857660/136279328-61059fcf-1554-478a-9dd6-680d243df486.png)

Merge the three tables:
```
merged_data <- merge(merged_activity_sleep, weight, by = c("Id"), all=TRUE)
```

Clean the data to prepare for analysis in 4. Analyze!

## 4. Analyze
[Back to Top](#author-emi-ly)

-  [Summary](#summary)
-  [Active Minutes](#active-minutes)
-  [Noticebal Day](#noticeable-day)
-  [Total Steps](#total-steps)
-  [Interesting Finds](#interesting-finds)
-  [Sleep](#sleep)


### Summary:
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
![summary](https://user-images.githubusercontent.com/62857660/136262678-18377ce4-3443-48a4-b108-eba6a273f963.PNG)

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

![newplot](https://user-images.githubusercontent.com/62857660/136252582-96e1f52a-dfe0-4247-a882-82d179d9b2b9.png)


The American Heart Association and World Health Organization recommend at least 150 minutes of moderate-intensity activity or 75 minutes of vigorous activity, or a combination of both, each week. That means it needs an daily goal of 21.4 minutes of FairlyActiveMinutes or 10.7 minutes of VeryActiveMinutes.

In our dataset, **30 users** met fairly active minutes or very active minutes.
```
active_users <- daily_activity %>%
  filter(FairlyActiveMinutes >= 21.4 | VeryActiveMinutes>=10.7) %>% 
  group_by(Id) %>% 
  count(Id) 
```

### Noticeable Day:
[Back to Analyze](#4-analyze)

The bar graph shows that there is a jump on Saturday: user spent LESS time in sedentary minutes and take MORE steps. Users are out and about on Saturday. 

![image](https://user-images.githubusercontent.com/62857660/136281021-fea4e732-982c-4404-8650-c3e943076856.png)
![image](https://user-images.githubusercontent.com/62857660/136281035-2b190f9e-a5a5-44c2-8183-20b1f7441e8d.png)





### Total Steps:
[Back to Analyze](#4-analyze)

Let's look at how active the users are per hourly in total steps. From 5PM to 7PM the users take the most steps. 
```
ggplot(data=hourly_step, aes(x=Hour, y=StepTotal, fill=Hour))+
  geom_bar(stat="identity")+
  labs(title="Hourly Steps")
```
![image](https://user-images.githubusercontent.com/62857660/136235391-bb22c15d-93aa-494d-bce2-76a984274fb7.png)


How active the users are weekly in total steps. Tuesday and Saturdays the users take the most steps. 
```
ggplot(data=merged_data, aes(x=Weekday, y=TotalSteps, fill=Weekday))+ 
  geom_bar(stat="identity")+
  ylab("Total Steps")
```
![image](https://user-images.githubusercontent.com/62857660/136252217-53d355de-2c25-4185-8e6d-27ba087573ae.png)




### Interesting Finds:
[Back to Analyze](#4-analyze)

The more active that you're, the more steps you take, and the more calories you will burn. This is an obvious fact, but we can still look into the data to find any interesting. Here we see that some users who are sedentary, take minimal steps, but still able to burn over 1500 to 2500 calories compare to users who are more active, take more steps, but still burn similar calories.

```
ggplot(data=daily_activity, aes(x=TotalSteps, y = Calories, color=SedentaryMinutes))+ 
  geom_point()+ 
  stat_smooth(method=lm)+
  scale_color_gradient(low="steelblue", high="orange")

```
![image](https://user-images.githubusercontent.com/62857660/136260311-a379b303-76ac-426c-9c30-ea2695569632.png)

Comparing the four active levels to the total steps, we see most data is concentrated on users who take about 5000 to 15000 steps a day. These users spent an average between 8 to 13 hours in sedentary, 5 hours in lightly active, and 1 to 2 hour for fairly and very active. 

![image](https://user-images.githubusercontent.com/62857660/136269396-7019cf93-6e0c-4216-9944-5e58e017f593.png)

According to [this healthline.com article](https://www.healthline.com/nutrition/how-many-calories-per-day#average-calorie-needs), moderately active woman between the ages of 26–50 needs to eat about 2,000 calories per day and moderately active man between the ages of 26–45 needs 2,600 calories per day to maintain his weight. Comparing the four active levels to the calories, we see most data is concentrated on users who burn 2000 to 3000 calories a day. These users also spent an average between 8 to 13 hours in sedentary, 5 hours in lightly active, and 1 to 2 hour for fairly and very active. Additionally, we see that the sedentary line is leveling off toward the end while fairly + very active line is curing back up. This indicate that the users who burn more calories spend less time in sedentary, more time in fairly + active. 

![image](https://user-images.githubusercontent.com/62857660/136263632-ac5c1958-23db-4374-b810-df6f322b047b.png)

### Sleep:
[Back to Analyze](#4-analyze)

According to article: [Fitbit Sleep Study](https://blog.fitbit.com/sleep-study/#:~:text=The%20average%20Fitbit%20user%20is,is%20spent%20restless%20or%20awake.&text=People%20who%20sleep%205%20hours,the%20beginning%20of%20the%20night.), 55 minutes are spent awake in bed before going to sleep. We have 13 users in our dataset spend 55 minutes awake before alseep. 

```
awake_in_bed <- mutate(sleep_day, AwakeTime = TotalTimeInBed - TotalMinutesAsleep)
awake_in_bed <- awake_in_bed %>% 
  filter(AwakeTime >= 55) %>% 
  group_by(Id) %>% 
  arrange(AwakeTime) 
```

We can use regression analysis look at the variables and correlation. For R-squared, 0% indicates that the model explains none of the variability of the response data around its mean. Higher % indicates that the model explains more of the variability of the response data around its mean. Postive slope means variables increase/decrease with each other, and negative means one variable go up and the other go down. We want to look at if users who spend more time in sedentary minutes spend more time sleeping as well. We can use regression analysis ```lm()``` to check for the dependent and indepedent variables. We also find that how many minutes an user asleep have an very weak correlation with how long they spend in sedentary minutes during the day.  
```
sedentary_vs_sleep.mod <- lm(SedentaryMinutes ~ TotalMinutesAsleep, data = merged_data)
summary(sedentary_vs_sleep.mod)
```
![calvssteps2](https://user-images.githubusercontent.com/62857660/136107919-65c86392-4f12-4038-b3d3-09166d8d5381.PNG)

How about calories vs asleep? Do people sleep more burn less calories? Plotting the two variables we can see that there is not much a correlation. 
```
ggplot(data=merged_data, aes(x=TotalMinutesAsleep, y = Calories, color=TotalMinutesAsleep))+ 
  geom_point()+ 
  labs(title="Total Minutes Asleep vs Calories")+
  xlab("Total Minutes Alseep")+
  stat_smooth(method=lm)+
  scale_color_gradient(low="orange", high="steelblue")
```
![image](https://user-images.githubusercontent.com/62857660/136283073-360f9a07-e4ef-4307-9c65-4b877f62e58b.png)



## 5. Share 


### 🎨 [Bellabeat Data Analysis Dashboard](https://public.tableau.com/app/profile/heena.begum4134/viz/Bellabeat_17198883873430/Dashboard1?publish=yes)



## 6. Act


Conclusion based on our analysis:
- Sedentary make up a significant portion, 81% of users daily active minutes. Users spend on avg 12 hours a day in sedentary minutes, 4 hours lightly active, and only half-hour in fairly+very active! 
- We see the most change on Saturday: users take more steps, burn more calories, and spend less time sedentary. Sunday is the most "lazy" day for users. 
- 54% of the users who recorded their sleep data spent 55 minutes awake in bed before falling asleep.
- Users takes the most steps from 5 PM to 7 PM
Users who are sedentary take minimal steps and burn 1500 to 2500 calories compared to users who are more active, take more steps, but still burn similar calories.



Marketing recommendations to expand globally:

##### 🔢  Obtain more data for an accurate analysis, encouraging users to use a wifi-connected scale instead of manual weight entries. 

##### 🚲  Educational healthy style campaign encourages users to have short active exercises during the week, longer during the weekends, especially on Sunday where we see the lowest steps and most sedentary minutes.

##### 🎁  Educational healthy style campaign can pair with a point-award incentive system. Users completing the whole week's exercise will receive Bellabeat points on products/memberships.

##### 🏃‍♂️ The product, such as Leaf wellness tracker, can beat or vibrate after a prolonged period of sedentary minutes, signaling the user it's time to get active! Similarly, it can also remind the user it's time to sleep after sensing a prolonged awake time in bed.










