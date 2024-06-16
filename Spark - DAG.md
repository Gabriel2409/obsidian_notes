---
reviewed: 2023-09-05
---

#spark 

## What is a Spark DAG?

- **Directed Acyclic Graph (DAG)**: In the context of Spark, a DAG is a graph data structure consisting of vertices and directed edges. It's called "acyclic" because there are no loops or cycles in the graph.
- **Logical Execution Plan**: The DAG represents the logical execution plan of a Spark application, detailing how data is transformed from its source to the desired output.
- **Transformations and Dependencies**: Each vertex in the DAG represents a [[Spark - RDD|RDD]] or a stage (a set of transformations), and edges indicate dependencies between them. Edges show which RDDs are used as inputs to produce other RDDs through transformations.

Example:
On executing an action in Spark, our application (for ex Spark shell) accesses a driver to retrieve a Spark context. The Spark context gives the logical plan to the DAG scheduler which will create an execution plan and pass it to the task scheduler. The task scheduler will then submit tasks to executors on different worker nodes. Then result is sent back to the driver

## Key Components of Spark DAG:

- **Vertices (RDDs or Stages)**:
  - Vertices in the DAG represent either RDDs or stages of computation.
  - RDD vertices typically correspond to data partitions that are generated through transformations or read from external data sources.
- **Edges (Dependencies)**:
  - Edges in the DAG represent the dependencies between RDDs or stages.
  - There are two types of dependencies: Narrow and Wide.
    - **Narrow Dependencies**: each partition of the parent RDD is used by at most one partition of the child RDD.
    - **Wide Dependencies**: These dependencies occur when multiple child partitions depend on the same parent partitions. They may require shuffling of data across the cluster.
    - Note that you can have a narrow dependency even if you have several parents and a wide dependency even of you have only one parent

## How Spark DAG Works:

- **Transformation Planning**:
  - When Spark applications are executed, the sequence of transformations and actions is translated into a DAG.
  - Spark's Catalyst optimizer and Tungsten execution engine analyze the DAG and optimize it for performance. They can reorder transformations and minimize data shuffling, reducing the overall execution time.
- **Stages**:
  - The DAG is divided into stages separated by **wide dependencies**. Each stage represents a set of transformations that can be executed together without data shuffling.
  - Stages allow for **pipelining** narrow transformations within a stage and shuffling data only between stages, which optimizes performance.
  - Type of dependencies will determine nb of tasks. For ex, chaining multiple narrow dependencies together will create a unique task per partition even if there are several operations.
- **Execution**:
  - Spark executes the DAG in a topological order (see [[Topological sort]], starting with the stages that have no dependencies.
  - Stages are scheduled for execution based on their dependencies and data locality, if possible, to minimize data transfer across the network.
- **Caching and Checkpoints**:
  - DAG information is used to optimize the caching of intermediate data (e.g., persisted RDDs) and to determine when checkpointing is necessary for fault tolerance.
  - Checkpointing involves saving the RDD (Resilient Distributed Dataset) data to a reliable distributed file system, like Hadoop Distributed File System (HDFS) or a distributed storage system like Amazon S3, and then creating a new RDD from that persisted data

## Benefits of Spark DAG:

- **Optimization**: The DAG allows Spark to optimize the execution plan, making the best use of available resources and minimizing data shuffling.
- **Fault Tolerance**: By tracking dependencies and stages, Spark can recover lost data partitions by recomputing them from their source data.
- **Performance**: The DAG execution model, especially when combined with Spark's in-memory processing, contributes to faster data processing and improved performance.
