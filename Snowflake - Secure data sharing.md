---
reviewed: 2023-07-06
---

#snowflake

## Overview

- Allows an account to provide read-only access to selected database objects to other snowflake accounts **WITHOUT** transferring data
- Other accounts will only be billed for the compute resources used to make the queries, not the storage costs
- Sharing is enabled with an account-level SHARE object. It is created by a data provider and has 2 key configurations:
  - Grants on db objects
  - consumer account definitions
- An account can share Tables, external tables, secure views, secure materialized views, secure UDFs
- Data consumers create a db from a SHARE which contains the read-only db granted by the data provider
- Not available in VPS edition of snowflake

## Data provider

- Db objects added to a share become immediately available to all consumers
- Creating a share requires the `CREATE SHARE` privilege
- A share can only be granted to accounts in the same region and cloud provider as the data provider account. However, we can create a [[Snowflake - Replication|replica]] first and share the replica.

## Data consumer

- Only one db can be created per share
- Cannot create objects inside shared database
- Cannot reshare shared databases
- Can not use time travel on shared objects
- Cannot create a clone of a shared database or db objects
- To create a db from a share, you need the `IMPORT SHARE` privilege

## Reader account

- A provider can request snowflake to create a Reader account to allow non Snowflake customers to gain access to providers data
- Very limited in what they can do (query and view db objects created from a share)
- Resources are billed to the provider account
- Soft limit of 20 reader accounts per provider (can be increased by request)

## Example

In the main account

```sql
-- set context
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE DEMO_DB;
USE SCHEMA DEMO_SCHEMA;

CREATE SECURE VIEW DEMO_SEC_VIEW
AS
SELECT FIRST_NAME FROM DEMO_TABLE
WHERE AGE > 30;



-- Create the share and add grants
CREATE SHARE DEMO_SHARE COMMENT='Sharing demo data';
GRANT USAGE ON DATABASE DEMO_DB TO SHARE DEMO_SHARE;
GRANT USAGE ON SCHEMA DEMO_DB.DEMO_SCHEMA TO SHARE DEMO_SHARE;
GRANT SELECT ON TABLE DEMO_DB.DEMO_SCHEMA.DEMO_TABLE TO SHARE DEMO_SHARE;

-- Create a reader account
CREATE MANAGED ACCOUNT DEMO_READER_ACCOUNT
admin_name='reader'
admin_password='Password1234'
type = reader;

-- see the reader account
-- you can also go to Data > Private sharing in snowsite
SHOW MANAGED ACCOUNTS;
SELECT
"url" -- url to access the reader account with the name and password
,"locator" -- reader account locator
FROM TABLE(result_scan(last_query_id()));

-- Add reader account to share
ALTER SHARE DEMO_SHARE
ADD ACCOUNTS = <locator>; -- reader account locator
-------------------------------------------------------------------------
-------------------------------------------------------------------------
```

Go to the url and enter the admin name and password. You now have access to a snowflake reader account. Within the reader account

```sql
USE ROLE ACCOUNTADMIN;
-- Creates db from the share
-- here <account_locator> corresponds
-- to the locator of the account that
-- created the share, not the reader account
CREATE DATABASE DEMO_READER
FROM SHARE <account_locator>.DEMO_SHARE


USE ROLE SYSADMIN;

-- we need a warehouse for compute
CREATE WAREHOUSE COMPUTE_XS
WAREHOUSE_SIXE='XSMALL'
WHAREHOUSE_TYPE='STANDARD'
AUTO_SUSPEND=600
AUTO_RESUME=TRUE
SCALING_POLICY='STANDARD';

USE WAREHOUSE COMPUTE_XS;
USE DATABASE DEMO_READER;
USE SCHEMA DEMO_SCHEMA;

-- we can access the shared objects
SELECT * FROM DEMO_TABLE;

-- Note that following statement failed
SELECT * FROM DEMO_SEC_VIEW;
-------------------------------------------------------------------------
-------------------------------------------------------------------------
```

Now within the main account:

```sql
-- Add new objects to the share
GRANT SELECT ON VIEW DEMO_SEC_VIEW
TO SHARE DEMO_SHARE;
```

Then in the reader account

```sql
-- this time it works. Adding the
-- view to the share made it
-- immediately accessible
SELECT * FROM DEMO_SEC_VIEW;
```

Finally in the main account:

```sql
-- remove reader account access to share
ALTER SHARE DEMO_SHARE
REMOVE ACCOUNTS = <reader_locator>;
-- removes the share
DROP SHARE DEMO_SHARE;
-- drops the reader account
-- note that if you are connected to the reader account
-- you will be redirected to a 404
DROP MANAGED ACCOUNT DEMO_READER_ACCOUNT;
```
