+++
title = 'Logging'
date = '2025-09-02T08:26:42-04:00'
weight = 10
draft = false
+++

Logs are records of events that occur during the running of an application. When something breaks, they are an invaluable diagnostic resource.

The IETF maintains standards for logging in [RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424).

## Types of logs

You can generate logs for development purposes or observation. These are called debugging and monitoring logs.

### Debugging logs

Debugging logs help during the development phase or when you need to diagnose issues. They provide developers with detailed, contextual information about the application's behavior at a specific moment in time. The most common examples include stack traces and variables at certain checkpoints.

Debugging logs have the following properties:
- Granualr: Detailed an verbose information about the state of the application, variable values, execution paths, and error messages.
- Temporary: These logs are verbose, so they are not running permanently. They might be generated in a development environment or temporarily enabled in production to track down issues.
- Developer focused: The audience is developers that are familiar with the application's code base.

### Monitoring logs

These logs are for ongoing observation of an application in production so you can understand health and usage patterns over time. For example, an HTTP request log that tracks the HTTP method, URL, status code, user agent, etc.

Monitoring logs have the following properties:
- Aggregation friendly: Structured to be easily aggregated and analyzed by monitoring tools. They follow a consistent format.
- Persistent: Continuously generated as part of the app's normal operation, so they are less informative to reduce overhead.
- Operational insight: Provide information relevant to the application, user activity, and error rates.

## Logging formats

The most popular logging formats are JSON and structured text. The following table compares use cases for both:


| Use case                                | JSON Logs                                                                                                                      | Structured Text Logs (key=value)                                                                            |
| :-------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------- |
| **Log consumption tools**               | Excellent support in modern log pipelines (ELK, Loki, Datadog, Splunk). Native parsing with no custom rules required.          | Good support, but often requires regex or logfmt parsers. Less universally standardized than JSON.          |
| **Logging data complexity**             | Handles nested objects, arrays, and rich data naturally. Well suited for complex event data.                                   | Best for flat, simple fields. Nested or hierarchical data becomes awkward or lossy.                         |
| **Performance and overhead**            | Slightly higher CPU and allocation overhead due to encoding and larger payload size.                                           | Lower overhead. Faster to write and smaller on the wire, especially for high-volume logs.                   |
| **Log analysis**                        | Strong querying, filtering, aggregation, and correlation using structured fields. Ideal for dashboards and metrics extraction. | Effective for basic filtering and searches, but advanced analytics may require extra parsing steps.         |
| **Troubleshooting**                     | Excellent for distributed systems where correlation IDs and structured context are critical.                                   | Very readable for humans during live debugging, but less powerful at scale.                                 |
| **Human readability**                   | Lower without formatting; optimized for machines first.                                                                        | Higher; easy to read directly in terminals and text files.                                                  |
| **Schema evolution**                    | More tolerant of adding new fields without breaking consumers.                                                                 | Field additions can break brittle parsers or regex-based tooling.                                           |
| **Development and maintenance context** | Slightly more setup and discipline required, but easier to maintain long-term as systems grow and teams change.                | Faster to start with and simpler locally, but maintenance cost increases as log volume and complexity grow. |
| **Common formats**                      | JSON (NDJSON)                                                                                                                  | logfmt (`key=value`), custom structured text                                                                |
|                                         |

## What to log?

Knowing what to log and what not to log is critical.

### Log

Log these events:

- Errors: Include stack traces to facilitate debugging.
- System state changes: Significant changes in your application, such as system startup or shutdown, configuration changes, status changes of critical components.
- User actions: Actions that modify data or trigger significant processes.

If you do not have a metrics server in place:
- Performance metrics: Response times, throughput, resource utilization, etc.
- Security events: Login attempts, access control violations, etc.
- API calls: Any interactions with external services through APIs.

### Do not log

- Sensitive information: Passwords, personal identification information (PII), credit card numbers, and security tokens.
- Verbose or debug information in prod: Large logging messages can overwhelm a production system.
- Redundant or irrelevant information: Don't log the same information multiple times or capture useless details.
- Large binary data: Do not log files or images, they degrade performance.
- User input without sanitzation: Raw user input can increase security risks, such as injection attacks. Always sanitize user input before logging.



## log package

The log package has the same print methods as the `fmt` package, but it prepends the date and time in `YYY/MM/DD HH:MM:SS` format before the message:

```go
func main() {
	fmt.Println("This is fmt to the console")
	log.Println("This is log to the console")
}
```

The previous snippet outputs the following:

```bash
This is fmt to the console
2025/09/02 08:32:03 This is log to the console
```

### SetFlags

You can customize log output with the `SetFlags` function. This function accepts one or more log-formatting options in a union:

```go
func main() {
    log.Println("Standard flags")               // 2025/09/02 08:51:49 Standard flags
	log.SetFlags(log.Ltime | log.Lshortfile)    // 2025/09/02 08:51:49 Standard flags
	log.Println("Time and short file")          // 08:51:49 main.go:18: Time and short file
}
```

By default, the logger prints the date and time. To unset these defaults, pass `0` to `SetFlags`:

```go
func main() {
	log.SetFlags(0)
}
```

Here is a list of the `log.SetFlags` options with their descriptions. If you do not pass any options, the logger outputs the date and time:

| Flag                | Description                                                       |
| ------------------- | :---------------------------------------------------------------- |
| `log.LstdFlags`     | Default setting: `Ldate \| Ltime`.                                |
| `log.Ldate`         | Prints the local date in the format `2009/01/23`.                 |
| `log.Ltime`         | Prints the local time in the format `01:23:23`.                   |
| `log.Lmicroseconds` | Adds microsecond precision to the time: `01:23:23.123123`.        |
| `log.Llongfile`     | Full file path and line number of the log call: `/a/b/c/d.go:23`. |
| `log.Lshortfile`    | Final file name element and line number: `d.go:23`.               |
| `log.LUTC`          | Use UTC instead of local time for `Ldate` and `Ltime`.            |
| `log.Lmsgprefix`    | Place the log prefix *before* the date/time instead of after.     |

### SetOutput

By default, a `log` type writes to `os.Stderr`. To change this, pass an `io.Writer` such as a file or a custom writer to `SetOutput`:

```go
log.SetOutput(myCustomLogger)
```

## Custom logger

You can create a custom logger by implementing a `Writer` interface on a custom type:

```go
type Writer interface {
    Write(p []bytes) (n int, err error)
}
```

The `Writer` implementation can return by writing the bytes to `os.Stdout` (or another location) directly.

### Formatting output

To create a custom logger, define a struct and a write method. The most important section of the `Write` implementation is the output formatting:
1. Cast the slice of bytes to a string.
2. Format the logger `output` with the date and time and the `msg` string.
3. Write to `os.Stdout` and check for errors.
4. Return the number of bytes and any error.

```go
type customLogger struct{}

func (l *customLogger) Write(p []byte) (int, error) {
	msg := strings.TrimSpace(string(p))                         // 1

	output := fmt.Sprintf("[%s] - %s",                          // 2
		time.Now().Format("2006-01-02 01:02:03"),
		msg,
	)

	n, err := os.Stdout.Write([]byte(output + "\n"))            // 3
	if err != nil {
		return n, fmt.Errorf("Logger failed write: %w", err)
	}

	return n, nil                                               / 4
}
```

You can also use [ANSI color codes](https://gist.github.com/JBlond/2fea43a3049b38287e5e9cefc87b2124) to add color to your logs. 

{{< admonition "ANSI support" warning >}}
Note that many output formats do not support ANSI, so you should enable it with a flag if you use it in your logger.
{{< /admonition >}}

Here is an example `output` function that uses ANSI.

```go
func (l *myLogger) Write(msg []byte) (int, error) {
    // some work
	output := fmt.Sprintf("%s%s - %s%s (called from %s%s)",
		"\033[32m", time.Now().Format("2006/01/02 3:04:05 pm"),
		"\033[0m", strings.TrimSpace(string(msg)),
		"\033[35m", caller)

    return os.Stdout.Write([]byte(output + "\n"))
}
```

### Using the logger

To use the custom logger, you pass it to `log.SetOutput`, which tells the logging package to use your custom logger. While you could write logs with your custom logger directly, this technique gives you access to the log package API, such as `log.Println` while using your custom output:
1. Creates a new `myLogger` instance.
2. Unsets the default logger flags (`Ldate`, `Ltime`)
3. Redirects the log package output to your custom logger
4. Alternate expression that instantiates a logger and injects in log package in one expression

Below 

```go
func main() {
	myLog := new(customLogger)      // 1
	log.SetFlags(0)                 // 2
	log.SetOutput(myLog)            // 3
	
    // Concise injection
    log.SetOutput(&customLogger{})  // 4
}
```

### Logging to a file

Go's logging package can write to any `Writer` type, which includes file handlers:

```go
func main() {
	file, err := os.OpenFile("logging.log", os.O_RDWR|os.O_CREATE, 0755)
	if err != nil {
		panic(errors.New("could not open log file"))
	}
    defer file.Close()

	log.SetOutput(file)
	log.SetFlags(log.LUTC | log.Lshortfile)
	log.Println("Display in UTC and use a short filename")
}
```

## Structured logging

Structured logging organizes log files into a structured format with machine-readable data in key-value format, typically in JSON. This helps you query and analyze log messages. It also includes log levels in its log messages. Log levels help distinguish between messages, increase readability and searchability, simplify parsing, and add context to logs. Many production log analysis tools ingest structured logs.

Go provides structured logging in its `slog` package. This example shows a basic implementation:

```go
func main() {
	slog.Info("default info logger")
	slog.Warn("this is a warning log that might indicate an issue")
	slog.Error("something is broken")
	slog.Debug("development environment message, does not print to the console")
}
```

In the previous example, the `Debug` messages are not logged to the console. You can change that with a custom structured logger.

### Creating a logger

These examples create custom structured loggers:
1. `jsonLogger` creates a new JSON logger that writes to a file with custom options.
2. `textLogger` creates a new text logger that writes to SDTERR. `With` attaches a persistent structured field to each log entry, which adds context and makes the log more searchable and filterable. For example, every log with this logger includes `app=linkd`.

```go
	jsonLogger := slog.New(slog.NewJSONHandler(file, &slog.HandlerOptions{ 		// 1
		Level: slog.LevelDebug,
	}))

	textLogger := slog.New(slog.NewTextHandler(os.Stderr, nil)) 				// 2
		.With("app", "linkd")

	// time=2026-03-07T08:15:00Z level=INFO msg="server started" app=linkd
```


### JSON logging

Go provides tools to simplify structured logging in JSON:
1. Creates a log file.
2. Creates a new structured logger.
   - The `New` method returns a logger and accepts a log handler that handles any logs produced by the logger.
   - `NewJSONHandler` returns a handler that writes log records as line-delimited JSON objects to an `io.Writer`. It accepts the `Writer` type and a `HandlerOptions` struct. Here, the `HandlerOptions` struct sets the logging level at debug or higher.
1. Replaces the default global logger with the new JSON `logger`
2. `Debug` messages log to the console because how the log handler set the logging level.
3. `slog` provides helper functions for each type so you can build key/value structured logs without worrying about formatting. The first value is the log message, which is followed by additional key/value pairs of the specified type.
   
   `String` returns an `slog.Attr` struct, which returns key/value pairs of the given values.
   
   The `Group` attribute lets you create nested structured logs.


```go
func main() {
	file, err := os.OpenFile("structured.log", os.O_RDWR|os.O_CREATE, 0755)     // 1
	if err != nil {
		panic(errors.New("could not open log file"))
	}

	logger := slog.New(slog.NewJSONHandler(file, &slog.HandlerOptions{          // 2
		Level: slog.LevelDebug,
	}))

	slog.SetDefault(logger)                                                     // 3
	slog.Info("default info logger")
	slog.Warn("this is a warning log that might indicate an issue")
	slog.Error("something is broken")
	slog.Debug("development environment message")                               // 4
	slog.Info("complex message example",                                        // 5
		slog.String("accepted values",
			"key/value pairs with specific types for marshalling"),
		slog.Int("an int:", 30),
		slog.Group("grouped_info",
			slog.String("you can", "do this too")))
}
```

This logger produces the following output:

```bash
{"time":"2025-09-04T08:44:51.867026258-04:00","level":"INFO","msg":"default info logger"}
{"time":"2025-09-04T08:44:51.867110342-04:00","level":"WARN","msg":"this is a warning log that might indicate an issue"}
{"time":"2025-09-04T08:44:51.867114852-04:00","level":"ERROR","msg":"something is broken"}
{"time":"2025-09-04T08:44:51.867118309-04:00","level":"DEBUG","msg":"development environment message, does not print to the console"}
{"time":"2025-09-04T08:44:51.867122179-04:00","level":"INFO","msg":"complex message example","accepted values":"key/value pairs with specific types for marshalling","an int:":30,"grouped_info":{"you can":"do this too"}}
```

### Typed attributes

| Function                                             | Description                                                                    |
| ---------------------------------------------------- | ------------------------------------------------------------------------------ |
| **`slog.String(key, value string)`**                 | Adds a string attribute.                                                       |
| **`slog.Int(key string, value int)`**                | Adds an integer attribute.                                                     |
| **`slog.Int64(key string, value int64)`**            | Explicit 64-bit int.                                                           |
| **`slog.Uint64(key string, value uint64)`**          | Unsigned 64-bit int.                                                           |
| **`slog.Float64(key string, value float64)`**        | Floating-point number.                                                         |
| **`slog.Bool(key string, value bool)`**              | Boolean value.                                                                 |
| **`slog.Duration(key string, value time.Duration)`** | Duration in nanoseconds.                                                       |
| **`slog.Time(key string, value time.Time)`**         | Timestamp.                                                                     |
| **`slog.Any(key string, value any)`**                | Generic — lets slog infer the type. Use for custom structs, slices, maps, etc. |
| **`slog.Group(key string, attrs ...slog.Attr)`**     | Groups attributes under a nested object.                                       |

## Syslog

Syslog is system-level logging that writes to the `syslog` daemon, such as `rsyslog` or `journald`. It is not supported across all operating systems, so you should probably use the `slog` package for application-level logging and collect logs with another method.

Here is basic implementation:

```go
var logger *log.Logger

func init() {
	var err error
	logger, err = syslog.NewLogger(syslog.LOG_USER|syslog.LOG_NOTICE, 0)
	if err != nil {
		log.Fatal("cannot write to syslog: ", err)
	}
}
```

### Log levels

Syslog uses log levels, which indicate the event's severity and importance. This table describes the log levels as defined in RFC 5424, the syslog protocol spec:

| Code | Level     | Description                                                  |
| ---- | --------- | ------------------------------------------------------------ |
| 0    | EMERGENCY | System is unusable.                                          |
| 1    | ALERT     | Action must be taken immediately.                            |
| 2    | CRITICAL  | Critical conditions (hard failures).                         |
| 3    | ERROR     | Error conditions.                                            |
| 4    | WARNING   | Warning conditions (not an error, but something to look at). |
| 5    | NOTICE    | Normal but significant condition.                            |
| 6    | INFO      | Informational messages.                                      |
| 7    | DEBUG     | Debug-level messages, lowest priority.                       |

## Stack traces

The stack trace (also stack dump) lets you fetch a human-readable list of the functions in use at a critical time in your application. Go's `runtime` package provides utilities to fetch the stack trace.

### Log to stdout
This example demonstrates a simple way to retrieve the call stack. The `debug.PrintStack` functon logs the trace to STDOUT:

```go
func main() { first() }
func first() { second() }
func second() { third() }
func third() {
	debug.PrintStack()
}
```
The output shows how the call stack unwinds from the `PrintStack` function up to main. The first few functions are in the Go library, and the remaining functions are in the program's `main` function in the order in which they were invoked, along with the line number where they were invoked:

```bash
goroutine 1 [running]:
runtime/debug.Stack()
	/usr/local/go/src/runtime/debug/stack.go:26 +0x5e       # source files
runtime/debug.PrintStack()
	/usr/local/go/src/runtime/debug/stack.go:18 +0x13
main.third(...)                                             # program files
	/path/to/main.go:20
main.second(...)
	/path/to/main.go:16
main.first(...)
	/path/to/main.go:12
main.main()
	/path/to/main.go:8 +0x12
```

### Persist trace

If you want to send the trace somewhere other than stdout, use the `Stack` function.

This example has a `persistentTrace` funciton that creates and opens a file, then writes the stack trace to that file. A `caller` funciton calls `persistentTrace`, and `main` calls `caller`:
1. Create the file and defer its close until the function returns.
2. Create a bytes buffer to store the trace. `Stack` takes a bytes buffer and a boolean flag that determines whether to print the stack trace for all running goroutines. Setting this to `true` can substantially increase the output.
   
   {{< admonition "Buffer size" note >}}
   There is no way to anticipate the size of the trace, so use your best judgement when allocating the bytes buffer.
   {{< /admonition >}}
3. Write the buffer to the file.

```go
func main() {
	caller()
}

func caller() {
	persistentTrace()
}

func persistentTrace() {
	f, err := os.OpenFile("trace.file", os.O_RDWR|os.O_CREATE, 0755)    // 1
	if err != nil {
		panic(errors.New("could not open log file"))
	}
	defer f.Close()

	buf := make([]byte, 1024)                                           // 2
	runtime.Stack(buf, false)

	_, err = f.Write(buf)                                               // 3
	if err != nil {
		log.Fatalf("Error writing bytes: %v", err)
	}
    fmt.Printf("Trace written to %s\n", f.Name())
}
```