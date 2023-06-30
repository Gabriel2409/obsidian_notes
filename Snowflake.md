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

- [[Snowflake - Data loading]]
- [[Snowflake - Data unloading]]

# Data transformation

## Snowflake functions

#todo: combine with UDFs

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


##### Similarity  estimation
- Similarity estimation estimates similarity of 2 or more sets
- Similarity is  the measure Jaccard Similarity coeff  (intersection over union)
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

## Table sampling