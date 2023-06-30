---
sr-due: 2023-06-29
sr-interval: 1
sr-ease: 230
---

#snowflake

## Network policies

- provides the user with the ability to allow or deny access to their Snowflake account based on IP address
- Network policies only support ipv4
- we use the CIDR notation to express an IP subnet range
- Network policies can be applied on the account level or to individual users (user level policy takes precedence)
  - Only one network policy can be associated with an account at any one time
  - Only one network policy can be associated with a user at any one time
- Policy can be bypassed but you need to contact snowflake support
- Policies can be applied by `SECURITYADMIN` or any role with the `ATTACH POLICY` global privilege

```sql
-- creating a network policy
CREATE NETWORK POLICY MY_POLICY
ALLOWED_IP_LIST = ('192.168.1.0/24') -- if specified, all addresses not in the list are blocked
BLOCKED_IP_LIST = ('192.168.1.99'); -- additional blocked ips within allowed range

-- apply policy to acount / user
ALTER ACCOUNT SET NETWORK_POLICY MY_POLICY;
ALTER USER USER1 SET NETWORK_POLICY MY_POLICY;

-- see list of policies
SHOW NETWORK POLICIES;

-- See applied policies
SHOW PARAMETERS LIKE 'MY_POLICY' IN ACCOUNT;
SHOW PARAMETERS LIKE 'MY_POLICY' IN USER USER1;
```