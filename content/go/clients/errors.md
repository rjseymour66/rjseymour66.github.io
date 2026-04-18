+++
title = 'Errors'
date = '2025-11-16T08:38:14-05:00'
weight = 20
draft = false
+++

## Basic error handling

A client can read error messages and codes from the response. This snippet prints the value of each:
1. `Status` provides a text message for the response status. For example, `200 OK` or `404 Not Found`.
2. `StatusCode` provides the status code as an integer.

```go
func main() {
    res, _ := http.Get("http://example.com")
    fmt.Println(res.Status)                     // 1
    fmt.Println(res.StatusCode)                 // 2
}
```

## Common errors

### 2xx – Success

| Status Code | Name       | Description                                                          |
| ----------- | ---------- | -------------------------------------------------------------------- |
| 200         | OK         | The request succeeded and the server returned the requested data.    |
| 201         | Created    | The request succeeded, and a new resource was created.               |
| 202         | Accepted   | The request has been accepted for processing, but not completed yet. |
| 204         | No Content | The request succeeded, but there is no content to send back.         |

### 3xx – Redirection

| Status Code | Name               | Description                                                                 |
| ----------- | ------------------ | --------------------------------------------------------------------------- |
| 301         | Moved Permanently  | The resource has been moved to a new permanent URL.                         |
| 302         | Found              | The resource is temporarily located at a different URL.                     |
| 303         | See Other          | The client should retrieve the resource using a GET request to another URI. |
| 304         | Not Modified       | The resource has not changed since the last request (used with caching).    |
| 307         | Temporary Redirect | The resource is temporarily located at a new URL, method not changed.       |
| 308         | Permanent Redirect | The resource has permanently moved to a new URL, method not changed.        |

### 4xx – Client Error

| Status Code | Name                   | Description                                                                  |
| ----------- | ---------------------- | ---------------------------------------------------------------------------- |
| 400         | Bad Request            | The server could not understand the request due to invalid syntax.           |
| 401         | Unauthorized           | Authentication is required or has failed.                                    |
| 403         | Forbidden              | The client is authenticated but does not have permission to access resource. |
| 404         | Not Found              | The requested resource could not be found.                                   |
| 405         | Method Not Allowed     | The HTTP method is not supported for this resource.                          |
| 408         | Request Timeout        | The server timed out waiting for the request.                                |
| 409         | Conflict               | The request conflicts with the current state of the resource.                |
| 410         | Gone                   | The resource requested is no longer available and will not return.           |
| 413         | Payload Too Large      | The request body is larger than the server is willing to process.            |
| 415         | Unsupported Media Type | The request format is not supported by the server.                           |
| 429         | Too Many Requests      | The client has sent too many requests in a given time.                       |


### 5xx – Server Error

| Status Code | Name                  | Description                                                                |
| ----------- | --------------------- | -------------------------------------------------------------------------- |
| 500         | Internal Server Error | A generic server error. Something went wrong on the server.                |
| 501         | Not Implemented       | The server does not support the functionality required to fulfill request. |
| 502         | Bad Gateway           | The server, acting as a gateway, received an invalid response.             |
| 503         | Service Unavailable   | The server is temporarily unable to handle the request (overloaded/down).  |
| 504         | Gateway Timeout       | The server, acting as a gateway, timed out waiting for an upstream server. |



## Check successful request

Here is a quick `if` clause to check whether a request returned a successful 2xx success code. If the status code is below 200 or greater than 300, it returns a formatted error with the HTTP status:

```go
if res.StatusCode < 200 || res.StatusCode > 300 {
    errFmt := "Unsuccessful HTTP request. Status: %s"
    return fmt.Errorf(errFmt, res.Status)
}
```
## Check error class

This snippet checks the error class with a `switch` statement:

```go
res, err := http.Get("https://example.com")
switch res.StatusCode {
case 300 <= res.StatusCode && res.StatusCode < 400:
    fmt.Println("Redirect Message")
case 400 <= res.StatusCode && res.StatusCode < 500:
    fmt.Println("Client error")
case 500 <= res.StatusCode && res.StatusCode < 600:
    fmt.Println("Server error")
}
```

## Creating custom errors

Custom errors give you more control over how you communicate your error codes. Your frontend application needs to consume the errors and present them within your application, and an API server needs to make errors that are consumable to HTTP clients. Standard HTTP plaintext errors are insufficient in both scenarios.

The `Error` function in the `http` package is plaintext only and sets `X-Content-Type-Options: nosniff` as a header, which means clients cannot try to guess the content type of the response.

Create custom errors as structs. Because HTTP is often parsed as JSON, include struct tags to control what is returned:
1. HTTP status code. For example, `404` or `500`. `json:"-"` means that this value is not included in the JSON response---it is used only in server logic. For example, the JSON consumed by the application does not need this value, but the API client can read it in the response on the protocol level.
2. This returns application-specific code, such as `1001` for invalid input. `omitempty` means that if this value is `0`, it is not marshaled into JSON.
3. Human readable string that is always returned in the JSON message.
   
```go
type Error struct {
    HTTPCode int    `json:"-"`                  // 1
    Code     int    `json:"code,omitempty"`     // 2
    Message  string `json:"message"`            // 3
}
```

This outputs an error in the following format:
```json
{
    "error": {
        "code": 123,
        "message": "An Error Occurred"
    }
}
```

1. Create an anonymous struct that contains your custom error type. The struct tag wraps the custom error in another struct under the key `error`. This essentially renames the object from `Err` to `error` in the JSON output. For example, if you do not wrap the error in `error`, it outputs like this:
   
   ```json
   {
      "Err": {
         "code": 123,
         "message": "An Error Occurred"
      }
   }
   ```
   Wrapping the error makes the output clear, and also lets you extend the response. For example, you might want to return additional information at the same level as `error`, such as a `data` object or metadata like a `trace-id`.
2. Marshal the struct into memory in JSON format.
3. If the marshalling fails, return an HTTP 500 error.
4. Set the response header to notify the client the response includes data in JSON format.
5. Write the status code with the custom error.
6. Write the marshaled JSON to the `Response`.

```go
func JSONError(w http.ResponseWriter, e Error) {
    data := struct {                                        // 1
        Err Error `json:"error"`
    }{e}
    b, err := json.Marshal(data)                            // 2
    if err != nil {                                         // 3
        http.Error(w, "Internal Server Error", 500) 
        return
    }
    w.Header().Set("Content-Type", "application/json")      // 4
    w.WriteHeader(e.HTTPCode)                               // 5
    fmt.Fprint(w, string(b))                                // 6
}
```

You can use `JSONError` in a handler. This handler creates an `Error` type and then writes it as JSON to the response:

```go
func displayError(w http.ResponseWriter, r *http.Request) {
    e := Error{
        HTTPCode: http.StatusForbidden,
        Code:     123,
        Message:  "An Error Occurred",
    }
    JSONError(w, e)
}
```

### Using custom errors

This example uses the [custom error type](#creating-custom-errors) in an HTTP request. This error type implments the `Error` interface, which means you can return it where Go expects an `error` type:

```go
type Error struct {
    HTTPCode int    `json:"-"`
    Code     int    `json:"code,omitempty"`
    Message  string `json:"message"`
}

func (e Error) Error() string {
    fs := "HTTP: %d, Code: %d, Message: %s"
    return fmt.Sprintf(fs, e.HTTPCode, e.Code, e.Message)
}
```

The `get` method is a wrapper around the `http.Get` method with smarter error handling:
1. Use the native `Get` method and return any errors.
2. Check if this is a successful response:
   - If it was successful, skip the `if` clause and return the response and a `nil` error.
   - If it was not successful, perform additional error checking.
3. Check if the correct content type---JSON---was returned.
4. Read the response body into a buffer.
5. Create an anonymous `data` struct with the custom `Err` error type.
6. Unmarshal the buffer into the `data` struct. While parsing the JSON, if there is a top-level field named `error`, store its contents in `data.Err`.
7. Check whether there was an error parsing the JSON.
8. Set the `data.Err.HTTPCode` to the response status code.
9.  Return the response and the populated custom `Error` struct.

```go
func get(u string) (*http.Response, error) {
    res, err := http.Get(u)                                         // 1
    if err != nil {
        return res, err
    }

    if res.StatusCode < 200 || res.StatusCode >= 300 {              // 2
        if res.Header.Get("Content-Type") != "application/json" {   // 3
            sm := "Unknown error. HTTP status: %s"
            return res, fmt.Errorf(sm, res.Status)
        }

        b, _ := io.ReadAll(res.Body)                                // 4
        res.Body.Close()
        
        var data struct {                                           // 5
            Err Error `json:"error"`
        }
        err = json.Unmarshal(b, &data)                              // 6
        if err != nil {                                             // 7
            sm := "Unable to parse JSON: %s. HTTP status: %s"
            return res, fmt.Errorf(sm, err, res.Status)
        }

        data.Err.HTTPCode = res.StatusCode                          // 8
        return res, data.Err                                        // 9
    }
    return res, nil
}
```

Here is how you can call the method in the application. Notice how it behaves like the native `Get` method, but it return the custom `Error`:

```go
func main() {
    res, err := get("http://example.com")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    b, _ := io.ReadAll(res.Body)
    res.Body.Close()
    fmt.Printf("%s", b)
}
```