---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 230
---

#snowflake

## Clustering

- Snowflake stores data in micro partitions. If data is correctly clustered, it can help with micro-partition pruning and make queries along a clustered dimension faster. On the other hand, if we try to get result along a dimension that is poorly clustered, we have to retrieve all micro-partitions
- table data is partitioned and stored in the order it was loaded. If the data was ordered prior to ingestion, it will be naturally clustered along the order dimension
- Snowflake maintains clustering metadata for the micro-partitions of a given table:
  - Total nb of micro-partitions
  - Nb of overlapping micro-partitions for a table column
  - Depth of overlapping micro-partitions = nb of micro-partitions I would have to look into to find one value

```sql
-- Create a table with cluster keys
CREATE TABLE T1 (C1 date, C2 string, C3 number) CLUSTER BY (MONTH(C1), C2)

-- ALTER a table to add cluster keys
ALTER TABLE T1 (C1 date, C2 string, C3 number) CLUSTER BY (C3)

-- Suspend reclustering
ALTER TABLE T1 SUSPEND RECLUSTER;
-- Resume reclustering
ALTER TABLE T1 RESUME RECLUSTER;

```

```text
Depth vs overlap

___          ____     __
___        ____           __
___     ____                  __

O: 3        O: 3      O: 0
D: 3        D: 2      D: 1

In the final case, we are in constant state
```

```sql
-- get clustering depth
SELECT SYSTEM$clustering_depth('TABLE', '(col1,col3)');

-- get all infos (see result below)
SELECT SYSTEM$clustering_information('TABLE', '(col1,col3)');
```

```json
{
"cluster_by_keys" : "LINEAR(col1, col3)",
"notes" : "...",
// total nb of micropartitions in the table
"total_partition_count" : XX,
// how many partitions have exclusively the key combination, the higher the better
"total_constant_partition_count" : XX,
// measures quality of clustering, the lower the better for overlap and depth
"average_overlaps" : XX,
"average_depth" : XX,
// distribution of the depth among the micropartitions
// Each depth is associated with the number of values that have said depth
// Ideally, we would want a high number for low depths
"partition_depth_histogram" : {
	"00000" : XX,
	"00001" : XX
	...
	}
}
```

- Clustering can degrade, particularly if the table is large and there are a lot of DML operations
- Snowflake support automatic clustering by designating one or more table columns as clustering keys (we can also define them on materialized views): the goal is to colocate date of the clustering key in the same micro-partitions
- Automatic clustering is a serverless feature and uses compute and storage cost
- Clustering should be reserved for large tables in the multi terabyte range where typical queries filter a large part of the data
- Clustering is better for tables that are not modified often
- Snowflake recommends a maximum of 3 or 4 columns with a cardinality that is not too low and not too high and with the lowest cardinality selected first

Example below

```sql
-- context
USE ROLE SYSADMIN;
USE DATABASE SNOWFLAKE_SAMPLE_DATA;
USE SCHEMA TPCDS_SF100TCL;


-- check cluster_by column to see if the table has a cluster key (will be empty if there is no cluster)
SHOW TABLES;
SELECT "name", "database_name", "schema_name", "cluster_by" FROM TABLE(result_scan(last_query_id()));


-- shows how the table is clustered according to the CLUSTERED keys
-- will throw an error if table is not clustered
SELECT SYSTEM$clustering_information('CATALOG_SALES');

-- shows how the table is clustered according to the SPECIFIED key(s)
SELECT SYSTEM$clustering_information('CATALOG_SALES', '(cs_sold_date_sk)');

-- very fast retrieval when using a good clustered key
-- query profile shows that we only scanned 4 micropartitions
SELECT CS_SOLD_DATE_SK, CS_SOLD_TIME_SK, CS_ITEM_SK, CS_ORDER_NUMBER FROM CATALOG_SALES
WHERE CS_SOLD_DATE_SK = 2451092 AND CS_ITEM_SK = 140947;

-- Monitor automatic clustering
USE ROLE ACCOUNTADMIN;
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;
SELECT START_TIME, END_TIME, CREDITS_USED, NUM_BYTES_RECLUSTERED, TABLE_NAME, SCHEMA_NAME, DATABASE_NAME
FROM AUTOMATIC_CLUSTERING_HISTORY;
```
