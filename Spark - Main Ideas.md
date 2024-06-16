---
reviewed: 2023-09-05
---

 #spark

Apache Spark is a multi-language engine for executing data engineering, data science, and machine learning on single-node machines or clusters

## Key Features

- [[Spark - Why Spark is fast|Speed]]
- **Ease of Use**: high-level APIs in Java, Scala, Python, and R
- **Versatile**: Spark supports batch processing, interactive queries, streaming data, machine learning, and graph processing
- **In-Memory Processing**: intermediate data is stored in memory
- **[[Spark - RDD|Resilient Distributed Datasets]] (RDDs)**
- [[Spark - DAG|(Directed Acyclic Graph)]] execution engine that supports acyclic dataflow, which means it processes data in a way that follows a directed graph structure without any cycles
- **Built-in Libraries**: includes libraries for SQL (Spark SQL), machine learning (MLlib), graph processing (GraphX), and stream processing (Structured Streaming).

Notes:

- Spark does not come with a storage solution such as [[HDFS]] but can be uses with multiple storage solutions including local filesystem (but it is not ideal)
- Spark comes with its own standalone resource manager contrary to Hadoop that needs YARN (you can still use YARN with Spark)
- Spark can run as a standalone cluster

## Use Cases

- **Data Processing**: ETL (Extract, Transform, Load) operations, data cleansing, and data enrichment.
- **Machine Learning**: Spark's MLlib library allows data scientists to build and deploy machine learning models at scale.
- **Real-time Data Streaming**: Spark's Structured Streaming provides real-time data processing capabilities, making it suitable for applications like fraud detection, monitoring, and recommendation systems.
- **Graph Processing**: With GraphX, Spark can analyze and process large-scale graphs efficiently. This is useful in social network analysis, network security, and recommendation systems.
- **Interactive Analytics**: Spark SQL enables users to perform interactive SQL queries on large datasets, making it useful for business intelligence and data exploration.

## Ecosystem

Spark has a rich ecosystem of tools and extensions that complement its core functionality. Some notable components include:

- **Spark Streaming**: This allows processing of real-time data streams.
- **SparkR**: An R package that provides an R API for Spark, enabling data scientists to leverage Spark's capabilities from within R.
- **PySpark**: The Python library for Spark, which has become popular for its ease of use and data science capabilities.
- **Cluster Managers**: Spark can run on various cluster managers like Apache [[Hadoop]] YARN, Apache Mesos, and its built-in standalone cluster manager.
- **Integration**: It can integrate with various storage systems like [[HDFS]], Apache Cassandra, Apache HBase, and more.
