# Homework for week 2 of DE Zoomcamp

### Question 1 - load January 2020 data

The dataset has 447770 rows.

Corresponding Python code:


```
from pathlib import Path
import pandas as pd
from prefect import flow,task
from prefect_gcp.cloud_storage import GcsBucket



@task(retries=3)
def fetch(dataset_url: str) -> pd.DataFrame: 
    """Read data from web into pandas df"""


    df = pd.read_csv(dataset_url)
    return df

@task(log_prints=True)
def clean(df = pd.DataFrame) -> pd.DataFrame:
    """Fix dtype issues"""
    df['lpep_pickup_datetime'] = pd.to_datetime(df['lpep_pickup_datetime'])
    df['lpep_dropoff_datetime'] = pd.to_datetime(df['lpep_dropoff_datetime'])
    print(df.head(2))
    print(f"columns: {df.dtypes}")
    print(f"rows: {len(df)}")
    return df

@task()
def write_local(df: pd.DataFrame, color: str, dataset_file: str) -> Path:
    """Write df out locally as parquet file"""
    path = Path(f"data/{color}/{dataset_file}.parquet")
    df.to_parquet(path, compression="gzip")
    return path

@task()
def write_gcs(path: Path) -> None: #we are not returning anything here
    """Upload local parquet file to GCS"""
    #gcs_block = GcsBucket.load("zoom-terraform") # to try it with the already created bucket in the terraform lesson
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.upload_from_path(
        from_path=f"{path}",
        to_path=path
    )
    return 



@flow()
def etl_web_to_gcs() -> None:
    """The main ETL function"""
    color = "green"
    year = 2020
    month = 1
    dataset_file = f"{color}_tripdata_{year}-{month:02}"
    dataset_url = f"https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{color}/{dataset_file}.csv.gz"

    df = fetch(dataset_url)
    df_clean = clean(df)
    path = write_local(df_clean, color, dataset_file)
    write_gcs(path)

if __name__ == '__main__':
    etl_web_to_gcs()



```


### Question 2 - scheduling with Cron

The cron schedule for a run on the first of every month at 5am UTC is 0 5 1 * *

The corresponding lines in the command line:

`prefect deployment build etl_web_to_gcs.py:etl_web_to_gcs -n homework2 --cron "0 5 1 * * " -a`

`prefect agent start -q 'default'`

### Question 3 - loading data to BigQuery

First: load two tables with python etl_web_to_gcs.py 



prefect deployment build etl_gcs_to_bq.py:etl_parent_flow -n "homework_yellow"

prefect deployment apply etl_parent_flow-deployment.yaml

start agent
prefect agent start -q 'default'

processed lines:
14,851,920


corresponding code

```
from pathlib import Path
import pandas as pd
from prefect import flow,task
from prefect_gcp.cloud_storage import GcsBucket
from prefect_gcp import GcpCredentials


@task(retries=3)
def extract_from_gcs(color: str, year: int, month: int) -> Path:
    """Download trip data from GCS"""
    gcs_path = f"data/{color}/{color}_tripdata_{year}-{month:02}.parquet"
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.get_directory(from_path=gcs_path, local_path=f"../data/")
    return Path(f"{gcs_path}")

@task(log_prints=True)
def transform(path: Path) -> pd.DataFrame:
    """Data cleaning example (very minimal)"""
    df = pd.read_parquet(path)
    return df

@task()
def write_bq(df: pd.DataFrame) -> None:
    """Write DF to BigQuery
    We are using pandas BigQuery functions"""
    
    gcp_credentials_block = GcpCredentials.load("zoom-gcp-creds")

    df.to_gbq(
        destination_table="dezoomprefect.rides",
        project_id="sonorous-house-375411",
        credentials=gcp_credentials_block.get_credentials_from_service_account(),
        chunksize=500000,
        if_exists="append",
    )

@flow(log_prints=True)
def etl_gcs_to_bq(year: int, month: int, color: str):
    """main etl flow to load data into BigQuery"""

    path = extract_from_gcs(color, year, month)
    df = transform(path)
    write_bq(df)
    print("Transform Complete. Length of df: " + str(len(df)) + " rows.")


@flow()
def etl_parent_flow(
    months: list[int] = [2, 3], year: int = 2019, color: str = "yellow"
):
    for month in months:
        etl_gcs_to_bq(year, month, color)

if __name__ == '__main__':
    color = "yellow"
    months = [2, 3]
    year = 2021
    etl_parent_flow(months, year, color)


```

### Question 4 - Github Storage Block

1. create github_deploy.py, push to github
2. make Block gh-block
3. command line: `prefect deployment build github_deploy.py:etl_web_to_gcs --name task4 --apply -sb github/gh-block`
4. command line: `prefect agent start -q 'default'`
5. "quick run" in the UI:

88605 rows are processed


### Question 5 - Email or Slack notifications

I used Prefect Cloud to setup up a notification system via Email.

514392 rows were processed.

Received the following email:

Flow run etl-web-to-gcs/quirky-dingo entered state `Completed` at 2023-02-03T14:38:19.054290+00:00.
Flow ID: a343be8e-fff3-4922-9312-9ff72cca7d2b
Flow run ID: 2de069e1-71d5-4dcf-b037-4d3b0ee205c5
Flow run URL: https://app.prefect.cloud/account/cecdbbf0-5b0f-40a3-9b04-ab02393bf8b2/workspace/590c3c49-4407-4232-864e-e82faaa953b6/flow-runs/flow-run/2de069e1-71d5-4dcf-b037-4d3b0ee205c5
State message: All states completed.



### Question 6 - Secrets

```
from prefect.blocks.system import Secret

secret_block = Secret.load("secret-homework")

# Access the stored secret
secret_block.get()

```

8
