# Part 1: Docker & SQL

### Question 1 - knowing docker tags:

`docker --help` tells us we need to run `docker build --help` to find out more about the `docker build` command.

We can see that  `--iidfile string` corresponds to "Write the image ID to the file".


### Question 2 - understanding docker first run:

The following command runs docker with the python:3.9 image in an interactive mode and the entrypoint of bash:

`docker run -it --entrypoint=bash python:3.9`

Running `pip list` in the bash prompt tells us that there are 3 packages installed.


### Question 3 - count records:

Start pgadmin & postgres in one network with docker compose

Contents of the .yaml-file:
`
services:

  pgdatabase:
  
    image: postgres:13
    
    environment:
    
    - POSTGRES_USER=root
    
    - POSTGRES_PASSWORD=root
    
    - POSTGRES_DB=ny_taxi
    
    volumes:
    
    - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
    
    ports:
    
    - "5431:5432"
  
  pgadmin:
  
  image: dpage/pgadmin4
  
  environment:
  
  - PGADMIN_DEFAULT_EMAIL=admin@admin.com
  
  - PGADMIN_DEFAULT_PASSWORD=root
  
  ports:
  
  - "8080:80"
`

Insert green taxi trips data via jupyter notebook:

`
import pandas as pd
from time import tme
from sqlalchemy import create_engine

engine = create_engine('postgresql://root:root@localhost:5431/ny_taxi')

!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz

df_iter_green = pd.read_csv('green_tripdata_2019-01.csv.gz', iterator=True, chunksize=100000)

green_taxi_data.head(n=0).to_sql(name='green_taxi_data', con=engine, if_exists='replace')

while True:
    t_start = time()

    green_taxi_data = next(df_iter_green)
    
    green_taxi_data.lpep_pickup_datetime = pd.to_datetime(green_taxi_data.lpep_pickup_datetime)
    green_taxi_data.lpep_dropoff_datetime = pd.to_datetime(green_taxi_data.lpep_dropoff_datetime)
    
    green_taxi_data.to_sql(name='green_taxi_data', con=engine, if_exists='append')
    
    t_end = time()
    
    print('inserted another chunk..., took %.3f seconds' % (t_end - t_start))
 
 !wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
 
 df_zones = pd.read_csv('taxi_zone_lookup.csv')
 
 df_zones.to_sql(name='zones', con=engine, if_exists='replace')
`

How many trips were made on January 15th?

&rarr; 20530

```
SELECT
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-15 00:00:00' AND lpep_dropoff_datetime < '2019-01-15 23:59:59';

```

### Question 4 - largest trip for each day:

2019-01-18: 80,96
```
SELECT
	trip_distance,
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-18 00:00:00' AND lpep_dropoff_datetime < '2019-01-18 23:59:59'
ORDER BY trip_distance DESC;
```
2019-01-28: 64,27
```
SELECT
	trip_distance,
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-28 00:00:00' AND lpep_dropoff_datetime < '2019-01-28 23:59:59'
ORDER BY trip_distance DESC;
```

2019-01-15: 117.99
```
SELECT
	trip_distance,
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-15 00:00:00' AND lpep_dropoff_datetime < '2019-01-15 23:59:59'
ORDER BY trip_distance DESC;
```

2019-01-10: 64.2

```
SELECT
	trip_distance,
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-10 00:00:00' AND lpep_dropoff_datetime < '2019-01-10 23:59:59'
ORDER BY trip_distance DESC;
```


### Question 5 - the number of passengers

2: 1282 ; 3: 254

```
SELECT
	COUNT(passenger_count)
FROM green_taxi_data
WHERE passenger_count = 2 AND lpep_pickup_datetime > '2019-01-01 00:00:00' AND lpep_dropoff_datetime < '2019-01-01 23:59:59';
```
and 

``` 

SELECT
	COUNT(passenger_count)
FROM green_taxi_data
WHERE passenger_count = 3 AND lpep_pickup_datetime > '2019-01-01 00:00:00' AND lpep_dropoff_datetime < '2019-01-01 23:59:59';
```




