#sd #spark #todo

## Introduction

### Spark vs Hadoop

- DAG (Directed Acyclic Graph) execution engine that supports acyclic dataflow, which means it processes data in a way that follows a directed graph structure without any cycles.
- This design helps optimize the execution of tasks by allowing the system to determine the most efficient order in which to perform computations, leading to better performance and resource utilization.
- Spark implementation of [[MapReduce]] is more efficient than [[Hadoop]] implementation
- Spark does not come with a storage solution such as [[HDFS]] but can be uses with multiple storage solutions including local filesystem (but it is not ideal)
- Spark comes with its own standalone resource manager contrary to Hadoop that needs YARN (you can still use YARN with Spark)
- Spark can run as a standalone cluster

### Why Spark is fast

- Spark offers in memory computing at a distributed scale
- Spark DAG execution engine will first transform the instructions into a logical plan, then it creates a physical Plan and execute tasks on multiple nodes
- Spark relies on RDDs
- Spark architecture

### Challenges to solve

- Hadoop is slow for both iterative algorithms and interactive data mining which is what Spark tries to solve
- For iterative problems such as Finding the degree of separation of people in a social media graph, in Hadoop you would run several jobs, each time saving intermediary outputs to disk. On the other hand, Spark keep them in memory which makes computation faster (note that keeping everything in memory in a distributed system comes with its own complications)
- For interactive problems, Spark offers a Rich set of functions to perform different kind of operations on the dataset without having to rely on [[MapReduce]] (some problems are hard to fit in the mapreduce paradigm). Moreover, as Spark can keep **any** piece of data in memory (memory caching), the interactive process can be made faster

## RDDs

- Spark keep track of all the operations done on a dataset. If a node in the DAG (which stores Intermediate results in memory) becomes unresponsive, the transformation is done again in another node. This makes in memory computing fault tolerant by keeping it efficient because we dontneed to write to disk
- under the hood, Spark creates a RDD (resilient distributed dataset)
- Not all RDDs are kept in memory (actual execution plan != logical plan)
- Transformation functions transform a RDD to another. Spark does lazy évaluations on such transformations
- Action functions such as `collect` trigger an execution
- Dependencies
  - narrow dependency if child partition depends on entire pare t partition
  - wide dependency if it depends on a portion of parent
  - Note that you can have a narrow dependency even if you have several parents and à wide dependency even of you have only one parent
- Type of dependency determines nb of tasks (Huge role in physical execution plan): each wide dependency divided the execution plan into stages while each narrow dependencies are grouped in a process called pipelining

On executing an action in Spark, our application (for ex Spark shell) accesses a driver to retrieve a Spark context. The Spark context gives the logical plan to the DAG scheduler which will create an execution plan and pass it to the task scheduler. The task scheduler will then submit tasks to executors on different worker nodes. Then result is sent back to the driver

Caching: by default Spark caches Intermediate date (data just before a wide dependency) But not the RDD themselves. It is possible to instruct Spark to cache a RDD with `cache` or `persist`. Spark uses [[LRU Cache]]

Faut tolerance: if a node shuts down, Spark will only recompute the missing Intermediate data
