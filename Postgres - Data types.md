---
id: Postgres - Data types
aliases: []
tags: []
---

, #postgres

To see the types and the defaults of a table, a good thing to do is to set expanded mode with `\x` and
then to run `\d mytable`

### Boolean

- stored on 1 byte
- TRUE: Represented by the byte value 1.
- FALSE: Represented by the byte value 0.

Note: In PostgreSQL, NULL values are not stored directly within the boolean byte itself.
Instead, NULL handling is part of the overall tuple structure in a PostgreSQL table.
Each tuple (row) has a null bitmap that indicates whether any column in the tuple is NULL.

The datatype input function for type boolean accepts string representations.

- For the true state: 'true', 'yes', 'on', '1'
- For the false state: 'false', 'no', 'off', '0'

```sql
-- equivalent statements
SELECT * FROM users WHERE users_on_line = '0';
SELECT * FROM users WHERE users_on_line = false;
SELECT * FROM users WHERE NOT users_on_line;
```

### Numeric

#### ints

- smallint: 2 bytes: -(2^15) to (2^15)-1
- integer or int4: 4 bytes
- bigint or int8: 8 bytes

Int4 in more details:

- First bit is sign bit: 0 for positive and 1 for negative
- Postgres uses 2 complement notation.
  So if we have a negative nb, add 1 and invert the bits to get the values. This can't be done
  for the nb 1000000000000000 (you can't add one without overflow) which is why there is an extra negative nb

#### small serials

- smallserial: autoincrementing integer: 2 bytes: 1 to (2^15)
- serial: autoincrementing integer: 4 bytes
- serial: autoincrementing integer: 8 bytes

Serials are not true types but merely a notational convenience for creating unique identifier columns.
Under the hood they create a signed integer column, which is why the range is identical even though there are no
negative numbers

#### Arbitrary precision numbers

numeric or decimal: user specified precision: allow for EXACT calculation

- up to 131072 digits before the decimal point
- up to 16383 digits after the decimal point
- recommended for storing monetary amount or other quantities where exactness is required.

In a numeric type, the **precision** is the total count of significant digits and the **scale** is the nb of digits after the decimal point.
For ex 34.567 has a precision of 5 and a scale of 3. To declare the type:

- `NUMERIC(precision, scale)`
- `NUMERIC(precision)` specifies a scale of 0
- `NUMERIC` creates an unconstrained numeric column in which numeric values of any length can be stored, up to the implementation limits.

Note: precision must be positive

- `NUMERIC(3,1)` => round values to 1 decimal place and can store values between -99.9 and 99.9, inclusive.
- `NUMERIC(2,-3)` => will round values to the nearest thousand and can store values between -99000 and 99000, inclusive.
- `NUMERIC(3, 5)` => will round values to 5 decimal places and can store values between -0.00999 and 0.00999, inclusive.

Note: Numeric values are physically stored without any extra leading or trailing zeroes. Thus, the declared precision and scale of a column are maximums, not fixed allocations. (In this sense the numeric type is more akin to varchar(n) than to char(n).) The actual storage requirement is two bytes for each group of four decimal digits, plus three to eight bytes overhead.

Numeric types also have special values:

- 'Infinity': For ex `UPDATE table SET x = 'Infinity'`, can also be spelled 'inf'
- '-Infinity'
- 'NaN'

Infinity + Infinity = Infinity
Infinity - Infinity = NaN

In most implementations of the “not-a-number” concept, NaN is not considered equal to any other numeric value (including NaN). In order to allow numeric values to be sorted and used in tree-based indexes, PostgreSQL treats NaN values as equal, and greater than all non-NaN values.

Note: Infinity can only be stored on unconstrained NUMERIC types

Note: when rounding, numeric types ties away from 0 while real and double precision types ties to the nearest even nb

#### Floating point nb

- real: 4 bytes - 6 decimal digits precision
- double precision: 8 bytes - 15 decimal digits precision

Precision is variable. Calculation with them is inexact

- Also have -Infinity, Infinity and NaN

### Char types

- character(n) or char(n) = fixed length, blank padded
- character varying(n) or varchar(n): variable length with limit
- varchar or text: variable unlimited length

Note: error if we try to insert a too long char

```sql

CREATE TABLE tags(
 pk integer NOT NULL PRIMARY KEY,
 tag_char char(10),
 tag_varchar varchar(10),
 tag_text text
);

INSERT INTO tags VALUES (1, 'first_tag', 'first_tag', 'first_tag'), (2, 'tag', 'tag', 'tag');

SELECT pk,
length(tag_char) AS tag_char_len,
length(tag_varchar) AS tag_varchar_len,
length(tag_text) AS tag_text_len,
octet_length(tag_char) AS tag_char_octetlen,
octet_length(tag_varchar) AS tag_varchar_octetlen,
octet_length(tag_text) AS tag_text_octetlen
FROM tags;
```

```bash
pk|tag_char_len|tag_varchar_len|tag_text_len|tag_char_octetlen|tag_varchar_octetlen|tag_text_octetlen|
--+------------+---------------+------------+-----------------+--------------------+-----------------+
 1|           9|              9|           9|               10|                   9|                9|
 2|           3|              3|           3|               10|                   3|                3|

```

### Dates

```sql
SELECT * FROM pg_settings
WHERE name='DateStyle';
-- setting field shows ISO, DMY
```

To modify it, go to postgresql.conf (`SHOW config_file;`) and look for datestyle

```bash
# EXTRACT OF postgresql.conf

# - Locale and Formatting -

datestyle = 'iso, dmy'
#intervalstyle = 'postgres'
timezone = 'Europe/Paris'
#timezone_abbreviations = 'Default'     # Select the set of available time zone
```

After modification, we need to restart the service

```sql
-- to get a date
SELECT '12-30-2022'::date;
-- Note: the query above will fail in `dmy` but work in `mdy`,
-- which is why it is not great to use dates like this

-- better alternative, specify the format
SELECT to_date('12-30-2022', 'mm-dd-yyyy');


-- to output a date to the format you want
SELECT
  to_char(
    to_date('12-30-2022', 'mm-dd-yyyy'),
  'yyyy/mm/dd'
);
```

### Timestamps

Postgres has timestamps with and without timezone

```sql
-- shows the server timezone
-- also available in the postgresql.conf
SHOW TIMEZONE;
-- set the timezone
SET timezone='UTC';

CREATE TABLE zones(
  without_zone TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  with_zone TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO zones DEFAULT VALUES;

SELECT * FROM zones;

```

```bash
without_zone           |with_zone                    |
-----------------------+-----------------------------+
2024-06-01 09:37:22.760|2024-06-01 11:37:22.760 +0200|
```

I am not exactly sure how it works but it seems that with time zone always stores based
on the current session timezone. Without timezone uses the set timezone.
Indeed, if i did not change the timezone to UTC, i would have has

```bash
without_zone           |with_zone                    |
-----------------------+-----------------------------+
2024-06-01 11:37:22.760|2024-06-01 11:37:22.760 +0200|
```

### NoSQL data type

Postgres handles:

- hstore
- xml
- json/jsonb
