+++
title = 'Iterators (TODO)'
date = '2026-02-21T10:20:17-05:00'
weight = 20
draft = false
+++


Beginning with Go 1.23, the standard library supports custom iterators. An _iterator_ is a function that standardizes the way you push values from a sequence (e.g. a slice or channel) to a consumer. This gives consumers more control to  decide how they retrieve values from the iterator.

Prefer using an iterator rather than a slice because the consumer doesn't have to preallocate memory for a slice that might be large.

Read the [Range Over Function Types](https://go.dev/blog/range-functions) blog post for more information.

## Push iterators

Push iterators push values from a sequence of values into a `yield` function that the iterator's consumer provides. Here is the signature:

```go
type Seq[V any] func(yield func(V) bool)
```

The return value of `yield` tells the iterator to push more values or stop.

### Implementation

This declaration creates an iterator function named `Results` that pushes a sequence of `Result` values:
1. This type definition creates a new iterator function type named `Results`.  

```go
type Results iter.Seq[Result]

type Result struct {
	Status   int
	Bytes    int64
	Duration time.Duration
	Error    error
}
```

Conceptually, `Results` has this function signature:

```go
type Results func(yield func(Result) bool)
```
This function type accepts a callback `yield` that takes a `Result` and returns a Boolean. This means that `Results` does not return `Result` values, it pushes each `Result` into the callback.

#### Producer

The iterator function definition is not yet implemented---we need to write a function that returns an iterator with the required signature.

`SendN` returns a `Results` iterator as a closure.



1. An anonymous function that captures the `SendN` parameters, `n` and `req`. It returns a `Results` iterator and an `error`.
   
   It "captures" these values because the `yield` signature cannot explicitly accept them. Instead, it uses them in its logic. After `SendN` returns, these variables remain inside the closure.

   For example, you call `SendN` like this:
   ```go
   results, err := SendN(100, req)
   ```
   This returns an anonymous `yield` function that includes `100` and `req` as values, but the function has not been executed.
2. Range over `n`, the number of requests.
3. This line is where the value is pushed. Get the `result` of the `Send` function. `Send` takes the captured `req` value.
4. If `yield` returns false, stop consuming.

```go
func SendN(n int, req *http.Request) (Results, error) {
	if n <= 0 {
		return nil, fmt.Errorf("n must be positive: got %d,", n)
	}

	return func(yield func(Result) bool) {
		for range n {
			result := Send(http.DefaultClient, req)
			if !yield(result) {
				return
			}
		}
	}, nil
}
```









1. SendN returns a Results iterator
2. Consumers provide the `yield` function to the iterator
3. Iterator generates a Result for each request, then calls the consumers `yield` function. This function pushes the Result to the consumer.
4. Step 3 continues until the iterator pushes each Result or the consumer's yield function returns false.



`Send` mimics an HTTP call that returns a `Result` struct:
```go
func Send(_ *http.Client, _ *http.Request) Result {
	const roundTripTime = 100 * time.Millisecond

	time.Sleep(roundTripTime)

	return Result{
		Status:   http.StatusOK,
		Bytes:    10,
		Duration: roundTripTime,
	}
}
```

`SendN` returns a `Results` iterator, which is a closure.
1. An anonymous function that captures the `SendN` parameters, `n` and `req`. It returns a `Results` iterator and an `error`.
   
   It "captures" these values because the `yield` signature cannot explicitly accept them. Instead, it uses them in its logic. After `SendN` returns, these variables remain inside the closure.

   For example, you call `SendN` like this:
   ```go
   results, err := SendN(100, req)
   ```
   This returns an anonymous `yield` function that includes `100` and `req` as values, but the function has not been executed.
2. Range over `n`, the number of requests.
3. Get the `result` of the `Send` function. `Send` takes the captured `req` value.
4. If `yield` returns false, stop consuming.

```go
// SendN sends N requests using [Send].
// It returns a single-use [Results] iterator that pushes a
// [Result] for each [http.Request] sent.
func SendN(n int, req *http.Request) (Results, error) {
	if n <= 0 {
		return nil, fmt.Errorf("n must be positive: got %d,", n)
	}

	return func(yield func(Result) bool) {
		for range n {
			result := Send(http.DefaultClient, req)
			if !yield(result) {
				return
			}
		}
	}, nil
}
```

## Consumer

- A `nil` iterator is results in a panic
- Compiler has built-in support for iterators, so you can use a `for range` loop.


```go
// result.go
type Summary struct {
	Requests int
	Errors   int
	Bytes    int64
	RPS      float64
	Duration time.Duration
	Fastest  time.Duration
	Slowest  time.Duration
	Success  float64
}

// Summarize returns a [Summary] of [Results].
func Summarize(results Results) Summary {
	var s Summary
	if results == nil {
		return s
	}

	started := time.Now()
	for r := range results {
		s.Requests++
		s.Bytes += r.Bytes

		if r.Error != nil || r.Status != http.StatusOK {
			s.Errors++
		}
		if s.Fastest == 0 {
			s.Fastest = r.Duration
		}
		if r.Duration < s.Fastest {
			s.Fastest = r.Duration
		}
		if r.Duration > s.Slowest {
			s.Slowest = r.Duration
		}
	}
	if s.Requests > 0 {
		s.Success = (float64(s.Requests-s.Errors) /
			float64(s.Requests)) * 100
	}
	s.Duration = time.Since(started) // makes sure you don't return a nil
	s.RPS = float64(s.Requests) / s.Duration.Seconds()

	return s
}
```

## Testing 