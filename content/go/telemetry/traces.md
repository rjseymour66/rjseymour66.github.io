+++
title = 'Traces'
date = '2026-01-16T16:30:20-05:00'
weight = 20
draft = false
+++

Tracing allows you to observe the behavior of your program during execution. It offers invaluable insights into performance bottlenecks and bugs. Go provides the [runtime/trace](https://pkg.go.dev/runtime/trace) package to collect event data on your goroutines, heap allocation, etc.

Tracing should focus on parts of your program where performance is critical.

{{< admonition "Performance impact" note >}}
Tracing impacts performance more than logging, so use it wisely.
{{< /admonition >}}

## runtime/trace features

This table describes the fundamental parts of the `runtime/trace` package:

| Feature        | Use case                 |
| :------------- | :----------------------- |
| Start/Stop     | Bound trace duration     |
| Task           | Define logical work      |
| Region         | Break work into phases   |
| Log            | Add context              |
| Context        | Propagate causality      |
| Runtime events | Diagnose scheduling & GC |

Here is how these features work together:

```
Trace Session
│
├─ Task (HTTP request)
│   ├─ Region: auth
│   │   └─ Log: user_id=42
│   ├─ Region: db query
│   └─ Region: response write
│
└─ Runtime events (GC, scheduler, syscalls)
```

## Distrbuted tracing

Distributed tracing monitors a request as it travels across services in a distributed system. For example, an online shopper's request might travel through the login, search, shopping cart, and payment services. Tracing provides the following benefits:
- Enhanced observability, which can reveal bottlenecks.
- Root cause analysis to help you pinpoint the exact service or component responsible for an error.
- Performance optimization by showing you where communication between services is slow.
- Debugging complex interactions between microservices.

### Key concepts

To effectively trace, you need to understand these key concepts:
- **Unique identifier**: Assign a unique ID to the initial request. This ID ties together all logs and events related to the request.
- **Propagation**: Propagate the ID through all services. You can send it in HTTP request headers or messages in queues.
- **Spans**: Each service creates a span of time in which it captures information about the role it plays in the request. The span might include timestamps, service names, function calls, and errors.
- **Collection and analysis**: A central tracing system should collect the spans and stitch them together based on the ID. This provides a full view of the request's lifecycle across all services.

## Basic tracing

This example demonstrates tracing with regions, which let you break your tracing into phases of work. It runs concurrent goroutines:

`busyWork` simulates work to show scheduler and CPU activity:

```go
func busyWork() {
	start := time.Now()
	for time.Since(start) < 50*time.Millisecond {
		// Burn CPU
	}
}
```

`worker` simulates real work and annotates it with trace regions. `WithRegion` defines a phase in the program. It marks a chunk of time in the program and gives it a name. A region is a start time, end time, and a label.

```go
func worker(ctx context.Context, wg *sync.WaitGroup, name string, delay time.Duration) {
	defer wg.Done()

	// Mark a region of work in the trace
	trace.WithRegion(ctx, name, func() {
		fmt.Println(name, "starting")

		// Simulate blocking work
		time.Sleep(delay)

		// Simulate CPU work
		busyWork()

		fmt.Println(name, "finished")
	})
}
```

1. Create a file where you will write the trace. This file is later consumed by the go trace tool.
2. `Start` takes the trace file and begins the tracing. This enables global tracing and tells the runtime to emit scheduler, garbage collection, syscall, and other user events.
3. Defer a call to `Stop` so tracing ends when the function returns.
4. Get the context of the current goroutine.
5. `NewTask` defines on logical operation named "main-task". It attaches metadata to the goroutine context.
6. Defer a call to `End` so the task ends when the function returns.
7. Create a WaitGroup for concurrent goroutines.
8. Each goroutine is passed the context, which links them both to the same tracing task.

```go
func main() {
	f, err := os.Create("trace.out")
	if err != nil {
		panic(err)
	}
	defer f.Close()

	if err := trace.Start(f); err != nil {
		panic(err)
	}
	defer trace.Stop()

	ctx := context.Background()

	// Create a top-level trace region
	ctx, task := trace.NewTask(ctx, "main-task")
	defer task.End()

	var wg sync.WaitGroup

	wg.Add(2)

	go worker(ctx, &wg, "worker-1", 100*time.Millisecond)
	go worker(ctx, &wg, "worker-2", 200*time.Millisecond)

	wg.Wait()

	fmt.Println("Program complete")
}
```

To run the file, use `go tool trace`. This command opens your default browser and lets you view the tracing information:

```bash
go tool trace trace.out 
2026/01/17 10:12:02 Preparing trace for viewer...
2026/01/17 10:12:02 Splitting trace for viewer...
2026/01/17 10:12:02 Opening browser. Trace viewer is listening on http://127.0.0.1:33341
```

## HTTP servers

Complex HTTP servers should implement tracing as middleware. You can wrap an HTTP handler with a function that adds tracing to your code. First, create a simple handler. This handler writes a message to the writer:

```go
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, Tracing!")
}
```

Next, define the trace wrapper. This function takes a `HandlerFunc` and returns a `HandlerFunc`.
1. `HandlerFunc` is an adaptor that lets you return a function that will be called later by the HTTP server---rather than returning data, you return executable behavior. This function captures the `inner` argument and returns it for execution at a later time.
2. Return a function with that satisfies the `ServeHTTP` interface.
3. `trace.NewTask` creates a logical unit of work that groups activity across time and across goroutines. For example, across a single HTTP request or CLI command. It takes a context and a task type, which is the task name. Here, it takes the request context and the request path. When its called, it creates a task ID that is passed to the context, which lets you follow a single request across goroutines.
4. `End` marks the end of the request in the trace.
5. `Log` takes a context, optional category, and message. This is a timestamped key-value annotation that is attached to a task or region trace timeline. It adds human readable context to execution.
6. Returns the `inner` handler and passes the context along with the Request.

```go
func TraceHandler(inner http.HandlerFunc) http.HandlerFunc { 	// 1
	return func(w http.ResponseWriter, r *http.Request) { 		// 2
		ctx, task := trace.NewTask(r.Context(), r.URL.Path) 	// 3
		defer task.End() 										// 4

		trace.Log(ctx, "HTTP Method", r.Method) 				// 5
		trace.Log(ctx, "URL", r.URL.String())

		inner(w, r.WithContext(ctx)) 							// 6
	}
}



func main() {
	http.HandleFunc("/", TraceHandler(handler))
	fmt.Println("Server is listening on :8080")
	http.ListenAndServe(":8080", nil)
}
```