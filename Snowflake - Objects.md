---
sr-due: 2023-06-28
sr-interval: 1
sr-ease: 230
---

#snowflake

## Snowflake Objects

- An object is something we can interact with
- Each object has its own set of privileges
- Snowflake object Model is hierarchical

At the top, we have an `Organisation` that can manage several `Account` objects.
Below we have `Account level objects` to configure different parts of the account (how many `User` exist, how many `Warehouse`, etc...).

Among them, `Database` are organized into `Schema` which are comprised of different objects: `Table`, `View`, `Stream`, ...

- [[Snowflake - Organisation and Account]]
- [[Snowflake - Database, Schema, Table, View]]
- [[Snowflake - UDFs, external functions and procedures]]
- [[Snowflake - Sequences]]
- [[Snowflake - Tasks and streams]]
