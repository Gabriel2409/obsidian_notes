#snowflake

- see [[Snowflake - Data loading]] and [[Snowflake - Data unloading]]

### File formats

- supported types are CSV, JSON, AVRO, ORC, PARQUET, XML
- Each type has its own set of properties related to parsing that specific format
- file format can be set on a stage directly or in the COPY INTO statement
  - If set on both, value in COPY INTO takes precedence

```sql
-- Set file format directly in an internal stage
CREATE STAGE MYSTAGE
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);

-- better way: create a file format
CREATE FILE_FORMAT MYCSVFF
TYPE='CSV'
SKIP_HEADER=1;
CREATE OR REPLACE STAGE MYSTAGE
FILE_FORMAT = MYCSVFF
```

Available semi structured file formats:

- JSON
- XML
- AVRO: binary row-based storage format originally developped for Apach HADOOP
- ORC: highly efficient binary format used to store Hive data
- PARQUET: binary format designed for projects in the HADOOP ecosystem

Note: among these, only JSON and PARQUET can be unloaded with `COPY INTO <location>`
