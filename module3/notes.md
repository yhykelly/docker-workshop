## Module 3 Data Warehousing and BQ

### power of partitioning

```{sql}
-- 1
select PULocationID from `DEzoomcamp_module3.yello-taxi-trip-records_external_table` where tpep_dropoff_datetime BETWEEN '2019-01-01' AND '2019-03-31';

-- 2
select PULocationID from `DEzoomcamp_module3.yellow_taxi_trip_records` where tpep_dropoff_datetime BETWEEN '2019-01-01' AND '2019-03-31';

-- 3
select PULocationID from `DEzoomcamp_module3.yello-taxi-trip-records_external_table_partitioned` where tpep_dropoff_datetime BETWEEN '2019-01-01' AND '2019-03-31';
```

method 1: table is external (outside of BQ and lives in GCS)
method 2: table is in BQ, not partitioned: uses 330 MB to query
method 3: table is in BQ, partitioned: uses 0B to query!

although external tables are cheap --  
1. it is slower than query native tables using BQ.  
2. with external tables we cannot do partitioning and clustering.  
3. Limited caching in external tables. Each time the query needs to reread from GCS

### what is partitioning and clustering?
when we create a dataset, we have certain column that is almost always ussed as a filter, for example some `TIMESTAMP` data. With partition, the data is divided into partitions based on this column, and allowing BQ to prune the entire partitions during query execution instead of scanning the full table. It is relevant to how data is stored and accessed.  DO NOT MIX UP WITH `GROUP BY`! (`GROUP BY` used to aggregate data from the table)  

When queries need to filter on additional columns other than the partition key, we can use clustering. Clustering organizes the data further within each partition. Clustering can involve more columns. BQ can hence reduce the data scanned for irrelevant data blocks.