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

| Method          | Description |
|-----------------|:------------|
| `t.Log()`        | Log a message. |
| `t.Logf()`       | Log a formatted message.|
| `t.Fail()`       | Mark the test as failed, but continue test execution. |
| `t.FailNow()`    | Mark the test as failed, and immediately stop execution. |
| `t.Error()`      | Combination of `Log()` and `Fail()`. |
| `t.Errorf()`     | Combination of `Logf()` and `Fail()`. |
| `t.Fatal()`      | Combination of `Log()` and `FailNow()`. |
| `t.Fatalf()`     | Combination of `Logf()` and `FailNow()`. |


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

| product     | rating  | price |
|:------------|---------|-------|
| prod one    | 5       | 20    |
| prod two    | 10      | 30    |
| prod three  | 15      | 40    |

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

Use the `t.Helper()` function to designate a function as a helper function. the testing pacakge ignores the helper function line number and instead logs the line number of the failing test:

```go
func helper(t *testing.T)  {
    t.Helper()
    // helper logic 
}

func TestWithHelper(t *testing.T)  {
    // testing...
    helper(t)
    // continue test ...
}
```

Helper functions accept an instance of the testing.T type, so make sure you pass the `t` testing instance to the helper in the `TestXxx` function. This provides the helper with access to the testing instance as the rest of the test function.

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

| Signature                    | Description |
|:-----------------------------|:------------|
| `func Example()`             | Example for the entire package. |
| `func ExampleParse()`        | Example for the `Parse` function. |
| `func ExampleURL()`          | Example for the `URL` type. |
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