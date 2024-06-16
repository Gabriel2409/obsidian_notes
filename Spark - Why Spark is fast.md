---
reviewed: 2023-09-05
---

 #spark

- **In-Memory Processing**: Unlike [[Hadoop]] [[MapReduce]], which frequently writes intermediate results to disk, Spark caches data in memory, reducing the overhead of reading and writing to disk. This dramatically improves processing speed, especially for **iterative algorithms and interactive data analysis**.
  - For iterative problems such as Finding the degree of separation of people in a social media graph, in Hadoop you would run several jobs, each time saving intermediary outputs to disk. On the other hand, Spark keep them in memory which makes computation faster
  - For interactive problems, Spark offers a Rich set of functions to perform different kind of operations on the dataset without having to rely on [[MapReduce]] (some problems are hard to fit in the mapreduce paradigm). Moreover, as Spark can keep **any** piece of data in memory (memory caching), the interactive process can be made faster
- **Lazy Evaluation**: Spark doesn't compute transformations immediately when they are called. Instead, it builds a [[Spark - DAG|directed acyclic graph (DAG)]] of transformations and optimizes their execution. This allows Spark to skip unnecessary computations, leading to faster execution.
- **In-Memory Caching**: Spark provides the ability to cache intermediate [[Spark - RDD|RDDs (Resilient Distributed Datasets)]] in memory.
- **Persistent Storage**: Spark allows you to persist intermediate results in memory or on disk, making them readily available for subsequent computations. This persistence reduces the need for recomputation, further improving speed.
- **Data Partitioning**: Spark divides large datasets into smaller partitions and processes them in parallel across a cluster of machines.
- **Distributed Processing**: Spark distributes data and computations across a cluster of machines, enabling parallel processing (horizontal [[Scalability|scaling]])
- **Data Locality**: Spark tries to schedule tasks on nodes where the data is already stored (data locality). This reduces network overhead by minimizing data transfer across the cluster.
- **Optimized Query Execution**: Spark's Catalyst query optimizer and Tungsten execution engine analyze and optimize query plans. They make transformations like predicate pushdown, filter pushdown, and other optimizations that reduce the amount of data processed and improve query performance.
