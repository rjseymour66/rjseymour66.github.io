+++
title = 'Go'
date = '2025-08-05T10:30:35-04:00'
weight = 40
draft = false
+++

This curriculum takes you from advanced beginner to intermediate Go developer through five practical projects. Each project builds on the last and introduces patterns you'll use in production systems.

## Project 1: `ghstat`, GitHub repository CLI

**Difficulty:** 3 / 5

### What you build

`ghstat` is a CLI tool that fetches and displays statistics for one or more GitHub repositories: stars over time, top contributors, and open issues by label. A local cache prevents redundant API calls on repeated runs.

### Requirements

- Implement subcommands (`ghstat repo stats`, `ghstat repo compare`) using [Cobra](https://github.com/spf13/cobra).
- Configure the HTTP client (timeout, retry count, auth token) using the functional options pattern.
- Fetch multiple repositories concurrently using goroutines and `errgroup`.
- Cache responses to disk as JSON. Invalidate the cache after a configurable TTL.
- Wrap all errors with context: `fmt.Errorf("fetching repo %s: %w", name, err)`.

### What you learn

- Cobra subcommand structure and flag binding
- Functional options pattern for configuring structs
- Concurrent fan-out with `errgroup`
- JSON serialization and file I/O for caching
- Error wrapping with `%w`

### Why it matters

CLIs are the fastest path to shipping something real in Go. This project forces a clean package boundary between a `github` client package and a `cmd` package. The fan-out pattern you implement here reappears directly in Projects 3 and 5.

---

## Project 2: Webhook ingestion service

**Difficulty:** 3.5 / 5

### What you build

You'll build an HTTP service that receives, validates, and stores inbound webhooks from any HMAC-signed provider (GitHub, Stripe, and similar). The service exposes three endpoints:

- `POST /hooks/:provider`: receive and store an event
- `GET /events`: list stored events, filterable by provider and status
- `POST /events/:id/replay`: re-deliver a stored event to a configurable target URL

### Requirements

- Build the server using `net/http` directly. Do not use a router framework.
- Implement HMAC-SHA256 signature verification as a `func(http.Handler) http.Handler` middleware. Read and buffer the request body once. Pass it to downstream handlers without consuming it twice.
- Define a `Store` interface with `Save`, `List`, `Get`, and `MarkReplayed` methods. Implement it as an in-memory struct guarded by a `sync.RWMutex`.
- Inject `http.Client` as a dependency for the replay handler. Do not use `http.DefaultClient`.
- Implement graceful shutdown: listen for `os.Signal`, call `server.Shutdown(ctx)` with a timeout, and drain in-flight replay requests before exiting.
- Write table-driven tests using `httptest.NewRecorder`. Include cases that send invalid signatures and confirm rejection.

### What you learn

- `net/http` handler and middleware composition
- HMAC signature verification, the standard webhook security pattern
- Interface-driven storage design
- Graceful shutdown with `context.WithTimeout`
- Testing HTTP handlers with `httptest`

### Why it matters

Webhook handling appears in nearly every backend integration. Stripe, GitHub, Twilio, and most SaaS platforms use the same HMAC pattern. Injecting `http.Client` as a dependency rather than using `http.DefaultClient` is a small habit with large consequences for testability. The `Store` interface you define here is the same pattern you'll use in Project 4 with a real database.

---

## Project 3: `logwatch`, concurrent log file analyzer

**Difficulty:** 4 / 5

### What you build

`logwatch` ingests one or more log files concurrently, parses lines against configurable patterns (error rate, slow requests above a threshold, IP frequency), and emits a structured summary report to stdout or as JSON.

### Requirements

- Implement a worker pool: a fixed number of goroutines consume from a shared `chan string` of lines. Do not spawn one goroutine per line.
- Structure processing as a pipeline with three stages: `producer → parser → aggregator`. Connect each stage with a channel.
- Propagate a `context.Context` through every stage. When the user cancels (Ctrl-C), all stages must drain and exit cleanly. No goroutine leaks.
- Use `sync.Pool` to reuse line buffers and reduce garbage collection pressure on large files.
- Read files with `bufio.Scanner`.
- Emit structured output using `log/slog` (Go 1.21+).

### What you learn

- Worker pool pattern with bounded concurrency
- Pipeline pattern: staged processing over channels
- Context cancellation and goroutine lifecycle management
- `sync.Pool` for buffer reuse
- Structured logging with `slog`

### Why it matters

After completing this project, you'll have implemented the two concurrency patterns that appear most often in real Go systems: worker pools and pipelines. You'll find both in data ingestion systems, ETL pipelines, stream processors, and CI runners. Context cancellation is required knowledge for any production Go codebase. Every blocking operation must respect it.

---

## Project 4: Task management API with PostgreSQL

**Difficulty:** 4 / 5

### What you build

You'll build a REST API for managing tasks, users, and labels, backed by PostgreSQL. Users authenticate with JWT. All data operations run against a real database with versioned migrations.

### Requirements

- Use `pgx` with `pgxpool` for connection pooling. Set the pool size explicitly. Do not rely on defaults.
- Write database migrations with `golang-migrate`. Run migrations at startup.
- Implement transactions for multi-step operations (for example, creating a task and assigning labels atomically).
- Define custom error types: `ErrNotFound`, `ErrConflict`, and `ErrUnauthorized`. Map them to HTTP status codes in a single handler helper. No type switch per handler.
- Implement JWT authentication as middleware, building on the middleware chain from Project 2.
- Write integration tests using `testcontainers-go`: spin up a real PostgreSQL container per test run. Do not mock the database layer.

### What you learn

- `pgx` and `pgxpool` for production database access
- Schema versioning with `golang-migrate`
- Transaction handling for atomic multi-step writes
- Custom error types that carry behavior, not just messages
- Integration testing with real infrastructure via `testcontainers-go`
- JWT middleware layered on an existing middleware chain

### Why it matters

Most production Go services talk to a database. Understanding `pgxpool` prevents connection exhaustion, one of the most common production incidents in Go. Integration tests with `testcontainers-go` catch real bugs that mock-based tests hide. Custom error types let you handle failures consistently across handlers without duplicating logic.

---

## Project 5: Reverse proxy with health checks and circuit breaking

**Difficulty:** 4.5 / 5

### What you build

You'll build an HTTP reverse proxy that load-balances across a configurable list of backend addresses. Background health checks remove unhealthy backends from rotation. A per-backend circuit breaker stops forwarding to failing backends and retries them after a cooldown period. The proxy exposes Prometheus metrics on `/metrics`.

### Requirements

- Use `httputil.ReverseProxy`. Customize it with a `Director` function and a `ModifyResponse` hook.
- Run background health checkers on a `time.Ticker`. Each checker must respect context cancellation and exit cleanly when the proxy shuts down.
- Implement a circuit breaker state machine (closed → open → half-open) as a struct guarded by `sync.Mutex`. Do not use a third-party circuit breaker library.
- Use `sync/atomic` for high-frequency counters (in-flight requests, failure counts) to avoid lock contention.
- Instrument the proxy with the Prometheus Go client: expose in-flight request counts, request duration histograms, and a backend health gauge.
- Configure backends, timeouts, and circuit breaker thresholds through functional options.

### What you learn

- `httputil.ReverseProxy` customization
- Background goroutine lifecycle management with context
- Circuit breaker state machine design
- `sync/atomic` for low-contention counters
- Prometheus instrumentation: counters, histograms, gauges
- Functional options for complex component configuration

### Why it matters

A proxy concentrates the hardest production Go problems in one place: concurrent state mutation, goroutine lifecycle, and observability. The circuit breaker is a foundational distributed systems pattern used in virtually every service-to-service call at scale. After this project, you can read and contribute to production codebases like Traefik, Caddy, or internal platform tooling.

---

## Project 6: Real-time event fan-out service

**Difficulty:** 5 / 5

### What you build

You'll build a real-time event fan-out service with three components: a gRPC ingestion endpoint that accepts a stream of events from producers, a NATS message bus that decouples ingestion from delivery, and a WebSocket endpoint that broadcasts matching events to connected browser clients in real time.

### Requirements

- Define a protobuf service with a client-streaming RPC for ingest and a server-streaming RPC for subscription. Generate Go code with `protoc` and `protoc-gen-go-grpc`.
- Implement a gRPC server interceptor for authentication that reads a bearer token from incoming metadata. Mirror the middleware pattern from Projects 2 and 4.
- Publish each received event to NATS using the JetStream API. Configure a durable consumer with at-least-once delivery semantics. Handle redelivery and deduplication.
- Consume from NATS in a worker pool (bounded goroutines, same pattern as Project 3) and fan out to active WebSocket connections using `gorilla/websocket` or `nhooyr.io/websocket`.
- Track connected WebSocket clients in a registry guarded by `sync.RWMutex`. Support topic-based filtering so clients receive only events matching their subscription.
- Propagate OpenTelemetry trace context from gRPC metadata through NATS message headers to WebSocket delivery. Export spans to a local OTLP collector.
- Write a load test with `ghz` (gRPC) and `k6` (WebSocket) to verify the fan-out holds under 1,000 concurrent subscribers.

### What you learn

- Protobuf service definition and Go code generation
- gRPC interceptors (unary and streaming)
- gRPC client and server streaming patterns
- NATS JetStream: publishing, durable consumers, and at-least-once delivery
- WebSocket connection management and topic-based fan-out
- OpenTelemetry trace propagation across transport boundaries
- Load testing gRPC and WebSocket endpoints

### Why it matters

The previous five projects cover the patterns that appear in most Go services. This project covers the class they don't: event-driven, streaming systems. gRPC is the dominant inter-service protocol in cloud-native Go, and NATS is a common backbone for low-latency messaging. Propagating trace context across three transports (gRPC → NATS → WebSocket) is a skill that separates developers who understand observability from those who have only configured it. After this project, you can read and contribute to systems like Loki, Tempo, or any internal event pipeline built in Go.
