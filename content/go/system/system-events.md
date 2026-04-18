+++
title = 'System events'
date = '2025-12-03T21:50:21-05:00'
weight = 30
draft = false
+++

A signal notifies a process that a specific event was triggered by one of these sources:
- Hardware: Hardware detects a fault condition. It notifies the kernel and dispatches the signal to the affected process.
- User: Generated from the terminal, such as an interrupt (Ctrl + C).
- Software: Terminated child process that is a associated with the main process.

Signals are a form of inter-process communication (IPC) and are crucial for several reasons:
- Graceful shutdown: Properly handling signals such as `SIGTERM` and `SIGINT` lets an application close its resources, save state, and exit cleanly. See [Graceful shutdown](../../servers/http-servers#graceful-shutdown).
- Resource management: Send signals such as `SIGUSR1` and `SIGUSR2` to trigger an application to rotate logs, reload configs without downtime, or perform housekeeping tasks.
- Inter-process communication: Instruct a process to perform specific actions, like pause (`SIGSTOP`) or resume (`SIGCONT`). 
- Emergency stops: Stop a program after a critical error with `SIGKILL` or `SIGABRT`.

## Signals in Go

Go handles system signals with the `os/signals` package. There are synchronous and asynchronous signals:

Synchronous
: Triggered by errors in program execution. These signals are converted into runtime panics. Examples include the following:
  - `SIGBUS`
  - `SIGFPE`
  - `SIGSEGV`

Asynchronous
: Sent from the kernel or some other process. These are not triggered by program execution.


## Handling signals

There are a few mechanisms to handle signals:
- Buffered channel to receive a signal.
- Buffered channel to communicate that the signal was received.
- Notify function that registers the expected signal to the channel that receives the signal.
- A goroutine that listens for the expected signal within an infinite loop.

This example shows how you can implement all these mechanisms to listen for an interrupt signal:

1. Create a buffered channel of type `os.Signal` to recieve signals from the OS.
2. Create a buffered channel of empty structs that is used to program when the signal should exit. This is a common idiom for handling signals in Go.
   An empty struct is the cheapest type in Go: it has no fields and occupies 0 bytes. This works great for signals because you care only about the signal, not any data. Sending a 0-byte value is the most efficient and clear way to send signals.
3. `signal.Notify` registers the given signal to the a channel of type `os.Signal`. When the program receives the given signal, it is relayed to the given signal channel. Here, we register `os.Interrupt` signal with the `signals` channel.
4. Handle incoming signals in a goroutine.
5. Loop forever.
6. A buffered channel blocks until another it receives a value from another goroutine. This line blocks until an interrupt signal is sent on the `signals` channel.
7. When the signal is received, use a `switch` statement to determine the signal.
8. If it is an interrupt signal, send an empty struct to the done channel. This allows the program to continue execution in the main thread.
9. When an interrupt signal is received in the done channel, `main` can continue execution. This line blocks until it is received. 

```go
func main() {
	signals := make(chan os.Signal, 1)          // 1
	done := make(chan struct{}, 1)              // 2

	signal.Notify(signals, os.Interrupt)        // 3

	go func() {                                 // 4
		for {                                   // 5
			s := <-signals                      // 6
			switch s {
			case os.Interrupt:                  // 7
				fmt.Println("INTERRUPT")
				done <- struct{}{}              // 8
			default:
				fmt.Println("OTHER")
			}
		}
	}()

	fmt.Println("awaiting signal")
	<-done                                      // 9
	fmt.Println("exiting")
}
```






## Examples (todo)

### Generic graceful shutdown

```go
func main() {
	// Create a cancellable context that is automatically cancelled on SIGINT or SIGTERM.
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	// Start your background workers, goroutines, etc.
	go worker(ctx)

	fmt.Println("Application started. Press Ctrl+C to exit.")

	// Block until a shutdown signal arrives.
	<-ctx.Done()
	fmt.Println("Shutdown signal received.")

	// Optional: give workers time to clean up.
	shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Perform cleanup (DB close, flush logs, stop workers)
	if err := cleanup(shutdownCtx); err != nil {
		fmt.Println("Cleanup timed out:", err)
	} else {
		fmt.Println("Cleanup complete.")
	}

	fmt.Println("Exiting gracefully.")
}

// Sample worker
func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Worker stopping...")
			return
		default:
			// Do work
			time.Sleep(500 * time.Millisecond)
		}
	}
}

// Example cleanup function
func cleanup(ctx context.Context) error {
	select {
	case <-time.After(2 * time.Second): // simulate cleanup time
		return nil
	case <-ctx.Done():
		return ctx.Err()
	}
}
```

### HTTP server shutdown

```go
func main() {
	server := &http.Server{
		Addr:         ":8080",
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
		IdleTimeout:  120 * time.Second,
		Handler:      http.HandlerFunc(handler),
	}

	// Channel for graceful shutdown
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	// Start server in a goroutine
	go func() {
		fmt.Println("Server running on :8080")
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			panic(err)
		}
	}()

	// Wait for SIGINT/SIGTERM
	<-ctx.Done()
	fmt.Println("Shutdown signal received.")

	// Create shutdown context (timeout for draining)
	shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	// Gracefully shut down
	if err := server.Shutdown(shutdownCtx); err != nil {
		fmt.Println("Graceful shutdown failed:", err)
		if err := server.Close(); err != nil {
			fmt.Println("Forced close failed:", err)
		}
	}

	fmt.Println("HTTP server stopped cleanly.")
}

func handler(w http.ResponseWriter, r *http.Request) {
	time.Sleep(2 * time.Second) // simulate work
	w.Write([]byte("Hello"))
}
```





















## Common signals


| **Signal** | **Name**       | **Typical Meaning / When It Happens**                        | **Why You Handle It in Go**                                  |
| ---------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `SIGINT`   | Interrupt      | User pressed **Ctrl+C** in the terminal                      | Graceful shutdown (close DB, flush logs, stop goroutines)    |
| `SIGTERM`  | Terminate      | “Please stop” sent by system tools (`kill`, systemd, Docker) | Primary signal for graceful shutdown in production           |
| `SIGQUIT`  | Quit           | Ctrl+\ — quits and produces a core dump                      | Rarely handled; sometimes treated like SIGINT                |
| `SIGHUP`   | Hangup         | Terminal closed OR system wants app to reload config         | Reload configuration files without restarting                |
| `SIGUSR1`  | User-defined 1 | Custom behavior (app-specific)                               | Toggle debug logging, rotate logs, dump state                |
| `SIGUSR2`  | User-defined 2 | Custom behavior                                              | Hot reloads, triggers maintenance mode, etc.                 |
| `SIGALRM`  | Alarm          | Timer expired                                                | Legacy; usually handled via timers instead of actual signals |
| `SIGCHLD`  | Child          | A child process exited                                       | Required if you manually spawn processes (rare in Go)        |
| `SIGPIPE`  | Broken pipe    | Writing to a closed pipe or socket                           | Often ignored; prevents crashes in CLI tools                 |
| `SIGKILL`  | Kill           | Forced kill (`kill -9`)                                      | Cannot be caught or handled                                  |
| `SIGSTOP`  | Stop           | Pause the process                                            | Cannot be caught or handled                                  |
