---
id: Postgres - Installation
aliases: []
tags: []
---

#postgres

## Ubuntu

see: <https://wiki.postgresql.org/wiki/Apt>
user setup: <https://gist.github.com/michaeltreat/40a2f444d8ff6c89af958733448da093>

Note: postgresql works with the user postgres by default.

After install connect with `psql`:

```bash
sudo -u postgres psql
```

Then type the psql special command

```sql
\password
# then enter password
```

### Connect to postgres

- start the postgresql server: `sudo systemctl postgresql start`
- connect to it as postgres user: `sudo -u postgres psql`
- stop: `sudo systemctl postgresql stop`

Note: postgres also comes with `pg_ctl` (type `apropos pg_ctl`) to manage the cluster.
However, in debian distributions, using systemctl is probably better

### Uninstall

- `sudo apt-get --purge remove postgresql`
- `sudo apt-get purge postgresql*`
- `sudo apt-get --purge remove postgresql postgresql-doc postgresql-common`

## Docker

Basic usage: 
```bash
docker container run 
-e POSTGRES_PASSWORD=<postgres_password> 
-e POSTGRES_USER=<postgres_user> 
--name <container_name> 
-d -p <port>:5432 postgres:13-alpine
```

### connect inside the container

- Go in the container: `docker container exec -it <container_name> bash`
- Type `psql -U <postgres_user>`

### connect outside the container with a psql client

- `psql -h <host> -p <port> -U <postgres_user>` : and then type password. More detail on [[Postgres - psql|psql]] section

## pgenv

Manages different versions of postgres
<https://github.com/theory/pgenv>
