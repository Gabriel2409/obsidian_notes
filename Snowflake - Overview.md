#snowflake

## Overview

Snowflake is a Cloud Native Data Platform SaaS

Data platform

- data warehouse:
  - Structured and relational data
  - ANSI standard SQL
  - ACID compliant transactions
  - Data stored in databases, schemas and tables
- data lake:
  - scalable storage and compute
  - schema does not need to be defined upfront
  - Native processing of semi structured data formats
- data engineering
  - Simplify data ingestion for batch and streaming workloads (COPY INTO and Snowpipe)
  - Instantiate on the fly separate compute clusters to eliminate contention between ETL and analytic jobs
  - Native objects to create data pipelines (Tasks and Streams)
  - Data encryption at rest and in transit
- data science:
  - Have a unique centralized storage
  - Partner eco system with SageMaker, DataRobot, Dataiku...
- data sharing:
  - secure data sharing with other accounts
  - can connect to BI tools such as Tableau or PowerBI
- data applications:
  - connectors and drivers
  - UDFs and stored procedures written in sql, python, java or javascript
  - External UDFs
  - querying with Snowpark

Cloud Native

- Snowflake built from scratch for the cloud
- All infrastructure is provisionned in either AWS, GCP and Azure
- Snowflake makes use of the cloud benefits (elasticity, scalability, high availability, cost effectiveness, durability)

SaaS

- No management of hardware
- transparent updates and patches
- Pay as you use subscription model
- easy to access
- automatic optimisation (no need to specify indexes or static partitions)


## Snowflake editions

https://www.snowflake.com/pricing

- Standard
- Enterprise
- Business Critical
- Virtual Private Snowflake: here service layer is separated from other accounts
