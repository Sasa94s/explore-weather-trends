# Explore Weather Trends

## Summary

Analyzing local and global temperature data and compare the temperature trends in Alexandria (EG) to overall global temperature trends.

## Prerequisites

- Download [Anaconda](https://conda.io/miniconda.html) environment
- Install [sql_magic](https://github.com/pivotal-legacy/sql_magic) `pip install sql_magic`


## Preparing Data

- Download global temperature data and saved as `global-data.csv`
```
Select *
FROM global_data;
```
- Download cities temperature data and saved as `local-data.csv`
```
SELECT *
FROM city_data;
```
- Download the temperature data for Alexandria and saved as `alexandria-city-data.csv`
```
SELECT *
FROM city_data
WHERE country = 'Egypt'
AND city = 'Alexandria';
```

## Getting Started

1. Download and Install [PostgreSQL](https://www.postgresql.org/download/) 
Note: Make sure you install **SQL Shell** and **Command Line Tools**, there is no need for *pgAdmin4* graphical interface.
2. Launch **SQL Shell (psql)**
3. Create `weather` database
```
DROP DATABASE IF EXISTS weather;

CREATE DATABASE weather;
```
4. Connect to `weather` database
```
\c weather
```
5. Create both tables `global_data` and `city_data`
```
CREATE TABLE global_data (
	year	 TEXT,
	avg_temp	 NUMERIC
);

CREATE TABLE city_data (
	year	 TEXT,
	city	 TEXT,
	country	 TEXT,
	avg_temp	 NUMERIC
);
```
6. Import CSV data to the both tables
```
SET CLIENT_ENCODING TO 'utf8';

\copy city_data FROM '\explore-weather-trends\data\local-data.csv' DELIMITER ',' CSV HEADER;

\copy global_data FROM '\explore-weather-trends\data\global-data.csv' DELIMITER ',' CSV HEADER;
```