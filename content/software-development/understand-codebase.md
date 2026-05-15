+++
title = 'Understanding codebases'
date = '2026-05-11T21:18:19-04:00'
weight = 40
draft = false
+++

Before reading a single line of code, understand why the project exists. Code without context leads to changes that are technically correct but practically wrong. Start with intent.

## Understand the purpose

Ask the fundamental questions first: what problem does this software solve, and for whom? Who are the customers, and what do they need from the system? A developer who understands the business domain can navigate unfamiliar code faster than one who knows the syntax but not the purpose.

Look through any documentation available before opening source files. README files, wikis, architecture decision records, onboarding guides, and API references all reveal intent that the code itself cannot express. Even outdated documentation is useful: it tells you what the system was designed to do, which helps you recognize where it has grown beyond its original purpose.

When documentation is sparse or missing, look at the product itself. Use it. Read any public-facing descriptions of what it does. Talk to stakeholders or teammates who know the customers. Understanding who relies on the system and what they expect from it shapes every decision you make inside the codebase.

## Architecture and project structure

Before making changes to a codebase, identify which architectural pattern it follows. The directory structure is your first signal. Most codebases use one of four patterns, or a combination of them.

### Package by layer

*Package by layer* organizes code horizontally based on technical responsibility. All HTTP handlers live together, all services live together, all database logic lives together. The structure reflects the technical tiers of the application rather than its business domain.

```
myapp/
├── handlers/
│   ├── user.go
│   └── order.go
├── services/
│   ├── user.go
│   └── order.go
├── repositories/
│   ├── user.go
│   └── order.go
└── models/
    ├── user.go
    └── order.go
```

This pattern is common in smaller codebases. Its weakness is that a single feature (a user, an order) is scattered across every layer, making it harder to trace the full behavior of any one thing.

### Package by feature

*Package by feature* organizes code vertically around business capabilities. Everything related to users lives in one package, everything related to orders in another. Each package is self-contained.

```
myapp/
├── user/
│   ├── handler.go
│   ├── service.go
│   └── repository.go
├── order/
│   ├── handler.go
│   ├── service.go
│   └── repository.go
└── payment/
    ├── handler.go
    ├── service.go
    └── repository.go
```

This pattern makes it easier to find all the code relevant to a feature and to change or delete a feature without touching unrelated packages.

### Hexagonal architecture

*Hexagonal architecture*, also called *ports and adapters*, separates business logic from external concerns. The core domain has no knowledge of HTTP, databases, or any infrastructure detail. Instead, it defines interfaces (ports) that external adapters implement.

```
myapp/
├── internal/
│   ├── core/
│   │   ├── domain/
│   │   │   └── order.go        # business entities and rules
│   │   └── service/
│   │       └── order.go        # business logic
│   └── ports/
│       ├── inbound/
│       │   └── order_service.go    # interfaces the core exposes
│       └── outbound/
│           └── order_repository.go # interfaces the core depends on
└── adapters/
    ├── http/
    │   └── order_handler.go    # inbound: HTTP → core
    └── postgres/
        └── order_repository.go # outbound: core → database
```

The core has no imports pointing outward. Adapters depend on the core, never the reverse. This makes the business logic easy to test in isolation and straightforward to swap infrastructure without touching domain code.

### Microservices

A microservices architecture splits functionality into multiple independent services, each responsible for a specific business capability and deployable on its own.

If you are working in a microservices environment, start by understanding two things: the architecture of your specific service, and how it fits into the larger ecosystem. Each service typically follows one of the patterns above internally. At the system level, you need to know which services yours calls, which services call yours, and what the contracts between them are. Sequence diagrams and context diagrams are especially useful here.

```
platform/
├── user-service/
│   ├── cmd/
│   │   └── main.go
│   ├── internal/
│   │   ├── handler.go
│   │   ├── service.go
│   │   └── repository.go
│   └── Dockerfile
├── order-service/
│   ├── cmd/
│   │   └── main.go
│   ├── internal/
│   │   ├── handler.go
│   │   ├── service.go
│   │   └── repository.go
│   └── Dockerfile
├── payment-service/
│   ├── cmd/
│   │   └── main.go
│   ├── internal/
│   │   ├── handler.go
│   │   ├── service.go
│   │   └── repository.go
│   └── Dockerfile
└── api-gateway/
    └── main.go
```

Each service is an independent deployable unit with its own entrypoint, internal logic, and container definition. The API gateway routes incoming traffic to the appropriate service.

## Execution flow

*Execution flow* is the sequential path of instructions that a program follows during runtime, including all decisions, loops, and function calls that determine which code executes and in what order. Understanding execution flow tells you how a request enters the system, what happens to it, and how a response is produced.

### Find the entry point

Every application starts somewhere. In Go, that's `main.go`. In a web service, the entry point initializes configuration, wires dependencies, registers routes, and starts the server. Reading it tells you how the application is assembled before any request arrives.

```go
func main() {
    cfg := config.Load()
    db := postgres.Connect(cfg.DatabaseURL)

    repo := order.NewRepository(db)
    svc := order.NewService(repo)
    h := order.NewHandler(svc)

    router := http.NewServeMux()
    router.HandleFunc("POST /orders", h.Create)
    router.HandleFunc("GET /orders/{id}", h.Get)

    http.ListenAndServe(cfg.Addr, router)
}
```

From here you can see every dependency the application needs and every route it exposes.

### Trace requests and responses

Once you know the entry points, follow a request through the system. Pick a single endpoint and trace it from the HTTP handler down through the service layer to the repository and back. This path reveals how data is transformed at each layer, where validation happens, where errors are handled, and what the caller receives in return.

Use your IDE's **Go to Definition** and **Find References** to follow the call chain without losing your place. In Go, `gopls` provides a full call hierarchy that shows both the callers of a function and the functions it calls.

### Locate external dependencies

External dependencies are the boundaries where your code hands off to something outside its control: databases, message queues, third-party APIs, caches. Identify them early.

In Go, look for:

- Packages imported from outside the standard library and your own module (`go.mod` lists them all)
- Interface implementations that wrap a client or driver
- Configuration values loaded from environment variables, which often point to external services

```go
// go.mod reveals all external dependencies at a glance
require (
    github.com/jackc/pgx/v5 v5.5.0       // PostgreSQL
    github.com/redis/go-redis/v9 v9.4.0   // Redis
    github.com/aws/aws-sdk-go-v2 v1.24.0  // AWS services
)
```

Once you know what the service depends on, you know where its failure modes live. A slow database query, a misconfigured queue, or an unavailable API each affect execution flow in ways the unit tests won't reveal.

## Finding entry points

An entry point is any location where execution begins in response to an external trigger. Most applications have more than one. Identifying them all gives you a complete picture of how the system receives and processes work.

### Main or bootstrap method

The most direct entry point. In Go, execution always begins in `main()`, typically in `cmd/main.go` or `main.go`. The bootstrap method initializes configuration, wires dependencies, and starts the application. Read it first to understand how the pieces connect before any request arrives.

```
cmd/
└── main.go    ← start here
```

### Public API and controllers

HTTP handlers and controllers are entry points for every external request the service accepts. In Go, look for route registration in `main.go` or a dedicated router file. Each registered route is an entry point.

```go
router.HandleFunc("POST /orders", h.Create)
router.HandleFunc("GET /orders/{id}", h.Get)
router.HandleFunc("DELETE /orders/{id}", h.Delete)
```

Each handler is a thread of execution triggered by an incoming HTTP request. Trace any one of them to understand what the service does in response.

### Event handlers and listeners

Event-driven code executes in response to signals from within the application or from an external system. In Go, look for goroutines reading from channels, or callbacks registered with an event bus or pub/sub library.

```go
go func() {
    for event := range eventBus.Subscribe("order.created") {
        handleOrderCreated(event)
    }
}()
```

Event handlers are easy to miss because they aren't called directly — they wait for something to happen. Search for channel reads and subscription registrations to find them.

### Scheduled tasks

Scheduled tasks execute on a timer rather than in response to a request. In Go, look for `time.Ticker`, `time.AfterFunc`, or a cron library such as `robfig/cron`.

```go
c := cron.New()
c.AddFunc("0 * * * *", func() {
    reconcileOrders()
})
c.Start()
```

These are easy to overlook because they produce no visible output unless you check the logs or look for scheduler setup in the bootstrap code.

### Lifecycle hooks

Lifecycle hooks run at specific moments in the application's lifetime: startup, shutdown, or both. In Go, look for signal handling in `main.go` and `defer` statements that register cleanup logic.

```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

// shutdown logic runs here
server.Shutdown(ctx)
db.Close()
```

Startup hooks configure state before the application begins accepting traffic. Shutdown hooks release resources cleanly. Both are entry points for logic that doesn't belong to any single request.

### Plugin and extension points

Frameworks and applications often expose extension points where you can inject behavior without modifying the core. In Go, middleware is the most common form.

```go
router.Use(loggingMiddleware)
router.Use(authMiddleware)
router.Use(rateLimitMiddleware)
```

Each middleware function is an entry point that executes for every matching request. Look for middleware registration in the bootstrap code and trace each one to understand what it does before and after the handler runs.

### Message consumers

Message consumers read from a queue or topic and process each message as it arrives. In Go, look for goroutines that call `Receive`, `Consume`, or `Poll` on a messaging client such as a Kafka consumer, SQS client, or RabbitMQ channel.

```go
go func() {
    for msg := range queue.Receive(ctx) {
        processMessage(msg)
        msg.Ack()
    }
}()
```

Consumer goroutines often run for the lifetime of the application. Search for calls to messaging libraries and look for the goroutine that drives them.

### Command line argument processors

CLI applications and tools accept input through command line arguments rather than HTTP or messages. In Go, look for the `flag` package or a CLI library such as `cobra`.

```go
var port = flag.Int("port", 8080, "server port")
var env  = flag.String("env", "production", "runtime environment")
flag.Parse()
```

Each flag or subcommand is an entry point into a specific behavior. With `cobra`, look for `cmd/root.go` and the registered subcommands to find every execution path the tool supports.

### Database triggers and stored procedures

Some business logic lives in the database rather than the application. Database triggers fire automatically in response to insert, update, or delete operations. Stored procedures encapsulate logic that can be called directly from SQL.

Look for these in migration files or schema definitions:

```sql
CREATE TRIGGER update_inventory
AFTER INSERT ON orders
FOR EACH ROW
EXECUTE FUNCTION decrement_stock();
```

Triggers are invisible from the application code. If behavior seems to happen without explanation, check whether a trigger is responsible. Search migration files for `CREATE TRIGGER` and `CREATE FUNCTION` to find them.

## Logging

Logs are a window into a system's internal behavior at runtime. Where code tells you what the system is designed to do, logs tell you what it actually did, in what order, and under what conditions. Reading the logs of an unfamiliar codebase is one of the fastest ways to understand how it behaves in practice.

### Log levels

Most logging libraries use a hierarchy of levels that reflect the severity and intent of each message. In Go, the standard `log/slog` package defines four levels:

| Level | Use |
|---|---|
| `DEBUG` | Detailed diagnostic information, useful during development |
| `INFO` | Normal operational events: a request received, a job completed |
| `WARN` | Unexpected conditions that didn't cause a failure but deserve attention |
| `ERROR` | Failures that need investigation |

When reading an unfamiliar codebase, search for `WARN` and `ERROR` log statements first. They mark the paths the original developers expected to be problematic.

### Structured logging

Structured logs attach key-value pairs to each message rather than embedding values in a string. They are easier to search, filter, and correlate across requests.

```go
slog.Info("order created",
    "order_id", order.ID,
    "customer_id", order.CustomerID,
    "total", order.Total,
)

slog.Error("payment failed",
    "order_id", order.ID,
    "reason", err.Error(),
)
```

Unstructured logs embed everything in a string:

```go
log.Printf("order %s created for customer %s", order.ID, order.CustomerID)
```

Structured logs make it straightforward to find all events related to a specific order or customer. Unstructured logs require pattern matching.

### What logs reveal

Logs surface behavior that source code alone can't show you:

- **Execution order.** A sequence of `INFO` messages shows which functions ran and in what order for a given request.
- **Branching decisions.** A log message inside an `if` block tells you which path the code took and why.
- **Timing and performance.** Log timestamps reveal where time is being spent across a request lifecycle.
- **Error conditions.** `ERROR` and `WARN` messages document the failure modes the system has already encountered in production.

```go
slog.Info("processing order", "order_id", id)

items, err := repo.GetItems(ctx, id)
if err != nil {
    slog.Error("failed to load order items", "order_id", id, "err", err)
    return err
}

slog.Info("items loaded", "order_id", id, "count", len(items))

if len(items) == 0 {
    slog.Warn("order has no items, skipping fulfillment", "order_id", id)
    return nil
}

slog.Info("fulfillment started", "order_id", id)
```

Reading this sequence in a log file, you can reconstruct exactly what happened to a specific order without attaching a debugger.

### Finding logging in a codebase

Search for the logging library the codebase uses (`slog`, `zap`, `zerolog`, `logrus` are common in Go). Then look at what is being logged and at what level. Gaps in logging — functions or branches with no log statements — often indicate code the original developers considered unimportant or untested. Those gaps are worth noting.

## Sample process

Follow these steps when approaching an unfamiliar codebase for the first time.

1. **Clone the project** from your source code management system.

2. **Review the README, coding standards, architecture docs, and any other supporting documentation.** Take notes on the architecture, build commands, common SQL patterns, industry-specific standards, and domain terminology. Understanding the language of the domain before reading the code makes everything easier to follow.

3. **Review the build scripts.** Makefiles, shell scripts, and task runners reveal how the project is assembled, what steps are required, and what assumptions the build process makes about the environment.

4. **Review the project dependencies.** In Go, read `go.mod`. In other ecosystems, check `package.json`, `pom.xml`, `requirements.txt`, or the equivalent. Dependencies tell you what problems the project delegates to external libraries and which third-party systems it integrates with.

5. **Review the project structure.** Identify which architectural pattern the codebase follows. Locate the entry points, the domain logic, and the infrastructure layer. A few minutes here saves hours of disorientation later.

6. **Review the CI/CD pipelines.** Pipeline configuration files (GitHub Actions, Jenkins, GitLab CI) document the full lifecycle of the code: how it is tested, built, and deployed. They also reveal quality gates, environment-specific configuration, and deployment targets.

7. **Install any project dependencies.** Follow the setup instructions in the README. Note any steps that are missing or outdated — those gaps are your first contribution opportunity.

8. **From the command line, build the project artifacts, run the unit tests, start any required containers, and run the application locally.** Completing this step end-to-end confirms that your environment is configured correctly and gives you a working baseline before you change anything.
