# GDA-Capstone_Cyclistic-Case_Study

## Introduction 
This is a capstone project as a part of [Google Data Analytics Professional Certificate Course](https://www.coursera.org/professional-certificates/google-data-analytics). For the analysis I will be using SQL and Power BI for data visualization.

For this project data analysis step followed are: **Ask**, **Prepare**, **Process**, **Analyze**, **Share** and **Act**

### Scenario
You are a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing and your manager Lily Moreno believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members.

### About the Company
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. 
The bikes can be unlocked from one station and returned to any other station in the system anytime.
Customers who purchase single-ride or full-day passes are referred to as **casual riders**.
Customers who purchase annual memberships are Cyclistic **members**.
Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders.


## Ask
Three questions will guide the future marketing program:

1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

Moreno has assigned you the first question to answer: How do annual members and casual riders use Cyclistic bikes differently? 

## Prepare
I will use Cyclistic historical trip data to analyze the trends from Jan 2023 through April 2023 which was downloaded from [here](https://divvy-tripdata.s3.amazonaws.com/index.html). 
#### Data Source: 
The data has been made available by Motivate International Inc. under this [license](https://divvybikes.com/data-license-agreement) This is public data that you can use to explore how different customer types are using Cyclistic bikes. But note that data-privacy issues prohibit you from using riders’ personally identifiable information. This means that you won’t be able to connect pass purchases to credit card numbers to determine if casual riders live in the Cyclistic service area or if they have purchased multiple single passes.

#### Data Organization: 
All the trip data is in comma-delimited(.csv) format.
For this analysis, I have downloaded quarter 1 data which is January, February, March and April of year 2023.
It is a structured data. There are 13 columns in each file namely "ride_id", "rideable_type", "started_at", "ended_at", "start_station_name", "start_station_id", "end_station_name", "end_station_id", "start_lat", "start_lng", "end_lat", "end_lng" and "member_casual".

## Process
This step involves data cleaning and manipulation. This makes sure data is stored appropriately and prepared for analysis. 
I have created a dataset in BigQuery named Cyclistic_bike_share_data and added 4 tables for each month i.e. january_trip_data, february_trip_data, march_trip_data, april_trip_data.

#### Combining the tables
In this step, I combined all 4 tables to create a single table cyclistic_q1 with the function UNION ALL

```
--Creating a new table combining all four months data

CREATE TABLE Cyclistic_bike_share_data.cyclistic_q1 AS 

SELECT * FROM Cyclistic_bike_share_data.january_trip_data
UNION ALL
SELECT * FROM Cyclistic_bike_share_data.february_trip_data
UNION ALL
SELECT * FROM Cyclistic_bike_share_data.march_trip_data
UNION ALL
SELECT * FROM Cyclistic_bike_share_data.april_trip_data

```
#### Checking the total number of rows in the new table

```
SELECT COUNT(*) FROM Cyclistic_bike_share_data.cyclistic_q1 
```
The total number of rows returned were 1066014. 

#### Check for duplicates
Here, the __ride_id__ is unique so checked for duplicates for this column. 

```
SELECT COUNT(DISTINCT (ride_id)) FROM Cyclistic_bike_share_data.cyclistic_q1 
```
This returned the same number of rows as the previous 1066014, so there are no duplicate rides.

#### Check for null values

Next, all the columns were checked for null values. 

```
SELECT COUNT(*) - COUNT(ride_id) ride_id,
 COUNT(*) - COUNT(rideable_type) rideable_type,
 COUNT(*) - COUNT(started_at) started_at,
 COUNT(*) - COUNT(ended_at) ended_at,
 COUNT(*) - COUNT(start_station_name) start_station_name,
 COUNT(*) - COUNT(start_station_id) start_station_id,
 COUNT(*) - COUNT(end_station_name) end_station_name,
 COUNT(*) - COUNT(end_station_id) end_station_id,
 COUNT(*) - COUNT(start_lat) start_lat,
 COUNT(*) - COUNT(start_lng) start_lng,
 COUNT(*) - COUNT(end_lat) end_lat,
 COUNT(*) - COUNT(end_lng) end_lng,
 COUNT(*) - COUNT(member_casual) member_casual 
 
 FROM Cyclistic_bike_share_data.cyclistic_q1
```
The total number of null values found in the columns were as below:

ride_id - 0,
rideable_type - 0,
started_at - 0 ,
ended_at - 0,
start_station_name - 151918,
start_station_id - 152050,
end_station_name - 161646,
end_station_id - 161787,
start_lat - 0,
start_lng - 0,
end_lat - 861,
end_lng - 861,
member_casual - 0

#### Check for the rideable type
Check the different types of bike in our data and number of trips by each bike type. 

```
SELECT DISTINCT (rideable_type), COUNT (rideable_type) AS total_trips,
FROM Cyclistic_bike_share_data.cyclistic_q1
GROUP BY
rideable_type
```
![image](https://github.com/deepika-wadhawa/GDA-Capstone_Cyclistic-Case_Study/assets/157010535/0958ccda-3909-4e56-99cb-93ba52c25589)

There are 3 unique bike types: electric, classic and docked. 

#### Create new table 
I have decided to remove null values. 
I am working on BigQuery Sandbox, so I cannot use delete function to remove nulls, hence I will create a new table.
In the new table I will also add new column __day_of_week__ and a calculative column __ride_length__ which will be calculated in minutes by substracting column __started_at__ from the column __ended_at__. 

```
CREATE TABLE Cyclistic_bike_share_data.cyclistic_q1_final AS
SELECT *, 
ROUND(((unix_seconds(ended_at) - unix_seconds(started_at))/60),2) AS ride_length_minutes,--converted started and ended into seconds and divided by 60 to convert to minutes and rounded final value to 2 decimals
FORMAT_DATETIME('%A', started_at) AS day_of_week
FROM Cyclistic_bike_share_data.cyclistic_q1
WHERE 
start_station_name IS NOT NULL AND
start_station_id IS NOT NULL AND
end_station_name IS NOT NULL AND
end_station_id IS NOT NULL AND
end_lat IS NOT NULL AND
end_lng IS NOT NULL
```
The new table returned 822,488 number of rows as compared to 1066014 prior.

## Analyze

#### Calculate the number of rides taken by each user type (e.g., casual vs. member).

```
SELECT 
COUNT(ride_id),
FROM `Cyclistic_bike_share_data.cyclistic_q1_final`
WHERE 
member_casual = 'casual'
```
 Total number of rides taken by casual is 219727

```
SELECT 
COUNT(ride_id),
FROM `Cyclistic_bike_share_data.cyclistic_q1_final`
WHERE 
member_casual = 'member'
```
 Total number of rides taken by member is 602761

 This shows the members are already taking more rides than casuals.


#### Analyze total number of rides taken on weekend and weekdays

```
SELECT 
Count(ride_id) AS total_rides
FROM Cyclistic_bike_share_data.cyclistic_q1_final
WHERE 
member_casual = 'casual'AND
day_of_week IN ('Saturday','Sunday')
````
```
SELECT 
Count(ride_id)
FROM Cyclistic_bike_share_data.cyclistic_q1_final
WHERE 
member_casual = 'casual'AND
day_of_week IN ('Monday','Tuesday', 'Wednesday', 'Thursday', 'Friday')
```

This returns that member took total 72929 rides on weekends and 146798 rides in weekdays (total rides of casual 219727)


```
SELECT 
Count(ride_id) AS total_rides
FROM Cyclistic_bike_share_data.cyclistic_q1_final
WHERE 
member_casual = 'member'AND
day_of_week IN ('Saturday','Sunday')
```

```
SELECT 
Count(ride_id)
FROM Cyclistic_bike_share_data.cyclistic_q1_final
WHERE 
member_casual = 'memebr' AND
day_of_week IN ('Monday','Tuesday', 'Wednesday', 'Thursday', 'Friday')
```
This returns that member took total 131004 rides on weekends and 471757 rides in weekdays (total rides of member are 602761)

 #### Analyze the average trip duration for different user types

 ```
SELECT 
AVG(ride_length_minutes)
FROM `my-first-project-260523.Cyclistic_bike_share_data.cyclistic_q1_final`
WHERE 
member_casual = 'casual'
```
Average trip duration for casual user type is 19.58 minutes


```
SELECT 
AVG(ride_length_minutes)
FROM `my-first-project-260523.Cyclistic_bike_share_data.cyclistic_q1_final`
WHERE 
member_casual = 'member'
```
Average trip duration for member user type is 10.67 minutes

The trip duration for casual is more than members. 


