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

This facilitates *peer authentication*. Peer authentication occurs when the current operating system username matches a valid Postgres username. The OS authenticates the user, so no password is required. Change to the postgres system user to connect:

```shell
$ sudo -u postgres psql
```

Postgres controls authentication rules in the `pg_hba.conf` (host-based authentication) file. Each line specifies a connection type, database, user, address, and authentication method. To locate the file:

```shell
$ sudo -u postgres psql -c 'SHOW hba_file;'
```

A typical `pg_hba.conf` entry looks like this:

```
# TYPE  DATABASE  USER     ADDRESS    METHOD
local   all       postgres            peer
host    all       all      0.0.0.0/0  scram-sha-256
```

*scram-sha-256* is the recommended password authentication method as of Postgres 14. It replaces the older `md5` method, which is vulnerable to replay attacks. After editing `pg_hba.conf`, reload Postgres to apply the change:

```shell
$ sudo systemctl reload postgresql
```

## Extensions

Extensions are additional features that you can add to a Postgres database. Only superusers can add extensions to a specific database:

```sql
db=# CREATE EXTENSION IF NOT EXISTS ext-name;
```

The following extensions are commonly used in production applications:

`uuid-ossp`
: Generates universally unique identifiers (UUIDs) using `uuid_generate_v4()`. Useful for distributed systems where sequential integer IDs create coordination problems.

`pgcrypto`
: Provides cryptographic functions for hashing, encryption, and random data generation. Often used for server-side password hashing.

`pg_trgm`
: Enables trigram-based text similarity matching. Powers fuzzy search features such as "did you mean?" suggestions.

`postgis`
: Adds geospatial data types and functions. Required for storing and querying location data such as coordinates and geographic boundaries.

For available extensions, see the following links:
- https://www.postgresql.org/docs/current/contrib.html
- https://www.postgresql.org/download/products/6-postgresql-extensions/

## Optimizations

You can tune database performance with the `postgresql.conf` file, or with `ALTER SYSTEM` statements. To locate the conf file, issue the following command:

```shell
$ sudo -u postgres psql -c 'SHOW config_file;'
```

The following parameters have the greatest impact on query throughput:

| Parameter | Description | Starting point |
|---|---|---|
| `shared_buffers` | Memory Postgres allocates for caching data pages | 25% of total RAM |
| `work_mem` | Memory available per sort or hash operation | 4–16 MB |
| `max_connections` | Maximum number of concurrent client connections | 100 (configure a connection pool for high concurrency) |
| `effective_cache_size` | Planner estimate of total memory available for caching | 50–75% of total RAM |

After editing `postgresql.conf`, reload the configuration without restarting the server:

```shell
$ sudo systemctl reload postgresql
```

Some parameters require a full server restart rather than a reload. To check whether a specific parameter requires a restart:

```sql
db=# SELECT name, context FROM pg_settings WHERE name = 'max_connections';
```

For guidance, see [PGTune](https://pgtune.leopard.in.ua/).

### Query analysis with EXPLAIN

`EXPLAIN ANALYZE` executes a query and returns the query plan alongside actual runtime statistics. Run it to diagnose slow queries:

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 42;
```

The output shows each step of the query plan, the estimated and actual row counts, and the time each step consumed. A *Seq Scan* (sequential scan) on a large table is a signal that an index would improve performance.

## Administration

Postgres provides *metacommands* for common administrative tasks. Enter `\?` for a full list. The following metacommands cover most day-to-day work:

| Metacommand | Description |
|---|---|
| `\l` | List all databases |
| `\c dbname` | Connect to a database |
| `\dt` | List tables in the current database |
| `\d tablename` | Describe a table's columns, types, and constraints |
| `\du` | List roles and their attributes |
| `\timing` | Toggle query execution time display |

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
```

Connect to the database with the new credentials:

```shell
$ psql --host=localhost --dbname=dbname --username=username
Password for user username:
...
username=>
```

*Schemas* provide a namespace for organizing tables within a database. By default, Postgres places all objects in the `public` schema. Creating a separate schema per application or team prevents naming collisions and simplifies permission management. To create a schema and a table within it:

```sql
CREATE SCHEMA app;
CREATE TABLE app.orders (
  id          SERIAL PRIMARY KEY,
  customer_id INT NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

To grant a user access to the schema and its tables:

```sql
GRANT USAGE ON SCHEMA app TO username;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO username;
```

## Backup and restore

*pg_dump* creates a logical backup of a single Postgres database. It exports the schema and data either as plain SQL statements or in a compressed binary format.

To back up a database as plain SQL:

```shell
$ pg_dump -U postgres dbname > backup.sql
```

To back up in the custom format, which is compressed and supports parallel restore:

```shell
$ pg_dump -U postgres -Fc dbname > backup.dump
```

To restore from a plain SQL backup:

```shell
$ psql -U postgres -d dbname < backup.sql
```

To restore from a custom format backup, run `pg_restore`:

```shell
$ pg_restore -U postgres -d dbname backup.dump
```

To back up all databases at once, including roles and tablespace definitions, run `pg_dumpall`:

```shell
$ pg_dumpall -U postgres > all_databases.sql
```
