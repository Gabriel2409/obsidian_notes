#snowflake

https://docs.snowflake.com/

# Features and architecture


- [[Snowflake - Overview]]
- [[Snowflake - Architecture]]
- [[Snowflake - Objects]]
- [[Snowflake - Billing]]
- [[Snowflake - Connectivity]]


# Account Access and security

- [[Snowflake - Access Control]]
- [[Snowflake - Authentication and Authorization]]
- [[Snowflake - Network policies]]
- [[Snowflake - Data encryption]]
- [[Snowflake - Column and row level security]]
- [[Snowflake - Account Usage and Information Schema]]



# Virtual Warehouse

- [[Snowflake - Virtual Warehouse]]

# Query optimization

- [[Snowflake - Query performance tool and sql tuning]]
- [[Snowflake - Caching]]
- [[Snowflake - Clustering]]
- [[Snowflake - Search optimization service]]

# Data loading and unloading

## Data Loading and unloading

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

-- removes all rows and then inserts statement
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

### Stages

- Area to temporarily hold RAW data file before being loaded into a table or after being unloaded from a table
- Comes into two groups: **internal** and **external**
  - Internal stages are stored directly within snowflake using the underlying blob storage of the cloud platform
  - External stages are external areas managed outside of snowflake (for ex if you have your raw data on another storage) and you must give snowflake permission to read from it
- Internal stages are further divided
  - **User stage**:
    - automatically allocated when a user is created and only accessible to the user
    - cannot be altered or dropped
    - Appropriate if only one user needs to access the stage
    - to get files into the stage from a local machine, use the `PUT` command
    - to reference it from sql, `ls @~;`
  - **Table stage**:
    - area for multiple users to stage data to be loaded but only for a given table
    - automatically allocated when a table is created and accessible to users having ownership privileges on the table
    - `PUT`
    - `ls @%MYTABLE`
  - **Named stage**:
    - not automatically allocated by snowflake but created directly by a user (more flexible): must be created with `CREATE STAGE MYSTAGE`
    - Securable object: privileges can be granted to roles to manage it
    - `PUT`
    - `ls @MYSTAGE`
- Internal stages always compress files using **gzip** unless specified otherwise
- Internal stage files are encrypted using AES 128 bit keys
- External stages are always named stages
  - Storage location can be public or private
  - can be used with storage integrations that are reusable and securable objects which can be applied accross stages to avoid having to explicitely set sensitive information at each stage
  - `ls @MYSTAGE`

```sql
-- Create external stage
CREATE STAGE MYEXTSTAGE
URL='S3://MYBUCKET/PATH/'
CREDENTIALS=(AWS_KEY_ID='' AWS_SECRET_KEY='')
ENCRYPTION=(MASTER_KEY='')

-- Better, create a storage integration
CREATE STAGE MYEXTSTAGE
URL='S3://MYBUCKET/PATH/'
STORAGE_INTEGRATION=MYINT;

CREATE STORAGE INTEGRATION MYINT
TYPE=EXTERNAL_STAGE
STORAGE_PROVIDER=S3
STORAGE_AWS_ROLE_ARN='ARN:AWS:IAM:98765:ROLE/MYROLE'
ENABLED=TRUE
STORAGE_ALLOWED_LOCATIONS=('S3://MYBUCKET/PATH/')
```

- Useful commands

```sql
-- list contents of a stage, can use ls or list
-- contains path, size, MD5 hash, last updated timestamp of staged file
ls @MYSTAGE;
ls @~;
ls @%MYTABLE;
ls @~/films.csv; -- includes folder


-- Query content directly (useful for inspection)
-- You can specify the file format to correctly parse the data file
CREATE FILE FORMAT CSV_FILE_FORMAT
	TYPE = CSV
	SKIP_HEADER = 1;


SELECT
metadata$filename,
metadata$file_row_number,
$1, -- column number of underlying file, relevant for csv files
$2
FROM @MYSTAGE/MYFOLDER(FILE_FORMAT => 'CSV_FILE_FORMAT', pattern = '.*csv');

-- Remove files from stage using rm or remove
rm @MYSTAGE;
rm @~;
rm @%MYTABLE;


-- upload files from local directory of client machine to stage
PUT FILE:////FOLDER/MYDATA.CSV @MYSTAGE; -- or @~ or @%MYTABLE

-- for windows
PUT FILE://c:\FOLDER\MYDATA.CSV @MYSTAGE;
PUT FILE://c:\FOLDER\MYDATA.CSV @~ auto_compress=false;
```

### Bulk loading with COPY INTO

- copies the contents of a stage directly into a table
- Requires a virtual warehouse to execute
- Load history is stored in the metadata of the target table for 64 days to ensure files are not loaded twice
- Allows to perform simple transformations during loading process
- External stages work the same as internal for COPY INTO but additional billing charges may apply in case of data transfer

```sql
-- Copy  contents of a stage into a table
COPY INTO MYTABLE FROM @MYSTAGE;
COPY INTO MYTABLE FROM @MYSTAGE/MYFOLDER; -- only a folder
COPY INTO MYTABLE FROM @MYSTAGE/MYFOLDER/MYFILE.CSV; -- only a file

-- also possible to specify the files directly
COPY INTO MYTABLE FROM @MYSTAGE
FILE=('folder1/file1.csv', 'folder2/file2.csv')

-- or a regex pattern
COPY INTO MYTABLE FROM @MYSTAGE
PATTERN=('folder/.*[.]csv')

-- Transformations
COPY INTO MYTABLE FROM (
	SELECT TO_DOUBLE(T.$1), T.$2
	FROM @MYSTAGE T
);


-- copy directly from outside (note that it is better to use an external stage)
COPY INTO MYTABLE FROM S3://MYBUCKET/
STORAGE INTEGRATION=MY_INTEGRATION
ENCRYPTION=(MASTER_KEY='');

-- Performs a dry run of load process to expose errors with VALIDATION_MODE
COPY INTO MYTABLE FROM @MYSTAGE
VALIDATION_MODE = 'RETURN_ERRORS'; -- or RETURN_N_ROWS|RETURN_ALL_ERRORS

-- View all errors encountered in a previous COPY INTO execution
-- accest the job id of a previous query or the last load operation executed
SELECT * FROM TABLE(VALIDATE(MYTABLE,JOB_ID => '<job_id>'));
```
- `COPY INTO` comes with a list of options such as
  - `ON_ERROR`: specifies error handling for load operation
  - `SIZE_LIMIT`
  - `PURGE`: whether to remove data from the stage after a successful load
  - ...

### File formats

- supported types are CSV, JSON, AVRO, ORC, PARQUET, XML
- Each type has its own set of properties related to parsing that specific format
- file format can be set on a stage directly or in the COPY INTO statement
  - If set on both, value in COPY INTO takes precedence

```sql
-- Set file format directly in an internal stage
CREATE STAGE MYSTAGE
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);

-- better way: create a file format
CREATE FILE_FORMAT MYCSVFF
TYPE='CSV'
SKIP_HEADER=1;
CREATE OR REPLACE STAGE MYSTAGE
FILE_FORMAT = MYCSVFF


```
