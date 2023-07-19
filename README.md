# GoogleDataAnalyticsCapstone: "How Does a Bike-Share Navigate Speedy Success?"

## Introduction 

This capstone project for the Google Data Analysis Professional Certificate. This is a fictional scenario with real world data.

## Scenario 

Cyclistic, a bike-share company in Chicago. The director of marketing believes 
the company’s future success depends on maximizing the number of annual 
memberships. Therefore, your team wants to understand how casual riders and 
annual members use Cyclistic bikes differently.

### Cyclistic
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime. They have a flexibile pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members.

## Business Task

* How do annual members and casual riders use Cyclistic bikes differently?
* Why would casual riders buy Cyclistic annual memberships?
* How can Cyclistic use digital media to influence casual riders to become members?

## Prepare

[Cyclistic’s historical trip data](https://divvy-tripdata.s3.amazonaws.com/index.html) has been made available by Motivate International Inc. under this [license](https://ride.divvybikes.com/data-license-agreement).

Data used is 05-22 to 04-23

## Setting up work environment

```r
library(tidyverse)
library(scales)
```

Collecting the last 12 months of data in the same folder reading all into R

<details>
  <summary>Click me</summary>
  
```{r gathering data}
csv_list <- c("202205-divvy-tripdata.csv", "202206-divvy-tripdata.csv",
              "202207-divvy-tripdata.csv", "202208-divvy-tripdata.csv",
              "202209-divvy-tripdata.csv", "202210-divvy-tripdata.csv",
              "202211-divvy-tripdata.csv", "202212-divvy-tripdata.csv",
              "202301-divvy-tripdata.csv", "202302-divvy-tripdata.csv",
              "202303-divvy-tripdata.csv", "202304-divvy-tripdata.csv")

all_trips <- data.frame()

for (csv in csv_list) {
  file_loc <- paste("bike_data/",csv, sep = "")
  bike_df <- read_csv(file_loc)
  all_trips <- bind_rows(all_trips, bike_df)
}
```

</details>

looped over a list of all 12 months merged into 1 data frame.


```{r}
colnames(all_trips)
```

Confirming that each column is the correct data type and member_casual has only 2 values member or casual.

```{r}
str(all_trips)
unique(all_trips$member_casual)
```

The data is correct typing now to remove start lat, start lng, end lat, and end lng as they are not needed columns and create date fields for filtering and ride length.

```{r}
all_trips <- all_trips %>% 
  select(ride_id, rideable_type, started_at, ended_at, start_station_name, 
           start_station_id, end_station_name, end_station_id, 
           member_casual)
```

Day fields

```{r}
all_trips$date <- date(all_trips$started_at) #The default format is yyyy-mm-dd
all_trips$month <- month(all_trips$date, label = TRUE, abbr = FALSE)
all_trips$day <- day(all_trips$date)
all_trips$year <- year(all_trips$date)
all_trips$day_of_the_week <- wday(all_trips$date, label = TRUE, abbr = FALSE)
```

Adding ride_length

```{r}
all_trips$ride_length <- difftime(all_trips$ended_at, all_trips$started_at)
all_trips$ride_length <- as.numeric(all_trips$ride_length)
```

Removal of records negative ride length.

```{r}
all_trips <- filter(all_trips, ride_length >= 0)
```

## Analysis

Overall average, median, max and min ride length.
<details>

```{r}
all_trips %>% 
  summarize(average_ride_length = mean(ride_length), 
            median_ride_length = median(ride_length),
            max_ride_length = max(ride_length),
            min_ride_length = min(ride_length))
```
</details>


Comparing number of rides and ride length over the days of the week and member status.

```{r}
all_trips %>% 
  group_by(member_casual, day_of_the_week) %>% 
  summarise(number_of_rides = n(), avg_length = mean(ride_length), median_length = median(ride_length),
            max_length = max(ride_length), min_length = min(ride_length), .groups = 'drop')
```

## Share

Daily Usage

* Members make more trips than casual throughout the week especially during the middle of the week.

```{r daily usage}
all_trips %>% 
  group_by(member_casual, day_of_the_week) %>% 
  summarise(number_of_rides = n(), average_duration = mean(ride_length), .groups = 'drop') %>% 
  arrange(member_casual, day_of_the_week) %>% 
  ggplot(aes(x = day_of_the_week, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge") +
  labs(x = "Weekdays", y = "Numer of Rides", title = "Usage per Weekday") +
  scale_y_continuous(labels = comma)
```

```{r monthly duration}
all_trips %>% 
  group_by(member_casual, month) %>% 
  summarise(number_of_rides = n(), average_duration = mean(ride_length), .groups = 'drop') %>% 
  arrange(member_casual, month) %>% 
  ggplot(aes(x = month, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge") + theme(axis.text.x = element_text(angle = 45)) +
  labs(x = "Months", y = "Average Trip Duration", title = "Duration Per Month")
```
## Act

After analysing the data I have found how casual bike rider and annual member use the bikes differenetly, why a casual rider may become a annual member, and how digital media can be used to influence casual riders. Based off those findings here are my conclusions.

* Members use the bikes more than casual riders.
* Members bike trips ride are shorter than casual rider, about 10-11mins trips.
* Casual riders use the bikes for longer periods of time mostly on the weekends.
* Casual riders mostly using the bikes around summer time(May-Sept.)

To increase annual members i would suggest, through the use of digiatal media, increase the awareness of the benifits of being an annual member especially during the summer months.

