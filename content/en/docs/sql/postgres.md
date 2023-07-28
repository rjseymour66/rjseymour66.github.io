---
title: "Postgres"
linkTitle: "Postgres"
weight: 20
description: >
  Getting up and running with Postgres.
---

## Peer authentication

When you install Postgres, the installation process creates a postgres operating system user on your machine:

```shell
$ cat /etc/passwd | grep 'postgres'
postgres:x:128:133:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
```

This facilitates _peer authentication_. Peer authentication occurs when the current operating system user anme matches a valid application--in this case, Postgres--user name. The user is authenticated through the OS, so there are no passwords required. All you have to do is change to the postgres system user:

```shell
$ sudo -u postgres psql
```

## Extensions

Extensions are additional features that you can add to a Postgres database. Only superusers can add extensions to a specific database:

```sql
db=# CREATE EXTENSION IF NOT EXISTS ext-name;
```

For available extensions, see the following links:
- https://www.postgresql.org/docs/current/contrib.html
- https://www.postgresql.org/download/products/6-postgresql-extensions/

## Optimizations

You can tweak database performance with the `postgres.conf` file, or with `ALTER SYSTEM` statements. To locate the conf file, issue the following command:
```shell
$ sudo -u postgres psql -c 'SHOW config_file;'
```
For guidance, see [PGTune](https://pgtune.leopard.in.ua/).


## Administration

Postgres uses metacommands (`\*`). For a full list, enter `\?`.

Create a database and change to it:
```sql
postgres=# CREATE DATABASE dbname;
postgres=# \c dbname
dbname=#
```

Create a new user and then connect with that user:
```sql
dbname=# CREATE ROLE username WITH LOGIN PASSWORD 'pword';
dbname=# exit
shell
$ psql --host=localhost --dbname=dbname --username=username
$ Password for user username:
...
username=>
```