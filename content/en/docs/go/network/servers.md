---
title: "Servers"
weight: 70
description: >
  Working with Servers in Go.
---


The `net/http` package provides the `ListenAndServer()` function that creates a default server. However, this function does not allow you to define timeouts to manage bad connections or server resources, so you should define custom server.

## Server type

Create a custom server with the [`Server` type](https://pkg.go.dev/net/http#Server). The `Server` type is a struct with the following fields:

```go
type Server struct {
	Addr              string
	Handler           Handler
	TLSConfig         *tls.Config
	ReadTimeout       time.Duration
	ReadHeaderTimeout time.Duration
	WriteTimeout      time.Duration
	IdleTimeout       time.Duration
	MaxHeaderBytes    int
	TLSNextProto      map[string]func(*Server, *tls.Conn, Handler)
	ConnState         func(net.Conn, ConnState)
	ErrorLog          *log.Logger
	BaseContext       func(net.Listener) context.Context
	ConnContext       func(ctx context.Context, c net.Conn) context.Context
}
```

## HTTP handlers

Servers receive requests, dispatch those requests to the handler that is registered to the receiving path, and the handler sends a response. In Go, each incoming request is served in its own goroutine, which means each request is handled concurrently.

You create the server, then use `HandleFunc` to register routes to handler functions. Then you use `HandlerFunc` to define the handler function for the route.

- `http.Handler` is an interface that has the `ServeHTTP(w ResponseWriter, r *Request)` signature.
- `http.HandlerFunc` is an adapter type that lets you define ordinary functions as HTTP handlers. It adds a `ServerHTTP(w, r)` method to whatever function you pass it.
- `http.HandleFunc` registers a handler with a multiplexer server. It is syntactic sugar--it transforms a function to a handler and registers it in one step. It accepts two arguments: the path as a `string`, and the handler function. It implements `ServeHTTP()`.
- `http.NewServeMux()` returns a custom server. It implements `ServeHTTP()`.

The following example registers the `homeHandler` twice, but each method is functionally equivalent:

```go
func homeHandler(w http.ResponseWriter, r *http.Request) {
	// handler logic 
}

func main() {
	...
	mux.Handle("/", http.HandlerFunc(homeHandler))
	mux.HandleFunc("/", homeHandler)
}
```

### Handlers as methods

Create handlers and helpers as methods on the application struct. This allows the handlers to access any application dependencies, like loggers:

```go
// main.go
type application struct {
	log *log.Logger,
	...
}

// handlerName.go
func (app *application) handlerName(w, r) {...}
```


## Requests

### URL query strings

Sometimes requests pass information as key-value pairs in the URL:

```
http://localhost:4000/snippet/view?id=123
```
The values after the `?` are _query strings_, and they take the form of `key`=`value`. You can access the `key` portion of the query string and retrieve the `value`:

```go
id, err := r.URL.Query().Get("id")
```

The proceeding example uses the `Query()` method on the request's `URL` field. `Query()` returns a `Values` type, which is essenitally a map that holds all the query parameter key-value pairs. The `Values` type has the `Get()` method that returns a `string` value associated with the `key` provided to `Get()`.


## Responses

Responses are made with the `http.ResponseWriter` interface, which is usually represented with a `w`.

It is common to use `fmt.Fprintf()` to write a response: 

```go
fmt.Fprintf(w, "Display a specific snippet with ID %d...", id)
```

_http.Error(writer, error message, status code)_
: Writes an error message on the writer. You can use helper methods like `http.StatusText`. For example:

```go
http.Error(w, http.StatusText(http.StatusInternalServerError), http.StatusInternalServerError)
``` 


_http.NotFound(w, r)_
: Replies with a `404 Not Found` status code.

### Headers

Go provides the following headers automatically:
- `Date`
- `Content-Length`
- `Content-Type` (see w.Header().Set())

_w.Header().Set(`header-name`, `header-value`)_
: Adds a custom header to the HTTP response in the `header-name`:`header-value` format. A common example is setting the `Content-Type` header when you send JSON data:

```go
w.Header().Set("Content-Type", "application/json")
```
You must write the `w.Header().Set()`method before `w.WriteHeader()` or `w.Write()`.

_w.WriteHeader(`status-code`)_
: Writes a status code to the response. You should call this method only once.

  If you do not call this method before `w.Write([]bytes())`, then the `Write` method returns a `200 OK` status code automatically.

  `w.WriteHeader` is not commonly used. You usually pass the writer (`w`) value to an `http` package helper function.


### Constants

The `http` package provides [constants](https://pkg.go.dev/net/http#pkg-constants) for request methods and status code. These constants improve readability and reduce runtime errors that result from typos. For example:

```go
func snippetCreate(w http.ResponseWriter, r *http.Request) {

	if r.Method != http.MethodPost {
		w.Header().Set("Allow", http.MethodPost)
		http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
		return
	}
	w.Write([]byte("Create a new snippet..."))
}
```

## Routers

You can create a router with Go's standard library or a 3rd party package.

### Go router

> The Go standard library does not support naed parameters in URLs, so you might consider a 3rd-party solution for servers with complex routes.

By default, Go supports two types of URL patterns:

_fixed path `/path/endpoint`_
: A fixed path does not have a trailing slash (`/`). A request must match a fixed path exactly.

_subtree path `/path/bucket/`_
: A subtree path has a trailing slash. A request must match only the start of the subtree path. Think of subtree paths with a wildcard appended. For example, `/path/bucket/*`.

Create a Go router with these components:
- `http.Handler` type to satisfy the server's `Handler` field.
- `Server` struct that wraps the `http.Handler` type.
- Constructor that returns an instance of the struct that wraps the new type.
- A method on the `Server` that registers routes.
- One or more constants that define the routes.

The following example creates all required components:

```go
const (
	shorteningRoute  = "/s"
	resolveRoute     = "/r/"
	healthCheckRoute = "/health"
)

// mux is an unexported http.Handler
type mux http.Handler

// Server is a custom server type.
type Server struct {
	mux // the server only exports ServeHTTP
}

// NewServer returns an instance of the custom Server type.
func NewServer() *Server {
	var s Server
	s.registerRoutes()
	return &s
}

func (s *Server) registerRoutes() {
	mux := http.NewServeMux()
	mux.Handle(shorteningRoute, httpio.Handler(s.shorteningHandler))
	mux.Handle(resolveRoute, httpio.Handler(s.resolveHandler))
	mux.HandleFunc(healthCheckRoute, s.healthCheckHandler)
	s.mux = mux
}
```

### 3rd-party router

This example uses the [julienschmidt](https://github.com/julienschmidt/httprouter) router. This router has a few advantages over the Go standard library router:
- Supports named parameters.
- Validates routes. For example, this router matches the home path (`"/"`) path exactly, so no need for manual checks.
- Easily create `404` error handlers.


To create a router, use the `New()` method. You can register handlers with the `Handler()` or `HandlerFunc()` method:

```go
router := httprouter.New()
router.HandlerFunc(http.MethodGet, "/snippet/view/:id", app.snippetView)
```
`:id` is a named parameter--it acts as a wildcard for the path segment that it represents.

> Always use the [HTTP method constants](https://pkg.go.dev/net/http#pkg-constants) when you register a route.

You can also use a parameter that matches everything, which is useful when registering static files:

```go
fileServer := http.FileServer(http.Dir("./ui/static/"))
router.Handler(http.MethodGet, "/static/*filepath", http.StripPrefix("/static", fileServer))
```
#### 404 handler

The `router.NotFound` handler is called when no matching route is found. You can assign it an `http.HandlerFunc(func{w, r})` to handle `404` responses:

```go
router.NotFound = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
	app.notFound(w)
})

```

#### Middleware and router return

Use `alice.New()` to chain middleware in a reader-friendly format:

```go
standard := alice.New(app.recoverPanic, app.logRequest, secureHeaders)
```

Next, return the router:

```go
return standard.Then(router)
```
### Placeholder routers

When you create a router, it is a good practice to register placeholder handlers that return dummy plaintext responses. This helps you set up the routing infrastructure while managing the business logic at a later time.

In your handlers file, add the placeholder routes:

```go
func (app *application) userSignup(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Display an HTML form for signing up a new user")
}

func (app *application) userSignupPost(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Create a new user...")
}
...
```

In your routes file (where you register your routes), register the routes:

```go
	router := httprouter.New()
	...

	router.Handler(http.MethodGet, "/user/signup", dynamic.ThenFunc(app.userSignup))
	router.Handler(http.MethodPost, "/user/signup", dynamic.Then(app.userSignupPost()))
```

## Writing a basic server

There are two important concepts that are central to creating a custom server in Go:
- `Server` type: listens for client connections and routes incoming requests in a goroutine to a type that implements the `Handler` interface.
- `Handler` interface: contains one method, `ServeHTTP(ResponseWriter, *Request)`.

You can create a server with any type as long as it implements `Handler` interface. A server should have the following:
- One or more registered routes with an associated function to handle requests to that route.
- A server multiplexer that can manage multiple registered routes.

### Directory structure

A suggested server directory structure:

```shell
.
├── cmd
│   └── server
│       └── server.go
├── internal
│   └── helpers
│       ├── handler.go
│       └── helpers.go
├── errors
│   └── errors.go
└── service-name
    ├── server.go
    ├── server_test.go
    └── service.go

```

In the preceding example:
- `cmd/serverdir/server.go` contains the `main` method. This is where you create a logger and execute the server's `ListenAndServe()` method.
- `internal/helpers/*` contains private code such as JSON formatters and handlers for the handler chaining pattern.
- `errors/errors.go` contains custom errors.
- `server-name/server.go` contains the service's server implementation, including the constructor, routes, etc.
- `service-name/server_test.go` tests `server.go` with the `httptest.NewRecorder()` method.
- `server-name/service.go` contains the service's business logic.





### Define the custom Server type

In the project root, create a subdirectory and file called `json-server/server.go`:

```bash
mkdir json-server
touch json-server/server.go
```

The `Server` type must implement the `Handler` interface, so embed the `Handler` interface in the Server type:

```go
type mux http.Handler

type Server struct {
	mux
}
```
Interface embedding elevates the interface methods to the containing type, so the `Server` implements the `Handler`'s `ServeHTTP` method.

### Create the main method

1. In the project root, create the main `server.go` file:

   ```bash
   mkdir -p cmd/server/server.go
   ```
1. In the `main` method, define the `address` that the server listens on in `host:port` format. If you provide only the `port`, then the server listens on all available network interfaces. This means that `host` is important only if your machine has multiple network interfaces.
   
   Additionally, define a `timeout` duration. The `timeout` makes the server return after `timeout` seconds if the response is not complete:
   ```go
   const (
       addr    = "localhost:8080"
	   timeout = 10 * time.Second
   )
   ```
   > In some circumstances, you will see `:http` or `:http-alt` in place of a port number. This means the server looks up the port number in the `etc/services` file.
2. Create a new multiplexer server:
   ```go
   server := server.NewServer()
   ```
3. Create the server. Assign the `address` and `timeout` variables. For `Handler`, use `TimeoutHandler` so you can assign the timeout value:
   ```go
   server := &http.Server{
	   Addr:        addr,
	   Handler:     http.TimeoutHandler(server, timeout, "timeout"), // runs server for timeout seconds
	   ReadTimeout: timeout, // max time for reading the entire request
   }
   ```
4. Start the server. The normal behavior for stopping a server returns the `http.ErrServerClosed` error, so check for that error and return a message:
   ```go
   if err := server.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
	   fmt.Fprintln(os.Stderr, "server closed unexpectedly:", err)
   }   
   ```

#### Example 2

```go 
customServer := &http.Server{
    Addr:         fmt.Sprintf("%s:%d", *host, *port),
    Handler:      MuxFunc(*todoFile),
    ReadTimeout:  10 * time.Second,
    WriteTimeout: 10 * time.Second,
}
```

Start the server with `ListenAndServe`:

```go
if err := s.ListenAndServe(); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
}
```
## Timeouts

Add timeouts to the server instance. The following timeouts apply to all requests:

```go
srv := &http.Server{
	...
	IdleTimeout:  time.Minute,
	ReadTimeout:  5 * time.Second,
	WriteTimeout: 10 * time.Second,
}
```
In the preceding configuration:
- `IdleTimeout` closes all keep-alive connections after one minute. A keep-alive is HTTP connection reuse, where a single TCP connection is used to send and receive multiple HTTP requests and responses rather than opening new connections.
  
  Always set an `IdleTimeout` for the server.
- `ReadTimeout` sets the amount of time that the request headers and body are read after the request is first accepted. If the read operation exceeds the setting, the connection is closed.
  
  By default, `IdleTimeout` uses the same setting as `ReadTimout` if it is not explicitly set. Again--always set `IdleTimeout` on the server.
- `WriteTimeout` closes the connection if the server tries to write to the connection after the set length. This setting's timeout period behavior depends upon the protocol:
  - HTTP: Times out _setting seconds_ after the request header is read.
  - HTTPS: Times out _setting seconds_ after the request is accepted.
  
  > This setting does not impact long-running handlers--it impacts how long the handler writes from its buffer to the connection when it returns.


## Static files

To server static files (files that the server does not process), you have to create a `Handler` with the `FileServer()` function. The `FileServer()` handler serves HTTP requests with the files stored in the path that you pass as a function argument:

```go
staticFiles := http.FileServer(http.Dir("/public"))
```

When you register the file server handle, you have to strip the path pattern from the request URL, or the server attempts to serve files from an invalid path and returns a 404:


```go
mux.Handle("/static", http.StripPrefix("/static", staticFiles))
```
The preceding examples create a `Handler` named `staticFiles` that serves HTTP requests with the contents of the `/public` directory. When you register the `staticFiles`, you are telling the multiplexer that when there is a request to the `/static` path, strip `/static` from the request URL, and then search the `/public` directory for a file that matches the request.

### Directory listings

After you create a file server, users can access the static files in a browser by going to `/static/path/to/assets/`. To disable this, create a blank `index.html` file in each subdirectory in the `/static/` directory.


## Middleware

> The following sections detail how to create, register, and chain middlewares with Go's standard library. You can use the [justinas/alice](https://github.com/justinas/alice) package to simplify middleware code.

Middleware is code that executes on a request before or after your handlers. It is functionality that you want to share among your HTTP request handlers. For example:
- Logging
- Response compression
- Serving files from a cache

You chain middleware. After one middleware function executes, it calls the `ServeHTTP()` method on the next handler until a response returns.

Middleware uses the following pattern:

```go
func myMiddleware(next http.Handler) http.Handler {
    fn := func(w http.ResponseWriter, r *http.Request) {
        // TODO: Execute our middleware logic here...
        next.ServeHTTP(w, r)
    }

    return http.HandlerFunc(fn)
}
```

The pattern does the following:
- `myMiddleware` wraps the `next` handler that you pass as an argument.
- `fn` is a closure that has access to any values in the `next` handler's scope. It returns `next.ServerHTTP()`.
- `myMiddleware` converts the closure to a handler and returns.

This pattern can be simplified as follows:

```go
func myMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // TODO: Execute our middleware logic here...
        next.ServeHTTP(w, r)
    })
}
```

### Positioning

If you want the middleware to execute on all requests, place it before the server mux:

```
myMiddleware -> mux -> handler
```
If you want the middleware to execute on a specific request, place it after the server mux, but before the request handler:

```
mux -> myMiddleware -> handler
```

### Security headers

Create a new `/cmd/web/middleware.go` file and add a middleware function that adds secure headers to each request. These headers satisfy [OWASP standards](https://owasp.org/www-project-secure-headers/):

```go
func secureHeaders(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Security-Policy", "default-src 'self'; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com")
		w.Header().Set("Referrer-Policy", "origin-when-cross-origin")
		w.Header().Set("X-Content-Type-Options", "nosniff")
		w.Header().Set("X-Frame-Options", "deny")
		w.Header().Set("X-XSS-Protections", "0")

		next.ServeHTTP(w, r)
	})
}
```
The previous example adds these headers:
- [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) (CSP) retricts the resources that you site can access.
  > During development, CSP headers are a common cause of blocked resource loads.
- [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) includes the site URL in a `Referrer` header when users navigate away from your site.
- [X-Content-Type-Options: nosniff](https://security.stackexchange.com/questions/7506/using-file-extension-and-mime-type-as-output-by-file-i-b-combination-to-dete/7531#7531) prevents content-sniffing attacks.
- [X-Frame-Options: deny](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#click-jacking) prevents clickjacking in old browsers that do not support CSP.
- [X-XSS-Protection: 0](https://owasp.org/www-project-secure-headers/#x-xss-protection) disables blocking of cross-site scripting attacks because we are using CSP headers.

### Request logging

Log all requests before they are dispatched to a handler:

```go
func (app *application) logRequest(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		app.infoLog.Printf("%s - %s %s %s", r.RemoteAddr, r.Proto, r.Method, r.URL.RequestURI())

		next.ServeHTTP(w, r)
	})
}
```
The preceding middleware is added as a method on the application so that it can access the application dependencies, like the `infoLog`:

```go
type application struct {
	...
	infoLog       *log.Logger
	...
}
```

### Register middleware

If your middleware needs to execute on all routes, register it where you register routes. This example returns the [security middleware](#security-headers) from the [logging middleware](#request-logging), and the security middleware returns the `mux`.

```go
// return a handler
func (app *application) routes() http.Handler {

	mux := http.NewServeMux()
	// register routes

	return app.logRequest(secureHeaders(mux))
}
```

> Think of the `mux` as the main handler that you are returning--it is the handler registered to your server struct in `main`--so the other handlers enclose it.

### Panics

If there is a panic in a handler, the panic is isolated in the goroutine that runs the handler, so your application does not stop. However, you should handle panics to provide a good user experience.

The following middleware handles panics in a handler goroutine. It recovers from the panic, then sets a `Connection: close` header so the server closes the panicked connection.

```go
func (app *application) recoverPanic(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Create a deferred function (which will always be run in the event
        // of a panic as Go unwinds the stack).
        defer func() {
            // Use the builtin recover function to check if there has been a
            // panic or not. If there has...
            if err := recover(); err != nil {
                // Set a "Connection: close" header on the response.
                w.Header().Set("Connection", "close")
                // Call the app.serverError helper method to return a 500
                // Internal Server response.
                app.serverError(w, fmt.Errorf("%s", err))
            }
        }()

        next.ServeHTTP(w, r)
    })
}
```

Add this handler to the beginning of the middleware chain so that it recovers from any panic that occurs in any handler:

```go
func (app *application) routes() http.Handler {

	mux := http.NewServeMux()
	// register routes

	return app.recoverPanic(app.logRequest(secureHeaders(mux)))
}

```

The previous panic handlers recover only when the panic occurs in the same goroutine that runs the handler. If your handler spins off another goroutine that might panic, you can add the following anonymous function to do the work and handle the possible panic:

```go
func myHandler(w http.ResponseWriter, r *http.Request) {
    ...

    // Spin up a new goroutine to do some background processing.
    go func() {
        defer func() {
            if err := recover(); err != nil {
                log.Print(fmt.Errorf("%s\n%s", err, debug.Stack()))
            }
        }()

        doSomeBackgroundProcessing()
    }()

    w.Write([]byte("OK"))
}
```
## Forms

Parse a form with the Request's `.ParseForm()` method. This method stores data from the form in the requests `PostForm` field as a map of `Values`. In addition, the Request type has a `Form` field. The two have the following differences:
- `PostForm` is populated for only `POST`, `PATCH`, and `PUT` requests.
- `Form` is populated for all requests, including query string parameters. Use the `Get()` method to access query string parameters.

The `name` attribute of each HTML form element is the key in the map, and you access the value with the `.Get()` method. For example:

```html
<input type="text" name="title">
```
To get the value of this input, use the following:

```go
title := r.PostForm.Get("title")
```
Consider the following form:

```html
<form action="/snippet/create" method="post">
    <div>
        <label>Title:</label>
        <input type="text" name="title">
    </div>
    <div>
        <label>Content:</label>
        <textarea name="content"></textarea>
    </div>
    <div>
        <label>Delete in:</label>
        <input type="radio" name="expires" value="365" checked> One Year
        <input type="radio" name="expires" value="7"> One Week
        <input type="radio" name="expires" value="1"> One Day
    </div>
    <div>
        <input type="submit" value="Publish snippet">
    </div>
</form>
```

The following method parses the form, extracts its values with error checking, then inserts the values in a table. At the end, it redirects the request to a new path with `Redirect()`:

```go
type snippetCreateForm struct {
	Title   string
	Content string
	Expires int
	validator.Validator
}

func (app *application) snippetCreatePost(w http.ResponseWriter, r *http.Request) {
    // First we call r.ParseForm() which adds any data in POST request bodies
    // to the r.PostForm map. This also works in the same way for PUT and PATCH
    // requests. If there are any errors, we use our app.ClientError() helper to 
    // send a 400 Bad Request response to the user.
	err := r.ParseForm()
	if err != nil {
		app.clientError(w, http.StatusBadRequest)
		return
	}

    // The r.PostForm.Get() method always returns the form data as a *string*.
    // However, we're expecting our expires value to be a number, and want to
    // represent it in our Go code as an integer. So we need to manually covert
    // the form data to an integer using strconv.Atoi(), and we send a 400 Bad
    // Request response if the conversion fails.
	expires, err := strconv.Atoi(r.PostForm.Get("expires"))
	if err != nil {
		app.clientError(w, http.StatusBadRequest)
		return
	}
    // Use the r.PostForm.Get() method to retrieve the title and content
    // from the r.PostForm map.
	form := snippetCreateForm{
		Title:   r.PostForm.Get("title"),
		Content: r.PostForm.Get("content"),
		Expires: expires,
	}

	form.CheckField(validator.NotBlank(form.Title), "title", "This field cannot be blank")
	form.CheckField(validator.MaxChars(form.Title, 100), "title", "This field cannot be more than 100 characters long")
	form.CheckField(validator.NotBlank(form.Content), "content", "This field cannot be blank")
	form.CheckField(validator.PermittedInt(form.Expires, 1, 7, 365), "expires", "This field must equal 1, 7 or 365")

	if !form.Valid() {
		data := app.newTemplateData(r)
		data.Form = form
		app.render(w, http.StatusUnprocessableEntity, "create.tmpl.html", data)
		return
	}

	id, err := app.snippets.Insert(form.Title, form.Content, form.Expires)
	if err != nil {
		app.serverError(w, err)
		return
	}

	http.Redirect(w, r, fmt.Sprintf("/snippet/view/%d", id), http.StatusSeeOther)
}
```

If a form field sends multiple values, you have to loop through the `PostForm`. For example:

```html
<input type="checkbox" name="items" value="foo"> Foo
<input type="checkbox" name="items" value="bar"> Bar
<input type="checkbox" name="items" value="baz"> Baz
```

```go
for i, item := range r.PostForm["items"] {
    fmt.Fprintf(w, "%d: Item %s\n", i, item)
}
```

`ParseForm` cannot parse request bodies that exceed 10MB, or it returns an error. You can change this amount using `MaxBytesReader()`:

```go
// Limit the request body size to 4096 bytes
r.Body = http.MaxBytesReader(w, r.Body, 4096)

err := r.ParseForm()
if err != nil {
    http.Error(w, "Bad Request", http.StatusBadRequest)
    return
}
```
When the limit is reached, the server closes the underlying TCP connection.

### Validation

See [Validation Snippets for Go](https://www.alexedwards.net/blog/validation-snippets-for-go) for field validation patterns.

Validate form values with standard `if` clauses and your validation criteria for best practices. A useful pattern is to create a map, and store the field and a discription of any invalid data in the map:

```go
// Initialize a map to hold any validation errors for the form fields.
fieldErrors := make(map[string]string)

// validate fields

// If there are any errors, dump them in a plain text HTTP response and
    // return from the handler.
if len(fieldErrors) > 0 {
	fmt.Fprint(w, fieldErrors)
	return
}
```

## Sessions

Create a session with the [scs package](https://github.com/alexedwards/scs) with the following steps:

First, create a table to store the session data.

In the `main` method:
1. Add the session manager to the application struct.
2. Create the session manager with `New()`.
3. Configure the new session manager to use the database.
4. Set a lifetime on the session manager.

In `routes.go`:
5. Create a new middleware chain.
6. Update any routes with the middleware chain.

In the relevant handlers:
1. Add the message or busniess logic into the session context.

In the `newTemplateData` helper method:
1. Add a field that makes the business logic available to the application when it is present.

In `templates.go`:
1. Add a field to the `templateData` struct that stores the data that you want to pass to the template.

Update the template to display the dynamic data from `templateData`.

Now, the `LoadAndSave()` middleware checks each incoming request for a cookie. If a cookie is present, it does the following:
1. Reads the token
2. Uses the token to retrieve session data from the database
3. Adds the session data to the request context so the handlers can use it.

Any changes to session data are updated in the handler request context, and then the middleware updates the changes when it returns.

## Shutdown gracefully

You might have concurrent code that is still running when your server fails, and you need to make sure that the process completes before the server stops. You can add a WaitGroup to the application struct, and wait until all goroutines complete before you shut down the server.

Add the WaitGroup to the application struct:

```go
// cmd/api/main.go
type application struct {
    config config
    logger *jsonlog.Logger
    models data.Models
    mailer mailer.Mailer
    wg     sync.WaitGroup
}
```

Wait for all goroutines to complete before you shutdown the server:

```go
// cmd/api/server.go

func (app *application) serve() error {
    srv := &http.Server{
        Addr:         fmt.Sprintf(":%d", app.config.port),
        Handler:      app.routes(),
        IdleTimeout:  time.Minute,
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
    }

    shutdownError := make(chan error)

    go func() {
        quit := make(chan os.Signal, 1)
        signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
        s := <-quit

        app.logger.PrintInfo("caught signal", map[string]string{
            "signal": s.String(),
        })

        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer cancel()

        // Call Shutdown() on the server like before, but now we only send on the
        // shutdownError channel if it returns an error.
        err := srv.Shutdown(ctx)
        if err != nil {
            shutdownError <- err
        }

        // Log a message to say that we're waiting for any background goroutines to
        // complete their tasks.
        app.logger.PrintInfo("completing background tasks", map[string]string{
            "addr": srv.Addr,
        })

        // Call Wait() to block until our WaitGroup counter is zero --- essentially
        // blocking until the background goroutines have finished. Then we return nil on
        // the shutdownError channel, to indicate that the shutdown completed without
        // any issues.
        app.wg.Wait()
        shutdownError <- nil
    }()

    app.logger.PrintInfo("starting server", map[string]string{
        "addr": srv.Addr,
        "env":  app.config.env,
    })

    err := srv.ListenAndServe()
    if !errors.Is(err, http.ErrServerClosed) {
        return err
    }

    err = <-shutdownError
    if err != nil {
        return err
    }

    app.logger.PrintInfo("stopped server", map[string]string{
        "addr": srv.Addr,
    })

    return nil
}
```

## Existing docs


Implement the fields that you need when you define a custom server:



### Multiplexers

A multiplexer maps incoming requests to the proper handler functions using the request URL. `net/http` provides the DefaultServeMux function that returns the default multiplexer, but you should define a custom multiplexer for the following reasons:
- The default registers routes globally.
- You can add dependencies to the routes.
- Custom multiplexers allow integration testing

### Router functions

```go
func todoRouter(todoFile string, l sync.Locker) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		list := &todo.List{}

		l.Lock()
		defer l.Unlock()
		if err := list.Get(todoFile); err != nil {
			replyError(w, r, http.StatusInternalServerError, err.Error())
			return
		}

		if r.URL.Path == "" {
			switch r.Method {
			case http.MethodGet:
				getAllHandler(w, r, list)
			case http.MethodPost:
				addHandler(w, r, list, todoFile)
			default:
				message := "Method not supported"
				replyError(w, r, http.StatusMethodNotAllowed, message)
			}
			return
		}

		id, err := validateID(r.URL.Path, list)
		if err != nil {
			if errors.Is(err, ErrNotFound) {
				replyError(w, r, http.StatusNotFound, err.Error())
				return
			}
			replyError(w, r, http.StatusBadRequest, err.Error())
			return
		}

		switch r.Method {
		case http.MethodGet:
			getOneHandler(w, r, list, id)
		case http.MethodDelete:
			deleteHandler(w, r, list, id, todoFile)
		case http.MethodPatch:
			patchHandler(w, r, list, id, todoFile)
		default:
			message := "Method not supported"
			replyError(w, r, http.StatusMethodNotAllowed, message)
		}
	}
}
```

Multiplexer definition:

```go
func newMux(todoFile string) http.Handler {
	m := http.NewServeMux()
	mu := &sync.Mutex{}

	m.HandleFunc("/", rootHandler)

	t := todoRouter(todoFile, mu)

	m.Handle("/todo", http.StripPrefix("/todo", t))
	m.Handle("/todo/", http.StripPrefix("/todo/", t))

	return m
}
```