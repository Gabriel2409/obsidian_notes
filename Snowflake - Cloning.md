#snowflake

## Cloning Overview

- Process of creating a copy of an existing object within an account
- Users can clone databases, schemas, tables, streams, stages, file formats, sequences, tasks and pipes (reference external stage only), see [[Snowflake - Objects]]
- When cloning an object, it can allow to clone children that you can't directly clone. For ex, you can not clone a view but a cloned schemas will include the views
- Cloning is a metadata only operation (**Zero-copy cloning**), cloning the properties, structure and configuration of its source.
  - For that reason it does NOT contribute to storage costs until data is modified or new data is added to the clone
  - It is also very quick
- Changes made after the point of cloning start to create additional micro-partitions
- Changes made to the source or the clone are not reflected between each other, they are completely independent
- Clones can be cloned
- Cloning can be combined with [[Snowflake - Time Travel and Fail Safe|time travel]] to create a clone from a given moment in time
- Caveats
  - A cloned object does not retain the privileges of the source objects (except for tables)
  - External tables and internal named stages are never cloned
  - A cloned table does not contain the load history of the source table
  - Temporary and transient tables can only be cloned as temporary and transient tables (but permanent can be cloned as any)

```sql
-- Context
USE DATABASE DEMO_DB;
USE SCHEMA DEMO_SCHEMA;

CREATE TABLE DEMO_TABLE(
ID STRING,
FIRST_NAME STRING,
AGE NUMBER
);

INSERT INTO DEMO_TABLE VALUES
('55sfgdgf', 'Gab', 50),
('Mmdgd', 'Jo', 80),
('7Dgfdg', 'Jack', 20);

SELECT * FROM DEMO_TABLE; -- 3 records

-- cloning is metadata operation only:
-- no data is transferred (zero-copy cloning)
CREATE TABLE DEMO_TABLE_CLONE CLONE DEMO_TABLE;

-- same rows as DEMO_TABLE
SELECT * FROM DEMO_TABLE_CLONE;

-- clone a clone
CREATE TABLE DEMO_TABLE_CLONE2 CLONE DEMO_TABLE_CLONE;


-- recursively clone entire database
CREATE DATABASE DEMO_DB_CLONE CLONE DEMO_DB;
USE DATABASE DEMO_DB_CLONE;
USE SCHEMA DEMO_SCHEMA;
SHOW TABLES; -- all clones are showing

-- Data added to cloned db table will start to store micro-partitions
-- this incurs additional costs
INSERT INTO DEMO_TABLE VALUES ('fdsf', 'Paul', 78);

SELECT * FROM DEMO_TABLE; -- record added
SELECT * FROM DEMO_DB.DEMO_SCHEMA.DEMO_TABLE; -- source was not modified


-- Create clone with time travel
CREATE TABLE DEMO_TABLE_CLONE_TT CLONE DEMO_TABLE
AT(OFFSET => -60 * 2)

```
