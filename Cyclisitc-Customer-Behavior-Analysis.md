---

Title: 'Case Study: How Annual Members and Casual Riders Use Cyclistic Bikes Differently'
Author: 'Hoang Do Phan'
Date: '2026-03-24'
---

# Case Study: How Annual Members and Casual Riders Use Cyclistic Bikes Differently

**Author:** Hoang Do Phan
**Date:** 2026-03-24
## Background

**Cyclistic:** A bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can't use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each day.

Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members. Cyclistic's finance analysts have concluded that annual members are much more profitable than casual riders.

Moreno (Director of Marketing) has set a clear goal: Design marketing strategies aimed at converting casual riders into annual members. In order to do that, however, the marketing analyst team needs to better understand how annual members and casual riders differ, why casual riders would buy a membership, and how digital media could affect their marketing tactics. Moreno and her team are interested in analyzing the Cyclistic historical bike trip data to identify trends.

## Step 1: Getting Information of The Task

### Business Task

To identify how do annual members and casual rider s use Cyclistic differently.

### Stakeholders

**Primary:** Lily Monero, Director of Marketing

**Secondary:** Cyclistic executive team (approves recommended marketing program)

## Step 2: Collection and Preparation of Data

### Data & Source Integrity

-   Data is stored in remote server [(url)](https://divvy-tripdata.s3.amazonaws.com/index.html) and provided by Motivate International Inc. and separated in chunks of querterly .csv files.

-   Data has been protected under data-privacy license mentioned here: [license](https://ride.divvybikes.com/data-license-agreement). This license states that this data has been provided "As is" as per bikeshares sole discretion. So reliability of the data can be vetted eventhough this has been provided by third party.

-   This data cannot be connected to the individual riders and their credit card numbers as unique rider ID has been used to record ride data.

### Observations

1.  There are 13 columns in the data files. These are: ride_id, rideable_type, started_at, ended_at, start_station_name, end_station_name, start_station_id, end_station_id, start_station, start_lat, start_lng, end_lat, end_lng, member_casual

2.  started_at and ended_at columns contains date-time data and formatted as YYYY-MM-DD HH:MM:SS format.

3.  start_station_id and end_station_id has discrepancy. Some of the IDs contain alphabets at the beginning (12 char length) and some contains only numbers (variable length 3-8).

4.  Although some of the csv files did not have start_station_name, start_station_id, end_station_name, end_station_id; these rows contain latitude and longitude data. This can be used to fill these empty values.

5.  member_casual column contains 2 types of membership data: member or casual.


### Tools Used

-   SQL (sqlite, DB Browser): Because the files is too large for sql so R will be the main tool :(

-   R: Cleaning and analysis data

-   Tableau: Data visualization
  
## Step 3: Data Collection and Consolidation

The Cyclistic trip data was downloaded from the Divvy Trip Data repository. For this project, I used twelve monthly datasets from January 2025 to December 2025.

The original downloaded files were stored in a `zip_files` folder, while the extracted CSV files were stored in a `cvs_files` folder for easier access and organization.

<img width="1000" height="718" alt="Screenshot 2026-05-20 201212" src="https://github.com/user-attachments/assets/30f5dfdd-1c1b-4369-a2d7-0a082cec9eb6" />
<img width="1000" height="717" alt="Screenshot 2026-05-20 201159" src="https://github.com/user-attachments/assets/3a431da8-8204-41ea-9fd1-ba4e261f1687" />


### 3.1 Using R to Merge the Monthly Datasets
 To simplify data management and analysis, all twelve monthly CSV files were combined into a single dataset using R.
 
```r
install.packages("tidyverse")
```

### 3.2 Verifying Monthly Files

```r
library(tidyverse)

files <- list.files(
  path = "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/cvs_files",
  full.names = TRUE
)

length(files)
```

Output:

```r
[1] 12
```
<img width="793" height="203" alt="Screenshot 2026-05-20 202608" src="https://github.com/user-attachments/assets/07a74de4-caa9-4964-952d-0792118b34f3" />

This confirmed that all twelve monthly datasets were available.

### 3.3 Merging Monthly Datasets

```r
cyclistic_data <- map_df(files, read_csv)
```

The `map_df()` function was used to merge all monthly files into a single dataframe named `cyclistic_data`.

### 3.4 Validating the Merged Dataset

```r
dim(cyclistic_data)

glimpse(cyclistic_data)
```
<img width="815" height="391" alt="Screenshot 2026-05-20 203149" src="https://github.com/user-attachments/assets/17cac96e-d1c0-43af-90f9-aeadc54e2357" />

These checks were used to review the number of rows, columns, variable names, and data types before starting the cleaning process.

  ## ## Step 4: Data Cleaning and Transformation

### 4.1 Checking Missing Values

Missing values were assessed using the `colSums(is.na())` function.

```r
colSums(is.na(cyclistic_data))
```

<img width="793" height="191" alt="Screenshot 2026-05-20 220705" src="https://github.com/user-attachments/assets/d839f09e-907b-4a07-9c0c-60afb635d929" />

Some station-related columns contained missing values. However, no missing values were found in key variables such as `ride_id`, `rideable_type`, `started_at`, `ended_at`, and `member_casual`.

---

### 4.2 Checking Duplicate Records

Duplicate ride IDs were checked to ensure data integrity.

```r
sum(duplicated(cyclistic_data$ride_id))
```

Output:

```r
[1] 0
```

<img width="412" height="52" alt="Screenshot 2026-05-20 222734" src="https://github.com/user-attachments/assets/ef296628-ea69-4ee7-82a8-b802bfeb02b7" />

No duplicate ride IDs were identified in the dataset.

---

### 4.3 Creating New Variables

Date-time columns were converted to the appropriate format, and additional variables were created for analysis.

```r
cyclistic_data <- cyclistic_data %>%
  mutate(
    started_at = as.POSIXct(started_at),
    ended_at = as.POSIXct(ended_at)
  )

cyclistic_data <- cyclistic_data %>%
  mutate(
    ride_length = as.numeric(
      difftime(ended_at, started_at, units = "mins")
    )
  )

cyclistic_cleaned <- cyclistic_data %>%
  filter(ride_length > 0)

cyclistic_cleaned <- cyclistic_cleaned %>%
  mutate(
    day_of_week = weekdays(started_at),
    month = format(started_at, "%B"),
    year = format(started_at, "%Y")
  )
```

The following variables were added:

- `ride_length`
- `day_of_week`
- `month`
- `year`

---

### 4.4 Validating the Cleaned Dataset

The cleaned dataset was reviewed to confirm that all transformations were successfully applied.

```r
dim(cyclistic_cleaned)

glimpse(cyclistic_cleaned)
```

Output:

```r
[1] 5552965 17
```

<img width="807" height="467" alt="Screenshot 2026-05-20 223258" src="https://github.com/user-attachments/assets/92507881-f87b-4e96-a666-8c2e46d5013a" />

The final cleaned dataset contains **5,552,965 records** and **17 variables**, and is ready for exploratory data analysis.

## Step 5: Exploratory Data Analysis

The objective of this analysis is to identify how annual members and casual riders use Cyclistic bikes differently.

### 5.1 Total Rides by Rider Type

```r
rides_by_member <- cyclistic_cleaned %>%
  group_by(member_casual) %>%
  summarise(total_rides = n())

rides_by_member
```

Output:

| Rider Type | Total Rides |
|------------|------------:|
| Casual | 1,999,488 |
| Member | 3,553,477 |

<img width="485" height="211" alt="Screenshot 2026-05-21 210442" src="https://github.com/user-attachments/assets/68402d00-31a3-4826-9188-0f5eea274ece" />

**Observation:** Annual members generated significantly more rides than casual riders, accounting for approximately 64% of all rides.

---

### 5.2 Average Ride Duration

```r
avg_duration <- cyclistic_cleaned %>%
  group_by(member_casual) %>%
  summarise(
    avg_ride_length = mean(ride_length)
  )

avg_duration
```

Output:

| Rider Type | Average Ride Length (Minutes) |
|------------|------------------------------:|
| Casual | 22.6 |
| Member | 12.3 |

<img width="477" height="260" alt="Screenshot 2026-05-21 210526" src="https://github.com/user-attachments/assets/ef7a1c61-4f71-47df-981d-36f9244b29d8" />

**Observation:** Casual riders spent nearly twice as much time per ride compared to annual members.

---

### 5.3 Riding Patterns by Day of Week

```r
rides_by_weekday <- cyclistic_cleaned %>%
  group_by(
    day_of_week,
    member_casual
  ) %>%
  summarise(
    total_rides = n(),
    .groups = "drop"
  )

rides_by_weekday
```

<img width="488" height="588" alt="Screenshot 2026-05-21 210604" src="https://github.com/user-attachments/assets/430cd11f-b0f0-4138-b614-ed9c88c3ce93" />

**Observation:** Member rides remained relatively consistent throughout the week, while casual riders showed noticeably higher activity during weekends, particularly on Saturday.

---

### 5.4 Monthly Riding Patterns

```r
rides_by_month <- cyclistic_cleaned %>%
  group_by(
    month,
    member_casual
  ) %>%
  summarise(
    total_rides = n(),
    .groups = "drop"
  )

rides_by_month
```

<img width="452" height="525" alt="Screenshot 2026-05-21 210721" src="https://github.com/user-attachments/assets/1e0f48eb-3422-4ee6-a820-13e2dc694ec0" />

**Observation:** Both rider groups demonstrated strong seasonality, with ride volumes increasing during warmer months and declining during winter.

---

### 5.5 Bike Type Preferences

```r
bike_type_usage <- cyclistic_cleaned %>%
  group_by(
    rideable_type,
    member_casual
  ) %>%
  summarise(
    total_rides = n(),
    .groups = "drop"
  )

bike_type_usage
```

Output:

| Bike Type | Rider Type | Total Rides |
|------------|------------|------------:|
| Classic Bike | Casual | 672,670 |
| Classic Bike | Member | 1,275,359 |
| Electric Bike | Casual | 1,326,818 |
| Electric Bike | Member | 2,278,118 |

<img width="475" height="380" alt="Screenshot 2026-05-21 210740" src="https://github.com/user-attachments/assets/34713f1b-78c9-45b4-9443-c7e25e34acc9" />

**Observation:** Electric bikes were the most popular option for both rider groups, accounting for the majority of rides.

---

### 6.6 Average Ride Duration by Day of Week

```r
duration_by_weekday <- cyclistic_cleaned %>%
  group_by(
    day_of_week,
    member_casual
  ) %>%
  summarise(
    avg_ride_length = mean(ride_length),
    .groups = "drop"
  )

duration_by_weekday
```

<img width="524" height="582" alt="Screenshot 2026-05-21 210806" src="https://github.com/user-attachments/assets/c8a84f50-c429-434b-a4b3-0b9c662c924f" />

**Observation:** Casual riders consistently recorded longer ride durations across every day of the week. The highest average ride duration was observed on Sunday (26.1 minutes), suggesting that casual riders primarily use Cyclistic bikes for leisure and recreational purposes.
### 5.7 Exporting Summary Tables for Tableau

After completing the exploratory analysis in R, six summary tables were exported as CSV files for visualization in Tableau.

```r
write_csv(
  rides_by_member,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/rides_by_member.csv"
)

write_csv(
  avg_duration,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/avg_duration.csv"
)

write_csv(
  rides_by_weekday,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/rides_by_weekday.csv"
)

write_csv(
  rides_by_month,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/rides_by_month.csv"
)

write_csv(
  bike_type_usage,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/bike_type_usage.csv"
)

write_csv(
  duration_by_weekday,
  "C:/Users/pc/OneDrive/Máy tính/cycle data raw 12month/duration_by_weekday.csv"
)
```

<img width="898" height="613" alt="Screenshot 2026-05-21 212111" src="https://github.com/user-attachments/assets/90dc57b0-d39e-43b0-a904-a7733638a3aa" />

These exported CSV files were used as the data sources for creating Tableau visualizations and building the final dashboard.
