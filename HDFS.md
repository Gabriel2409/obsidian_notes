---
reviewed: 2023-07-13
---

#hadoop #sd

## Overview

- HDFS ([[Hadoop]] Distributed [[File system]]) is an implementation of GFS (google file system)
- HDFS handles the distributed storage of data on the Hadoop cluster nodes
- Multiple servers (horizontal [[Scalability]]): data are copied on several servers, making it fault tolerant

## How it works

- When a file is sent to HDFS, it is cut in blocks (default of 128Mo) and duplicated (default replication factor of 3)

  - Note that the block size is independant of the file size. If you send a 10Mo file, it will occupy on the cluster 128Mo \* 3
  - These blocks are stored on **data nodes** which correspond to the storage of the servers (hard drives, ssd)
    - Datanodes know only the blocks they are responsible for and their location on their local file system

- File location is handled by the master server of the cluster: the **name node**
  - The name node role is to handle the location of all the files and their replications on all the blocks on the whole cluster.
  - The name node handles the metadata of the cluster (owner, creation date, ...) and persists information on disk
  - The name node stores the information on which data nodes contain which block but **NOT** the exact location of the block on the data node local file system (as the data node already knows it). This information is only stored on memory when the name node tries to reconstruct a file (never on disk)
  - The name node is the brain of HDFS, it is unique and therefore fragile
  - To make it more resilient,
    - save metadata regularly,
    - add a second name node in stand by mode that takes over in case of the named node failure
    - create a HDFS federation, where multiple independent name nodes manage different portions of the file system, distributing the metadata handling across multiple nodes.
- HDFS is optimized for sequential read and write operations rather than random access

- Note: HDFS does not replace the local file system of each node. In fact the OS does not care about HDFS ans uses the local FS. HDFS is just placed on top and allows to have a distributed view of the cluster

## Working with HDFS

### With Ambari

- go to [[Apache Ambari|Ambari]] dashboard and then click on the blue rectangles at the top then go to `Files View`
- go to the folder where you want to add the file and click `Upload`

### With ssh

- ssh into hadoop: `ssh maria_dev@localhost:2222`
- list HDFS files/folders: `hadoop fs -ls`: note that it shows the replication factor
- create HDFS folder: `hadoop fs -mkdir <foldername>`
- copy a file within hdfs: `hadoop fs -cp <file> <folder/file>`
- move a file: `hadoop fs -mv ...`
- copy a standard linux file to hdfs: `hadoop fs -copyFromLocal <file> <hdfs_folder/file>`
- copy from hdfs to standard file: `hadoop fs -copyToLocal <hdfs_folder/file> <file>`
- it seems there are some subtletites between `-get` vs `-copyToLocal` and `-put` vs `-copyFromLocal`
- change replication factor: `hdfs fs -dfs.replication=2 -cp <path> <newpath>`
- change permission: `hdfs fs -chmod 777 <file>`

### How is HDFS linked to local FS?

- to get information about how HDFS stores blocks on the local fs, you need admin rights. We can use the file system check command: `sudo -u hdfs hdfs fsck <hdfsfolder/hdfsfile> -files -blocls -locations` which shows the ip addresses of the nodes where each block is stored and give information about whether the file is heathy.
- if you go to `hdfs-site.xml` (location is decided on setup of the cluster, for ex `/etc/hadoop/conf/`), you can check the property `dfs.datanode.data.dir` To see where the files are stored in the local fs. Note that this file also co tains information of the name node location so that datanode can communicate with name node
- **The local file system can only see the blocks. HDFS known how to reconstruct the file from the blocks**
