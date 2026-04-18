+++
title = 'HTTP client'
date = '2025-09-25T08:29:56-04:00'
weight = 10
draft = false
+++

{{< admonition "HTTPBin for testing" note >}}
You can test clients with [HTTPBin](https://httpbin.org/).
{{< /admonition >}}

Network programming in Go uses the `http` package---built on top of the `net` package---to perform the fundamental aspects of network programming:
1. Establish a connection to a remote server.
2. Retrieve data.
3. Close the connection gracefully.

Go's HTTP client can perform almost any HTTP request, and it is highly customizable. Sending a request and returning a response is an HTTP round trip.

## Basic client

A simple Go client performs a a helper function. Helper functions are a wrapper around a request a `Request` object and HTTP client. Other common helper functions include the following:
-`http.Get`
- `http.Head`
- `http.Post`
- `http.PostForm`

This example demonstrates a GET request:
1. Makes a GET request to the given URL. This function returns an `http.Response` and an `error`, which you ignore.
2. `ReadAll` accepts a Reader and returns a byte slice and an `error`. 
3. Handle the error.
4. Closes the network connection. When you make a GET request, Go opens a TCP connection to the web server. This prevents memory leaks that result from open connection, and it lets the client's transport layer reuse the TCP keep-alive connection.
5. Prints the contents of the body to the console.

```go
func main() {
	res, _ := http.Get("https://www.manning.com/")  // 1
	b, err := io.ReadAll(res.Body)                  // 2
	if err != nil {                                 // 3
		panic(err)
	}
	defer res.Body.Close()                          // 4
	fmt.Printf("%s", b)                             // 5
}
```

## Default HTTP client

Go's `http.DefaultClient` is a pointer to an `http.Client` struct with default settings:

```go
var DefaultClient = &Client{}
```

Use `DefaultClient` when you need a quick and convenient HTTP client where the following default settings are suitable:
- Timeout: `0` (hang forever)
- Redirects: Up to 10
- Transport: See [http.DefaultTransport](https://pkg.go.dev/net/http#DefaultTransport)

## Custom client

Go's `http.Client` lets you create a client with custom properties, like redirects and timeouts. Here is the `Client` implementation. Read the [Go documentation](https://pkg.go.dev/net/http#Client) for a complete description of all fields:

```go
type Client struct {
	Transport RoundTripper
	CheckRedirect func(req *Request, via []*Request) error
	Jar CookieJar
	Timeout time.Duration
}
```

Here is a sample implementation:

```go
client := &http.Client{
			Transport: &http.Transport{
				MaxIdleConnsPerHost: o.Concurrency,
			},
			CheckRedirect: func(_ *http.Request, _ []*http.Request) error {
				return http.ErrUseLastResponse
			},
			Timeout: 30 * time.Second,
		}
```
### Transport (Roundtripper)

The `Transport` field uses the `Roundtripper` interface type, which enables the `Client` to delegate HTTP request and response handling to your `Roundtripper` implementation. It uses this interface:

```go
type RoundTripper interface {
    RoundTrip(*http.Request) (*http.Response, error)
}
```

The `DefaultClient` keeps 100 connections open and only allows 2 connections to be reused for the same host. If you are sending more than 2 requests to a host, you might consider creating a custom client with a transport layer.

You can set a custom `RoundTripper` in a custom `Client`. It must perform the following tasks:
- Establish TCP connections
- Send HTTP requests
- Return HTTP responses.


#### Set connections per host

This example customizes the [`Transport`](https://pkg.go.dev/net/http#Transport) type that sets the number of idle clients connections equal to the concurrency used in the client tool:

```go
client := &http.Client{
			Transport: &http.Transport{
				MaxIdleConnsPerHost: o.Concurrency,
			}
		}
```

{{< admonition "Custom Transport" tip >}}
`http.Transport` is the default concrete implementation of the `http.RoundTripper` interface. This differs from `http.DefaultTransport` because the `http.Transport` does not have other default settings. For a better pattern, see [Clone default transport](#clone-default-transport)
{{< /admonition >}}


#### Logging Transport

This example creates a 

```go
type LoggingTransport struct {
    Base http.RoundTripper
}

func (t *LoggingTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    fmt.Println("Sending request to:", req.URL)

    resp, err := t.Base.RoundTrip(req)
    if err != nil {
        return nil, err
    }

    fmt.Println("Received response:", resp.Status)
    return resp, nil
}
```

#### Modify headers

```go
type HeaderTransport struct {
    Base http.RoundTripper
}

func (t *HeaderTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    req.Header.Set("X-Custom-Header", "myvalue")
    return t.Base.RoundTrip(req)
}
```

#### Clone default transport

This example clones the `DefaultTransport` type so you can set some custom values but maintain the sensible defaults: 

```go
main {
	transport := http.DefaultTransport.(*http.Transport).Clone()

	transport.MaxIdleConns = 100
	transport.IdleConnTimeout = 30 * time.Second

	client := &http.Client{
	    Transport: transport,
	}
}
```

### CheckRedirect

This field lets you handle HTTP redirects. You can assign it a function to prevent redirects. The function must have the following signature:

```go
CheckRedirect func(req *Request, via []*Request) error
```
In the preceding function:
- `req` is the next request that the client is about to send
- `via` is a slice that contains all previous requests in the redirect chain, oldest to newest. `via[0]` is the original request.


For example, imagine that you make a request to `a.com`, and then get redirected to `b.com` then `c.com`. When the client follows the last redirect to make a request to `c.com`, then `req` is an HTTP request to `c.com` and `via` is a slice that contains the previous requests, `[a.com, b.com]`.

```bash
Original Request  ──▶ Redirect 1 ──▶ Redirect 2 ──▶ Redirect 3
      via[0]             via[1]         via[2]         req
```


#### Disable redirects

The following setting disables HTTP redirects:

```go
client := &http.Client{
			CheckRedirect: func(_ *http.Request, _ []*http.Request) error {
				return http.ErrUseLastResponse
		}
```

#### Limit redirects

This example limits the client to 3 redirects:

```go
client := &http.Client{
	CheckRedirect: func(req *http.Request, via []*http.Request) error {
		if len(via) >= 3 {
			return fmt.Errorf("too many redirects")
		}
		return nil
	},
}
```

#### Log redirects

This example logs redirects to the console:

```go
client := &http.Client{
	CheckRedirect: func(req *http.Request, via []*http.Request) error {
		fmt.Println("Redirecting to:", req.URL)
		return nil
	},
}
```

#### Cross-domain redirects

This example blocks any redirects to a domain that differs from the domain for the original request:

```go
client := &http.Client{
	CheckRedirect: func(req *http.Request, via []*http.Request) error {
		if len(via) == 0 {
			return nil
		}

		originalHost := via[0].URL.Host
		if req.URL.Host != originalHost {
			return fmt.Errorf("cross-domain redirect blocked")
		}

		return nil
	},
}
```

### Timeout

HTTP allows the server and client to keep established connections alive until there is a timeout. This is called _keep-alive_.

This example creates a client with a custom `Timeout` value:
1. Create a custom client with a 1 second `Timeout`.
2. Send a request with its `Get` method.

```go
func main() {
	client := &http.Client{Timeout: time.Second}            // 1
	res, err := client.Get("https://www.manning.com/")      // 2
	if err != nil {
		panic(err)
	}

	b, err := io.ReadAll(res.Body)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()
	fmt.Printf("%s", b)
}
```



## Sending Requests

### Request object and Do

In its most basic form, making a request with the `DefaultClient` requires that you create two objects: a `Request` object and a client that makes the request:
1. Create a new `Request` object. `NewRequest` takes a method, URL, and request body. Because this is a DELETE request, the body is `nil`.
2. Handle any errors.
3. `DefaultClient` sends the request with its `Do` method. The `Do` method is how the HTTP client sends a request. It accepts a `Request` object, passes it to the client's Transport layer, opens a connection, sends the request, then waits for the response. It returns the `Response`, but it does not download the response body immediately.
4. Handle any errors.
5. Print the response status code to the console.

```go
func main() {
	req, err := http.NewRequest( 				// 1
		"DELETE", 
		"https://jsonplaceholder.typicode.com/posts/1",
		nil,
	)      
	if err != nil {                             // 2
		panic(err)
	}

	res, err := http.DefaultClient.Do(req)      // 3
	if err != nil {                             // 4
		panic(err)
	}
	fmt.Printf("%s\n", res.Status)              // 5
}
```

### Request with context

You can attach a context to a request with one of these methods:
- Request `Clone` method: Attach an existing context to a Request object.
- `NewRequestWithContext`: Create a new request with a new Context.

This example returns a new request with a new Context:
1. Creates a new request with a context.
2. Creates a root context.
3. `http.NoBody` is a variable that explicitly represents a request with no body and sets `ContentLength` to 0. Use this rather than `nil`.

```go
func main() {
	req, err := http.NewRequestWithContext(
		context.Background(),
		http.MethodGet,
		"http://www.example.com",
		http.NoBody,
	)

	if err != nil {
		panic(err)
	}

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}

}
```

### POST

Send a POST request with a custom client. `Post` takes the URL, content type, and request body that is of type `io.Reader`. An easy way to create a Reader is with the `strings.NewReader`:
1. Create a client and set the timeout to one second.
2. Create a Reader from a string to pass as the request body.
3. Make the request.

```go
func main() {
	client := &http.Client{Timeout: time.Second} 									// 1
	body := strings.NewReader(`{"message": "Sending a request"}`) 					// 2
	res, err := client.Post("https://httpbin.org/post", "application/json", body) 	// 3
	if err != nil {
		panic(err)
	}

	b, err := io.ReadAll(res.Body)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()
	fmt.Printf("%s", b)
}
```

### Form data

Send form data to a server with the `PostForm` method:
1. Create a client and set the timeout to one second.
2. `url.Values` is a map used for form encoding. Its keys are strings, and its values are slices of strings. This lets you send form data if a field has multiple values. This expression creates a map literal.
3. `Add` takes a key and a value and stores it in the map. Because each key has a slice of strings as a value, you can add multiple values to the same key.
4. `PostForm` takes a URL and a `url.Values` type as parameters.

```go
func main() {
	client := &http.Client{Timeout: time.Second}
	formValues := url.Values{}
	formValues.Add("message", "Hello form!")
	formValues.Add("message", "Nice to meet you!")

	res, err := client.PostForm("https://httpbin.org/post", formValues)
	if err != nil {
		panic(err)
	}

	b, err := io.ReadAll(res.Body)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()
	fmt.Printf("%s", b)
}
```

### Cookies

To add a cookie to the request, create a Request object and use the `addCookie` method. HTTP is a stateless protocol, and cookies help you with things like authentication and user settings. Cookies are sent as a header in the following format: `Cookie: <key>=<value>`:
1. Create a client and set the timeout to one second.
2. Create a Request object. `NewRequest` takes a method, URL, and request body. We're not sending a body, so set that to `nil`.
3. Add a cookie to the `Header` field in the Request object with `AddCookie`.
4. Make the request with `Do`.


```go
func main() {
	client := &http.Client{Timeout: time.Second} 							// 1
	req, err := http.NewRequest("GET", "https://httpbin.org/cookies", nil) 	// 2
	if err != nil {
		panic(err)
	}
	req.AddCookie(&http.Cookie{ 											// 3
		Name:  "cookie",
		Value: "oreo",
	})

	resp, err := client.Do(req) 											// 4
	if err != nil {
		panic(err)
	}

	b, err := io.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	fmt.Printf("%s", b)
}
```

## Reading responses

{{< admonition "I/O" tip >}}
For more information about Readers and Writers, see [Input/Output](../../fundamentals/input-output).
{{< /admonition >}}

A response `Body` is a Reader, so we can stream it as chunks of bytes. This means that you don't have to use `io.ReadAll` to store the entire response body in memory. `io.ReadAll` allocates a 512-byte array, then appends memory to that array as needed. This leads to inefficient memory and CPU use.

### io.Copy

`io.Copy` lets you transfer bytes in a memory-efficient way. It transfers 32 KB chunks of bytes from a Reader to a Writer, then returns the number of bytes written and an error. Internally, `io.Copy` loops and reads from Reader and Writes to Writer. This continues until it reaches an EOF or an error occurs:

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

To demonstrate, this `Send` function sends an HTTP request and returns a custom `Result` type. It does not read the content of the response body, it only counts the bytes:
1. Send the request.
2. If the request fails, continue and assign the error to `Result.Error`. If it succeeds:
   1. Close the response body to free the keep-alive connection. You want to close the body within these brackets because `Body.Close` discards the error if it is not `nil`.
   2. Get the response status code. 
   3. Get the number of bytes in the response. Because we don't care about the content in the response body, we write the bytes to `io.Discard`, which is the Go equivalent of `/dev/null`.

```go
func Send(client *http.Client, req *http.Request) Result {
	started := time.Now()
	var (
		bytes int64
		code  int
	)
	resp, err := client.Do(req) 						// 1
	if err == nil { 									// 2
		defer resp.Body.Close() 						// 2.1
		code = resp.StatusCode 							// 2.2
		bytes, err = io.Copy(io.Discard, resp.Body) 	// 2.3
	}

	return Result{
		Duration: time.Since(started),
		Bytes:    bytes,
		Status:   code,
		Error:    err,
	}
}
```



## Handling timeouts

Timeout errors occur when the client waits too long for a response from a server and terminates the operation or connection. This might happen whether or not you explicitly set a timeout. You can detect a timeout error and retry the operation. The server might respond, or you might be routed to another running instance.

### Detecting error types

Each error type in the `net` package has a `Timeout()` method that returns `true` when there is a timeout. When an error is returned from the `net` package, you can check it against known cases that show a timeout error. This table describes some common error types and their triggers:

| Error Type     | Source                                          | Example Trigger                                    |
| -------------- | ----------------------------------------------- | -------------------------------------------------- |
| `*url.Error`   | `http.Client` methods (`http.Get`, `Do`, etc.)  | Invalid domain or bad URL                          |
| `*net.OpError` | Low-level networking (`net.Dial`, `net.Listen`) | Connection refused, DNS failure, read/write errors |
| `net.Error`    | Interface implemented by many network errors    | Timeout or temporary error                         |

Here is an example of how to check for these error types with a `switch` statement:

```go
func hasTimedOut(err error) bool {
	switch err := err.(type) {
	case *url.Error:
		if err, ok := err.Err.(net.Error); ok && err.Timeout() {
			return true
		}
	case net.Error:
		if err.Timeout() {
			return true
		}
	case *net.OpError:
		if err.Timeout() {
			return true
		}
	}

	errTxt := "use of closed network connection"
	if err != nil && strings.Contains(err.Error(), errTxt) {
		return true
	}
	return false
}
```

The following example is how you can use `hasTimedOut`:

```go
func main() {
	client := &http.Client{Timeout: time.Second}
	res, err := client.Get("https://www.manning.com/")
	if hasTimedOut(err) {
		panic("request has timed out")
	}
	if err != nil {
		panic("not a timeout error")
	}

	// read res.Body
}
```

### Resuming after timeout

In some circumstances, a timeout occurs when you download a large resource, and you do not want to restart the download from the beginning.

If a server that range requests, it sends the `Accept-Ranges: bytes` server response header. It either supports `bytes` or `none`.

The `download` function accepts the following arguments:
- `location`: URL for the resource` (a URL), 
- `file`: pointer to an open file where the data is written
- `retries`: number of times to retry on timeout errors

This funciton uses the `hasTimedOut` function described in [Detecting error types](#detecting-error-types):
1. Create a new GET request with the `location` argument.
2. Get details about the opened file you are writing to. `Stat` returns a [`FileInfo`](https://pkg.go.dev/io/fs#FileInfo).
3. If the opened file has any `Size()`, then you are resuming an interrupted download. `start` is a size in bytes, so you get the `string` representation of that value with `FormatInt` so you can pass it to the `Range` header.
   
   The `Range` header accepts the range of bytes that you want to retrieve in the format `Range: bytes=<start>-<end>`. Because we want to resume the download, we only specify the `start` value when we set the `Range` header in our request. For more information about `Range`, see the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Range#syntax).
4. Create a custom client with a 5 minute timeout.
5. Send the request.
6. If the request returns a timeout error and there are retries remaining, call `download` to resume the download. Decrement the `retries` argument when you call `download`.
7. If there was an error that was not a timeout error, return the error.
8. Verify that the server responded with a 2xx success code.
9. Check if the server supports range requests. If not, set `retries` to `0` because you cannot resume a download.
10. Write the response into the local file with `Copy`.
11. If there is an error during a `Copy` operation, check if it is a timeout error. If it is a timeout error and retries remain, resume the download. Otherwise, return the error.

```go
func download(location string, file *os.File, retries int64) error {
	req, err := http.NewRequest("GET", location, nil)           // 1
	if err != nil {
		return err
	}

	fi, err := file.Stat()                                      // 2
	if err != nil {
		return err
	}

	current := fi.Size()                                        // 3
	if current > 0 {
		start := strconv.FormatInt(current, 10)
		req.Header.Set("Range", "bytes="+start+"-")
	}

	cc := &http.Client{Timeout: 5 * time.Minute}                // 4
	res, err := cc.Do(req)                                      // 5
	
    if err != nil && hasTimedOut(err) {                         // 6
		if retries > 0 {
			return download(location, file, retries-1)
		}
		return err
	} else if err != nil {                                      // 7
		return err
	}

	if res.StatusCode < 200 || res.StatusCode > 300 {           // 8
		errFmt := "Unsuccessful HTTP request. Status: %s" 
		return fmt.Errorf(errFmt, res.Status)
	}

	if res.Header.Get("Accept-Ranges") != "bytes" {             // 9
		retries = 0
	}

	_, err = io.Copy(file, res.Body)                            // 10
	if err != nil && hasTimedOut(err) {                         // 11
		if retries > 0 {
			return download(location, file, retries-1)
		}
		return err
	} else if err != nil {
		return err
	}
	return nil
}
```

{{< admonition "Improvement" tip >}}
You can improve this by adding a function that checks the file hash to confirm the integrity of the downloaded file.
{{< /admonition >}}

Here is the code that calls `download` and writes to a local file:
1. Create the file.
2. Invoke `download` with 100 retries.
3. Get file metadata so you can log the number of bytes downloaded.
4. Log the number of bytes downloaded.

```go
func main() {
	file, err := os.Create("file.zip")                          // 1
	if err != nil {
		fmt.Println(err)
		return
	}
	defer file.Close()
	location := "https://example.com/file.zip"
	err = download(location, file, 100)                         // 2
	if err != nil {
		fmt.Println(err)
		return
	}

	fi, err := file.Stat()                                      // 3
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("got it with %v bytes downloaded", fi.Size())    // 4
}
```


## JSON

The most common data format for REST APIs is JSON. Go's [encoding/json package](https://pkg.go.dev/encoding/json) provides all the tools required to parse JSON data into Go data structs.

When JSON data is parsed, it is _unmarshaled_. When you unmarshal a JSON object, you convert the JSON-encoded bytes into an in-memory representation, commonly a struct.

The following example parses JSON data into a struct. For simplicity, the JSON object is stored in a string, but it is more likely to be read from an HTTP response body:
1. Create the struct that models the data. Use struct tags to map a struct field to a field in the JSON object.
2. In-memory JSON object.
3. Create a `Person` object to hold the parsed JSON.
4. `json.Unmarshal` takes a slice of bytes and a pointer to a data structure to store the data. This method mutates the data, so remember to pass a memory address rather than a value.
5. Handle the error.
6. Do something with the parsed JSON.

```go
type Person struct {                            // 1
	Name string `json:"name"`
}
                                                // 2
var JSON = `{                                   
	"name": "Jimmy John"
}`

func main() {
	var p Personv                               // 3
	err := json.Unmarshal([]byte(JSON), &p)     // 4
	if err != nil {                             // 5
		fmt.Println(err)
		return
	}
	fmt.Println(p)                              // 6
}
```
### Unstructured JSON

In some circumstances, you might not know the structure of the JSON data before you consume it. To parse arbitrary JSON, unmarshal the data into an `interface{}`.

For example, here is an in-memory JSON object with an unknown schema:

```go
var ks = []byte(`{ 
"firstName": "Jean", 
"lastName": "Bartik", 
"age": 86, 
"education": [ 
     { 
            "institution": "Northwest Missouri State Teachers College", 
            "degree": "Bachelor of Science in Mathematics" 
     }, 
     {  
            "institution": "University of Pennsylvania", 
            "degree": "Masters in English" 
     } 
], 
"spouse": "William Bartik", 
"children": [ 
     "Timothy John Bartik", 
     "Jane Helen Bartik", 
     "Mary Ruth Bartik" 
]  
}`)
```

To parse the data, create an `interface{}` type and then unmarshal the JSON into a pointer to that interface:

```go
func main() {
	var f interface{}
	err := json.Unmarshal(ks, &f)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	fmt.Println(f)
}
```

After the data is marshaled into the `interface{}`, you need to inspect it. This table describes the types that Go unmarshals data into:

| JSON Type   | Go Type                  |
| ----------- | ------------------------ |
| **string**  | `string`                 |
| **number**  | `float64`                |
| **boolean** | `bool`                   |
| **null**    | `nil`                    |
| **array**   | `[]interface{}`          |
| **object**  | `map[string]interface{}` |

This method shows how you can walk through an unstructured JSON object to learn each field's type and value:

```go
func printJSON(v interface{}) {
	switch vv := v.(type) {
	case string:
		fmt.Println("is string,", vv)
	case float64:
		fmt.Println("is float64,", vv)
	case []interface{}:
		fmt.Println("is an array:")
		for i, u := range vv {
			fmt.Print(i, " ")
			printJSON(u)
		}
	case map[string]interface{}:
		fmt.Println("is an object:")
		for i, u := range vv {
			fmt.Print(i, " ")
			printJSON(u)
		}
	default:
		fmt.Println("Unknown type")
	}
}
```

## Reusing connections

### Keep-alive

HTTP keep-alive is a feature of the HTTP protocol that lets a single TCP connection be reused for multiple requests and responses. Go's `DefaultClient` uses the `http.DefaultTransport`, which enables HTTP keep-alive for 30 seconds. To maintain this default, do not change the `KeepAlive` setting in a custom client.


### Close response bodies

Another method to reuse connections is closing response bodies after you read them rather than deferring their closing until the caller returns.

For example, this snippet makes multiple GET requests and closes the body after each call.

1. Make a request.
2. Read the body.
3. Close the request body.
4. Do work with the body.

```go
func main() {
	res, err := http.Get("http://example.com")      // 1
	if err != nil {
		os.Exit(1)
	}

	body, err := io.ReadAll(res.Body)               // 2
	if err != nil {                 
		os.Exit(1)
	}
	res.Body.Close()                                // 3

	fmt.Println(body)                               // 4

	res2, err := http.Get("http://example.com")     // 1
	if err != nil {
		os.Exit(1)
	}

	body2, err := io.ReadAll(res2.Body)             // 2
	if err != nil {
		os.Exit(1)
	}
	res2.Body.Close()                               // 3

	fmt.Println(body2)                              // 4
}
```

## Testing

### Roundtripper

Implement a fake `RoundTripper` and pass it to the `Client`. You can satisfy the `RoundTripper` interface with a function type, and inject the function during testing so you don't have to make network calls in tests.

This functional test verifies whether our `Send` method returns the correct error code.

First, create the function type that satisfies `RoundTripper`:
1. The function definition. Any function that takes an `*http.Request` and returns `*http.Response` and `error` can be this type.
2. `RoundTrip` is the only method in the `RoundTripper` interface, so this means that the `roundTripperFunc` function type can be used as a `RoundTripper` type in a client's `Transport` field.
   
   Here, `RoundTrip` only calls the underlying function.

```go
type roundTripperFunc func(*http.Request) (*http.Response, error)

func (f roundTripperFunc) RoundTrip(r *http.Request) (*http.Response, error) {
	return f(r)
}
```

Next, test that `Send` responds with the correct error code:
1. Create the request.
2. Create the fake `Transport`. This function ignores the request, then returns a `Response` with the status 500.
3. Inject the fake. When `client` makes a call, it performs the following:
   ```bash
   client.Do ──▶ client.Transport.RoundTrip(req) ──▶ roundTripperFunc.RoundTrip ──▶ fake(req)
   ```
   When you inject the `fake` transport, you ensure that there are no live network calls. Think of a `RoundTripper` as the instructions needed to send a request over the network and a `Transport` as the engine that sends the request. By injecting "fake" instructions, the engine (`http.Transport`) doesn't call anything over the network.

```go
func TestSendStatusCode(t *testing.T) {
	t.Parallel()

	req, err := http.NewRequest(http.MethodGet, "/", http.NoBody)
	if err != nil {
		t.Fatalf("creating http request: %v", err)
	}

	fake := func(_ *http.Request) (*http.Response, error) {
		return &http.Response{
			StatusCode: http.StatusInternalServerError,
		}, nil
	}
	client := &http.Client{
		Transport: roundTripperFunc(fake),
	}
	result := Send(client, req)

	if result.Status != http.StatusInternalServerError {
		t.Errorf("got %d, want %d", result.Status, http.StatusInternalServerError)
	}
}
```