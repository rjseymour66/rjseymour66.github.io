+++
title = 'Tooling'
date = '2025-08-12T23:44:57-04:00'
weight = 10
draft = false
+++

For a deeper reference, see Alex Edwards' [An overview of Go's tooling](https://www.alexedwards.net/blog/an-overview-of-go-tooling).

## Dependencies and maintenance

Go modules enable reproducible builds through a module cache and a defined set of versioned dependencies. They address three common problems in software releases:

- **Reliable versioning**: Go modules use Semantic Versioning (`MAJOR.MINOR.PATCH`) to communicate compatibility:
  - `MAJOR`: Breaking changes that require code modification. Read the changelog.
  - `MINOR`: Backward-compatible new features.
  - `PATCH`: Backward-compatible bug fixes.
- **Reproducible builds**: The Go module proxy sits between your code and source control to download, cache, and serve modules. Your code never talks directly to GitHub.
- **Dependency hygiene**: Running `go mod tidy` removes unused dependencies and ensures `go.mod` reflects only what your code imports.

### go.mod and go.sum

These files support the Minimal Version Selection (MVS) algorithm that manages your project dependencies:

- `go.mod`: Lists your module's direct dependencies and their minimum required versions. MVS selects the minimum version that satisfies all requirements across your full dependency graph.
- `go.sum`: Records cryptographic checksums for each module version your project uses. Go verifies these checksums on every build to detect tampering or unexpected changes.


### go mod

`go mod` is Go's built-in dependency manager. Start every project with `go mod init` to create a `go.mod` file at the project root. The module path is typically the URL of the repository, but any path works for local-only projects:

```bash
go mod init [module/path]
go mod init github.com/username/repo-name   # GH repo
go mod init myproject                       # local-only
```

`go.mod` records the module name, the minimum Go version, and any third-party dependencies.

### go get

`go get` adds or upgrades a module dependency in your project:

```bash
go get github.com/entire/module/path            # 1
go get -u github.com/entire/module/path         # 2
go get github.com/entire/module/path@v2.0.0     # 3
go get github.com/entire/module/path@none       # 4
```

1. Add the latest version of a module.
2. Upgrade to the latest minor version or patch.
3. Upgrade or downgrade to a specific version.
4. Remove a module from `go.mod`.

Run `go mod tidy` after adding or removing dependencies to keep `go.mod` and `go.sum` consistent.

### go list

`go list -m all` prints your module and every dependency in the build graph:

```bash
go list -m all
myproject
github.com/alecthomas/kingpin/v2 v2.4.0
github.com/alecthomas/units v0.0.0-20211218093645-b94a6e3cc137
github.com/beorn7/perks v1.0.1
...
```

### Finding versions

Use `git ls-remote` to list the available tags and branches of any module repository:

1. `-t` returns tags (versions).
2. `-h` returns heads (branch tips).

```bash
git ls-remote -t https://github.com/spf13/cobra.git               # 1
0eeaf8392f5b04950925b8a69fe70f110fa7cbfc	refs/tags/v1.7.0
b12896167c61cb7a17ee5f15c2ba0729d78793db	refs/tags/v1.8.0
...

git ls-remote -h https://github.com/spf13/cobra.git               # 2
db9d1d0073d27a0a2d9a8c1bc52aa0af4374d265	refs/heads/main
b4617d0b9670ad14039b2739167fd35a60f557c5	refs/heads/release-1.8
```

### go mod tidy

Run `go mod tidy` after adding, removing, or updating any imports. It removes unused dependencies, adds any missing ones, and updates both `go.mod` and `go.sum`:

```bash
go mod tidy
```

### go fmt

`go fmt` formats every `.go` file in a package using Go's built-in style rules. It's a wrapper around `gofmt` that operates on whole packages rather than individual files:

```bash
go fmt ./...                    # format all packages in the module
gofmt -w <file>.go              # format a single file
gofmt -l dirname/*.go           # list files that don't conform to Go style
```

### goimports

`goimports` extends `gofmt` by automatically adding missing import statements and removing unused ones. Configure your editor to run it on save for the best experience:

```bash
goimports -w <file>.go      # format a single file and write changes
goimports -w ./...          # format all .go files in the module
```

The `-w` flag writes changes back to the file. Without it, `goimports` prints the diff to stdout without modifying anything. Use this to preview changes before applying them.

### go vet

`go vet` analyzes your code for correctness issues that the compiler doesn't catch: mismatched `Printf` format verbs, unreachable code, suspicious composite literals, and more. Run it on every commit:

```bash
go vet program.go
go vet ./...        # check all packages in the module
```



## Executables

### go run

`go run` compiles and executes a Go program in one step without producing a permanent binary. Go compiles to a temporary directory and removes it after the program exits:

```bash
go run .                    # runs binary in cwd
go run <binary-name>
go run ./cmd/web            # runs proj-root/cmd/web/main.go
```

### go build

`go build` compiles your Go code into an executable binary in the current directory:

```bash
go build                        # uses module name for binary name
go build -o <binary-name>       # provide binary name
go build -o bin/hit ./cmd/hit   # find /cmd/hit, confirm its package main, build hit binary in bin/
```

### Cross-platform compilation

Prefix `GOOS` and `GOARCH` to any `go build` or `go run` command to compile for a different operating system or architecture:

```bash
GOOS=linux GOARCH=amd64 go build -o app
GOOS=windows GOARCH=amd64 go build -o app.exe
GOOS=darwin GOARCH=amd64 go run
```

This table lists the accepted values:

| GOOS        | Description                             |
| ----------- | --------------------------------------- |
| `aix`       | IBM AIX                                 |
| `android`   | Android                                 |
| `darwin`    | macOS, iOS, iPadOS (Apple platforms)    |
| `dragonfly` | DragonFly BSD                           |
| `freebsd`   | FreeBSD                                 |
| `illumos`   | Illumos/Solaris derivatives             |
| `ios`       | iOS (device & simulator)                |
| `js`        | WebAssembly JavaScript host environment |
| `linux`     | Linux                                   |
| `netbsd`    | NetBSD                                  |
| `openbsd`   | OpenBSD                                 |
| `plan9`     | Plan 9                                  |
| `solaris`   | Oracle Solaris                          |
| `wasip1`    | WASI Preview 1                          |
| `windows`   | Windows                                 |


| GOARCH     | Description                                                                  |
| ---------- | ---------------------------------------------------------------------------- |
| `386`      | 32-bit x86                                                                   |
| `amd64`    | 64-bit x86                                                                   |
| `amd64p32` | 64-bit x86 with 32-bit pointers (old, only for nacl; effectively deprecated) |
| `arm`      | ARM 32-bit                                                                   |
| `arm64`    | ARM 64-bit                                                                   |
| `loong64`  | LoongArch 64                                                                 |
| `mips`     | MIPS (big-endian), 32-bit                                                    |
| `mips64`   | MIPS64 (big-endian)                                                          |
| `mipsle`   | MIPS (little-endian), 32-bit                                                 |
| `mips64le` | MIPS64 (little-endian)                                                       |
| `ppc64`    | POWERPC 64-bit (big-endian)                                                  |
| `ppc64le`  | POWERPC 64-bit (little-endian)                                               |
| `riscv64`  | RISC-V 64                                                                    |
| `s390x`    | IBM Z / s390x                                                                |
| `wasm`     | WebAssembly                                                                  |


## Testing

### go test

`go test` compiles and runs tests in the specified packages:

```bash
go test ./...                   # run all tests in the module
go test -v ./...                # verbose output with individual test names
go test -v ./<dirname>/         # run tests in a specific directory
```

### go test -cover

The `-cover` flag reports the percentage of statements covered by tests:

```bash
go test -cover ./...
```

To view coverage line by line, generate an HTML report:

```bash
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

