# Week 3 Homework - BigQuery Optimization

## Table of Contents
- [Overview](#overview)
- [Data Setup](#data-setup)
  - [1. Data Loading](#1-data-loading)
  - [2. External Table Creation](#2-external-table-creation)
  - [3. Native Table Creation](#3-native-table-creation)
- [Questions and Solutions](#questions-and-solutions)
- [Key Learnings](#key-learnings)

---

## Overview

This is my homework explores BigQuery optimization.

**Total Records**: 20,332,093  
**Dataset**: `krish-485219.trips_data_all`

---
## Data Setup

### 1. Data Loading

The data was uploaded to a Google Cloud Storage (GCS) bucket consisting of 6 Parquet files:

- `yellow_tripdata_2024-01.parquet`
- `yellow_tripdata_2024-02.parquet`
- `yellow_tripdata_2024-03.parquet`
- `yellow_tripdata_2024-04.parquet`
- `yellow_tripdata_2024-05.parquet`
- `yellow_tripdata_2024-06.parquet`

### 2. External Table Creation

External tables allow querying data directly from GCS without loading it into BigQuery storage.

```sql
CREATE OR REPLACE EXTERNAL TABLE `krish-485219.trips_data_all.external_yellow_tripdata`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://your-bucket-name/yellow_tripdata_2024-*.parquet']
);
```

### 3. Native Table Creation

Creating a non-partitioned table in BigQuery native storage for comparison:

```sql
CREATE OR REPLACE TABLE `krish-485219.trips_data_all.yellow_tripdata_non_partitioned` AS
SELECT * FROM `krish-485219.trips_data_all.external_yellow_tripdata`;
```

---

## Questions and Solutions

### Question 1: Counting Records

**Task**: Count the total number of records in the dataset.

**Query**:
```sql
SELECT COUNT(*) 
FROM `krish-485219.trips_data_all.yellow_tripdata_non_partitioned`;
```

**Answer**: `20,332,093`

---

### Question 2: Data Read Estimation

**Task**: Compare the estimated bytes processed between external and materialized tables.

**Answer**: 
- **External Table**: `0 MB`
- **Materialized Table**: `155.12 MB`

---

### Question 3: Understanding Columnar Storage

**Task**: Explain why querying two columns processes more data than querying one column.

**Answer**: 

BigQuery is a columnar database that only scans the specific columns requested in the query. When querying two columns (`PULocationID`, `DOLocationID`), BigQuery must read data from both column stores, resulting in approximately double the bytes processed compared to querying a single column (`PULocationID`).

---

### Question 4: Counting Zero Fare Trips

**Task**: Count trips with zero fare amount.

**Query**:
```sql
SELECT COUNT(*) 
FROM `krish-485219.trips_data_all.yellow_tripdata_non_partitioned` 
WHERE fare_amount = 0;
```

**Answer**: `8,333`
---

### Question 5: Partitioning and Clustering

**Task**: Create an optimized table with partitioning and clustering.

**Query**:
```sql
CREATE OR REPLACE TABLE `krish-485219.trips_data_all.yellow_tripdata_partitioned_clustered`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `krish-485219.trips_data_all.external_yellow_tripdata`;
```

**Answer**: 
- **Partition by**: `tpep_dropoff_datetime` (date)
- **Cluster by**: `VendorID`
---

### Question 6: Partition Benefits

**Task**: Compare bytes processed between non-partitioned and partitioned tables.

**Query**:
```sql
SELECT DISTINCT VendorID
FROM `krish-485219.trips_data_all.yellow_tripdata_partitioned_clustered`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';
```

**Answer**: 
- **Non-partitioned table**: `310.24 MB`
- **Partitioned table**: `26.84 MB`

---

### Question 7: External Table Storage

**Task**: Where is the data for external tables stored?

**Answer**: `GCP Bucket` 

---

### Question 8: Clustering Best Practices

**Task**: Is the statement about clustering best practices true or false?

**Answer**: `False`
---

### Question 9: Understanding Table Scans

**Task**: Determine bytes processed for a specific query operation.

**Answer**: `0 bytes`

---

## Dataset Information

**Project**: `krish-485219`  
**Dataset**: `trips_data_all`  
**Time Period**: January 2024 - June 2024  
**Data Format**: Parquet  
**Storage Location**: Google Cloud Storage  

---

**Author**: Krish  
**Course**: Data Engineering  
**Week**: 3  
**Date**: February 2026
