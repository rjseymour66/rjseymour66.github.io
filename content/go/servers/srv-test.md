+++
title = 'Testing Servers and Services'
linkTitle = "Testing"
date = '2026-03-01T16:09:09-05:00'
weight = 70
draft = false
+++

## Servers

`httptest.Server` is a server that is tuned for testing. It serves HTTP requests with an [`http.Handler` interface](../http-servers/#handler-interface).

This example tests that the `SendN` function sends the correct amount of requests to a server. It uses 5 concurrent goroutines. Because we are counting concurrent tasks, we use an `atomic` counter:
1. Create a thread-safe counter. Use the `atomic` package rather than a normal `int` to prevent a race condition.
2. Create the test server. This passes a `HandlerFunc` that converts a function into an `http.Handler`. This handler increments the `hits` atomic variable.
3. Call the function you are testing. `t.Context` returns a context tied to the life of the test.
4. When you run concurrent tests that use a channel, you need to make sure you consume the results, or the channel might block and break your test.
5. The `Load` function retrieves the number stored in the atomic variable.

```go
func TestSendN(t *testing.T) {
	t.Parallel()

	var hits atomic.Int64 													// 1

	srv := httptest.NewServer(http.HandlerFunc( 							// 2
		func(_ http.ResponseWriter, _ *http.Request) {
			hits.Add(1)
		},
	))
	defer srv.Close()

	req, err := http.NewRequest(http.MethodGet, srv.URL, http.NoBody)
	if err != nil {
		t.Fatalf("creating http requests: %v", err)
	}
	results, err := SendN(t.Context(), 10, req, Options{ 					// 3
		Concurrency: 5,
	})
	if err != nil {
		t.Fatalf("SendN() err=%v, want nil", err)
	}

	for range results { // just consume the results 						// 4
	}

	if got := hits.Load(); got != 10 { 										// 5
		t.Errorf("got %d hits, want 10", got)
	}
}
```

## Handlers

Testing a handler involves providing a `Request` and `ResponseWriter` and observing its response. You can inspect a handler's response with `httptest.ResponseRecorder` to verify it responds with what you expect.

### Response recording

`ResponseRecorder` is a `ResponseWriter`, so you can pass it to a handler to observe its response.

Here is table of useful fields:

| Field       | Type            | Purpose                                      |
| :---------- | :-------------- | :------------------------------------------- |
| `Code`      | `int`           | HTTP status code written by the handler      |
| `Body`      | `*bytes.Buffer` | Contains the response body                   |
| `HeaderMap` | `http.Header`   | Stored headers (deprecated — use `Header()`) |
| `Flushed`   | `bool`          | Indicates whether `Flush()` was called       |


#### Example 

For example, we want to test this `Health` handler function that sends an "OK" response to requests:

```go
func Health(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "OK")
}
```

First, create a test helper so you can create a new request. The `http.NewRequest()` function panics instead of returning an error, so we handle this with a helper:
1. You are using the helper function for unit testing, so you do not need to export it.
2. Use `testing.TB` so you can use the function for both tests and benchmarks.
3. Don't return an error from a test helper or the caller has to handle the error.

```go
func newRequest( 		// 1
	tb testing.TB, 		// 2
	method string,
	target string,
	body io.Reader,
) *http.Request {
	tb.Helper() 		// 3

	r, err := http.NewRequest(method, target, body)
	if err != nil {
		tb.Fatalf("newRequest() err = %v, want nil", err)
	}
	return r
}
```

Create the `TestHealth` function that validates the handler response with a `ResponseRecorder`:
1. Create the recorder.
2. Call `Health`. You don't have to call this with a server because it is a regular function that you convert to a handler with `HandleFunc`. You can create the `Request` inline with `NewRequest`.
3. Create a new request with the `newRequest` helper.
4. Check the response code.
5. Check the response body.

```go
func TestHealth(t *testing.T) { 												// 1
	rec := httptest.NewRecorder() 												// 2
	Health(rec, newRequest(t, http.MethodGet, "/ ", http.NoBody)) 				// 3

	if rec.Code != http.StatusOK { 									 			// 4
		t.Errorf("got status code = %d, want %d", rec.Code, http.StatusOK)
	}
	if got := rec.Body.String(); !strings.Contains(got, "OK") { 				// 5
		t.Errorf("\ngot body = %s\nwant contains %s", got, "OK")
	}
}
```

## Services


Test HTTP applications or services with the `httptest.NewRecorder`, which you can use to create an `httptest.ResponseRecorder` to capture what was written the the `ResponseWriter`.

To demonstrate, here is a simple handler function that returns "Hello World" in the response:

```go
func hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World!")
}
```

To test, you need to register the handler function, create a `NewRecorder` that can simulate a ResponseWriter, create a test server, then check the values in the ResponseWriter:
1. Register the HTTP handler function to a path. This registers the handler to `http.DefaultServerMux`.
2. `httptest.NewRecorder()` helps test what was written to the ResponseWriter. It returns an initialized `ResponseRecorder`. A `ResponseRecorder` has the following fields: `Code`, `HeaderMap`, `Body`, and `Flushed`.
3. Create a new HTTP request.
4. Serve the request using a handler and the ResponseWriter you created. `ServeHTTP` dispatches the request to the handler function that matches its URL.
5. Check the response status code.
6. Check the response body.

```go
func TestHttpHello(t *testing.T) {
	http.HandleFunc("/hello", hello)                        // 1
	writer := httptest.NewRecorder()                        // 2
	request, _ := http.NewRequest("GET", "/hello", nil)     // 3
	http.DefaultServeMux.ServeHTTP(writer, request)         // 4

	if writer.Code != http.StatusOK {                       // 5
		t.Errorf("Response code is %v", writer.Code)
	}

	if expected, actual := "Hello, World!", writer.Body.String(); expected != actual {      // 6
		t.Errorf("Response body is %v", actual)
	}
}
```



## Methods 


### Server methods

| Method                     | Signature                                   | What It Does                                      | When You Use It                                |
| :------------------------- | :------------------------------------------ | :------------------------------------------------ | :--------------------------------------------- |
| `Close()`                  | `func (s *Server) Close()`                  | Shuts down the server immediately                 | Always call to clean up                        |
| `CloseClientConnections()` | `func (s *Server) CloseClientConnections()` | Closes all active client connections              | Test retry logic or dropped connections        |
| `URL` (field)              | string                                      | Base URL of the test server                       | Use to build request URLs                      |
| `Listener` (field)         | `net.Listener`                              | Underlying network listener                       | Advanced customization                         |
| `Client()`                 | `func (s *Server) Client() *http.Client`    | Returns an HTTP client configured for this server | Safest way to make requests to the test server |


### Constructor methods


| Function           | Signature                                       | What It Does                                                                         |
| :----------------- | :---------------------------------------------- | :----------------------------------------------------------------------------------- |
| NewServer          | func NewServer(h http.Handler) *Server          | Starts an HTTP test server immediately                                               |
| NewTLSServer       | func NewTLSServer(h http.Handler) *Server       | Starts an HTTPS test server immediately                                              |
| NewUnstartedServer | func NewUnstartedServer(h http.Handler) *Server | Creates a server without starting it (allows configuration before Start or StartTLS) |
