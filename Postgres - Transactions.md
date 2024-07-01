---
id: Postgres - Transactions
aliases: []
tags: []
---

, #postgres

Postgres [[Transaction]] are [[ACID]] compliant.
Each individual statement is automatically wrapped in a transaction but you can define `BEGIN [TRANSACTION]` and `COMMIT` to have more control (explicit transaction).

Note: for explicit transactions, if an error is encountered, Postgres will refuse to commit. Leaving the transaction or issuing commit will instead `ROLLBACK`

Moreover, transactions are time-discrete: the time does not change during a transaction.

### Transactions identifiers

Postgres implements MVCC.

Any transaction is assigned a unique number called **transaction identifier** (or xid). The xid is automatically assigned and guarantees that no transactions with the same xid exist at the same time in the db.

```sql
-- gets the current transaction id
SELECT current_time, txid_current();
```

In postgres, each row (or tuple) contains metadata that includes information about the transaction that created or last modified the row.

Postgres maintains a few hidden column such as

- `xmin` = transaction id that modified/created row
- `xmax` = transaction id that deleted row
- `cmin` = command id of insertion
- `cmax` = command id of deletion

```sql
CREATE TABLE names (name varchar(255));

INSERT INTO names (name) VALUES ('Alice'), ('Bob'), ('Charlie');
BEGIN;
INSERT INTO names (name) VALUES ('David');
UPDATE names SET name = 'Adam' WHERE name = 'Alice';
DELETE FROM names WHERE name = 'Charlie';
COMMIT;

SELECT *, xmin, xmax, cmin, cmax FROM names;
```

```txt
   │ name  │ xmin │ xmax │ cmin │ cmax
───┼───────┼──────┼──────┼──────┼──────
 1 │ Bob   │ 800  │ 0    │ 0    │ 0
 2 │ David │ 801  │ 0    │ 0    │ 0
 3 │ Adam  │ 801  │ 0    │ 1    │ 1
```

- here the first insert has a transaction id of 800 (xmin)
- then there is a complete transaction which inserts David, updates Alice to Adam and deletes charlie. This transaction has an id of 801. David is inserted with the first command (cmin == 0), Adam is updated with the second (cmin == 1)
- Alice and Charlie were deleted so we can't see them but they should have a xmax of 801. cmax for Charlie should be equal to 2 and 1 for Alice.

- xid is a cyclical counter with a modulo 2^31 so that for any current values, there are 2^31 smaller values and 2^31 larger values.
- If we keep doing transactions, we may have a warparound. Because postgres does not authorize two xid to have the same number, you may receive messages like

```txt
WARNING: database "forumdb" must be vacuumed within 177009986 transactions
HINT: To avoid a database shutdown, execute a database-wide VACUUM in "forumdb"
```

By default a larger xid means the transaction occured later. But when xid overflows, xid restarts. Postgres implements **tuple freezing**: once a tuple is frozen, its xmin is always considered in the past with regards to any other running transactions even if its xmin is larger. Postgres uses a status bit to indicate whether the tuple has been frozen.

### Virtual xid

in order to avoid wasting xid, Postgres first creates a virtual xid. This virtual xid is converted to a real xid only in non read workload.
Function `txid_current()` will always return a xid even in read only workload, which means it will create a xid if it was not created. To avoid creating one, you can use `txid_current_if_assigned()`

### MVCC

Multi Version Concurrency Control dictates that instead of modifying an existing tuple, the system has to replicate the tuple, apply the change and invalidate the original one. This allows to perform the snapshots that are used by all Postgres isolation levels.
A snapshot is a time window in which a certain transaction is allowed to receive data. Use `txid_current_`snapshot() to see it.

- `587:587`: sees the state of the db before the transaction started
- `587:589`: also includes transaction 588 (upper bound is excluded)

Note that MVCC does not prevent locks. If 2 or more concurrent transactions start manipulating the same set of data, the system has to apply ordered changes and therefore must force a lock.
At any moment, check the locks with `SELECT * FROM pg_locks`

```sql
-- check active transactions
SELECT
    pid
    ,datname
    ,usename
    ,application_name
    ,client_hostname
    ,client_port
    ,backend_start
    ,query_start
    ,query
    ,state
FROM pg_stat_activity
WHERE state = 'active';
```

As MVCC needs to maintain tuple versions, to reclaim storage space, we use `VACUUM` and the backround running process autovacuum.

### More examples with xmin, xmax, cmin, cmax

```sql
DROP TABLE IF EXISTS names;
CREATE TABLE names(
name varchar(255)
);
INSERT INTO names(name)
VALUES ('Alice'), ('Bob'), ('Charlie'), ('David'), ('Eric');

BEGIN TRANSACTION;

DECLARE name_cursor CURSOR FOR
SELECT
name,
-- id of transaction that created/modified the row
xmin,
-- id of transaction that deleted the row
xmax,
-- Statement nb in the transaction that created/modified the row
cmin,
-- Statement nb in the transaction that deleted the row
cmax,
-- Forces postgres to assign a xid
txid_current()
FROM names
ORDER by name;

DELETE FROM names WHERE name = 'Bob';

DELETE FROM names WHERE name = 'Charlie';

UPDATE names SET name = 'Didier' WHERE name = 'David';

-- iterates over the cursor
FETCH ALL FROM name_cursor;
```

```txt
name   |xmin|xmax|cmin|cmax|txid_current|
-------+----+----+----+----+------------+
Alice  |939 |0   |0   |0   |         940|
Bob    |939 |940 |0   |0   |         940|
Charlie|939 |940 |1   |1   |         940|
David  |939 |940 |2   |2   |         940|
Eric   |939 |0   |0   |0   |         940|
```

xmin is the same for everyone as they were inserted in the same transaction

Charlie, Bob and David were deleted in current transaction (statements 0,1,2 as the SELECT does not count).

In postgres, cmin and cmax are identical (overlapped fields). It is a design choice as it is rare to have a transaction both create and delete a row at the same time.

```sql
COMMIT;

SELECT name, xmin, xmax, cmin, cmax, txid_current()
FROM names
ORDER by name;
```

```txt
name  |xmin|xmax|cmin|cmax|txid_current|
------+----+----+----+----+------------+
Alice |939 |0   |0   |0   |         941|
Didier|940 |0   |2   |2   |         941|
Eric  |939 |0   |0   |0   |         941|

```

Here we don't see the deleted rows anymore. We see that Didier has the same cmax and cmin as David had.

### Postgres Isolation levels

See [[Transaction Isolation levels]]

```txt
|Isolation Level |Dirty Read            |Nonrepeatable Read|Phantom Read          |Serialization Anomaly|
|----------------|----------------------|------------------|----------------------|---------------------|
|Read uncommitted|Allowed, but not in PG|Possible          |Possible              |Possible             |
|Read committed  |Not possible          |Possible          |Possible              |Possible             |
|Repeatable read |Not possible          |Not possible      |Allowed, but not in PG|Possible             |
|Serializable    |Not possible          |Not possible      |Not possible          |Not possible         |
```

Postgres does not have READ UNCOMMITTED but you can still request it.

```sql
-- Preparation run before each exp
DROP TABLE IF EXISTS names;
CREATE TABLE names(
name varchar(255)
);
INSERT INTO names(name)
VALUES ('Alice'), ('Bob'), ('Charlie'), ('David'), ('Eric');
```

```sql
BEGIN TRANSACTION
-- ISOLATION LEVEL READ COMMITED; -- default


-- forces assignment of transaction id
SELECT txid_current();-- 1026

SELECT txid_current_snapshot(); -- 1026:1026:

-- In another session,
BEGIN TRANSACTION;
SELECT txid_current(); -- 1027
DELETE FROM names WHERE name = 'Alice';
INSERT INTO names VALUES ('Frank');
UPDATE names SET name = 'Bernard' WHERE name = 'Bob';
COMMIT;
----

-- Back in current session
SELECT txid_current_snapshot(); -- 1026:1028

SELECT xmin, xmax, cmin, cmax, * FROM names;
```

We see the rows committed by the other transaction

```txt

xmin|xmax|cmin|cmax|name   |
----+----+----+----+-------+
1025|0   |0   |0   |Charlie|
1025|0   |0   |0   |David  |
1025|0   |0   |0   |Eric   |
1027|0   |1   |1   |Frank  |
1027|0   |2   |2   |Bernard|
```

```sql
COMMIT;
```

If instead, we had set a higher isolation level

```sql
BEGIN TRANSACTION
ISOLATION LEVEL REPEATABLE READ;

-- SAME AS BEFORE
...

-- second call, snapshot has not changed
SELECT txid_current_snapshot(); -- 1026:1026

SELECT xmin, xmax, cmin, cmax, * FROM names;
```

We don't see the rows committed by the other transaction.
Alice and Bob have the xmax set.
but I dont see Bernard and Frank

```txt
xmin|xmax|cmin|cmax|name   |
----+----+----+----+-------+
1025|1027|0   |0   |Alice  |
1025|1027|2   |2   |Bob    |
1025|0   |0   |0   |Charlie|
1025|0   |0   |0   |David  |
1025|0   |0   |0   |Eric   |
```

```sql
COMMIT;
```

So all in all, READ COMMITED guarantees that you do not see uncommitted values. However, because the snapshot can include higher transaction ids, it does not stop non repeatable reads even though it uses MVCC.

REPEATABLE READ guarantees that the snapshot taken stops at the current transaction id. in Postgres, it stops both Non Repeatable reads and phantom reads.

In other words,
REPEATABLE READ sees a snapshot as of the start of the first non-transaction-control statement in the transaction.
READ COMMITTED sees it as of the start of the current statement within the transaction

Now what happens if after the other transaction commits, in the current transaction we try:

```sql
UPDATE names SET name = 'Bonjour' WHERE name = 'Bob';
```

NON REPEATABLE READ: ERROR: could not serialize due to concurrent update

READ COMMITTED: Bob was already updated to Bernard by other transaction so no row is affected. We can however do `UPDATE names SET name = 'Bonjour' WHERE name = 'Bernard';` and there won't be any problem';

SERIALIZABLE offers stronger guarantees against anomalies:
If 2 concurrent transactions lead to a different state depending on the COMMIT order, one of them will be forced to rollback.

### Savepoints

A savepoint is a way to split the transaction into smaller blocks that can be rolled back independently of one another.

```sql
BEGIN;
-- do stuff
SAVEPOINT mysave;
-- do stuff
SAVEPOINT mysave2;
-- do stuff
ROLLBACK TO SAVEPOINT mysave;
-- now everything after mysave is gone.
-- we can't go back to mysave2
```

We can also remove a savepoint with `RELEASE SAVEPOINT mysave`, it is as if the save point was not created in the first place (it does not rollback)

### Deadlocks

Occur when different transactions depend on each other in a circular way. Postgres has a deadlock detection engine which finds stalled transactions and terminates them in case of deadlock.
Type of locks: <https://www.postgresql.org/docs/current/explicit-locking.html>

### Vacuum

VACUUM analyses stored tuple versions and reove the ones that are no longer perceivable. It can be run agains a single table, a subset of columns or a full db.

- plain `VACUUM` throws away dead tuples but does not defragment the table (no space reclaimed)
- `VACUUM FULL` performs a full rewrite, throwing dead tuples and also reclaiming disk size
- `VACUUM FREEZE` marks already consolidated tuples as frozen, preventing the xid warparound problem

Postgres also comes with an autovacuum process.
