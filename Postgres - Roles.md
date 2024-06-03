---
id: Postgres - Roles
aliases: 
tags:
---
#postgres

## Managing roles

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
DROP ROLE IF EXIST authors
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
-- pg_authid to get the encrypted pwd
```

NOTE: login role is not enough to connect. Postgres checks the incoming connection request agains a kind of firewall table formerly known as host-based access (**pg_hba.conf**). The connection is only granted if the file states that the role can connect to the specified database.  
NOTE: when modifying pg_hba.conf, we need to reload the new rules via a HUP signal (`sudo -u postgres pg_ctl reload -D $PGDATA`). From within psql, you can also `SELECT pg_reload_conf();`

## pg_hba.conf

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
