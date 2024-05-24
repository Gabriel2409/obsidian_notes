#postgres 



## use Postgres from the cmd line

To use Postgres from the command line, you must be the correct user or do `sudo -u postgres` before the command

- `createdb tuto`: creates a table called tuto
- `dropdb tuto`: drops the db
- it is also possible to add `-U user -h host` after each command
When installing postgres, a lot of commands are installed to help interact with the cluster. But the easiest thing to do is use psql.

## psql

Note: alternative: pgcli

- `psql tuto`: accesses the db tuto. If no db is specified, will access default (named postgres by default)
- `psql -h 127.0.0.1 -p 5432 -U myuser -d mydb`: connects to localhost as myuser on database mydb on port 5432. 
- with connection string: `psql postgresql://<user>@<host>:<port>/<db>`: ex `psql postgresql://postgres@127.0.0.1:5432/postgres`

NOTE: while you can pass the password in the conn string replace `<user>` with `<user>:<password>`, it is better to either type the password on connect or set the `PGPASSWORD` env var.

In case of connection refused, in PGDATA dir, check `postgresql.conf` and `pg_hba.conf` 

- Use `-s` to ask for verification after each command



- you can use any SQL command but be sure to end the commands with semi colons or `\g`
- `\h`: shows help
- `\?`: shows all commands
- `\l`: lists dbs
- `\dt`: lists tables
- `\q`: exits
- `\c tuto`: connects to base tuto
- `\i file`: executes commands from file. useful with `-s` option when connecting with psql
- `\e`: opens editor
- `\set ECHO_HIDDEN` to see the actual queries when you launch a special command with psql
- `\x`: extended mode, see more infor per record