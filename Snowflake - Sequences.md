---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 226
---

#snowflake 
- see [[Snowflake - Objects]]

### Sequences

- used to generate unique numbers across sessions and statements, including concurrent statements. They can be used to generate values for a primary key or any column that requires a unique value.
- There is no guarantee that values from a sequence are contiguous (gap-free) or that the sequence values are assigned in a particular order.
- NOTE: Changing the sequence interval from positive to negative (e.g. from `1` to `-1`), or vice versa may result in duplicates

```sql
CREATE OR REPLACE SEQUENCE DEFAULT_SEQ
START = 1
INCREMENT = 2;

SELECT DEFAULT_SEQ.NEXTVAL; -- 1
SELECT DEFAULT_SEQ.NEXTVAL; -- 3

SELECT DEFAULT_SEQ.NEXTVAL, DEFAULT_SEQ.NEXTVAL; -- 5,7
SELECT DEFAULT_SEQ.NEXTVAL, DEFAULT_SEQ.NEXTVAL; -- 11,13 (There is a gap)
```

- We can use a sequence inside a table

```sql
CREATE OR REPLACE SEQUENCE TRANSACTION_SEQ
START = 1001
INCREMENT = 1;

CREATE TABLE TRANSACTIONS(ID INTEGER DEFAULT TRANSACTION_SEQ.NEXTVAL, AMOUNT DOUBLE);
INSERT INTO TRANSACTIONS(AMOUNT) VALUES(5487); -- automatically adds the ID column
INSERT INTO TRANSACTIONS(ID,AMOUNT) VALUES(12,45); -- also works and inserts 12 as an ID
```

- `GETNEXTVAL(myseq)` returns a result set, we can see it with `TABLE(...)`
- SEQUENCES can be altered: `ALTER SEQUENCE myseq SET INCREMENT = -4;`

