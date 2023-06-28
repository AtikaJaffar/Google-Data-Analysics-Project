
# Title: Case Study: How does a bike-share navigate speedy success?

# Author: Atika Rizvi

# Date: 2023-06-24

# output: html_document

# The objective of this script is to centralize the analysis and provide insights into the distinct usage patterns between members and casual riders of Cyclistic bikes.

**Installed below packages by running below command**


```{r}
install.packages("tidyverse")
```

```{r}
install.packages("lubridate")
```


```{r}
install.packages("janitor")
```


```{r}
install.packages("dplyr")
```


```{r}
install.packages("skimr")
```


```{r}
install.packages("ggplot2")
```

**We loaded packages by running below library s**

**For data wrangling and visualizations**

```{r}
library(tidyverse)
```

**For dates and times functions**

```{r}
library(lubridate)
```

**For data manipulation**

```{r}
library(dplyr)
```

**for examining and cleaning dirty data**

```{r}
library(janitor)
```

**For providing summary statistics about variables in data frames**

```{r}
library(skimr)
```
**Helps visualize data**

```{r}
library(ggplot2)
```
**Set working directory**

```{r}
setwd("C:\\Divvy Cyclistic Case Study")
```


**We ran below commands to import CSV files and merge them into single file**

```{r}
directory_path <- "C:/Divvy Cyclistic Case Study"
```

```{r}
CombinedDivvytrips <- data.frame()
```

```{r}
csv_files <- list.files(path = "C:\\Divvy Cyclistic Case Study", recursive = TRUE, full.names=TRUE)
CombinedDivvytrips <- do.call(rbind, lapply(csv_files, read.csv))
```

**Inspect the data to gain a better understanding of its structure and contents**

```{r}
str(CombinedDivvytrips)
```

**Modify the column names according to the specified requirements**

```{r}
library(dplyr)
```

```{r}
CombinedDivvytrips <- rename(CombinedDivvytrips,
                             trip_id = ride_id,
                             bike_type = rideable_type,
                             start_time = started_at,
                             end_time = ended_at,
                             start_station = start_station_name,
                             ss_id = start_station_id,
                             end_station = end_station_name,
                             es_id = end_station_id,
                             user_type = member_casual)
str(CombinedDivvytrips)
```

**It is advisable to create a backup before initiating the data cleaning process**

```{r}
backup_df <- CombinedDivvytrips
```

**Step 2: Perform data cleanup and preparation to ensure the data is ready for analysis**

**Identify and remove any duplicate records and eliminate N/A's and missing values from the dataset**

```{r}
anyDuplicated(CombinedDivvytrips$trip_id)
original_rows <- nrow(CombinedDivvytrips)
CombinedDivvytrips <- drop_na(CombinedDivvytrips)
CombinedDivvytrips <- CombinedDivvytrips[!(CombinedDivvytrips$start_station == "" |
                                             CombinedDivvytrips$ss_id == "" |
                                             CombinedDivvytrips$end_station == "" |
                                             CombinedDivvytrips$es_id == "" ),]
print(paste("Removed", original_rows - nrow(CombinedDivvytrips), " rows"))
```


**The data frame provides a high-level summary and overview of the dataset, offering key information and insights about the variables and their corresponding attributes**

```{r}
library(skimr)
```

```{r}
skim(CombinedDivvytrips)
```


```{r}
summary(CombinedDivvytrips)
```


**Provide a general overview of the data frame, giving a high-level understanding of its structure and content**


**The time values in the dataset are currently in character format. To facilitate further analysis and manipulation, it is necessary to convert them to datetime format.**

```{r}
CombinedDivvytrips$start_time= ymd_hms(CombinedDivvytrips$start_time)
CombinedDivvytrips$end_time= ymd_hms(CombinedDivvytrips$end_time)
```

**Calculate ride length of all trips**

```{r}
CombinedDivvytrips$ride_length <- as.double(difftime(CombinedDivvytrips$end_time, CombinedDivvytrips$start_time, units = "mins"))
```

**Add columns that list the date, month, day, and year of each ride**

**The default format is yyyy-mm-dd**


```{r}

CombinedDivvytrips$date <- as.Date(CombinedDivvytrips$start_time) 
CombinedDivvytrips$month <- month(CombinedDivvytrips$date, label = TRUE)
CombinedDivvytrips$day <- mday(CombinedDivvytrips$date)
CombinedDivvytrips$year <- year(CombinedDivvytrips$date)
CombinedDivvytrips$day_of_Week <- wday(CombinedDivvytrips$date, label = TRUE)
```

**Let's take another broad view of the data.**

```{r}
skim(CombinedDivvytrips)
```

**Let's examine the ride lengths more closely, as some of the values are negative and there are also some unusually high values**

**Examine the ride length data, sorted in both ascending and descending order, for the first 30 rows.**


```{r}
CombinedDivvytrips %>%
  select(ride_length, start_station, end_station) %>%
  arrange(desc(ride_length)) %>%
  slice(1:30)
```

```{r}
CombinedDivvytrips %>%
  select(ride_length, start_station, end_station) %>%
  arrange(ride_length) %>%
  slice(1:30)
```

**Examine the percentile breakdown.**

```{r}
percentiles <- quantile(CombinedDivvytrips$ride_length, probs = seq(0, 1, 0.01), na.rm = TRUE)
data.frame(percentiles)
```


```{r}
CombinedDivvytrips <- CombinedDivvytrips[!(CombinedDivvytrips$ride_length <= 1 |
                                             CombinedDivvytrips$ride_length >= percentiles[100]),]


```

```{r}
summary(CombinedDivvytrips)
```


**Exported clean data to a csv file**


```{r}
write.csv(CombinedDivvytrips, "CombinedDivvytrips_2022.csv")
```


**Performing descriptive analysis.**

**Phases of data analysis**

**Type of user**

```{r}
CombinedDivvytrips %>%
  group_by(user_type) %>%
  summarize(count = length(user_type), '%Rides' = (length(user_type) / nrow(CombinedDivvytrips)) * 100 )
```


```{r}
CombinedDivvytrips %>%
  ggplot(aes(user_type, fill = user_type)) + geom_bar() + labs(x= "Type of User", y= "Number of rides", title = "Rides Distribution By User Type")
```


**The number of Rides by months**

```{r}
CombinedDivvytrips %>%
  group_by(user_type, month) %>%
  summarise(number_of_ride = n(),
            .groups = "drop")
CombinedDivvytrips %>%
  group_by(user_type, month) %>%
  summarise(number_of_ride = n(),
            .groups = "drop")%>%
  ggplot(aes(month, number_of_ride, fill = user_type)) +
  geom_col(position = "dodge") +
  labs(title = "The number of rides by month", x = "Month", y = "Number of rides")

```

**The frequency of rides categorized by each day of the month**

```{r}
CombinedDivvytrips %>%
  group_by(user_type, day) %>%
  summarise(number_of_ride = n(),
            .groups = "drop")

CombinedDivvytrips %>%
  group_by(user_type, day) %>%
  summarise(number_of_ride = n(),
            .groups = "drop")%>%
  ggplot(aes(day, number_of_ride, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 100000, 10000)) +
  scale_x_continuous(breaks = seq(1, 31, 1)) +
  labs(title = "The frequency of rides categorized by each day of the month", x = "Day of the month", y = "Number of rides")
```


**The count of rides grouped by weekdays**

```{r}
CombinedDivvytrips %>%
  group_by(user_type, day_of_Week) %>%
  summarise(number_of_ride = n(),
            .groups = "drop")

CombinedDivvytrips %>%
  group_by(user_type, day_of_Week)%>%
  summarise(number_of_ride = n(),
            .groups = "drop")%>%
  ggplot(aes(day_of_Week, number_of_ride, fill = user_type)) +
  geom_col(position = "dodge") +
  labs(title = "The count of rides grouped by weekdays", x = "Weekday", y = "Number of rides")
```

**The count of rides categorized by the hour of the day**

```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time)) %>%
  group_by(user_type, hour) %>%
  summarise(no_of_rides = n(),
            .groups = "drop") %>%
  ggplot(aes(hour, no_of_rides, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 300000, 25000)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "The count of rides categorized by the hour of the day", x = "Hour of the day", y = "Number of rides")

```

**The count of rides divided by hour and weekday**

```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time)) %>%
  group_by(user_type, hour, day_of_Week) %>%
  summarise(no_of_ride = n(),
            .groups = "drop") %>%
  ggplot(aes(hour, no_of_ride, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 50000, 10000)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "The count of rides divided by hour and weekday", x = "Hour of the day", y = "Number of rides") +
  facet_wrap(~ day_of_Week)
```


**The count of rides divided by hour, categorized into weekdays and weekends**


```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time),
         day_type = ifelse(day_of_Week %in% c("Sat", "Sun"), "Weekend", "Weekday")) %>%
  group_by(user_type, hour, day_type) %>%
  summarise(no_of_ride = n(),
            .groups = "drop") %>%
  ggplot(aes(hour, no_of_ride, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 230000, 25000)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "The count of rides divided by hour, categorized into weekdays and weekends", x = "Hour of the day", y = "Number of rides") +
  facet_wrap(~ day_type)
```


**Calculating summary statistics for the duration of rides.**

**Creating a user-defined function to calculate the mode.**

```{r}
Mode <- function(x) {
  ux <- unique(x)
  ux[which.max(tabulate(match(x, ux)))]
}
```

**Summary**

```{r}
print("Ride Length Summary")
CombinedDivvytrips %>% 
  summarize(mean = mean(ride_length),
            sd = sd(ride_length),
            min = min(ride_length),
            median = median(ride_length),
            max = max(ride_length),
            mode = Mode(ride_length))
```


**Grouping the data by user type**

```{r}
CombinedDivvytrips %>%
  group_by(user_type) %>%
  summarize(mean = mean(ride_length),
            sd = sd(ride_length),
            min = min(ride_length),
            median = median(ride_length),
            max = max(ride_length),
            mode = Mode(ride_length))
```

**Visualize the distribution of ride length using box plots.**


```{r}
ggplot(data = CombinedDivvytrips, aes(x = user_type, y = ride_length, fill = user_type)) +
  geom_boxplot() +
  scale_y_continuous(breaks = seq(0, 130, 10)) +
  labs(title = "Distribution of ride length", x = "User Type", y = "Length (min)")
```

**Calculating the average ride length for each day**

```{r}
CombinedDivvytrips %>%
  group_by(user_type, day) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  arrange(day)
```


```{r}
CombinedDivvytrips %>%
  group_by(user_type, day) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(day, mean, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(breaks = seq(0, 28, 2)) +
  scale_x_continuous(breaks = seq(1, 31, 1)) +
  labs(title = "Calculating the average ride length for each day", x = "Day", y = "Average (min)")
```


**Calculating the average ride length for each month**

```{r}
CombinedDivvytrips %>% 
  group_by(user_type, month) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(month, mean, fill = user_type)) +
  geom_col(position = "dodge") +
  scale_y_continuous(breaks = seq(0, 28, 2)) +
  labs(title = "Calculating the average ride length for each month", x = "Month", y = "Average (min)")
```

**Calculating the average ride length for each day of the week**

```{r}
CombinedDivvytrips %>% 
  group_by(user_type, day_of_Week) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  arrange(day_of_Week)

CombinedDivvytrips %>% 
  group_by(user_type, day_of_Week) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(day_of_Week, mean, fill = user_type)) +
  geom_col(position = "dodge") +
  scale_y_continuous(breaks = seq(0, 28, 2)) +
  labs(title = "Calculating the average ride length for each day of the week", x = "Days of the week", y = "Average (min)")
```


**Calculating the average ride length for each hour of the day**

```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time)) %>%
  group_by(user_type, hour) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  arrange(hour)


CombinedDivvytrips %>% 
  mutate(hour = hour(start_time)) %>%
  group_by(user_type, hour) %>%
  summarize(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(hour, mean, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(breaks = seq(0, 28, 2)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "Calculating the average ride length for each hour of the day", x = "Hour of the day", y = "Average (min)")
```

**Calculating the average ride length for each hour of the day, divided by days of the week**

```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time),
         day_type = ifelse(day_of_Week %in% c("Sat", "Sun"), "Weekend", "Weekday")) %>%
  group_by(user_type, hour, day_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(hour, mean, color = user_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(breaks = seq(0, 30, 2)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "Calculating the average ride length for each hour of the day, divided by days of the week", x = "Hour of the day", y = "Average (min)") +
  facet_wrap(~ day_type)
```

**Calculating the average ride length for each hour of the day, divided by weekdays and weekends**

```{r}
CombinedDivvytrips %>% 
  mutate(hour = hour(start_time),
         day_type = ifelse(day_of_Week %in% c("Sat", "Sun"), "Weekend", "Weekday")) %>%
  group_by(user_type, hour, day_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(hour, mean, color = user_type, shape = day_type)) +
  geom_line() +
  geom_point() +
  scale_y_continuous(breaks = seq(0, 30, 2)) +
  scale_x_continuous(breaks = seq(0, 23, 1)) +
  labs(title = "Calculating the average ride length for each hour of the day, divided by weekdays and weekends
", x = "Hour of the day", y = "Average (min)")
```

**Analyzing the bike types used in the dataset**

**Calculating the number of bike type usages**

```{r}
CombinedDivvytrips %>% 
  group_by(bike_type) %>%
  summarise(number_of_ride = n())

CombinedDivvytrips %>% 
  group_by(bike_type) %>%
  summarise(number_of_ride = n()) %>%
  ggplot(aes(bike_type, number_of_ride, fill = bike_type)) +
  geom_col(position = "dodge", show.legend = FALSE) +
  scale_y_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 3000000, 1000000)) +
  labs(title = "Calculating the number of bike type usages", x = "Bike type", y = "Number of bike usages")
```

**The number of bike type usages by user type**

```{r}
CombinedDivvytrips %>% 
  group_by(user_type, bike_type) %>%
  summarise(number_of_ride = n(),
            .groups = "drop") %>%
  arrange(bike_type)

CombinedDivvytrips %>% 
  group_by(user_type, bike_type) %>%
  summarise(number_of_ride = n(),
            .groups = "drop") %>%
  ggplot(aes(user_type, number_of_ride, fill = bike_type)) +
  geom_col(position = "dodge") +
  labs(title = "The number of bike type usages by user type", x = "User type", y = "Number of bike usages")
```

**Analyzing the relationship between bike types and average ride time**


```{r}
CombinedDivvytrips %>% 
  group_by(bike_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  arrange(bike_type)

CombinedDivvytrips %>% 
  group_by(bike_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(bike_type, mean)) +
  geom_col(position = "dodge") +
  labs(title = "Analyzing the relationship between bike types and average ride time", y = "Average Ride Length", x = "Bike Type")
```


**Analyzing the relationship between bike types and average ride time, segmented by user type**


```{r}
CombinedDivvytrips %>% 
  group_by(user_type, bike_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  arrange(bike_type)

CombinedDivvytrips %>% 
  group_by(user_type, bike_type) %>%
  summarise(mean = mean(ride_length),
            .groups = "drop") %>%
  ggplot(aes(bike_type, mean, fill = user_type)) +
  geom_col(position = "dodge") +
  labs(title = "Analyzing the relationship between bike types and average ride time, segmented by user type", y = "Average Ride Length", x = "Bike Type")
```

**Conducting a descriptive analysis on the station data**

**Unique station names in the dataset**

```{r}
stations<- CombinedDivvytrips %>%
  gather(key, station_name, start_station, end_station) %>%
  distinct(station_name)

print(paste("Number of stations are", nrow(stations)))
```

**The most popular stations based on the number of rides started from each station**

```{r}
pop_stations <- CombinedDivvytrips %>%
  gather(key, station_name, start_station, end_station) %>%
  group_by(station_name) %>%
  summarise(number_trip = n()/2) %>%
  arrange(desc(number_trip))

head(pop_stations,10)

pop_stations %>%
  slice(1:10) %>%
  ggplot(aes(number_trip, reorder(station_name, number_trip))) +
  geom_col() +
  labs(title = "The top 10 most visited stations", x = "Number of trips", y = "Station name")
```


**The most popular stations based on the number of rides started from each station, segmented by user type**

```{r}
pu_station <- CombinedDivvytrips %>%
  gather(key, station_name, start_station, end_station) %>%
  group_by(user_type, station_name) %>%
  summarise(number_trip = n()/2,
            .groups = "drop_last") %>%
  arrange(desc(number_trip)) %>%
  slice(1:10)
```


**The top 10 most visited stations by Casual riders**

pu_station %>%
  head(10)

```{r}
pu_station %>%
  head(10) %>%
  ggplot(aes(number_trip, reorder(station_name, number_trip))) +
  geom_col() +
  scale_x_continuous(labels = scales::label_number_si(),
                     breaks = seq(0, 65000, 5000)) +
  labs(title = "The top 10 most visited stations by Casual riders", x = "Number of trips", y = "Station name")
```


**The top 10 most visited stations by member riders**

```{r}
pu_station %>%
  tail(10)

pu_station %>%
  tail(10) %>%
  ggplot(aes(number_trip, reorder(station_name, number_trip))) +
  geom_col() +
  labs(title = "The top 10 most visited stations by member riders", x = "Number of trips", y = "Station name")
```


























