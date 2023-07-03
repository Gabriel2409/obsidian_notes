#snowflake 
## Overview

- A virtual warehouse is a named abstraction for a Massively Parallel Processing (MPP) compute cluster (see [[Snowflake - Architecture]]).
- Virtual warehouses constitute the compute layer
- Virtual warehouses execute:
  - DQL operations (SELECT)
  - DML operations (UPDATE)
  - Data loading operations (COPY INTO)
- As a user, we only interact with the named warehouse, not the individual nodes
- We can spin up an unlimited nb of warehouses and config can be changed on the fly
- Virtual warehouses contain local SSD storage to store raw data retrieved from the storage layer (warehouse cache, which is cleared on suspend)

```sql
-- creation
CREATE WAREHOUSE MY_WH 
WAREHOUSE_SIZE='MEDIUM';
-- deletion
DROP WAREHOUSE MY_WH;
-- update params
ALTER WAREHOUSE MY_WH 
WAREHOUSE_SIZE='SMALL';

-- get information on warehouses, such as nb of running and queued requests
SHOW WAREHOUSES;

```

## States

- States:
  - STARTED, it consumes credits
  - SUSPENDED, it does not consume credit
  - RESIZING
- By default, on creation it is in started state

```sql
-- Suspend
ALTER WAREHOUSE MY_WH SUSPEND; -- cache is cleared
-- RESUME
ALTER WAREHOUSE MY_WH RESUME;

-- Auto suspend after 5 mins (default is 10 min)
-- Note that suspend process runs one per minute and min billing is 1 min so it makes
-- no sense to use an AUTO_SUSPEND < 60
CREATE WAREHOUSE MY_WH AUTO_SUSPEND=300;

-- Auto resume = default behavior, WH restarts on query
CREATE WAREHOUSE MY_WH AUTO_RESUME=TRUE;

-- Initially suspended (default is in started state)
CREATE WAREHOUSE MY_WH INITIALLY_SUSPENDED=TRUE;
```

## Size

- To choose a size, snowflake recommends experimenting with a representative query of a workload
  - Data loading does not typically require large virtual warehouses and sizing up does not guarantee increased data loading performance.
  - Also see [[Snowflake - Billing]]

## Resource monitors

- Can only be created by account administrators by default
- Objects allowing users to set credit limits on user managed warehouses
- Can either be set at the account level or individual warehouse level
- Limits can be set for a specified interval or date range
- When limits are reached, an action such as notify user or suspend warehouse can be triggered

```sql
-- Creation
CREATE RESOURCE MONITOR MYRM
WITH CREDIT_QUOTA=100 -- credits allocated per frequency interval
FREQUENCY=MONTHLY
START_TIMESTAMP='2023-01-01 00:00 GMT' -- if not specified, starts immediately
TRIGGERS ON 50 PERCENT DO NOTIFY -- alert notification to account admins
		 ON 75 PERCENT DO NOTIFY
		 ON 95 PERCENT DO SUSPEND -- suspend after running queries are complete
		 ON 100 PERCENT DO SUSPEND_IMMEDIATE; -- hard stop

-- Apply at the account level
ALTER ACCOUNT SET RESOURCE_MONITOR=MYRM;
-- Apply at the warehouse level
ALTER WAREHOUSE MYWH SET RESOURCE_MONITOR=MYRM;
```

- To view consumption, you can also use SNOWFLAKE database:

```sql
-- context
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;

-- all information on billing
SELECT * FROM WAREHOUSE_METERING_HISTORY
WHERE WAREHOUSE_NAME = 'COMPUTE_WH';

-- Total credits grouped by warehouse
SELECT WAREHOUSE_NAME, SUM(CREDITS_USED) AS TOTAL_CREDITS_USED
FROM WAREHOUSE_METERING_HISTORY
GROUP BY WAREHOUSE_NAME;

-- Using the information schema, get last seven days with a table function
SELECT *
FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(dateadd('days', -7, current_date())));
```

## Multi cluster warehouses

- scale up: increase warehouse size, can only be done manually
- scale out: add clusters: A multi cluster warehouse is a named group of virtual warehouses which can automatically scale in and out based on the nb of concurrent users/queries

```sql
-- dynamically scale
CREATE WAREHOUSE MY_WH
MIN_CLUSTER_COUNT = 1 -- lower bound
MAX_CLUSTER_COUNT = 3 -- upper bound
SCALING_POLICY=STANDARD;
```

If MIN_CLUSTER_COUNT == MAX_CLUSTER_COUNT > 1, we are in MAXIMIZED mode. If nb are different, we are in AUTOSCALED mode

Scaling policy:

- STANDARD - maximize perf:
  - Scale out: When a query is queued, a new warehouse is added automatically
  - Scale in: Every min, a background process will check if the load on the least busy warehouse can be redistributed to another warehouse. If condition is met after 2 consecutive checks, the warehouse is marked for shutdown
- ECONOMY:

  - Scale out: only add a warehouse if there's enough query load to keep a new warehouse busy for 6 min
  - Scale in: same as STANDARD, but we only mark for shutdown after 6 minutes

- Billing: each cluster consumes credit depending on the size of the warehouse

```sql
CREATE WAREHOUSE MYWH
-- nb of concurrent SQL statements that can be executed against a warehouse
-- before queuing / providing additional compute power
MAX_CONCURRENCY_LEVEL = 6
-- time a SQL statement can be queued before it is aborted (default is no timeout)
STATEMENT_QUEUED_TIMEOUT_IN_SECONDS=60;
-- time after which any running SQL statement is aborted
STATEMENT_TIMEOUT_IN_SECONDS=600;
```
