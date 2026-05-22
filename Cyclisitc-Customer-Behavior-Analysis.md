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

-   Teableau: Data visualization
  
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
