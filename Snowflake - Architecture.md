---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 230
---

#snowflake

## Multi-cluster shared data architecture

- [[Shared disk architecture]] was the first move away from [[Shared memory architecture|Single node architecture]] by allowing compute on several machines but keeping storage in one place
- [[Shared nothing architecture]] is the next evolution: no sharing of hardware. This is what we have in HADOOP

Snowflake built a new architecture specifically for the cloud to do better than these two prevalent architectures: the multi cluster shared data architecture:

- Data Storage layer: layer where all the data is stored.
- Query processing layer: separate compute clusters executing the computation to process the queries using virtual warehouses that snowflake provisions and manages
- Cloud services layers handling everything that is not storage and compute

Note: there is also a Cloud agnostic layer here to make sure snowflake works on any cloud provider

Each layer are **physically separated** and communicate over a network via RESTful interfaces. For ex, the virtual warehouses are completely separate from the storage where we keep the long term data.
Each layer is scaled independently

![[snowflake_multicluster.png.png]]

### Storage Layer

- Blob storage of the cloud provider we deployed our account into. For AWS, it will be S3 for ex. As such we get the availability and durability guarantee this blob services offer
- Data loaded into Snowflake is organized by databases, schemas and accessible primarily as tables
- Snowflakes support structured and semi structured file formats (CSV, JSON, Avro, ORC, Parquet, XML)
- When datafiles are loaded or rows inserted into a table, Snowflake reorganizes the data into its proprietary compressed, columnar table file format (optimized for OLAP workloads)
- The data is partitioned into [[Snowflake - Micro-partitions|micro-partitions]]
- Storage is billed by how much is stored based on a flat rate per TB
- Data is not directly accessible in the underlying blob storage, only via SQL commands

### Query Processing Layer

- also referred as the compute layer
- consists of Virtual Warehouses that execute the processing tasks required to return results for most SQL statements
- A Virtual Warehouse
  - is a named abstraction for a cluster of cloud based compute instances that Snowflake manages.
  - For ex, if we execute `CREATE WAREHOUSE MY_WAREHOUSE WAREHOUSE_SIZE=LARGE;`, behind the scene, AWS would provision EC2 instances.
  - As Snowflake is a SaaS, we don't have access to the nodes, we can only interact with the named abstraction.
  - Underlying nodes cooperate in a similar way to [[Shared nothing architecture|shared nothing]] compute clusters with local caching.
  - can be created or removed instantly
  - can be paused or resumed (no billing when paused)
  - come in many sizes: small, medium, large, ... (from xs to 6xl)
- We can create virtually unlimited virtual warehouses, each with its own config. Each warehouse is isolated from each other
- All running virtual warehouses have **consistent** (see [[Consistency patterns]]) access to the same data in the storage layer. **Snowflake is NOT eventually consistent, it us strongly consistent** (strict ACID compliant processing), which is achieved thanks to the Transaction service in the Global service layer which synchronises data access.

### Services layer

- collection of highly available and scalable services that coordinate activities across ALL snowflake accounts
  - authentication and access control
  - Infrastructure management: handles creation and management of underlying cloud resources
  - Transaction management: ensures that the data warehouse is ACID compliant
  - metadata management: keeps information and stats on objects and data
  - query parsing and optimization: takes the sql query we submit and turn it into an actionnable plan for the virtual warehouse to execute
  - security
- Because the services layer is multi tenant, it allows snowflake to perform economies of scale and make features such as secure data sharing easier.
- Behind the scenes, it works on cloud compute based instances similarly to virtual warehouses

