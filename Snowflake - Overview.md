---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 230
reviewed: 2023-07-27
---

#snowflake

## Overview

Snowflake is a Cloud Native Data Platform SaaS

Data platform

- data warehouse:
  - Structured and relational data
  - ANSI standard SQL
  - [[ACID]] compliant [[Transaction]]
  - Data stored in databases, schemas and tables
- data lake:
  - scalable storage and compute
  - schema does not need to be defined upfront
  - Native processing of semi structured data formats
- data engineering
  - Simplify data ingestion for batch and streaming workloads ([[Snowflake - Bulk loading with COPY INTO table|Bulk loading]] and [[Snowflake - Continuous loading with Snowpipe|Snowpipe]])
  - Instantiate on the fly separate compute clusters to eliminate contention between ETL and analytic jobs
  - Native objects to create data pipelines ([[Snowflake - Tasks and streams]])
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
- automatic optimisation (no need to specify [[Index]] or static partitions)


## Snowflake editions

https://www.snowflake.com/pricing

- Standard
- Enterprise
- Business Critical
- Virtual Private Snowflake: here service layer is separated from other accounts
