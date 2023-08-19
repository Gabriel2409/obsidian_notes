---
sr-due: 2023-08-04
sr-interval: 38
sr-ease: 248
reviewed: 2023-07-14
---

#sd

## Definition

In a database, an Index is an additionnal structure that is derived from the primary data.

- slows down writes, because index needs to be updated when new data is written
- speeds up reads when it can be used to retrieve data

## Examples

- [[Hash Index]] is the simplest indexing strategy
- [[SSTable and LSM Tree]], usually used in NoSQL
- [[B Tree Index]], usually used in SQL databases
- Fuzzy indexes allowing to search for similar keys (usually implemented with a [[Trie]])

## Notes

- It is very common to have a secondary index. The main difference is that the keys are not unique so you can for ex append a unique row identifier to each key.
- When using an index,
  - the value can be a pointer to the location of the row (the rows are stored on the **heap file**). In this case, when updating the value, you just need to update the pointer in the heap file. Note that if the new value is larger than the one in the heap file, it is common to append the new value at the end and leave a forwarding pointer at the previous location
  - The value can also be the actual row (**clustered index**)
  - We can also use a **covering index with included columns** which only stores some of the table columns within the index
- Multi column indexes are needed when we need to query multiple columns / fields at the same time (for ex if we want to get all the places with a given range of latitude / longitude). The most common type is a concatenated index
