+++
title = 'Fundamentals'
date = '2025-11-16T09:57:24-05:00'
weight = 10
draft = false
+++

Go includes built-in testing tools for functional and performance testing.

Unit testing
: Checks code in individual functions or modules.

Integration testing
: Verifies that different modules work well together.

Functional testing
: Verifies the correctness of the program output.

Test suite
: Collection of test cases. If you use testing resources, set up the resources at the start of the test suite, then tear them down at the end.

## Conventions

All Go tests must follow these conventions:
- All test files must end in `_test.go`, which lets the Go test tool identify them.
- Each test function must start with `TestXxx`, with the remainder of the name in camel case.
- A test function takes a single argument, a pointer to `testing.T`. `T` is a struct with testing methods that manages test state and can log test results.

### File location

Place test files in the same package as the code that they test.
  ```bash
  project-root
  ├── go.mod
  ├── main.go
  └── packagename
      ├── functional.go         # source code
      └── testing_test.go       # test file
  ```
### External tests

For external (integration) tests, use the original package name followed by `_test`. For example:
```go
package original_test
```
This package format requires that you import the source into the test file.



## Writing tests

Each test contain three main phases:

1. Arrange: Set up the test inputs and expected values:
   ```go
   a := 2, b := 3
   ```
2. Act: Execute the code that you are testing:
   ```go
   got := Add(a, b), want := 5
   ```
3. Assert: Verify that the code returns the correct values. You can use the `got`/`want` or `got`/`expected` semantics:
   ```go
   if got != want { ... }
   ```

| Phase   | Purpose                                                                                                      | Example                       |
| :------ | :----------------------------------------------------------------------------------------------------------- | :---------------------------- |
| Arrange | Set up the test inputs and expected values.                                                                  | `a := 2`, `b := 3`            |
| Act     | Execute the code that you are testing.                                                                       | `got := Add(a, b), want := 5` |
| Assert  | Verify that the code returns the correct values. You can use the `got`/`want` or `got`/`expected` semantics. | `if got != want { ... }`      |



### Single test

Run a single test to validate one specific behavior of a function. The three phases are shown in comments because single tests are usually simple:

```go
func TestParse(t *testing.T) {
	
    const uri = "https://github.com/username"                       // 1. Arrange

	got, err := Parse(uri)                                          // 2. Act
	if err != nil {
		t.Fatalf("Parse (%q) err = %q, want <nil>", uri, err)
	}
	want := &URL{
		Scheme: "https",
		Host:   "github.com",
		Path:   "username",
	}

	if *got != *want {                                              // 3. Assert
		t.Errorf("Parse (%q)\ngot   %#v\nwant  %#v", uri, got, want)
	}
}
```

### Table test

Table tests separate the test data from the logic so you can reuse the logic for different test cases. Use a table test when you need to test multiple inputs.

#### Arrange

A common way to arrange a test is to use table tests. Table tests are a way to provide multiple test cases that you loop over and test during the `Act` stage. To set up a table test, complete the following:
1. Create a `testCase` struct that models the inputs and expected outputs of the test:
   ```go
   type testCase struct {
	   a        int
	   b        int
	   expected int
   }
   ```
   {{< admonition "`name` field" note >}}
   Table tests commonly use a `name` field so you can distinguish between tests. For more information, see [Unit testing: Subtests](../unit-testing#subtests).
   {{< /admonition >}}
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
#### Act

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

#### Assert

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

## Test commands

1. Verbose output.
2. Test results are cached, but this skips the cache and forces Go to rerun the test.
3. Add the `-shuffle=on` flag to execute tests in a random order. This command returns the "shuffling seed" number. You can use this to run tests in the same order. This is helpful if you have a bug and want to reproduce the tests until you fix the issue:
   ```bash
   go test -v -shuffle=on
   -test.shuffle 1769368631148682425
   ...
   ```
4. Stops tests in a single package if there is a failing test. This is helpful if you want to work on the first failing test.
5. Run a specific test.
6. Run a specific subtest.
7. Globbing syntax. This test runs a specific subtest that begins with `with_port`.
8. Use the short flag to skip long-running tests, like integration tests. The test function must use the `testing.Short()` function, and optionally use `t.Skip` to provide context for skipping the test.

```go
go test -v                              // 1
go test -count=1                        // 2
go test -v -shuffle=on                  // 3
go test -v -shuffle=1769368631148682425
go test -v -failfast                    // 4
go test -v -run=TestName                // 5
go test -v -run=TestName/with_port      // 6
go test -v -run=TestName/^with_port     // 7
go test -v -short ./...                 // 8
```

## Example tests

Writing example tests within your test code shows how to use your package. If you change your code, the documentation updates automatically.


[Blog post](https://go.dev/blog/examples)

A _testable example_ is live documentation for code. You write a testable example to demonstrate the package API to other developers. The API includes the exported identifiers, such as functions, methods, etc. A testable example never goes out of date.

The testing package runs testable examples and checks their results, but it does not report successes or failures.

Conventionally, testable example files are named `example_test.go`. If you create multiple files, use the `_test.go` suffix. These examples display alongside the corresponding package in the documentation:

1. The `_test` suffix declares a test-only package that is named after the package that it tests. This is called _blackbox testing_---it can access only the package that it tests.
   
   Go usually doesn't allow multiple packages in the same directory, but it makes an exception for test packages. The `_test` suffix means that you cannot import it from other packages.
2. You have to import the package that you are testing.
3. Name the test `Example<FuncToTest>`. Example tests print their output to stdout, so they do not take a `*T` type.
4. Demonstrate how to use the function you are testing.
5. Print the result of the test to stdout. This is critical for the output assertion in the next step.
6. Output assertion. The `// Output` comment signals that the following line should equal what was printed to stdout.
7. This line match stdout. Whitespace and newlines matter. 

```go
package urlcopy_test                                            // 1

import (
	"fmt"
	"log"
	"urlcopy"                                                   // 2
)

func ExampleParse() {                                           // 3
	uri, err := urlcopy.Parse("https://github.com/username")    // 4
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(uri)                                            // 5
	// Output:                                                  // 6
	// https://github.com/username                              // 7
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


## Comparing structs

Comparing structs is not as straightforward as primitive types. You can use the `reflect` package, but the [`go-cmp` package](https://pkg.go.dev/github.com/google/go-cmp/cmp) is easier. Run this command to download the package:

```bash
go get github.com/google/go-cmp/cmp
```

Here is an example of how to compare values:

```go
func TestParseWithoutPath(t *testing.T) {
	const uri = "https://github.com"

	got, err := Parse(uri)
	if err != nil {
		t.Fatalf("Parse (%q) err = %q, want <nil>", uri, err)
	}
	want := &URL{
		Scheme: "https",
		Host:   "github.com",
		Path:   "",
	}

	if diff := cmp.Diff(want, got); diff != "" {
		t.Errorf("Parse(%q) mismatch (-want +got):\n%s", uri, diff)
	}
}
```

Here is the sample output:

```bash
go test -v .
=== RUN   TestParseWithoutPath
    url_test.go:45: Parse("https://github.com") mismatch (-want +got):
          &urlcopy.URL{
          	Scheme: "https",
          	Host:   "github.com",
        - 	Path:   "",
        + 	Path:   "username",
          }
...
```

## Failure messages

Test failure messages should be easy to read and show you how the test failed. Use `\n` and the correct format verbs: 

{{< admonition "Formatting verbs" tip >}}
If you use `%s`, `Errorf` calls the type's `String` method. Instead, use `%#v` to show exactly how the value is represented in code. Here are all versions of this format verb:

- `%v`: Default value format
- `%+v`: Include struct field names
- `%#v`: Go-syntax representation
{{< /admonition >}}

```go
got, err := Parse(uri)
if err != nil {
    t.Fatalf("Parse (%q) err = %q, want <nil>", uri, err)
}

if *got != *want {
	t.Errorf("Parse (%q)\ngot   %#v\nwant  %#v", uri, got, want)
}
```

## Log, Error, Fatal

t.Log
: You want to debug or show output but do not want to fail the test.

t.Error
: Something is wrong but the test can still keep running to collect more failures.

t.Fatal
: There’s no point continuing the test. For example, bad input, required init failed.

| Function    | Description                                                  | Use Case                                                                   |
| ----------- | ------------------------------------------------------------ | -------------------------------------------------------------------------- |
| `t.Log`     | Prints debugging or informational output.                    | Show internal state or progress without failing the test.                  |
| `t.Logf`    | Prints formatted log output.                                 | Outputs formatted debugging output.                                        |
| `t.Error`   | Logs an error and marks the test as failed.                  | Something is wrong but the test can continue running.                      |
| `t.Errorf`  | Logs a formatted error message and marks the test as failed. | Need a detailed or formatted error message.                                |
| `t.Fail`    | Marks the test as failed without printing a message.         | Use in helper functions to mark failure without logging.                   |
| `t.FailNow` | Marks test as failed and immediately stops execution.        | When continuing the test is pointless (e.g., missing required test setup). |
| `t.Fatal`   | Logs a fatal error and stops the test immediately.           | When a critical issue prevents further execution.                          |
| `t.Fatalf`  | Same as `t.Fatal` but with formatting support.               | Outputs formatted fatal error messages.                                    |
