# Homework for week 3 of DE Zoomcamp

### Question 1 

Creating external table with gcs path:

```
CREATE OR REPLACE EXTERNAL TABLE `sonorous-house-375411.nytaxi.external_fhv_2019`
OPTIONS (
  format = 'CSV',
  uris = ['gs://nyc-tl-data_23/trip-data/fhv_tripdata_2019-*.csv.gz']
);
```
count rows

```

```
