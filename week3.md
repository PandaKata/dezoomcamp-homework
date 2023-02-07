# Homework for week 3 of DE Zoomcamp


Creating external table with gcs path:

```
CREATE OR REPLACE EXTERNAL TABLE `sonorous-house-375411.nytaxi.external_fhv_2019`
OPTIONS (
  format = 'CSV',
  uris = ['gs://nyc-tl-data_23/trip-data/fhv_tripdata_2019-*.csv.gz']
);
```
creating a non partitioned table from external table:
```
CREATE OR REPLACE TABLE sonorous-house-375411.nytaxi.external_fhv_2019_non_partitoned AS
SELECT * FROM sonorous-house-375411.nytaxi.external_fhv_2019;
```

### Question 1 

count rows:

```
SELECT COUNT(1) 
FROM `sonorous-house-375411.nytaxi.external_fhv_2019` 
LIMIT 1000
```
&rarr; 43,244,696 rows

### Question 2

zero for external, 317.94 for table

```
SELECT COUNT(DISTINCT(affiliated_base_number))
FROM sonorous-house-375411.nytaxi.external_fhv_2019;
```

### Question 3:


```
SELECT COUNT(*)
FROM sonorous-house-375411.nytaxi.external_fhv_2019
WHERE DOLocationID IS NULL AND PULocationID IS NULL;
```
&rarr; in 717748 cases both values are NULL

### Question 4:

&rarr; Partition by pickup_datetime Cluster on affiliated_base_number

created partitioned & clustered table (his query will process 1.92 GB when run.):

```
CREATE OR REPLACE TABLE sonorous-house-375411.nytaxi.fhv_2019_partitioned_clustered
PARTITION BY DATE(pickup_datetime)
CLUSTER BY affiliated_base_number AS
SELECT * FROM sonorous-house-375411.nytaxi.external_fhv_2019_non_partitoned;
```


### Question 5:

Query for partitioned & clustered table:
```
-- This query will process 23.05 MB when run.
SELECT COUNT(DISTINCT(affiliated_base_number))
FROM sonorous-house-375411.nytaxi.fhv_2019_partitioned_clustered
WHERE DATE(pickup_datetime) BETWEEN '2019-03-01' AND '2019-03-31';
```


Query for table that is neither partitioned nor clusteres

```
-- This query will process 647.87 MB when run.
SELECT COUNT(DISTINCT(affiliated_base_number))
FROM sonorous-house-375411.nytaxi.external_fhv_2019_non_partitoned
WHERE DATE(pickup_datetime) BETWEEN '2019-03-01' AND '2019-03-31';
```

### Question 6:

&rarr; GCP Bucket (external table means: the data itself is not in BQ, it is in an external system (GCS))


### Question 7:

No, clustering is not always the best practice
