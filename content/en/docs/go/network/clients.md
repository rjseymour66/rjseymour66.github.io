---
title: "Clients"
weight: 60
description: >
  Working with Clients in Go.
---


## Clients

In Go, a single client can create multiple connections.

### Creating clients

Go provides a default client, but you cannot customize it with functionality such as a connection timeout:

```go
func newClient() *http.Client {
	c := &http.Client{
		Timeout: 10 * time.Second,
	}
	return c
}
```

### Model responses

You have to model responses with structs. Create a struct to model an individual resource and a struct to model the server response.

### Sending requests

Create a generic method that can send any type of request and handle any response code. The `.Do()` method can send any type of request (GET, POST, PUT, DELETE, etc.).

A request should perform the following:
- Create a request object with [.NewRequest(method, url string, body io.Reader)](https://pkg.go.dev/net/http#NewRequest)
- Set any content headers with [Header.Set(header-name, value)](https://pkg.go.dev/net/http#Header.Set)
- Execute the request with the .Do() method. Save the response in a var
- Close the response body (sooner than later)
- Check that the response code is what you expected. If not, use custom error messages with the `%w` formatting verb.
- If the request was successful, return `nil`

For example:

```go
func sendRequest(url, method, contentType string, expStatus int, body io.Reader) error {
    // create a new request
	req, err := http.NewRequest(method, url, body)
	if err != nil {
		return err
	}

    // Set any headers
	if contentType != "" {
		req.Header.Set("Content-Type", contentType)
	}

    // execute the request with .Do() and save the response in a var
	r, err := newClient().Do(req)
	if err != nil {
		return err
	}

    // make sure the response body is closed 
	defer r.Body.Close()

    // check status codes
	if r.StatusCode != expStatus {
		msg, err := io.ReadAll(r.Body)
		if err != nil {
			return fmt.Errorf("Cannot read body: %w", err)
		}
		err = ErrInvalidResponse
		if r.StatusCode == http.StatusNotFound {
			err = ErrNotFound
		}
		return fmt.Errorf("%w: %s", err, msg)
	}

    // return nil for a successful request
	return nil
}
```

### CRUD functions

Use the following functions with the generic `sendRequest()` function for CRUD operations:

```go


// PATCH
// Use Sprintf to format a url with query parameters
func completeItem(apiRoot string, id int) error {
	u := fmt.Sprintf("%s/todo/%d?complete", apiRoot, id)

	return sendRequest(u, http.MethodPatch, "", http.StatusNoContent, nil)
}
```

### Integration tests

When you run unit tests, you are using local resources that mock the live API. You can run these as much as you'd like. However, to run integration tests, you need to run your client against the actual API. To make sure that you do not make too many requests to the actual API, use build constraints.

Main challenge is that the test needs to be reproducible. 

Define build constraints at the top of the file:

```go
// +build integration

package cmd

// file contents...
```

To exclude a file from integration tests, use the `!` operator before `integration`:

```go
//go:build !integration

package cmd

// file contents
```
When you run the tests, use `-tags <tag-name>` in the command. For example:
```shell
$ go test -v ./cmd -tags integration
```

After you run the integration tests one time, add the `-count=1` tag to ensure that the test does not used cached results:

```shell
$ go test -v ./cmd -tags integration -count=1
```

-------------------------------------------------------------------------

The HTTP protocol exchanges plain text messages between a server and client: the client sends a request with a simple text message, and the server returns a response body.

## Design

An HTTP client should have a `Client` type that sends requests, and a `Results` type that models the server response.

Things to consider when designing a client:
- Easy to use
- Hides internal complexity
- Consists of composable parts that users can bring together
- Synchronous by default
- Allow users to fine-tune API behavior

## Client type

[httbin.org](http://httpbin.org/) test server.

### Client Connections

TCP connections are expensive, so the HTTP protocol has a caching mechanism called _keep-alive_ that keeps established client/server connections open until a timeout. Then, a client can use the same connection to send HTTP requests without establishing a new connection.

Go's `DefaultClient` keeps 100 connections open and only allows you to reuse 2. You can optimize the connection pool with the Go [`Transport`](https://pkg.go.dev/net/http#Transport) type.


### Responses

You read a response body incrementally, as a stream of bytes. Create a `bytes.Buffer` and read the stream little by little until you read the entire body.

> Use an `io.Reader` to read any resource, and use an `io.Writer` to write to any resource. You can also use [`io.Copy(w, r)`](https://pkg.go.dev/io#Copy) that writes directly to a writer from a reader. Use the [`Discard`](https://pkg.go.dev/io#Discard) variable (of type `Writer`) to discard anything after you read it. You can treat `Discard` as `/dev/null`. This method preserves resources.

## Testing HTTP Clients 

Create handlers depending on what you want to test. For example, if you want to test successful HTTP requests:
1. Create a handler that responds with HTTP status code 200
2. Launch a test server with `httptest.NewServer(handler)`. For testing criteria:
   - Request: input
   - ResponseWriter: output

The test server requires a handler to handle requests and responses. The `Handler` interface has the following signature:

```go 
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
`Request` is the input, and `ResponseWriter` handles the output. After you create a handler that satisfies this interface, you pass it to the `NewServer` function to launch the test server. `NewServer` returns a `Server` server that contains a `URL` value to send requests.

Instead of writing an entirely new type to satisfy the `Handler` interface, Go provides the [`HandlerFunc` type](https://pkg.go.dev/net/http#HandlerFunc--a function that has a `ServeHTTP` method. This means that you can create a function that performs some action with a `Request` and `ResponseWriter`, then you can pass it to `HandlerFunc` to start a test server.

So, Go provides the `HandlerFunc` type:
```go 
type HandlerFunc func(ResponseWriter, *Request)
 
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    // Forwards the call to the converted function.
    f(w, r)
}
```
1. Create a function with the same signature as HandlerFunc:
   ```go
	handler := func(w http.ResponseWriter, r *http.Request) {
		// logic
	}
   ```
2. Convert that function to a `Handler` type with the `HandlerFunc`. `HandlerFunc` is an adapter that creates HTTP handlers from ordinary functions:
   ```go
	httpHandler := http.HandlerFunc(handler)
   ```
3. Pass the new handler to the `http.HandlerFunc(func)` method.
   ```go
	server := httptest.NewServer(httpHandler)
   ```
HTTP handlers in GO are concurrent, so the test server handles each request in its own goroutine. When you are tracking the number of requests, you should use the `atomic` package's concurrency-safe counters.


### httptest server

[httptest](https://pkg.go.dev/net/http/httptest)

Prefer the `.Cleanup(server.Close)` function over `defer` functions when testing. The `Cleanup` function runs after the tests complete rather than the enclosing function. This is useful with test helpers.