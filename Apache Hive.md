---
reviewed: 2023-09-06
---

#Sd #hadoop

- On [[Hadoop]] clusters, data is represented as files and the programming language [[Apache Pig]] does not allow us to see our data in a table structure. That is where **Apache Hive** comes in: it is a data warehouse infrastructure built on top of Hadoop, supporting multiple execution engines
- you can create table like structures for your dataset and write SQL like queries with HiveQL to analyse your dataset
- Hive will take the query and convert it into one or several jobs to submit to the cluster. For ex, with Hadoop, it would be converted to [[MapReduce]] jobs running on [[HDFS]] but Hive can work with other engines and other file systems
- Hive allows to do the same thing as [[Apache Pig]] but while Pig may be more suitable for large ETL jobs, Hive seems better suite for ad hoc analysis

Example below to calculate max closing price

```hive

### www.hadoopinrealworld ###
### Hive Queries To Compute Max Close Price By Stock Symbol ###

### CREATE EXTERNAL TABLE ###
hive> CREATE EXTERNAL TABLE IF NOT EXISTS stocks_starterkit (
exch STRING,
symbol STRING,
ymd STRING,
price_open FLOAT,
price_high FLOAT,
price_low FLOAT,
price_close FLOAT,
volume INT,
price_adj_close FLOAT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/user/hirw/input/stocks';

### SELECT 100 RECORDS ###
hive> SELECT * FROM stocks_starterkit
LIMIT 100;

### DESCRIBE TO GET MORE INFORMATION ABOUT TABLE ###
hive> DESCRIBE FORMATTED stocks_starterkit;

### CALCULATE MAX CLOSING PRICE ###
hive> SELECT symbol, max(price_close) max_close FROM stocks_starterkit
GROUP BY symbol;
```

- 2 types of tables: Managed and external
- For Managed tables, if you drop the table, the associated data is deleted
- For external tables, underlying data is not deleted on drop (preferred table type)
