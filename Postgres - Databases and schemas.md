---
id: Postgres - Databases and schemas
aliases: []
tags: []
---

# postgres

- **DDL** (Data Definition Language): Commands used to manage databases
- **DML** (Data Manipulation Language): Commands used to insert, delete, update and select

Connect with [[Postgres - psql|psql]],

```sql
-- makes a physical copy of template1 db
CREATE DATABASE mydb;
```

Under the hood, creating a db makes a copy of template1, so if objects were added to template1, they will be present when creating a new db.
To use another db as template:

```sql
CREATE DATABASE db2 template mydb
```

- the exists keyword is useful when scripting. Instead of erroring, dbs that don't match the condition will be skipped

```sql
CREATE DATABASE IF NOT EXISTS mydb
DROP DATABASE IF EXISTS mydb
```

Databases are organized into namespaces, called schemas. Schemas can not be nested (flat namespace)

Starting Postgres15, a normal user will not be able to execute DDL on the public schema and not perform DML on the public schema unless they receive permission from a superuser.

```sql
CREATE ROLE myuser WITH PASSWORD '1234' LOGIN;
SET ROLE TO myuser;
CREATE TABLE mytable(id integer);
-- ERROR: permission denied for public schema
```

- `search_path` is the internal postgres variable to find tables.
  By default it is `$user,public`, which means, if we don't specify the schema, it first looks for a table in the schema corresponding to the role name and then public

```sql
-- after connecting as superuser
CREATE SCHEMA myuser AUTHORIZATION myuser
SET ROLE TO myuser;
CREATE TABLE mytable(id integer);
-- it works
\dt
-- now shows the mytable table
```

- Find the size of a db

```sql
\x -- for extended display
\l+ mydb -- then look for Size field

-- or with SQL
SELECT pg_database_size('mydb')
```

### Relation with file system

In [[Postgres - theory]], we saw that all postgres data is stored in the PGDATA dir.

```sql
-- in expanded mode
SELECT * FROM pg_database
WHERE datname = 'mydb'
-- first field is oid
```

Then in PGDATA dir, go to the **base** directory. If you list the files, there is a directory whose name is the OID from above => postgres stores databases as folders in the PGDATA dir

```bash
# use -i option with oid2name to see all oids associated with an index
oid2name -i
# one of the element should be the db
```

