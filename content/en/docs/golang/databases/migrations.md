---
title: "Migrations"
weight: 60
description: >
  Automate schema management, statement and query execution,
---

A database migration is a version-controlled database automation workflow that creates and reverses changes to a database schema. Mirgration files come in pairs and contain database statements that create and delete objects in the database: the `up` files create, the `down` files delete. For example, an `up` file might create a table, and the `down` file might drop the table.

## Install

In Go, you can manage PostreSQL database migrations with the [migrate](https://github.com/golang-migrate/migrate) tool. The following command downloads the binary, then extracts the archive file into your current directory:

```shell
$ curl -L https://github.com/golang-migrate/migrate/releases/download/v4.14.1/migrate.linux-amd64.tar.gz | tar xvz
```

Next, move it in your path. This command places it with your other Go binaries:

```shell
$ mv migrate.linux-amd64 $GOPATH/bin/migrate
```

## Create migration files

Create a pair of migrations files that are numbered sequentially, use the `.sql` extension, and are stored in the `migrations` directory. If there is not a `migrations` directory, this command creates one. Give the migration files a descriptive name, such as `create_x_table`:
```shell
$ migrate create -seq -ext=.sql -dir=./migrations <migration-file-name>
```

## Execute migrations

Execute migrations with the following syntax:

```shell
$ migrate -path=<migrations directory> -database=<db-dsn> <migration-command>
```

For example, the following command runs the `up` migrations using an environment variable:
```shell
$ migrate -path=./migrations -database=$DB_DSN up
```

The following code block describes common migration commands:

```shell
# view the sequence number of the most recently executed migration file
$ migrate -path=<migrations directory> -database=<db-dsn> version
# roll back to a specific migration version
$ migrate -path=<migrations directory> -database=<db-dsn> goto <version-number>
# roll back all migrations
$ migrate -path=<migrations directory> -database=<db-dsn> down
```

## View migrations in the database

Migrations are traced in the `schema_migrations` file in the database. This file tracks the version and whether the migrations were cleanly applied, indicated in the `dirty` column:

```sql
=> \dt
                List of relations
 Schema |       Name        | Type  |   Owner    
--------+-------------------+-------+------------
 public | movies            | table | greenlight
 public | schema_migrations | table | greenlight
(2 rows)

=> select * from schema_migrations;
 version | dirty 
---------+-------
       2 | f
(1 row)
```
The `version` column displays `2` because the `movies` database uses sequential numbering and contains 2 migration files. These two migration files executed cleanly, so `dirty` displays `false`.

When you rollback all migrations, any database objects are reverted to their original state, and the `schema_migrations` table is empty:

```shell
$ migrate -path=<migrations directory> -database=$DB_DSN down
```
```sql
=> \dt
                List of relations
 Schema |       Name        | Type  |   Owner    
--------+-------------------+-------+------------
 public | schema_migrations | table | greenlight
(1 row)

=> select * from schema_migrations;
 version | dirty 
---------+-------
(0 rows)
```

## Troubleshooting

### DSN issues

If the database user and the database share a name, you have to add the `search_path` parameter to your DSN:

```shell
export POSTGRESQL_URL='postgres://postgres:password@localhost:5432/example?sslmode=disable&search_path=public'
```

For details, refer to the [PostgreSQL tutorial](https://github.com/golang-migrate/migrate/blob/master/database/postgres/TUTORIAL.md).

### Fixing errors

Errors, such as incorrect SQL syntax, might result in failed or partially applied statements. When you run the `migration...up` command, it returns the specific SQL error:

```shell
$ migrate -path=<migrations directory> -database=$DB_DSN up
error: migration failed: type "inteer" does not exist (column 13) in line 6: CREATE TABLE IF NOT EXISTS movies (
    id bigserial PRIMARY KEY,  
    created_at timestamp(0) with time zone NOT NULL DEFAULT NOW(),
    title text NOT NULL,
    year integer NOT NULL,
    runtime inteer NOT NULL,
    genres text[] NOT NULL,
    version integer NOT NULL DEFAULT 1
); (details: pq: type "inteer" does not exist)
```

If you check your `schema_migrations` table, the `dirty` column shows that the files did not execute cleanly:

```sql
=> select * from schema_migrations;
 version | dirty 
---------+-------
       1 | t
(1 row)
```

You cannot rerun the migration, or you receive an error:
```shell
$ migrate -path=<migrations directory> -database=$DB_DSN up
error: Dirty database version 1. Fix and force version.
```

To correct the error, use the `force` command to update the `schema_migrations` table to the correct version:

```shell
$ migrate -path=<migrations directory> -database=$DB_DSN force 1
```

`schema_migrations` now shows that the files executed cleanly:

```sql
=> select * from schema_migrations;
 version | dirty 
---------+-------
       1 | f
(1 row)
```

If you try to run the migration files again, you will receive errors. So, you should roll back the database and run the files again.

## Links

- [Decoupling migrations](https://pythonspeed.com/articles/schema-migrations-server-startup/)