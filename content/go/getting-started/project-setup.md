+++
title = 'Setting up a production-ready Go application'
linkTitle = 'Project setup'
date = '2026-03-28T00:00:00-04:00'
weight = 30
draft = false
+++

Production-ready Go applications share a common structure: a domain model that captures business rules, a service layer that orchestrates logic, adapters that handle I/O, and a composition root that wires everything together.

This page covers the blueprint. Apply it to any Go application regardless of domain.

## Define requirements

Requirements answer three questions: what does the system do, what are the rules, and what does failure look like?

Document requirements in three areas before writing any code:

**Operations:** what actions the system performs
- A client submits data and receives a response.
- The system exposes a health check endpoint.

**Rules:** constraints the system enforces
- Input must meet defined validation criteria.
- Duplicate entries must be rejected.

**Failure modes:** how the system responds to each error condition
- Invalid input: return a bad request error.
- Duplicate entry: return a conflict error.
- Missing resource: return a not found error.
- Unexpected error: log the full error, return a generic message to the client.

Writing failure modes down before you write any code keeps your error taxonomy honest. Each failure mode becomes a sentinel error. Each rule becomes a validation.

## Plan the project layout

Go doesn't enforce a directory structure, but the community has converged on a layout that scales well. The key principle from [Effective Go](https://go.dev/doc/effective_go): organize by responsibility, not by type.

```
myapp/
├── cmd/
│   └── myapp/
│       ├── main.go      # entry point: two lines, calls run
│       ├── run.go       # wiring: config, deps, server, shutdown
│       └── config.go    # config struct and loadConfig
├── internal/
│   ├── domain/          # types, validation, sentinel errors
│   ├── service/         # business logic, orchestration
│   ├── store/           # storage adapter (database)
│   ├── rest/            # HTTP transport layer
│   └── kit/             # shared utilities (logging, tracing)
├── go.mod
└── go.sum
```

Key decisions:
- `cmd/<app>/` contains only wiring and startup. No business logic.
- `internal/` prevents other modules from importing your packages. The Go compiler enforces this.
- The domain package imports nothing outside the standard library.
- Adapters (`rest`, `store`) import the domain. The domain never imports them.

## Plan package structure

A package is a unit of responsibility. Each package does one thing, and nothing outside it needs to know how it does it.

Use your requirements to identify four kinds of packages:

1. **Domain:** core types, validation rules, and sentinel errors.
2. **Service:** business operations that orchestrate domain logic and storage.
3. **Adapters:** packages that connect your service to external systems (HTTP, databases, queues).
4. **Utilities:** behavior that cuts across all requests (logging, tracing, auth).

Draw the dependency arrows. Each package depends only on packages below it:

```
cmd/myapp  → rest, service, store, kit
rest       → service (via interface), domain
service    → domain, store (via interface)
store      → domain
domain     → standard library only
kit        → standard library only
```

This direction is not optional. It keeps your business rules free of infrastructure. If you find yourself importing `rest` from `store`, you've created a cycle. Go refuses to compile it.

## What is a service?

A service is a struct that implements your business operations. It sits between your transport layer (HTTP handlers) and your storage layer (database). Handlers call service methods. The service calls storage. Neither layer knows about the other.

A service:
- Holds its dependencies as interface fields, not concrete types.
- Implements operations as methods: `Register`, `Fetch`, `Delete`.
- Enforces business rules: validation, authorization, sequencing.
- Does not know about HTTP status codes, SQL, or request parsing.

```go
// internal/service/user.go

type UserStore interface {                                        // 1
    Save(ctx context.Context, u *domain.User) error
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
}

type UserService struct {
    store UserStore
}

func NewUserService(store UserStore) *UserService {              // 2
    return &UserService{store: store}
}

func (s *UserService) Register(ctx context.Context, email, password string) (*domain.User, error) {
    if err := domain.ValidateEmail(email); err != nil {          // 3
        return nil, fmt.Errorf("register: %w: %w", err, domain.ErrBadRequest)
    }
    _, err := s.store.FindByEmail(ctx, email)
    if err == nil {
        return nil, fmt.Errorf("register: %w", domain.ErrConflict) // 4
    }
    if !errors.Is(err, domain.ErrNotFound) {
        return nil, fmt.Errorf("register: %w", err)
    }
    u := &domain.User{Email: email}
    if err := u.SetPassword(password); err != nil {
        return nil, fmt.Errorf("register: %w", err)
    }
    if err := s.store.Save(ctx, u); err != nil {
        return nil, fmt.Errorf("register: %w", err)
    }
    return u, nil
}
```

1. The service defines the interface it needs. It does not import the `store` package directly.
2. The constructor accepts the interface. Pass the concrete implementation from the composition root.
3. The service calls domain validation. It does not duplicate validation logic.
4. If `FindByEmail` returns no error, the user already exists.

The HTTP handler calls `svc.Register(r.Context(), email, password)` without knowing about databases. The store saves users without knowing about HTTP or business rules. The service coordinates them.

### Define interfaces at package boundaries

Per [Effective Go](https://go.dev/doc/effective_go#interfaces_and_types): interfaces belong in the package that uses them, not the package that implements them.

Your `rest` package defines the interface it needs from the service:

```go
// internal/rest/handlers.go

type UserRegistrar interface {
    Register(ctx context.Context, email, password string) (*domain.User, error)
}
```

Your `service.UserService` satisfies this interface without knowing it exists. You wire them together in `cmd/myapp`, where both sides are visible.

Define interfaces with one or two methods focused on what the consumer needs. Large interfaces are hard to implement in tests and couple packages too tightly.

## Choose your implementation order

Build layers in this order: domain first, then storage, then service, then transport, then wiring.

**1. Domain**
Write types, validation, and sentinel errors. No imports outside the standard library. Test immediately with no setup.

**2. Storage adapter**
Implement persistence against your domain types. Test against a real database, not mocks. A mocked database cannot catch SQL errors, constraint violations, or migration failures.

**3. Service**
Implement business operations against the storage interface. Test with a fake in-memory implementation of the interface.

**4. Transport**
Write HTTP handlers that call the service interface. Test with `net/http/httptest` against a fake service implementation.

**5. Composition root**
Wire all layers together in `cmd/<app>`. If wiring grows large, that's a signal your dependencies are too tangled, not that `main` needs more functions.

## Model the domain

The domain layer defines your core types, validates your rules, and declares your error taxonomy. It has no dependencies on HTTP, databases, or any other infrastructure.

### Define core types

Start with the entities your system works with. Keep types small and give each distinct concept its own type even if the underlying representation is the same.

```go
// internal/domain/user.go

type User struct {
    ID           int64
    Email        string
    PasswordHash []byte
    CreatedAt    time.Time
}

func (u *User) SetPassword(password string) error {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return fmt.Errorf("hashing password: %w", err)
    }
    u.PasswordHash = hash
    return nil
}
```

### Implement validation

Each type validates itself. Attach a `Validate` method or standalone `Validate<Field>` functions that enforce the rules from your requirements.

```go
// internal/domain/validate.go

func ValidateEmail(email string) error {
    if !strings.Contains(email, "@") {
        return errors.New("invalid email address")
    }
    return nil
}
```

### Declare sentinel errors

Define failure modes as package-level error values. Per the [Go Blog on error handling](https://go.dev/blog/go1.13-errors): other packages use `errors.Is` to check for them. Your transport layer maps them to HTTP status codes.

```go
// internal/domain/errors.go

var (
    ErrBadRequest = errors.New("bad request")
    ErrConflict   = errors.New("conflict")
    ErrNotFound   = errors.New("not found")
    ErrInternal   = errors.New("internal error")
)
```

Keeping sentinel errors in the domain package means your storage adapters and HTTP handlers share the same error vocabulary. The storage layer wraps them with context. The transport layer unwraps them with `errors.Is`.

## Configure the application

Load configuration from environment variables at startup. Per the [12-factor app methodology](https://12factor.net/config), configuration that varies between deployments belongs in the environment, not the code.

### Define a config struct

Define a single `config` struct that holds all runtime settings. Keep fields unexported — only `cmd/<app>` reads them.

```go
// cmd/myapp/config.go

type config struct {
    addr     string
    dsn      string
    logLevel slog.Level
}
```

### Load config with a getenv function

Accept a `getenv func(string) string` parameter instead of calling `os.Getenv` directly. This makes `loadConfig` testable without manipulating the process environment.

```go
func loadConfig(getenv func(string) string) (config, error) {
    cfg := config{                                    // 1
        addr:     ":8080",
        logLevel: slog.LevelInfo,
    }
    if v := getenv("MYAPP_ADDR"); v != "" {
        cfg.addr = v
    }
    cfg.dsn = getenv("MYAPP_DSN")
    if cfg.dsn == "" {
        return config{}, errors.New("MYAPP_DSN is required") // 2
    }
    return cfg, nil
}
```

1. Provide sensible defaults for optional settings.
2. Fail fast on missing required variables before the server starts.

### Use a run function

Keep `main` to two lines. Put all startup logic in a `run` function that accepts context and a `getenv` function, and returns an error. This makes the full startup path testable.

```go
// cmd/myapp/main.go

func main() {
    ctx := context.Background()
    if err := run(ctx, os.Getenv); err != nil {
        fmt.Fprintf(os.Stderr, "%s\n", err)
        os.Exit(1)
    }
}
```

```go
// cmd/myapp/run.go

func run(ctx context.Context, getenv func(string) string) error {
    cfg, err := loadConfig(getenv)                             // 1
    if err != nil {
        return fmt.Errorf("config: %w", err)
    }

    lg := slog.New(slog.NewTextHandler(os.Stderr,             // 2
        &slog.HandlerOptions{Level: cfg.logLevel},
    )).With("app", "myapp")

    db, err := sql.Open("postgres", cfg.dsn)                  // 3
    if err != nil {
        return fmt.Errorf("open db: %w", err)
    }
    defer db.Close()

    userStore   := store.NewUserStore(db)                     // 4
    userService := service.NewUserService(userStore)

    mux := http.NewServeMux()
    rest.Register(mux, lg, userService)                       // 5

    srv := &http.Server{
        Addr:         cfg.addr,
        Handler:      mux,
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }

    go func() {
        lg.Info("starting", "addr", cfg.addr)
        if err := srv.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
            lg.Error("server error", "err", err)
            os.Exit(1)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    lg.Info("shutting down")
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        return fmt.Errorf("shutdown: %w", err)
    }
    lg.Info("stopped")
    return nil
}
```

1. Load and validate config before allocating any resources.
2. Set up the logger immediately so all subsequent errors are structured.
3. Open the database connection. `defer db.Close()` ensures it closes when `run` returns.
4. Construct concrete implementations and pass them to services. This is the only place that knows about all layers.
5. Register HTTP routes. `rest.Register` receives the logger and service interface, not concrete types.

The `run` pattern comes from Mat Ryer's [How I write HTTP services in Go](https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/). It separates startup logic from process management and makes the entire boot sequence testable.

## Set up structured logging

Use `log/slog` (Go 1.21+) for structured, leveled logging. Structured logs emit key-value pairs that log aggregation tools can parse and query without regex.

### Create a logger in main

Initialize the logger in `run` and pass it as a dependency. Don't use a global logger — it makes behavior implicit and tests harder to isolate.

```go
lg := slog.New(slog.NewTextHandler(os.Stderr, &slog.HandlerOptions{
    Level: cfg.logLevel,
})).With("app", "myapp")                   // 1
```

1. `.With("app", "myapp")` attaches a static field to every log record this logger produces. Use this for fields that never change: application name, version, environment.

For production, swap `NewTextHandler` for `NewJSONHandler` to emit machine-readable JSON.

### Use context in log calls

Inside HTTP handlers, use `LogAttrs` with `r.Context()` instead of `Info`:

```go
lg.LogAttrs(r.Context(), slog.LevelError, "register failed",
    slog.Any("error", err),
)
```

The context argument lets the logger pull request-scoped values like a trace ID and add them to the record automatically. See [Propagate context](#propagate-context).

## Propagate context

Per the [Go Blog on context](https://go.dev/blog/context): context carries request-scoped values (trace IDs, deadlines, cancellation signals) across package boundaries without changing function signatures. Every function that does I/O accepts `context.Context` as its first argument.

### Store values in context

Use an unexported struct type as the context key to prevent key collisions with other packages.

```go
// internal/kit/traceid/traceid.go

type contextKey struct{}

func WithContext(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, contextKey{}, id)
}

func FromContext(ctx context.Context) (string, bool) {
    id, ok := ctx.Value(contextKey{}).(string)
    return id, ok
}
```

The unexported type is the key. Other packages cannot construct it, so they cannot overwrite or read the value accidentally.

### Inject values with middleware

Inject the trace ID at the edge of your system so it's available for the full lifetime of the request.

```go
// internal/kit/traceid/http.go

func Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if _, ok := FromContext(r.Context()); !ok {
            ctx := WithContext(r.Context(), New())  // 1
            r = r.WithContext(ctx)                  // 2
        }
        next.ServeHTTP(w, r)
    })
}
```

1. Generate a new ID only if one isn't already present. This lets upstream proxies inject their own trace IDs.
2. Replace the request's context. All downstream handlers receive the updated context from `r.Context()`.

### Enrich logs automatically

Write a custom `slog.Handler` that reads the trace ID from context and adds it to every log record.

```go
// internal/kit/traceid/slog.go

type handler struct{ slog.Handler }

func (h *handler) Handle(ctx context.Context, r slog.Record) error {
    if id, ok := FromContext(ctx); ok {
        r = r.Clone()                                // 1
        r.AddAttrs(slog.String("trace_id", id))
    }
    return h.Handler.Handle(ctx, r)
}

func NewLogHandler(base slog.Handler) slog.Handler {
    return &handler{base}
}
```

1. Clone the record before modifying it. `slog.Record` is not safe to modify in place.

Wrap the base handler in `run`:

```go
base := slog.NewTextHandler(os.Stderr, &slog.HandlerOptions{Level: cfg.logLevel})
lg   := slog.New(traceid.NewLogHandler(base)).With("app", "myapp")
```

Every `lg.LogAttrs(ctx, ...)` call that passes a context containing a trace ID includes `trace_id` in the output, with no changes to individual call sites.

### Pass context through the call stack

Every function that performs I/O accepts and forwards the context.

```go
func (s *UserService) Register(ctx context.Context, email, password string) (*domain.User, error) {
    // ...
    if err := s.store.Save(ctx, u); err != nil {   // 1
        return nil, fmt.Errorf("register: %w", err)
    }
    return u, nil
}
```

1. `Save` passes `ctx` to the database driver. If the request is cancelled or times out, the database call returns immediately instead of blocking.

The context flows from the HTTP request through the handler into the service and down to the database driver, propagating the trace ID and respecting cancellation at every level.
