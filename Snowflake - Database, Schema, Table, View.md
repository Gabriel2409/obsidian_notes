---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 226
---

#snowflake

see [[Snowflake - Objects]]

### Database and schemas

- Must have a unique identifier in an account

```sql
-- Basic creation
CREATE DATABASE MYDATABASE;
-- Clone an existing db
CREATE DATABASE MYCLONE CLONE MYOTHERDB;

-- create a replica of a db in another account
CREATE DATABASE MYDB1
	AS REPLICA OF MYORG.ACCOUNT1.MYDB1
	DATA_RETENTION_TIME_IN_DAYS = 10;

-- create from a share object
CREATE DATABASE SHARED_DB FROM SHARE UTT783.SHARE
```

- More details on cloning: [[Snowflake - Cloning]]
- More details on replication: [[Snowflake - Replication]]
- More details on shares: [[Snowflake - Secure data sharing]]

Schemas allow for further segmentation

- Schemas must have a unique identifier in a database

```sql
-- switch database
USE DATABASE MYDATABASE;
-- Basic creation
CREATE SCHEMA MYSCHEMA;
-- Clone an existing schema
CREATE SCHEMA MYCLONE CLONE MYOTHERSCHEMA;
```

A **namespace** consists of a database and a schema together: `MYDB.MYSCHEMA`

### Table

- Permanent table:
  - Default table type
  - exists until explicitely dropped
  - **[[Snowflake - Time Travel and Fail Safe|Time travel retention period]]** = how long can we go into the past to restore data (up to 90 days for an enterprise account vs 1 for standard)
  - 7 days **[[Snowflake - Time Travel and Fail Safe|fail-safe]]**: snowflake can restore deleted data for us during this period
- Temporary table:
  - Used for transitory data
  - Persists for duration of a session
  - Can not be converted
  - Max time travel of 1 day and no fail safe
- Transient table
  - Similar to permanent: exist until explicitely dropped
  - However no fail safe period and max time travel of 1 day
  - Can decrease storage cost compared to permanent
- External table:
  - Query data outside of Snowflake
  - read-only table
  - slower
  - no time travel or failsafe

Type can be seen with `SHOW TABLES;` and checking the kind.
Table data is stored on the storage layer across many [[Snowflake - Micro-partitions|micro-partitions]]

### View

- Standard view:
  - Stores the query definition
  - can be used as a table (we can even use `ORDER BY`)
  - does not contribute to storage cost as they don't store any data
  - If source table is dropped, view will produce an error

```sql
CREATE VIEW MYVIEW AS
SELECT COL1, COL2 FROM MY_TABLE;
```

- Materialized view:
  - Stores the query definition
  - ALSO stores the result of the query definition and periodically refreshes it
  - Incurs cost as a serverless feature
  - Also incurs storage cost and compute cost (automatic background maintenance)
  - can be used to boost performance of external tables
  - Check `MATERIALIZED_VIEW_REFRESH_HISTORY` in the `ACCOUNT_USAGE` schema to track consumption
  - Limitations:
    - Limited to a single table
    - Can not use join
    - Can not use UDF, HAVING, ORDER BY, LIMIT, WINDOWS FUNCTIONS

```sql
CREATE MATERIALIZED VIEW MYVIEW AS
SELECT COL1, COL2 FROM MY_TABLE;

-- suspend a materialized view to pause the update
-- (also makes it inaccessible)
ALTER MATERIALIZED VIEW MYVIEW SUSPEND;

-- restarts the background process
ALTER MATERIALIZED VIEW MYVIEW RESUME;
```

- Both standard and materialized views can be secure by adding the `SECURE` keyword: `CREATE SECURE VIEW...`
  - Query definition is only visible to authorized users
  - Some query optimizations (for ex doing filter first) will be bypassed to improve security (can make view slower)

Check views with `SHOW VIEWS;` and `SHOW MATERIALIZED VIEWS`


While a regular view is basically a saved SQL query that can be executed by users who have access to underlying data, a secure view does not require these permissions. Indeed the secure view's query is executed in the context of the owner account ans only the result is returned (underlying tables and metadata are hidden). This is why you can [[Snowflake - Secure data sharing|share]] secure views but not regular views

### Worksheet Example query - Database, schema, table, views

- Navigate to worksheet and create a new worksheet
- set context with sql or select it from dropdown from within the script.

```sql
-- Set context
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SNOWFLAKE_SAMPLE_DATA;
USE SCHEMA TPCH_SF1

-- Get information on a table
DESCRIBE TABLE CUSTOMER;

-- show all tables in current context
SHOW TABLES;

-- Show all tables starting with PA
SHOW TABLES LIKE 'PA%';

-- show name column from before last query
SELECT "name" FROM
TABLE(result_scan(last_query_id(-2)));

-- Create new db
CREATE DATABASE DEMO_DB;
CREATE SCHEMA DEMO_SCHEMA;
-- switch context
USE DATABASE DEMO_DB;
USE SCHEMA DEMO_SCHEMA;
-- create tables
CREATE TABLE PERMANENT_TABLE(
    NAME STRING,
    AGE INT
);
CREATE TEMPORARY TABLE TEMPORARY_TABLE(
    NAME STRING,
    AGE INT
); -- will disappear after logging out and relogging in.
CREATE TRANSIENT TABLE TRANSIENT_TABLE(
    NAME STRING,
    AGE INT
);
SHOW TABLES; -- check kind (should be TABLE, TEMPORARY and TRANSIENT)

-- successful, see list of parameters here:
-- https://docs.snowflake.com/en/sql-reference/parameters
ALTER TABLE PERMANENT_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 90;

-- failure, can only be 1 or 0;
ALTER TABLE TRANSIENT_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 2;


-- create views
CREATE VIEW STANDARD_VIEW AS
SELECT * FROM PERMANENT_TABLE;

CREATE SECURE VIEW SECURE_VIEW AS
SELECT * FROM PERMANENT_TABLE;

CREATE MATERIALIZED VIEW MATERIALIZED_VIEW AS
SELECT * FROM PERMANENT_TABLE;

 -- check is_secure and is_materialized fields.
 -- text contains the view definition
SHOW VIEWS;
SELECT "name", "database_name", "schema_name", "is_secure", "is_materialized", "text"
FROM TABLE(result_scan(last_query_id()));

-- Secure view functionality
GRANT USAGE ON DATABASE DEMO_DB TO ROLE SYSADMIN;
GRANT USAGE ON SCHEMA DEMO_SCHEMA TO ROLE SYSADMIN;
GRANT SELECT, REFERENCES ON TABLE STANDARD_VIEW TO ROLE SYSADMIN;
GRANT SELECT, REFERENCES ON TABLE SECURE_VIEW TO ROLE SYSADMIN;
USE ROLE SYSADMIN;
SELECT get_ddl('view', 'STANDARD_VIEW'); -- definition shows correctly
SELECT get_ddl('view', 'SECURE_VIEW'); -- error
```
