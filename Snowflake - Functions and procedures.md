#snowflake

see [[Snowflake - Objects]]


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
		var result_set = snowflake.execute({sqlText: sql_command}); -- use snowflake js API to execute sql
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