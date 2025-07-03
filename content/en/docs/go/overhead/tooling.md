---
title: "Go tooling"
weight: 20
description: >
  Go tooling and tips.
---

See [An Overview of Go's Tooling](https://www.alexedwards.net/blog/an-overview-of-go-tooling) for a much better overview.

```shell
# go fmt
$ gofmt -w <file>.go    # formats <file>.go
$ gofmt -l dirname/*.go # lists files in dir that do not conform to go formatting 

# go run
$ go run .                    # runs binary in cwd
$ go run <binary-name>
$ go run ./cmd/web            # runs proj-root/cmd/web/main.go

# go build
$ go build                    # uses module name for binary name
$ go build -o <binary-name>   # provide binary name

# go test
$ go test -v                  # verbose output
$ go test -v ./<dirname>/     # run tests in a specific directory
$ go test -v ./cmd/

# go get for dependencies
$ go get github.com/entire/module/path
$ go get -u github.com/entire/module/path         # upgrade to latest minor version or patch
$ go get -u github.com/entire/module/path@v2.0.0  # upgrade to specific version
$ go get github.com/entire/module/path@none       # remove unused package (same as 'go mod tidy -v')

# After go get, update dependencies
$ cd <project-root>
$ go mod tidy
```

## Go modules

Go modules group related packages into a single unit to be versioned together. Because they track an application's dependencies, they ensure that users build the application with the same dependencies as the original developer. Go modules allow you to write go programs outside of the $GOPATH directory, as in previous releases.

Go sum records the checksum for each module in the application to ensure that each build uses the correct version.

Go modules are tracked in `go.mod`. Update the `go.mod` file with [mod commands](https://go.dev/ref/mod#mod-commands):

```shell
$ go mod tidy [-v]            # reconcile project dependencies
$ go mod verify               # verifies the checksums in go.sum match the downloaded packages on your machine

$ go list                     # list project packages
$ go list -m                  # list project modules
```

In the source file, import each dependency as it is described in `go.mod`:

```go
module moduleName

go 1.19

require (
  github.com/entire/module/path/v1
  github.com/entire/module/path/dependcy2
)
```

```go
import (
    "github.com/entire/module/path/v1"
    "github.com/entire/module/path/dependcy2"
)
```

## Cross-compilation

Build static go binaries for operating systems that are different than the one that you are building it on. Because you build a static binary, the target machine does not need any additional libraries or tools to run the binary.

For example, use the `GOOS` environment variable with the `build` command to compile for a Windows machine:

```go
$ GOOS=window go build
```

For a list of accepted GOOS values, see https://go.dev/src/go/build/syslist.go