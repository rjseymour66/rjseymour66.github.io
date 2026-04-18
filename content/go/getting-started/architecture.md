+++
title = 'Architecture'
date = '2026-03-22T22:51:14-04:00'
weight = 40
draft = false
+++

`linkd` is a URL shortener. `POST /shorten` accepts a URL and returns a short key. `GET /r/{key}` redirects to the original URL. The program stores links in SQLite or in memory. It's a concrete example of the layered architecture from [Project setup](../project-setup) and [Structuring packages and services](../structuring).

## Project layout

```bash
link/
├── cmd/linkd/
│   └── linkd.go         # composition root: config, wiring, server startup
├── link.go              # domain: Link and Key types, Shorten function
├── error.go             # domain: sentinel errors
├── shortener.go         # in-memory Shortener (development and test use)
├── rest/
│   ├── health.go        # health check handler
│   └── shortener.go     # HTTP handlers, Shortener and Resolver interfaces
├── sqlite/
│   ├── schema.sql       # embedded table schema
│   ├── sqlite.go        # Dial and DialTestDB helpers
│   └── shortener.go     # SQLite-backed Shortener implementation
└── kit/
    ├── hio/
    │   ├── hio.go       # Handler type and ServeHTTP
    │   ├── request.go   # DecodeJSON, MaxBytesReader
    │   └── response.go  # Responder: JSON, Text, Redirect, Error
    ├── hlog/
    │   └── hlog.go      # HTTP request logging middleware
    └── traceid/
        ├── traceid.go   # trace ID generation and context storage
        ├── http.go      # middleware that injects a trace ID into context
        └── slog.go      # slog handler that adds trace_id to every log record
```

Dependency direction:

```
cmd/linkd  →  rest, sqlite, kit
rest       →  link (domain), kit/hio
sqlite     →  link (domain)
kit        →  standard library only
link       →  standard library only
```

## The domain layer

The domain package is the root of the module (`package link`). It defines:

- `Link`: a struct with a URL and a `Key`
- `Key`: a named string type with `Validate`, `Empty`, and `String` methods
- `Shorten`: a function that hashes the URL to generate a key if one isn't provided, then validates the result
- Sentinel errors in `error.go`: `ErrConflict`, `ErrNotFound`, `ErrBadRequest`, `ErrInternal`

Placing the domain at the root works because this is a single-domain module. The package name `link` matches the module name, and every other package imports it as `"link"`. In a multi-domain service, move these types to `internal/domain` instead.

The domain imports nothing outside the standard library. All business rules live here: key length limits, URL validation (scheme, host), and key generation. No other layer needs to know how shortening works — they call `link.Shorten` and receive a key.

## The storage adapters

Two types implement the shortener operations.

`link.Shortener` (`shortener.go`, root package) stores links in a `map[Key]Link` protected by a `sync.RWMutex`. It has no dependencies beyond the domain. Use it in development and tests where a real database isn't needed.

`sqlite.Shortener` (`sqlite/shortener.go`) stores links in SQLite. `sqlite.Dial` opens the connection, pings it, and applies the schema from the embedded `schema.sql` file. The schema is embedded with `//go:embed schema.sql`, so the binary carries it and applies it automatically at startup.

Both implementations call `link.Shorten` to generate and validate the key before persisting. Neither knows about HTTP.

`sqlite.Shortener` maps storage errors to domain sentinels. A primary key violation becomes `link.ErrConflict`. Callers check errors with `errors.Is` against the domain sentinels and never see a SQLite error directly.

The package uses a `base64String` type that implements `driver.Valuer` and `sql.Scanner`. This encodes URLs as base64 in the database and decodes them transparently on read.

`sqlite.DialTestDB` creates an in-memory SQLite database scoped to a single test. It uses `tb.Name()` as the DSN so parallel tests don't share state, and registers a `Cleanup` to close the connection when the test ends.

## The HTTP transport

`rest/shortener.go` defines two interfaces:

```go
type Shortener interface {
    Shorten(context.Context, link.Link) (link.Key, error)
}

type Resolver interface {
    Resolve(context.Context, link.Key) (link.Link, error)
}
```

Both `link.Shortener` and `sqlite.Shortener` satisfy these interfaces without knowing they exist. The interface is defined where it's used — in the `rest` package — not where it's implemented. This is the recommendation from [Effective Go](https://go.dev/doc/effective_go#interfaces_and_types).

Handlers use a custom `hio.Handler` type:

```go
type Handler func(w http.ResponseWriter, r *http.Request) Handler
```

A handler is a function that returns its next handler. An error handler returns a handler that writes the error response. A success handler returns one that writes the success response. `ServeHTTP` calls the first handler and, if it returns a non-nil handler, calls that one too. This makes the control flow explicit: instead of `if err != nil { return }` scattered through a handler body, the error path is returned as a value.

`httpError` maps domain sentinel errors to HTTP status codes using `errors.Is`. An unknown error becomes `500` and is logged; the client receives only `"internal error"`.

## The kit

`kit` holds cross-cutting utilities. Each sub-package depends only on the standard library.

### kit/traceid

`traceid` assigns each HTTP request a unique ID and makes it available wherever the request context flows.

`traceid.Middleware` generates an ID at the edge of the system and stores it in the request context. `traceid.NewLogHandler` wraps a `slog.Handler` and adds `trace_id` to every log record whose context contains one. You wire them together in the composition root:

```go
lg  := slog.New(traceid.NewLogHandler(cfg.lg.Handler()))
srv := &http.Server{
    Handler: traceid.Middleware(hlog.Middleware(lg)(mux)),
    ...
}
```

Every log line for a given request includes the same `trace_id` with no changes at individual call sites.

`traceid.go` uses an unexported struct type as the context key:

```go
type traceIDContextKey struct{}
```

This prevents any package from accidentally overwriting or reading the value by constructing the same key.

### kit/hlog

`hlog.Middleware` wraps each request in `RecordResponse`, which captures the status code and duration, then logs them as structured fields after the handler returns. It uses `hlog.Interceptor` to intercept calls to `WriteHeader` without modifying the response.

### kit/hio

`hio` handles the mechanical work of decoding requests and encoding responses. `DecodeJSON` reads the body, unmarshals it, and calls `Validate()` if the target implements that method. `Responder` provides `JSON`, `Text`, `Redirect`, and `Error` methods that each return an `hio.Handler`.

## The composition root

`cmd/linkd/linkd.go` is the only file that knows about all layers. It:

1. Defines `config` with nested structs for HTTP and database settings.
2. Parses command-line flags with the `flag` package.
3. Creates the logger and wraps its handler with `traceid.NewLogHandler`.
4. Dials the database and constructs `sqlite.NewShortener`.
5. Registers routes on an `http.NewServeMux`.
6. Wraps the mux in the middleware chain: `traceid.Middleware(hlog.Middleware(lg)(mux))`.
7. Starts the server.

`cmd/linkd` is the only place where concrete types (`sqlite.Shortener`) are assigned to interface parameters (`rest.Shortener`). Everything else works through interfaces.

## How a request flows

A `POST /shorten` request with body `{"URL": "https://example.com"}` passes through each layer in order:

1. `traceid.Middleware` checks the context for a trace ID. If absent, it generates one and stores it in the request context.
2. `hlog.Middleware` records the start time and wraps the `ResponseWriter` to capture the status code.
3. `mux` matches `POST /shorten` and routes to `rest.Shorten`.
4. `rest.Shorten` decodes the JSON body into `link.Link` and calls `links.Shorten(r.Context(), lnk)`.
5. `sqlite.Shortener.Shorten` calls `link.Shorten(lnk)` to generate the key, then executes the `INSERT`.
6. `rest.Shorten` writes `{"key": "abc123"}` with status `201 Created`.
7. `hlog.Middleware` logs `path=/shorten method=POST status=201 duration=1.2ms trace_id=...`.

## Apply this pattern to a new project

**1. Define the domain**

Create types, validation methods, and sentinel errors in the root package or `internal/domain`. Add no imports outside the standard library. Define `ErrBadRequest`, `ErrNotFound`, `ErrConflict`, and `ErrInternal` before writing any other code.

**2. Write the storage adapter**

Create a `sqlite/` or `store/` package. Implement `Dial` and `DialTestDB`. Map storage errors to domain sentinels in every method. Embed your schema with `//go:embed`. Test against a real database using `DialTestDB`.

**3. Define interfaces in the transport layer**

Create a `rest/` package. Define the smallest interface each handler needs — one or two methods. Write handlers that call those interfaces. Use `httpError` to map domain sentinels to HTTP status codes.

**4. Build the kit**

Copy or adapt `traceid` and `hlog`. These rarely change between projects. Add `hio` if you want the chaining handler pattern. Keep all `kit` packages dependent only on the standard library.

**5. Wire everything in `cmd/<app>`**

Construct concrete types (`sqlite.NewShortener`) and pass them where interface parameters are expected. Build the middleware chain in one place. Keep `main` to two lines: create a context, call `run`.
