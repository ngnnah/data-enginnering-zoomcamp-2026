# Module 1 Homework

## Setup and Running

### Build Docker Image

This Dockerfile creates the containerized application for the taxi data ingestion service.

```bash
docker build -t taxi_ingest:v001 .
```

### Start Services

In another terminal:

```bash
docker-compose up
```

### Verify Ingestion

Check docker-compose logs:

```bash
docker-compose logs
```

Expected output:

```bash
taxi_ingest  | Successfully ingested 265 taxi zones
taxi_ingest  | Ingesting green taxi trip data from https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet...
```

## Accessing pgAdmin

Navigate to http://127.0.0.1:8085/browser/ to access pgAdmin and view the data.

### Verify Data

```sql
-- 46912 records
SELECT COUNT(*) FROM public.green_trip_data

-- 265 rows
SELECT * FROM taxi_zones
```

## Homework Questions

### Question 3

Count trips in November 2025 with trip distance less than or equal to 1 mile:

```sql
SELECT COUNT(*) FROM green_trip_data
WHERE lpep_pickup_datetime >= '2025-11-01' 
  AND lpep_pickup_datetime < '2025-12-01' 
  AND trip_distance <= 1
```

### Question 4

Which was the pick up day with the longest trip distance? Only consider trips with `trip_distance` less than 100 miles (to exclude data errors).

```sql
SELECT 
    DATE(lpep_pickup_datetime) as pickup_day,
    MAX(trip_distance) as max_distance
FROM green_trip_data
WHERE trip_distance < 100
GROUP BY DATE(lpep_pickup_datetime)
ORDER BY max_distance DESC
LIMIT 1;
```

### Question 5

Top 10 pickup zones by total amount on November 18, 2025:

```sql
SELECT 
    zone."Zone", 
    SUM(trip.total_amount) AS total_amount
FROM green_trip_data trip
JOIN taxi_zones zone
ON trip."PULocationID" = zone."LocationID"
WHERE DATE(trip.lpep_pickup_datetime) = '2025-11-18'
GROUP BY zone."Zone"
ORDER BY total_amount DESC
LIMIT 10;
```

### Question 6

For the passengers picked up in the zone named "East Harlem North" in November 2025, which was the drop off zone that had the largest tip? 

**Note**: it's `tip`, not `trip`. We need the name of the zone, not the ID.

```sql
SELECT 
    zone."Zone" as pu_zone, 
    dropoff."Zone" as do_zone,
    trip.tip_amount
FROM green_trip_data trip
JOIN taxi_zones zone
ON trip."PULocationID" = zone."LocationID"
JOIN taxi_zones dropoff
ON trip."DOLocationID" = dropoff."LocationID"
WHERE DATE(trip.lpep_pickup_datetime) >= '2025-11-01'
  AND DATE(trip.lpep_pickup_datetime) < '2025-12-01'
  AND zone."Zone" = 'East Harlem North'
ORDER BY tip_amount DESC 
LIMIT 10;
```
