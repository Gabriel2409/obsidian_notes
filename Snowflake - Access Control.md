#snowflake

## Access Control

### RBAC and DAC

- Two ways:
  - RBAC: Privilege granted to a role granted to a user, similar to [[Azure - RBAC]],
  - DAC (Discretionary access control): each object has its owner who in turn can grant access to that object
    - Every securable object is owned by a single role, which can be seen when running `SHOW ...`
    - the owner can grant and remove privileges, transfer ownership to another role,..
    - Access to objects is also defined by privileges granted to roles

### Role

- A role is an entity to which privileges on securable objects can be granted (GRANT) or revoked (REVOKE): `GRANT USAGE ON DATABASE MYDB TO ROLE MYROLE;`
- roles are assigned to users and a user can switch between roles within a snowflake session.
- As roles are also securable objects, roles can be granted to other roles as well: `GRANT ROLE ROLE3 TO ROLE ROLE2`: parent roles inherit privileges of children roles

- System roles hierarchy:
  - `ORGADMIN` manages operation at an org level + `ACCOUNTADMIN`
  - `ACCOUNTADMIN` encapsulates `SECURITYADMIN` and `SYSADMIN` + certain account level privileges
- `SECURITYADMIN` manages grants globally with `MANAGE GRANTS` privilege + `USERADMIN`
- `USERADMIN` manages users and roles via `CREATE ROLE` and `CREATE USER` security privileges
- `SYSADMIN` manages all objects
- `PUBLIC` is automatically granted to users, can own securable objects but they are available to every other user and role

```text
			  ORGADMIN
			ACCOUNTADMIN
SECURITYADMIN           SYSADMIN
USERADMIN
			   PUBLIC
```

- System roles can not be dropped but we can add privileges to them. But this is STRONGLY discouraged. It is better to create custom roles instead. It is recommended to create a hierarchy of custom roles with the top-most custom role assigned to the `SYSADMIN` role so that `SYSADMIN` can manage the objects owned by the custom role

### Privilege

- Defines the level of access to a securable object
- Each object has an associated set of privileges that can be granted
- 4 catagories of security privileges
  - Global privileges: for ex create db, monitor account usage
  - Privileges for account objects, grants different access levels to objects such as db
  - Privileges for schemas: defines what you can do within a schema
  - Privileges for schema objects: defines what you can do to objects such as table
- Privileges are managed using `GRANT` and `REVOKE` (can be done by Owners and roles that have `MANAGE GRANT` privilege)
- Future grant allow privileges to be defined for objects not yet created: `GRANT SELECT ON FUTURE TABLES IN SCHEMA MY_SCHEMA TO ROLE MY_ROLE`

### Worksheet example

```sql
USE ROLE ACCOUNTADMIN;

-- get information about roles
SHOW ROLES;
SELECT "name", "comment" FROM TABLE(result_scan(last_query_id()));

-- see all privileges for a role
SHOW GRANTS TO ROLE SECURITYADMIN;

-- Create new db and schema
USE ROLE SYSADMIN;
CREATE DATABASE FILMS_DB;
CREATE SCHEMA FILMS_SCHEMA;
CREATE TABLE FILMS_SYSADMIN
(
    ID STRING,
    TITLE STRING,
    RELEASE_DATE DATE,
    RATING INT
);

-- create a new role
USE ROLE SECURITYADMIN;
CREATE ROLE ANALYST;

-- allow ANALYST role to execute USE FILMS_DB and SHOW <objects> in the db;
GRANT USAGE ON DATABASE FILMS_DB TO ROLE ANALYST;

-- allow ANALYST role to USE the schema, show <objects> in it and also create tables
GRANT USAGE, CREATE_TABLE ON SCHEMA FILMS_DB.FILMS_SCHEMA TO ROLE ANALYST;

-- Without this, no compute is possible (commands such as SHOW are still possible)
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST;

-- Make ANALYST a child role to SYSADMIN => SYSADMIN inherits every privileges of ANALYST
-- Note that it does NOT mean that a user that has the SYSADMIN role can switch to ANALYST
GRANT ROLE ANALYST TO ROLE SYSADMIN;

-- Allow the user to switch to the ANALYST role
GRANT ROLE ANALYST TO USER MYUSER;

-- shows the details of the privileges
SHOW GRANTS TO ROLE ANALYST;
SHOW GRANTS TO ROLE SYSADMIN; -- one of the row is USAGE on ROLE ANALYST

-- Future grants
USE ROLE SECURITYADMIN;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE FILMS_DB TO ROLE ANALYST;


USE ROLE SYSADMIN;
CREATE SCHEMA MUSIC_SCHEMA;
USE ROLE ANALYST;
SHOW SCHEMAS; -- MUSIC_SCHEMA is here

--Create a user and assign the role to it
USE ROLE USERADMIN;
CREATE USER USER1 password="temp" default_role = ANALYST DEFAULT_WAREHOUSE='COMPUTE_WH' MUST_CHANGE_PASSWORD=TRUE;
USE ROLE SECURITYADMIN;
GRANT ROLE ANALYST TO USER USER1;


SHOW USERS;
```