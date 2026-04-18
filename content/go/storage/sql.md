+++
title = 'SQL'
date = '2026-03-20T21:47:23-04:00'
weight = 10
draft = false
+++

## SQL databases 

Go has two packages to interact with a SQL db:
- `database/sql`: API to interact with a various dbs
- `database/sql/driver`: Defines behaviors that db drivers must implement


## Registering a driver

### Download the driver

```bash
go get modernc.org/sqlite@latest        # most recent stable release
go get modernc.org/sqlite@v1.38.0       # specific version
go get github.com/mattn/go-sqlite3      # requires C bindings (CGO_ENABLED=1 go build)
```

### Import the driver

Use the blank import to tell the compiler that you won't use the driver package name `sqlite` in this file:

```go
// project/sqlite/sqlite.go
package sqlite

import (
	_ "modernc.org/sqlite"
)
```

If you don't use the blank import, you get an "imported and not used" error. This is because Go executes any `init` function in the package upon import, and all drivers have an `init` function.

After you import the driver, `go.mod` lists the driver dependency as an indirect dependency. Clean up the dependency tree:

```go
go mod tidy
```


### Open a connection pool

The `Dial` function connects to a database and opens a connection pool. After you open a connection to a database, you need to maintain it throughout the program's life so you don't have to reconnect for each query that you send:
1. The DSN is a data-source name, which specifies how to connect to the database.
1. `Open` returns a `*sql.DB` handle to interact with the database. It opens an empty connection pool and takes the following arguments:
   - Driver name
   - Driver-specific connection string
2. `PingContext` connects to the database, adding the first connection. 

```go
func Dial(ctx context.Context, dsn string) (*sql.DB, error) { 	// 1
	db, err := sql.Open("sqlite", dsn) 							// 2
	if err != nil {
		return nil, fmt.Errorf("opening: %w", err)
	}
	if err := db.PingContext(ctx); err != nil { 				// 3
		return nil, fmt.Errorf("pinging: %w", err)
	}
	return db, nil
}
```

Here are some DSN examples:

```go
db, err := sql.Open("sqlite", "file:links.db")
db, err := sql.Open("postgres", "dbname=links sslmode=disable")
```

The `*DB` returned manages a connection pool, including connecting, reconnecting, closing connections, etc.


#### Optimizing the connection pool

| Setting            | Description                                                                 | Default Behavior                              |
| ------------------ | :-------------------------------------------------------------------------- | :-------------------------------------------- |
| SetMaxOpenConns    | Maximum number of open (in-use + idle) connections allowed to the database. | Unlimited (0 = no limit)                      |
| SetMaxIdleConns    | Maximum number of idle (unused but open) connections kept in the pool.      | 2 (or 0 depending on Go version)              |
| SetConnMaxLifetime | Maximum total time a connection can be reused before being closed.          | Unlimited (0 = connections live forever)      |
| SetConnMaxIdleTime | Maximum amount of time a connection can remain idle before being closed.    | Unlimited (0 = idle connections never expire) |

## File embedding

File embedding lets you include the file's content in a variable so it is available in the final compiled binary. This means that you can distribute your program with the schema included:
1. Import the `embed` package with a blank import.
2. Add the embed directive to tell the compiler to embed the `schema.sql` file in the `schema` variable.
3. Create the schema on the database with `DB.ExecContext`. This function grabs a connection from the pool, executes the query, then returns the connection to the pool.

```go
import (
	_ "embed"                                                           // 1
    ...
	_ "modernc.org/sqlite"
)

//go:embed schema.sql                                                   // 2
var schema string

func Dial(ctx context.Context, dsn string) (*sql.DB, error) {
	db, err := sql.Open("sqlite", dsn)
	if err != nil {
		return nil, fmt.Errorf("opening: %w", err)
	}
	if err := db.PingContext(ctx); err != nil {
		return nil, fmt.Errorf("pinging: %w", err)
	}
	if _, err := db.ExecContext(ctx, schema); err != nil {              // 3
		return nil, fmt.Errorf("applying schema: %w", err)
	}
	return db, nil
}
```

## Service connection

This service connects to the database and calls a link shortening package to save the `link` type to the database:


### Service definition

Create a service object that holds a database connection. The constructor should return a new `Shortner` service:

```go
type Shortener struct {
	db *sql.DB
}

func NewShortener(db *sql.DB) *Shortener {
	return &Shortener{db: db}
}
```

### Insert in database

This method takes a link, shortens it, then stores the original link and shortened link in a database:
1. `ExecContext` inserts a link into the database. It will cancel the database operation if context is canceled. For example, during an HTTP request.
   This takes a context, query string, and any amount of arguments to inject in the placeholder variables in the query string.


```go
func (s *Shortener) Shorten(ctx context.Context, lnk link.Link) (link.Key, error) {
	var err error
	if lnk.Key, err = link.Shorten(lnk); err != nil {
		return "", fmt.Errorf("%w: %w", err, link.ErrBadRequest)
	}

	// Persist the link in the db
	_, err = s.db.ExecContext( 															// 1
		ctx,
		`INSERT INTO links (short_key, uri) VALUES ($1, $2)`,
		lnk.Key, base64String(lnk.URL),
	)
	if isPrimaryKeyViolation(err) {
		return "", fmt.Errorf("saving: %w", link.ErrConflict)
	}
	if err != nil {
		return "", fmt.Errorf("saving: %w: %w", err, link.ErrInternal)
	}
	return lnk.Key, nil
}
```

#### Key validator helper

This uses a helper function in `sqlite/sqlite.go` to test that the primary key is valid by identifying duplicate keys or other conflicts:

```go
func isPrimaryKeyViolation(err error) bool {
	var serr *sqlite.Error
	if errors.As(err, &serr) {
		return serr.Code() == 1555
	}
	return false
}
```

### Retrieve from database

This function takes a key for the shortened link and returns the original link:
1. `QueryRowContext` takes a context, query string, and any amount of arguments to inject in the placeholder variables in the query string.
2. `Scan` copies data from a database row into your variables. Here, it copies the value returned from the query string into `uri`. It takes a variable because it modifies your variables.

```go
func (s *Shortener) Resolve(ctx context.Context, key link.Key) (link.Link, error) {
	if key.Empty() {
		return link.Link{}, fmt.Errorf("validating: empty key: %w", link.ErrBadRequest)
	}
	if err := key.Validate(); err != nil {
		return link.Link{}, fmt.Errorf("validating: %w: %w", err, link.ErrBadRequest)
	}

	// Retrieve the link from the db
	var uri base64String
	err := s.db.QueryRowContext(ctx, `SELECT uri FROM links WHERE short_key = $1`, key).Scan(&uri)

	if errors.Is(err, sql.ErrNoRows) {
		return link.Link{}, link.ErrNotFound
	}

	if err != nil {
		return link.Link{}, fmt.Errorf("retrieving: %w: %w", err, link.ErrInternal)
	}

	return link.Link{
		Key: key,
		URL: uri.String(),
	}, err
}
```

## Valuer and Scanner

These interfaces enable database-specific features without breaking the abstraction provided by the `sql` package.

- `Valuer` can transform values before sending them to a db
- `Scanner` can transform values retrieved from a db

For example, Postgres has an array type that stores an array as binary data in a single db column. Go's `sql` package doesn't natively support that, so the `pql` driver implements `Value` and `Scanner` in its `pq.Array` type.

For example:

```go
var scores []int
db.QueryRowContext(..., `SELECT ARRAY[42, 84]`).Scan(pq.Array(&scores))

_, err := db.ExecContext(..., `INSERT INTO results(scores) VALUES($1)`, pq.Array(scores))
```

This example uses the `Valuer` and `Scanner` interfaces to endcode URLs in Base64 before saving, and then decode them after retrieving.
- Declare a new type that satisfies `Valuer` and `Scanner`

```go
type base64String string

func (bs base64String) Value() (driver.Value, error) {
	return base64.StdEncoding.EncodeToString([]byte(bs)), nil
}

func (bs *base64String) Scan(src any) error {
	ss, ok := src.(string)
	if !ok {
		return fmt.Errorf("decoding: %q is %T, not string", ss, src)
	}
	dst, err := base64.StdEncoding.DecodeString(ss)
	if err != nil {
		return fmt.Errorf("decoding %q: %w", ss, err)
	}
	*bs = base64String(dst)
	return nil
}

func (bs base64String) String() string {
	return string(bs)
}
```

Integrate this into your database service methods when you insert and retrieve values:

```go
func (s *Shortener) Shorten(ctx context.Context, lnk link.Link) (link.Key, error) {
	//...
	
	// Persist the link in the db
	_, err = s.db.ExecContext(
		ctx,
		`INSERT INTO links (short_key, uri) VALUES ($1, $2)`,
		lnk.Key, base64String(lnk.URL),
	)
	
	//..
	
	return lnk.Key, nil
}
```

This declares the `uri` type as the `base64String` type, and uses its `String` method in the return statement:

```go
func (s *Shortener) Resolve(ctx context.Context, key link.Key) (link.Link, error) {
	//.. 

	// Retrieve the link from the db
	var uri base64String
	err := s.db.QueryRowContext(ctx, `SELECT uri FROM links WHERE short_key = $1`, key).Scan(&uri)
	
	//...
	
	return link.Link{
		Key: key,
		URL: uri.String(),
	}, err
}
```

## Service interface

The `rest` package defines two small interfaces:

- `Shortener` with `Shorten(context.Context, link.Link) (link.Key, error)`
- `Resolver` with `Resolve(context.Context, link.Key) (link.Link, error)`

The handlers depend on these interfaces, not on concrete storage types:

- `Shorten(lg, links Shortener)` accepts anything that can shorten links.
- `Resolve(lg, links Resolver)` accepts anything that can resolve keys.

This means the transport layer (`rest`) asks for only the behavior it needs. It does not know whether the implementation is SQLite, in-memory, or a test fake.

In this project, `sqlite.Shortener` satisfies both interfaces because it has methods with matching signatures. The in-memory `link.Shortener` does as well. No extra registration code is required:

```go
type Shortener interface {
	Shorten(context.Context, link.Link) (link.Key, error)
}

// Shorten handles HTTP requests to create a shortened link.
func Shorten(lg *slog.Logger, links Shortener) http.Handler {
	// ...
}

type Resolver interface {
	Resolve(context.Context, link.Key) (link.Link, error)
}

// Resolve handles HTTP requests to redirect from a key to its full URL.
func Resolve(lg *slog.Logger, links Resolver) http.Handler {
	// ...
}
```

Go interfaces are implicit. A type implements an interface automatically when it has the required methods. You do not write `implements` declarations.

That gives you a consumer-first model:

- The consumer package (`rest`) defines the minimal contract it needs.
- Producer packages (`sqlite`, `link`, test fakes) adapt by implementing methods with matching signatures.
- Dependencies stay pointed inward to behavior, not outward to concrete packages.

This approach lowers coupling and improves testability. You can swap implementations at wiring time in `cmd/linkd` without changing handler code.

## Wire the db into main

First, add the database to your applicaiton configuration:

```go
// config holds application configuration and logger dependencies.
type config struct {
	http struct {
		addr     string
		timeouts struct{ read, idle time.Duration }
	}
	lg *slog.Logger
	db struct{ dsn string }
}
```

Next, add a flag so the user can define a custom DSN. If one isn't provided, data is persisted in a `linksdb` file in the same directory as the program. `rwc` mode creates the file if it doesn't exist:

```go
func main() {
	var cfg config

	// ...

	flag.StringVar(&cfg.db.dsn, "db.dsn", "file:linksdb?mode=rwc", "database DSN")

	// ...
}
```
Next, add the database to the configuration:
1. Connect to the database.
2. Get a new SQL-backed shortener service.


```go
func run(ctx context.Context, cfg config) error {
	db, err := sqlite.Dial(ctx, cfg.db.dsn)						// 1
	if err != nil {
		return fmt.Errorf("dialing database: %w", err)
	}
	shortener := sqlite.NewShortener(db) 						// 2

	lg := slog.New(traceid.NewLogHandler(cfg.lg.Handler()))

	// ...
}
```