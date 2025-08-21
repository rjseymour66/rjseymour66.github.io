---
title: "Testing"
weight: 60
description: >
  Working with Testing in Go.
---

Place both internal and external tests in the same folder as the code they test.
- _Internal tests_ verify code from the same package. They are called 'white-box tests' and can test exported and unexported identifiers.
- _External tests_ verify code from another package. They are called 'black-box tests'. External tests use the `_test` suffix in the package name. For example, `package url_test`.

## Package naming

Place `*_test.go` files in the same directory as the code that you are testing. For external (integration) tests, use the original package name followed by `_test`. For example:

```go
package original_test
```
This package format requires that you import the source into the test file.

## Methods

The most useful are `t.Errorf()` and `t.Fatalf()`. The following table describes all available `t.*` test methods:

| Method        | Description                                              |
| ------------- | :------------------------------------------------------- |
| `t.Log()`     | Log a message.                                           |
| `t.Logf()`    | Log a formatted message.                                 |
| `t.Fail()`    | Mark the test as failed, but continue test execution.    |
| `t.FailNow()` | Mark the test as failed, and immediately stop execution. |
| `t.Error()`   | Combination of `Log()` and `Fail()`.                     |
| `t.Errorf()`  | Combination of `Logf()` and `Fail()`.                    |
| `t.Fatal()`   | Combination of `Log()` and `FailNow()`.                  |
| `t.Fatalf()`  | Combination of `Logf()` and `FailNow()`.                 |


## Write a test

Each test function must use the `TestXxx(t *testing.T)` signature, or the Go compiler does not recognize and run the test. For example, the following test verifies the functionality of a simple `Add(a, b int) int` function:

```go
func TestAdd(t *testing.T) {
    ...
}
```

Each test contain three main sections:

Arrange
: Set up the test inputs and expected values.

Act
: Execute the portion of code that you are testing.

Assert
: Verify that the code returned the correct values. You can use the `got`/`want` or `got`/`expected` formatting.

### Arrange

A common way to arrange a test is to use table tests. Table tests are a way to provide multiple test cases that you loop over and test during the `Act` stage. To set up a table test, complete the following:
1. Create a `testCase` struct that models the inputs and expected outputs of the test:
   ```go
   type testCase struct {
	   a        int
	   b        int
	   expected int
   }
   ```
2. Use a map literal with a `string` key and `testCase` value. The `string` key is the name of the test, and `testCase` is the test values:
   ```go
   tt := map[string]testCase{   // tt for table tests
        "test one": {
            a:        4,
            b:        5,
            expected: 9,
        },
        "test two": {
            a:        -4,
            b:        15,
            expected: 11,
        },
        "test three": {
            a:        5,
            b:        1,
            expected: 6,
        },
   }
   ```
### Act

Within the same `TestAdd()` function, write a `for range` loop. This is where you execute each test case with the code that you are testing. Use the `t.Run()` subtest method in the `for range` loop to run each individual test case with a name. `t.Run()` accepts two parameters: the name of the test, and an unnamed test function:

```go
for name, tc := range tt {
	t.Run(name, func(t *testing.T) {
		// act
		got := add(tc.a, tc.b)
		...
	})
}
```
In the previous example, `name` is the key in the `tt` map, and `tc` is the `testCase` struct in the `tt` map.

### Assert

In the assert step, you compare the actual values (what you got in the Act step) with the expected value, which is usually a field in the `testCase` struct. Asserts are generally `if` statements that return a formatted error with `t.Errorf` when the `got` and `expected` values do not match:

```go
for name, tc := range tt {
	t.Run(name, func(t *testing.T) {
        ...
		// assert
		if got != tc.expected {
			t.Errorf("expected %d, got %d", tc.expected, got)
		}
	})
}
```
#### Assert helper

Writing assert logic can get tedious, so you should extract it into a helper function. Add the following code to `/internal/assert/assert.go`:

```go
package assert

import (
    "testing"
)

func Equal[T comparable](t *testing.T, got, expected T) {
    t.Helper()

    if got != expected {
        t.Errorf("got: %v; want: %v", got, expected)
    }
}
```

The preceding example uses [generics](https://go.dev/doc/tutorial/generics).

Now, you can use this helper function to verify test output during the assert stage:

```go
for name, tc := range tt {
	t.Run(name, func(t *testing.T) {
        ...
		// assert
		assert.Equal(t, got, tc.expected)
	})
}
```

## Test flags

### shuffle

Add the `-shuffle=on` flag to execute tests in a random order:

```shell
$ go test -v -shuffle=on
```

### failfast

Stops tests in a single package if there is a failing test. This is helpful if you want to work on the first failing test:

```go
$ go test -v -failfast
```
### run

Provide the name of a specific test that you want to run:

```go
// run TestName
$ go test -v -run=TestName

// run a specific subtest
$ go test -v -run=TestName/^with_port

// run a specific subtest
$ go test -v -run=TestName/with_port
```
### short

Use the short flag to skip long-running tests, like integration tests. To designate a test as a short test, add the following snippet at the beginning of the test function:

```go
func TestFunction(t *testing.T) {
    // Skip the test if the "-short" flag is provided when running the test.
    if testing.Short() {
        t.Skip("models: skipping integration test")
    }

    ...
}
```

When you run tests, include the `-short` flag to skip any tests that include the `testing.Short()` function:

```shell
$ go test -v -short ./...
```

## Unit tests 

In Go, a _unit_ is a package, and a _unit test_ verifies the behavior of a single package. Unit tests test simple pieces of code, such as functions or methods.

### Table-driven tests

Also called data-driven and parameterized tests. They verify code with varying inputs. You can also implement subtests that run tests in isolation.

Imagine table-driven tests as actual tables, where the headers are struct fields, and the rows become individual slices in the test cases:

| product    | rating | price |
| :--------- | ------ | ----- |
| prod one   | 5      | 20    |
| prod two   | 10     | 30    |
| prod three | 15     | 40    |

You can represent this in a test as follows:

```go
func TestTable(t *testing.T) {
    type product struct {
        product string
        rating  int
        price   float64
    }
    testCases := []product {
        {"prod one", 5, 20},
        {"prod two", 10, 30},
        {"prod three", 15, 40},
    }
}
```

Alternatively, you can use a map with an anonymous struct:

```go
tt := map[string]struct {
    rating int
    price  float64
}{
    "prod one":   {5, 20},
    "prod two":   {10, 30},
    "prod three": {15, 40},
}

for _, tt := ...
```

Reuse assertion logic and naming makes each test identifiable.
`t.Run()` defines a subtest. It accepts the name of the test, and then a testing function:
```go
for _, tt := range testCases {
    t.Run(tt.name, func (t *testing.T){
        ...
    })
}
```

### Subtests 

Subtests let you run different testing scenarios for a testing context.

Use `t.Run()` to run subtests. Subtests run in isolation, so you can use `t.Fatal[f]()` and not stop test execution. In addition, you can run subtests [in parallel](#parallel-testing).

```go
for name, tc := range tt {
	t.Run(name, func(t *testing.T) {
        ...
		// assert
		if got != tc.expected {
			t.Errorf("expected %d, got %d", tc.expected, got)
		}
	})
}
```

### Parallel testing 

Tests that have `t.Parallel()` on the first line of the `t.Run()` function.

Go pauses all tests with `t.Parallel()` and then runs them when other tests complete. `GOMAXPROCS` determines how many tests run in parallel. By default, `GOMAXPROCS` is set to the number of CPUs on the machine. You can override this with the `-parallel` flag:

```shell
$ go test -parallel=4 ./...
```

Tests marked with `t.Parallel()` are run in parallel with _and only with_ other parallel tests.

### Skipping tests 

Use `t.Skip()` to skip tests during execution. Use this with `testing.Short()` and the `-test.short` argument to tell the testing package to skip specific tests.

This helps separate unit tests from integration tests. Integration tests take longer than unit tests, so you can designate a test as a unit test as follows:
```go
func TestIntegrationTest(t *testing.T)  {
    if testing.Short() {
        t.Skip("Skip integration tests")
    }
    // continue test ...
}
```

To skip this test, use `-test.short` when you execute the tests:
```shell 
$ go test -v -test.short
```

### Cleanup 

Use `t.Cleanup()` to clean up testing resources. The `Cleanup()` method is available to helper and testing methods, so it cancels after they are complete. The standard `defer` function operator executes a function before it completes, even if it is a helper function.

The following is a simple example of a function that uses `Cleanup`:

```go
func TestWithResources(t *testing.T)  {
    // testing...
    t.Cleanup(func() {
        // cleanup logic
    })
    // continue test ...
}
```

### Helpers

A test helper can improve readability of your tests. Unfortunately, when a test fails within the test helper, the testing package logs helper function line number after failure--this makes it difficult to pinpoint where the test failed.

Include the `t.Helper()` function to designate a function as a helper function. The testing package ignores the helper function line number and instead logs the line number of the failing test.

Here is an example of a reusable assert statement that you could include at the end of the test or subtest:

```go
// Helper
func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}

	// optionally prints function input:
	if got != want {
		t.Errorf("got %q want %q given, %v", got, want, numbers)
	}
}

// Helper in the test
func TestWithAssert(t *testing.T) {
	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Chris")
		want := "Hello, Chris"

		assertCorrectMessage(t, got, want)
	})
	t.Run("say 'Hello, World' when an empty string is supplied", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"

		assertCorrectMessage(t, got, want)
	})
}
```

Helper functions accept an instance of the `testing.T` type, so make sure you pass the `t` testing instance to the helper in the `TestXxx` function. This provides the helper with access to the testing instance as the rest of the test function.

To benchmark helper functions, include the `testing.TB` type so you an access the benchmarking package:

```go
func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

### Temporary directories

The testing type provides the `TempDir()` function to create a temporary directory that is deleted automatically when testing completes. This means that you do not have to write cleanup logic:

```go 
func TestTempDir(t *testing.T)  {
    tmpDir := t.TempDir()
    
    // testing logic 
}
```

## Integration tests

Integration tests test how the program behaves when interacted with from the outside world--how a user uses it. This means that you test the `main()` method.

In Go, you test the main method with the `TestMain()` function so you can set up and tear down resources more easily. For example, you might need to create a temporary file or build and execute a binary. You do not want to keep these artifacts in the program after testing.

Follow these general guidelines when running integration tests:
1. Check the machine with `runtime.GOOS`
2. Create the build command with `exec.Command()`, then use `.Run()` to execute that command. Check for errors
3. Run the tests with `m.Run()`
4. Clean up any artifacts with `os.Remove(artifactname)`

## Testing the main method

The `main` method uses globals, which you cannot easily test. The trick is to put the logic into the `run()` function. You can inject the globals to the `run` function, and you can mimic those values during tests.

The flag package uses `NewFlagSet` function to create the default flag set, `CommandLine`.

## Testable examples

[Blog post](https://go.dev/blog/examples)

A _testable example_ is live documentation for code. You write a testable example to demonstrate the package API to other developers. The API includes the exported identifiers, such as functions, methods, etc. A testable example never goes out of date.

The testing package runs testable examples and checks their results, but it does not report successes or failures:

```go
func ExampleURL_fields() {
	u, err := url.Parse("https://foo.com/go")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(u.Scheme)
	fmt.Println(u.Host)
	fmt.Println(u.Path)
	fmt.Println(u)
	// Output:
	// https
	// foo.com
	// go
	// https://foo.com/go
}
```

### Naming conventions

Testable examples use the following naming conventions:

| Signature                    | Description                                          |
| :--------------------------- | :--------------------------------------------------- |
| `func Example()`             | Example for the entire package.                      |
| `func ExampleParse()`        | Example for the `Parse` function.                    |
| `func ExampleURL()`          | Example for the `URL` type.                          |
| `func ExampleURL_Hostname()` | Example for the `Hostname` method on the `URL` type. |


### godoc server

You can generate docs that include your [testable examples](#testable-examples) with `godoc`. The following command installs the latest version:

```shell
$ go install golang.org/x/tools/cmd/godoc@latest
```

To view any `ExampleXxx` functions as Go documentation, run the `go doc` server with the following command:
```shell
$ godoc -play -http ":6060"
```
To show additional examples of the same type, use the `_xxx()` suffix on the function name. For example:
```go
func ExampleURL(){...}
func ExampleURL_fields(){...}
```


## Test unexported functions

If you want to test an unexported function from an external test, you must export the function from the external test package. For example, the `parseScheme` function is unexported:

```go
// url.go
package url

func parseScheme() {...}
```

Create a new file in the same package that assigns the unexported function to an exported function:

```go
// export_test.go
package url

var ParseScheme = parseScheme
```

Finally, test the function in an external test file:

```go
// parse_scheme_test.go
package url_test

func TestParseScheme(t *testing.T) {...}
```

## Coverage tests 

Go can generate a test report that details what code is sufficiently tested. In its simplest form, you can display the percentage of tested code with the `cover` flag:

```shell 
go test -cover
```

View how much of your code is covered by tests:
```shell
$ go test -coverprofile cover.out
$ go test -coverprofile=coverage_output_filename
```
The coverage output file is optional. By convention, the coverage profile file is named `cover.out`.

Use a coverage output file to view test coverage by function:
```shell
$ go tool cover -func=cover.out
```

After you create the `coverprofile` file, use the `cover` go tool to generate HTML output to view what code is and is not covered. This command opens the coverage output in the browser:
```shell
$ go tool cover -html=cover.out
```

To create the HTML file, but not open it in the browser automatically:
```shell
$ go tool cover -html=cover.out -o coverage.html
```

By default, the testing package only tests packages with `*_test.go` files. To test all packages, use the `coverpkg` flag: 

```shell 
$ go test ./... -coverpkg=./...
```


The cover tool uses three colors to identify code coverage:
- grey: not tracked by the coverage tool
- green: sufficiently tested
- red: not covered by tests

To view test coverage by function, use the following commands:

```shell
$ go test -coverprofile=/tmp/profile.out ./...
$ go tool cover -func=/tmp/profile.out
```
> All test must pass or the preceding command fails.

## Benchmark tests 

Benchmark your methods to determine their efficiency. Benchmark functions use the `BenchmarkXxx(b *testing.B)` signature. Place them in the `x_test.go` file with the other test functions.

To write a benchmark function, call the method that you want to benchmark within a `for` loop in the `BenchmarkXxx` function. The `for` loop uses `b.N` as its upper bound. `b.N` adjusts the test runner to properly measure performance at runtime. In addition, you can run the `b.ReportAllocs()` function to see how many memory allocations your code makes.

For example, the following function benchmarks the `String()` method on the `URL` type:

```go
func BenchmarkURLString(b *testing.B) {
	b.ReportAllocs()
	b.Logf("Loop %d times\n", b.N)

	u := &URL{Scheme: "https", Host: "foo.com", Path: "go"}

	for i := 0; i < b.N; i++ {
		u.String()
	}
}
```
To run the test, use the `-bench` flag. Use a dot (`.`) to run every benchmark in the package:

```shell
$ go test -bench .
...
BenchmarkURLString-12    	 8868142	       153.8 ns/op	      64 B/op	       4 allocs/op
--- BENCH: BenchmarkURLString-12
    ...
PASS
ok  	url/url	1.506s
```

The `B/op` column indicates that there were 64 bytes allocated in each operation. The `allocs/op` value indicates the number of memory allocations that made by the code in the benchmark.

When you run benchmarks with the `-bench` flag, the regular tests run as well (use the `-v` flag to verify). If you want to run only the benchmark tests, use the `-run` flag with the `^$` regular expression:

```shell
$ go test -run=^$ -bench .
```

The `^$` regex tells the runner to ignore tests other than the benchmarks.

### Loops

The `testing.B` package gives you access to the `Loop()` function that benchmarks your `for` loops. Set it up like this:

```go
func Benchmark(b *testing.B) {
	//... setup ...
	for b.Loop() {
		//... code to measure ...
	}
	//... cleanup ...
}
```

Run it with the `-benchmem` flag to see how much memory your loop uses:

```bash
go test -bench=. -benchmem
```


### Sub-benchmarks

You can run sub-benchmarks, just as you can run subtests:

```go
func BenchmarkURLString(b *testing.B) {
    var benchmarks = []*URL{
        {Scheme: "https"},
        {Scheme: "https", Host: "foo.com"},
        {Scheme: "https", Host: "foo.com", Path: "go"},
    }
    for _, u := range benchmarks {
        b.Run(u.String(), func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                u.String()
            }
        })
    }
}
```

### Comparing benchmarks

1. Save the current benchmark result of the method:
    ```shell
    go test -bench . -count 10 > old.txt
    ```
    The `-count` flag runs the benchmark the number of times that you pass to it. There is no recommendation for the number you pass to `count`--`10` is a  random number.

2. Refactor your code.
3. Run the benchmarks again and compare with `benchstat`.
   First, install `benchstat`:
   ```shell
   $ go install golang.org/x/perf/cmd/benchstat@latest
   ```
   Next, compare the `old.txt` and `new.txt` files:
   ```shell
   $ benchstat old.txt new.txt 
   goos: linux
   goarch: amd64
   pkg: url/url
   cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
                │   old.txt    │               new.txt                │
                │    sec/op    │    sec/op     vs base                │
   URLString-12   138.85n ± 5%   99.70n ± 14%  -28.19% (p=0.000 n=10)

                │  old.txt   │              new.txt               │
                │    B/op    │    B/op     vs base                │
   URLString-12   64.00 ± 0%   56.00 ± 0%  -12.50% (p=0.000 n=10)

                │  old.txt   │              new.txt               │
                │ allocs/op  │ allocs/op   vs base                │
   URLString-12   4.000 ± 0%   3.000 ± 0%  -25.00% (p=0.000 n=10)
   ```

## Fuzz tests 

Fuzz tests use random input to detect bugs and edge cases. Fuzz tests require the following:
- Test name must start with `FuzzXxx`
- Accepts the `*testing.F` parameter
- Each function must define the initial value (seed corpus) with `f.Add()`
- There must be a Fuzz target

```go 
func FuzzHelloWorld(f *testing.F)  {
    f.Add(5)
    f.Fuzz(func(t *testing.T, a int)  {
        HelloWorld(a)
    })
}
```
## Build tags 

How do build tags work? [Digital Ocean](https://www.digitalocean.com/community/tutorials/customizing-go-binaries-with-build-tags)

Add build tags at the top of test files so you can specify whether or not you want to run them in your test commands. For example, the following build tag designates the file for go tests that use the `cli` tag:

```go
//go:build cli
```
To run tests in this file, run the following command:
```shell
$ go test -tags=cli
```
For additional details, see [Build constraints](https://pkg.go.dev/cmd/go#hdr-Build_constraints).


## Mocks, stubs, fakes (dependencies)

Mocks, stubs, and fakes are simple implemenatations of code that you need to test. The different terms are interchangeable--the important part is that the test dependency _exposes the same interface as the production object_.

These test dependencies should be in their own package. For example, `internal/models/mocks/file.go`. A mock is a hard-coded values that you can use in your test cases:

```go
func (m *UserModel) Insert(name, email, password string) error {
	switch email {
	case "dupe@example.com":
		return models.ErrDuplicateEmail
	default:
		return nil
	}
}

func (m *UserModel) Authenticate(email, password string) (int, error) {
	if email == "alice@example.com" && password == "pa$$word" {
		return 1, nil
	}

	return 0, models.ErrInvalidCredentials
}

func (m *UserModel) Exists(id int) (bool, error) {
	switch id {
	case 1:
		return true, nil
	default:
		return false, nil
	}
}
```


## Test data

Store any testing files in the `testdata` directory. Go tooling ignores this directory when building and compiling the program.

## Strategies

When testing file writes, use _goldenfiles_: files that contain the expected results and that you load during tests to validate output.

`iotest.TimeoutReader(r)` simulates a reading failure and returns a timeout error.

## Integration testing

If you are testing external commands that modify the state of an external resource, the testing conditions change after each test. That is why you should use mocks.

### Test db setup and teardown

First, you have to create a new test database. Login as the root user:

```shell
$ mysql...
```
Create the database and user, then grant the user privileges:

```sql
> CREATE DATABASE test_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
> CREATE USER 'test_web'@'localhost';
> GRANT CREATE, DROP, ALTER, INDEX, SELECT, INSERT, UPDATE, DELETE ON test_database.* TO 'test_web'@'localhost';
> ALTER USER 'test_web'@'localhost' IDENTIFIED BY 'pass';
```
Next, create a _setup script_ (setup.sql) and a _teardown script_ (teardown.sql). Create each file in the `internal/models/testdata/` directory.

> Go tooling ignores the `testdata` directory.

Next, create a `testutils_test.go` file in `/internal/models/`. This file includes code that performs the following:
- Creates a new database connection pool for the test db
- Executes the `setup.sql` script
- Executes the `teardown.sql` script from within a cleanup function
- Closes the connection pool


## HTTP

The primary tool for testing handlers and middleware is the [ResponseRecorder](https://pkg.go.dev/net/http/httptest#ResponseRecorder). ResponseRecorder behaves the same as a ResponseWriter, but it does not write responses to a connection, it records them. To use a ResponseRecorder, perform the following:
1. Create the ResponseRecorder object.
2. Pass the object to the handler.
3. Examine the object after the handler returns. Use the [`Result()`](https://pkg.go.dev/net/http/httptest#ResponseRecorder.Result) function.

### Handlers

To illustrate a handler test, the following function writes 'OK' as the response on an HTTP connection:

```go
func ping(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("OK"))
}
```

The following code uses the [assert helper](#assert-helper) to test a the `ping` function:

```go
func TestPing(t *testing.T) {
	// Initialize a new httptest.ResponseRecorder.
	rr := httptest.NewRecorder()

	// Initialize a new dummy http.Request.
	r, err := http.NewRequest(http.MethodGet, "/", nil)
	if err != nil {
		t.Fatal(err)
	}

	// Call the ping handler function, passing in the
	// httptest.ResponseRecorder and http.Request.
	ping(rr, r)

	// Call the Result() method on the http.ResponseRecorder to get the
	// http.Response generated by the ping handler.
	rs := rr.Result()

	// Check that the status code written by the ping handler was 200.
	assert.Equal(t, rs.StatusCode, http.StatusOK)

	// And we can check that the response body written by the ping handler
	// equals "OK".
	defer rs.Body.Close()
	body, err := io.ReadAll(rs.Body)
	if err != nil {
		t.Fatal(err)
	}
	bytes.TrimSpace(body)

	assert.Equal(t, string(body), "OK")
}
```

### Middleware

Middleware tests use the same general format as handler tests, but you have to create a mock HTTP handler that you can pass as the `next` value to the middleware (the `next` value is a parameter that represents the subsequent function in the middleware chain--every middleware calls the `ServeHTTP` method on the `next` argument).

For example, one middleware sets security headers on the response:

```go
func secureHeaders(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Content-Security-Policy", "default-src 'self'; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com")
		w.Header().Set("Referrer-Policy", "origin-when-cross-origin")
		w.Header().Set("X-Content-Type-Options", "nosniff")
		w.Header().Set("X-Frame-Options", "deny")
		w.Header().Set("X-XSS-Protection", "0")

		next.ServeHTTP(w, r)
	})
}
```

The following function tests the middleware:

```go
func TestSecureHeaders(t *testing.T) {
	// Initialize a new httptest.ResponseRecorder and dummy http.Request.
	rr := httptest.NewRecorder()

	r, err := http.NewRequest(http.MethodGet, "/", nil)
	if err != nil {
		t.Fatal(err)
	}

	// Create a mock HTTP handler that we can pass to our secureHeaders
	// middleware, which writes a 200 status code and an "OK" response body.
	next := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("OK"))
	})

	// Pass the mock HTTP handler to our secureHeaders middleware. Because
	// secureHeaders *returns* a http.Handler we can call its ServeHTTP()
	// method, passing in the http.ResponseRecorder and dummy http.Request to
	// execute it.
	secureHeaders(next).ServeHTTP(rr, r)

	// Call the Result() method on the http.ResponseRecorder to get the results
	// of the test.
	rs := rr.Result()

	// Check that the middleware has correctly set the Content-Security-Policy
	// header on the response.
	expectedValue := "default-src 'self'; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com"
	assert.Equal(t, rs.Header.Get("Content-Security-Policy"), expectedValue)

	// Check that the middleware has correctly set the Referrer-Policy
	// header on the response.
	expectedValue = "origin-when-cross-origin"
	assert.Equal(t, rs.Header.Get("Referrer-Policy"), expectedValue)

	// Check that the middleware has correctly set the X-Content-Type-Options
	// header on the response.
	expectedValue = "nosniff"
	assert.Equal(t, rs.Header.Get("X-Content-Type-Options"), expectedValue)

	// Check that the middleware has correctly set the X-Frame-Options header
	// on the response.
	expectedValue = "deny"
	assert.Equal(t, rs.Header.Get("X-Frame-Options"), expectedValue)

	// Check that the middleware has correctly set the X-XSS-Protection header
	// on the response
	expectedValue = "0"
	assert.Equal(t, rs.Header.Get("X-XSS-Protection"), expectedValue)

	// Check that the middleware has correctly called the next handler in line
	// and the response status code and body are as expected.
	assert.Equal(t, rs.StatusCode, http.StatusOK)

	defer rs.Body.Close()
	body, err := io.ReadAll(rs.Body)
	if err != nil {
		t.Fatal(err)
	}
	bytes.TrimSpace(body)

	assert.Equal(t, string(body), "OK")
}
```

## End-to-end tests

End-to-end (E2E) tests verify how your routing, middleware, and handlers work together as a single unit, rather than in isolation.

E2E test require the `httptest.New[TLS]Server` and test client to mimic a production environment. The following example demonstrates the following:
- Minimal application
- `httptest.NewTLSServer`.
- `ts.Client`, which is a method on the httptest package's `Server` type.
- Assert the status code and response body are correct

```go
func TestPingE2E(t *testing.T) {
	// Create a new instance of our application struct. For now, this just
	// contains a couple of mock loggers (which discard anything written to
	// them). The loggers are required by a few of our middlewares.
	app := &application{
		errorLog: log.New(io.Discard, "", 0),
		infoLog:  log.New(io.Discard, "", 0),
	}

	// We then use the httptest.NewTLSServer() function to create a new test
	// server, passing in the value returned by our app.routes() method as the
	// handler for the server. This starts up a HTTPS server which listens on a
	// randomly-chosen port of your local machine for the duration of the test.
	// Notice that we defer a call to ts.Close() so that the server is shutdown
	// when the test finishes.
	ts := httptest.NewTLSServer(app.routes())
	defer ts.Close()

	// The network address that the test server is listening on is contained in
	// the ts.URL field. We can  use this along with the ts.Client().Get() method
	// to make a GET /ping request against the test server. This returns a
	// http.Response struct containing the response.
	rs, err := ts.Client().Get(ts.URL + "/ping")
	if err != nil {
		t.Fatal(err)
	}

	// We can then check the value of the response status code and body using
	// the same pattern as before.
	assert.Equal(t, rs.StatusCode, http.StatusOK)

	defer rs.Body.Close()
	body, err := io.ReadAll(rs.Body)
	if err != nil {
		t.Fatal(err)
	}
	bytes.TrimSpace(body)

	assert.Equal(t, string(body), "OK")
}
```

The following E2E test checks whether a handler returns the correct response. First, the mock:

```go
var mockSnippet = &models.Snippet{
	ID:      1,
	Title:   "An old silent pond",
	Content: "An old silent pond...",
	Created: time.Now(),
	Expires: time.Now(),
}
```

Next, the handler test:

```go
func TestSnippetView(t *testing.T) {
	// Create a new instance of our application struct which uses the mocked
	// dependencies.
	app := newTestApplication(t)

	// Establish a new test server for running end-to-end tests.
	ts := newTestServer(t, app.routes())
	defer ts.Close()

	// Set up some table-driven tests to check the responses sent by our
	// application for different URLs.
	tests := []struct {
		name     string
		urlPath  string
		wantCode int
		wantBody string
	}{
		{
			name:     "Valid ID",
			urlPath:  "/snippet/view/1",
			wantCode: http.StatusOK,
			wantBody: "An old silent pond...",
		},
		{
			name:     "Non-existent ID",
			urlPath:  "/snippet/view/2",
			wantCode: http.StatusNotFound,
		},
		{
			name:     "Negative ID",
			urlPath:  "/snippet/view/-1",
			wantCode: http.StatusNotFound,
		},
		{
			name:     "Decimal ID",
			urlPath:  "/snippet/view/1.23",
			wantCode: http.StatusNotFound,
		},
		{
			name:     "String ID",
			urlPath:  "/snippet/view/foo",
			wantCode: http.StatusNotFound,
		},
		{
			name:     "Empty ID",
			urlPath:  "/snippet/view/",
			wantCode: http.StatusNotFound,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			code, _, body := ts.get(t, tt.urlPath)

			assert.Equal(t, code, tt.wantCode)

			if tt.wantBody != "" {
				assert.StringContains(t, body, tt.wantBody)
			}
		})
	}
}
```

### Testing utilities

E2E tests require lots of boilerplate code--spinning up test servers, making requests, etc. You can abstract much of that code away to a `testutils_test.go` file.

If you plan to use the test utilities across multiple packages, place them in the `/internal` directory. Otherwise, place them in the same directory as the code they help test.

In the previous section, we created the `TestPingE2E` function to test the `ping` handler from end to end. We can create a new `testutils_test.go` file that creates the following test components:
- Application
- Server
- Client request

For example:

```go
// Create a newTestApplication helper which returns an instance of our
// application struct containing mocked dependencies.
func newTestApplication(t *testing.T) *application {
	return &application{
		errorLog: log.New(io.Discard, "", 0),
		infoLog:  log.New(io.Discard, "", 0),
	}
}

// Define a custom testServer type which embeds a httptest.Server instance.
type testServer struct {
	*httptest.Server
}

// Create a newTestServer helper which initalizes and returns a new instance
// of our custom testServer type.
func newTestServer(t *testing.T, h http.Handler) *testServer {
    // Initialize the test server as normal.
    ts := httptest.NewTLSServer(h)

    // Initialize a new cookie jar.
    jar, err := cookiejar.New(nil)
    if err != nil {
        t.Fatal(err)
    }

    // Add the cookie jar to the test server client. Any response cookies will
    // now be stored and sent with subsequent requests when using this client.
    ts.Client().Jar = jar

    // Disable redirect-following for the test server client by setting a custom
    // CheckRedirect function. This function will be called whenever a 3xx
    // response is received by the client, and by always returning a
    // http.ErrUseLastResponse error it forces the client to immediately return
    // the received response.
    ts.Client().CheckRedirect = func(req *http.Request, via []*http.Request) error {
        return http.ErrUseLastResponse
    }

    return &testServer{ts}
}

// Implement a get() method on our custom testServer type. This makes a GET
// request to a given url path using the test server client, and returns the
// response status code, headers and body.
func (ts *testServer) get(t *testing.T, urlPath string) (int, http.Header, string) {
	rs, err := ts.Client().Get(ts.URL + urlPath)
	if err != nil {
		t.Fatal(err)
	}

	defer rs.Body.Close()
	body, err := io.ReadAll(rs.Body)
	if err != nil {
		t.Fatal(err)
	}
	bytes.TrimSpace(body)

	return rs.StatusCode, rs.Header, string(body)
}
```

> Note how the `newTestServer` function handles cookies. Cookies are stored for each HTTP response and sent back to the test server for any responses. Additionally, the client never follows redirects so that hit always returns the first HTTP response sent by our server for the request.

Now, you can simplify the `TestPingE2E` function considerably:

```go
func TestPingE2E(t *testing.T) {
    app := newTestApplication(t)

    ts := newTestServer(t, app.routes())
    defer ts.Close()

    code, _, body := ts.get(t, "/ping")

    assert.Equal(t, code, http.StatusOK)
    assert.Equal(t, body, "OK")
}
```

## Testing HTML

None of this worked. Try it later as you write the code, instead of at the very end.

## select

`select` synchronizes processes. You wait on multiple channels--the first channel that sends a value "wins", and its code is executed. Here, we are trying to figure out which process finishes first:

```go
func Racer(a, b string) (winner string, error error) {
    select {
    case <-ping(a):
        return a, nil
    case <-ping(b):
        return b, nil
    case <-time.After(10 * time.Second):
        return "", fmt.Errorf("timed out waiting for %s and %s", a, b)
    }
}

func ping(url string) chan struct{} {
    ch := make(chan struct{})
    go func() {
        http.Get(url)
        close(ch)
    }()
    return ch
}
```

The `select` statement waits on two channels with a blocking call (waiting on a value). When the channel points to a value or a `case` statement (`val := <- ch` or `case <-func(x)`), you are waiting on a value to pass to the channel.

The `ping` function uses a `chan struct{}`. The empty struct is the data type that uses the least amount of memory, so we use that for the signal. This works because we are just closing the `chan` when a value is received--we are not sending anything.

Always use `make` to create a chan. Using `var` initiates the variable as the type's zero value. This is `nil` for a `chan`, and if you send a `nil` value into a channel it blocks forever.

### time.After

`time.After` is great add a timeout and make sure you don't write code that blocks forever. This function returns a `chan Time` value when the Duration argument has elapsed. In the preceding example, the `select` statement returns an error if more than 10s elapses from the time the function executes.

### time package

You can measure how long an HTTP call (or any other process) takes to finish with the `time` package:

```go
startA := time.Now()
http.Get(a)
aDuration := time.Since(startA) // returns a time.Duration value
```

### httptest servers

When you want to test an HTTP service, use the `httptest.NewServer` method. It will spin up a test server on an open port so you can test your HTTP code.

Remember to close the server right after you start it so it stops listening on any other port and cleans up any other resources.

```go
func TestRacer(t *testing.T) {

    slowServer := makeDelayedServer(20 * time.Millisecond)
    fastServer := makeDelayedServer(0 * time.Millisecond)

    defer slowServer.Close()
    defer fastServer.Close()

    slowURL := slowServer.URL
    fastURL := fastServer.URL

    want := fastURL
    got := Racer(slowURL, fastURL)

    if got != want {
        t.Errorf("got %q, want %q", got, want)
    }
}

func makeDelayedServer(delay time.Duration) *httptest.Server {
    return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        time.Sleep(delay)
        w.WriteHeader(http.StatusOK)
    }))
}
```

## Configurable functions

If you want to create a function and a configurable version of that function:

- Create a `var` that holds the value you want to configure
- Create the basic function, which does not accept an argument of the var type. It should returns a instance of the configurable function that DOES accept the `var`. You don't have to add logic to this function, it just decorates the configurable function.
- Create a configurable function that accepts a value of the var type.

```go
var tenSecondTimout = 10 * time.Second

func Racer(a, b string) (winner string, error error) {
    return ConfigurableRacer(a, b, tenSecondTimout)
}

func ConfigurableRacer(a, b string, timeout time.Duration) (winner string, error error) {
    select {
    case <-ping(a):
        return a, nil
    case <-ping(b):
        return b, nil
    case <-time.After(timeout):
        return "", fmt.Errorf("timed out waiting for %s and %s", a, b)
    }
}
```

## Reflection

Reflection is the ability of a program to examine its own structure, particularly through types.

Go blog: [The Laws of Reflection](https://go.dev/blog/laws-of-reflection)

In some scenarios, you want to write a function where you don't know the type at compile time.

## Sync

Sync lets you define a waitgroup that waits for a specified number of goroutines to finish. The main goroutine sets the number of goroutines to wait for with `Add`. When each goroutine is done, it calls `Done` to tell the wait group it is done executing. When all goroutines finish and call `Done`, the code continues executing.

In the meantime, the waitgroup calls `Wait` and blocks until all goroutines are complete:

```go
var wg sync.WaitGroup
wg.Add(wantedCount)

for i := 0; i < wantedCount; i++ {
    go func() {
        counter.Inc()
        wg.Done()
    }()
}
wg.Wait()
```

### Mutexes

A mutex is a "mutual exclusion" lock. It locks your values so that only one goroutine or process can access the value at a time. Make sure that you always provide a name to the mutex, or you might expose the `Lock` and `Unlock` methods in the public API. This is because embedding types make the type's methods part of the public interface.

Here is a struct with a mutex, and a simple implementation of the mutex:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func NewCounter() *Counter {        // constructor so users know not to initialize a Counter any other way
    return &Counter{}
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}
```

> We ran `go vet`, and it explained that our tests copied the mutex—it copied its value rather than mutate the value at an address. This is an issue, so we created a constructor that returns a pointer to a Counter struct.

### go vet

Use `go vet` in build scripts so you can catch subtle bugs.

### channels vs mutexes

- Channels when passing ownership of data
- Mutexes for managing state

| Situation                                                    | Use                                                         |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| Communicating between goroutines with no shared memory       | ✅ **Channels**                                              |
| Managing shared in-memory data with concurrent access        | ✅ **Mutexes**                                               |
| Want to coordinate work without giving direct access to data | ✅ **Channels**                                              |
| Performance-critical shared data structure                   | ✅ **Mutexes** (more efficient than channels in tight loops) |


## Repetition in loops

If you find yourself repeating yourself in loops, that probably means there is an abstraction in the loop, particularly if there is a `break` statement.

Think about ways that you can remove a `for` loop and maybe make it a `switch` statement:

```go
// original
for i := arabic; i > 0; i-- {
    if i == 5 {
        result.WriteString("V")
        break
    }
    if i == 4 {
        result.WriteString("IV")
        break
    }
    result.WriteString("I")
}

// refactored
for arabic > 0 {
    switch {
    case arabic > 4:
        result.WriteString("V")
        arabic -= 5
    case arabic > 3:
        result.WriteString("IV")
        arabic -= 4
    default:
        result.WriteString("I")
        arabic--
    }
}
```

## Switch statements

When you have extensive `switch` statements, that might mean you are capturing data or behavior in imperative code when it should be captured in a struct.

For example:

```go
// original
func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for arabic > 0 {
        switch {
        case arabic > 4:
            result.WriteString("V")
            arabic -= 5
        case arabic > 3:
            result.WriteString("IV")
            arabic -= 4
        default:
            result.WriteString("I")
            arabic--
        }
    }

    return result.String()
}

// refactored
type RomanNumeral struct {
    Value  int
    Symbol string
}

var allRomanNumerals = []RomanNumeral{
    {50, "L"},
    {10, "X"},
    {9, "IX"},
    {5, "V"},
    {4, "IV"},
    {1, "I"},
}

func ConvertToRoman(arabic int) string {

    var result strings.Builder

    for _, numeral := range allRomanNumerals {
        for arabic >= numeral.Value {
            result.WriteString(numeral.Symbol)
            arabic -= numeral.Value
        }
    }

    return result.String()
}
```

The refactored code declares rules about the numerals as data, rather than keeping it in an algorithm. Loop through the available Roman number values, and if `arabic` is greater or equal than a Roman numeral arabic value, write the corresponding `Symbol` to the string builder.

## Property-based tests

Property-based tests evaluate the rules of your domain. For example, if we are building a tool that converts Roman numerals to Arabic, we have these rules:

- Can't have more than 3 consecutive symbols
- Only I (1), X (10) and C (100) can be "subtractors"
- Taking the result of ConvertToRoman(N) and passing it to ConvertToArabic should return us N

These are rules that help define our _domain_. Property-based tests test these rules and make sure that our code adheres to them.

## Quick testing library

[Quick testing library](https://pkg.go.dev/testing/quick)

Here is a quick example:

```go
func TestPropertiesOfConversion(t *testing.T) {
    assertion := func(arabic uint16) bool {
        if arabic > 3999 {
            t.Log("testing", arabic)
        }
        roman := ConvertToRoman(arabic)
        fromRoman := ConvertToArabic(roman)
        return fromRoman == arabic
    }

    if err := quick.Check(assertion, &quick.Config{
        MaxCount: 1000,
    }); err != nil {
        t.Error("failed checks", err)
    }
}
```

## General

When writing tests:

- Write the test how you want to use the code from a consumer's point of view.
- Don't think about implementation. Focus on the what and why, but not the how.

[Sliming](https://deniseyu.github.io/leveling-up-tdd/) is when you test the structure of the code--the interface. You don't need to test the logic. Sliming is usually the first iteration where your tests pass, but they contain no logic. For example, your first test fails because you haven't defined a function or object. Your next step passes, but it doesn't solve your problem. Its a skeleton for your code. Here is an example:

```go
func NewPostsFromFS(fileSystem fs.FS) []Post {
    return []Post{{}, {}}
}
```

### Function arguments

To loosen coupling, always think about your function arguments. What functionality do you need? Can you replace it with an interface?

## Reading files

Suggested reading:

- [A Tour of Go 1.16's io/fs package](https://benjamincongdon.me/blog/2021/01/21/A-Tour-of-Go-116s-iofs-package/)
- [Discussion on GitHub](https://github.com/golang/go/issues/41190)

For tests, use [fstest](https://pkg.go.dev/testing/fstest) for file system interactions.

To mimic a filesystem, use `.MapFS`. It simulates an fs as a map, so you don't have to save files on disk. Us this where a function accepts a filesystem (`fs.FS`), such as `fs.ReadFile(fs, "filename.md")` so you don't have to rely on disk IO. It has the following definition, where `string` is the filepath and `MapFile` is a data structure that holds file content and metadata:

```go
type MapFS map[string]*MapFile


type MapFile struct {
    Data    []byte      // file content
    Mode    fs.FileMode // fs.FileInfo.Mode
    ModTime time.Time   // fs.FileInfo.ModTime
    Sys     any         // fs.FileInfo.Sys
}
```

Here is an example implementation, where the key is the filename, and the value defines the file contents:

```go
fs := fstest.MapFS{
    "hello world.md": {Data: []byte("hi")},
    "hello-world.md": {Data: []byte("hola")},
}
```

### Scanner

[`bufio.Scanner`](https://pkg.go.dev/bufio#Scanner) is an interface for reading newline-delimited lines of text from a file. You just call `Scan()` to read the line, and then call `Text()` to extract the text:

```go
scanner := bufio.NewScanner(filename)

scanner.Scan()
lineOne := scanner.Text()

scanner.Scan()
lineTwo := scanner.Text()
```

Here is a refactoring:

```go
readLine := func() string {
    scanner.Scan()
    return scanner.Text()
}

lineOne := readLine()[7:]
lineTwo := readLine()[13:]
```

And refactored yet again...:

```go
scanner := bufio.NewScanner(postBody)

readMetaLine := func(tagName string) string {
    scanner.Scan()
    return strings.TrimPrefix(scanner.Text(), tagName)
}
```

Here, we scan the lines of a file into a buffer. Use `Fprintln` because the scanner removes newline characters, and we need to maintain them in this case. we use the `TrimSuffix()` function to remove the final newline. In addition, this example shows how you can ignore a line:

```go
scanner.Scan() // ignore a line

buf := bytes.Buffer{}
for scanner.Scan() {
    fmt.Fprintln(&buf, scanner.Text())
}
body := strings.TrimSuffix(buf.String(), "\n")
```

#### Example

Here is a full example of what you can do with a single scanner:

- scan a single line and return the line
- create a function that accepts a scanner and returns text

```go
func newPost(postBody io.Reader) (Post, error) {
    scanner := bufio.NewScanner(postBody)                       // create the scanner

    readMetaLine := func(tagName string) string {
        scanner.Scan()                                          // read a line
        return strings.TrimPrefix(scanner.Text(), tagName)      // return the text
    }

    return Post{
        Title:       readMetaLine(titleSeparator),
        Description: readMetaLine(descriptionSeparator),
        Tags:        strings.Split(readMetaLine(tagsSeparator), ", "),
        Body:        readBody(scanner),
    }, nil
}

func readBody(scanner *bufio.Scanner) string {
    scanner.Scan()                                              // ignore a line

    buf := bytes.Buffer{}                                       // create a buffer
    for scanner.Scan() {                                        // scan text until there EOF
        fmt.Fprintln(&buf, scanner.Text())                      // write line to buffer
    }

    return strings.TrimSuffix(buf.String(), "\n")               // remove final newline
}
```

## Templates

[Calhoun.io blogs](https://www.calhoun.io/intro-to-templates-p1-contextual-encoding/) have helpful information about templates.

The basic format of all these examples:

- the program parses the template
- uses the template to render a post to any `io.Writer`

Basic template:

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.New("blog").Parse(postTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, p); err != nil {
        return err
    }

    return nil
}
```

Embedded in FS. This lets you load multiple templates and combine them.

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return err
    }

    if err := templ.Execute(w, p); err != nil {
        return err
    }

    return nil
}
```

With multiple templates, where you import into other files:

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return err
    }

    if err := templ.ExecuteTemplate(w, "blog.gohtml", p); err != nil {
        return err
    }

    return nil
}
```

### Benchmarking templates

A program must parse the template files repeatedly, and this affects performance. Here is the benchmark test before we refactor:

```bash
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: learngotests/renderer
cpu: Apple M3 Pro
BenchmarkRender-11        149312          8087 ns/op
PASS
ok      learngotests/renderer   1.455s
```

To improve this, we create a type that holds the parsed template and has a method to rerender the template:

```go
var (
    //go:embed "templates/*"
    postTemplates embed.FS
)

type Post struct {
    Title       string
    Body        string
    Description string
    Tags        []string
}

type PostRenderer struct {
    templ *template.Template
}

func NewPostRenderer() (*PostRenderer, error) {
    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return nil, err
    }

    return &PostRenderer{templ: templ}, nil
}

func (r *PostRenderer) Render(w io.Writer, p Post) error {

    if err := r.templ.ExecuteTemplate(w, "blog.gohtml", p); err != nil {
        return err
    }

    return nil
}
```

Here are the tests:

```go

func TestRender(t *testing.T) {
    var (
        aPost = Post{
            Title:       "hello world",
            Body:        "This is a post",
            Description: "This is a description",
            Tags:        []string{"go", "tdd"},
        }
    )

    postRenderer, err := NewPostRenderer()

    if err != nil {
        t.Fatal(err)
    }

    t.Run("it converts a single post into HTML", func(t *testing.T) {
        buf := bytes.Buffer{}

        if err := postRenderer.Render(&buf, aPost); err != nil {
            t.Fatal(err)
        }

        approvals.VerifyString(t, buf.String())
    })
}

func BenchmarkRender(b *testing.B) {
    var (
        aPost = Post{
            Title:       "hello world",
            Body:        "This is a post",
            Description: "This is a description",
            Tags:        []string{"go", "tdd"},
        }
    )

    postRenderer, err := NewPostRenderer()

    if err != nil {
        b.Fatal(err)
    }

    for b.Loop() {
        postRenderer.Render(io.Discard, aPost)
    }

}
```

Here are the post-refactor benchmarks:

```go
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: learngotests/renderer
cpu: Apple M3 Pro
BenchmarkRender-11       1847904           629.1 ns/op
PASS
ok      learngotests/renderer   1.434s
```

### FuncMaps

Before you parse a template, you can use a `FuncMap` to define functions that are called within your template. Here, we define a `sanitizeTitle` function between the `New` and `Parse` template methods. Our template string calls the function on the first title occurence:

```go
func (r *PostRenderer) RenderIndex(w io.Writer, posts []Post) error {
    indexTemplate := `<ol>{{range .}}<li><a href="/post/{{sanitizeTitle .Title}}">{{.Title}}</a></li>{{end}}</ol>`

    templ, err := template.New("index").Funcs(template.FuncMap{
        "sanitizeTitle": func(title string) string {
            return strings.ToLower(strings.Replace(title, " ", "-", -1))
        },
    }).Parse(indexTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, posts); err != nil {
        return err
    }

    return nil
}
```

The problem with this approach is that you can only test the sanitize the function by generating HTML output. You can't test the actual function itself. Also, you should avoid logic in templates at all costs.

The solution is to use a ["view model"](https://stackoverflow.com/questions/11064316/what-is-viewmodel-in-mvc/11074506#11074506), which is a type that represents the data that you want to display on the page and can be saved to a database. A view model is different from the domain model because it contains only information that you want to display in the UI.

To fix this, we create a view model that contains testable data and logic for our view:

```go
type PostViewModel struct {
    Title, SanitizedTitle, Description, Body string
    Tags                                     []string
}
```

Here, we call the `.SanitizedTitle` function in place of the URL title:

```go
func (r *PostRenderer) RenderIndex(w io.Writer, posts []Post) error {
    indexTemplate := `<ol>{{range .}}<li><a href="/post/{{.SanitizedTitle}}">{{.Title}}</a></li>{{end}}</ol>`

    templ, err := template.New("index").Parse(indexTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, posts); err != nil {
        return err
    }

    return nil
}

func (p Post) SanitizedTitle() string {
    return strings.ToLower(strings.Replace(p.Title, " ", "-", -1))
}
```

## Approval tests

[Approval tests](https://github.com/approvals/go-approval-tests)

```bash
go get github.com/approvals/go-approval-tests
```

An approval tool compares program output with an approved file that you created. Use this instead of using golden files. A golden file is the master file that you test your development code against.

## Generics

Types exist so you can tell the compiler what form of data to look for: a string, int, etc. Generics let you design functions that do not requrethat accept types that do not require concrete types, but rather types that have the behavior you need.

Generic functions need a type parameter that includes a description of your generic type and a label. Here, `T` is the label, and `comparable` is the description:

```go
func AssertEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

Alternately, you can use the empty interface (`interface{}` or `any`), which lets you pass any type. Make sure you use the `%+v` format in formatted strings.

The problem with `any` is that we are not telling the compiler anything about the types we pass to the function. There are absolutely NO constraints, which can lead to runtime errors. Generics let you provide some guidance (constraings) to the compiler. Also, you don't have to make type assertions if your function returns a generic type, the caller can use the type as it is returned.

Here is an implementation using generics:

```go
type Stack[T any] struct {
    values []T
}

func NewStack[T any]() *Stack[T] {
    return new(Stack[T])
}

func (s *Stack[T]) Push(value T) {
    s.values = append(s.values, value)
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.values) == 0
}

func (s *Stack[T]) Pop() (T, bool) {
    if s.IsEmpty() {
        var zero T
        return zero, false
    }

    index := len(s.values) - 1
    el := s.values[index]
    s.values = s.values[:index]
    return el, true
}
```

Here are the tests:

```go
type Stack[T any] struct {
    values []T
}

func NewStack[T any]() *Stack[T] {
    return new(Stack[T])
}

func (s *Stack[T]) Push(value T) {
    s.values = append(s.values, value)
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.values) == 0
}

func (s *Stack[T]) Pop() (T, bool) {
    if s.IsEmpty() {
        var zero T
        return zero, false
    }

    index := len(s.values) - 1
    el := s.values[index]
    s.values = s.values[:index]
    return el, true
}
```