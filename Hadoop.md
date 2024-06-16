---
reviewed: 2023-09-06
---

#hadoop 

# Overview

Framework for distributed processing of large datasets across clusters of commodity computers

## Install sandbox

- Useful links
  - https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html
  - https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html
  - Be sure to launch the deploy script with bash: https://superuser.com/questions/1478921/docker-sandbox-proxy-crashes-immediately-hdp-3-0-1
- Administer cluster with [[Apache Ambari]],
- Note: depending on the distribution administration can also be done with Cloudera Manager

## Hadoop ecosystem

![[Hadoop_overview.excalidraw]]

### At the heart of Hadoop

- [[HDFS]] (Hadoop Distributed File System) which uses named nodes and datanodes to handle very large amount of data
- [[MapReduce]] is used to parallelize a very large volume of data: even though new frameworks now exist, it is still widely used
- YARN (Yet Another Resource Negociator) allows for the execution of various distributed computing frameworks other than MapReduce on the Hadoop cluster by acting as a resource management and job scheduling layer

### Databases

- [[Apache Hive]]
- HBase:
  - NoSQL database
  - designed to handle large-scale, sparse, and structured datasets
- [[Sqoop]]:
  - software to import/export relational data
  - commonly used to import data from relational databases, such as MySQL, Oracle, SQL Server, and others, into HDFS or Hive for further processing and analysis
- MySQL: relational database
- Cassandra:
  - NoSQL database specifically created to handle large amount of data
  - distributed architecture, fault tolerance, and linear scalability
- MongoDB: NoSQL database with document oriented storage

### Programming

- [[Apache Pig]]: proposes a high level programming language (Pig Latin) designed to abstract the complexity of writing MapReduce programs
- Apache Spark: Platform proposing APIs to manipulate data of a HDFS cluster

### Real time

- [[Kafka]]: enables reliable and scalable data ingestion and streaming between systems
- Flume: more specialised to retrieve log files
- Flink: stream processing framework that provides advanced processing capabilities on data streams, including complex event processing and low-latency analytics.
- Storm
- Spark streaming: uses a micro-batch processing model where data is divided into small batches for processing

### Management

- TEZ: is coordinated with Hive and Apache Pig and can provide a graphical representation of tasks (DAG)
- Zeppelin provides a visual representation of the data treated by spark
- Zookeeper provides a configuration and coordination service for hadoop
- Mesos is there to optimize hadoop resources

## What can Hadoop do

- ETL treatments on very large amount of data (petaoctect)
  - Batch processing treatments typically lasting several hours
- Data analysis
- Real time analysis
