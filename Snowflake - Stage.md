#snowflake
see [[Snowflake - Data loading]]

## Stages

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
