---
title: "MySQL"
weight: 30
description: >
  Connecting and working with MySQL and Go.
---

[golang/go SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers)

This section borrows heavily from [Tutorial: Accessing a relational database](https://go.dev/doc/tutorial/database-access). For information about transactions, query cancellation, and connection pools, see [Accessing a relational database](https://go.dev/doc/database/).

### Import the driver

1. Go to [SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers) and locate the [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql/) link.
2. In the go file that runs SQL code, import the driver. Because you are not using the driver directly, import it with the `blank identifier`:
   ```go
   package main

   import "_ github.com/go-sql-driver/mysql"
   ```
   Use the blank identifier because you do not use any methods from the `mysql` driver--you use only the driver's `init()` function so the driver can register itself with the `database/sql` package.

3. If you have not installed the package dependency, run `go get` in the shell:
   ```shell
   $ go get -u github.com/go-sql-driver/mysql
   ```
   This installs the latest version of the driver.

### Get a database connection pool

To get a database connection pool, you need to provide a database source name (DSN) to the `sql` package's `Open()` function. The `Open()` function returns a pool of many conncurrent database connections. Go opens and closes these connections through the database driver, as needed.

The DSN is different for each driver. The `mysql` driver has a `parseTime=true` option, which tells the driver to convert SQL `TIME` and `DATE` to Go `time.Time` objects.

The database connection pool should be long-lived and passed among handlers, so implement it in the `main` method. Include a flag for the `dsn` so you can easily use a different data source. In addition, include a helper function to encapsulate the database connection code:

```go
func main() {

	dsn := flag.String("dsn", "web:pass@/snippetbox?parseTime=true", "MySQL data source name")

	...

    db, err := openDB(*dsn)
    if err != nil {
        errorLog.Fatal(err)
    }
    defer db.Close()

	...
}

func openDB(dsn string) (*sql.DB, error) {
	// sql.Open(driver-name, DSN) {...}
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }
    // Connections are created lazily (as needed), so
    // ping the db to establish a connection and confirm 
    // everything is working.
    if err = db.Ping(); err != nil {
        return nil, err
    }
    return db, nil
}
```


