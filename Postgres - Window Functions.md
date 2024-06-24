---
id: Postgres - Window Functions
aliases: []
tags: []
---

, #postgres

Let's consider this table

```sql
CREATE TEMP TABLE c AS
SELECT
generate_series(1,6) % 3 AS cat,
10 *(generate_series(1,6) % 3 +1) as num
ORDER BY num
```

```txt
num|val|
---+---+
  0| 10|
  0| 10|
  1| 20|
  1| 20|
  2| 30|
  2| 30|
```

### Basic examples

```sql

-- standard aggregation
SELECT SUM(num) FROM c
```

```txt
120
```

```sql
-- Window function do not remove nb of rows
-- Here we have the total sum added for every row
SELECT num, SUM(num) OVER () FROM c
```

```txt
sum|
---+
120|
120|
120|
120|
120|
120|
```

```sql

-- Add an explicit partition
SELECT SUM(num) OVER (PARTITION BY cat) FROM c
-- Even better, add ordering
SELECT SUM(num) OVER (PARTITION BY cat ORDER BY num) FROM c
```

```txt
sum|
---+
 20|
 20|
 40|
 40|
 60|
 60|
```

It is possible to use the **WINDOW** keyword

```sql
SELECT cat, num, row_number() OVER w, SUM(num) OVER w
FROM c
WINDOW w AS (ORDER BY num)
```

```txt
cat|num|row_number|sum|
---+---+----------+---+
  0| 10|         1| 20|
  0| 10|         2| 20|
  1| 20|         3| 60|
  1| 20|         4| 60|
  2| 30|         5|120|
  2| 30|         6|120|
```

### Frame clause

#### Interaction with window functions

Aggregate functions such as `SUM()`, `AVG()`, `MAX()`, `MIN()`, and `COUNT()` can utilize the `FRAME` clause.

Ranking functions such as `RANK()`, `DENSE_RANK()`, `ROW_NUMBER()`, and `NTILE()` do not use the `FRAME` clause because they inherently operate over the entire partition or ordered set.

Value functions such as `LEAD()`, `LAG()`, `FIRST_VALUE()`, and `LAST_VALUE()` can use the `FRAME` clause to limit the rows considered.

#### Theory

[PostgreSQL: Documentation: 16: 4.2. Value Expressions](https://www.postgresql.org/docs/current/sql-expressions.html#:~:text=The%20frame_clause%20specifies%20the%20set,instead%20of%20the%20whole%20partition.)

- Specifies the set of rows constituting the *window frame*, which is a subset of the current partition
- Useful in partitions with `ORDER BY` to ensure reproducibility

```txt
{ RANGE | ROWS | GROUPS } frame_start [ frame_exclusion ]
{ RANGE | ROWS | GROUPS } BETWEEN frame_start AND frame_end [ frame_exclusion ]
```

- In **ROWS** mode, the window function considers each individual row separately. The frame specified using **ROWS** operates row by row, without any grouping.
- In **RANGE** mode, the window function looks at rows grouped together based on their **values** in the **ORDER BY** column. It’s like grouping rows with similar values together.
- In **GROUPS** mode (supported in PostgreSQL and SQLite), you can refer to preceding groups. It operates on peer groups of rows with the same value (or combination of values) in the **ORDER BY** column. A peer group contains consecutive rows with equivalent values

- Default is `RANGE UNBOUNDED PRECEDING`, which is the same as `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- In `RANGE` or `GROUPS` mode, a *`frame_start`* of `CURRENT ROW` means the frame starts with the current row's first *peer* row (a row that the window's `ORDER BY` clause sorts as equivalent to the current row), while a *`frame_end`* of `CURRENT ROW` means the frame ends with the current row's last peer row. In `ROWS` mode, `CURRENT ROW` simply means the current row.

#### frame start and frame end examples

We'll look at the effect of the frame clause on the sum column.

```sql
SELECT cat, num, row_number() OVER w, SUM(num) OVER w
FROM c
WINDOW w AS (
 ORDER BY num
 <FRAME_CLAUSE>
)
```

Note: The clause without `BETWEEN` is syntactic sugar for `... BETWEEN ... AND CURRENT ROW`

```sql
-- row_up: takes all the rows before up to the current row
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- row_uf: takes all the rows from the current row up to end
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
-- row_1p1f: takes previous, current and next row
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
-- range_1p1f: takes all values in range [val-1;val+1]
RANGE BETWEEN 1 PRECEDING AND 1 FOLLOWING
-- group_1p1f: takes current group of value and preceding
GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING
-- range_12p: takes  and all values in range [val-12;val]
RANGE BETWEEN 12 PRECEDING AND CURRENT ROW
```

We can run multiple partitions at once

```sql
SELECT cat, num,
SUM(num) OVER (ORDER BY num ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as row_up,
SUM(num) OVER (ORDER BY num ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) as row_uf,
SUM(num) OVER (ORDER BY num ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) as row_1p1f,
SUM(num) OVER (ORDER BY num RANGE BETWEEN 1 PRECEDING AND 1 FOLLOWING) as range_1p1f,
SUM(num) OVER (ORDER BY num GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING) as group_1p1f,
SUM(num) OVER (ORDER BY num RANGE BETWEEN 12 PRECEDING AND CURRENT ROW) as range_12p,
FROM c
```

```txt
cat|num|row_up|row_uf|row_1p1f|range_1p1f|group_1p1f|range_12p|
---+---+------+------+--------+----------+----------+---------+
  0| 10|    10|   120|      20|        20|        60|       20|
  0| 10|    20|   110|      40|        20|        60|       20|
  1| 20|    40|   100|      50|        40|       120|       60|
  1| 20|    60|    80|      70|        40|       120|       60|
  2| 30|    90|    60|      80|        60|       100|      100|
  2| 30|   120|    30|      60|        60|       100|      100|
```

Note: Range looks at the values to know if it should add adjascent rows or not. Groups on the other hand looks at the groups, which means the nb of distinct values. Rows makes a diff between rows even if they have the same value.

#### frame exclusion

Note: If an exclusion makes it so that there is nothing left, value will be NULL

- `EXCLUDE CURRENT ROW` excludes the current row from the frame. Useful with `ROW` frame clauses

- `EXCLUDE GROUP` excludes the current row and its ordering peers from the frame. Useful with `GROUP` and `RANGE` frame clauses

- `EXCLUDE TIES` excludes any peers of the current row from the frame, but not the current row itself.  Will have no effect on a row frame clause as each row is considered independently

- `EXCLUDE NO OTHERS` simply specifies explicitly the default behavior of not excluding the current row or its peers.

### Useful window functions

- `ROW_NUMBER()`: assigns a progressive number for each row within the partition (does not interact with frame clause)
- `FIRST_VALUE()`: returns the first value in the partition
- `LAST_VALUE()`: returns the current last value in the partition

```sql
CREATE TEMP TABLE d AS
SELECT
generate_series(1,9) % 3 AS cat,
generate_series(1,9) as num;

SELECT cat, num,
row_number() OVER w,
first_value(num) OVER w,
last_value(num) OVER w
FROM d
WINDOW w AS (PARTITION BY cat ORDER BY num)
```

```txt
cat|num|row_number|first_value|last_value|
---+---+----------+-----------+----------+
  0|  3|         1|          3|         3|
  0|  6|         2|          3|         6|
  0|  9|         3|          3|         9|
  1|  1|         1|          1|         1|
  1|  4|         2|          1|         4|
  1|  7|         3|          1|         7|
  2|  2|         1|          2|         2|
  2|  5|         2|          2|         5|
  2|  8|         3|          2|         8|
```

- `RANK()` and `DENSE_RANK()` can be used to rank rows within the partition. They give the same result as `ROW_NUMBER()` in the absence of duplicates. In case of ties, `RANK` will jump values (for ex 1 1 3) and `DENSE_RANK` won't (1 1 2)

- `LAG` and `LEAD` allow to look at next and previous rows

```sql
SELECT x,
LAG(x) OVER w AS lag1,
LAG(x,2) OVER w AS lag2,
LEAD(x) OVER w AS lead1,
LEAD(x,2) OVER w AS lead2
FROM
(SELECT generate_series(1,5) as x)
WINDOW w as (order by x)
```

```txt
x|lag1|lag2|lead1|lead2|
-+----+----+-----+-----+
1|    |    |    2|    3|
2|   1|    |    3|    4|
3|   2|   1|    4|    5|
4|   3|   2|    5|     |
5|   4|   3|     |     |
```

- `CUME_DIST` calculates the cumulative distribution of a value within a partition
- `NTILE` groups the rows sorted in the partition into buckets

