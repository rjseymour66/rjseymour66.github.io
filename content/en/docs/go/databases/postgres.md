---
title: "PostgreSQL"
weight: 40
description: >
  Connecting and working with PostgreSQL and Go.
---
Links:

- [Official documentation](https://www.postgresql.org/docs/current/datatype.html)
- [Blog post: CHAR(x) vs. VARCHAR(x) vs. VARCHAR vs. TEXT ](https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/)

## Import the driver

1. Go to https://github.com/lib/pq
2. Go to the root of the project, and use `go get` to download the driver:
   ```shell
   $ go get github.com/lib/pq@v1
   ```
3. In the Go file that runs the SQL code, import the driver. Because you are not using the driver directly, import it with the `blank identifier`:
   ```go
   package main

   import "_ github.com/lib/pq"
   ```

## Get a database connection pool

The Postgres data source name (DSN) uses the following format:

```
postgres://<username>:<password>@<host>/<dbname>
```

If you set the DSN as an environment variable, you can use it to authenticate to the database:

```shell
$ psql $POSTGRES_DSN_ENV_VAR
```


## Viewing databases

PostgreSQL uses meta commands, such as `/dt <table-name>`.


## TODO 

`pq.Array()` method

## Array operators and functions

You can add logic to your queries with [array functions and operators](https://www.postgresql.org/docs/9.6/functions-array.html).

For example, the `@>` operator checks whether an array contains a value. The following expression returns `true` if the `genres` array contains the value that the `$2` placeholder represents, or if the placeholder is empty:

```sql
(genres @> $2 OR $2 = '{}')
```