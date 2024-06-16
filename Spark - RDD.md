---
reviewed: 2023-09-05
---

#spark 

Resilient Distributed Datasets, commonly known as RDDs, are a fundamental data structure in Apache [[Spark]]. RDDs are the building blocks that enable distributed data processing with fault tolerance. They offer a simple and efficient way to work with large datasets across a cluster of machines.

## Characteristics of RDDs

1. **Resilient**: RDDs are resilient because they can recover from node failures. If a partition of an RDD is lost due to a node failure, Spark can recompute it automatically based on the original data and the operations used to create it (Spark tracks the lineage = all operations done on a dataset)
2. **Distributed**: RDDs are distributed across multiple nodes in a Spark cluster. Each partition of an RDD can be processed on a separate node, enabling parallelism.
3. **Immutable**: RDDs are immutable, meaning once created, they cannot be modified. Instead, any transformation on an RDD creates a new RDD.
4. **Lazy Evaluation**: Transformations on RDDs are lazily evaluated, which means they are not executed immediately. Instead, Spark builds a [[Spark - DAG|DAG]] of transformations and only computes the result when an action is called. This allows Spark to optimize the execution plan. Action functions such as `collect` trigger an execution
5. **In-Memory**: RDDs can be stored in memory, allowing for much faster data processing compared to disk-based systems like [[Hadoop]] [[MapReduce]]. Not all RDDs are kept in memory

- Note on Caching: by default Spark caches Intermediate data (data just before a wide dependency) But not the RDD themselves. It is possible to instruct Spark to cache a RDD with `cache` or `persist`. Spark uses [[LRU Cache]]
- Note on Faut tolerance: if a node shuts down, Spark will only recompute the missing Intermediate data
