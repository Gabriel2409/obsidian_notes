---
sr-due: 2023-06-30
sr-interval: 1
sr-ease: 230
---

#snowflake

## Data Loading and unloading

- [[Snowflake - Direct loading]]
- [[Snowflake - Stage]]
- [[Snowflake - File format]]
- [[Snowflake - Semi-structured data]]
- [[Snowflake - Bulk loading with COPY INTO table]]
- [[Snowflake - Continuous loading with Snowpipe]]
- [[Snowflake - Loading semi-structured data]]

### Best practices for data loading

- Split files into multiple files of 100-250 MB of compressed data because a single server in a warehouse can process up to 8 files in parallel
- Organize data by path
- Snowflake recommends having separate warehouses for data loading tasks and other tasks
- Pre-sorting data creates a natural clustering of data so that data in micro partitions are more evenly distributed and improves pruning
- Not recommended to load more than 1 file per minute
