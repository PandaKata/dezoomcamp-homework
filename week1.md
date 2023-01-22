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

# add green taxi trips
!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz

df_iter_green = pd.read_csv('green_tripdata_2019-01.csv.gz', iterator=True, chunksize=100000)

# add columns
green_taxi_data.head(n=0).to_sql(name='green_taxi_data', con=engine, if_exists='replace')

# ingest data
while True:
    t_start = time()

    green_taxi_data = next(df_iter_green)
    
    green_taxi_data.lpep_pickup_datetime = pd.to_datetime(green_taxi_data.lpep_pickup_datetime)
    green_taxi_data.lpep_dropoff_datetime = pd.to_datetime(green_taxi_data.lpep_dropoff_datetime)
    
    green_taxi_data.to_sql(name='green_taxi_data', con=engine, if_exists='append')
    
    t_end = time()
    
    print('inserted another chunk..., took %.3f seconds' % (t_end - t_start))
 
 # add lookup table
 !wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
 
 df_zones = pd.read_csv('taxi_zone_lookup.csv')
 
 df_zones.to_sql(name='zones', con=engine, if_exists='replace')
`

How many trips were made on January 15th?

&rarr; 20530

`
SELECT
	lpep_pickup_datetime,
	lpep_dropoff_datetime
FROM green_taxi_data 
WHERE 
	lpep_pickup_datetime > '2019-01-15 00:00:00' AND lpep_dropoff_datetime < '2019-01-15 23:59:59';

`

### Question 4 -l argest trip for each day:

