---
sr-due: 2023-07-05
sr-interval: 1
sr-ease: 224
reviewed: 2023-07-12
---

#snowflake

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
