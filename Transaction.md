---
sr-due: 2025-09-14
sr-interval: 479
sr-ease: 229
reviewed: 2023-07-08
---

#sd

Way for an application to group several reads and writes together into a logical unit.

Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback). If it fails, the application can safely retry. No need to worry about partial failure

In most SQL databases, transactions statisfy the [[ACID]] safety guarantees.
In such dbs, everything between a BEGIN TRANSACTION and a COMMIT statement is
considered to be part of the same transaction.
=> More details on the I of ACID here: [[Transaction Isolation levels]]

Systems that do not meet the ACID criteria are sometimes called BASE, which
stands for Basically Available, Soft state, and Eventual consistency. This is even more vague than the definition of ACID.
