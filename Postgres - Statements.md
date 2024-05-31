---
id: Postgres - Statements
aliases: []
tags: []
---

, #postgres

### Standard statements

Note: only shows statements where I can forget the syntax, so no need for SELECT

- Insert syntax:

```sql
INSERT INTO users (username, email)
VALUES ('myuser', 'myemail'), ('myuser2', 'myemail2')

-- or from a select
INSERT INTO temp_categories
SELECT * FROM categories;
```

On sucess it shows, `INSERT 0 2`, which says that 2 rows were inserted.
The first 0 is the oid of inserted row. In new versions of postgres, by default, tables are created without oid on the row, so it shows 0.

- Update syntax:

```sql
UPDATE categories
SET title = 'no title'
WHERE description = 'Languages';
```

On success shows `UPDATE <num>` where num is the nb of updated rows

- Delete syntax:

```sql
DELETE FROM categories
WHERE description IS NULL;
```

On success shows `DELETE <num>` where num is the nb of updated rows

Note: use `RETURNING` keyword to get associated values (works with insert, update and delete)

- you can also truncate the table to remove all rows (very fast): `TRUNCATE TABLE temp_categories;`

- `ORDER BY NULLS LAST` is the default for `ASC` and `NULLS FIRST` is the default for `DESC` when using `ORDER BY`

- Pattern matching: <https://www.postgresql.org/docs/current/functions-matching.html>

  - `LIKE` syntax: `WHERE desc LIKE '%disc_ss%'` (can also use `NOT LIKE`). `ILIKE` for case insensitive
  - `SIMILAR TO`: uses sql regex instead (intersection between POSIX regex and LIKE notation). For ex, you can use `substring` like this `substring(<string> similar <pattern> escape <escape-char>)` or like this `substring(<string>, <pattern>, <escape-char>)`
  - There are also regex functions: `regexp_like`, `regex_match`, etc...

- Use `COALESCE` to returns first non null value.
- change NULL display: `\pset null (NULL)` => NULL display is "(NULL)"

- pagination: `SELECT * FROM CATEGORIES ORDER BY pk OFFSET 1 LIMIT 2`

### Subqueries

```sql
-- IN subqueries, for filtering
SELECT * FROM categories WHERE pk IN (1,2); -- (can also use NOT IN)
SELECT * FROM posts WHERE category IN (
  SELECT pk from categories where title = 'Database'
);

-- EXISTS subquery, check if subquery returns true or false
SELECT * FROM posts WHERE EXISTS (
  SELECT 1 FROM categories
  WHERE title='Database'
  AND posts.category=pk
)
```

### Joins

- INNER JOIN (default)
- LEFT/ RIGHT / FULL OUTER JOINS

```sql
SELECT *
FROM tbA JOIN tbB
USING (col1, col2);
-- equivalent to
SELECT *
FROM tbA JOIN tbB
ON tbA.col1 = tbB.col1
AND tbA.col2 = tbB.col2;
-- equivalent to
SELECT *
FROM tbA NATURAL JOIN tbB
-- only equivalent if tbA and tbB have only col1 and col2 in common
-- pretty bad practice
```

A `LATERAL JOIN` allows to join a table with a subquery where the subquery is run for each row of the main table.
The subquery is run BEFORE joining the rows and the result is used to join the rows. This allows to use information from one table to filter or process data from another table.

```sql
-- retrieves users who have at least one post with more than 2 likes
SELECT u.* FROM users u WHERE EXISTS (
  SELECT 1 FROM posts p
  WHERE u.pk = p.author
  AND likes >2
);

-- Now if we want to get all the posts with more than 2 likes
SELECT u.* , q.likes FROM users
LATERAL JOIN (
  SELECT p.author, p.likes
  FROM posts p
  WHERE u.pk = p.author
  AND likes > 2
) as q ON TRUE;
-- Note that this could be done with a standard join
```

### Combining queries

- `UNION` Combines the resultset of two or more SELECT statements. Each statement must have the same nb of columns and they should have the same datatypes.
- `INTERSECT` returns all rows that are both in first and second resultset.
- `EXCEPT` returns all rows that are in first resultset but not in second.
- For all commands, adding `ALL` allows to keep duplicates (they are removed by default)

```sql
SELECT tag as datalist FROM tags
UNION
SELECT title as datalist from titles
```

### UPSERT

There is no UPSERT statement = performs an INSERT operation, but if a row already exists with a conflicting key, it updates the existing row instead of inserting a new one. But there is an `ON CONFLICT` instruction

```sql
INSERT INTO tablename (columnlist)
VALUES (valueslist)
  ON CONFLICT target_action;
```

`ON CONFLICT` means the record already exists (meaning a record with same primary key exists)
We can also specify the conflict column `ON CONFLICT (colname)` but it is better to use a constraint
`ON CONFLICT ON CONSTRAINT constraint_name`

`target_action` could be `DO NOTHING`
or we could do an update, for ex:

```sql
INSERT INTO j_posts_tags (post_pk, tag_pk) values (2,1)
ON CONFLICT(tag_pk, post_pk)
DO UPDATE set tag_pk=excluded.tag_pk +1;
-- here excluded represents the values that would have been inserted
-- if there was no conflict
```

Here, if there is a conflict, it is as if we replace `INSERT INTO j_posts_tags (post_pk, tag_pk) values (2,1)` with
`UPDATE SET tag_pk=tag_pk+1 WHERE tag_pk=1 and post_pk=2`

### UPDATE with another table

We can update values in a table from another table.
With the method below, if there are several matches, all of them are applied, which means we get the last value

```sql
CREATE TEMP TABLE c1(
   val int,
   txt VARCHAR(255)
);
INSERT INTO c1
VALUES (1, 'a'), (2, 'b'), (3,'c');

CREATE TEMP TABLE c2 AS
SELECT * FROM c1 LIMIT 0;
INSERT INTO c2
VALUES (2, 'b_1'), (2, 'b_2'), (3,'c_1'), (4, 'd_1');

UPDATE c1 SET
txt = c2.txt
FROM c2
WHERE c1.val = c2.val
-- Final c1 is
-- 1 a
-- 2 b_2
-- 3 c
```

It is also possible to use the `MERGE` keyword

```sql
MERGE INTO c1
USING c2 on c1.val = c2.val
WHEN MATCHED THEN
 UPDATE SET txt = c2.txt
```

Note that it comes with an extra security.
With the example above I get

```txt
SQL Error [21000]: ERROR: MERGE command cannot affect row a second time
Hint: Ensure that not more than one source row matches any one target row.
```

Let's look at a more advanced scenario

```sql
CREATE TEMP TABLE c1(
 pk int GENERATED ALWAYS AS IDENTITY,
 txt VARCHAR(255),
 PRIMARY KEY(pk)
);
INSERT INTO c1 (txt) VALUES ('a'), ('b'), ('c')

CREATE TEMP TABLE c2 (LIKE c1 INCLUDING ALL);
INSERT INTO c2 (txt) VALUES ('a_1'), ('b_1'), ('c_1'), ('d_1');
DELETE FROM c2 WHERE c2.pk = 3;

MERGE INTO c1
USING c2 on c1.pk = c2.pk
WHEN MATCHED THEN
 UPDATE SET txt = c2.txt
WHEN NOT MATCHED THEN
 INSERT (pk, txt) OVERRIDING SYSTEM VALUE
 VALUES (c2.pk, c2.txt)
-- Final c1 is
-- 1 a_1
-- 2 b_1
-- 3 c
-- 4 d_1
```

We used the OVERRIDING SYSTEM VALUE because we can't insert a primary key by default.
Note that the next time we try to insert a value in c1, we get an error saying that we can't insert a duplicate primary key.

```sql
SELECT MAX(pk) FROM c1; -- returns 4
SELECT nextval('c1_pk_seq'); -- returns 4 as well

BEGIN;
LOCK TABLE c1 IN EXCLUSIVE MODE; -- Lock the table to prevent concurrent inserts

SELECT setval('c1_pk_seq', COALESCE((SELECT MAX(pk) + 1 FROM c1), 1), false); -- updates the sequence of the primary key to be the max + 1

COMMIT;
```
