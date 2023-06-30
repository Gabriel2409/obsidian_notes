#snowflake

## Directory table

- can be enabled on internal and external named [[Snowflake - Stage|stages]]

```sql
-- enables it at creation
CREATE STAGE INT_STAGE
DIRECTORY = (ENABLE = TRUE);

-- or after
-- enables it at creation
ALTER STAGE INT_STAGE SET
DIRECTORY = (ENABLE = TRUE);
```

- It is simply an interface we can query to get information on metadatas of the files residing in the stage

```sql
-- get information on files
-- one of the fields is FILE_URL which is the same
-- as what we would get with BUILD_STAGE_FILE_URL
SELECT * FROM DIRECTORY(@INT_STAGE);
```

- Directory tables must be refreshed to reflect the most up to date changes

```sql
-- Look for the DIRECTORY property to see
-- if it is enabled and when was the last refresh
DESCRIBE STAGE DEMO_STAGE;
```

- Refresh can be done manually: `ALTER STAGE INT_STAGE REFRESH`
- External stages also support automated refreshes

```sql
-- creates the stage
CREATE STAGE EXT_STAGE
URL='s3//load/files'
STORAGE_INTEGRATION=MYINT
DIRECTORY = (ENABLE = TRUE)

-- retrieve ARN for snowflake managed SQS queue
-- in field directory_notification_channel
DESCRIBE STAGE EXT_STAGE;

-- In S3, we must configure event notification
```
