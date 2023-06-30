---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 224
---

#snowflake

see [[Snowflake - Data loading]]

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
