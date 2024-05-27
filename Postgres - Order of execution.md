---
id: Postgres - Order of execution
aliases:
  - Postgres - Order of execution
tags: []
---

#postgres

- FROM and JOINs: The query execution starts with the FROM clause to identify the source tables and apply any JOIN operations.
- WHERE: The WHERE clause filters rows based on the specified conditions.
- GROUP BY: If the query has a GROUP BY clause, it groups the filtered rows.
- HAVING: The HAVING clause filters the grouped rows.
- SELECT: The SELECT clause specifies the columns to be returned.
- DISTINCT: If DISTINCT is used, it removes duplicate rows from the results.
- ORDER BY: The ORDER BY clause sorts the results.
- LIMIT and OFFSET: Finally, LIMIT and OFFSET are applied to the sorted results to restrict the number of rows returned.
