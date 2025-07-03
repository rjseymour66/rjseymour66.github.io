---
title: "Logging"
weight: 160
description: >
  Creating and working with loggers in Go.
---

Create a logger with Go's default [*log.Logger](https://pkg.go.dev/log#Logger) package to provide feedback for background processes or error tracking. Go's default logger is concurrency-safe, so you can use a single logger across multiple goroutines without worry.

The following code creates two leveled loggers: one that logs information messages to the console, and one that sends errors to STDERR:

```go
	infoLog := log.New(os.Stdout, "INFO\t", log.Ldate|log.Ltime)
	errorLog := log.New(os.Stderr, "ERROR\t", log.Ldate|log.Ltime|log.Lshortfile)
```
The `New()` function accepts the following arguments:
- An `io.Writer` where you write the logs.
- Prefix for the log message.
- One or more [log package flag constants](https://pkg.go.dev/log#pkg-constants). These constants include contextual information in the log message, such as the date (`Ldate`), the time (`Ltime`), and the file and line number where the error occurred (`Lshortfile`). You can OR multiple flags together.

## Log to a file

By default, Go's `log` library sends messages to STDERR, but you can configure it to write to a file. It adds the date and time to each log entry, and you can add a prefix to the string to help searchability
```go
f, err := os.OpenFile("/path/to/logfile.log", os.O_RDWR|os.O_CREATE, 0666)
if err != nil {
    log.Fatal(err)
}
defer f.Close()

l := log.New(f, "LOGGER PREFIX: ", log.LstdFlags)
```
`log.LstdFlags` uses the default log flags, such as date and time.

## Add logger to server

You can create an error logger and then assign it to a custom server's ErrorLog field:

```go
errorLog := log.New(os.Stderr, "ERROR\t", log.Ldate|log.Ltime|log.Lshortfile)

srv := &http.Server{
    ...
    ErrorLog: errorLog,
    ...
}
```