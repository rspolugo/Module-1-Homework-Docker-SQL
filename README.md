# Module 1 Homework: Docker and SQL

## Task 1: Run Docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

   ```bash
   $ winpty docker run -it --entrypoint bash python:3.12.8
   root@1b36c9a0d91e:/# pip --version
   pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
   ```
## Task 2: Understanding Docker networking and docker-compose
```bash
db:5432
```
## Task 3:  Trip Segmentation Count
```bash
WITH filtered_trips AS (
    SELECT *
    FROM green_taxi_data
    WHERE "lpep_pickup_datetime" >= '2019-10-01 00:00:00'
      AND "lpep_pickup_datetime" < '2019-11-01 00:00:00'
      AND "trip_distance" >= 0  
)
SELECT
    SUM(CASE WHEN "trip_distance" <= 1 THEN 1 ELSE 0 END) AS "Up_to_1_mile",
    SUM(CASE WHEN "trip_distance" > 1 AND "trip_distance" <= 3 THEN 1 ELSE 0 END) AS "1_to_3_miles",
    SUM(CASE WHEN "trip_distance" > 3 AND "trip_distance" <= 7 THEN 1 ELSE 0 END) AS "3_to_7_miles",
    SUM(CASE WHEN "trip_distance" > 7 AND "trip_distance" <= 10 THEN 1 ELSE 0 END) AS "7_to_10_miles",
    SUM(CASE WHEN "trip_distance" > 10 THEN 1 ELSE 0 END) AS "Over_10_miles"
FROM filtered_trips;

answer is:
104,838; 199,013; 109,645; 27,688; 35,202 
```

## Task 4:  Longest trip for each day

```bash
WITH daily_max_trips AS (
    SELECT
        DATE("lpep_pickup_datetime") AS "pickup_day",
        MAX("trip_distance") AS "max_trip_distance"
    FROM green_taxi_data
    WHERE "lpep_pickup_datetime" >= '2019-10-01 00:00:00'
      AND "lpep_pickup_datetime" < '2019-11-01 00:00:00'
    GROUP BY DATE("lpep_pickup_datetime")
)
SELECT
    "pickup_day",
    "max_trip_distance"
FROM daily_max_trips
ORDER BY "max_trip_distance" DESC
LIMIT 1;

answer is:
"2019-10-31"	515.89
so
2019-10-31
```

## Task 5:  Three biggest pickup zones

```bash
WITH daily_total_amount AS (
    SELECT
        "PULocationID",
        SUM("total_amount") AS "total_amount"
    FROM green_taxi_data
    WHERE "lpep_pickup_datetime"::date = '2019-10-18'
    GROUP BY "PULocationID"
    HAVING SUM("total_amount") > 13000
)
SELECT
    z."LocationID",
    z."Borough",
    z."Zone",
    d."total_amount"
FROM daily_total_amount d
JOIN zones z ON d."PULocationID" = z."LocationID"
ORDER BY d."total_amount" DESC
LIMIT 3;

answer is:
East Harlem North, East Harlem South, Morningside Heights
```

## Task 6:  Largest tip
```bash
WITH filtered_trips AS (
    SELECT
        gtd."PULocationID",
        gtd."DOLocationID",
        gtd."tip_amount"
    FROM green_taxi_data gtd
    JOIN zones pu ON gtd."PULocationID" = pu."LocationID"
    WHERE gtd."lpep_pickup_datetime"::date >= '2019-10-01'
      AND gtd."lpep_pickup_datetime"::date < '2019-11-01'
      AND pu."Zone" = 'East Harlem North'
)
SELECT
    z."Zone" AS "Dropoff_Zone",
    ft."tip_amount" AS "Max_Tip"
FROM filtered_trips ft
JOIN zones z ON ft."DOLocationID" = z."LocationID"
ORDER BY ft."tip_amount" DESC
LIMIT 1;

answer is:
"JFK Airport"	87.3
JFK Airport
```