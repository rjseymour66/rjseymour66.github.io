+++
title = 'Robot'
date = '2026-03-20T22:33:18-04:00'
weight = 20
draft = false
+++


The Go [`database/sql`](https://pkg.go.dev/database/sql) package exposes a generic API for SQL databases. It does not connect to a database by itself. You supply a driver that implements the wire protocol for the database you use.

This article focuses on PostgreSQL and SQLite, which are common choices for services and tests.

---

## Register drivers

Drivers register in package `init` by calling:

```go
sql.Register("driverName", driver)
```

You rarely call `Register` yourself. Instead, import the driver only for its side effect (a blank import):

### PostgreSQL: pgx stdlib

The [`pgx`](https://github.com/jackc/pgx) driver registers the name `pgx` when you import `github.com/jackc/pgx/v5/stdlib`.

```go
import (
    "database/sql"

    _ "github.com/jackc/pgx/v5/stdlib"
)
```

### SQLite: modernc.org/sqlite

The [`modernc.org/sqlite`](https://pkg.go.dev/modernc.org/sqlite) package is pure Go and registers the name `sqlite`.

```go
import (
    "database/sql"

    _ "modernc.org/sqlite"
)
```

### Alternative SQLite: mattn/go-sqlite3

[`github.com/mattn/go-sqlite3`](https://github.com/mattn/go-sqlite3) uses CGO and registers `sqlite3`. You need a C toolchain. The `database/sql` surface is the same; connection string details differ.

The blank import runs the driver `init`, which registers the name. After that, `sql.Open("pgx", …)` or `sql.Open("sqlite", …)` works.

---

## Open a database with `sql.Open`

```go
db, err := sql.Open(driverName, dataSourceName)
if err != nil {
    // invalid driver name or bad Open arguments (rare)
}
defer db.Close()
```

- `driverName` must match the driver you imported: `pgx`, `sqlite`, or `sqlite3` for mattn.
- `dataSourceName` is driver-specific (URL, DSN, or file path).

### PostgreSQL examples

```go
// URL form (common with pgx)
db, err := sql.Open("pgx", "postgres://user:pass@localhost:5432/mydb?sslmode=disable")

// Keyword/value form also works with pgx
db, err := sql.Open("pgx", "host=localhost port=5432 user=user password=pass dbname=mydb sslmode=disable")
```

### SQLite examples

```go
// File on disk (modernc.org/sqlite)
db, err := sql.Open("sqlite", "file:app.db?cache=shared")

// In-memory (handy for tests); shared cache so multiple connections see one database
db, err := sql.Open("sqlite", "file:memdb1?mode=memory&cache=shared")
```

`sql.Open` returns a `*sql.DB` that represents a connection pool. It might not open a network or file handle until the first query or until you call `Ping`. The first query or `Ping` establishes a real connection.

After you open the database, verify connectivity:

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
if err := db.PingContext(ctx); err != nil {
    // cannot reach the database (wrong URL, file path, auth, and so on)
}
```

For how the pool behaves and how to tune it for PostgreSQL and SQLite, see [Connection pools](#connection-pools).

---

## Connection pools

`*sql.DB` is not a single connection. It holds a pool: a bounded set of physical connections to the server (or file, for SQLite) that the runtime creates, reuses, and recycles.

### What the pool does

1. When you call `QueryContext`, `ExecContext`, or similar methods, the pool takes an idle connection or opens a new one if you are under the limit and the driver allows it.
2. When the work finishes, the connection goes back to the pool as idle (subject to `MaxIdleConns` and idle timeouts). It is not always closed.
3. Many goroutines can share `db` safely. The pool coordinates access and hands out connections concurrently up to `MaxOpenConns`.

Create one `*sql.DB` per process (per DSN) and share it across the application. Do not open a new `sql.DB` for each HTTP request.

### Pool settings

| Setter                  | Behavior                                                                                                                                                                                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SetMaxOpenConns(n)`    | Upper bound on open connections at once (in use and idle). If the pool is exhausted, another goroutine blocks until a connection returns or the context is canceled. If `n <= 0`, there is no limit (the default).                                                     |
| `SetMaxIdleConns(n)`    | How many idle connections to keep ready. If `n <= 0` (the default), no idle connections are kept, so each operation may open a new connection (slower, fewer idle resources). Raising the value reduces reconnect cost and uses more sockets on the client and server. |
| `SetConnMaxLifetime(d)` | Close each connection after it has existed for at most `d` (from creation). If `d <= 0`, there is no limit. Use this when the database sits behind a load balancer, credentials rotate, or you need to pick up DNS changes.                                            |
| `SetConnMaxIdleTime(d)` | Close idle connections older than `d`. If `d <= 0`, there is no idle-time limit (idle connections still follow `MaxIdleConns` and max lifetime). Helps drop unused connections during quiet periods.                                                                   |

These settings interact. For example, a low `MaxOpenConns` under high traffic increases wait time. A low `MaxIdleConns` increases latency from reconnects. A short `ConnMaxLifetime` reduces stale connections but adds churn.

### Observe the pool with `Stats`

Use `db.Stats()` (`sql.DBStats`) in metrics or when you debug:

- `WaitCount` and `WaitDuration` record how often goroutines waited for a free connection. That usually means `MaxOpenConns` is too low or the database is slow.
- `MaxOpenConnections`, `OpenConnections`, `InUse`, and `Idle` describe the current pool.
- `MaxIdleClosed` and `MaxLifetimeClosed` count connections closed because of idle or lifetime limits.

If wait time spikes, raise `MaxOpenConns` or optimize queries. Do not create a new `sql.DB` per request.

### PostgreSQL tuning

- The server enforces `max_connections`. For each deployment, keep (instances × `MaxOpenConns`) well below what PostgreSQL and tools such as PgBouncer can support.
- A practical starting range is roughly 25 to 100 open connections per process, depending on load. Watch `WaitDuration` and `pg_stat_activity`. Raise the limit until waits fall or you hit CPU or `max_connections`.
- `ConnMaxLifetime` is often 30 minutes to one hour in cloud or managed setups so connections rotate cleanly. Align it with load balancer and failover behavior.
- `MaxIdleConns` is often set to a fraction of `MaxOpenConns` (for example half, or equal) so bursts avoid paying full reconnect cost. Do not set idle higher than the server can support.

If you use PgBouncer in transaction pooling mode, lifetime and idle behavior interact with how long the client holds connections. Read the PgBouncer documentation together with these settings.

### SQLite tuning

SQLite allows one writer per database file. Many readers can run at once, but writes are serialized. The `database/sql` pool can still open multiple connections to the same file (depending on the driver and `cache=shared`), which can raise the chance of `database is locked` when many goroutines write.

- `SetMaxOpenConns(1)` is a common choice for write-heavy or mixed read/write workloads. One connection serializes use and matches how SQLite works. Throughput is limited but behavior is predictable.
- For read-mostly workloads, defaults or a small `MaxOpenConns` may be enough.
- `SetConnMaxLifetime` matters less for a local file unless you change paths; it still helps if tests leak connections over a long run.
- With `MaxOpenConns(1)`, `MaxIdleConns` is usually 0 or 1. A large idle set rarely helps.

Use a shared-cache DSN for in-memory databases when multiple connections must see the same data (see [Open a database with `sql.Open`](#open-a-database-with-sqlopen)).

### Practices to avoid

- Creating a new `sql.DB` per request undermines pooling. Share one `db`.
- Leaving `MaxOpenConns` unlimited on PostgreSQL can exhaust `max_connections` or overload the server. Set an explicit cap.
- Ignoring context cancellation. Blocked acquires respect `ctx`; set timeouts on queries.
- Running very long transactions. They hold a connection and reduce pool capacity. Keep transactions short.

---

## PostgreSQL and SQLite SQL differences

| Topic                   | PostgreSQL                                                                | SQLite (Go drivers)                                                               |
| ----------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Positional placeholders | `$1`, `$2`, …                                                             | `?` (each placeholder is one argument)                                            |
| Auto-increment keys     | `SERIAL`, `BIGSERIAL`, or `GENERATED … AS IDENTITY`; often `RETURNING id` | `INTEGER PRIMARY KEY AUTOINCREMENT`; `LastInsertId()` or SQLite 3.35+ `RETURNING` |
| Booleans                | Native `BOOLEAN`                                                          | `INTEGER` 0/1 is common                                                           |
| Concurrent writers      | Many connections                                                          | Single writer; tune the pool or use `SetMaxOpenConns(1)`                          |

The `database/sql` calls are the same; only the SQL text and connection strings change. If you support both engines in one codebase, you can maintain two query shapes, add a thin abstraction, or use a library that normalizes placeholders.

Placeholder example:

```go
// PostgreSQL
db.QueryRowContext(ctx, `SELECT url FROM links WHERE key = $1`, key)

// SQLite
db.QueryRowContext(ctx, `SELECT url FROM links WHERE key = ?`, key)
```

---

## Main types and operations

### `sql.DB`

Represents a pool of connections and is safe for concurrent use. See [Connection pools](#connection-pools). Methods include `Ping` and `PingContext`, plus `SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`, and `SetConnMaxIdleTime`.

### `sql.Tx`

A transaction from `Begin` or `BeginTx`. Call `Commit` or `Rollback`.

### `sql.Rows`, `sql.Row`, and `sql.Result`

- `Rows` comes from `Query`. Always call `defer rows.Close()` and check `rows.Err()` after the loop.
- `Row` comes from `QueryRow` for a single row.
- `Result` comes from `Exec`. It exposes `LastInsertId` and `RowsAffected` when the driver supports them (SQLite often supports `LastInsertId` well).

### Core operations

| Method            | Typical use                        |
| ----------------- | ---------------------------------- |
| `ExecContext`     | `INSERT`, `UPDATE`, `DELETE`, DDL  |
| `QueryContext`    | `SELECT` returning many rows       |
| `QueryRowContext` | `SELECT` returning one row         |
| `PrepareContext`  | Same statement executed many times |

Prefer methods that take `context.Context` so you can cancel work and set deadlines.

---

## Common patterns

### Single row

PostgreSQL (`$1`):

```go
var url string
err := db.QueryRowContext(ctx, `SELECT url FROM links WHERE key = $1`, key).Scan(&url)
```

SQLite (`?`):

```go
var url string
err := db.QueryRowContext(ctx, `SELECT url FROM links WHERE key = ?`, key).Scan(&url)
```

```go
if err == sql.ErrNoRows {
    // not found
} else if err != nil {
    // query error
}
```

### Many rows

Same structure; only the placeholder style differs (`$n` versus `?`).

### Insert

PostgreSQL:

```go
_, err := db.ExecContext(ctx,
    `INSERT INTO links (key, url) VALUES ($1, $2) ON CONFLICT (key) DO NOTHING`,
    key, url,
)
```

SQLite supports a similar idea with different upsert syntax. Check your SQLite version for `ON CONFLICT`.

### Transaction

The API is the same for both engines:

```go
tx, err := db.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelReadCommitted})
if err != nil {
    return err
}
defer tx.Rollback()

if _, err := tx.ExecContext(ctx, `/* ... */`); err != nil {
    return err
}
return tx.Commit()
```

### Null columns

Use `sql.NullString`, `sql.NullInt64`, `sql.NullTime`, and related types for nullable columns.

---

## Summary

1. Import `_ "github.com/jackc/pgx/v5/stdlib"` and/or `_ "modernc.org/sqlite"`.
2. Call `sql.Open("pgx", postgresURL)` or `sql.Open("sqlite", fileURL)`. You get a pool, not a single connection.
3. Tune the pool for your engine. On PostgreSQL, cap open connections against `max_connections`. On SQLite, `SetMaxOpenConns(1)` is common when writers contend.
4. Call `PingContext` to verify that the database is reachable.
5. Use `ExecContext`, `QueryContext`, and `QueryRowContext` for work. Use `BeginTx` when several statements must commit or roll back together.
6. Use PostgreSQL placeholders (`$1`) or SQLite placeholders (`?`) in SQL strings as required.

---

## See also

- [database/sql](https://pkg.go.dev/database/sql) (includes `DB.SetMaxOpenConns`, `DB.Stats`, and related APIs)
- [Go database/sql tutorial](https://go.dev/doc/database/open-and-manage) (connection pooling)
- [pgx stdlib and database/sql](https://github.com/jackc/pgx/wiki/Getting-started-with-pgx-through-database-sql)
- [modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite)
