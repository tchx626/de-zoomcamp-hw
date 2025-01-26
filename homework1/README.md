# Module 1 Homework: Docker & SQL

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

```bash
Invoke-WebRequest -Uri "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz" -OutFile "green_tripdata_2019-10.csv.gz"

Invoke-WebRequest -Uri "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv" -OutFile "taxi_zone_lookup.csv"
```

**Run database and database administrator**

Related files: `docker-compose.yaml`

```bash
docker-compose up
```



**Load data into database**

Related files: `Dockerfile` `ingest_data.py`

