---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 230
---

#snowflake

see [[Snowflake - Data loading]]

### Insert statement

- basic way to load data

```sql
-- context
USE ROLE SYSADMIN;
CREATE DATABASE FILMS_DB;
USE DATABASE FILMS_DB;
CREATE SCHEMA FILMS_SCHEMA;
USE SCHEMA FILMS_SCHEMA;
CREATE TABLE FILMS(
    ID STRING,
    TITLE STRING,
    RELASE_DATE DATE
);

-- clone empty table
CREATE TABLE FILMS_2000 CLONE FILMS;

-- Insert one row via a select query without specifying columns
INSERT INTO FILMS SELECT 'fgf564', 'Parasite', DATE('2019-05-30');

-- Insert one row via a select query but ommitting a column
INSERT INTO FILMS (ID, TITLE) SELECT 'dm564df', '12 Angry Men';

-- Insert one row with VALUES syntax
INSERT INTO FILMS VALUES('iofd54', 'Back to the Future', DATE('1985-12-04'));

-- Insert multiple rows with VALUES syntax
INSERT INTO FILMS VALUES
('9x46ddf', 'Citizen Kane', DATE('1942-01-24')),
('2wsfgh544', 'Old Boy', DATE('2004-10-15')),
('0gf545fg', 'Ratatouille', DATE('2007-06-29'));

SELECT * FROM FILMS;

-- Insert  rows from another table - only works if tables are structurally identical
INSERT INTO FILMS_2000 SELECT * FROM FILMS WHERE RELASE_DATE > DATE('2000-01-01');

-- Truncages table (removes all rows) and then inserts statement
INSERT OVERWRITE INTO FILMS_2000 SELECT * FROM FILMS;

```

### Loading from the UI

- In the data tab, go to the database, then the schema, then the table.
- Click load data, select the data you want to load and the options
- For the UI, there is a limit of 50MB
- You can also see the generated sql, for ex:

```sql
COPY INTO "FILMS_DB"."FILMS_SCHEMA"."FILMS"
FROM '@"FILMS_DB"."FILMS_SCHEMA"."%FILMS"/__snowflake_temp_import_files__/films.csv'
FILE_FORMAT = (
    TYPE=CSV,
    SKIP_HEADER=1,
    FIELD_DELIMITER=',',
    TRIM_SPACE=FALSE,
    FIELD_OPTIONALLY_ENCLOSED_BY=NONE,
    DATE_FORMAT=AUTO,
    TIME_FORMAT=AUTO,
    TIMESTAMP_FORMAT=AUTO
)
ON_ERROR=ABORT_STATEMENT
PURGE=TRUE
```
