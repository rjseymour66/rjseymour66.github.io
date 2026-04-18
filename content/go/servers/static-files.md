+++
title = 'Static files'
date = '2025-09-22T23:11:44-04:00'
weight = 30
draft = false
+++

Go's standard library can serve static files the same as a standalone web server like Apache or nginx.


## Serving files

### All files in a directory

You can use the `FileServer` function as the handler in `ListenAndServe` to serve files from a directory. It returns a `If-Modified-Since` HTTP header and `304 Not Modified` response if the file is already cached on the user's machine:

1. `http.Dir` implements the `FileSystem` interface. This means that the path you pass the function is treated like the root directory on disk that the program serves files from.  
   In this example, the app serves files from the `file/` directory. You do not need to specify a file because it serves all in that directory. If you were to go to `localhost:8080`, you would see a list of links to the files in the directory. When you clicked the link, you are sent to `localhost:8080/<filename>.html`.
2. Use `FileServer` as the server's handler.
```go
func main() {
	dir := http.Dir("./files")                                                      // 1
	if err := http.ListenAndServe(":8080", http.FileServer(dir)); err != nil {      // 2
		panic(err)
	}
}
```

### Single file

You can use `ServeFile` to serve a specific file with a handler. This example registers a handler with the `DefaultMux` server. The handler uses `ServeFile` to serve a file named `hello.html` at the web root path (`/`):
1. `ServeFile` takes the response, request, and a file or directory string as its arguments.

```go
func main() {
	http.HandleFunc("/", hello)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}

func hello(w http.ResponseWriter, r *http.Request) {
	http.ServeFile(w, r, "./files/hello.html") 
}
```

### Subdirectories


1. Defines a root directory on your app's filesystem. This root directory is where you store static files to serve.
2. This line contains a few components:
   - `http.FileServer` returns a handler that serves files from the given directory (`./files/`).
   - `StripPrefix` returns a handler that removes the given string from the path before it serves HTTP requests. Here, it removes `/static/` from the path. It also accepts a handler, which is the `FileServer(dir)` handler.
      
     For example, if you are serving CSS from `/static/style.css`, `StripPrefix` changes the path from `/static/style.css` to `/style.css`, and then `FileServer` looks for the file in `./files/style.css`. So, if you go to `http://localhost:8080/static/style.css`, the server serves `./files/style.css` but still displays `http://localhost:8080/static/style.css` in the address bar.

	 This line could be rewritten with the following ways:
	 ```go
	 dir := http.Dir("./files/")
	 fs := http.FileServer(dir)
	 fs = http.StripPrefix("/files/", fs)
	 http.Handle("/files/", handler)
	 http.ListenAndServe(":8080", nil)
	 ```
3. Register the `/static/` path with the handler that serves the static files.
```go
func main() {
	dir := http.Dir("./files/")                                         // 1
	handler := http.StripPrefix("/static/", http.FileServer(dir))       // 2
	http.Handle("/static/", handler)                                    // 3
	http.HandleFunc("/", hello)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}

func hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "Hello, World!")
}
```

### From an alternate location

You might need to serve your styles or JS from a CDN to increase performance. This example lets you pass the location of the stylesheet as a command-line argument:
1. Define a flag with the default stylesheet location. For example, in your dev environment.
2. The template string adds a `Location` in place of the `href` URL.
3. Create an anonymous struct that contains the template data.
4. Parse the flags and template before `main` runs.

```go
var t *template.Template
var l = flag.String("location", "http://localhost:8080", "A location")          // 1

var tpl = `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello!</title>
	<link rel="stylesheet" href="{{.Location}}/style.css">                      // 2
</head>
<body>
    <h1>This is a test!</h1>
</body>
</html>`

func serveStyles(w http.ResponseWriter, r *http.Request) {
	data := struct{ Location *string }{                                         // 3
		Location: l,
	}
	t.Execute(w, data)
}

func init() {
	flag.Parse()                                                                // 4
	t = template.Must(template.New("page").Parse(tpl))
}

func main() {
	http.HandleFunc("/styles", serveStyles)
	if err := http.ListenAndServe(":8080", nil); err != nil {
		panic(err)
	}
}
```


## Embedding files in a binary

You might need to store non-executable files in a binary, such as images or stylesheets, directly in the Go binary. This means that your program does not have to locate the files in the host's file system. Embedding files keeps your application self-contained, which is great for the following use cases:
- Container image or CLI tool: Limi the number of resources that needs to be built into the image or tool.
- Doc servers: Bundle markdown files into the binary to serve.
- Web UI for CLI apps: Monitoring or db tools can ship wit a built-in web UI.
- Improved portability: No risk of missing critical files.


### Filesystem

The following example embeds a filesystem in your binary. It converts a file to bytes or a string, stores that data in a variable, then it uses the variable ot serve the file with the `ServeContent` method.

Here is a basic example:
1. At build time, the compiler takes the `files/` directory and embeds it in the binary. The embedded value is defined in the line directly below the `go:embed` directive. These files are served from the `<server-IP>:<port>/files/<filename>` directory. For example, you can view `style.css` at `http://localhost:8080/files/style.css` with the following project structure:
   ```bash
   project-root
   ├── files
   │   ├── hello.html
   │   ├── style.css
   │   └── test.html
   ├── go.mod
   └── main.go
   ```
   The `//go:embed` directive cannot exist within a function---it is for the compiler only.
2. `embed.FS` is a filesystem type that lets you access embedded files like a read-only filesystem. `f` is the in-memory representation of that filesystem.
3. `http.FS` is an adapter that converts the `embed.FS` into a `FileSystem` implementation.

```go
//go:embed files            // 1
var f embed.FS              // 2

func main() {
	if err := http.ListenAndServe(":8080", http.FileServer(http.FS(f))); err != nil {   // 3
		panic(err)
	}
}
```

### Single string

You can also embed a single file and store its contents in a string. This has the following use cases:
- Inline templates.
- Configuration file, so your app always has fallback settings.
- License or EULA, such as a legal notice or README.
- SQL queries that you can load at runtime instead of shipping `.sql` files.

As a simple demonstration, this example reads `hello.html`, stores it in a variable, then logs the variable to STDOUT:

```go
//go:embed files/hello.html
var myString string

func main() {
	log.Println(fmt.Sprintf("embedded value: %s", myString))
}
```

### Config files

Here is an example of an embedded JSON configuration file:
1. Embed the config file as a `string`.
2. Create a struct for the config file contents.
3. Parse the embedded config into a struct with `json.Unmarshal`.
4. Use the config. Here, we print its contents to STDOUT.
```go
import (
	_ "embed"
	"encoding/json"
	"fmt"
	"log"
)

//go:embed config.json
var configData string                           // 1

type Config struct {                            // 2
	Server   string `json:"server"`
	Port     int    `json:"port"`
	Username string `json:"username"`
	Password string `json:"password"`
}

func main() {
	var cfg Config                              // 3
	if err := json.Unmarshal([]byte(configData), &cfg); err != nil {
		log.Fatalf("error parsing config: %v", err)
	}

	fmt.Printf("Connecting to %s:%d as %s\n",   // 4
    cfg.Server, 
    cfg.Port, 
    cfg.Username
    )
}
```