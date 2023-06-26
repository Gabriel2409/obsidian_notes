#snowflake

## Billing

Note: snowflake credits are a billing unit of measure for compute resources

- On demand (Pay as you go with min 25$) VS Pay for usage upfront (lower rate)
- There are 5 main areas where an account is billed:
  - **Virtual warehouse Service**: operations that use user managed virtual warehouses
    - Uses snowflake credits
    - credit calculated based on size of warehouse, nb of cluster on a per second basis while warehouse is in a started state (min of 60 seconds)
  - **Cloud Services**: includes operations that don't use virtual warehouse but still needs compute (for ex: executing a `SHOW` or `DESCRIBE` command)
    - Uses snowflake credits (4.4 credits per hour)
    - Only cloud services that exceed 10% of daily usage of compute resources are billed (this is called cloud services adjustment)
  - **Serverless Services**: where snowflake spins compute
    - Uses snowflake credits
    - Each serverless feature has its own credit rate
    - Composed of both compute and cloud services
    - Cloud services adjustment does not apply
  - **Data storage**: temporary holding location like a stage or long term storage like a table
    - charged in currency
    - calculated monthly based on avg nb of on disk bytes per day for database tables and internal stages
    - calculated based on flat dollar value rate per TB based on capacity or on-demand, cloud provider and region
  - **Data transfer**: transfer data out of snowflake or to another account if we change region or cloud provider
    - charged in currency
    - when moving data from one region to another
    - when unloading data from snowflake with `COPY INTO <location>`
    - when replicating data to a snowflake account in a different region or cloud provider
    - when using external function to transfer data in and out of snowflake