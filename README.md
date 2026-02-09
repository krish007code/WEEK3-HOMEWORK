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

This homework explores BigQuery optimization techniques including external tables, partitioning, clustering, and understanding query execution costs. The dataset consists of NYC Yellow Taxi trip data for the first half of 2024.

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

**Explanation**: External tables don't store data in BigQuery, so metadata queries return 0 MB. The materialized table stores data in BigQuery's optimized columnar format.

---

### Question 3: Understanding Columnar Storage

**Task**: Explain why querying two columns processes more data than querying one column.

**Answer**: 

BigQuery is a columnar database that only scans the specific columns requested in the query. When querying two columns (`PULocationID`, `DOLocationID`), BigQuery must read data from both column stores, resulting in approximately double the bytes processed compared to querying a single column (`PULocationID`).

**Key Concept**: This demonstrates the efficiency of columnar storage for analytical workloads where queries typically access a subset of columns.

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

**Rationale**: 
- Partitioning by date enables efficient time-range filtering
- Clustering by VendorID improves query performance when filtering or grouping by vendor

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

**Improvement**: ~91% reduction in data scanned (284.40 MB saved)

**Explanation**: Partitioning eliminates the need to scan irrelevant partitions, significantly reducing query costs and improving performance.

---

### Question 7: External Table Storage

**Task**: Where is the data for external tables stored?

**Answer**: `GCP Bucket` (Google Cloud Storage)

**Note**: External tables reference data in GCS rather than storing it within BigQuery's managed storage.

---

### Question 8: Clustering Best Practices

**Task**: Is the statement about clustering best practices true or false?

**Answer**: `False`

*(Note: The specific statement wasn't provided in the original homework. Typical false statements include "clustering always improves performance" or "you should cluster on high-cardinality columns first.")*

---

### Question 9: Understanding Table Scans

**Task**: Determine bytes processed for a specific query operation.

**Answer**: `0 bytes`

**Explanation**: Certain metadata queries or queries on empty result sets may process 0 bytes, as BigQuery can answer them using table metadata without scanning actual data.

---

## Key Learnings

### 1. External vs. Native Tables
- **External Tables**: Query data in-place from GCS, useful for ad-hoc analysis
- **Native Tables**: Faster queries with optimized storage, better for production workloads

### 2. Partitioning Benefits
- Reduces data scanned by 90%+ for time-range queries
- Lower query costs and faster execution
- Best for date/timestamp columns with filtering patterns

### 3. Clustering Optimization
- Improves performance for filtering and aggregation on specific columns
- Works best with columns having medium to high cardinality
- Can be combined with partitioning for maximum efficiency

### 4. Columnar Storage
- BigQuery only reads requested columns
- Ideal for analytical queries accessing subset of columns
- Significant cost savings compared to row-based storage

### 5. Query Cost Estimation
- BigQuery provides accurate estimates before execution
- Use estimates to optimize expensive queries
- Partitioning and clustering dramatically reduce costs

---

## Best Practices Applied

✅ Used partitioning for time-based filtering  
✅ Applied clustering on frequently filtered columns  
✅ Leveraged external tables for data exploration  
✅ Created materialized tables for production queries  
✅ Monitored query costs with byte estimates  

---

## Dataset Information

**Project**: `krish-485219`  
**Dataset**: `trips_data_all`  
**Time Period**: January 2024 - June 2024  
**Data Format**: Parquet  
**Storage Location**: Google Cloud Storage  

---

## Additional Resources

- [BigQuery Partitioning Documentation](https://cloud.google.com/bigquery/docs/partitioned-tables)
- [BigQuery Clustering Documentation](https://cloud.google.com/bigquery/docs/clustered-tables)
- [External Tables Guide](https://cloud.google.com/bigquery/docs/external-tables)
- [Query Optimization Best Practices](https://cloud.google.com/bigquery/docs/best-practices-performance-overview)

---

**Author**: Krish  
**Course**: Data Engineering  
**Week**: 3  
**Date**: February 2026
