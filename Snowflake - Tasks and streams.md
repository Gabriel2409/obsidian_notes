---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 224
---

#snowflake

see [[Snowflake - Objects]]


### Tasks

- Object used to schedule the execution of a SQL statement, a stored procedure or procedural logic with Snowflake Scripting
- can be created by `ACCOUNTADMIN` or people with `CREATE TASK` privilege

```sql
CREATE TASK T1 -- task name
	WAREHOUSE = MYWH -- warehouse can be skipped to use snowflake managed compute resources instead
	SCHEDULE = '30 MINUTE' -- triggering mechanism
AS
	COPY INTO MYTABLE FROM $MYSTAGE -- query definition
```

- creating a task does not automatically start it, we must first issue: `ALTER TASK MYTASK RESUME;`
- A task can be paused with `ALTER TASK MYTASK SUSPEND;`

- Tasks can be chained together to form a DAG (directed acyclic graph)
  - up to 1000 tasks in a DAG
  - child tasks do not define a schedule: In the create task definition, replace the `SCHEDULE` with `AFTER T2, T3` to signify that the tasks must be executed after T2 and T3 were successfully executed.

### Streams

- Object created to view and track DML changes to a source table - inserts, updates and deletes
- streams can be created on top of a table: `CREATE STREAM MYSTREAM ON TABLE MYTABLE`
- streams can be queried: `SELECT * FROM MYSTREAM`. We get an object with the same structure that contains the changes since the stream creation. Note that querying the stream does not progress the offset
- Streams also contain 3 additional columns:
  - `METADATA$ACTION` : INSERT or DELETE
  - `METADATA$IS_UPDATE` : TRUE or FALSE
  - `METADATA$ROW_ID`: ref to the row
- To progress the offset of a stream, insert it into a downstream table

```sql
CREATE TABLE 
MYTABLE(NAME STRING,AGE INT);

INSERT INTO MYTABLE VALUES ('a', 1);

CREATE STREAM MYSTREAM 
ON TABLE MYTABLE;

SELECT * FROM MYSTREAM; -- does not show the initial record

INSERT INTO MYTABLE VALUES ('b', 2);
INSERT INTO MYTABLE VALUES ('c', 3);

SELECT * FROM MYSTREAM; -- shows b,2 and c,3 record
SELECT * FROM MYSTREAM; -- also shows b,2 and c,3 record

-- create a downstream table
CREATE OR REPLACE TABLE TABLE2(
    NAME STRING,
    AGE INT,
    METADATA$ACTION STRING,
    METADATA$IS_UPDATE BOOLEAN,
    METADATA$ROW_ID STRING
);

INSERT INTO TABLE2 
SELECT * FROM MYSTREAM; -- adds stream elements to table and progresses the offset

SELECT * FROM MYSTREAM; -- empty
SELECT SYSTEM$STREAM_HAS_DATA('MYSTREAM'); -- returns False
```
### Combining task and stream

- we can create tasks to automatically process streams:

```sql
CREATE TASK MYTASK
	WAREHOUSE = MYWH
	SCHEDULE = '5 MINUTE' -- tasks is launched every 5 mins
WHEN
	SYSTEM$STREAM_HAS_DATA('MYSTREAM') -- check that stream is not empty
AS
	... -- do all the processing

	INSERT INTO MYTABLE(ID,NAME) SELECT ID, NAME FROM MYSTREAM -- progresses the offset
```