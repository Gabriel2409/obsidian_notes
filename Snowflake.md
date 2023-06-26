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

### Bulk loading with COPY INTO table

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

### Snowpipe

- Snowpipe is a continuous ingestion service used to automatically load data files in real time as soon as they are available in a stage. For ex, you upload a file in an S3 external stage, snowflake will be listening to this event and spin up compute to copy the new data into a table.
- Snowpipe works in two modes:
  - `AUTO_INGEST=TRUE` = automatic snowpipe using cloud messaging (external stages only)
  - `AUTO_INGEST=FALSE` = calling snowpipe REST endpoints (works for both internal and external stages), Note that this method requires key pair authentication with JWT

```sql
CREATE PIPE MYPIPE
AUTO_INGEST=TRUE
AS
COPY INTO MYTABLE FROM @MYSTAGE
... -- extra parameters

```

- In contrast with bulk loading which requires a virtual warehouse, snowpipe is a Serverless feature
- In addition to resource consumption, snowpipe charges 0.06 credits per 1000 files
- Snowpipe is intended to load small files quickly and it takes about 1 minute for a file to be loaded once they receive a notification
- Snowpipe load history is stored in the metadata of the pipe for 14 days
- Pipes can be paused. Event messages received for the pipe enter a limited retention period (14 days by default) to allow resuming processing when it is restarted

- Cloud messaging example:
  - Upload file to S3 with cloud utils.
  - S3 bucket will then send a PutObject notification to an SQS queue
  - This queue is managed by snowflake and was created when the pipe object was created
  - By looking at the queue, the pipe will know that it must perform the COPY INTO command
- REST endpoint example
  - client application can call snowflake via an Insert Files REST endpoint

### Best practices for data loading

- Split files into multiple files of 100-250 MB of compressed data because a single server in a warehouse can process up to 8 files in parallel
- Organize data by path
- Snowflake recommends having separate warehouses for data loading tasks and other tasks
- Pre-sorting data creates a natural clustering of data so that data in micro partitions are more evenly distributed and improves pruning
- Not recommended to load more than 1 file per minute

### Data unloading

- Get data out of snowflake
- Table data dan be unloaded with `COPY INTO <location>`

```sql
COPY INTO @MYSTAGE
FROM MYTABLE;
```

- `COPY INTO <location>` supports delimited file formats such as CSV, JSON and Parquet
- By default results unloaded to a stage are split between multiple files (size depends on warehouse)
- By default these files are in CSV compressed with GZIP
- All files are automatically encrypted using AES 128 bit keys when unloaded to an internal stage

```sql
-- output files can be prefixed
COPY INTO @MYSTAGE/FOLDER/PREFIX_
-- unloading from a query allows more complex operations such as joins
FROM (SELECT * FROM T1)
FILE_FORMAT = MYFORMAT; -- changes the default format

-- partition unloaded data into a directory structure
COPY INTO @%T1
FROM T1
PARTITION BY ('DATE=' || TO_VARCHAR(DT))


-- copy directly to external cloud provider
-- (however it is better to create an external stage)
COPY INTO 'S3://MYBUCKET/UNLOAD'
FROM T1
STORAGE_INTEGRATION=MYINT;
```

- Useful options for `COPY INTO <location>`

  - `OVERWRITE`: behavior for when COPY command overwrites files with matching names
  - `SINGLE`: boolean specifying whether to create a single file
  - `MAX_FILE_SIZE` (default is about 50MB compressed)
  - `INCLUDE_QUERY_ID`

- Opposite to the `PUT` command, we can download a file from a stage with the `GET` command from a local machine
- Note that files are unencrypted when unloaded to a local directory
- GET can not be used for external stages

```sql
GET @MYSTAGE file:////folder//files/
PARALLEL = 99 -- optional parameter specifying the nb of threads to dl the files
PATTERN = '*\\.(csv)' -- optional parameter to target specific files
;
```

### Semi-structured data

- Historically very hard to accommodate in data warehouses that were built to store structured data
- Snowflake extended SQL to allow to store semi-structured data types and include semi-structured functions

#### Semi structured data types

- `ARRAY`: contains 0 or more elements of data. Each element is accessed by its position on the array:

```sql
CREATE TABLE MYTABLE(NAME VARCHAR, HOBBIES ARRAY);
INSERT INTO MYTABLE
SELECT 'Gab', ARRAY_CONSTRUCT('Travel', 'Sports');

-- access element with bracket notation
SELECT HOBBIES[0] FROM MYTABLE
```

- `OBJECT`: collections of key-value pairs

```sql
CREATE TABLE MYTABLE(NAME VARCHAR, ADDRESS OBJECT);
INSERT INTO MYTABLE
SELECT 'Gab', OBJECT_CONSTRUCT('postcode', '75000', 'road', 'Rivoli');

-- access element with . or bracket notation
SELECT ADDRESS['postcode'], ADDRESS.road FROM MYTABLE
```

- `VARIANT`: universal semi-structured data type used to represent arbitrary data structures that can hold up to 16MB of compressed data per row

```sql
CREATE TABLE MYTABLE(NAME VARIANT, ADDRESS VARIANT, HOBBIES VARIANT);
INSERT INTO MYTABLE
SELECT 'Gab'::VARIANT, -- we need to cast type for non object and non arrays
ARRAY_CONSTRUCT('Travel', 'Sports')
OBJECT_CONSTRUCT('postcode', '75000', 'road', 'Rivoli', 'nums':[1,2]);

-- access with : or bracket
SELECT
ADDRESS:nums[0],
-- if we do not cast value, it is shown with double quotes which can mess up joins
ADDRESS:road::string,
TO_STRING(ADDRESS:road)
FROM MYTABLE
```

#### Semi structured file formats

- JSON
- XML
- AVRO: binary row-based storage format originally developped for Apach HADOOP
- ORC: highly efficient binary format used to store Hive data
- PARQUET: binary format designed for projects in the HADOOP ecosystem

Note: among these, only JSON and PARQUET can be unloaded with `COPY INTO <location>`

#### Loading semi-structured data

- ELT style approach: good when operation to perform on the data is not known upfront

```sql
-- load full content into a variant column
CREATE TABLE MYTABLE(V VARIANT);
COPY INTO MYTABLE FROM @MYSTAGE/file1.json
FILE_FORMAT = FF_JSON;
```

- ETL: extract column values directly from the semi-structured data file

```sql
CREATE TABLE MYTABLE(NAME STRING, AGE NUMBER, DOB DATE);
COPY INTO MYTABLE
FROM (
	SELECT V:name, V:age, V:dob)
	FROM @MYSTAGE/file1.json)
FILE_FORMAT = FF_JSON;
```

- Automatic schema detection

```sql
CREATE TABLE MYTABLE
USING TEMPLATE(
	SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
	FROM TABLE(
		INFER_SCHEMA(
			LOCATION => '@MYSTAGE',
			FILE_FORMAT=> 'FF_PARQUET'
			)
	)
);
COPY INTO MYTABLE
FROM @MYSTAGE/file1.json
FILE_FORMAT = (TYPE='JSON')
MATCH_BY_COLUMN_NAME = CASE_SENSITIVE;

```

#### Example worksheet

```sql
-- context
USE ROLE SYSADMIN;
USE DATABASE FILMS_DB;
USE SCHEMA FILMS_SCHEMA;

-- file format creation
CREATE OR REPLACE FILE FORMAT JSON_FILE_FORMAT
TYPE='JSON',
FILE_EXTENSION=NULL,
DATE_FORMAT='AUTO',
TIME_FORMAT='AUTO',
TIMESTAMP_FORMAT='AUTO',
BINARY_FORMAT='HEX',
TRIM_SPACE=FALSE,
NULL_IF='',
COMPRESSION='AUTO',
ENABLE_OCTAL=FALSE,
ALLOW_DUPLICATE=FALSE,
STRIP_OUTER_ARRAY=TRUE, -- very important
STRIP_NULL_VALUES=FALSE,
IGNORE_UTF8_ERRORS=FALSE,
REPLACE_INVALID_CHARACTERS=FALSE,
SKIP_BYTE_ORDER_MARK=TRUE;

-- in snowsql: PUT file:////path/to/films.json @FILMS_STAGE auto_compress=false;

CREATE OR REPLACE TABLE FILMS_ELT (JSON_VARIANT VARIANT);

COPY INTO FILMS_ELT FROM @FILMS_STAGE/films.json
FILE_FORMAT=JSON_FILE_FORMAT
FORCE=TRUE -- allows reloading of the same file
;

-- Result will depend on STRIP_OUTER_ARRAY in the file format
-- If STRIP_OUTER_ARRAY=FALSE, only one column and one row, the full json array is loaded here
-- If STRIP_OUTER_ARRAY=TRUE, a new row is created per element in the array
SELECT JSON_VARIANT FROM FILMS_ELT;

-- TRUNCATE TABLE FILMS_ELT;


-- Query data and casting
SELECT
json_variant:id::string as id -- better practice is to cast as underlying data type
,json_variant:actors[0] as first_actor -- access first element of array. Not cast so enclosed in quotes
,json_variant:release_date as release_date -- appears as variant, enclosed in quotes
,json_variant:release_date::date as release_date_cast -- casted as date
,to_date(json_variant:release_date) as release_date_func_cast -- casted as date with func
,json_variant['id'] -- bracket instead of column notation
,json_variant['ratings'].imdb_rating -- subfields can be queried with . or with bracket
,json_variant:ratings['imdb_rating'] -- subfields can be queried with . or with bracket
FROM FILMS_ELT;

-- Case sensitivity
-- works as col names are insensitive in sql
SELECT jSOn_vArIanT:id FROM FILMS_ELT;

-- fails as fields in variants are case sensitive (returns null for all rows)
SELECT json_variant:Id FROM FILMS_ELT;



-- flatten
SELECT json_variant:ratings from FILMS_ELT LIMIT 1; -- here ratings is an object with 2 keys

-- Will produce two rows
SELECT
VALUE
FROM TABLE(
-- flatten input must return only one row hence the LIMIT
FLATTEN(INPUT => SELECT json_variant:ratings FROM FILMS_ELT LIMIT 1)
);


SELECT
json_variant:title
,json_variant:release_date::date
,L.value
,L.key
FROM FILMS_ELT F,
-- LATERAL allowed to combine the flattening of each row to itself,
LATERAL FLATTEN (INPUT => F.json_variant:ratings) L;

-----------------------------------
CREATE OR REPLACE TABLE FILMS_ETL (
ID STRING,
TITLE STRING,
RELEASE_DATE DATE,
STARS ARRAY,
RATINGS OBJECT
);


COPY INTO FILMS_ETL FROM
(SELECT
$1:id -- contrary to variant, no need to cast
,$1:title
,$1:release_date::date
,$1:actors
,$1:ratings
FROM @FILMS_STAGE/films.json
)
FILE_FORMAT = JSON_FILE_FORMAT
FORCE=TRUE;

-- we get a more standard table
SELECT
ID -- no need to cast contrary to variant
,TITLE
,RELEASE_DATE
,STARS[0]::string AS FIRST_STAR
,RATINGS['imdb_rating']::float AS imdb_rating
FROM FILMS_ETL;

-- match by column name
TRUNCATE TABLE FILMS_ETL;
COPY INTO FILMS_ETL
FROM @FILMS_STAGE/films.json
FILE_FORMAT=JSON_FILE_FORMAT
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
FORCE=TRUE;

SELECT * FROM FILMS_ETL;
```
