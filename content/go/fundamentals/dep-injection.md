+++
title = 'Dependency injection'
date = '2026-02-16T22:54:21-05:00'
weight = 80
draft = false
+++

`main` is hard to test because it produces side effects: actions that reach outside a function's own scope. Common side effects include:

- Reading from `os.Args` or environment variables
- Writing to `os.Stdout` or `os.Stderr`
- Calling `os.Exit`
- Reading or writing files directly

You can't capture what was written to stdout in a test, and you can't stop `os.Exit` from killing your test process. Dependency injection solves this by making those dependencies explicit parameters rather than global calls. Go doesn't need a framework for it. Interfaces, structs, and explicit parameters are enough.

## Decoupling `main`

Move the core logic of `main` into a `run` function and collect all external dependencies into an `env` struct. Pass real dependencies from `main` and controlled substitutes in tests.

1. Declare a `run` function next to `main`.
2. Move the core logic into `run`. Return an error instead of calling `os.Exit`.
3. Create an `env` struct to hold external dependencies: writers, args, and any other I/O.
4. Pass `env` to `run`.
5. In `main`, construct `env` with real OS values and pass it to `run`.

This confines all side effects to `main`. `run` never reads from `os.Args`, writes to `os.Stdout`, or calls `os.Exit` directly.

### Before

This `main` is tightly coupled to global dependencies:

1. `parseArgs` reads directly from `os.Args`.
2. `os.Exit` terminates the process on error.
3. `fmt.Printf` writes output directly to stdout.

```go
func main() {
	c := config{
		n: 100,
		c: 1,
	}

	if err := parseArgs(&c, os.Args[1:]); err != nil {
		os.Exit(1)
	}

	fmt.Printf(
		"%s\n\nSending %d requests to %q (concurrency: %d)\n",
		logo, c.n, c.url, c.c)
}
```

None of these can be redirected or substituted in a test.

### After

First, create an `env` struct to hold the external dependencies:

1. `stdout` accepts output so tests can capture it instead of it going to the terminal.
2. `stderr` accepts error output for the same reason.
3. `args` accepts a string slice instead of reading `os.Args` directly.

```go
type env struct {
	stdout io.Writer    // 1
	stderr io.Writer    // 2
	args   []string     // 3
	dryRun bool
}
```

Next, move the logic from `main` into `run`. It accepts `*env` and returns an error instead of calling `os.Exit`:

```go
func run(e *env) error {
	c := config{
		n: 100,
		c: 1,
	}

	if err := parseArgs(&c, e.args[1:], e.stderr); err != nil {
		return err
	}

	fmt.Fprintf(
		e.stdout,
		"%s\n\nSending %d requests to %q (concurrency: %d)\n",
		logo, c.n, c.url, c.c,
	)
	return nil
}
```

`main` constructs `env` with real OS values and passes it to `run`:

```go
func main() {
	if err := run(&env{
		stdout: os.Stdout,
		stderr: os.Stderr,
		args:   os.Args,
	}); err != nil {
		os.Exit(1)
	}
}
```

Pass `env.stderr` to `parseArgs` so flag errors go to the injected writer, not directly to the terminal:

```go
func parseArgs(c *config, args []string, stderr io.Writer) error {
	fs := flag.NewFlagSet("hit", flag.ContinueOnError)
	fs.SetOutput(stderr)
	fs.Usage = func() {
		fmt.Fprintf(fs.Output(), "usage: %s [options] url\n", fs.Name())
		fs.PrintDefaults()
	}
	// ...
}
```

### Testing

Pass controlled substitutes to `run` in place of the real OS values. `bytes.Buffer` implements `io.Writer`, so it captures anything written to `stdout` or `stderr`:

```go
var out bytes.Buffer
var errOut bytes.Buffer

e := &env{
	stdout: &out,
	stderr: &errOut,
	args:   []string{"cmd", "-n", "10", "https://example.com"},
}

err := run(e)
```

After `run` returns, inspect `out.String()` and `errOut.String()` to assert on what was written. The test process is never at risk from `os.Exit`.


