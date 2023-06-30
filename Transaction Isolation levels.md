#sd #todo

NOTE NOT FINISHED

Usually, [[Transaction|transactions]] in SQL databases are [[ACID]] compliant.

Databases support various isolation levels (letter I) providing different level of guarantees.
Usually stronger guarantees come at a performance cost

## Problems with concurrent transactions

### Dirty reads

Happens when Transaction includes a series of write operations and another Transaction reads the values written by the first Transaction before it is **committed**

### Dirty writes

Happens when there are two Transactions concurrently writing to the database to the same set of rows and if T1 executes write operations followed by T2 overwriting the values written by T1 before it is **committed**

### Non repeatable reads (Read skew)

Happens when, within a transaction, we read the same rows(s) multiple times and get different results for those rows.
For ex we have two identical subqueries retrieving a field for a given row but by the time the second subquery is executed, the value was modified by another transaction

### Lost update problem and write skew

Classic problem statement where the client reads some value from the database, modifies it, and updates it back to the database via different database Transactions. This is commonly referred to as the Read-Modify-Write cycle.

For ex, client A sees value 1 in a transaction for field Counter. Then after some processing it wants to increase the counter, so he writes 2 to the Counter field.
However at the same time client B also saw value 1 and wants to increase the counter. Final value will be 2 and not 3 (we lost the update).

Write skew is a generalization of the lost update problem where a client takes a decision based on out of date data and writes back the result

### Phantom reads

Happens when a transaction reads objects that match some search condition. Another client
makes a write that affects the results of that search. Running the same query will return different rows.
Phantom reads can happen even if you prevent non repeatable reads

Example:
Transaction 1 is launched.
Transaction 2 is launched.
Transaction 2 wants to get all dates after 2022
Transaction 1 adds a new date
Transaction 1 commits
Transaction 2 wants to get all dates after 2022 again: it gets a new result.

## Isolation level

### Read Uncommitted

- Lowest guarantees
- Can be useful in certain cases when performance is very low
- Does not protect against any of the problems mentioned above

### Read Committed

- Usually default isolation level
- Prevents dirty reads and dirty writes

#### Preventing dirty read

- Read Committed maintains two versions of a value: the committed value and the value being modified by a write transaction
- Read requests only read the committed value

#### Preventing dirty writes

- To write a value, a transaction must acquire a lock on the rows. Lock is released when the transaction commits/aborts
- This lock does not block reads but it prevents other transactions from writing

### Repeatable Read

- Also called Snapshot Isolation
- Prevents non repeatable reads

#### Preventing non repeatable reads

- Snapshot Isolation uses Multi Version Concurrency Control: the database maintains **multiple versions** of the same row at a given point in time, due to multiple concurrent transactions.
- When a transaction is started, it gets an increasing identifier which is used to tag created_by and deleted_by fields on a row, which indicates the **Transaction** that created this row and deleted this row respectively.
- That way a transaction will not read a value that was committed after it has started
- Since the database maintains **multiple versions** of the rows at a given point in time, periodically the database management system runs a Garbage Collection process and removes deleted and unreferenced entries from the table freeing up space.
