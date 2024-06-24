---
id: Postgres - Roles
aliases: []
tags: []
---
# postgres

## Basics

### Managing roles

Users and groups are grouped in the notion of **role** = collection of database permissions and connection properties. Because a role is defined at the cluster level and the permissions at the db level, a role can have different privileges and properties based on the db it uses

- Superusers can do everything across databases and schemas (just add `SUPERUSER` keyword on role creation)
- Normal users can do operations depending on their privilege set

```sql
CREATE ROLE gab
-- allows interactive login
WITH LOGIN PASSWORD '1234'
VALID UNTIL '2030-12-31 23:59:59'
```

To create a group, simply create a role without the `LOGIN` keyword (or use `NOLOGIN`)

```sql
-- create the group
CREATE ROLE authors WITH NOLOGIN;
-- add gab in the group
CREATE ROLE gab
WITH LOGIN PASSWORD '1234'
IN ROLE AUTHORS;
-- OR grant the role instead
GRANT authors to gab -- opposite is REVOKE
WITH ADMIN OPTION; -- also make gab admin of group
```

To drop a role

```sql
-- deleting a group does not delete its members
DROP ROLE IF EXISTS authors
```

Change owner

```sql
ALTER <object> OWNER TO <role>;
```

Inspect roles

```sql
SELECT current_role;

-- describe user command
\du -- see all roles
-- describe role grants
\drg -- all grants

-- see all roles in the catalog (pwd is hidden)
SELECT
rolname
,rolcanlogin
,rolconnlimit
,rolpassword
FROM pg_roles;
-- do the same query against
-- pg_authid to get the encrypted pwd, only accessible to superuser
```

NOTE: login role is not enough to connect. Postgres checks the incoming connection request agains a kind of firewall table formerly known as host-based access (**pg_hba.conf**). The connection is only granted if the file states that the role can connect to the specified database.  
NOTE: when modifying pg_hba.conf, we need to reload the new rules via a HUP (hangup) signal (`sudo -u postgres pg_ctl reload -D $PGDATA`). From within psql, you can also `SELECT pg_reload_conf();`

### pg_hba.conf

each line of the file is

```bash
<connection-type> <database> <role> <remote-machine> <auth-method>
```

- connection-type: either `local`, `host`, `hostssl` or `nohostssl`
- database: dbname or `all` (for all db) or `replication` (used to replicate data to another cluster)
- role: user or group or `all` for all roles
- remote-machine: hostname, IP address or subnet from which connection is expected, or `all`, `samehost`, `samenet`
- auth-method: deals with how the login credentials have to be checked: `scram-sha-256`, `md5` (old), `reject` to always reject, `trust` to always trust

Note: if you lose access to the cluster, you can temporarily add a line at the top of the file to trust all connections from your user.

The order of rules matter: the first that satisfies the logic is applied and others are skipped

It is possible to merge rules by separating fields with comma:

```bash
host forumdb,learnpgdb luca,enrico same-net scram-sha-256
```

By default, roles must be exactly matched. To be able to use a group (with no login) in the rules, it must be prefixed with a +

```bash
host forumdb +book_authors same-net scram-sha-256
```

The role field can also be a file of roles (line or comma separated)

```bash
host forumdb @allowed_users.txt same-net scram-sha-256
```

Inspect the rules with

```sql
SELECT * FROM pg_hba_file_rules;
```

Note: it is possible to include other HBA configuration files into the main `pg_hba.conf` file with `include_file`, `include_if_exist`, and `include_dir`

## Advanced concepts

### More on roles

```sql
-- can create db
ALTER ROLE luca WITH CREATEDB;
-- can create roles
ALTER ROLE luca WITH CREATEROLE;
-- remove permissions
ALTER ROLE luca WITH NOCREATEDB NOCREATEROLE;
-- rename
ALTER ROLE luca RENAME to fluca
```

- `SESSION_USER` = role with which we connect to db
- `CURRENT_USER` (or `CURRENT_ROLE`) = role set with `SET ROLE` (initially identical to `SESSION_USER`)

```sql
-- Automatically execute a set of command
-- when switching to this role
ALTER ROLE luca
IN database forumdb
SET client_min_messages to 'DEBUG'
```

Remember that we can query `pg_roles` to have info about the roles. Superusers can also query `pg_authid`. To have info about membership:

```sql
SELECT
  r.rolname,
  g.rolname AS group,
  m.admin_option AS is_admin
FROM pg_auth_members m
JOIN pg_roles r on r.oid = m.member
JOIN pg_roles g on g.oid = m.roleid
ORDER BY r.rolname;
```

```txt
   │ rolname    │ group                │ is_admin
───┼────────────┼──────────────────────┼──────────
 1 │ luca       │ book_authors         │ false
 2 │ luca       │ forum_admins         │ false
 3 │ luca       │ forum_stats          │ false
 4 │ pg_monitor │ pg_read_all_settings │ false
 5 │ pg_monitor │ pg_read_all_stats    │ false
 6 │ pg_monitor │ pg_stat_scan_tables  │ false
```

### About role inheritance

When a role inherits from another role, by default it can do everything the parent can. That is because `GRANT` comes with an optional `WITH INHERIT` clause that is true by default. It is also possible to grant a role without inheritance:
In this case, user will need to explicitely set the role to be able to perform the action.

```sql
-- good practice to remove grants first
REVOKE SELECT ON forum.users FROM luca;

CREATE ROLE forum_email
-- either specify NOINHERIT here
-- or do GRANT WITH NOINHERIT after
WITH NOLOGIN NOINHERIT;

GRANT USAGE ON SCHEMA forum TO forum_emails;
GRANT SELECT(email) ON forum.users to forum_emails;
GRANT forum_emails to luca;
-- now luca needs to explicitely SET role to
-- forum_emails to perform operation
```

To know if a role has the right to perform an action, postgres first checks if the role was granted the necessary privileges.

If not, then postgres checks all the groups that the user belongs to recursively. If the group is set without the inherit clause for this user, then the search on this branch stops.

### Access control lists (ACL)

- stores permissions assigned to roles and objects
- strictly tied to the role AND the db object: granting a role for a db object does not grant it for another object

- structure = `grantee/flags/grantor` (grantee is the role who has the privileges, grantor is the one granting the privileges. If they are identical, it means the role is the owner of the object). Flags are one letter (`a` for INSERT (append), `r` for SELECT (read), `w` for UPDATE (write), `d` for DELETE, etc.)
- check the ACL with `\dp`

If an object is created without `GRANT`, no ACL is stored. Instead, `PUBLIC` permissions are applied.
By default, PUBLIC has only execute permission (X) on routines, connect to and create temp objects on db (cT), use of languages, types and domains (U).

```sql
-- check current privileges
SELECT acldefault('f', r.oid) FROM
pg_roles r
WHERE r.rolname = CURRENT_ROLE
```

```txt
   │ acldefault
───┼──────────────────────────
 1 │ {=X/forum,forum=X/forum}
```

NOTE: check `aclexplode` function to have individual parts of the acl

### Grants to cols

```sql
-- luca will only be able to select
-- specified columns
REVOKE ALL ON forum.users;
GRANT SELECT(username, gecos)
ON forum.users TO luca
```

### Permissions related to schemas

- only two permissions: `CREATE` and `USAGE` (or `ALL`)
- USAGE is needed to access anything in the schema. SO even if you are owner of a table,
  you need USAGE to query it.
- Having USAGE permission on a schema is not enough to use the underlying objects. You need permissions on these objects as well.

- Mass grants of underlying objects

```sql
GRANT SELECT, INSERT, UPDATE
  ON ALL TABLES IN SCHEMA myschema
  TO luca;
```

- Note that if we want this to apply to future objects

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA myschema
GRANT SELECT, INSERT, UPDATE ON TABLES TO luca;
```

### RLS (Row level security)

- if you have `BYPASSRLS`, you are not affected

```sql
CREATE POLICY show_only_my_posts
ON posts
FOR SELECT
-- determines which rows are visible
USING (author = (SELECT pk FROM users
  WHERE username = CURRENT_ROLE
  ));

ALTER TABLE posts ENABLE ROW LEVEL SECURITY
```

THen check policies with `\dp posts`

By default policies are permissive (if one of them is valid, you can see the row). You can mark it as restricive (must only be true with `AS RESTRICTIVE`)

### Configuring the cluster for ssl

in postgresql.conf,

```bash
ssl=on
ssl_key_file='path/to/server.key'
ssl_cert_file='path/to/server.crt'
```

Then in `pg_hba.conf`, use `hostssl`

To connect to the cluster, add `?sslmode=require` to the connection string

