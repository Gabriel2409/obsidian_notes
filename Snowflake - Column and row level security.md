---
sr-due: 2023-06-29
sr-interval: 1
sr-ease: 230
---

#snowflake

## Column and row level security

### Column level security

- Dynamic data masking is applied dynamically at query run time
  - we apply the masking policy to the column of a table or view and it will mask the data for unauthorized users (masking can be partial)

```sql
-- create the policy
CREATE MASKING POLICY EMAIL_MASK AS (VAL STRING) RETURNS STRING ->
    CASE
        WHEN CURRENT_ROLE() IN ('SUPPORT') THEN VAL -- unmasked
        WHEN CURRENT_ROLE() IN ('ANALYST') THEN REGEXP_REPLACE(VAL, '.+\@', '****@') -- partial mask
        WHEN CURRENT_ROLE() IN ('HR') THEN SHA2(VAL) -- system functions
        WHEN CURRENT_ROLE() IN ('FINANCE') THEN MY_MASK(VAL) -- UDF
        ELSE '******'
    END;

-- apply to the column
ALTER TABLE MYTABLE MODIFY COLUMN
USER_EMAIL SET MASKING POLICY EMAIL_MASK
```

- Data masking policies are schema-level objects like tables and views
- Creating and applying them can be done independently of object owners
- Masking policies can be nested
- A masking policy is applied no matter where the column is referenced in a SQL statement. **If you try to do a join on masked data, it will be performed on the masked data** not the underlying data
- It can be added at object creation or after object is created

Masking policies can also use external tokenization. Unlike other form of data masking, data is directly stored tokenized in snowflake and will be detokenized at run time for authorized users by calling the external tokenization service.

**Note: You can also use tags that you artach to columns and attach the security to the tag**

### Row level security

- We can restrict which rows are returned in a query
- Row level security is applied transparently: if an unauthorized user executes a SELECT statement, restricted rows will be filtered

```sql
-- create the policy
-- here RAP_ID is the identifier
-- ACC_ID is the column we would like to attach the policy (serves as a sort of anchor)
-- output data type is always boolean
CREATE ROW ACCESS POLICY RAP_ID AS (ACC_ID VARCHAR) RETURNS BOOLEAN ->
    CASE
        WHEN CURRENT_ROLE() IN ('ADMIN') THEN TRUE -- return all rows
        ELSE FALSE -- return no rows
    END;

-- apply policy to the column
ALTER TABLE MYTABLE ADD ROW ACCESS POLICY RAP_ID ON (ACC_ID);
```

- Most of the remarks on column data masking apply
- NOTE: Adding a masking policy to a column fails if the column is referenced by a row access policy
- Row access policies are evaluated before data masking policies

**Possibility to use mapping table as well to store the roles**

### Worksheet example

```sql
-- initial context
USE ROLE SYSADMIN;
USE WAREHOUSE COMPUTE_WH;
-- Create new db
CREATE DATABASE SALES_DB;
CREATE SCHEMA SALES_SCHEMA;
-- switch context
USE DATABASE SALES_DB;
USE SCHEMA SALES_SCHEMA;

-- Create table with values
CREATE TABLE CUSTOMERS(
    ID NUMBER,
    NAME STRING,
    EMAIL STRING,
    COUNTRY_CODE STRING
);
INSERT INTO CUSTOMERS VALUES
(1357, 'Gab Prk', 'gabprk@gmail.com', 'FR'),
(8978, 'John Doe', 'jdoe@stanford.eu', 'US'),
(7997, 'Un Known', 'unkown@outlook.co.uk', 'IE');

-- grant usage to ANALYST to query the data
USE ROLE SECURITYADMIN;
GRANT USAGE ON DATABASE SALES_DB
TO ROLE ANALYST;
GRANT USAGE ON SCHEMA SALES_DB.SALES_SCHEMA
TO ROLE ANALYST;
GRANT SELECT ON TABLE SALES_DB.SALES_SCHEMA.CUSTOMERS
TO ROLE ANALYST;

-- Create MASKING_ADMIN role
CREATE ROLE MASKING_ADMIN;
GRANT USAGE ON DATABASE SALES_DB
TO ROLE MASKING_ADMIN;
GRANT USAGE ON SCHEMA SALES_DB.SALES_SCHEMA
TO ROLE MASKING_ADMIN;
GRANT CREATE MASKING POLICY, CREATE ROW ACCESS POLICY
ON SCHEMA SALES_DB.SALES_SCHEMA
TO ROLE MASKING_ADMIN;

USE ROLE ACCOUNTADMIN;
GRANT APPLY MASKING POLICY, APPLY ROW ACCESS POLICY
ON ACCOUNT TO ROLE MASKING_ADMIN; -- we need the account admin role for this one
GRANT ROLE MASKING_ADMIN TO USER ADMIN;

-- Create masking policy
USE ROLE MASKING_ADMIN;
USE SCHEMA SALES_DB.SALES_SCHEMA;

CREATE OR REPLACE MASKING POLICY EMAIL_MASK AS (VAL STRING) RETURNS STRING ->
    CASE
        WHEN CURRENT_ROLE() IN ('ANALYST') THEN VAL
        ELSE REGEXP_REPLACE(VAL, '.+\@', '****@')
    END;
-- apply masking policy
ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL SET MASKING POLICY EMAIL_MASK;

-- Check policy
USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- we see the data

USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- data is masked even though ANALYST role is GRANTED to SYSADMIN

-- Create simple row access policy
USE ROLE MASKING_ADMIN;
USE SCHEMA SALES_DB.SALES_SCHEMA;

CREATE OR REPLACE ROW ACCESS POLICY RAP AS (VAL VARCHAR) RETURNS BOOLEAN ->
    CASE
        WHEN 'ANALYST' = CURRENT_ROLE() THEN TRUE
        ELSE FALSE
    END;

-- following statement fails because column EMAIL is already masked
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY RAP ON (EMAIL);
-- ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL UNSET MASKING POLICY;

ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY RAP ON (NAME);

-- Verify policy
USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- we see the data

USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- we don't see any rows

-- Create row access policy with lookup
CREATE TABLE TITLE_COUNTRY_MAPPING (TITLE STRING, country_iso_code STRING);
INSERT INTO TITLE_COUNTRY_MAPPING VALUES ('ANALYST', 'FR');
GRANT SELECT ON TABLE TITLE_COUNTRY_MAPPING TO ROLE MASKING_ADMIN;

USE ROLE MASKING_ADMIN;
CREATE OR REPLACE ROW ACCESS POLICY CUSTOMER_POLICY AS (COUNTRY_CODE VARCHAR) RETURNS BOOLEAN ->
    'SYSADMIN' = CURRENT_ROLE() -- Sysadmin can view all roles
        OR EXISTS(
            SELECT 1 FROM TITLE_COUNTRY_MAPPING -- mapping table check
            WHERE TITLE = CURRENT_ROLE()
            AND country_iso_code = COUNTRY_CODE
        );

-- Fails because there is already a row access policy on the table
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY CUSTOMER_POLICY ON (COUNTRY_CODE);

ALTER TABLE CUSTOMERS DROP ALL ROW ACCESS POLICIES;
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY CUSTOMER_POLICY ON (COUNTRY_CODE);

-- Verify policies
USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- all rows, we still have data masking on EMAIL

USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- only one row but no masking
```
