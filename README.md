# How Can a Wellness Technology Company Play It Smart?
## Capstone Project for Google Data Analytics Specialisation

## Background:
Bellabeat is a wellness tech company that empowers women through data-driven insights and innovative products. Founded in 2013, Bellabeat offers smart devices like fitness trackers and health monitors to help women improve their overall well-being with personalized recommendations.

The cofounders of Bellabeat are eager to delve into data from non-Bellabeat fitness devices to uncover how consumers are using these products. By gaining these insights, the company aims to craft innovative marketing strategies that will captivate and engage their target audience even more effectively.

### Products
* Bellabeat app: The Bellabeat app provides users with health data related to their activity, sleep, stress,
menstrual cycle, and mindfulness habits. This data can help users better understand their current habits and
make healthy decisions. The Bellabeat app connects to their line of smart wellness products.
* Leaf: Bellabeat’s classic wellness tracker can be worn as a bracelet, necklace, or clip. The Leaf tracker connects
to the Bellabeat app to track activity, sleep, and stress.
* Time: This wellness watch combines the timeless look of a classic timepiece with smart technology to track user
activity, sleep, and stress. The Time watch connects to the Bellabeat app to provide you with insights into your
daily wellness.
* Spring: This is a water bottle that tracks daily water intake using smart technology to ensure that you are
appropriately hydrated throughout the day. The Spring bottle connects to the Bellabeat app to track your
hydration levels.
* Bellabeat membership: Bellabeat also offers a subscription-based membership program for users.
Membership gives users 24/7 access to fully personalized guidance on nutrition, activity, sleep, health and
beauty, and mindfulness based on their lifestyle and goals.

## Step 1: Ask

### 1.1: Business Task 
To uncover insights that will inform their marketing strategies, Bellabeat plans to 
analyze consumer data from non-Bellabeat smart devices.

### 1.2: Stakeholders
1. Urška Sršen: Bellabeat’s cofounder and Chief Creative Officer
2. Sando Mur: Mathematician and Bellabeat’s cofounder; key member of the Bellabeat executive team
3. Bellabeat marketing analytics team: A team of data analysts responsible for collecting, analyzing, and reporting
data that helps guide Bellabeat’s marketing strategy.

### 1.3: Guiding Questions:
What are the trends in fitness trackers in the market?
How do these trends apply to Bellabeat customers?
How do we use these trends to shape the marketing strategy for Bellabeat?

## Step 2: Prepare

### 2.1: Data
FitBit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius)

* Description:
This Kaggle data set contains personal fitness tracker from thirty fitbit users. Thirty eligible Fitbit users consented to the submission of
personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes
information about daily activity, steps, and heart rate that can be used to explore users’ habits.

This dataset generated by respondents to a distributed survey via Amazon Mechanical Turk between 03.12.2016-05.12.2016. Variation between output represents use of different types of Fitbit trackers and individual tracking behaviors / preferences.

### 2.2: Data Organisation
There are 18 CSV files storing data.  Fitbit tracks a variety of different quantitative data and represents them in each CSV file.

The data is in long format. Each row in the dataset corresponds to a specific time point for a subject, meaning each user has multiple entries. Users are identified by unique IDs, with their data recorded by day and time.

### 2.3: Data Bias & Credibility

1. There is a limitation of 30 users. This means that the data may not be fully representative of the population we are trying to analyze/
2. We do not know further demographic details about the users who were surveyed, so there may have been a sampling bias.
3. We do not know how the survey was conducted and the specific questions that were asked. There could have been leading questions, which might influence the responses and skew the results.
4. The dataset is outdated, which may not reflect current user behaviors and trends.
5. The survey's limited duration of 2 months might not capture longer-term patterns and variations in user behavior.

### 2.4: Licensing, Privacy, Security & Accessibility

The dataset is open-source, meaning it is freely available for use, modification, and distribution under an open-source license. This promotes transparency and collaboration, allowing researchers and developers to leverage the data for various analyses and applications.

The data is anonymized as well, as we do not have any accessed to the identifying personal information of the users directly from the dataset.

## Step 3: Process

I am using R as my primary tool of data analysis, as the dataset is rather large and I want to have the ease of creating visualisations as well. 

### 3.1: Files Chosen:

For a more logical and convenient timeframe, I will select the datasets that are only by the day. All other datasets involving seconds or minutes would not be enough to provide a macro view of any interesting relationships / trends.

Hence, I have chosen the following 5 datasets:

1. dailyActivity_merged.csv
2. dailyCalories_merged.csv
3. dailyIntensities_merged.csv
4. dailySteps_merged.csv
5. sleepDay_merged.csv
    
---

### 3.3 Data Set Up and Cleaning
1. Load the tidyverse packages

```
library(tidyverse)
library(here)
library(lubridate)
library(janitor)
library(skimr)
```

2. Load files
   
```
    activities <- read_csv("dailyActivity_merged.csv")
    calories <- read_csv("dailyCalories_merged.csv")
    intensities <- read_csv("dailyIntensities_merged.csv")
    steps <- read_csv("dailySteps_merged.csv")
    sleeps <- read_csv("sleepDay_merged.csv")

```

3. Check if data has been loaded

```
    head(activities)
    head(calories)
    head(intensities)
    head(steps)
    head(sleeps)
```

4. Check structure of data sets

```
    str(activities)
    str(calories)
    str(intensities)
    str(steps)
    str(sleeps)
```

5. Check Unique Rows

Now I will check the unique users per tibble

```
n_unique(activities$Id)
n_unique(calories$Id)
n_unique(intensities$Id)
n_unique(steps$Id)
n_unique(sleeps$Id)
```

From the result, I can see that there are 33 unique users per tibble, except for sleeps (24) and weights (8).
I will drop the weights dataset as it is too small to make appropriate conclusions representative of the population.

Even though the sleeps dataset is still less than 30, which is the recommended minimum sample size, in this case, 
I feel like 24 is a number that is relatively close enough to 33, which is the maximum sample size out of all the 
datasets. Furthermore, I believe that the sleeps dataset provides useful insights about participants' sleep schedules which we may 
take advantage of in our marketing strategy.

6. Remove duplicates and drop any null values

```
activities <- activities %>%
  distinct() %>%
  drop_na()
calories <- calories %>%
  distinct() %>%
  drop_na()
intensities <- intensities %>%
  distinct() %>%
  drop_na()
steps <- steps %>%
  distinct() %>%
  drop_na()
sleeps <- sleeps %>%
  distinct() %>%
  drop_na()
```


7. Clean the column names

```
clean_names(activities)
clean_names(calories)
clean_names(intensities)
clean_names(steps)
clean_names(sleeps)
```

8. Make the column names consistent

```
activities <- rename_with(activities, tolower)
calories <- rename_with(calories, tolower)
intensities <- rename_with(intensities, tolower)
steps <- rename_with(steps, tolower)
sleeps <- rename_with(sleeps, tolower)
```

9. Check for any outliers and remove them

```
# Function to remove outliers using the IQR method
remove_outliers <- function(df, cols) {
  for (col in cols) {
    Q1 <- quantile(df[[col]], 0.25, na.rm = TRUE)
    Q3 <- quantile(df[[col]], 0.75, na.rm = TRUE)
    IQR <- Q3 - Q1
    lower_bound <- Q1 - 1.5 * IQR
    upper_bound <- Q3 + 1.5 * IQR
    
    df <- df[df[[col]] >= lower_bound & df[[col]] <= upper_bound, ]
  }
  return(df)
}

# Select relevant columns
activity_data <- activities[, c("totalsteps", "sedentaryminutes", "lightlyactiveminutes", 
                                "fairlyactiveminutes", "veryactiveminutes", "calories")]

# Remove outliers
cleaned_activity_data <- remove_outliers(activity_data, c("totalsteps", "sedentaryminutes", 
                                                          "lightlyactiveminutes", "fairlyactiveminutes", 
                                                          "veryactiveminutes", "calories"))

# Function to create box plots for each variable before and after outlier removal
create_boxplots <- function(before_data, after_data, variable) {
  p1 <- ggplot(before_data, aes_string(y = variable)) +
    geom_boxplot(fill = "blue", alpha = 0.6) +
    labs(title = paste(variable, " - Before Outlier Removal"),
         y = variable) +
    theme_minimal()
  
  p2 <- ggplot(after_data, aes_string(y = variable)) +
    geom_boxplot(fill = "green", alpha = 0.6) +
    labs(title = paste(variable, " - After Outlier Removal"),
         y = variable) +
    theme_minimal()
  
  return(list(p1, p2))
}

# Create box plots for each variable and store them in a list
plots_list <- list()
variables <- c("totalsteps", "sedentaryminutes", "lightlyactiveminutes", 
               "fairlyactiveminutes", "veryactiveminutes", "calories")

for (var in variables) {
  plots <- create_boxplots(activity_data, cleaned_activity_data, var)
  plots_list <- c(plots_list, plots)
}

# Arrange and display the plots
do.call("grid.arrange", c(plots_list, ncol = 2))
```

## Step 4 & 5: Analyse & Share
An analysis of the datasets will be done to discover any meaningful trends which can be used to guide Bellabeat's
marketing strategy

### 5.1 Activities dataset

```
glimpse(activities)
```

Let us get the summary statistics for activities. We will be selecting totalsteps, sedentaryminutes, 
                                                                        lightlyactiveminutes, fairlyactiveminutes,
                                                                        veryactiveminutes and calories columns.

```
cleaned_activity_data %>% 
  select(totalsteps,
         sedentaryminutes,
         lightlyactiveminutes,
         fairlyactiveminutes,
         veryactiveminutes,
         calories) %>% 
  summary()
```

From the above, we can tell that:

1. The average steps taken per day was 6727, which is well below CDC's reccomended 10,000 steps per day (Kling et al., 2016). 
2. Average sedentary minutes was 1012, average light active minutes was 192.4, average fairly active minutes was 9.501 and average very active minutes was 
14.01. This shows that on average, users spend way more time being sedentary than being active. 


Next, we will plot a heatmap between all the variables in the activities dataset to spot any string correlations between the variables that are worth investigating.

```
# Get the correlation matrix 
# Calculate the correlation matrix
correlation_matrix <- cor(cleaned_activity_data, use = "complete.obs")

# Plot the heatmap using ggcorrplot
ggcorrplot(correlation_matrix, 
           hc.order = TRUE, 
           type = "lower", 
           lab = TRUE, 
           lab_size = 3, 
           method="circle", 
           colors = c("blue", "white", "red"), 
           title = "Correlation Heatmap",
           ggtheme = theme_minimal())

```

Observations: 
From what I can observe, the correlation coefficients range from -0.5 to 0.68.

Based on the results, I have determined that any correlation value above 0.60 is considered to be a strong relationship and they warrant a further investigation. Hence, I will investigate the relationships between lightlyactiveminutes and totalsteps (0.68) and also between veryactiveminutes and totalsteps (0.66).

Both correlation values are positive, indicating that each relationship is a positive one.

### Relationship between lightlyactiveminutes and totalsteps
We will plot a scatterplot between the two variables to investigate the relationship further.

```
ggplot(cleaned_activity_data, aes(x = lightlyactiveminutes, y = totalsteps)) +
  geom_point(color = "blue", alpha = 0.6) +
  geom_smooth(method = "lm", color = "red", se = FALSE) +
  labs(title = "Lightly Active Minutes vs. Total Steps",
       x = "Lightly Active Minutes",
       y = "Total Steps") +
  theme_minimal()
```

From what I can see, there is a positive relationship between lightlyactiveminutes and totalsteps. As lightlyactiveminutes increases, totalsteps will also increase.

### Relationship between veryactiveminutes and totalsteps
We will plot another scatterplot between the two variables to investigate the relationship further.

```
ggplot(cleaned_activity_data, aes(x = veryactiveminutes, y = totalsteps)) +
  geom_point(color = "blue", alpha = 0.6) +
  geom_smooth(method = "lm", color = "red", se = FALSE) +
  labs(title = "Total Steps vs. Very Active Minutes",
       x = "Very Active Minutes",
       y = "Total Steps") +
  theme_minimal()
```

It seems that there is also a positive relationship between veryactiveminutes and totalsteps. As veryactiveminutes increases, totalsteps will also increase.

### Analysis of the results
Here, we can tell that as people spend more time being active, their total daily steps will also increase. Hence, more people should be encouraged to be more active so that their total daily steps can increase in order to lead healthier lifestyles.


### 5.2  calories dataset

```
glimpse(calories)
```

Firstly, I will convert the column of activityday from string type to Date objects in mm/dd/yyyy format

```
# Function to safely convert date strings to Date objects in mm/dd/yyyy format
safe_mdy <- function(date_str) {
  parsed_date <- mdy(date_str)
  if (is.na(parsed_date)) {
    warning(paste("Failed to parse date:", date_str))
  }
  return(parsed_date)
}

# Apply the safe_mdy function to the activityday column
calories$activityday <- sapply(calories$activityday, safe_mdy)

# Ensure the activityday column is of Date type
calories$activityday <- as.Date(calories$activityday, origin = "1970-01-01")

# Check for NA values which indicate failed parsing
failed_parsing <- calories[is.na(calories$activityday), ]

# Print failed parsing entries if any
if (nrow(failed_parsing) > 0) {
  print("Failed to parse the following dates:")
  print(failed_parsing)
} else {
  print("All dates parsed successfully.")
}
```

After which, we will extract the day of the week for each successfully parsed date

```
# Extract day of the week for successfully parsed dates
calories$day_of_week <- wday(calories$activityday, label = TRUE, abbr = FALSE)

# View the updated data
print(calories)
```

Now, I will create the barplot to visualise the data.

```
# Create the barplot
ggplot(calories, aes(x = day_of_week, y = calories)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  labs(title = "Total Calories by Day of the Week", x = "Day of the Week", y = "Total Calories") +
  theme_minimal()
```
### Analysis of the results
From the barplot, I can see that the greatest amount of calories were burnt on Tuesdays. The lowest were Sundays, which is expected as Sundays are often considered by many people to be rest days.

### 5.3 intensities dataset

```
glimpse(intensities)
```

Firstly, we will add another column by converting activityday column into days of the week column.

```
# Apply the safe_mdy function to the activityday column
intensities$activityday <- sapply(intensities$activityday, safe_mdy)

# Ensure the activityday column is of Date type
intensities$activityday <- as.Date(intensities$activityday, origin = "1970-01-01")

# Check for NA values which indicate failed parsing
failed_parsing <- intensities[is.na(intensities$activityday), ]

# Print failed parsing entries if any
if (nrow(failed_parsing) > 0) {
  print("Failed to parse the following dates:")
  print(failed_parsing)
} else {
  print("All dates parsed successfully.")
}

# Extract day of the week for successfully parsed dates
intensities$day_of_week <- wday(intensities$activityday, label = TRUE, abbr = FALSE)

# View the updated data
glimpse(intensities)
```
Now we will create a stacked barplot for the activity minutes against days of the week

Firstly, we will transform the data to long format.

```
# Transform the data to long format
long_data <- intensities %>%
  pivot_longer(cols = c(sedentaryminutes, lightlyactiveminutes, fairlyactiveminutes, veryactiveminutes),
               names_to = "activity_type", values_to = "minutes")
```

Now, we can plot the stacked barplot

```
# Plot the stacked bar plot
ggplot(long_data, aes(x = day_of_week, y = minutes, fill = activity_type)) +
  geom_bar(stat = "identity") +
  labs(title = "Activity Minutes by Day of the Week",
       x = "Day of the Week",
       y = "Minutes",
       fill = "Activity Type") +
  theme_minimal()
```

### Analysis of the results
We see that the non-sedentary minutes for people are generally the same across the days of the week. However, we see that sedentary minutes occupy a very large proportion of all total minutes across the week. Although this may be due to hours spent sleeping, resting or working, we should definitely look to reduce the sedentary minutes if possible as it occupies a disproportionately large amount of people's time. Interestingly, people tend to generally spend less time using FitBit devices on the weekends compare to the weekdays. It may be due to them bringing the devices to work on the weekdays, but not necessarily wearing them out on the weekends when they are not at work. We can look to improve the usage on the weekends.

Now, we will plot the stacked barplot of the activedistances against days of the week.

We will repeat the above process by transforming the data to long format first, and then plotting the stacked barplot

```
# Transform the data to long format
long_data <- intensities %>%
  pivot_longer(cols = c(sedentaryactivedistance, lightactivedistance, moderatelyactivedistance, veryactivedistance),
               names_to = "distance_type", values_to = "distance")

# Plot the stacked bar plot
ggplot(long_data, aes(x = day_of_week, y = distance, fill = distance_type)) +
  geom_bar(stat = "identity") +
  labs(title = "Activity Distance by Day of the Week",
       x = "Day of the Week",
       y = "Distance",
       fill = "Distance Type") +
  theme_minimal()
```

### Analysis of the results

From the stacked barplot, we can see that people generally do a larger proportion of light active distances compared to moderate of very active distances for their workouts across the week. As previously mentioned, people tend to be most active on Tuesdays but least active on Sundays. We can aim to increase the total active distance in general. Having a larger proportion of light active distances compared to moderate or very active distances is still okay as the users are still being active in some way, which is better than being sedentary. 


### 5.4 Steps dataset

```
glimpse(sleeps)
```

Now, we will investigate the steps dataset. Our goal is to plot a barplot of total steps against days of the week.

Again, we will repeat the procedure to convert the activityday column from chr type to days of the week

```
# Apply the safe_mdy function to the activityday column
steps$activityday <- sapply(steps$activityday, safe_mdy)

# Ensure the activityday column is of Date type
steps$activityday <- as.Date(steps$activityday, origin = "1970-01-01")

# Check for NA values which indicate failed parsing
failed_parsing <- steps[is.na(steps$activityday), ]

# Print failed parsing entries if any
if (nrow(failed_parsing) > 0) {
  print("Failed to parse the following dates:")
  print(failed_parsing)
} else {
  print("All dates parsed successfully.")
}

# Extract day of the week for successfully parsed dates
steps$day_of_week <- wday(steps$activityday, label = TRUE, abbr = FALSE)

# View the updated data
glimpse(steps)
```

We will now plot the barplot of steps against days of the week.

```
ggplot(steps, aes(x = day_of_week, y = steptotal)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  labs(title = "Total Steps by Day of the Week",
       x = "Day of the Week",
       y = "Total Steps") +
  theme_minimal()
```

The trend remains the same, with Tuesdays having the most totalsteps and Sundays having the least totalsteps. CDC reccommends a total of 10,000 steps per day (Kling et al., 2016), which roughly means that it is reccommended to have at least 70,000 steps per week. 

Since we have 33 unique rows, there are 33 unique participants. So the ideal total steps taken for each day should be 33 * 10000 = 330,000.

```
cdcRecommendedDailySteps <- 10000

# There are 33 unique rows in steps.

# Plot the bar plot with a red line at 330,000
ggplot(steps, aes(x = day_of_week, y = steptotal)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  geom_hline(yintercept = cdcRecommendedDailySteps * 33, color = "red", linetype = "dashed", size = 1) +
  labs(title = "Total Steps by Day of the Week",
       x = "Day of the Week",
       y = "Total Steps") +
  theme_minimal() +
  scale_y_continuous(labels = scales::comma)  # Adds comma formatting for y-axis labels
```

### Analysis of results
We can see that on average, the daily steps taken per person is well above the CDC's reccomended level.


### 5.5 Sleeps dataset

```
# Original dataset
glimpse(sleeps)
```

I will convert the sleepday column to days of the week

```
# Convert sleepday to Date-Time format and extract day of the week
sleeps <- sleeps %>%
  mutate(
    sleepday = mdy_hms(sleepday),  # Convert to POSIXct Date-Time format
    day_of_week = wday(sleepday, label = TRUE, abbr = FALSE)  # Extract the day of the week
  )

# After adding the days_of_week column
glimpse(sleeps)
```

Now, I will calculate the mean total minutes asleep for each day of the week before doing the barplot
```
# Calculate the mean total minutes asleep for each day of the week
sleeps_summary <- sleeps %>%
  group_by(day_of_week) %>%
  summarise(mean_total_minutes_asleep = mean(totalminutesasleep, na.rm = TRUE))
```

CDC reccomends that adults sleep 7 or more hours per day (Watson et al., 2015). This equates to 7 * 60 = 420 minutes of sleep per day.

We will plot the barplot of mean total minutes asleep against each day of the week.

```
# Plot the mean total minutes asleep with a reference line at 420 minutes and a label
ggplot(sleeps_summary, aes(x = day_of_week, y = mean_total_minutes_asleep)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  geom_hline(yintercept = 420, color = "red", linetype = "dashed", size = 1) +
  annotate("text", x = 1, y = 420, label = "CDC Recommended Min. Sleep", color = "red", vjust = -0.5, hjust = 0) +
  labs(title = "Mean Total Minutes Asleep by Day of the Week",
       x = "Day of the Week",
       y = "Mean Total Minutes Asleep") +
  theme_minimal() +
  scale_y_continuous(labels = scales::comma) 
```

We can see from the plot that on average, people are getting less than the CDC reccomended 7 hours of minimum sleep per day. Only Sundays and Wednesdays have on average a higher total minutes asleep than what the CDC reccommends. The rest of the 5 days of the week have on average less sleep duration than what the CDC reccomends daily.


## Step 6: Act Phase - Concluding Statements

Bellabeat is a wellness tech company that empowers women through data-driven insights and innovative products. 

I did an analysis of the FitBit dataset and investigated the relationship between active minutes and total steps. I also did an investigation of user calories, intensities, steps, minutes slept across the week.

Target Audience
Our primary audience consists of young and adult women with full-time jobs who engage in light activities to stay healthy. To develop an effective marketing strategy for this demographic, we should focus on trends that enhance their well-being and promote the development of healthy habits.

Recommendations for the Bellabeat App

Daily Step Notifications
The reccomended daily steps by CDC is 10,000 for a healthy lifestyle. Bellabeat can motivate users to aim for at least 10,000 steps daily by sending notifications at different times throughout the day. These notifications can remind users of their progress and encourage them to reach their goal. Additionally, the app can educate users on the health benefits of walking the recommended number of steps daily, these may include infographics or visual aids that can introduce activities that can help users keep fit and meet the reccomended daily steps.

Sleep Notifications
Our analysis reveals that users often get less than the recommended amount of sleep of 7 hours by the CDC. To address this, the app can include a feature that allows users to set a desired bedtime and receive a notification a few minutes before to help them prepare for sleep. 
An alarm could also be set to remind them to go to bed at their chosen time.
If more motivation is required, we can take inspiration from Pokemon Sleep, which is a sleep tracking app developed by the Pokemon Company, which aims to motivate users to be more well rested through a "monster catching" system. Similarly, we can have little creatures/pets which will not only be popular with the demographic, but it also creates a form of attachment of the users to the BellaBeat devices as they feel that they have to care for their virtual pets through healthy sleep schedules.

Personalized Notifications
For users aiming to lose weight, monitoring daily calorie intake is crucial. Bellabeat can provide tailored suggestions for low-calorie meals to help these users achieve their goals.

Workout Reccomendations and Tips
With the target demographic being mainly working women, they may lead rather hectic lifestyles and may not have enough time to plan their workouts. As such, the devices should also have reccommend detailed workout routines/ tips for the users to stay active.

Reward System
Introducing a reward system based on activity levels could significantly boost user engagement. The app could feature stages that users progress through by maintaining their activity level for a set period, such as a month. Each stage could reward users with stars, which can be converted into discounts on other Bellabeat products.

Recommendations for the Online Campaign
Ensure that the online campaign highlights the Bellabeat app as more than just a fitness tool. It should be portrayed as a comprehensive guide that empowers women to balance their personal, professional, and health-related goals. The campaign should emphasize how the app educates and motivates users through daily recommendations, helping them to integrate healthy habits into their busy lives.








