---
id: Postgres - Partitioning
aliases: []
tags: []
---
, #postgres

### Why partition

Idea is to divide a table into different partitions.
It is a transparent process for the client who will still only see one table.

Not the same as [[Sharding]] as all the tables are on the same node.

It is better to work with smaller sized tables because of shared buffers = part of server RAM that is shared among all the Postgres processes that is used to manage the data present in tables

In a typical operation,

- data is taken from hard disks
- data is placed in shared buffers
- data is processed in shared buffers
- data is downloaded to disks

Shared buffers are typically 1/3 or 1/4 of the total RAM so when a table grows too much compared to the shared_buffers size, performance can decrease.

- Range partitioning: data is divided into intervals (based on a set of keys): useful for partition by dates
- List partitioning: data is divided into specific list of values: useful for partition by a location, a customer,...
- Hash paritioning: data is divided based on the hash of a set of fields.

Parititioning can be done with table inheritance (old way) or declarative paritioning

### Table inheritance

```sql

CREATE TABLE table_a(
pk integer NOT NULL PRIMARY KEY,
tag text,
parent integer
);

CREATE TABLE table_b()
INHERITS(table_a);
ALTER TABLE table_b

ADD CONSTRAINT table_b_pk
primary key(pk);
```

- Doing a `\d table_a` shows the description as well as

```txt
Number of child tables: 1 (Use \d+ to list them.)
```

- Doing a `\d+ table_a` actually lists the tables
- Doing a `\d+ table_b` shows `Inherist: table_a`

```sql
INSERT INTO table_a (pk, tag, parent)
VALUES (1, 'Operating Systems', 0);

INSERT INTO table_b (pk, tag, parent)
VALUES (2, 'Linux', 0);

-- shows only Linux
SELECT * FROM table_b;

-- shows both values
SELECT * FROM table_a;

-- shows only Operating Systems
SELECT * FROM ONLY table_a;

-- actual physical update on table_b
UPDATE table_a SET tag = 'BSD Unix'
WHERE pk=2;

-- removes a child table
DROP TABLE table_b;

-- removes parent and all children
DROP TABLE table_a CASCADE;
```

### Declarative partitioning

- list partitioning

```sql
CREATE TABLE part_tags(
pk SERIAL NOT NULL,
level INTEGER NOT NULL DEFAULT 0,
tag VARCHAR(255) NOT NULL,
PRIMARY KEY(pk,level)
) PARTITION BY LIST(level);

CREATE TABLE part_tags_level_0
PARTITION OF part_tags
FOR VALUES IN (0);

CREATE TABLE part_tags_level_1
PARTITION OF part_tags
FOR VALUES IN (1);

-- Index is propagated to child tables
CREATE INDEX ON part_tags(tag);

-- Physically goes to part_tags_level_1
INSERT INTO part_tags(tag,level)
VALUES ('Linux', 1);

-- fails because partition does not exist
INSERT INTO part_tags(tag,level)
VALUES ('Other', 8);

-- adds a default partition
CREATE PARTITION part_tags_default
PARTITION OF part_tags DEFAULT;

-- now it works and physically goes to default partition
INSERT INTO part_tags(tag,level)
VALUES ('Other', 8);

-- Partition is transparent for the user
SELECT * FROM part_tags
```

```txt
   │ pk │ level │ tag
───┼────┼───────┼───────
 1 │  1 │     1 │ Linux
 2 │  3 │     8 │ Other
```

- Range partitioning

```sql
CREATE TABLE part_tags(
pk SERIAL NOT NULL,
ins_date DATE NOT NULL DEFAULT now()::date,
tag VARCHAR(255) NOT NULL,
primary key (pk, ins_date)
) PARTITION BY RANGE(ins_date);

CREATE TABLE part_tags_level_0123
PARTITION OF part_tags
FOR VALUES FROM('2023-01-01') TO ('2023-01-31');
```

- Attach, detach partitions

```sql
ALTER TABLE part_tags DETACH PARTITION
part_tags_level_0123;

ALTER TABLE part_tags ATTACH  PARTITION
part_tags_level_0123
FOR VALUES FROM('2023-01-01') TO ('2023-01-31');

```

### Tablespaces

It is possible to split the data into different tablespaces.

- First we create the tablespaces as superuser:

```sql
CREATE TABLESPACE ts_a
LOCATION '/data/tablespaces/ts_a';
CREATE TABLESPACE ts_b
LOCATION '/data/tablespaces/ts_b';

-- or see them with \db+
SELECT * FROM pg_tablespace;

-- give tablespace to user forum
ALTER TABLESPACE ts_a OWNER TO forum;
ALTER TABLESPACE ts_b OWNER TO forum;
```

- As forum user, we create the table as usual
- When creating the partition, we specify the table space.

```sql
CREATE TABLE part_tags( ...
) PARTITION BY RANGE(ins_date);

CREATE TABLE part_tags_a
PARTITION OF part_tags
FOR VALUES FROM('2023-01-01') TO ('2023-01-31')
TABLESPACE ts_a;
```

### Query plan

For partitioned data, if postgres need to join results of partitions, it performs a parallel sequence scan and then a parallel append to merge the results.
Partitions are better used when queries span as little partitions as possible.

Typically, when there is a where clause, depending on the value of `constraint_exclusion` (`on`, `off` or `partition`) in `postgresql.conf` (find it with `SHOW config_file;`)
postgres can avoid examining child partitions.

Note: setting the parameter on `on` forces Postgres to examine all tables and check for constraints in order to eliminate some tables. It might cause a lot of extra work. Usually setting on `partition` is better so that the analysis is performed only for partitions, where it makes the most sense to eliminate child tables.

