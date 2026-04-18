+++
title = 'Concurrent pipeline'
date = '2026-02-22T13:33:26-05:00'
weight = 30
draft = false
+++

A concurrent pipeline consists of several stages that work concurrently and are connected by channels. It is similar to a UNIX pipeline command, where the first stage produces a value, the next stages perform an operation on a value, and the last stage delivers the result to output. For example, this command has three stages, each connected by a pipe (`|`):
1. Stage 1: `echo` prints its arguments to STDOUT.
2. Stage 2: `tr` translates the output from stage 1, deleting the whitespace.
3. Stage 3: `wc` counts the characters in the output.

```bash
echo 'this is a test' | tr -d ' ' | wc -c
```

## Building a pipeline

In Go, a concurrent pipeline consists of any of the following components:
- Producer: Produces messages and sends them to the next stage.
- Throttler: Slows the passage of messages between producer and consumer.
- Dispatcher: Specialized goroutine that manages a worker pool of goroutines.
- `runPipeline`: Function that ties everything together
- Iterator: A function type that standardizes how values are retrieved by the pipeline's consumer.

### Configuration

This example is a CLI app that uses the following configuration:

```go
type config struct {
	url string
	n   int
	c   int
	rps int
}
```

This configuration is set with an `Options` type that you can initialize with different configurations:

```go
type Options struct {
	Concurrency int
	RPS         int
	Send        SendFunc
}

// Defaults returns the default [Options].
func Defaults() Options {
	return withDefaults(Options{})
}

func withDefaults(o Options) Options {
	if o.Concurrency == 0 {
		o.Concurrency = 1
	}
	if o.Send == nil {
		o.Send = func(r *http.Request) Result {
			return Send(http.DefaultClient, r)
		}
	}
	return o
}
```

### Producer

- Only has an output channel
- initializes a buffered channel, launches a goroutine that sends the request to the output channel `n` times
  - buffered channels hold only one value at a time and block until the value is received from another channel. When this producer sends the last value on the channel, the `defer close` line runs. This guarantees that the channel closes and that all values are delivered.
- Returns an output channel (receive channel type) after filling it with the given number of requests.
- The next stage 

```go
func produce(n int, req *http.Request) <-chan *http.Request {
	out := make(chan *http.Request)

	go func() {
		defer close(out)
		for range n {
			out <- req
		}
	}()
	return out
}
```

### Throttler

- Because this sends HTTP requests, we add a throttler to slow the message flow so we don't overload the server
- Gets messages from the producer with `in`. `in` is a receive channel (receive data from the channel)
- Returns a receive-only channel, which means that the caller can only receive from this channel, not send
- Uses a time.Ticker to implement the delay. A ticker sends the current time on a channel at regular intervals (`delay`). `<-t.C` sends a value, but since we don't need to do anything with its value, we discard it. The program blocks until `t.C` sends the value.

```go
func throttle(in <-chan *http.Request, delay time.Duration) <-chan *http.Request {
	out := make(chan *http.Request)

	go func() {
		defer close(out)
		t := time.NewTicker(delay)
		for r := range in {
			<-t.C
			out <- r
		}
	}()
	return out
}
```

### Dispatcher
- Uses the _fan-out_ pattern, which distributes incoming tasks across goroutines.
- Because it accepts a `SendFunc` that handles the HTTP logic, the dispatcher is decoupled from the request-handling specifics.
- Creates multiple goroutines to send HTTP reqs to the server concurrently and gather results.
- Takes a recieve-only channel of HTTP requests, concurrency value, and a function of type `SendFunc`, which returns a `Result`.
- Returns a receive-only channel 
- The first goroutine ranges over the `in` channel of HTTP requests, sends the requests with the given `SendFunc`, and stores the returned `Result` types in the `out` channel that is returned by the Dispatcher.
- the goroutine with `wg.Wait` is the "monitoring" goroutine, which monitors the active workers and closes the output channel when all worker goroutines complete their work (call `wg.Done()`). The monitoring goroutine blocks on `wg.Wait()` until its counter hits 0, then it closes the output channel.


```go
func dispatch(in <-chan *http.Request, concurrency int, send SendFunc) <-chan Result {
	out := make(chan Result)
	var wg sync.WaitGroup
	wg.Add(concurrency)

	for range concurrency {
		go func() {
			defer wg.Done()
			for req := range in {
				out <- send(req)
			}
		}()
	}

	go func() {
		wg.Wait()
		close(out)
	}()

	return out
}
```

### Running the pipeline

1. Produce `n` requests and put them in the `requests` channel.
2. If `RPS` is set in the config, then send the channel to the throttler. Delay each request by the `RPS` value.
   1. If `RPS` is not set, skip the throttler and go directly to the dispatcher.
3. Return the dispatcher's `out` channel, which is a receive-only channel of `Result` values.

```go
func runPipeline(n int, req *http.Request, opts Options) <-chan Result {
	requests := produce(n, req)
	if opts.RPS > 0 {
		requests = throttle(
			requests, time.Second/time.Duration(opts.RPS),
		)
	}
	return dispatch(requests, opts.Concurrency, opts.Send)
}
```

The caller would assign the output of `runPipeline` to a receive channel (a channel you read data from). For example, this function assigns `runPipeline` to `results`, and then uses `results` as the values in an iterator that reads from the channel until it is empty:

```go
func SendN(n int, req *http.Request, opts Options) (Results, error) {
	opts = withDefaults(opts)
	if n <= 0 {
		return nil, fmt.Errorf("n must be positive: got %d,", n)
	}

	results := runPipeline(n, req, opts)            // assignment
	return func(yield func(Result) bool) {          // iterator yield function
		for result := range results {
			if !yield(result) {
				return
			}
		}
	}, nil
}
```