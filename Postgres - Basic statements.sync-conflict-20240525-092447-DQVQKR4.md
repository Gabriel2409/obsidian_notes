---
id: Postgres - Basic statements
aliases: []
tags: []
---

#postgres

- DDL (Data Definition Language): Commands used to manage databases 
- DML (Data Manipulation Language): Commands used to insert, delete, update and select


## Databases and schemas
Connect with [[Postgres - psql|psql]],
```sql
-- makes a physical copy of template1 db
CREATE DATABASE mydb;
```

Databases are organized into namespaces, called schemas. Schemas can not be nested (flat namespace)

[[Postgres - roles]]

Starting Postgres15, a normal user will not be able to execute DDL on the public schema and not perform DML on the public schema unless they receive permission from a superuser.

```sql
CREATE USER myuser WITH PASSWORD '1234' LOGIN;
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
\dt now shows the mytable table
```

