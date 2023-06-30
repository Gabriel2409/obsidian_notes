---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 230
---

#snowflake

see [[Snowflake - Objects]]

### Organization

- Manage one or more Snowflake accounts
- Setup and administer Snowflake features which make use of multiple accounts
- Monitor biling accross multiple accounts
- People managing the organisation have an `ORGADMIN` role.

`ORGADMIN` can create accounts, enable cross-account features, monitor account usage

By default, even if you have the `ORGADMIN` role, it is not enabled by default.

```sql
-- switch role
use role ORGADMIN;
-- show all accounts
SHOW ORGANIZATION ACCOUNTS;
```

### Account

- administrative name for a collection of storage, compute and cloud services deployed and managed entirely on a selected cloud platform
- each account is hosted on a single cloud provider
- Each account resides in a single geographic region, which has an impact on regulation, compute price, and even available snowflake features
- Each account is created as a single snowflake edition (can be changed later on)
- By default created with the system defined role `ACCOUNTADMIN`

When an account is created,

- we have a url such as `https://<locator>.<region>.<cloudprovider>.snowflakecomputing.com`, for ex: `xyz123.us-east-2.aws.snowflake.computing.com`
- and an identifier : `<organisation>-<accountname>`, for ex: `ABCDEFG-AB12345`
  Both urls and identifiers are unique.
