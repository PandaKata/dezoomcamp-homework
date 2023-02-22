# Homework for week 5 of DE Zoomcamp

### Question 1

&rarr; 3.3.1

```
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .master('local[*]') \
    .appName('test') \
    .getOrCreate()
    

spark.version
```

### Question 2

&rarr; 24MB

```
from pyspark.sql import types

!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/fhvhv/fhvhv_tripdata_2021-06.csv.gz

schema = types.StructType([
    types.StructField('dispatching_base_num', types.StringType(), True),
    types.StructField('pickup_datetime', types.TimestampType(), True),
    types.StructField('dropoff_datetime', types.TimestampType(), True),
    types.StructField('PULocationID', types.IntegerType(), True),
    types.StructField('DOLocationID', types.IntegerType(), True),
    types.StructField('SR_Flag', types.StringType(), True),
    types.StructField('Affiliated_base_number', types.StringType(), True)
])

df = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv('fhvhv_tripdata_2021-06.csv.gz')
 
df = df.repartition(12)

df.write.parquet('fhvhv_2021_06')
```


### Question 3

&rarr; 452,470

```
from pyspark.sql import functions as F


df \
    .withColumn('pickup_date', F.to_date(df.pickup_datetime)) \
    .filter("pickup_date = '2021-06-15'") \
    .count()

```


### Question 4

&rarr; 66.87 Hours

```
df.registerTempTable('rides')

spark.sql("""
SELECT to_date(pickup_datetime) AS pickup_date, MAX((CAST(dropoff_datetime AS LONG) - CAST(pickup_datetime AS LONG)) / 60 / 60) AS duration
FROM rides
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
""").show()

```

### Question 5

&rarr; 4040


### Question 6

&rarr; Crown Heights North

```
df_zones = spark.read.parquet('zones')

df_zones.registerTempTable('zones')

spark.sql("""
SELECT zones.Zone, Count(zones.Zone)
FROM rides
LEFT JOIN zones
ON rides.PULocationID = zones.LocationID
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
""").show()

```

