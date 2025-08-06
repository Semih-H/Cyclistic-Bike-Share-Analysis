# Cyclistic Bike Share Data Analysis
Google Data Analytics Capstone Project Using SQL & Tableau

<br>

## Overview
This project is a detailed case study **analyzing the ride-sharing patterns** of a bike-share company in Chicago. The goal is to understand how different user types, **casual riders** and **annual members** utilize the service and how the company can optimize its operations for better customer retention and revenue growth. 

The analysis focuses on identifying key trends in user behavior, such as **ride duration**, **peak usage times**, and **seasonal variations**. By leveraging **SQL** for data extraction and **Tableau** for visualization, the study aims to provide actionable insights that can help the company make data-driven business decisions.

Through this project, we aim to answer critical business questions such as:
- What are the differences in riding behavior of casual users versus annual members?
- When do users ride the most, and how does seasonality impact usage trends?
- How can the company convert more casual riders into paying subscribers?

The findings from this study will help the bike-sharing company improve its services, enhance the user experience, and increase profitability by targeting key customer segments more effectively.

<br>

## Dataset

The data used for this analysis comes from a **bike-sharing company** called **Cyclistic** from Chicago, containing trip details such as ride duration, user type, and timestamps. The dataset helps in identifying key trends and differences between **casual riders** and annual **members**. The data used in this analysis spans **one year, from March 2024 to February 2025 (inclusive).** <br>

**Source**: [Cyclistic Bike Share Dataset](https://divvy-tripdata.s3.amazonaws.com/index.html)<br>
**Format**: CSV <br>
**Records**: 5,775,812 million rows of data 
<br>
**Fields Breakdown**: <br>

| Field Name            | Description                                                            |
|------------------------|-----------------------------------------------------------------------|
| ride_id               | Unique identifier for each ride                                        |
| rideable_type         | Type of bike used: `classic_bike`, `electric_bike`, `electric_scooter` |
| started_at            | Start date and time of the ride (timestamp)                            |
| ended_at              | End date and time of the ride (timestamp)                              |
| start_station_name    | Name of the station where the ride started                             |
| start_station_id      | Unique ID of the start station                                         |
| end_station_name      | Name of the station where the ride ended                               |
| end_station_id        | Unique ID of the end station                                           |
| start_lat             | Latitude of the starting location                                      |
| start_lng             | Longitude of the starting location                                     |
| end_lat               | Latitude of the ending location                                        |
| end_lng               | Longitude of the ending location                                       |
| member_casual         | User type: `member` (subscriber) or `casual` (one-time rider)          |

<br>

## Tools Used
- **SQL** (for data querying and manipulation)
- **Tableau** (for data visualization)

<br>

# Bikeshare Data Cleaning and Transformation

This document explains the SQL queries used for cleaning, validating, and transforming the `bikeshare_all_trips` dataset. The goal is to prepare the data for accurate analysis by handling missing values, removing duplicates, and adding new fields.

## 1. Uniting Multiple Tables into One

```sql
CREATE TABLE bikeshare_all_trips AS
SELECT * FROM "202403-divvy-tripdata"
UNION ALL
SELECT * FROM "202404-divvy-tripdata"
UNION ALL
SELECT * FROM "202405-divvy-tripdata"
UNION ALL
SELECT * FROM "202406-divvy-tripdata"
UNION ALL
SELECT * FROM "202407-divvy-tripdata"
UNION ALL
SELECT * FROM "202408-divvy-tripdata"
UNION ALL
SELECT * FROM "202409-divvy-tripdata"
UNION ALL
SELECT * FROM "202410-divvy-tripdata"
UNION ALL
SELECT * FROM "202411-divvy-tripdata"
UNION ALL
SELECT * FROM "202412-divvy-tripdata"
UNION ALL
SELECT * FROM "202501-divvy-tripdata"
UNION ALL
SELECT * FROM "202502-divvy-tripdata";
```
**Explanation:** This query merges the 12 monthly bikeshare datasets into a single table named `bikeshare_all_trips`. Using `UNION ALL` ensures that all records are retained without removing duplicates, which is useful for complete data analysis.

## 2. Checking for NULL Values

```sql
SELECT
    COUNT(*) - COUNT(ride_id) AS ride_id_nulls,
    COUNT(*) - COUNT(rideable_type) AS rideable_type_nulls,
    COUNT(*) - COUNT(started_at) AS started_at_nulls,
    COUNT(*) - COUNT(ended_at) AS ended_at_nulls,
    COUNT(*) - COUNT(start_station_name) AS start_station_name_nulls,
    COUNT(*) - COUNT(start_station_id) AS start_station_id_nulls,
    COUNT(*) - COUNT(end_station_name) AS end_station_name_nulls,
    COUNT(*) - COUNT(end_station_id) AS end_station_id_nulls,
    COUNT(*) - COUNT(start_lat) AS start_lat_nulls,
    COUNT(*) - COUNT(start_lng) AS start_lng_nulls,
    COUNT(*) - COUNT(end_lat) AS end_lat_nulls,
    COUNT(*) - COUNT(end_lng) AS end_lng_nulls,
    COUNT(*) - COUNT(member_casual) AS member_casual_nulls
FROM bikeshare_all_trips;
```
**Explanation:** This query counts the number of NULL values in each column by subtracting the count of non-null values from the total row count. This resulted in the detection of missing data in the following columns: start_station_name, start_station_id, end_station_name, end_station_id, end_lat, and end_lng.

## 3. Identifying Rows with Missing Data

```sql
SELECT *
FROM bikeshare_all_trips
WHERE start_station_name IS NULL
   OR start_station_id IS NULL
   OR end_station_name IS NULL
   OR end_station_id IS NULL
   OR end_lat IS NULL
   OR end_lng IS NULL;
```
**Explanation:** Retrieves all rows where important location-related fields contain NULL values. We should keep the records only if all coordinates are present. Otherwise, if end coordinates are missing, we can't determine where the ride ended, making the record incomplete and unreliable for analysis.

## 4. Deleting Rows with Critical Missing Values

```sql
DELETE FROM bikeshare_all_trips
WHERE 
    (start_lat IS NULL OR start_lng IS NULL 
     OR end_lat IS NULL OR end_lng IS NULL);
```
**Explanation:** Removes rows where latitude or longitude values are missing, as these records are not useful for analysis.

## 5. Replacing NULL Values with Default Values

```sql
UPDATE bikeshare_all_trips 
SET start_station_name = 'Unknown' 
WHERE start_station_name IS NULL;

UPDATE bikeshare_all_trips 
SET start_station_id = 'Unknown' 
WHERE start_station_id IS NULL;

UPDATE bikeshare_all_trips 
SET end_station_name = 'Unknown' 
WHERE end_station_name IS NULL;

UPDATE bikeshare_all_trips 
SET end_station_id = 'Unknown' 
WHERE end_station_id IS NULL;
```
**Explanation:** Updates NULL station names and IDs to 'Unknown' for better data consistency.

## 6. Detecting and Removing Duplicates

```sql
SELECT ride_id, COUNT(*)
FROM bikeshare_all_trips
GROUP BY ride_id
HAVING COUNT(*) > 1;
```
**Explanation:** Identifies duplicate records based on `ride_id`.

```sql
DELETE FROM bikeshare_all_trips
WHERE ride_id IN (
    SELECT ride_id
    FROM (
        SELECT ride_id, ROW_NUMBER() OVER (PARTITION BY ride_id ORDER BY started_at) AS row_num
        FROM bikeshare_all_trips
    ) AS subquery
    WHERE row_num > 1
);
```
**Explanation:** Deletes duplicate entries, keeping only the first occurrence based on `started_at`.

## 7. Cleaning Station Names (Trimming Whitespace)

```sql
UPDATE bikeshare_all_trips
SET ride_id = TRIM(ride_id),
    rideable_type = TRIM(rideable_type),
    start_station_name = TRIM(start_station_name),
    start_station_id = TRIM(start_station_id),
    end_station_name = TRIM(end_station_name),
    end_station_id = TRIM(end_station_id),
    member_casual = TRIM(member_casual);
```
**Explanation:** Removes leading and trailing whitespace from string fields.

## 8. Validating Latitude and Longitude Values

```sql
DELETE FROM bikeshare_all_trips
WHERE start_lat NOT BETWEEN -90 AND 90
   OR start_lng NOT BETWEEN -180 AND 180
   OR end_lat NOT BETWEEN -90 AND 90
   OR end_lng NOT BETWEEN -180 AND 180;
```
**Explanation:** Deletes rows where latitude or longitude values are outside the valid geographic range.


## 9. Ensuring Valid Rideable Types

```sql
DELETE FROM bikeshare_all_trips
WHERE rideable_type != 'classic_bike' AND 
    rideable_type != 'electric_bike' AND
    rideable_type != 'electric_scooter';
```
**Explanation:** Removes rows with invalid rideable types.

## 9. Ensuring No Negative Ride Durations

```sql
SELECT *
FROM bikeshare_1year_tripdata
WHERE ended_at < started_at;
```

```sql
DELETE
FROM bikeshare_1year_tripdata
WHERE ended_at < started_at;
```
**Explanation:** 202 rows with negative ride durations were detected and deleted.

## 9. Introducing New Calculated Fields

```sql
ALTER TABLE bikeshare_1year_tripdata
ADD COLUMN ride_duration INTERVAL,
ADD COLUMN day_of_week VARCHAR(20),
ADD COLUMN distance_km DOUBLE PRECISION;
```

Update ride_duration:

```sql
UPDATE bikeshare_1year_tripdata
SET ride_duration = ended_at - started_at;
```


Update day_of_week:

```sql
UPDATE bikeshare_1year_tripdata
SET day_of_week = 
    CASE
        WHEN EXTRACT(DOW FROM started_at) = 0 THEN 'Sunday'
        WHEN EXTRACT(DOW FROM started_at) = 1 THEN 'Monday'
        WHEN EXTRACT(DOW FROM started_at) = 2 THEN 'Tuesday'
        WHEN EXTRACT(DOW FROM started_at) = 3 THEN 'Wednesday'
        WHEN EXTRACT(DOW FROM started_at) = 4 THEN 'Thursday'
        WHEN EXTRACT(DOW FROM started_at) = 5 THEN 'Friday'
        WHEN EXTRACT(DOW FROM started_at) = 6 THEN 'Saturday'
    END;
```


Calculate distance (Haversine formula):

```sql
UPDATE bikeshare_1year_tripdata
SET distance_km = 6371 * ACOS(
    LEAST(1, GREATEST(-1,
        COS(RADIANS(start_lat)) * COS(RADIANS(end_lat)) *
        COS(RADIANS(end_lng) - RADIANS(start_lng)) +
        SIN(RADIANS(start_lat)) * SIN(RADIANS(end_lat))
    ))
);
```

ROUND distance_km records, casting to numeric:

```sql
UPDATE bikeshare_1year_tripdata
SET distance_km = ROUND(distance_km::numeric, 3);
```

For Tableau:

```sql
ALTER TABLE bikeshare_1year_tripdata
ADD COLUMN ride_duration_minutes NUMERIC;
```

```sql
UPDATE bikeshare_1year_tripdata
SET ride_duration_minutes = EXTRACT(EPOCH FROM ride_duration) / 60
```

<br>

# Analysis


- **Total Rides by User Type:**

<img width="500" height="216" alt="Total Rides" src="https://github.com/user-attachments/assets/77c7375f-bd6c-4cd8-9b22-3af21fcf8749" />

This simple bar chart provides an overview of the total number of rides and breaks it down by user type. It clearly shows that members make up the majority of the total rides, accounting for **3.66 million** out of **5.78 million** total rides.

<br>

- **Average Ride Duration:**

<img width="500" height="216" alt="Average Ride Duration" src="https://github.com/user-attachments/assets/41689c71-1b44-4f2c-9018-31b9ac1a84f5" />

This bar chart compares the average ride duration between casual riders and members. Casual riders have a significantly longer average ride duration at **21 minutes**, compared to members who average **12.1 minutes**.

<br>


- **Average Distance:**

<img width="500" height="216" alt="Avg Distance" src="https://github.com/user-attachments/assets/b1300d06-f15e-4c44-84f1-7fcf1ac51c8f" />

This chart shows that the average distance for both casual riders and members is very similar, at **just over 2 kilometers**. This suggests that while ride duration differs, the actual distance traveled per ride is consistent across both user groups.

<br>


- **Total Rides (Month):**


<img width="500" height="250" alt="month" src="https://github.com/user-attachments/assets/904f898f-0521-4235-ba17-3d5709de1ea2" />

This line chart illustrates the seasonal trend of total rides over the year. Both members and casual riders show a clear increase in rides during the warmer months (**May to September**), with a **peak in September**, and a decline in colder months.

<br>


- **Total Rides (Week):**


<img width="500" height="250" alt="weekdays" src="https://github.com/user-attachments/assets/ea0d9fdd-58f7-42d6-a29a-4b8094ce3c61" />

This line chart displays the total rides for each day of the week. Member rides are highest on **weekdays** and lower on **weekends**, indicating a primary use for commuting, while casual riders show a peak in usage on **Saturdays** and **Sundays**.

<br>


- **Total Rides (Hour):**


<img width="500" height="250" alt="hour" src="https://github.com/user-attachments/assets/21a34cfd-9e80-4700-8b60-fa40da2692a4" />

This chart visualizes the total rides per hour of the day for both members and casual riders. Members' rides peak during typical commuter hours (**around 8 AM and 5-6 PM**), reflecting a daily travel pattern, likely for work, while casual riders' usage is highest in the **late afternoon**.

<br>


- **Total Rides by Bike Type:**

<img width="500" height="330" alt="biketypes" src="https://github.com/user-attachments/assets/237822d5-38ea-4dd0-ad3d-3274ee5c927d" />

This stacked bar chart shows that Classic bikes and E-bikes are the most popular choices, with E-bikes having slightly more total rides. E-scooters account for a very small portion of the total rides.


<br>



- **Average Ride Duration by Bike Type:**

<img width="500" height="290" alt="avg ride duration bike types" src="https://github.com/user-attachments/assets/22039146-95ed-4d67-ab5a-0a495be5e8e7" />

This stacked bar chart breaks down the average ride duration for each bike type, showing the contribution from both member and casual riders. It reveals that the longest average rides, by a large margin, are taken on Classic bikes by casual riders.


<br>


- **Top Stations:**

<img width="500" height="290" alt="topstations" src="https://github.com/user-attachments/assets/38f28bab-3b09-434c-91d7-8183569d9748" />

This bar chart lists the top five stations by total number of rides. It identifies "Streeter Dr & Grand Ave" as the most popular station, with a significantly higher number of rides than the others.

<br>

# Conclusion and Recommendations

The data reveals a clear distinction between casual riders and members. Members primarily use the service for commuting, as seen by their peak usage during weekday rush hours. Their rides are shorter in duration but occur more frequently. In contrast, casual riders use the service for longer periods, often on weekends and in the late afternoon, suggesting a preference for recreational use. While both groups travel similar distances on average, their riding behaviors and patterns are distinct.

To convert more casual riders into paying subscribers, the company should focus on bridging this behavioral gap by offering tailored incentives and subscription models.

- **Introduce a Flexible Weekend Pass**: Casual riders' peak usage on weekends suggests they may be hesitant to commit to a full annual membership. The company could introduce a more flexible, multi-day pass or a weekend-specific subscription that is more appealing for recreational use. This would allow casual riders to experience the benefits of a membership (e.g., discounted rates) without the perceived commitment of an annual plan.

- **Offer "Family" or "Group" Membership Tiers**: Casual riders tend to have longer rides, which could indicate they are riding with friends or family. The company could introduce a membership that allows for multiple bikes to be checked out under one account at a reduced rate. This would directly appeal to the likely use case of casual riders and make a subscription a more cost-effective option for group outings.

- **Target Marketing with Usage-Based Incentives**: Utilize the data to create targeted marketing campaigns. For example, after a casual rider completes a ride that exceeds the average member ride duration, an in-app notification or email could be sent highlighting the savings they would have gained with a membership. This would use their specific usage patterns to demonstrate the value of a subscription in a tangible way.

<br>

# Dashboard 
**[Cyclistic Bike Share Dashboard Tableau Public Link](https://public.tableau.com/app/profile/semih.h.rmeydan/viz/BikeShareCyclistic/BikeDash)** <br>

<br>

<img width="2722" height="1540" alt="Bikesharegithub" src="https://github.com/user-attachments/assets/6da67b9f-f69e-4a31-a310-e854f0232b60" />



