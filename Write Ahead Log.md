---
sr-due: 2024-06-05
sr-interval: 9
sr-ease: 248
---

#sd

**Write Ahead Log (WAL, also called redo log)** = append only file to which each modification must be written before it can be applied to the pages of the tree. The log can be used to restore the data after a crash.

The fundamental principle behind WAL is to record changes to the database in a log before applying them to the database itself.

