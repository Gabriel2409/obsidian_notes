---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 226
---

#snowflake

see [[Snowflake - Data loading]]

## Semi-structured data

- Historically very hard to accommodate in data warehouses that were built to store structured data
- Snowflake extended SQL to allow to store semi-structured data types and include semi-structured functions

### Semi structured data types

- `ARRAY`: contains 0 or more elements of data. Each element is accessed by its position on the array:

```sql
CREATE TABLE MYTABLE(NAME VARCHAR, HOBBIES ARRAY);
INSERT INTO MYTABLE
SELECT 'Gab', ARRAY_CONSTRUCT('Travel', 'Sports');

-- access element with bracket notation
SELECT HOBBIES[0] FROM MYTABLE
```

- `OBJECT`: collections of key-value pairs

```sql
CREATE TABLE MYTABLE(NAME VARCHAR, ADDRESS OBJECT);
INSERT INTO MYTABLE
SELECT 'Gab', OBJECT_CONSTRUCT('postcode', '75000', 'road', 'Rivoli');

-- access element with . or bracket notation
SELECT ADDRESS['postcode'], ADDRESS.road FROM MYTABLE
```

- `VARIANT`: universal semi-structured data type used to represent arbitrary data structures that can hold up to 16MB of compressed data per row

```sql
CREATE TABLE MYTABLE(NAME VARIANT, ADDRESS VARIANT, HOBBIES VARIANT);
INSERT INTO MYTABLE
SELECT 'Gab'::VARIANT, -- we need to cast type for non object and non arrays
ARRAY_CONSTRUCT('Travel', 'Sports')
OBJECT_CONSTRUCT('postcode', '75000', 'road', 'Rivoli', 'nums':[1,2]);

-- access with : or bracket
SELECT
ADDRESS:nums[0],
-- if we do not cast value, it is shown with double quotes which can mess up joins
ADDRESS:road::string,
TO_STRING(ADDRESS:road)
FROM MYTABLE
```
