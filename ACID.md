---
sr-due: 2023-06-28
sr-interval: 3
sr-ease: 250
---

#sd

The safety guarantees provided by [[Transaction]] are often described by the wellknown
acronym ACID, which stands for Atomicity, Consistency, Isolation, and Durability.

## Atomicity

In the context of a transaction, atomicity is the **ability to abort a transaction on error and have all writes from that transaction discarded**. It guarantees that there is no partial writes.
If a transaction groups several writes, either the full transaction is committed or it is aborted and all the writes are discarded

Note:
Atomic refers to something that cannot be broken down. For ex, in multi-threaded programming, if one thread executes an atomic operation, that means there is no way that another thread could see the half-finished result of the operation.
In the context of ACID, atomicity is NOT about concurrency (this is covered by letter I Isolation)

## Consistency:

Consistency refers to an **application-specific notion of the database being in a good state**:

- Certain statements about the data must always be true (invariants)
- If a transaction starts with a database that is valid according to these invariants, and any writes during the transaction preserve the validity, then you can be sure that the invariants are always satisfied.
- Some invariants are directly guaranteed by the database (foreign key or uniqueness constraints for ex) while others are defined at the application level

Note: Consistency is a very bad choice of word. IT does NOT refer to [[Consistency patterns]], or [[Consistent hashing]] or linearizability

## Isolation

Isolation means that **concurrently executing transactions are isolated from each other**
Most databases are accessed by several clients at the same time. If they are accessing
the same database records, you can run into concurrency problems (race conditions).

The database ensures that when the transactions have committed, the result is the same as if they had run serially (one after another), even though in reality they may have run concurrently

## Durability

Durability is the **promise that once a transaction has committed successfully, any data it has written will not be forgotten**, even if there is a hardware fault or the database crashes.
