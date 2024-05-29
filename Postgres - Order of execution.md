---
id: Postgres - Order of execution
aliases:
  - Postgres - Order of execution
tags: []
---

#postgres

- `FROM` and `JOIN`s: The query execution starts with the `FROM` clause to identify the source tables and apply any `JOIN` operations.
- `WHERE`: filters rows based on the specified conditions.
- `GROUP BY`: groups the filtered rows.
- `HAVING`: filters the grouped rows.
- `SELECT`: specifies the columns to be returned.
- `DISTINCT`: If used, removes duplicate rows from the results.
- `ORDER BY`: sorts the results.
- `LIMIT` and `OFFSET`: restrict the number of rows returned.
