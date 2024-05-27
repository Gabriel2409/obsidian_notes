---
id: Postgres - theory
aliases: []
tags: []
---

#postgres

## Intro

- **Cluster**: The whole Postgresql service. Handle multiple databases
- **Postmaster**: first process the cluster executes, responsible for keeping track of the activites of the whole custer. It spawns a new backend process every time a connection is established. Process Id is stored in PGDATA dir in `postmaster.pid`
- **Database**: Isolated data container made up of different objects (schemas, tables, triggers,...)

Postgres keeps track of the tables' structures, indexes, functions and everything needed to manage the cluster in its dedicated storage, the catalog. Ex: `SELECT * FROM pg_catalog.pg_tables`

Postgres also provides the information schema which is a standard across many db engines, but the catalog gives more information. Ex: `SELECT * FROM INFORMATION_SCHEMA.tables`

## Processes

```bash
pstree -p postgres # shows all processes
```

- **checkpointer**: executes checkpoints = point in time at which **all data pages that were dirty (modified but not yet written to disk) have been flushed to disk**. A checkpoint is **NOT A SNAPSHOT!** After a checkpoint, the checkpointer writes a checkpoint record to the WAL. Then WAL entries and checkpoint records are flushed to the disk. Only then is the checkpoint complete. During recovery, PostgreSQL will replay WAL records from the last completed checkpoint to ensure the database is consistent. Note: the Background Writer may have already flushed some records to disk after the checkpoint. To avoid redundant operations during recovery, PostgreSQL uses a mechanism called page LSN (Log Sequence Number). If the LSN of the WAL record is greater than the LSN of the data page, PostgreSQL knows that the changes have not yet been applied to the page, and it applies them. Otherwise, it skips the redundant operation.

- **background writer**: push data out of memory to permanent storage in a background, asynchronous manner. Unlike the checkpointer, which triggers checkpoints at specific intervals, the background writer operates continuously in the background. The background writer does not necessarily flush all dirty pages during each cycle. Instead, it employs a **priority-based strategy to select which pages to flush**.

- **walwriter**: Writes the [[Write Ahead Log]]. When a user transaction modifies data, the changes made by these transactions are logged in the Write-Ahead Log (WAL). The WAL writer process is responsible for flushing WAL records from the WAL buffer to the WAL files on disk buffer as WAL records. NOTE: the walwriter does not interact with the data.

Example sequence:

- A user initiates a transaction by executing commands to modify data in the database.
- As part of the transaction, modifications are made to the data pages in the shared buffer cache. These changes are logged in the WAL buffer as WAL records.
- walwriter periodically flushes the WAL records from the WAL buffer to the WAL files on disk, ensuring their durable storage. Note: if a crash occurs after a transaction is committed but before WAL records are flushed to disk, we may lose some data.
- at some point, the background writer (or the checkpointer) Will flush the records to disk.

---

- **autovacuum launcher**: ensures continuous database maintenance by scheduling and managing autovacuum worker processes. These workers perform necessary tasks such as vacuuming and analyzing tables to reclaim space, maintain performance, and manage transaction IDs
- **logical replication launcher**: manages logical replication subscriptions by launching and monitoring subscription workers

When a client connects to the cluster, a backend process is spawned by the postmaster. This is postgres approach to concurrency (using processes and not threads)

## Init

Clients connect to a db, not the whole cluster. When initializing the cluster (with `initdb`), postgres builds 2 template databases: template0 and template1 which are used as starting points to clone dbs. template1 is created first and template0 is a clone of template1. By default, all new databases are clones of template1 so if you add an object in template1, it will be found in new dbs as well. template0 is the safety copy (you can not connect to it)

Usually a fresh cluster also comes with a postgres db that allows to interact with the cluster.

## PGDATA

- Postgres stores data in the PGDATA folder.
- In a docker container, run `echo $PGDATA`, it should show `/var/lib/postgresql/data`
- If not, when connected, run `SHOW data_directory;` to see it

- **base**: directory containing all user data (db, tables, etc)
- **global**: directory containing cluster wide objects
- **pg_wal**: stores WAL files
- **pg_stat** and **pg_stat_tmp**: contain information about status and health of cluster
- **pg_tblspc**: symbolic links to outside folders, allowing postgres to access data not in the PGDATA folder

- Each file in these folders is named as a numeric identifier (object identifier OID, the filenode). This allows postgres to know which file to interact with when specific objects are mentionned in human readable format

```bash
# shows mapping of dbs
oid2name
# shows a specific file
oid2name -d postgres -f 13414 # shows pg_toast_13411
# shows table spaces
oid2name -s # contains at least pg_default and pg_global
```

Note: files are at most 1GB so postgres will create new files (with suffix .1, .2, etc) if necessary

Note: on Ubuntu, config may be stored outside. Find location with `SHOW config_file;` and `SHOW hba_file;`

When systemd launches the service, it reads the conf file and is able to find location of pgdata
