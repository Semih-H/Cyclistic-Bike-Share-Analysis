# Cyclistic Bike Share Data Analysis
Google Data Analytics Capstone Project Using SQL & Tableau

## Overview
This project is a detailed case study **analyzing the ride-sharing patterns** of a bike-share company in Chicago. The goal is to understand how different user types, **casual riders** and **annual members** utilize the service and how the company can optimize its operations for better customer retention and revenue growth. 

The analysis focuses on identifying key trends in user behavior, such as **ride duration**, **peak usage times**, and **seasonal variations**. By leveraging **SQL** for data extraction and **Tableau** for visualization, the study aims to provide actionable insights that can help the company make data-driven business decisions.

Through this project, we aim to answer critical business questions such as:
- What are the differences in riding behavior between casual users and annual members?
- When do users ride the most, and how does seasonality impact usage trends?
- How can the company convert more casual riders into paying subscribers?

The findings from this study will help the bike-sharing company improve its services, enhance the user experience, and increase profitability by targeting key customer segments more effectively.

## Dataset

The data used for this analysis comes from a **bike-sharing company** called **Cyclistic** from Chicago, containing trip details such as ride duration, user type, and timestamps. The dataset helps in identifying key trends and differences between **casual riders** and annual **members**. The data used in this analysis spans **one year, from March 2024 to February 2025 (inclusive).** <br>

**Source**: [Cyclistic Bike Share Dataset](https://divvy-tripdata.s3.amazonaws.com/index.html)<br>
**Format**: CSV <br>
**Records**: More than 5 million rows of data <br>
<br>

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

## Key Insights
- To be added
- 

## Business Recommendations
1. To be added
2. 
3. 

# Bikeshare Data Cleaning and Transformation

This document explains the SQL queries used for cleaning, validating, and transforming the `bikeshare_all_trips` dataset. The goal is to prepare the data for accurate analysis by handling missing values, removing duplicates, and adding new features.

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
**Explanation:** This query counts the number of NULL values in each column by subtracting the count of non-null values from the total row count.

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
**Explanation:** Retrieves all rows where important location-related fields contain NULL values.

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

# Analysis

**[Cyclistic Bike Share Dashboard Tableau Public Link](https://public.tableau.com/app/profile/semih.h.rmeydan/viz/BikeShareCyclistic/BikeDash)** <br>


<img width="2765" height="1532" alt="Bikesharegithub" src="https://github.com/user-attachments/assets/65612a70-210a-4f57-b8c7-2047387fc36a" />


