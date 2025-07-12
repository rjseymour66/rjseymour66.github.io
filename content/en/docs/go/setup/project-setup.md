---
title: "Project setup"
weight: 10
description: >
  Setting up a Go project.
---

## Production-ready checklist

| Category      | Requirement                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------- |
| Reliability   | ✅ Graceful shutdown<br>✅ Timeouts<br>✅ Panic recovery (optional)                            |
| Observability | ✅ Logs with context (structured logging preferred)<br>⬜ Tracing (OpenTelemetry)             |
| Testing       | ✅ Unit tests<br>⬜ Integration tests                                                         |
| Security      | ✅ JSON validation<br>⬜ Input sanitation if needed<br>⬜ TLS (behind reverse proxy or native) |
| Performance   | ✅ Minimal allocations<br>⬜ Benchmark tests                                                  |
| Deployment    | ✅ Buildable with Go modules<br>✅ Single binary<br>⬜ Dockerfile                              |
| Operations    | ⬜ Health check endpoint<br>⬜ Metrics (Prometheus)                                           |
| Documentation | ✅ Self-explanatory code<br>⬜ README.md with usage and curl examples                         |

## Requirements analysis

### Functional requirements

_Functional requirements_ is a list of the core functionalities that the system is expected to implement and how the actors (users, other system components or services) interact with it. To establish the functional requirements, you must write _user stories_.

User stories describe business value and include a list of acceptance criteria that acts as a verification tool that each goal is met.

#### User story template

As an _`actor`_, I need to be able to _`short requirement`_, so as to _`reason/business value`_.

The acceptance criteria for this user story are as follows:
- _`acceptance criteria 1`_
- _`acceptance criteria 2`_
- ...

### Non-functional requirements
_Non-functional requirements_ include items like _service-level objectives_ (SLOs) and capacity and scalability requirements.

To 

## Makefile

Create a Makefile at the base of your project to manage project tasks, builds, and dependencies. A Makefile consists of targets, which are tasks that you can run by entering `make target-name`.


The following sections describe common Makefile idioms and targets for a basic Go project.

### Global variables

Add global variables at the top of the Makefile in all caps. For example, the following variable defines the Go version:

```Makefile
GO_VERSION := 1.19.4
```

To use this variable in the Makefile, enclose it in parentheses and prepend it with a `$`:

```Makefile
$(GO_VERSION)
```

### Build

Build the application for your host machine:

```Makefile
build:
    go build -o appname path/to/main.go
```

Build binaries for multiple architectures, and store them in a `bin/` directory at the project root:

```Makefile
build:
	# Linux
	GOOS=linux   GOARCH=amd64 go build -o ./bin/appname_linux_amd64   .path/to/main.go
	# macOS
	GOOS=darwin  GOARCH=amd64 go build -o ./bin/appname_darwin_amd64  .path/to/main.go
	# windows
	GOOS=windows GOARCH=amd64 go build -o ./bin/appname_win_amd64.exe .path/to/main.go
```

### Cross-compilation

You need to know the `GOOS` and `GOOARCH` values to compile the correct binaries.

Next, you can cross compile for multiple operating systems with a Makefile. Create a make target that compiles multiple binaries and places them in the `/bin` directory:

```makefile
compile:
	# Linux
	GOOS=linux GOARCH=amd64 go build -o ./bin/hit_linux_amd64 ./cmd/hit
	# macOS
	GOOS=darwin GOARCH=amd64 go build -o ./bin/hit_darwin_amd64 ./cmd/hit
	# windows
	GOOS=windows GOARCH=amd64 go build -o ./bin/hit_win_amd64.exe ./cmd/hit
```
> Make sure that you add the `/bin` directory to the `.gitignore` file.

### Test

Run go tests:

```Makefile
test:
    go test ./... -coverprofile=coverage.out
```

### Code coverage

View how much of the source code is adequetly tested:
> What does this do?

```Makefile
coverage:
	go tool cover -func coverage.out | grep "total:" | \
	awk '{print ((int($$3) > 80) != 1) }'
```

### Generate coverage report

Runs the Go coverage tool to generate an HTML page that describes what code is covered by tests:

```Makefile
report:
	go tool cover -html=coverage.out -o cover.html
```

### Format your source code

This is not an issue in VSCode, but you can add a target that verifies the Go source code is formatted correctly:

```Makefile
check-format:
	test -z $$(go fmt ./...)
```

### Static linters

[golangci-lint](https://golangci-lint.run/) is a Go linters aggregator--it provides multiple linters for Go source code.

#### Install

```Makefile
install-lint: 
	sudo curl -sSfL \
	https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh \
 	| sh -s -- -b $$(go env GOPATH)/bin v1.51.1
```

#### Run the linter

```Makefile
static-check:
	golangci-lint run ./...
```

## Github Actions


## Modules

A Go code repository comprises of exactly one module. The module includes packages, and these packages include source files. To create a module, go to the top-level directory of the project and enter the following command:

```bash
go mod init <project-name>
```
The preceding command creates a `go.mod` file in the top-level of your project that lists your project name at the top.

Module names should be unique within the Go community. This prevents conflicts with other public libraries. A common pattern is to use a URL that you own, such a `project-name.example.com`. Another very common pattern is to use the path where the project exists, minus the scheme. For example, `github.com/username/projectname`.

## Packages

Packages are directories in a Go project. A package name should describe what it provides, not what it does. The name of the directory is the package name. For example, source files in the `go-src/stocks/` package are in the `stocks` package. At the top of the file, declare package names with `package <package-name>`, and import packages with the `import <package-name>` statement.
> `import` statements use the fully-qualified package name. This begins with the module name containing the package. For example, `import go-src/<package-name>`

Prepend any imported package code with the package name, or an alias for the package: `alias package/name`. For example, `s go-src/stocks` allows you to prepend any code with `s.`, such as `s.Investment`.

*main*: any program that has to run as an application must be in the `main` package.




When you write external tests, use a `_test` suffix. For example, a package that contains external tests for the `url` package is `url_test`.

### Import external packages

When you import an external package, you list the module name in the `go.mod` file, followed by the path to the specific library from the project root. For example, if the `go.mod` file contains the following:

```go 
module url
...
```

Then you import the module as follows:
```go
import "url/path/to/library"
```

Commonly, packages are publically available in repositories, and the module name is the path to the root of the repository:

```go 
module github.com/rjs/url-parser
...
```

In this case, the import statement for the `parser` package within this repo is as follows:

```go
import "github.com/rjs/url-parser/parser"
```



## CLI tools


```shell
.
├── cmd 
│   └── todo
│       ├── main.go         # config, parse, switch {} flags
│       └── main_test.go    # integration tests (user interaction)
├── go.mod
├── todo.go                 # API logic for flags
└── todo_test.go            # unit tests

```


`/internal` directory is special because other projects cannot import anything  in this directory. This is a good place for domain code.
If an entity needs to be accessible to all domain code, place its file in the `/internal` directory. Its package name is the project package.
Each subdirectory in `/internal` is a domain.

You create a tool that the user interacts with and is responsible for the following:
- Parses the flags
- Validates flags
- Calls the business logic library

Go uses the `cmd` directory for executables (the entry point) such as CLI tools. Within each `cmd/subdirectory`, you can name the entry point `main.go` or the name of the package, such as `hit.go`. Regardless of the file name, it must be in the `main` package because that package is what makes a file executable.

Next, you have to create the tool library that contains the business logic. This is a standalone package, so use the name of the library that you are building.

The following is a simple directory structure for the `hit` tool:

```shell
hit-tool
├── cmd         # Executable directory
│   └── hit     # CLI tool directory
├── go.mod
└── hit         # Library directory

```

## Web apps

Web app structure separates the Go code and the web assets to simplify building and deploying. 

```shell
.
├── cmd
│   └── web
│       ├── handlers.go
│       └── main.go
├── go.mod
├── internal
├── README.md
└── ui
    ├── html
    └── static

```

`/cmd`
: Application-specific code for executables.

`/internal`
: Non-application-specific code, including reusable code like validation helpers and SQL database models.
Code in the `/internal` directory cannot be imported by external projects.

`/ui`
: User-interface assets for the web app, including templates and static files (CSS, Javascript).

### Configuration with CLI flags

Add flags to manage environment configurations. CLI flags are easier to manage, have default values, and have built-in help.

If you plan to use environment variables, you can pass the env vars to the flag:

```go
$ export ENV_VAR="value"
$ go run ./example -addr=$ENV_VAR
```
This prevents you from relying on env vars in your code and needing to convert the values from string to the flag type. The `flag` package handles those conversions themselves.

## Dependency injection

**Definition here**

In web applications, handlers need access to multiple dependencies. The easiest way to do that is to inject the dependencies with structs:

```go
type application struct {
	logger: *log.logger
	db:     *sql.DB
	cache:  *cache
}
```

Then, you can initialize your app in the main method, and _inject_ any dependencies in the new `application` object at runtime:

```go
func main() {
	// instantiate myLogger
	// mySQLHandle db config
	// instantiate myCache

	app := &application {
		logger: myLogger,
		db:     mySQLHandle,
		cache:  myCache,
	}
}
```

## Web app checklist


Plan your routes:

| Method | Pattern | Handler | Action |
| :----- | :------ | :------ | :----- |
|        |         |         |