---
sr-due: 2026-03-28
sr-interval: 675
sr-ease: 250
reviewed: 2023-07-14
---

#sd

## Definition

Most widely used type of [[Index]], especially in SQL databases, using [[B Tree]] (or [[B+ Tree]]) datastructure.

- B trees break the database down into fixed sized blocks or pages (traditionnaly 4kB) and read or write one page at a time
- The number of references to child pages in one page of the B-Tree is called the **branching factor**
- To update a value, you search for the leaf containing the value, update **in place** and write the page back to disk
- To add a value, you search for the page whose range encompasses the new key and, if there is not enough space, split the page into two. This ensures the tree remains balanced and access is in O(logn)

## Considerations

- Contrary to [[SSTable and LSM Tree |LSM trees]], the basic underlying write operation is to overwrite a page on disk with new data. Some operations may require to overwrite several pages. A problem may arise if a crash occurs during a write operation => B Tree typically implement a [[Write Ahead Log]]
- Additionnal complexity of updating in place is concurrency control. B Tree usually protect the tree data structure with **latches (lightweight locks)**
- Instead of modifying in place it is possible to write a new page and update the references to point to the new page

## Follow up

[[B Trees VS LSM Trees]]
