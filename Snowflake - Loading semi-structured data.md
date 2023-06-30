#snowflake

see [[Snowflake - Data loading]] and [[Snowflake - Semi-structured data]]

## Loading semi-structured data

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

## Example worksheet

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

CREATE OR REPLACE TABLE 
FILMS_ELT (JSON_VARIANT VARIANT);

COPY INTO FILMS_ELT 
FROM @FILMS_STAGE/films.json
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
