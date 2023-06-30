---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 230
---

#snowflake

## Caching

- Caching can occur at the services layer with metadata and results cache and at the warehouse level (local disk cache)

- Metadata cache refers to the store of statistics on the database objects (services layer).

  - This makes it so that some queries can be completed without requiring a running virtual warehouse
  - `SELECT COUNT(*)`, system functions, `DESCRIBE`, `SHOW` use this cache

- Results cache stores results of a query for reuse (services layer)

  - Also referred as the 24 hour cache, the query results cache...
  - Stores result of a query for 24 hour but each time it is reused, it is extended by 24 hours up until 31 days after which result is purged
  - For the cache to be used
    - new query must be identical to cached query (even adding a LIMIT bypasses the cache)
    - the underlying table data must NOT have changed
    - The same role (not user) is used as in the previous query
    - Time context functions such as `CURRENT_TIME()` must NOT be part of the query
  - Results reuse can also be disabled by using the session parameter `USE_CACHED_RESULT`

- Local disk cache (query processing layer) refers to the local ssd storage for the node on the warehouse cluster

  - Also referred as ssd cache, data cache, raw data cache
  - Improves perfs of queries in the same warehouse
  - [[LRU Cache|LRU policy]] is used
  - Purged when virtual warehouse is resized, suspended or dropped
  - Unlike results cache, can be used partially, retrieving the rest of the data required for a query from remote storage

- When a query can not use metadata cache, results cache and data cache, it retrieves the data from the storage layer

```sql
-- Context
USE ROLE ACCOUNTADMIN;
USE SCHEMA SNOWFLAKE_SAMPLE_DATA.TPCH_SF1000;

SHOW WAREHOUSES;
-- Stop the current warehouse
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
ALTER WAREHOUSE SET AUTO_RESUME=FALSE;

-- Meta data cache --
-- Proof that it uses the metadata cache can be seen if you go to the query profile
SELECT COUNT(*) FROM CUSTOMER; -- returns the count
SELECT CURRENT_USER(); -- context function
DESCRIBE TABLE CUSTOMER; -- shows the column details
SHOW TABLES; -- lists objects
SELECT SYSTEM$CLUSTERING_INFORMATION('LINEITEM', ('L_ORDERKEY')); -- system function

-- Results cache --
SELECT * FROM CUSTOMER LIMIT 1000; -- fails because warehouse is suspended
ALTER WAREHOUSE COMPUTE_WH RESUME;
SELECT * FROM CUSTOMER LIMIT 1000; -- query uses warehouse
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
SELECT * FROM CUSTOMER LIMIT 1000; -- This time it works, if you go to query profile, you will see QUERY RESULT REUSE
ALTER WAREHOUSE COMPUTE_WH RESUME;
SELECT * FROM CUSTOMER LIMIT 1000; -- even though warehouse is resumed, query uses the results cache
SELECT * FROM CUSTOMER LIMIT 100; -- limit was changed, we don't use the results cache

-- Local storage cache --

-- prevent use of results cache.
-- Alternatively, we can change the LIMIT parameter in the following queries
-- to make sure results cache is not used
ALTER ACCOUNT SET USE_CACHED_RESULT = FALSE;

-- Here in the query profile overview, Remote Disk I/O took the most time
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;

-- Query is a lot faster and in the query profile overview, there is no Remote Disk I/O
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;

-- We add other columns: We still use Remote Disk I/O for these columns but we use local storage for O_ORDERKEY
SELECT O_ORDERKEY, O_SHIPPRIORITY, O_COMMENT
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;


ALTER WAREHOUSE COMPUTE_WH SUSPEND;
ALTER WAREHOUSE COMPUTE_WH RESUME;
-- Cache was cleared when restarting the warehouse so query takes a lot of time
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;
```
