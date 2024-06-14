---
id: Postgres - CTE
aliases: []
tags: []
---

, #postgres

Common table expression is a temporary result taken from a sql statement. Its lifetime is the lifetime of the query.

```sql
WITH cte_name (column_list) AS (
 CTE_query_definition
)
statement;
-- example
WITH intermediary AS MATERIALIZED (
 SELECT txt from c1 ORDER BY pk limit 2
)
SELECT * FROM intermediary JOIN c2 USING (txt);
```

By default, CTE are not materialized

**Materialized CTE (WITH MATERIALIZED):**

- The result is treated as a temporary table, which persists throughout the query execution.
- Subsequent references to the same CTE reuse the stored result, avoiding redundant computation.
- Useful when the CTE is referenced multiple times or when the intermediate result is large.
- Similar to creating a temporary table explicitly.

**Non-Materialized CTE (WITHOUT MATERIALIZED):**

- It behaves like a subquery, re-evaluating the CTE expression each time it’s referenced.
- Suitable for small intermediate results or when the CTE is referenced only once.
- Allows the optimizer to apply further optimizations during query planning.
- Similar to using a subquery directly.

**Why Choose Materialized or Non-Materialized CTEs?**

Materialized CTEs are generally faster when:

- The CTE is referenced multiple times in the query.
- The tables in the CTEs aren’t optimized for the join condition.
- The work_mem setting is sufficient to handle the intermediate result in memory.

Non-materialized CTEs are preferable when:

- The CTE is referenced only once.
- The optimizer can optimize the subquery more effectively.
- The intermediate result is small and doesn’t justify materialization.

```sql
-- Move records from one table to another
WITH deleted AS (
 DELETE FROM c1 WHERE c1.txt LIKE '%\_1%'
 RETURNING *
)
INSERT INTO c2 (txt) SELECT txt FROM deleted
```

### Recursion

A recursive CTE allows an auxiliary statement to reference itself and, therefore, join itself onto previously computed results. Useful to join a table an unknown number of times.
It is made by an auxiliary statement built on top of:

- A non recursive statement, which works as a bootstrap statement
- A recursive statement that references the bootstrap statement or itself

The two parts are joined with an UNION

```sql
WITH RECURSIVE RecursiveCTE AS (
    -- Anchor (initial) query
    SELECT ...
    UNION [ALL]
    -- Recursive query
    SELECT ...
    FROM RecursiveCTE
    WHERE ...
)
SELECT ...
FROM RecursiveCTE;
```

Example: join each country with all of his ancestors

```sql
CREATE TEMP TABLE country(
pk int GENERATED ALWAYS AS IDENTITY,
country VARCHAR(255),
parent VARCHAR(255),
PRIMARY KEY(pk)
);

INSERT INTO country
(country, parent) VALUES
('France', 'Europe'),
('Espagne', 'Europe'),
('Europe', 'World'),
('Paris', 'France');

WITH RECURSIVE rec_country AS(
-- Anchor table is the initial table with an additional
-- column to track the level
SELECT country, parent, 1 as parent_level
FROM country
UNION ALL
-- now we join the initial table with
-- the recursive table to match a country with its next level ancestor
SELECT country.country, rec_country.parent, parent_level +1 as parent_level
FROM country JOIN rec_country
ON country.parent = rec_country.country
)
SELECT country, parent, parent_level
FROM rec_country
```

Result is

```txt
France Europe 1
Espagne Europe 1
Europe World 1
Paris France 1
Paris Europe 2
Espagne World 2
France World 2
Paris World 3
```

