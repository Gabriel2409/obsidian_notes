#snowflake

https://docs.snowflake.com/

# Features and architecture

## Overview

Snowflake is a Cloud Native Data Platform SaaS

Data platform

- data warehouse:
  - Structured and relational data
  - ANSI standard SQL
  - ACID compliant transactions
  - Data stored in databases, schemas and tables
- data lake:
  - scalable storage and compute
  - schema does not need to be defined upfront
  - Native processing of semi structured data formats
- data engineering
  - Simplify data ingestion for batch and streaming workloads (COPY INTO and Snowpipe)
  - Instantiate on the fly separate compute clusters to eliminate contention between ETL and analytic jobs
  - Native objects to create data pipelines (Tasks and Streams)
  - Data encryption at rest and in transit
- data science:
  - Have a unique centralized storage
  - Partner eco system with SageMaker, DataRobot, Dataiku...
- data sharing:
  - secure data sharing with other accounts
  - can connect to BI tools such as Tableau or PowerBI
- data applications:
  - connectors and drivers
  - UDFs and stored procedures written in sql, python, java or javascript
  - External UDFs
  - querying with Snowpark

Cloud Native

- Snowflake built from scratch for the cloud
- All infrastructure is provisionned in either AWS, GCP and Azure
- Snowflake makes use of the cloud benefits (elasticity, scalability, high availability, cost effectiveness, durability)

SaaS

- No management of hardware
- transparent updates and patches
- Pay as you use subscription model
- easy to access
- automatic optimisation (no need to specify indexes or static partitions)

## Multi-cluster shared data architecture

- [[Shared disk architecture]] was the first move away from [[Shared memory architecture|Single node architecture]] by allowing compute on several machines but keeping storage in one place
- [[Shared nothing architecture]] is the next eveolution: no sharing of hardware. This is what we have in HADOOP

Snowflake built a new architecture specifically for the cloud to do better than these two prevalent architectures: the multi cluster shared data architecture:

- Data Storage layer: layer where all the data is stored
- Query processing layer: separate compute clusters executing the computation to process the queries to create virtual warehouses that snowflake provisions and manages
- Cloud services layers handing everything that is not storage and compute

Note: there is also a Cloud agnostic layer here to make sure snowflake works on any cloud provider

Each layer are physically separated and communicate over a network via RESTful interfaces. For ex, the virtual warehouses are completely separate from the table where we keep the long term data.
Each layer is scaled independently, this is why analytics and pipeline jobs do not compete for resources.

![[snowflake_multicluster.png.png]]

### Storage Layer

- Blob storage of the cloud provider we deployed our account into. For AWS, it will be S3 for ex. As such we get the availability and durability guarantee this blob services offer
- Data loaded into Snowflake is organized by databases, schemas and accessible primarily as tables
- Snowflakes support structured and semi structured file formats (CSV, JSON, Avro, ORC, Parquet, XML)
- When datafiles are loaded or rows inserted into a table, Snowflake reorganizes the data into its proprietary compressed, columnar table file format (optimized for OLAP workloads)
- The data is partitioned ito micro-partitions
- Storage is billed by how much is stored based on a flat rate per TB
- Data is not directly accessible in the underlying blob storage, only via SQL commands

### Query Processing Layer

- also referred as the compute layer
- consists of Virtual Warehouses that execute the processing tasks required to return results for most SQL statements
- A Virtual Warehouse
  - is a named abstraction for a cluster of cloud based compute instances that Snowflake manages.
  - For ex, if we execute `CREATE WAREHOUSE MY_WAREHOUSE WAREHOUSE_SIZE=LARGE;`, behind the scene, AWS would provision EC2 instances.
  - As Snowflake is a SaaS, we don't have access to the nodes, we can only interact with the abstraction.
  - Underlying nodes cooperate in a similar way to [[Shared nothing architecture|shared nothing]] compute clusters with local caching.
  - can be created or removed instantly
  - can be paused or resumed (no billing when paused)
  - come in many sizes: small, medium, large, ... (from xs to 6xl)
- We can create virtually unlimited virtual warehouses, each with its own config. Each warehouse is isolated from each other
- All running virtual warehouses have **consistent** (see [[Consistency patterns]]) access to the same data in the storage layer. Snowflake is NOT eventually consistent, it uses strict ACID compliant processing, which is achieved thanks to the Transaction service in the Global service layer which synchronises data access

### Services layer

- collection of highly available and scalable services that coordinate activities across ALL snowflake accounts
  - authentication and access control
  - Infrastructure management: handles creation and management of underlying cloud resources
  - Transaction management: ensures that the data warehouse is ACID compliant
  - metadata management: keeps information and stats on objects and data
  - query parsing and optimization: takes the sql query we submit and turn it into an actionnable plan for the virtual warehouse to execute
  - security
- Because the services layer is multi tenant, it allows snowflake to perform economies of scale and make features such as secure data sharing easier.
- Behind the scenes, it works on cloud compute based instances similarly to virtual warehouses

## Snowflake editions

https://www.snowflake.com/pricing

- Standard
- Enterprise
- Business Critical
- Virtual Private Snowflake: here service layer is separated from other accounts

## Snowflake Objects

- An object is something we can interact with
- Each object has its own set of privileges
- Snowflake object Model is hierarchical

At the top, we have an `Organisation` that can manage several `Account` objects.
Below we have `Account level objects` to configure different parts of the account (how many `User` exist, how many `Warehouse`, etc...).

Among them, `Database` are organized into `Schema` which are comprised of different objects: `Table`, `View`, `Stream`, ...

### Organization

- Manage one or more Snowflake accounts
- Setup and administer Snowflake features which make use of multiple accounts
- Monitor biling accross multiple accounts
- People managing the organisation have an `ORGADMIN` role.

`ORGADMIN` can create accounts, enable cross-account features, monitor account usage

By default, even if you have the `ORGADMIN` role, it is not enable by default.

In snowsql, run `use role ORGADMIN;`
`SHOW ORGANIZATION ACCOUNTS;` will show the accounts

### Account

- administrative name for a collection of storage, compute and cloud services deployed and managed entirely on a selected cloud platform
- each account is hosted on a single cloud provider
- Each account resides in a single geographic region, which has an impact on regulation, compute price, and even available snowflake features
- Each account is created as a single snowflake edition (can be changed later on)
- By default created with the system defined role `ACCOUNTADMIN`

When an account is created,

- we have a url such as `https://<locator>.<region>.<cloudprovider>.snowflakecomputing.com`, for ex: `xyz123.us-east-2.aws.snowflake.computing.com`
- and an identifier : `<organisation>-<accountname>`, for ex: `ABCDEFG-AB12345`
  Both urls and identifiers are unique.

### Database

- Must have a unique identifier in an account

```sql
CREATE DATABASE MYDATABASE; -- Basic creation

CREATE DATABASE MYCLONE CLONE MYOTHERDB; -- Clone an existing db

-- create a replica of a db in another account
CREATE DATABASE MYDB1
	AS REPLICA OF MYORG.ACCOUNT1.MYDB1
	DATA_RETENTION_TIME_IN_DAYS = 10;

-- create from a share object
CREATE DATABASE SHARED_DB FROM SHARE UTT783.SHARE
```

Schemas allow for further segmentation

- Schemas must have a unique identifier in a database

```sql
CREATE SCHEMA MYSCHEMA; -- Basic creation

CREATE SCHEMA MYCLONE CLONE MYOTHERSCHEMA; -- Clone an existing schema
```

A **namespace** consists of a database and a schema together: `MYDB.MYSCHEMA`

### Table

- Permanent table:
  - Default table type
  - exists until explicitely dropped
  - **Time travel retention period** = how long can we go into the past to restore data (up to 90 days for an enterprise account vs 1 for standard)
  - 7 days **fail-safe**: snowflake can restore deleted data for us during this period
- Temporary table:
  - Used for transitory data
  - Persists for duration of a session
  - Can not be converted
  - Max time travel of 1 day and no fail safe
- Transient table
  - Similar to permanent: exist until explicitely dropped
  - However no fail safe period and max time travel of 1 day
  - Can decrease storage cost compared to permanent
- External table:
  - Query data outside of Snowflake
  - read-only table
  - slower
  - no time travel or failsafe

Type can be seen with `SHOW TABLES;` and checking the kind.

### View

- Standard view:
  - Stores the query definition
  - can be used as a table (we can even use `ORDER BY`)
  - does not contribute to storage cost as they don't store any data
  - If source table is dropped, view will produce an error

```sql
CREATE VIEW MYVIEW AS
SELECT COL1, COL2 FROM MY_TABLE;
```

- Materialized view:
  - Stores the query definition
  - ALSO stores the result of the query definition and periodically refreshes it
  - Incurs cost as a serverless feature
  - Also incurs storage cost and compute cost (automatic background maintenance)
  - can be used to boost performance of external tables
  - Check `MATERIALIZED_VIEW_REFRESH_HISTORY` in the `ACCOUNT_USAGE` schema to track consumption
  - Limitations:
    - Limited to a single table
    - Can not use join
    - Can not use UDF, HAVING, ORDER BY, LIMIT, WINDOWS FUNCTIONS

```sql
CREATE MATERIALIZED VIEW MYVIEW AS
SELECT COL1, COL2 FROM MY_TABLE;

-- suspend a materialized view to pause the update (also makes it inaccessible)
ALTER MATERIALIZED VIEW MYVIEW SUSPEND;

-- restarts the background process
ALTER MATERIALIZED VIEW MYVIEW RESUME;
```

- Both standard and materialized views can be secure by adding the `SECURE` keyword: `CREATE SECURE VIEW...`
  - Query definition is only visible to authorized users
  - Some query optimizations (for ex doing filter first) will be bypassed to improve security (can make view slower)

Check views with `SHOW VIEWS;` and `SHOW MATERIALIZED VIEWS`

### Worksheet Example query - Database, schema, table, views

- Navigate to worksheet and create a new worksheet
- set context with sql or select it from dropdown from within the script.

```sql
-- Set context so that we don't have to specify db and schema on all queries
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SNOWFLAKE_SAMPLE_DATA;
USE SCHEMA TPCH_SF1

-- Get information on a table
DESCRIBE TABLE CUSTOMER;

-- show all tables in current context
SHOW TABLES;

-- Show all tables starting with PA
SHOW TABLES LIKE 'PA%';

-- show name column from before last query
SELECT "name" FROM TABLE(result_scan(last_query_id(-2)));

-- Create new db
CREATE DATABASE DEMO_DB;
CREATE SCHEMA DEMO_SCHEMA;
-- switch context
USE DATABASE DEMO_DB;
USE SCHEMA DEMO_SCHEMA;
-- create tables
CREATE TABLE PERMANENT_TABLE(
    NAME STRING,
    AGE INT
);
CREATE TEMPORARY TABLE TEMPORARY_TABLE(
    NAME STRING,
    AGE INT
); -- will disappear after loging out and relogging in.
CREATE TRANSIENT TABLE TRANSIENT_TABLE(
    NAME STRING,
    AGE INT
);
SHOW TABLES; -- check kind (should be TABLE, TEMPORARY and TRANSIENT)

-- successful, see list of parameters here: https://docs.snowflake.com/en/sql-reference/parameters
ALTER TABLE PERMANENT_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 90;

-- failure, can only be 1 or 0;
ALTER TABLE TRANSIENT_TABLE SET DATA_RETENTION_TIME_IN_DAYS = 2;


-- create views
CREATE VIEW STANDARD_VIEW AS
SELECT * FROM PERMANENT_TABLE;

CREATE SECURE VIEW SECURE_VIEW AS
SELECT * FROM PERMANENT_TABLE;

CREATE MATERIALIZED VIEW MATERIALIZED_VIEW AS
SELECT * FROM PERMANENT_TABLE;

 -- check is_secure and is_materialized fields. text contains the view definition
SHOW VIEWS;
SELECT "name", "database_name", "schema_name", "is_secure", "is_materialized", "text"
FROM TABLE(result_scan(last_query_id()));

-- Secure view functionality
GRANT USAGE ON DATABASE DEMO_DB TO ROLE SYSADMIN;
GRANT USAGE ON SCHEMA DEMO_SCHEMA TO ROLE SYSADMIN;
GRANT SELECT, REFERENCES ON TABLE STANDARD_VIEW TO ROLE SYSADMIN;
GRANT SELECT, REFERENCES ON TABLE SECURE_VIEW TO ROLE SYSADMIN;
USE ROLE SYSADMIN;
SELECT get_ddl('view', 'STANDARD_VIEW'); -- definition shows correctly
SELECT get_ddl('view', 'SECURE_VIEW'); -- error
```

### User Defined functions(UDFs)

https://docs.snowflake.com/en/sql-reference/udf-overview

- Schema level objects that enable users to write their own functions in four languages:
  - SQL
  - Javascript
  - Python
  - Scala
  - Java
- accepts 0 or more parameters
- Result can be scalar or tabular (UDTF)

```sql
-- scalar
CREATE FUNCTION area_of_circle(radius FLOAT)
	RETURNS FLOAT
	AS
	$$
		pi() * radius * radius
	$$;

-- tabular
CREATE FUNCTION area_of_circle(radius FLOAT)
	RETURNS TABLE (area number)
	AS
	$$
		pi() * radius * radius
	$$;
```

- UDFs can be called as part of a SQL statement: `SELECT area_or_circle(col1) FROM MY_TABLE;`
- UDFs can be overloaded (we can create multiple with the same name provided nb of arguments are different)

- To specify a language and enable use of high level programming features:
- Note: Javascript UDFs can refer to themselves recursively
- Snowflake data types are mapped to JavaScript data types, for ex integer are mapped to double in js.

```sql
CREATE FUNCTION js_factorial(d double)
	RETURNS DOUBLE
	LANGUAGE JAVASCRIPT
	AS
	$$
	if (d <= 0){
	return 1
	} else {
		var result = 1;
		for (var i = 2; i <= d; i++){
			result = result * i
		}
		return result;
	}
	$$;
```

- For Java UDFs, Snowflake boots up a JVM to execute functions written in java
- Java UDFs can specify their definition as in-line code or precompiled jar file.
- There are two additional parameters: HANDLER which specifies which function to execute and TARGET_PATH which specifies the location of the jar file
- Java UDFs can not be designated as secure

```sql

CREATE FUNCTION java_double(x INTEGER)
	RETURNS INTEGER
	LANGUAGE JAVA
	HANDLER='TestDoubleFunc.double'
	TARGET_PATH='@~/TestDoubleFunc.jar'
	AS
	$$
		class TestDoubleFunc {
			public static int java_double(int x){
			return x * 2
			}
		}
	$$;
```

### External functions

https://docs.snowflake.com/en/sql-reference/external-functions-introduction

Unlike other UDFs, an external function does not contain its own code; instead, the external function calls code that is stored and executed outside Snowflake.

Inside Snowflake, the external function is stored as a database object that contains information that Snowflake uses to call the remote service.

Snowflake does not call a remote service directly. Instead, Snowflake calls the remote service through a cloud provider’s native HTTPS proxy service, for example API Gateway on AWS.

- Limitations
  - slower
  - scalar only
  - not sharable
  - less secure
  - can incur egress charges

### Stored procedures

- In RDBMS, stored procedures were named collections of SQL statements often containing procedural logic

- In snowflake we can include stored procedures in different ways:
  - Javascript
  - Snowflake scripting = SQL with procedural logic
  - Snowpark to create procedures in python, java and scala

```sql
CREATE OR REPLACE PROCEDURE TRUNCATE_ALL_TABLES_IN_SCHEMA(DATABASE_NAME STRING, SCHEMA_NAME STRING)
	RETURNS STRING
	LANGUAGE JAVASCRIPT -- can only return a scalar with js
	EXECUTE AS OWNER -- can execute with the owner's rights or the caller's rights
	AS
	$$
		var result = [];
		var namespace = DATABASE_NAME + '.' + SCHEMA_NAME;
		var sql_command = 'SHOW TABLES in ' + namespace;
		var result_set = snowflake.execute(sqlText: sql_command); -- use snowflake js API to execute sql
		while (result_set.next()){
			var table_name = result_set.getColumnValue(2);
			var truncate_result = snowflake.execute({sqlText: 'TRUCATE TABLE ' + table_name});
			result.push(namespace + '.' + table_name + ' has been successfully truncated');
		}
		return result.join("\n");
	$$;
```

A procedure is then called with `CALL TRUNCATE_ALL_TABLES_IN_SCHEMA('DEMO_DB', 'DEMO_SCHEMA')`;

Differences with UDFs:

- Only UDFs can be called as part of a SQL statement
- Both can be overloaded
- Both can take 0 or more input args
- Only stored procedure can use the Javascript API
- Stored procedures don't necessarily need to return a value. And the value returned by a stored procedure can not be used directly in sql
- Not all UDFs can refer to themselves recursively but stored procedures can

Stored procedures must be seen as performing actions instead of returning values

### Sequences

- used to generate unique numbers across sessions and statements, including concurrent statements. They can be used to generate values for a primary key or any column that requires a unique value.
- There is no guarantee that values from a sequence are contiguous (gap-free) or that the sequence values are assigned in a particular order.
- NOTE: Changing the sequence interval from positive to negative (e.g. from `1` to `-1`), or vice versa may result in duplicates

```sql
CREATE OR REPLACE SEQUENCE DEFAULT_SEQ
START = 1
INCREMENT = 2;

SELECT DEFAULT_SEQ.NEXTVAL; -- 1
SELECT DEFAULT_SEQ.NEXTVAL; -- 3

SELECT DEFAULT_SEQ.NEXTVAL, DEFAULT_SEQ.NEXTVAL; -- 5,7
SELECT DEFAULT_SEQ.NEXTVAL, DEFAULT_SEQ.NEXTVAL; -- 11,13 (There is a gap)
```

- We can use a sequence inside a table

```sql
CREATE OR REPLACE SEQUENCE TRANSACTION_SEQ
START = 1001
INCREMENT = 1;

CREATE TABLE TRANSACTIONS(ID INTEGER DEFAULT TRANSACTION_SEQ.NEXTVAL, AMOUNT DOUBLE);
INSERT INTO TRANSACTIONS(AMOUNT) VALUES(5487); -- automatically adds the ID column
INSERT INTO TRANSACTIONS(ID,AMOUNT) VALUES(12,45); -- also works and inserts 12 as an ID
```

- `GETNEXTVAL(myseq)` returns a result set, we can see it with `TABLE(GETNEXTVAL)`
- SEQUENCES can be altered: `ALTER SEQUENCE myseq SET INCREMENT = -4;`

### Tasks

- Object used to schedule the execution of a SQL statement, a stored procedure or procedural logic with Snowflake Scripting
- can be created by ACCOUNTADMIN or people with CREATE TASK privilege

```sql
CREATE TASK T1 -- task name
	WAREHOUSE = MYWH -- warehouse can be skipped to use snowflake managed compute resources instead
	SCHEDULE = '30 MINUTE' -- triggering mechanisme
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
- Streams also contain 3 additional metadatas:
  - `METADATA$ACTION` : INSERT or DELETE
  - `METADATA$IS_UPDATE` : TRUE or FALSE
  - `METADATA$ROW_ID`: ref to the row
- To progress the offset of a stream, insert it into a downstream table

```sql
CREATE TABLE MYTABLE(NAME STRING,AGE INT);
INSERT INTO MYTABLE VALUES ('a', 1);
CREATE STREAM MYSTREAM ON TABLE PERMANENT_TABLE;

SELECT * FROM MYSTREAM; -- does not show the initial record

INSERT INTO PERMANENT_TABLE VALUES ('b', 2);
INSERT INTO PERMANENT_TABLE VALUES ('c', 3);

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

INSERT INTO TABLE2 SELECT * FROM MYSTREAM; -- adds stream elements to table and progresses the offset

SELECT * FROM MYSTREAM; -- empty
SELECT SYSTEM$STREAM_HAS_DATA('MYSTREAM'); -- returns False
```

- we can combine streams with tasks to automatically process streams:

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

## Billing

Note: snowflake credits are a billing unit of measure for compute resources

- On demand (Pay as you go with min 25$) VS Pay for usage upfront (lower rate)
- There are 5 main areas where an account is billed:
  - **Virtual warehouse Service**: operations that use user managed virtual warehouses
    - Uses snowflake credits
    - credit calculated based on size of warehouse, nb of cluster on a per second basis while warehouse is in a started state (min of 60 seconds)
  - **Cloud Services**: includes operations that don't use virtual warehouse but still needs compute (for ex: executing a `SHOW` or `DESCRIBE` command)
    - Uses snowflake credits (4.4 credits per hour)
    - Only cloud services that exceed 10% of daily usage of compute resources are billed (this is called cloud services adjustment)
  - **Serverless Services**: where snowflake spins compute
    - Uses snowflake credits
    - Each serverless feature has its own credit rate
    - Composed of both compute and cloud services
    - Cloud services adjustment does not apply
  - **Data storage**: temporary holding location like a stage or long term storage like a table
    - charged in currency
    - calculated monthly based on avg nb of on disk bytes per day for database tables and internal stages
    - calculated based on flat dollar value rate per TB based on capacity or on-demand, cloud provider and region
  - **Data transfer**: transfer data out of snowflake or to another account if we change region or cloud provider
    - charged in currency
    - when moving data from one region to another
    - when unloading data from snowflake with `COPY INTO <location>`
    - when replicating data to a snowflake account in a different region or cloud provider
    - when using external function to transfer data in and out of snowflake

## Connectivity

### Snowsql

https://docs.snowflake.com/en/user-guide/snowsql-install-config

After installing, config file is by default in `~.snowsql/config`

In order not to have to pass credentials for each command, we need to modify the config to connect via snowsql by specifying account name, username and password.

```
[connections]
accountname = myaccountname
username = myusername
password = mypassword
```

Now if we launch `snowsql` with no parameters, we should be able to connect using these parameters as the new default

We can also specify the name of the connection:

```
[connections.example]
accountname = examplename
username = exampleuser
password = examplepassword
```

Now if we launch `snowsql -c example`, it will use the correct parameters

NOTE: As password is stored in plan text, we must be careful with permissions. More secure connection options exist

#todo, add some commands

### Connectors, drivers and partnered tools

#todo

### Snowflake scripting

#todo

### Snowpark

#todo

# Account Access and security

## Access Control

- Two ways:
  - RBAC: Privilege granted to a role granted to a user, similar to [[Azure - RBAC]],
  - DAC (Discretionary access control): each object has its owner who in turn can grant access to that object
    - Every securable object is owned by a single role, which can be seen when running `SHOW ...`
    - the owner can grant and remove privileges, transfer ownership to another role,..
    - Access to objects is also defined by privileges granted to roles

### Role

- A role is an entity to which privileges on securable objects can be granted (GRANT) or revoked (REVOKE): `GRANT USAGE ON DATABASE MYDB TO ROLE MYROLE;`
- roles are assigned to users and a user can switch between roles within a snowflake session.
- As roles are also securable objects, roles can be granted to other roles as well: `GRANT ROLE ROLE3 TO ROLE ROLE2`: parent roles inherit privileges of children roles

- System roles hierarchy:
  - ORGADMIN manages operation at an org level + ACCOUNTADMIN
  - ACCOUNTADMIN encapsulates SECURITYADMIN and SYSADMIN
  - SECURITYADMIN manages grants globally with `MANAGE GRANTS` privilege + USERADMIN
  - USERADMIN manages users and roles via `CREATE ROLE` and `CREATE USER` security privileges
  - SYSADMIN manages all objects
  - PUBLIC is automatically granted to users, can own securable objects but they are available to every other user and role

```text
			  ORGADMIN
			ACCOUNTADMIN
SECURITYADMIN           SYSADMIN
USERADMIN
			   PUBLIC
```

- System roles can not be dropped but we can add privileges to them. But this is STRONGLY discouraged. It is better to create custom roles instead. It is recommended to create a hierarchy of custom roles with the top-most custom role assigned to the SYSADMIN role so that SYSADMIN can manage the objects owned by the custom role

### Privilege

- Defines the level of access to a securable object
- Each object has an associated set of privileges that can be granted
- 4 catagories of security privileges
  - Global privileges: for ex create db, monitor account usage
  - Privileges for account objects, grants different access levels to objects such as db
  - Privileges for schemas: defines what you can do within a schema
  - Privileges for schema objects: defines what you can do to objects such as table
- Privileges are managed using GRANT and REVOKE (can be done by Owners and roles that have MANAGE GRANT privilege)
- Future grant allow privileges to be defined for objects not yet created: `GRANT SELECT ON FUTURE TABLES IN SCHEMA MY_SCHEMA TO ROLE MY_ROLE`

### Worksheet example

```sql
USE ROLE ACCOUNTADMIN;

-- get information about roles
SHOW ROLES;
SELECT "name", "comment" FROM TABLE(result_scan(last_query_id()));

-- see all privileges for a role
SHOW GRANTS TO ROLE SECURITYADMIN;

-- Create new db and schema
USE ROLE SYSADMIN;
CREATE DATABASE FILMS_DB;
CREATE SCHEMA FILMS_SCHEMA;
CREATE TABLE FILMS_SYSADMIN
(
    ID STRING,
    TITLE STRING,
    RELEASE_DATE DATE,
    RATING INT
);

-- create a new role
USE ROLE SECURITYADMIN;
CREATE ROLE ANALYST;

-- allow ANALYST role to execute USE FILMS_DB and SHOW <objects> in the db;
GRANT USAGE ON DATABASE FILMS_DB TO ROLE ANALYST;

-- allow ANALYST role to USE the schema, show <objects> in it and also create tables
GRANT USAGE, CREATE_TABLE ON SCHEMA FILMS_DB.FILMS_SCHEMA TO ROLE ANALYST;

-- Without this, no compute is possible (commands such as SHOW are still possible)
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST;

-- Make ANALYST a child role to SYSADMIN => SYSADMIN inherits every privileges of ANALYST
-- Note that it does NOT mean that a user that has the SYSADMIN role can switch to ANALYST
GRANT ROLE ANALYST TO ROLE SYSADMIN;

-- Allow the user to switch to the ANALYST role
GRANT ROLE ANALYST TO USER MYUSER;

-- shows the details of the privileges
SHOW GRANTS TO ROLE ANALYST;
SHOW GRANTS TO ROLE SYSADMIN; -- one of the row is USAGE on ROLE ANALYST

-- Future grants
USE ROLE SECURITYADMIN;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE FILMS_DB TO ROLE ANALYST;


USE ROLE SYSADMIN;
CREATE SCHEMA MUSIC_SCHEMA;
USE ROLE ANALYST;
SHOW SCHEMAS; -- MUSIC_SCHEMA is here

--Create a user and assign the role to it
USE ROLE USERADMIN;
CREATE USER USER1 password="temp" default_role = ANALYST DEFAULT_WAREHOUSE='COMPUTE_WH' MUST_CHANGE_PASSWORD=TRUE;
USE ROLE SECURITYADMIN;
GRANT ROLE ANALYST TO USER USER1;


SHOW USERS;
```

## Authentication and Authorization

### User authentication

- default method of authentication
- When a SYSADMIN creates a user, he can specify a very short password that does not meet the security requirements. In this case, it is better to force a change on first login

```sql
CREATE USER USER1
PASSWORD ='ABC123'
DEFAULT_ROLE = MYROLE
MUST_CHANGE_PASSWORD = TRUE;
```

### Multi factor authentication

- In snowflake, powered by Duo Security service, managed by snowflake
- user must download the DUO app
- MFA is enabled on a per-user basis and only via the UI
- Snowflake recommends that ACCOUNTADMIN have MFA

```sql
-- specifies the nb of minutes to temporarily disable MFA for the user to log in
ALTER USER USER1 SET MINS_TO_BYPASS_MFA=10;

-- disables MFA for the user, cancelling their enrolment
ALTER USER USER1 SET DISABLE_MFA=TRUE;

-- MFA token caching reduces the nb of prompts that must be acknowledged
ALTER USER USER1 SET ALLOWS_CLIENT_MFA_CACHING=TRUE;
```

### Federated Authentication

- enables users to connect to snowflake using secure SSO
- Snowflake can delegate authentication responsibility to an SAML 2.0 compliant external identity provaider (IdP) with native support for Okta and ADFS IdPs
- In a federated environment, snowflake is referred to as a Service Provider (SP)
- This is done by creating an object called security integration

```sql
CREATE SECURITY INTEGRATION <name>
	TYPE = SAML2
	ENABLED = TRUE
	SAML2_ISSUER = 'XXX'
	SAML2_SSO_URL = 'XXX'
	SAML2_PROVIDER = 'XXX'
	SAML2_X509_CERT = 'XXX'
```

### Key Pair authentication

- alternative to username/password authentication when connecting via a client (not the UI)
- generate public private key with openssl
- Then in snowflake:

```sql
ALTER USER USER1 SET RSA_PUBLIC_KEY='xxxx';
```

- Configure key roation

```sql
ALTER USER USER1 SET RSA_PUBLIC_KEY_2='yyyy';
ALTER USER USER1 UNSET RSA_PUBLIC_KEY='xxxx';
```

### OAuth and SCIM

- Snowflake supports the OAuth2 protocol with 2 pathways

  - Snowflake OAuth
  - External OAuth

- SCIM: System for Cross-domain Identity Management is used to manage snowflake users and roles using RESTful APIs.
  - for ex, an IdP like ADFS uses a SCIM client to make RESTful APIs requests to the Snowflake SCIM server
  - SCIM can be used in the user lifecycle (creating, updating settings,...)

## Network policies

- provides the user with the ability to allow or deny access to their Snowflake account based on IP address
- Network policies only support ipv4
- we use the CIDR notation to express an IP subnet range
- Network policies can be applied on the account level or to individual users (user level policy takes precedence)
  - Only one network policy can be associated with an account at any one time
  - Only one network policy can be associated with a user at any one time
- Policy can be bypassed but you need to contact snowflake support
- Policies can be applied by SECURITYADMIN or any role with the ATTACH POLICY global privilege

```sql
-- creating a network policy
CREATE NETWORK POLICY MY_POLICY
ALLOWED_IP_LIST = ('192.168.1.0/24') -- if specified, all addresses not in the list are blocked
BLOCKED_IP_LIST = ('192.168.1.99'); -- additional blocked ips within allowed range

-- apply policy to acount / user
ALTER ACCOUNT SET NETWORK_POLICY MY_POLICY;
ALTER USER USER1 SET NETWORK_POLICY MY_POLICY;

-- see list of policies
SHOW NETWORK POLICIES;

-- See applied policies
SHOW PARAMETERS LIKE 'MY_POLICY' IN ACCOUNT;
SHOW PARAMETERS LIKE 'MY_POLICY' IN USER USER1;
```

## Data encryption

- Encryption at rest
  - All data that snowflake has control over is AES-256 encrypted (table data, internal storage data, virtual warehouse and query results cache,...)
- Encryption in transit:
  - Secure https is used with TLS 1.2 protocol

=> End to End encryption

- When uploading to an internal stage, snowflake transparently encrypts data then encrypts with a different key to internal storage
- When uploading to an external stage, client is responsible for encryption and must provide snowflake with information to decrypt the data

Snowflake uses a hierarchical key model where each key encrypts the key below it and the final key encrypts the user's data:

- Root key
- Account master keys: encrypts an account
- Table master keys: encrypts a database
- File keys: each datafile or micropartition is encrypted with a separate key

This limits the amount of data a key protects.

- Snowflake also uses key rotation = practise of transparently replacing existing account and table encryption keys every 30 days with a new key. Previous key is only used to decrypt data.
- If we enable rekeying, after 1 year, key is retired and associated information is reencrypted with newest key: `ALTER ACCOUNT SET PERIODIC_DATA_REKEYING = TRUE;`

- Tri-secret secure allows to encrypt snowflake with a composite master key: one part customer managed key and one part account managed key by snowflake

## Column and row level security

### Column level security

- Dynamic data masking is applied dynamically at query run time
  - we apply the masking policy to the column of a table or view and it will mask the data for unauthorized users (masking can be partial)

```sql
-- create the policy
CREATE MASKING POLICY EMAIL_MASK AS (VAL STRING) RETURNS STRING ->
    CASE
        WHEN CURRENT_ROLE() IN ('SUPPORT') THEN VAL -- unmasked
        WHEN CURRENT_ROLE() IN ('ANALYST') THEN REGEXP_REPLACE(VAL, '.+\@', '****@') -- partial mask
        WHEN CURRENT_ROLE() IN ('HR') THEN SHA2(VAL) -- system functions
        WHEN CURRENT_ROLE() IN ('HR') THEN MY_MASK(VAL) -- UDF
        ELSE '******'
    END;

-- apply to the column
ALTER TABLE MYTABLE MODIFY COLUMN USER_EMAIL SET MASKING POLICY EMAIL_MASK
```

- Data masking policies are schema-level objects like tables and views
- Creating and applying them can be done independently of object owners
- Masking policies can be nested
- A masking policy is applied no matter where the column is referenced in a SQL statement. If you try to do a join on masked data, it will be performed on the masked data not the underlying data
- It can be added at object creation or after object is created

Masking policies can also use external tokenization. Unlike other form of data masking, data is directly stored tokenized in snowflake and will be detokenized at run time for authorized users by calling the external tokenization service.

### Row level security

- We can restrict which rows are returned in a query
- Row level security is applied transparently: if an unauthorized user executes a SELECT statement, restricted rows will be filtered

```sql
-- create the policy
-- here RAP_ID is the identifier
-- ACC_ID is the column we would like to attach the policy (serves as a sort of anchor)
-- output data type is always boolean
CREATE ROW ACCESS POLICY RAP_ID AS (ACC_ID VARCHAR) RETURNS BOOLEAN ->
    CASE
        WHEN CURRENT_ROLE() IN ('ADMIN') THEN TRUE -- return all rows
        ELSE FALSE -- return no rows
    END;

-- apply policy to the column
ALTER TABLE MYTABLE ADD ROW ACCESS POLICY RAP_ID ON (ACC_ID);
```

- Most of the remarks on column data masking apply
- NOTE: Adding a masking policy to a column fails if the column is referenced by a row access policy
- Row access policies are evaluated before data masking policies

### Worksheet example

```sql
-- initial context
USE ROLE SYSADMIN;
USE WAREHOUSE COMPUTE_WH;
-- Create new db
CREATE DATABASE SALES_DB;
CREATE SCHEMA SALES_SCHEMA;
-- switch context
USE DATABASE SALES_DB;
USE SCHEMA SALES_SCHEMA;

-- Create table with values
CREATE TABLE CUSTOMERS(
    ID NUMBER,
    NAME STRING,
    EMAIL STRING,
    COUNTRY_CODE STRING
);
INSERT INTO CUSTOMERS VALUES
(1357, 'Gab Prk', 'gabprk@gmail.com', 'FR'),
(8978, 'John Doe', 'jdoe@stanford.eu', 'US'),
(7997, 'Un Known', 'unkown@outlook.co.uk', 'IE');

-- grant usage to ANALYST to query the data
USE ROLE SECURITYADMIN;
GRANT USAGE ON DATABASE SALES_DB TO ROLE ANALYST;
GRANT USAGE ON SCHEMA SALES_DB.SALES_SCHEMA TO ROLE ANALYST;
GRANT SELECT ON TABLE SALES_DB.SALES_SCHEMA.CUSTOMERS TO ROLE ANALYST;

-- Create MASKING_ADMIN role
CREATE ROLE MASKING_ADMIN;
GRANT USAGE ON DATABASE SALES_DB TO ROLE MASKING_ADMIN;
GRANT USAGE ON SCHEMA SALES_DB.SALES_SCHEMA TO ROLE MASKING_ADMIN;
GRANT CREATE MASKING POLICY, CREATE ROW ACCESS POLICY ON SCHEMA SALES_DB.SALES_SCHEMA TO ROLE MASKING_ADMIN;

USE ROLE ACCOUNTADMIN;
GRANT APPLY MASKING POLICY, APPLY ROW ACCESS POLICY ON ACCOUNT TO ROLE MASKING_ADMIN; -- we need the account admin role for this one
GRANT ROLE MASKING_ADMIN TO USER ADMIN;

-- Create masking policy
USE ROLE MASKING_ADMIN;
USE SCHEMA SALES_DB.SALES_SCHEMA;

CREATE OR REPLACE MASKING POLICY EMAIL_MASK AS (VAL STRING) RETURNS STRING ->
    CASE
        WHEN CURRENT_ROLE() IN ('ANALYST') THEN VAL
        ELSE REGEXP_REPLACE(VAL, '.+\@', '****@')
    END;
-- apply masking policy
ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL SET MASKING POLICY EMAIL_MASK;

-- Check policy
USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- we see the data

USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- data is masked even though ANALYST role is GRANTED to SYSADMIN

-- Create simple row access policy
USE ROLE MASKING_ADMIN;
USE SCHEMA SALES_DB.SALES_SCHEMA;

CREATE OR REPLACE ROW ACCESS POLICY RAP AS (VAL VARCHAR) RETURNS BOOLEAN ->
    CASE
        WHEN 'ANALYST' = CURRENT_ROLE() THEN TRUE
        ELSE FALSE
    END;

-- following statement fails because column EMAIL is already masked
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY RAP ON (EMAIL);
-- ALTER TABLE CUSTOMERS MODIFY COLUMN EMAIL UNSET MASKING POLICY;

ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY RAP ON (NAME);

-- Verify policy
USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- we see the data

USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- we don't see any rows

-- Create row access policy with lookup
CREATE TABLE TITLE_COUNTRY_MAPPING (TITLE STRING, country_iso_code STRING);
INSERT INTO TITLE_COUNTRY_MAPPING VALUES ('ANALYST', 'FR');
GRANT SELECT ON TABLE TITLE_COUNTRY_MAPPING TO ROLE MASKING_ADMIN;

USE ROLE MASKING_ADMIN;
CREATE OR REPLACE ROW ACCESS POLICY CUSTOMER_POLICY AS (COUNTRY_CODE VARCHAR) RETURNS BOOLEAN ->
    'SYSADMIN' = CURRENT_ROLE() -- Sysadmin can view all roles
        OR EXISTS(
            SELECT 1 FROM TITLE_COUNTRY_MAPPING -- mapping table check
            WHERE TITLE = CURRENT_ROLE()
            AND country_iso_code = COUNTRY_CODE
        );

-- Fails because there is already a row access policy on the table
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY CUSTOMER_POLICY ON (COUNTRY_CODE);

ALTER TABLE CUSTOMERS DROP ALL ROW ACCESS POLICIES;
ALTER TABLE CUSTOMERS ADD ROW ACCESS POLICY CUSTOMER_POLICY ON (COUNTRY_CODE);

-- Verify policies
USE ROLE SYSADMIN;
SELECT * FROM CUSTOMERS; -- all rows, we still have data masking on EMAIL

USE ROLE ANALYST;
SELECT * FROM CUSTOMERS; -- only one row but no masking
```

## Account Usage and Information schema

- Snowflake provides a shared read only database called SNOWFLAKE, imported using a share object called ACCOUNT_USAGE
- By default, only accessible to ACCOUNTADMIN role
- 6 main schemas containing many views providing fine-grained usage metrics at the account object level
  - ACCOUNT_USAGE schema
    - views also contains dropped objects
  - CORE Schema
  - READER_ACCOUNT_USAGE schema contains views that display metadata and metrics for all the reader accounts
  - DATA_SHARING_USAGE: views for listings published in a data exchange
  - ORGANIZATION_USAGE: views containing infos for all accounts
  - INFORMATION_SCHEMA: not unique to snowflake, based on SQL-92 ANSI Information Schema
    - Views for all objects contained in db
    - Metadata for account level objects (such as roles, warehouses and databases)
    - Table functions displaying metadata for historical and usage data accros an account
    - Output of a view depends on the privileges of the user

ACCOUNT_USAGE and INFORMATION_SCHEMA are quite similar BUT:

- ACCOUNT_USAGE includes dropped objects
- INFORMATION_SCHEMA has no latency (while ACCOUNT_USAGE has between 45mins and 3 hours)
- Historical data is retained for 1 year in ACCOUNT_USAGE while it is kept from 7 days to 6 months for INFORMATION_SCHEMA

# Virtual Warehouse

## Overview

- A virtual warehouse is a named abstraction for a Massively Parallel Processing (MPP) compute cluster.
- Virtual warehouses constitute the compute layer
- Virtual warehouses execute:
  - DQL operations (SELECT)
  - DML operations (UPDATE)
  - Data loading operations (COPY INTO)
- As a user, we only interact with the named warehouse, not the individual nodes
- We can spin up an unlimited nb of warehouses and config can be changed on the fly
- Virtual warehouses contain local SSD storage to store raw data retrieved from the storage layer (warehouse cache, which is cleared on suspend)

```sql
-- creation
CREATE WAREHOUSE MY_WH WAREHOUSE_SIZE='MEDIUM';
-- deletion
DROP WAREHOUSE MY_WH;
-- update params
CREATE WAREHOUSE MY_WH WAREHOUSE_SIZE='SMALL';

-- get information on warehouses, such as nb of running and queued requests
SHOW WAREHOUSES;

```

## States

- States:
  - STARTED, it consumes credits
  - SUSPENDED, it does not consume credit
  - RESIZING
- By default, on creation it is in started state

```sql
-- Suspend
ALTER WAREHOUSE MY_WH SUSPEND; -- cache is cleared
-- RESUME
ALTER WAREHOUSE MY_WH RESUME;

-- Auto suspend after 5 mins (default is 10 min)
-- Note that suspend process runs one per minute and min billing is 1 min so it makes
-- no sense to use an AUTO_SUSPEND < 60
CREATE WAREHOUSE MY_WH AUTO_SUSPEND=300;

-- Auto resume = default behavior, WH restarts on query
CREATE WAREHOUSE MY_WH AUTO_RESUME=TRUE;

-- Initially suspended (default is in started state)
CREATE WAREHOUSE MY_WH INITIALLY_SUSPENDED=TRUE;
```

## Size

- To choose a size, snowflake recommends experimenting with a representative query of a workload
  - Data loading does not typically require large virtual warehouses and sizing up does not guarantee increased data loading performace.
  - Also see billing

## Resource monitors

- Can only be created by account administrators by default
- Objects allowing users to set credit limits on user managed warehouses
- Can either be set at the account level or individual warehouse level
- Limits can be set for a specified interval or date range
- When limits are reached, an action such as notify user or suspend warehouse can be triggered

```sql
-- Creation
CREATE RESOURCE MONITOR MYRM
WITH CREDIT_QUOTA=100 -- credits allocated per frequency interval
FREQUENCY=MONTHLY
START_TIMESTAMP='2023-01-01 00:00 GMT' -- if not specified, starts immediately
TRIGGERS ON 50 PERCENT DO NOTIFY -- alert notification to account admins
		 ON 75 PERCENT DO NOTIFY
		 ON 95 PERCENT DO SUSPEND -- suspend after running queries are complete
		 ON 100 PERCENT DO SUSPEND_IMMEDIATE; -- hard stop

-- Apply at the account level
ALTER ACCOUNT SET RESOURCE_MONITOR=MYRM;
-- Apply at the warehouse level
ALTER WAREHOUSE MYWH SET RESOURCE_MONITOR=MYRM;
```

- To view consumption, you can also use SNOWFLAKE database:

```sql
-- context
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;

-- all information on billing
SELECT * FROM WAREHOUSE_METERING_HISTORY WHERE WAREHOUSE_NAME = 'COMPUTE_WH';

-- Total credits grouped by warehouse
SELECT WAREHOUSE_NAME, SUM(CREDITS_USED) AS TOTAL_CREDITS_USED
FROM WAREHOUSE_METERING_HISTORY
GROUP BY WAREHOUSE_NAME;

-- Using the information schema, get last seven days with a table function
SELECT *
FROM TABLE(INFORMATION_SCHEMA.WAREHOUSE_METERING_HISTORY(dateadd('days', -7, current_date())));
```

## Multi cluster warehouses

- scale up: increase warehouse size, can only be done manually
- scale out: add clusters: A multi cluster warehouse is a named group of virtual warehouses which can automatically scale in and out based on the nb of concurrent users/queries

```sql
-- dynamically scale
CREATE WAREHOUSE MY_WH
MIN_CLUSTER_COUNT = 1 -- lower bound
MAX_CLUSTER_COUNT = 3 -- upper bound
SCALING_POLICY=STANDARD;
```

If MIN_CLUSTER_COUNT == MAX_CLUSTER_COUNT > 1, we are in MAXIMIZED mode. If nb are different, we are in AUTOSCALED mode

Scaling policy:

- STANDARD - maximize perf:
  - Scale out: When a query is queued, a new warehouse is added automatically
  - Scale in: Every min, a background process will check if the load on the least busy warehouse can be redistributed to another warehouse. If condition is met after 2 consecutive checks, the warehouse is marked for shutdown
- ECONOMY:

  - Scale out: only add a warehouse if there's enough query load to keep a new warehouse busy for 6 min
  - Scale in: same as STANDARD, but we only mark for shutdown after 6 minutes

- Billing: each cluster consumes credit depending on the size of the warehouse

```sql
CREATE WAREHOUSE MYWH
-- nb of concurrent SQL statements that can be executed against a warehouse
-- before queuing / providing additional compute power
MAX_CONCURRENCY_LEVEL = 6
-- time a SQL statement can be queued before it is aborted (default is no timeout)
STATEMENT_QUEUED_TIMEOUT_IN_SECONDS=60;
-- time after which any running SQL statement is aborted
STATEMENT_TIMEOUT_IN_SECONDS=600;
```

# Query optimization

## Query performance analysis tool

- Activity > Query history tab displays query history for the last 14 days
- Users can view other users queries but not view their results
- Click on a query to have a breakdown of the actions that snowflake actually did. Each box is an operation and the line represents relationships between blocks. You can also see the nb of rows you get at each step and the % of the total time it took. Click on a box to have statistics and a profile overview
- You can also check the query plan by running `EXPLAIN <MY QUERY>;`

NOTE: on the statistics you can see spilling information (nb of bytes spilled to local storage and spilled to remote storage): if there is no problem with the query, it may indicate that the warehouse in memory and then disk storage is too small

```sql
--- Get 10 longest running queries
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;
SELECT
QUERY_ID
,QUERY_TEXT
,USER_NAME
,ROLE_NAME
,EXECUTION_STATUS
,ROUND(TOTAL_ELAPSED_TIME / 1000, 2) AS TOTAL_ELAPSED_TIME_SEC
FROM QUERY_HISTORY
WHERE TOTAL_ELAPSED_TIME_SEC > 3
ORDER BY TOTAL_ELAPSED_TIME_SEC
LIMIT 10;

-- Note instead of ACCOUNT_USAGE, we can look at the info in the
-- information schema
FROM TABLE(INFORMATION_SCHEMA.query_history())
```

## SQL Tuning

Order of execution:

- ROWS: FROM, JOIN, WHERE
- GROUPS: GROUP BY, HAVING
- RESULT: SELECT, DISTINCT, ORDER BY, LIMIT

- So, for ex, LIMIT is always executed after GROUP BY, which means that it won't accelerate our group by
- The reason we do row operations first is to decrease the nb of data for more expensive operations
- GROUP BY work best with low cardinality groups

## Caching

- Caching can occur at the services layer with metadata and results cache and at the warehouse level (local disk cache)

- Metadata cache refers to the store of statistics on the database objects (services layer).

  - This makes it so that some queries can be completed without requiring a running virtual warehouse
  - `SELECT COUNT(*)`, system functions, `DESCRIBE`, `SHOW` use this cache

- Results cache stores results of a query for reuse (services layer)

  - Also referred as the 24 hour cache, the query results cache...
  - Stores result of a query for 24 hour but each time it is reused, it is extended by 24 hours up until 31 days after which result is purged
  - For the cache to be used
    - new query must be identical to cached query (even adding a LIMIT bypasses the cache)
    - the underlying table data must NOT have changed
    - The same role (not user) is used as in the previous query
    - Time context functions such as `CURRENT_TIME()` must NOT be part of the query
  - Results reuse can also be disabled by using the session parameter `USE_CACHED_RESULT`

- Local disk cache (query processing layer) refers to the local ssd storage for the node on the warehouse cluster

  - Also referred as ssd cache, data cache, raw data cache
  - Improves perfs of queries in the same warehouse
  - [[LRU Cache|LRU policy]] is used
  - Purged when virtual warehouse is resized, suspended or dropped
  - Unlike results cache, can be used partially, retrieving the rest of the data required for a query from remote storage

- When a query can not use metadata cache, results cache and data cache, it retrieves the data from the storage layer

```sql
-- Context
USE ROLE ACCOUNTADMIN;
USE SCHEMA SNOWFLAKE_SAMPLE_DATA.TPCH_SF1000;

SHOW WAREHOUSES;
-- Stop the current warehouse
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
ALTER WAREHOUSE SET AUTO_RESUME=FALSE;

-- Meta data cache --
-- Proof that it uses the metadata cache can be seen if you go to the query profile
SELECT COUNT(*) FROM CUSTOMER; -- returns the count
SELECT CURRENT_USER(); -- context function
DESCRIBE TABLE CUSTOMER; -- shows the column details
SHOW TABLES; -- lists objects
SELECT SYSTEM$CLUSTERING_INFORMATION('LINEITEM', ('L_ORDERKEY')); -- system function

-- Results cache --
SELECT * FROM CUSTOMER LIMIT 1000; -- fails because warehouse is suspended
ALTER WAREHOUSE COMPUTE_WH RESUME;
SELECT * FROM CUSTOMER LIMIT 1000; -- query uses warehouse
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
SELECT * FROM CUSTOMER LIMIT 1000; -- This time it works, if you go to query profile, you will see QUERY RESULT REUSE
ALTER WAREHOUSE COMPUTE_WH RESUME;
SELECT * FROM CUSTOMER LIMIT 1000; -- even though warehouse is resumed, query uses the results cache
SELECT * FROM CUSTOMER LIMIT 100; -- limit was changed, we don't use the results cache

-- Local storage cache --

-- prevent use of results cache.
-- Alternatively, we can change the LIMIT parameter in the following queries
-- to make sure results cache is not used
ALTER ACCOUNT SET USE_CACHED_RESULT = FALSE;

-- Here in the query profile overview, Remote Disk I/O took the most time
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;

-- Query is a lot faster and in the query profile overview, there is no Remote Disk I/O
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;

-- We add other columns: We still use Remote Disk I/O for these columns but we use local storage for O_ORDERKEY
SELECT O_ORDERKEY, O_SHIPPRIORITY, O_COMMENT
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;


ALTER WAREHOUSE COMPUTE_WH SUSPEND;
ALTER WAREHOUSE COMPUTE_WH RESUME;
-- Cache was cleared when restarting the warehouse so query takes a lot of time
SELECT O_ORDERKEY, O_CUSTKEY, O_ORDERSTATUS, O_TOTALPRICE, O_ORDERDATE
FROM ORDERS
WHERE O_ORDERDATE > DATE('1997-09-19')
ORDER BY O_ORDERDATE
LIMIT 1000;
```

## Clustering

- Snowflake stores data in micro partitions. If data is correctly clustered, it can help with micro-partition pruning and make queries along a clustered dimension faster. On the other hand, if we try to get result along a dimension that is poorly clustered, we have to retrieve all micro-partitions
- table data is partitioned and stored in the order it was loaded. If the data was ordered prior to ingestion, it will be naturally clustered along the order dimension
- Snowflake maintains clustering metadata for the micro-partitions of a given table:
  - Total nb of micro-partitions
  - Nb of overlapping micro-partitions for a table column
  - Depth of overlapping micro-partitions = nb of micro-partitions I would have to look into to find one value

```sql
-- Create a table with cluster keys
CREATE TABLE T1 (C1 date, C2 string, C3 number) CLUSTER BY (MONTH(C1), C2)

-- ALTER a table to add cluster keys
ALTER TABLE T1 (C1 date, C2 string, C3 number) CLUSTER BY (C3)

-- Suspend reclustering
ALTER TABLE T1 SUSPEND RECLUSTER;
-- Resume reclustering
ALTER TABLE T1 RESUME RECLUSTER;

```

```text
Depth vs overlap

___          ____     __
___        ____           __
___     ____                  __

O: 3        O: 3      O: 0
D: 3        D: 2      D: 1

In the final case, we are in constant state
```

```sql
-- get clustering depth
SELECT SYSTEM$clustering_depth('TABLE', '(col1,col3)');

-- get all infos (see result below)
SELECT SYSTEM$clustering_information('TABLE', '(col1,col3)');
```

```json
{
"cluster_by_keys" : "LINEAR(col1, col3)",
"notes" : "...",
// total nb of micropartitions in the table
"total_partition_count" : XX,
// how many partitions have exclusively the key combination, the higher the better
"total_constant_partition_count" : XX,
// measures quality of clustering, the lower the better for overlap and depth
"average_overlaps" : XX,
"average_depth" : XX,
// distribution of the depth among the micropartitions
// Each depth is associated with the number of values that have said depth
// Ideally, we would want a high number for low depths
"partition_depth_histogram" : {
	"00000" : XX,
	"00001" : XX
	...
	}
}
```

- Clustering can degrade, particularly if the table is large and there are a lot of DML operations
- Snowflake support automatic clustering by designating one or more table columns as clustering keys (we can also define them on materialized views): the goal is to colocate date of the clustering key in the same micro-partitions
- Automatic clustering is a serverless feature and uses compute and storage cost
- Clustering should be reserved for large tables that in the multi terabyte range where typical queries filter a large part of the data
- Clustering is better for tables that are not modified often
- Snowflake recommends a maximum of 3 or 4 columns with a cardinality that is not too low and not too high and with the lowest cardinality selected first

Example below

```sql
-- context
USE ROLE SYSADMIN;
USE DATABASE SNOWFLAKE_SAMPLE_DATA;
USE SCHEMA TPCDS_SF100TCL;


-- check cluster_by column to see if the table has a cluster key (will be empty if there is no cluster)
SHOW TABLES;
SELECT "name", "database_name", "schema_name", "cluster_by" FROM TABLE(result_scan(last_query_id()));


-- shows how the table is clustered according to the CLUSTERED keys
-- will throw an error if table is not clustered
SELECT SYSTEM$clustering_information('CATALOG_SALES');

-- shows how the table is clustered according to the SPECIFIED key(s)
SELECT SYSTEM$clustering_information('CATALOG_SALES', '(cs_sold_date_sk)');

-- very fast retrieval when using a good clustered key
-- query profile shows that we only scanned 4 micropartitions
SELECT CS_SOLD_DATE_SK, CS_SOLD_TIME_SK, CS_ITEM_SK, CS_ORDER_NUMBER FROM CATALOG_SALES
WHERE CS_SOLD_DATE_SK = 2451092 AND CS_ITEM_SK = 140947;

-- Monitor automatic clustering
USE ROLE ACCOUNTADMIN;
USE DATABASE SNOWFLAKE;
USE SCHEMA ACCOUNT_USAGE;
SELECT START_TIME, END_TIME, CREDITS_USED, NUM_BYTES_RECLUSTERED, TABLE_NAME, SCHEMA_NAME, DATABASE_NAME
FROM AUTOMATIC_CLUSTERING_HISTORY;
```

## Search optimization service

- Snowflake is best suited for analytics workload
- However, there are scenarios where a user might need to look up an individual value in a very large table
- The Search Optimization Service is a table level property aimed at improving the performance of selective point lookup queries, typically returning 1 or a small nb of rows: `WHERE XX = 'YY'` or `WHERE XX IN ('YY', 'ZZ')`
- A background process creates and maintains a search access path recording metadata on the table
- Selective lookup queries will use these metadata to find data faster than the usual pruning mechanism
- Computes storage and compute resources

```sql

-- implementation is easy as it is a table level property
ALTER TABLE MYTABLE ADD SEARCH OPTIMIZATION;

-- remove it
ALTER TABLE MYTABLE DROP SEARCH OPTIMIZATION;

-- check search optimization column
SHOW TABLES;
SELECT "search_optimization", -- wheter service is enabled
"search_optimization_progress", -- percentage of table optimized so far
"search_optimization_bytes"
FROM TABLE(result_scan(last_query_id()));
```
