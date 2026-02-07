#### BigQuery Setup

```{sql}
CREATE OR REPLACE EXTERNAL TABLE `DEzoomcamp_module3.yello-taxi-trip-records_external_table`
OPTIONS(
  format = 'parquet',
  uris = ['gs://learn-terraform-gcp-kestra-yhykelly/yellow_tripdata_2024-*.parquet']
);

CREATE OR REPLACE TABLE `DEzoomcamp_module3.yellow_taxi_trip_records`
AS
SELECT *
FROM `DEzoomcamp_module3.yello-taxi-trip-records_external_table`;
```

#### Question 1. Counting records
What is count of records for the 2024 Yellow Taxi Data?

ANS: 
20,332,093


#### Question 2. Data read estimation
count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

```{sql}
SELECT COUNT(distinct PULocationID) FROM DEzoomcamp_module3.yellow_taxi_trip_records
SELECT COUNT(distinct PULocationID) FROM `DEzoomcamp_module3.yello-taxi-trip-records_external_table`
```
**ANS:**  
0 MB for the External Table and 155.12 MB for the Materialized Table


#### Question 3. Understanding columnar storage
Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table.  
```{sql}
SELECT PULocationID, DOLocationID FROM DEzoomcamp_module3.yellow_taxi_trip_records
```

Why are the estimated number of Bytes different?   

**ANS:**  
BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.  


#### Question 4. Counting zero fare trips
```{sql}
select count(*) from `DEzoomcamp_module3.yello-taxi-trip-records_external_table`
where fare_amount = 0;
```

**ANS:**  
8,333


#### Question 5. Partitioning and clustering
What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)  
```{sql}

```
**ANS:**
Partition by tpep_dropoff_datetime and Cluster on VendorID


#### Question 6. Partition benefits
Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?  


```{sql}
select PULocationID from `DEzoomcamp_module3.yellow_taxi_trip_records` where tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2019-03-15';
select PULocationID from `DEzoomcamp_module3.yello-taxi-trip-records_external_table_partitioned` where tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2019-03-15';
```

**ANS:**   
310.24 MB for non-partitioned table and 26.84 MB for the partitioned table


#### Question 7. External table storage
Where is the data stored in the External Table you created?   

**ANS:**   
GCP Bucket


#### Question 8: Clustering best practices
It is best practice in Big Query to always cluster your data:  

**ANS:**   
False  

In BQ, Clustering does not let user preview the cost upfront.  Also depending on the data nature, if only one column needs to be filtered, partitioning is already enough. Clustering helps if queries further filter on secondary columns AND the table is large.


#### Question 9: Understanding table scans
Write a SELECT count(*) query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

**ANS:**   0 B because for count(*) the engine can reference the metadata of the materialized table and BQ does not need to scan the table. If we add condition filter, the engine will have to scan data and this will involve usage.