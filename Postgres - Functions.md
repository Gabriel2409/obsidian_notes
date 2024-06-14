---
id: Postgres - Functions
aliases: []
tags: []
---

, #postgres

```sql
CREATE FUNCTION function_name(p1 type, p2 type, ..., pn type)
RETURN type AS
BEGIN
  -- function logic
END;
LANGUAGE language_name
```

### Function types

NOTE: when dropping a function, specify the parameters types
`DROP FUNCTION myfunc(integer, integer)`

#### Basic functions (returning a value)

```sql

-- old way, with positional parameters
CREATE OR REPLACE FUNCTION
my_sum(integer, integer)
RETURNS integer AS $$
SELECT $1 + $2;
$$ LANGUAGE SQL;

-- new way, with named parameters
CREATE OR REPLACE FUNCTION
my_sum(x integer, y integer)
RETURNS integer AS $$
SELECT x + y;
$$ LANGUAGE SQL;

SELECT my_sum(1,2); -- returns 3
```

#### Functions returning a set of elements

```sql
CREATE TABLE posts(
pk int GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
title VARCHAR(255),
txt VARCHAR(255)
);

INSERT INTO posts (title, txt) VALUES
('Test', 'test'), ('Test', 'newtest'), ('Youpi', 'cool');

CREATE OR REPLACE FUNCTION delete_posts(p_title varchar(255))
RETURNS SETOF integer AS $$
DELETE FROM posts WHERE title=p_title RETURNING pk;
$$ LANGUAGE SQL;

SELECT delete_posts('Test');
```

```txt
delete_posts|
------------+
           1|
           2|
```

And the posts are deleted from the table
Note: `SETOF` simply defines a result set of a data type.
If we remove the `SETOF`, it only returns the first deleted row but still deletes the others.
`SETOF` does not remove duplicates

#### Polymorphic functions

- Useful when we need a function that works with different data types

```sql
CREATE OR REPLACE FUNCTION nvl(anyelement, anyelement)
RETURNS anyelement AS $$
SELECT COALESCE ($1,$2) $$ LANGUAGE SQL;

SELECT nvl(NULL, 1);
SELECT nvl(NULL, 'a'::text);
```

#### PL/pgSQL functions

- can be used to create functions and trigger procedures
- add new control structures
- add new data types
- Very similar to Oracle PL/SQL and supports variable declarations, expressions, conditional and loop structures, cursors

```sql
-- basic function
CREATE OR REPLACE FUNCTION my_sum(x integer, y integer)
RETURNS INTEGER AS
$BODY$ -- we can put any string between the $ but it has to match at beginning and end
DECLARE
ret integer;
BEGIN
  ret:= x + y;
  RETURN ret;
END
$BODY$
LANGUAGE 'plpgsql';
```

Note: we can use aliases. For ex, if we do the same function without naming the parameters,
in the DECLARE block we can do:

```sql
DECLARE
x alias for $1;
y alias for $2;
ret integer;
```

Instead of returning parameters in the `BEGIN` block, we can define `IN/OUT` parameters

```sql
-- basic function
CREATE OR REPLACE FUNCTION my_sum(IN x integer, IN y integer, OUT z integer) AS
$BODY$
BEGIN
  z:= x + y;
END
$BODY$
LANGUAGE 'plpgsql';
```

We can return multiple values:
`CREATE OR REPLACE FUNCTION my_ops(IN x integer, IN y integer, OUT z integer, OUT w integer) AS ...`

`SELECT my_ops(6,8)` will return a tuple

```txt
my_ops |
-------+
(14,48)|
```

However, we can select each individual component:
`SELECT * FROM my_ops(6,8)`:

```txt
z |w |
--+--+
14|48|
```

We can also define `INOUT` parameters

### Function volatility

- By default a function is `VOLATILE`, which can do everything including modifying the db.
  It is reevaluated at every row where a value is needed
- A `STABLE` function cannot modify the db and returns the same results given the same arguments
  for all rows within a single statement.
  This allows the optimizer to optimize multiple calls to this function. In particular, it is
  safe to use it in an index scan condition
- An `IMMUTABLE` function cannot modify the db and is guaranteed to return the same results
  given the same arguments forever.

For best optimization results, you should label your functions with the strictest volatility category that is valid for them.

- `now()` is stable: same result within the same transaction
- `lower` is immutable: given the same arg, it always returns the lowercase version

### Control structure

```sql
-- if statements
CREATE OR REPLACE FUNCTION above_0(IN x integer, OUT y varchar) AS
$BODY$
BEGIN
  IF x > 0 THEN   y:='positive';
  ELSEIF x = 0 THEN y:='zero';
  ELSE y:='negative';
  END IF;
END
$BODY$
LANGUAGE 'plpgsql';

-- case statement, match a var
CREATE OR REPLACE FUNCTION case_int(IN x integer) RETURNS VARCHAR AS
$BODY$
BEGIN
  CASE x
  WHEN 0 THEN   RETURN 'zero';
  WHEN 1 THEN   RETURN 'one';
  WHEN 2 THEN   RETURN 'two';
  ELSE RETURN 'unknown';
  END CASE;
END
$BODY$
LANGUAGE 'plpgsql';

-- case statement, search expr
CREATE OR REPLACE FUNCTION case_int(IN x integer) RETURNS VARCHAR AS
$BODY$
BEGIN
  CASE
  WHEN x > 0 THEN   RETURN 'positive';
  ELSE RETURN 'notpositive';
  END CASE;
END
$BODY$
LANGUAGE 'plpgsql';
```

### Loops

```sql
CREATE EXTENSION hstore;

CREATE TYPE my_ret_type AS (
  id integer,
  title TEXT,
  record_data hstore
);

CREATE OR REPLACE FUNCTION myfun (p_id INTEGER)
RETURNS SETOF my_ret_type AS
$$
DECLARE
rw posts%ROWTYPE;
ret my_ret_type;
BEGIN
 FOR rw IN SELECT * FROM posts
 WHERE pk = p_id
 LOOP
  ret.id := rw.pk;
  ret.title := rw.title;
  ret.record_data := hstore(ARRAY['title', rw.title, 'Title and Content', FORMAT('%s %s', rw.title, rw."content")]);
  RETURN NEXT ret;
 END LOOP;
RETURN;
END
$$ LANGUAGE 'plpgsql';
```

- `rw posts%ROWTYPE;`: rw is a row type == a container of a single row of the posts table.
  NOTE: we could have used a record type instead, which is a generalized version that can
  be associated to any record of any table: `rw record;`
- `LOOP`: with this statement, we iterate over the result of the selection and assign each
  row to the rw variable.
- next 3 steps: assign the ret variable
- `RETURN next ret`: returns the value of `ret` and go to next loop iteration
- `END LOOP`: ends the loop
- final `RETURN` has no arg because we already returned the set of `ret`

Note: PL/pgSQL language is inside the transaction system: functions are executed atomically
and the results are returned only at the final RETURN statement.

### Exception handling

```sql
CREATE OR REPLACE FUNCTION unsafe_div(IN x REAL, IN y REAL, OUT z REAL) AS
$BODY$
BEGIN
 z:= x/y;
END
$BODY$
LANGUAGE 'plpgsql'

SELECT unsafe_div(5,0) -- returns ERROR: division by zero

CREATE OR REPLACE FUNCTION safe_div(IN x REAL, IN y REAL, OUT z REAL) AS
$BODY$
BEGIN
  z:= x/y;
EXCEPTION
  WHEN division_by_zero THEN
  RAISE INFO 'DIVISION BY ZERO';
  RAISE INFO 'Error % %', SQLSTATE, SQLERRM;
  z:= 0;
END
$BODY$
LANGUAGE 'plpgsql'

SELECT safe_div(5,0)
-- Returns 0
-- also shows a message:
-- DIVISION BY ZERO
-- Error 22012 division by zero
```

### Security definer

Allows to execute a function as if we were its owner.
This avoids `<insufficient privilege>` results

As a super user:

```sql
CREATE FUNCTION forum.my_stat_activity()
RETURNS TABLE (pid integer, query text)
AS $$
  SELECT pid, query FROM pg_stat_activity;
$$ LANGUAGE 'sql'
SECURITY DEFINER;

GRANT EXECUTE ON FUNCTION forum.my_stat_activity TO forum;
```
