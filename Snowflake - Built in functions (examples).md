---
reviewed: 2023-07-20
---

#snowflake

## Snowflake functions

Supported function types:

- Scalar
- Aggregate
- Window
- Table
- System
- User defined (see [[Snowflake - UDFs, external functions and procedures]])
- External (see [[Snowflake - UDFs, external functions and procedures]])

### Scalar functions

- Returns one value per invocation: mostly used for returning one value per row

```sql
SELECT UUID_STRING(); -- returns a unique string identifier
```

### Aggregate functions

- Operate on values across rows to perform calculation such as sum, average, counting

```sql
SELECT MAX(AMOUNT) FROM ACCOUNT;
```

#### Estimation functions

- Type of aggregation functions that allow to perform a calculation quicker but with less accuracy

##### Cardinality estimation

- Cardinality estimation estimates the nb of distinct values
- Snowflake implements the HyperLogLog cardinality estimation algorithm
- Avg relative error when compared to COUNT DISTINCT is 1.62338% => If `SELECT COUNT(DISTINCT C1)` equals 1,000,000 then HyperLogLog will be between 983,767 and 1,016,234
- Function name is `HLL`, alias is `APPROX_COUNT_DISTINCT`

```sql
SELECT APPROX_COUNT_DISTINCT(L_ORDERKEY) FROM LINEITEM;
```

##### Similarity estimation

- Similarity estimation estimates similarity of 2 or more sets
- Similarity is the measure Jaccard Similarity coeff (intersection over union)
- Snowflake implemented a two-step process to estimate similarity without the need to compute intersection and union of two sets

```sql
-- Returns a MinHash state containing an array of size XX
-- constructed by applying XX number of different hash functions
-- to the input rows and keeping the minimum of each hash function.
SELECT MINHASH(XX, C_CUSTKEY) FROM CUSTOMER;


--  approximate similarity, returns one unique value
SELECT APPROXIMATE_SIMILARITY(MH) FROM
(
	-- first row
	(SELECT MINHASH(5, C_CUSTKEY) MH FROM CUSTOMER)
		UNION
	-- second row
	(SELECT MINHASH(5, O_CUSTKEY) MH FROM ORDERS)
);

```

##### Frequency estimation

- Frequency estimation estimates frequency of values in a set
- Snowflake implements the space-saving algorithm which is faster than running a group by followed by an order by

```sql
-- COLNAME is the column we want to track
-- XX is the nb of values we want to track
-- YY is the max num of distinct values that can be tracked during estimation process
SELECT APPROX_TOP_K(COLNAME, XX, YY)
```

##### Percentile estimation

- Percentile estimation: estimates percentile of values in a set
- Snowflake implements the t-Digest algorithm

```sql
-- approximates the median
SELECT APPROX_PERCENTILE(score, 0.5)
```

### Window functions

- Subset of aggregate functions, allowing us to aggregate on a subset of rows used as input to a function
- Note that this is not a group by (all rows are returned)

```sql
SELECT ACCOUNT_ID, MAX(AMOUNT) OVER (PARTITION BY ACCOUNT_ID) FROM ACCOUNT;
```

### Table functions

- returns a set of rows for each input row
- The returned set can contain zero, one or more rows
- Each row can contain one or more columns

```sql
SELECT RANDSTR(5, RANDOM())
-- GENERATOR creates rows of data
-- to query the result, we wrap in a TABLE litteral
FROM TABLE(GENERATOR(ROWCOUNT => 3));
```

### System function

- Executes actions in the system

```sql
SELECT system$cancel_query('<query_id>')
```

- Provides information about the system

```sql
SELECT system$pipe_status('mypipe')
```

- Provides information about the query

```sql
SELECT system$explain_plan_json(SELECT AMOUNT FROM ACCOUNT)
```

### Table sampling

- Convenient way to read a random subset from a table
- Fraction based

```sql
-- generic way to sample
SELECT * FROM MYTABLE SAMPLE/TABLESAMPLE [SamplingMethod](<probability>);

-- sample by row, each row with 50% chance (default behavior)
SELECT * FROM MYTABLE SAMPLE ROW(50); -- we can use BERNOUILLI keyword instead of ROW


-- sample by block of rows, each block with 30% chance
SELECT * FROM MYTABLE SAMPLE BLOCK(30); -- we can use SYSTEM keyword instead of BLOCK


-- include a seed with SEED or REPEATABLE
-- note that seed will produce different results if data is changed
-- or if we sample a copy of the table
SELECT * FROM MYTABLE SAMPLE(40) SEED(42);
```

- Fixed size sampling: Gets an exact nb of rows

```sql
-- generic way
SELECT * FROM MYTABLE SAMPLE/TABLESAMPLE [SamplingMethod](<num> rows);

-- example with ROW
SELECT * FROM MYTABLE SAMPLE ROW(5 rows);

-- block sampling and seeds are not supported with fixed size
```

- Sampling the result of a JOIN is allowed, but only when all of the following are true:
  - The sample is row-based (Bernoulli).
  - The sampling does not use a seed.
- The sampling is done after the join has been fully processed. Therefore, sampling does not reduce the number of rows joined and does not reduce the cost of the JOIN.
- SYSTEM | BLOCK sampling is often faster than BERNOULLI | ROW sampling.
- Sampling without a seed is often faster than sampling with a seed.
- Fixed-size sampling can be slower than equivalent fraction-based sampling because fixed-size sampling prevents some query optimization.

### Unstructured File functions

- `BUILD_SCOPED_FILE_URL` generates a scoped Snowflake-hosted URL to a staged file using the [[Snowflake - Stage|stage]] name and relative file path as inputs.
  - A scoped URL is encoded and permits access to a specified file for a limited period of time.

```sql
-- General form
BUILD_SCOPED_FILE_URL( @<stage_name> , '<relative_file_path>' );

-- use in a query, currently active role must have permission on the stage
SELECT build_scoped_file_url(@image_stage, 'prod_zlc.jpg');


--  if this function is called in a UDF, stored procedure or view,
-- the calling role does not require privileges on the underlying stage
CREATE VIEW PRODUCT_SCOPED_URL_VIEW AS
SELECT build_scoped_file_url(@image_stage, 'prod_zlc.jpg') AS scoped_file_url;
```

- `BUILD_STAGE_FILE_URL` is similar but for permanent access: the URL does not expire

  - The function takes the same parameters but the output is different: it is no longer encoded and includes the database, schema, stage identifier and the relative file path
  - However, it requires privileges on the underlying stage (USAGE for external and READ for internal) even if part of a view, a UDF or a stored procedure

- `GET_PRESIGNED_URL`: gets the url to download the file directly in the cloud provider environment (not snowflake hosted)

```sql
-- General form
GET_PRESIGNED_URL( @<stage_name> , '<relative_file_path>' , [ <expiration_time> ] )
```

- Example outputs

```sql
-- name column contains the files
ls @DEMO_STAGE;

SELECT get_stage_location(@DEMO_STAGE);
-- output: s3://sfc-.../.../stages/.../



-- removes the demo stage url from path
SELECT get_relative_path(@DEMO_STAGE, s3://.../../path/to/file);
-- output: path/to/file

-- proof: below query returns images/analytics.jpg
SELECT get_relative_path(@DEMO_STAGE, get_stage_location(@DEMO_STAGE) || 'images/analytics.jpg');

-- get absolute path of a stage file (appends the stage location)
SELECT GET_ABSOLUTE_PATH( @DEMO_STAGE , 'images/analytics.jpg' );
-- output: s3://.../../images/analytics.jpg


SELECT get_presigned_url( @DEMO_STAGE , 'images/analytics.jpg', 60 );
-- output: https://sfc-...s3.eu-west-3.amazonaws.com/.../stages/../images/analytics.jpg?<queryparams>

SELECT build_scoped_file_url( @DEMO_STAGE , 'images/analytics.jpg');
-- output: https://<locator>.<region>.<cloudprovider>.snowflakecomputing.com/api/files/...
```
