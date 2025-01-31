---
reviewed: 2023-07-14
sr-due: 2024-09-08
sr-interval: 3
sr-ease: 250
---

#sd 

Database architecture pattern used to horizontally partition data across multiple database instances. Main goal is to improve [[Scalability]] and performance by distributing data across multiple servers, which allows a system to handle larger datasets and higher transaction volumes.

- Horizontal partitioning: Each shard (or partition) contains a subset of the rows from a table. Each shard has the same schema, but the data in each shard is different. Unlike vertical partitioning, which splits a table into different columns across different databases, horizontal partitioning (sharding) splits the data rows.
- Shard and shard key: A shard is a single partition or database instance that holds a portion of the overall dataset. A shard key is a specific column (or a combination of columns) in a table that is used to determine which shard a particular piece of data should reside in
- Data is distributed across multiple shards based on the shard key
- Sharding allows a database to scale horizontally by adding more shards as the amount of data or traffic increases
- sharding can reduce the load on any single database instance


### Challenges
    
- Complexity: handling data distribution, rebalancing shards, and ensuring data consistency.
- Joins and Aggregations: Queries that need to join data or perform aggregations across shards can be more complex and may require special handling.
- Data Rebalancing: As the dataset grows, it might be necessary to rebalance or redistribute data across shards, which can be operationally challenging.
- Single Shard Overload: If the shard key is not chosen well, one shard might end up with a disproportionate amount of data or traffic, leading to performance bottlenecks.

### **Use Cases for Sharding:**

- **Large Scale Applications:** Sharding is often used in applications that handle large volumes of data 
- **Geographical Distribution:** Sharding can also be used to distribute data geographically
- **High-Volume Transactional Systems:** Systems that handle high volumes of transactions, such as payment processing systems, often use sharding to distribute the load across multiple servers