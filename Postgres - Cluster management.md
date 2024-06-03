---
id: Postgres - Cluster management
aliases: []
tags: []
---

#postgres

## pg_ctl

On debian distros, it is better to manage the cluster with systemctl. To use pg_ctl on docker image, connect as a non root user, for ex:
`docker exec -u postgres -it postgres bash`

- Note: needs to be connected as owner of PGDATA dir (see it with `stat -c %U $PGDATA`) as pg_ctl can't be used as root

```bash
# starts the server by launching the postmaster process
pg_ctl start
# stops the server, will exit docker container
pg_ctl stop
# status
pg_ctl status
```
