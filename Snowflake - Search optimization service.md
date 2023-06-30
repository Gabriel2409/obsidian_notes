---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 230
---

#snowflake


## Search optimization service

- Snowflake is best suited for analytics workload
- However, there are scenarios where a user might need to look up an individual value in a very large table
- The Search Optimization Service is a table level property aimed at improving the performance of selective point lookup queries, typically returning 1 or a small nb of rows: `WHERE XX = 'YY'` or `WHERE XX IN ('YY', 'ZZ')`
- A background process creates and maintains a search access path recording metadata on the table
- Selective lookup queries will use these metadata to find data faster than the usual pruning mechanism
- Uses storage and compute resources

```sql

-- implementation is easy as it is a table level property
ALTER TABLE MYTABLE ADD SEARCH OPTIMIZATION;

-- remove it
ALTER TABLE MYTABLE DROP SEARCH OPTIMIZATION;

-- check search optimization column
SHOW TABLES;
SELECT "search_optimization", -- wheter service is enabled
"search_optimization_progress", -- percentage of table optimized so far
"search_optimization_bytes"
FROM TABLE(result_scan(last_query_id()));
```
