---
sr-due: 2023-07-03
sr-interval: 1
sr-ease: 210
---

#snowflake

## Query performance analysis tool

- Activity > Query history tab displays query history for the last 14 days
- Users can view other users queries but not view their results
- Click on a query to have a breakdown of the actions that snowflake actually did. Each box is an operation and the line represents relationships between blocks. You can also see the nb of rows you get at each step and the % of the total time it took. Click on a box to have statistics and a profile overview
- You can also check the query plan by running `EXPLAIN <MYQUERY>;`

NOTE: on the statistics you can see spilling information (nb of bytes spilled to local storage and spilled to remote storage): if there is no problem with the query, it may indicate that the warehouse in memory and then disk storage is too small

```sql
--- Get 10 longest running queries
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;
SELECT
QUERY_ID
,QUERY_TEXT
,USER_NAME
,ROLE_NAME
,EXECUTION_STATUS
,ROUND(TOTAL_ELAPSED_TIME / 1000, 2) AS TOTAL_ELAPSED_TIME_SEC
FROM QUERY_HISTORY
WHERE TOTAL_ELAPSED_TIME_SEC > 3
ORDER BY TOTAL_ELAPSED_TIME_SEC
LIMIT 10;

-- Note instead of ACCOUNT_USAGE, we can look at the info in the
-- information schema
FROM TABLE(INFORMATION_SCHEMA.query_history())
```

## SQL Tuning

Order of execution:

- ROWS: FROM, JOIN, WHERE
- GROUPS: GROUP BY, HAVING
- RESULT: SELECT, DISTINCT, ORDER BY, LIMIT

- So, for ex, LIMIT is always executed after GROUP BY, which means that it won't accelerate our group by
- The reason we do row operations first is to decrease the nb of data for more expensive operations
- GROUP BY work best with low cardinality groups
