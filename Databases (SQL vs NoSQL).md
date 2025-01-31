#sd 

## SQL

An SQL database is a relational database that is composed of tables where each row reflects a data entity and each column defines specific information about the field.
SQL Databases are stored in a [[B Tree Index| B Tree]] structure that divides the available space into fixed-size blocks called pages.


Advantages: 
- powerful querying out of the box without having to write custom code. As such, the compiler that transforms your SQL query into machine code can be optimized over time independent of business logic.
- Stronger [[ACID]] guarantees

Disadvantages:
- B-Trees are slower to write into than what NoSQLs use under the hood
- For customer experiences that do not require strong consistency, SQL databases will have much higher latency compared with NoSQL databases
- does not work well for mixed schema data: Each schema change require a migration

## NoSQL

4 types: 
- Key-value stores: stores data with unique keys. The architecture scales through the usage of sharding of data across nodes and by default is eventually consistent.
- Document databases: similar to key value stores, except that they have the ability to perform aggregate searches across data and store in a variety of formats like JSON, XML, and YAML
- Columnar databases: stores data by column rather than row. They excel at reading and querying large datasets where only a subset of columns is needed
- Graph databases: store complicated node and edge relationships. They also allow for easy graph transversal and modification without writing and maintaining your own code.

Advantages: 
- Generally speaking, NoSQL is faster for writes but slower to query. (Most use [[SSTable and LSM Tree|LSM Trees]])
- Easier sharding

Disadvantages: 
- limited in the types of efficient queries that can be done. 
- less suitable for circumstances where strong consistency is required

## Conclusion
The type of database you should use depends on what you are storing, how often you are writing, and what sorts of retrievals you need to make.

Is it important for data to have structured relationship?
- Yes: SQL
- No: Do you need strong consistency? Strong ACID guarantee?
	- Yes: SQL
	- No: NoSQL

