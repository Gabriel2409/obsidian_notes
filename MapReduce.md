#sd #hadoop

- see [[Hadoop]]

## Overview

- Programming model (not language) for processing large data sets
- 3 phases:
  - map phase
  - souffle phase
  - reduce phase

## Components

- Data is divided in chunks called **input splits**
- The process that work on the chunks are called **mappers**. They process one record at a time and the output is a `key:value` pair. There is 1 mapper per input split
- Note that an input split can start on a given block (see [[HDFS]]) and end on another: they respect logical records boundaries
- In the reduce phase, the output of the mappers are grouped by keys and passed to the **reducer**: the reducer will receive a list of key, each associated with a list of values. Then for each key, the reducer can output a unique value by doing an operation on the list of values (min, max, sum...). Contrary to **mappers**, the nb of **reducers** is controlled by the user
- The process in which the map output is transferred to the reducer is known as a **shuffle**. When you have more than one reducer, you can have the same key in multiple input split and you want to make sure that the same key always reach the same reducer. To do so, in the map phase, each key is assigned to a partition (there is 1 partition per reducer). Hadoop guarantees that inputs to the reducer are sorted by key. Sorting happens in the map phase and when you merge them together to have the correct nb of partitions, merging must maintain sort order

- Note: after the map phase, you can have an Optional combine phase to only limit the nb of values to pass to the reducer. A combiner is like a mini reducer that works at the map phase
- You can write MapReduce programs in python, scala, java,... but you can also use [[Apache Pig]] which was designed specifically for mapreduce jobs on hadoop clusters
