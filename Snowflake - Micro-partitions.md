---
reviewed: 2023-07-05
---

#snowflake

## Overview

- In snowflake, a [[Snowflake - Database, Schema, Table, View#Table|table]] is stored across many micro-partitions
- Partitions contain 50-500MB of uncompressed data
- We do not specify a key: snowflake automatically partitions data along the natural ordering as it is inserted. (note: we can define cluster keys and recluster the data, see [[Snowflake - Clustering]])
- Micro-partitions undergo a reorganisation process into the snowflake columnar data format
- Micro partitions are immutable (write once and read many)
- That means that an update operation will cause a new partition to be written. This is especially useful to implement [[Snowflake - Time Travel and Fail Safe|time travel and fail safe]]. Indeed after an update, we still have the old micropartition and the new which contains a copy of the old with the updated records. At the service layer, metadata are updated to track the latest version.

Note: Within a micro-partition, all columns for each row are stored together, at a lower level, Snowflake organizes and stores the data in a columnar format,

## Metadata

- Global services layer keeps track of metadata at both table and micro-partition level
- For ex, we keep track of the min value and max value of each column as well as the distinct values for every micro-partition, how many micro-partitions make up the table...
  - when doing a Min or Max query, we can use the metadata [[Snowflake - Caching|cache]] and get the result without spinning warehouses.
  - this also helps with micro-partition pruning where snowflake can optimize a query by first checking the min-max metadata of a column and discard micro-partitions from the query plan that are not required
