#snowflake
see [[Snowflake - Data loading]]

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
