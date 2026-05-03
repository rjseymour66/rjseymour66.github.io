+++
title = 'Context'
date = '2026-02-25T23:07:21-05:00'
weight = 80
draft = false
+++

The `context` package provides a standard way to carry cancellation signals,
deadlines, and request-scoped values across API boundaries and between goroutines.

Pass a `context.Context` as the first argument to any function that performs I/O,
calls an external service, or runs work that should stop when the caller no longer
needs the result. When a context is cancelled or its deadline expires, every
function holding that context can detect the signal and return early, freeing
resources immediately rather than waiting for work to finish.

## Basics

`Context` is an idiomatic way to propagate cancellation signals through an app.
It is like a tree, with a root context, branches, and leaves. A cancellation
propagates through the hierarchy. If you cancel a parent, all children contexts
are cancelled. However, if you cancel a child context, the parent context is
untouched.

A root `Context` is immutable. You can't cancel it directly, but you can derive a
cancellable `Context` from it. For example:

1. Gets a root context.
2. Creates a cancellable context from `root`. `WithTimeout` takes a parent context
   and a `time.Duration` timeout. This context cancels automatically after 15 seconds.
3. Creates a grandchild context from `child` with a 5-second timeout.

```go
root := context.Background()                                         // 1
child, cancelChild := context.WithTimeout(root, 15*time.Second)      // 2
defer cancelChild()

grandChild, cancelGrand := context.WithTimeout(child, 5*time.Second) // 3
defer cancelGrand()
```

Always call the cancel function when you're done with a derived context.
Skipping it leaks the goroutines and resources associated with that context.

### Create a context

| Function | Creates | Usage | Auto-Cancels? | Cancel Func? |
|:---|:---|:---|:---|:---|
| `context.Background()` | Root context | In main, app startup, tests | No | No |
| `context.TODO()` | Placeholder root | When refactoring or undecided | No | No |
| `context.WithCancel(parent)` | Cancelable child | Manual cancellation (goroutines, shutdown) | No (manual only) | Yes |
| `context.WithTimeout(parent, d)` | Child with timeout | Limit operation duration | Yes (after duration) | Yes |
| `context.WithDeadline(parent, t)` | Child with deadline | Cancel at specific time | Yes (at deadline) | Yes |
| `context.WithValue(parent, key, val)` | Child with value | Request-scoped metadata | No | No |
| `context.WithoutCancel(parent)` | Non-cancelable child | Pass values without propagating cancellation (Go 1.21+) | No | No |
| `context.WithCancelCause(parent)` | Cancelable child with cause | Cancel with a descriptive error (Go 1.21+) | No | Yes |

### Context methods

| Method | Returns | Description | Usage |
|:---|:---|:---|:---|
| `ctx.Done()` | `<-chan struct{}` | Closes when cancelled or expired | In select to stop work |
| `ctx.Err()` | `error` | Why it was cancelled | After Done() fires |
| `ctx.Deadline()` | `(time.Time, bool)` | Reports deadline if set | Rare — mostly internal libs |
| `ctx.Value(key)` | `any` | Gets stored value | Request metadata |

## Context in practice

### Pass context as the first parameter

Pass `ctx context.Context` as the first argument to every function that performs
I/O, calls external services, or spawns goroutines. Never store a context in a struct.

```go
func fetchUser(ctx context.Context, id int) (*User, error) {
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        return nil, err
    }
    // ...
}
```

### Respond to cancellation

Use a `select` statement to stop work when the context is cancelled.

```go
func process(ctx context.Context, items []string) error {
    for _, item := range items {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            handle(item)
        }
    }
    return nil
}
```

### Use unexported key types for context values

Always define context value keys as an unexported type in your package. This
prevents key collisions across packages.

```go
type contextKey string

const requestIDKey contextKey = "requestID"

func WithRequestID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, requestIDKey, id)
}

func RequestIDFrom(ctx context.Context) (string, bool) {
    id, ok := ctx.Value(requestIDKey).(string)
    return id, ok
}
```

### Cancel with a cause (Go 1.21+)

`context.WithCancelCause` lets you attach a descriptive error to a cancellation.
Retrieve it with `context.Cause`.

```go
ctx, cancel := context.WithCancelCause(context.Background())
cancel(errors.New("upstream rate limit exceeded"))

fmt.Println(context.Cause(ctx)) // upstream rate limit exceeded
```

### HTTP server contexts

`http.Request.Context()` is cancelled when the client disconnects. Pass it to
downstream calls so they stop automatically.

```go
func handler(w http.ResponseWriter, r *http.Request) {
    result, err := fetchUser(r.Context(), userID)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    // ...
}
```

### Database queries with context

Pass context to every database call so queries respect timeouts and cancellations.

```go
func getOrder(ctx context.Context, db *sql.DB, id int) (*Order, error) {
    row := db.QueryRowContext(ctx, "SELECT * FROM orders WHERE id = $1", id)
    // ...
}
```
