---
title: "Errors"
weight: 30
description: >
  How to handle, create, and use errors.
---

[Error handling in Go](https://go.dev/blog/error-handling-and-go)

Errors are values in Go. The following example is the most common format for error handling:
```go
value, err := funcThatReturnsValandErr() {
    if err != nil {
        // handle err
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}
```

The preceding example sends the error to STDERR and exits the program.


## xCompact error checking

If a function or method returns only an error, you can assign any error and check it for `nil` on the same line:
```go
if err := returnErr(); err != nil {
    // handle error
}
```
## Basic error handling

`fmt.Errorf` creates a custom formatted error:

```go
return fmt.Errorf("Error: %s is not a valid string", s)
```

## Compact error checking

If a function or method returns only an error, you can assign any error and check it for `nil` on the same line:
```go
if err := returnErr(); err != nil {
    // handle error
}
```

## Check for specific errors with .Is()

Check if an error is a specific object type with the [`.Is()` function](https://pkg.go.dev/errors#Is). This function accepts the error value and an error type for comparison. This function is helpful during testings.

For example, the following snippet returns `nil` if the the error is `os.ErrNotExist` (the file does not exist); otherwise, it returns the error:
```go
file, err := os.ReadFile(filename)
if err != nil {
    if errors.Is(err, os.ErrNotExist) {
        return nil
    }
    return err
}
```

## Check for types with .As()

Check if an error is of a specific type. I _think_ you use `.Is()` for native Go errors, and `.As()` for custom error types. For example:

```go
func (app *application) readJSON(w http.ResponseWriter, r *http.Request, dst any) error {

	maxBytes := 1_048_576
	...
	if err != nil {
		
		var maxBytesError *http.MaxBytesError
        ...
        // native go error
		case errors.Is(err, io.EOF):
			return errors.New("body must not be empty")
        ...
        // custom error type
		case errors.As(err, &maxBytesError):
			return fmt.Errorf("body must not be larger than %d bytes", maxBytesError.Limit)
        ...
	}
	return nil
}
```

## Wrap errors

For additional details, see the [Go docs](https://go.dev/blog/go1.13-errors#wrapping-errors-with-w).

For errors, use `%w` to decorate the original error with additional information for the users. Essentially, you can customize the error message while also returning the default Go error:
```go
if err != nil {
    return nil, fmt.Errorf("Cannot read data from file: %w", err)
}
```
For more info, read [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-add-extra-information-to-errors-in-go).

## Custom error types

Create custom errors in the `errors.go` file. You can use these errors instead of error strings. Essentially, you are wrapping errors with additional messages to provide more information for the user while keeping the original error available for inspection (usually during tests) with `errors.Is(err, expectedErr)`.

Custom errors use the format `Err*`:
```go
var (
    ErrNotNumber        = errors.New("Data is not numeric")
    ErrInvalidColumn    = errors.New("Invalid column number")
    ErrNoFiles          = errors.New("No input files")
    ErrInvalidOperation = errors.New("Invalid operation")
)
```

## Returning errors from functions

Return only an error if you want to check that a method performs an operation correctly:

```go
func Add(a *int, b int) error {
    a += b
    return nil
}
```
When you are returning an error, use STDERR instead of STDOUT to display error messages, and exit with `1`:
```go
if err := l.Get(todoFileName); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
}
```

## Centralize errors

Applications might benefit from centralized errors. Centralized errors are not the same as [custom error types](#custom-error-types)--they use native Go error handling to wrap responses.

The following example creates three types of errors for an HTTP server:


```go
// The serverError helper writes an error message and stack trace to the errorLog,
// then sends a generic 500 Internal Server Error response to the user.
func (app *application) serverError(w http.ResponseWriter, err error) {
	trace := fmt.Sprintf("%s\n%s", err.Error(), debug.Stack())
	app.errorLog.Output(2, trace)

	http.Error(w, http.StatusText(http.StatusInternalServerError), http.StatusInternalServerError)
}

// The clientError helper sends a specific status code and corresponding description
// to the user. We'll use this later in the book to send responses like 400 "Bad
// Request" when there's a problem with the request that the user sent.
func (app *application) clientError(w http.ResponseWriter, status int) {
	http.Error(w, http.StatusText(status), status)
}

// For consistency, we'll also implement a notFound helper. This is simply a
// convenience wrapper around clientError which sends a 404 Not Found response to
// the user.
func (app *application) notFound(w http.ResponseWriter) {
	app.clientError(w, http.StatusNotFound)
}
```
In the preceding example:
- `serverError`: Writes an error's stack trace to the writer. 
- `clientError`: Sends an error to the client.
- `notFound`: Wraps `clientError` to simplify sending a 404 message to the client.