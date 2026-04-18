+++
title = 'Testing CLI tools'
linkTitle = "Testing"
date = '2026-02-17T23:08:26-05:00'
weight = 50
draft = false
+++

When you test a CLI tool, you test that the flags accept the correct values.

## Production environment

The following example tests a CLI tool that uses an `env` struct to inject dependencies to a `run` function, which is then passed to `main`. Here is the dependency struct:

```go
type env struct {
    args   []string
	stdout io.Writer
	stderr io.Writer
	dryRun bool
}
```

## Test environment

The tests need to pass test `args`, and then capture the output of both `stdout` and `stderr`. To capture the output, you can create a `testenv` struct that uses `strings.Builder` for `stdout` and `stderr`.

`strings.Builder` satisfies the `io.Writer` interface, and it has a `String()` method that lets us read what is written to its buffer.

```go
type testEnv struct {
	stdout strings.Builder
	stderr strings.Builder
}
```
## Helper function

Next, create a helper function that accepts a variable number of arguments, initializes the remaining `env` fields, and returns a `testEnv`:
1. Accept variadic string input so you can pass test flags and arguments.
2. Call `run`, which takes a pointer to an `env` struct. Initialize the `env` fields with `testEnv` values
3. `args` takes a slice of strings, and the first index in the slice is set to `hit`. The remaining value is the variadic `args` passed to `testRun`.
4. Inject the `string.Builder` Writers in `testEnv` into the `stdout` and `stderr` fields.
5. Set `dryRun` to true to prevent live HTTP calls.
6. Return `testEnv` so you can inspect the values in your tests.

```go
func testRun(args ...string) (*testEnv, error) {        // 1
	var tenv testEnv
	err := run(&env{                                    // 2
		args:   append([]string{"hit"}, args...),       // 3
		stdout: &tenv.stdout,                           // 4
		stderr: &tenv.stderr,
		dryRun: true,                                   // 5
	})
	return &tenv, err                                   // 6
}
```

## CLI tool tests

Run your tests. The first test ensures that the tool processes valid input correctly:
1. Get the `testEnv` values. This line is appends the URL to the `env.args`.
2. Assert that something was written to `stdout`.
3. Assert that nothing was written to `stderr`.

```go
func TestRunValidInput(t *testing.T) {
	t.Parallel()

	tenv, err := testRun("https://github.com/username")         // 1
	if err != nil {
		t.Fatalf("got %q;\nwant nil err", err)
	}
	if n := tenv.stdout.Len(); n == 0 {                         // 2
		t.Errorf("stdout = 0 bytes; want > 0")
	}
	if n := tenv.stderr.Len(); n != 0 {                         // 3
		t.Errorf(
			"stderr = %d bytes; want 0; stderr:\n%s",
			n, tenv.stderr.String(),
		)
	}
}
```
Next, test that the tool returns an error with invalid input:
1. Get the `testEnv` values. This line is appends invalid arguments to `env.args`.
2. When `testRun` calls `run`, it should return an error. This checks that an error was returned.
3. There should be output in `testEnv.stderr`.

```go
func TestRunInvalidInput(t *testing.T) {
	t.Parallel()

	tenv, err := testRun("-c=2", "-n=1", "invalid-url")     // 1

	if err == nil {                                         // 2
		t.Fatalf("got nil; want err")
	}
	if n := tenv.stderr.Len(); n == 0 {                     // 3
		t.Error("stderr = 0 bytes; want > 0")
	}
}
```

## Unit testing

Unit tests confirm that each component functions correctly in isolation. The following tests validate that a program can parse flags.


### Setup

These tests validate the CLI tool configuration, which is set with `parseArgs`. This function mutates a configuration struct, takes a string of arguments, and logs error messages to a Writer:

```go
func parseArgs(c *config, args []string, stderr io.Writer) error
```

To test this, we create a `parseArgsTest` struct to model the inputs and expected outputs:
1. Name for the test case.
2. Arguments that populate the config.
3. Expected output of the test, a mutated configuration object 

```go
type parseArgsTest struct {
	name string         // 1
	args []string       // 2
	want config         // 3
}
```

### Table tests

Test the configuration with table tests that run in parallel. These tests follow the same patterns described in [Unit testing](../unit-testing/#table-driven-tests), with the addition of `io.Discard`. Use this when you do not care about capturing the error output with `os.Stderr` or `strings.Builder`. `io.Discard` throws away the bytes passed to the Writer.

```go
func TestParseArgsValidInput(t *testing.T) {
	t.Parallel()

	for _, tt := range []parseArgsTest{
		{
			name: "all_flags",
			args: []string{"-n=10", "-c=5", "-rps=5", "http://test"},
			want: config{n: 10, c: 5, rps: 5, url: "http://test"},
		},
	} {
		t.Run(tt.name, func(t *testing.T) {
			t.Parallel()

			var got config
			if err := parseArgs(&got, tt.args, io.Discard); err != nil {
				t.Fatalf("parseArgs() error = %v, want no error", err)
			}
			if got != tt.want {
				t.Errorf("flags = %+v, want %+v", got, tt.want)
			}
		})
	}
}

func TestParseArgsInvalidInput(t *testing.T) {
	t.Parallel()

	for _, tt := range []parseArgsTest{
		{name: "n_syntax", args: []string{"-n=ONE", "http://test"}},
		{name: "n_zero", args: []string{"-n=0", "http://test"}},
		{name: "n_negative", args: []string{"-n=-1", "http://test"}},
	} {
		t.Run(tt.name, func(t *testing.T) {
			t.Parallel()

			err := parseArgs(&config{}, tt.args, io.Discard)
			if err == nil {
				t.Fatal("parseArgs() = nil, want error")
			}
		})
	}
}
```