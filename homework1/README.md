# Module 1 Homework: Docker & SQL

> Solutions are included in this documentary.

## Question 1. Understanding docker first run

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

```bash
docker run -it --entrypoint bash python:3.12.8
```



What's the version of `pip` in the image?

```bash
pip --version
```



- **24.3.1**
- 24.2.1
- 23.3.1
- 23.2.1

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- **db:5432**

*Since pgAdmin and PostgreSQL are in the same Docker network, pgAdmin can  directly use the internal port 5432 of the PostgreSQL container.*

If there are more than one answers, select only one of them



##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.



### Solution

> All bash commands run in **PowerShell**.

**Data exploration**

Related files: `exploration.ipynb`

```powershell
Invoke-WebRequest -Uri "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz" -OutFile "green_tripdata_2019-10.csv.gz"

Invoke-WebRequest -Uri "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv" -OutFile "taxi_zone_lookup.csv"
```

**Run database and database administrator**

Related files: `docker-compose.yaml`

```powershell
docker-compose up
```

**Load data into database**

Related files: `Dockerfile` `ingest_data.py`

Build Docker container:

```powershell
docker build -t hw1:1.0 .
```

Find the name of the network:

```powershell
docker inspect homework1-pgdatabase-1 | findstr NetworkMode
```



Load `green_tripdata_2019-10.csv` as table `green_trips`, and  `taxi_zone_lookup.csv` as table `zones`:

```powershell
$URL1 = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz'

docker run -it `
   --network=homework1_default `
   hw1:1.0 `
     --user=root `
     --password=root `
     --host=homework1-pgdatabase-1 `
     --port=5432 `
     --db=ny_taxi `
     --table_name=green_trips `
     --url=$URL1 `
     --convert_dates
     
$URL2 = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv'

docker run -it `
   --network=homework1_default `
   hw1:1.0 `
     --user=root `
     --password=root `
     --host=homework1-pgdatabase-1 `
     --port=5432 `
     --db=ny_taxi `
     --table_name=zones `
     --url=$URL2 
```

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- **104,802;  198,924;  109,603;  27,678;  35,189**
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202

### Solution

```postgresql
SELECT
    COUNT(1) FILTER (WHERE trip_distance <= 1.0) AS distance_0_1,
    COUNT(1) FILTER (WHERE trip_distance <= 3.0 AND trip_distance > 1.0) AS distance_1_3,
    COUNT(1) FILTER (WHERE trip_distance <= 7.0 AND trip_distance > 3.0) AS distance_3_7,
    COUNT(1) FILTER (WHERE trip_distance <= 10.0 AND trip_distance > 7.0) AS distance_7_10,
    COUNT(1) FILTER (WHERE trip_distance > 10.0) AS distance_10_plus
FROM green_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
  AND lpep_dropoff_datetime < '2019-11-01 00:00:00';
```

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- **2019-10-31**

### Solution

```postgresql
SELECT lpep_pickup_datetime
FROM green_trips
ORDER BY trip_distance DESC
LIMIT 1;
```

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in `total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.

- **East Harlem North, East Harlem South, Morningside Heights**
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

### Solution

```postgresql
SELECT
    z."Zone" AS pickup_location_name
FROM
    green_trips t
JOIN
    zones z ON t."PULocationID" = z."LocationID"
WHERE
    t.lpep_pickup_datetime >= '2019-10-18 00:00:00'
    AND t.lpep_pickup_datetime < '2019-10-19 00:00:00'
GROUP BY
    z."Zone"
HAVING
    SUM(t."total_amount") > 13000
ORDER BY
    SUM(t."total_amount") DESC;
```

## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- **JFK Airport**
- East Harlem North
- East Harlem South

### Solution

```postgresql
SELECT
    z2."Zone" AS drop_off_zone
FROM
    green_trips t
JOIN
    zones z1 ON t."PULocationID" = z1."LocationID" 
JOIN
    zones z2 ON t."DOLocationID" = z2."LocationID" 
WHERE
    z1."Zone" = 'East Harlem North' 
    AND t.lpep_pickup_datetime >= '2019-10-01 00:00:00'
    AND t.lpep_pickup_datetime < '2019-11-01 00:00:00'
ORDER BY
    t."tip_amount" DESC
LIMIT 1;
