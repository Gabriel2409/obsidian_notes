---
reviewed: 2023-07-05
---

#snowflake

## Steps in the data lifecycle

Current Data storage > Time travel retention > fail safe

- Current data storage = state of the current micro-partitions
- Time travel = When data was updated/deleted, they stay in storage for a configurable amount of time
- Fail safe = after the time travel period is over, data can still be recovered by snowflake during the fail safe
- Time travel and fail safe both contribute to storage costs

## Time travel

- Restore objects such as tables, schemas, databases that have been deleted
- Analyse historical data by querying it at points in the past
- can be combined with [[Snowflake - Cloning|cloning]] to create clones of objects from a point in the past
- Time travel retention period is configurable at account level, db level, schema level and table level (with child objects taking precedence)
  - Default retention period is 1 day
  - In enterprise edition, can be extended to 90 days (but not for transient and temporary tables, see [[Snowflake - Database, Schema, Table, View#Table|Tables]])
- Snowflake allows access at a given time
  - with the `AT` keyword: parameters can be either `TIMESTAMP`, `OFFSET` or `STATEMENT`
  - `BEFORE` allows to select historical data up to but not including changes from a specific transaction: only `STATEMENT` parameter is available
  - `UNDROP` restores the most recent version of a dropped table, schema, db. It fails if an object with the same name already exists

```sql

-- Setting retention_time (child objects take precedence)
SHOW DATABASES LIKE 'DEMO_DB'; -- retention_time is 1
ALTER ACCOUNT SET DATA_RETENTION_TIME_IN_DAYS = 90;
SHOW DATABASES LIKE 'DEMO_DB'; -- retention_time is 90
ALTER DATABASE DEMO_DB SET DATA_RETENTION_TIME_IN_DAYS = 45;
SHOW DATABASES LIKE 'DEMO_DB'; -- retention_time is 45


-- check dropped_on field for DEMO_TABLE_TT, should be null
SHOW TABLES HISTORY;
DROP TABLE DEMO_TABLE_TT;
-- dropped_on field now has the timestamp of when table was dropped
SHOW TABLES HISTORY;
UNDROP TABLE DEMO_TABLE_TT;
-- dropped_on field is null again
SHOW TABLES HISTORY;


-- Analyse older data
TRUNCATE TABLE DEMO_TABLE_TT; -- returns a statement id
-- table as it was 5 min ago
SELECT * FROM DEMO_TABLE_TT
AT(OFFSET => -60 * 5)
-- table as it was at given timestamp
SELECT * FROM MYTABLE
AT(TIMESTAMP=> DATEADD(minute, -15, current_timestamp()));
-- table as it was at given statement
SELECT * FROM MYTABLE
AT(STATEMENT => '015476g-45gf-5fgf');
-- table as it was BEFORE given statement
SELECT * FROM MYTABLE
BEFORE(STATEMENT => '015476g-45gf-5fgf');

-- Cloning
CREATE TABLE MYTABLE2 CLONE MYTABLE
AT(TIMESTAMP=> TO_TIMESTAMP('2023-01-01'));

```

## Fail safe

- Non configurable period of 7 days in which historical data can be recovered by contacting snowflake support. Reserved as a last resort recovery technique
- Not all objects support fail safe (for ex transient and temporary tables don't)
