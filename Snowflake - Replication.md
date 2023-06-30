#snowflake

## Overview

- Replication of databases between snowflake accounts WITHIN an organization, which is different from [[Snowflake - Cloning|cloning]] where it was within a given account
- With replication, data is moved while with cloning it is only referenced
- `ORGADMIN` role is needed to set up replication
- A db is selected to serve as primary db from which secondary dbs can be created in other accounts
- When a primary db is replicated, a snapshot of its database objects and data are transferred to the secondary db
- Currently some db objects are not replicated such as External tables, stages, pipes, streams and tasks
- Replica must be refreshed either manually or by using a [[Snowflake - Tasks and streams|Task]]
- Only databases and some of their child objects can be replicated, not users, roles, warehouses, resource monitors, shares.
- Privileges granted to db objects are not replicated to secondary db
- Billing is based on data transfer and compute resources

```sql
-- In account1, create primary db and allow replication for account2
ALTER DATABASE DB1
ENABLE REPLIACTION TO ACCOUNTS ORG1.account2

-- In account2, create your replica
CREATE DATABASE DB1_REPLICA
AS REPLICA OF ORG1.account1.DB1

-- Refresh the DB
ALTER DATABASE DB1_REPLICA REFRESH;

```
