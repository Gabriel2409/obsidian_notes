---
id: Postgres - Tables
aliases: []
tags: []
---

: #postgres

3 types of tables:

- **temporary**: very fast tables visible only by user who created them (`CREATE TEMP TABLE`)
- **unlogged**: very fast tables to be used as support tables common to all users (`CREATE UNLOGGED TABLE`)
- **logged**: regular tables (`CREATE TABLE`)

### table creation

```sql
CREATE TABLE myusers(
pk int GENERATED ALWAYS AS IDENTITY
, username text NOT NULL
, gecos text
, PRIMARY KEY (pk)
, UNIQUE (username)
);
```

Note: `GENERATED AS IDENTITY` IS SQL standard compliant while `SERIAL` is postgres specific. The `GENERATED` syntax is also more flexible.

- `ALWAYS`: errors out if we try to set the value manually
- `BY DEFAULT` allows to specify the value for insert and update and will only generate it if necessary

Running `\d myusers` Shows column types and indexes. Primary keys are implemented as unique index (no duplicate value)



### copy table structure
```sql
-- copies the column types
CREATE TABLE t2 AS
SELECT * FROM t1 LIMIT 0;

-- also includes primary keys and constraints
CREATE TABLE t2 
(LIKE t1 INCLUDING ALL)
```
### special keywords

Similarly to dbs, we can use the `EXISTS` keyword

By adding the `TEMP` keyword, we can have a table accessible only during the session.

If we want to have it accessible only during a [[Transaction]]:

```sql
BEGIN; -- WORK | TRANSACTION keyword is optional
CREATE TEMP TABLE tmptbl(
...
) ON COMMIT DROP;
-- do something
COMMIT;
-- tmptbl is no longer accessible
```

By adding the `UNLOGGED` keyword, we can have a very fast table but it is not crash safe (table will be empty after a crash)

### Relation with file system

```sql
SELECT oid, relname
FROM pg_class
WHERE relname = 'mydb';
```

Now in PGDATA dir, in `base/<mydb_oid>`, you should find a file whose name is the oid.
Each table is stored as one or more file. If the table size is more than **1 GB**, it will be stored in several files, for ex `16389`, then `16389.1`, then `16389.2`, etc.

```bash
oid2name -d <mydb> -f <table_oid>
# should show the table
```

NOTE: index are also stored as files.
For ex with a table `users` that have a primary key and a unique constraint on username

```sql
SELECT oid, relname
FROM pg_class
WHERE relname LIKE 'users%';
```

Result shows

```text
oid   | relname
----------------
16399 | users
16398 | users_pk_seq
16404 | users_pkey
16406 | users_username_pkey
```

