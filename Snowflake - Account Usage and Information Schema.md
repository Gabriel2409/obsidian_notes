#snowflake
## Account Usage and Information schema

- Snowflake provides a shared read only database called SNOWFLAKE, imported using a share object called ACCOUNT_USAGE
- By default, only accessible to ACCOUNTADMIN role
- 6 main schemas containing many views providing fine-grained usage metrics at the account object level
  - ACCOUNT_USAGE schema
    - views also contains dropped objects
  - CORE Schema
  - READER_ACCOUNT_USAGE schema contains views that display metadata and metrics for all the reader accounts
  - DATA_SHARING_USAGE: views for listings published in a data exchange
  - ORGANIZATION_USAGE: views containing infos for all accounts
  - INFORMATION_SCHEMA: not unique to snowflake, based on SQL-92 ANSI Information Schema
    - Views for all objects contained in db
    - Metadata for account level objects (such as roles, warehouses and databases)
    - Table functions displaying metadata for historical and usage data accros an account
    - Output of a view depends on the privileges of the user

ACCOUNT_USAGE and INFORMATION_SCHEMA are quite similar BUT:

- ACCOUNT_USAGE includes dropped objects
- INFORMATION_SCHEMA has no latency (while ACCOUNT_USAGE has between 45mins and 3 hours)
- Historical data is retained for 1 year in ACCOUNT_USAGE while it is kept from 7 days to 6 months for INFORMATION_SCHEMA