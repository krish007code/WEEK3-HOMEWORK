# WEEK3-HOMEWORK
## 1. Data Loading
The data was uploaded to a GCS bucket. The dataset consists of 6 Parquet files: yellow_tripdata_2024-01.parquet through yellow_tripdata_2024-06.parquet.

## 2. Create External Table

  CREATE OR REPLACE EXTERNAL TABLE `krish-485219.trips_data_all.external_yellow_tripdata`
  OPTIONS (
    format = 'PARQUET',
    uris = ['gs://your-bucket-name/yellow_tripdata_2024-*.parquet']
  );
  
## 3. Create Native (Materialized) Table
<em>To create a non-partitioned table in BigQuery storage:</em>

  CREATE OR REPLACE TABLE `krish-485219.trips_data_all.yellow_tripdata_non_partitioned` AS
  SELECT * FROM `krish-485219.trips_data_all.external_yellow_tripdata`;

## Questions
<strong>Question 1: Counting records</strong>
query
  SELECT count(*) FROM `krish-485219.trips_data_all.yellow_tripdata_non_partitioned`;
Answer: 20,332,093
Question 2: Data read estimation
Answer: 0 MB for the External Table and 155.12 MB for the Materialized Table
Question 3: Understanding columnar storage
Answer: BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
Question 4: Counting zero fare trips
  SELECT count(*) 
  FROM `krish-485219.trips_data_all.yellow_tripdata_non_partitioned` 
  WHERE fare_amount = 0;
Answer: 8,333
Question 5: Partitioning and clustering
  CREATE OR REPLACE TABLE `krish-485219.trips_data_all.yellow_tripdata_partitioned_clustered`
  PARTITION BY DATE(tpep_dropoff_datetime)
  CLUSTER BY VendorID AS
  SELECT * FROM `krish-485219.trips_data_all.external_yellow_tripdata`;
 Answer: Partition by tpep_dropoff_datetime and Cluster on VendorID

Question 6. Partition benefits
  SELECT DISTINCT VendorID
  FROM `krish-485219.trips_data_all.yellow_tripdata_partitioned_clustered`
  WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';
  Answer: 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table

Question 7: External table storage
Answer: GCP Bucket

Question 8: Clustering best practices
Answer: False

Question 9: Understanding table scans 
0 bytes

