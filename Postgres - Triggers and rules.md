---
id: Postgres - Triggers and rules
aliases: []
tags: []
---
, #postgres

### OLD and NEW variables

- `OLD` = state of the row in the table **before** the event
- `NEW` = state of the row in the table **after** the event

### Triggers vs Rules

- Rules can be used to perform actions such as query redirection, query modification, or the execution of additional queries.
  Rules are part of the query rewrite system. When a query is issued, the rules are applied first, and they can transform the query into another query or multiple queries. This happens before the query planner and executor get involved.
- Triggers are used to execute procedural code (typically written in PL/pgSQL) automatically in response to certain events on a particular table.

- Use rules when you need to transform or rewrite queries transparently. Use triggers when you need to execute procedural logic in response to data changes.

- If both are present, rules are executed first as they rewrite the query itself
- Both rules and triggers in PostgreSQL operate within the context of a transaction.

### Rules example

Rules are defined at a table/view level. You can inspect them with `\d mytable`

```sql
CREATE TABLE tags(
 pk integer NOT NULL PRIMARY KEY,
 tag TEXT,
 parent integer
);
CREATE TABLE O_tags
(LIKE tags INCLUDING ALL);
CREATE TABLE F_tags
(LIKE tags INCLUDING ALL);

-- If tag starts with O, also insert
-- it in O_tags table
CREATE OR REPLACE RULE r_tags1
AS ON INSERT TO tags
WHERE NEW.tag ILIKE 'O%' DO ALSO
INSERT INTO O_tags(pk, tag, parent)
VALUES (NEW.pk, NEW.tag, NEW.parent);

-- If tag starts with F, insert it
-- in F_tags table instead
CREATE OR REPLACE RULE r_tags2
AS ON INSERT TO tags
WHERE NEW.tag ILIKE 'F%' DO INSTEAD
INSERT INTO F_tags(pk, tag, parent)
VALUES (NEW.pk, NEW.tag, NEW.parent);

-- If tag starts with R, don't insert
CREATE OR REPLACE RULE r_tags3
AS ON INSERT TO tags
WHERE NEW.tag ILIKE 'R%'
DO INSTEAD NOTHING;

-- OpenBSD present in tags and O_tags
INSERT INTO tags(tag)
VALUES ('OpenBSD'); -- INSERT 1 0

-- Fedora only present in F_tags
INSERT INTO tags(tag) VALUES
  ('Fedora'); -- INSERT 0 0

-- Red Hat not present
INSERT INTO tags(tag) VALUES
  ('Red Hat'); -- INSERT 0 0
```

The number of Inserted lines return only correspond to the number of lines in the tags table
If we look at the explain plan: `EXPLAIN INSER ...`

```txt
QUERY PLAN                                        |
--------------------------------------------------+
Insert on tags  (cost=0.00..0.01 rows=0 width=0)  |
  ->  Result  (cost=0.00..0.01 rows=1 width=40)   |
                                                  |
Insert on o_tags  (cost=0.00..0.01 rows=0 width=0)|
  ->  Result  (cost=0.00..0.01 rows=1 width=40)   |
        One-Time Filter: false                    |
                                                  |
Insert on f_tags  (cost=0.00..0.01 rows=0 width=0)|
  ->  Result  (cost=0.00..0.01 rows=1 width=40)   |
        One-Time Filter: false                    |
```

- tags: filter present if value starts with R or F
- o_tags: filter present if value does not start with O
- f_tags: filter present if value does not start with F

The rules modified the query plan directly

NOTE: because pk is a primary key, the value on O_tags will be the value on tags + 1.

```sql

-- delete rule: also delete on O_tags
CREATE OR REPLACE RULE r_tags4
AS ON DELETE TO tags
WHERE OLD.tag ILIKE 'O%' DO ALSO
DELETE FROM O_tags WHERE tag = OLD.tag
;

-- update rule: two rules
CREATE OR REPLACE RULE r_tags5
AS ON UPDATE TO tags
WHERE OLD.tag ILIKE 'O%' DO ALSO
DELETE FROM O_tags WHERE tag = OLD.tag
;
CREATE OR REPLACE RULE r_tags6
AS ON UPDATE TO tags
WHERE NEW.tag ILIKE 'O%' DO ALSO
INSERT INTO O_tags(pk, tag, parent)
VALUES (NEW.pk, NEW.tag, NEW.parent);
```

To make a better update rule, we could instead create a [[Postgres - Functions|function]].

```sql
DROP RULE r_tags5 ON tags;
DROP RULE r_tags6 ON tags;

CREATE OR REPLACE FUNCTION move_record(
old_record_tag text,
new_record_pk INTEGER,
new_record_tag TEXT,
new_record_parent INTEGER
) RETURNS VOID AS $$
BEGIN
    IF old_record_tag ILIKE 'O%' THEN
        DELETE FROM O_tags WHERE tag = old_record_tag;
    END IF;
   IF new_record_tag ILIKE 'O%' THEN
        INSERT INTO O_tags(pk, tag, parent) VALUES
       (new_record_pk, new_record_tag, new_record_parent);
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE RULE r_tags8
AS ON UPDATE TO tags DO ALSO
SELECT move_record(OLD.tag, NEW.pk, NEW.tag, NEW.parent);
```

This is very cumbersome. For this type of functionality, triggers are better as they can allow to execute
procedural code by referencing the OLD and NEW record.

### Triggers

```sql
-- standard way to use a trigger
CREATE OR REPLACE FUNCTION myfunc
RETURNS TRIGGER AS ...

CREATE TRIGGER mytrigger
BEFORE INSERT ON mytable
FOR EACH STATEMENT
EXECUTE PROCEDURE myfunc
```

- Function returns a trigger.
- Trigger is associated to a table and uses the function
- Can be executed at different moments: `BEFORE`, `AFTER`, `INSTEAD OF`
- `FOR EACH ROW`: executed for each row involved in the operation VS `FOR EACH STATEMENT`: executed only once for each SQL statement satisfying the condition

```sql
CREATE TABLE new_tags(
  pk integer NOT NULL,
  tag TEXT,
  parent integer
);
CREATE TABLE new_O_tags(
  pk integer NOT NULL,
  tag TEXT,
  parent integer
);

-- first create a function that
-- returns a trigger.
-- it has access to OLD and NEW
CREATE OR REPLACE FUNCTION f_tags()
RETURNS TRIGGER AS
$$
BEGIN
-- check that first letter is o
IF LOWER(substring(NEW.tag FROM 1 FOR 1)) = 'o'
THEN
INSERT INTO new_O_tags(pk,tag,parent)
VALUES (NEW.pk, NEW.tag, NEW.parent);
END IF;
-- Returns the new record so that it is also inserted
-- in the original table
RETURN NEW;
END
$$ LANGUAGE plpgsql;

-- then create the trigger itself
CREATE TRIGGER t_tags BEFORE INSERT ON new_tags
FOR EACH ROW
EXECUTE PROCEDURE f_tags();
```

Trigger is visible when doing `\d new_tags`

However, contrary to rules, doing an `EXPLAIN INSERT...` only shows

```txt
QUERY PLAN                                          |
----------------------------------------------------+
Insert on new_tags  (cost=0.00..0.01 rows=0 width=0)|
  ->  Result  (cost=0.00..0.01 rows=1 width=40)     |
```

- With triggers, you can use the `TG_OP` variable to know
  which type of operation is going on. This helps creating more complex logic.
  For ex, we could add logic for insert, delete and update in a single function

```sql
CREATE OR REPLACE FUNCTION f_tags()
RETURNS TRIGGER AS
$$
BEGIN
-- check that first letter is o
IF TG_OP = 'INSERT' THEN
  IF LOWER(substring(NEW.tag FROM 1 FOR 1)) = 'o'
  THEN
  INSERT INTO new_O_tags(pk,tag,parent)
  VALUES (NEW.pk, NEW.tag, NEW.parent);
  END IF;
  RETURN NEW;
ELSEIF TG_OP = 'DELETE' THEN
  IF LOWER(substring(OLD.tag FROM 1 FOR 1)) = 'o'
  THEN
  DELETE FROM new_O_tags
  WHERE tag = OLD.tag;
  END IF;
  RETURN OLD;
END IF;
-- Default return, should not reach
RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

### Event triggers

Event triggers fire on DDL statements: they react to events that changes the data structure rather than the data content.
They are not related to a particular table but rather to DDL commands.
To create them:

- we need a function that returns an EVENT TRIGGER.

```sql
CREATE OR REPLACE FUNCTION myevfunc()
RETURUNS EVENT_TRIGGER AS ...
```

- we then need to create the event trigger itself.

```sql
CREATE EVENT_TRIGGER myevtrigg ON
ddl_command_end EXECUTE
FUNCTION myevfunc()
```

To perform introspection, within the function that returns an event trigger, we can use:

- `pg_event_trigger_ddl_commands()`, which returns every command that was executed during the DDL statement
- `pg_event_trigger_dropped_objects()` returns objects that were dropped during the event

Contrary to normal triggers, after the `ON`, we can only use:

- `ddl_command_start`
- `ddl_command_end`
- `table_rewrite`
- `sql_drop`

