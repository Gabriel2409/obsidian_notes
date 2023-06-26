#snowflake 

## Connectivity

### Snowsql

https://docs.snowflake.com/en/user-guide/snowsql-install-config

After installing, config file is by default in `~.snowsql/config`

In order not to have to pass credentials for each command, we need to modify the config to connect via snowsql by specifying account name, username and password.

```
[connections]
accountname = myaccountname
username = myusername
password = mypassword
```

Now if we launch `snowsql` with no parameters, we should be able to connect using these parameters as the new default

We can also specify the name of the connection:

```
[connections.example]
accountname = examplename
username = exampleuser
password = examplepassword
```

Now if we launch `snowsql -c example`, it will use the correct parameters

NOTE: As password is stored in plan text, we must be careful with permissions. More secure connection options exist

#todo, add some commands

### Connectors, drivers and partnered tools

#todo

### Snowflake scripting

#todo

### Snowpark

#todo