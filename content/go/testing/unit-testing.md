+++
title = 'Unit testing'
date = '2025-09-05T08:42:58-04:00'
weight = 30
draft = false
+++

In Go, a _unit_ is a package, and a _unit test_ verifies the behavior of a single package. Unit tests test simple pieces of code in isolation, such as functions or methods.


## Table-driven tests

Table tests use a slice of inputs and conditions that you feed a looping function to evalute their outputs.

Here is a simple example to illustrate. This tests a function named `addOne(a int) int`, which takes an integer as an argument and returns that argument plus 1. So, `addOne(3)` returns `4`:
1. Create a slice of anonymous structs
2. A name for each subtest
3. Inputs for the `addOne` function
4. Expected return value for each `addOne` test
5. The slice of test cases.
6. `for range` to iterate over the test cases. `tt` is short for "table test".
7. `t.Run` runs each test as a subtest. Subtests run in isolation, so you can use `t.Fatalf()` and stop only that test case. This also lets you run tests in parallel.
8. Get the return value of the function you are testing, passing in the table test input.
9. Get the expected value.
10. This is an idiomatic Go assertion test.

```go
func TestAddOne(t *testing.T) {
	tests := []struct {                             // 1
		name     string                             // 2
		a        int                                // 3
		expected int                                // 4
	}{                                              // 5
		{"add 1 to positive", 3, 4},
		{"add 1 to negative", -3, -2},
	}

	for _, tt := range tests {                      // 6
		t.Run(tt.name, func(t *testing.T) {         // 7
			got := addOne(tt.a)                     // 8
			want := tt.expected                     // 9

			if got != want {                        // 10
				t.Errorf("got %d, want %d", got, want)
			}
		})
	}
}
```


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

Subtests let you run different testing scenarios within a test function. You can use this within a single test function, but it works well for testing different contexts within a table test.

Use `t.Run()` to run subtests. Each subtest has its own `*T` and runs in its own goroutine in isolation, so you can use `t.Fatal[f]()`, and the table test continues execution. Because subtests run in isolation, you can also run them [in parallel](#parallel-testing).

{{< admonition "Panics" note >}}
A failure in a subtest does not stop test execution, but a panic will stop execution without running the remaining subtests.
{{< /admonition >}}

Here, we place the test cases in a separate, global variable:

```go
var parseTests = []struct {
	name string
	uri  string
	want *URL
}{
	{
		name: "with_data_scheme",
		...
	},
	{
		name: "full",
		...
	},
	{
		name: "without_path",
		...
	},
}

func TestParse(t *testing.T) {
	for _, tt := range parseTests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := Parse(tt.uri)
			if err != nil {
				t.Fatalf("Parse(%q) err = %v, want <nil>", tt.uri, err)
			}
			if *got != *tt.want {
				t.Errorf("Parse (%q)\ngot  %#v\nwant  %#v", tt.uri, got, tt.want)
			}
		})
	}
}
```


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


### Helpers

When a test fails within a helper function, the testing package logs the line number within the helper function. This makes it difficult to pinpoint where the test failed. To log the line number where the testing function calls the helper, use the `t.Helper()` function to designate a function as a helper function. 

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

## Parallel testing

By default, functions in the same function package run sequentially. Run the tests in parallel to reduce the time it takes to run tests.

To run tests in parallel, add `t.Parallel()` to your test or subtest function:

```go
func TestAddOne(t *testing.T) {
	t.Parallel()
	result := Add(1, 2)
	// assertions
}

func TestAddEvens(t *testing.T) {
	t.Parallel()
	result := Add(2, 4)
	// assertions
}

func TestAddOdds(t *testing.T) {
	t.Parallel()
	result := Add(3, 5)
	// assertions
}
```

Tests marked with `t.Parallel()` are run in parallel with other parallel tests. If you have a mixture of sequential and parallel tests, Go runs sequential tests first and then runs tests with `t.Parallel()`.

{{< admonition "Parallel subtests" error >}}
To run subtests in parallel, you must add `t.Parallel()` to both the parent test and subtest.
{{< /admonition >}}

`GOMAXPROCS` determines how many tests run in parallel. By default, `GOMAXPROCS` is set to the number of CPUs on the machine. You can override this with the `-parallel` flag, but make sure you do not set this flag to a value higher than the number of CPUs on the machine.

The following two commands set `parallel` and are equivalent:

```shell
go test -parallel=4 ./...
go test -parallel 4 ./...
```

{{< admonition "Parallel testing subtests" warning >}}
You get incorrect results when you run subtest in parallel because the `for...range` function that runs each test case uses a pointer to a variable, which is overwritten during each iteration. To fix this, make the variable a local variable within each subtest:

```go
for _, tc := range testCases {
	testCase := tc 								// local variable
	t.Run(testCase.name, func(t *testing.T) {
		t.Parallel()
		// ...
	})
}
```
{{< /admonition >}}

## Detecting data races

A data race occurs when multiple goroutines access the same variable simultaneously and at least one of them modifies it. For example, the `incr` function increments the `counter` variable:

```go
var counter int

func incr() { counter++ }
```

If you run this function in parallel subtests, you might get a data race:

```go
func TestDataRace(t *testing.T) {
	t.Parallel()
	t.Run("once", func(t *testing.T) {
		t.Parallel()
		incr()
		if counter != 1 {
			t.Errorf("counter = %d, want 1", counter)
		}
	})

	t.Run("twice", func(t *testing.T) {
		t.Parallel()
		incr()
		incr()
		if counter != 3 {
			t.Errorf("counter = %d, want 3", counter)
		}
	})
}
```

### race flag

{{< admonition "" note>}}
Only use the `race` flag in test code. This flag adds assembly code to the final binary, so it can dramatically slow down programs.
{{< /admonition >}}

Check for a data race with the `race` flag. You can set the `count` variable to a high number to make sure you give the race detectore enough changes to catch a data race:

```bash
go test . -run=DataRace$ -race -count=10
==================
WARNING: DATA RACE
Read at 0x0000007cc220 by goroutine 9:
...
```


#### Benchmarking helpers

To benchmark helper functions, include the `testing.TB` type so you an access the benchmarking package:

```go
func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

## Testing resources

Data and environment setup is called a _test fixture_. You need to make resources available for testing but also remove them from the file system when testing completes.

### setup and teardown

You can setup and create your testing resources with a `setup` helper function that returns a `teardown` function that cleans up the resources.

The code to test creates a simple configuration file:
1. `Config` struct.
2. `LoadConfig` takes path for a JSON file, reads the configuration in the file, then unmarshals it into a `Config` struct.
3. It returns the `Config` struct and an error.

```go
type Config struct { 										// 1
	Name string `json:"name"`
	Port int    `json:"port"`
}

func LoadConfig(path string) (*Config, error) { 			// 2
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, err
	}

	var c Config
	if err := json.Unmarshal(data, &c); err != nil {
		return nil, err
	}

	return &c, nil											// 3
}
```

To test this function, you need a configuration file in the file system. This means that you need to create a configuration file, test your function, then delete the configuration file:
1. Designate the function as a helper so you can pinpoint any errors in the output.
2. Create the test file path.
3. Create the test configuration file contents as a raw string.
4. Create the test file.
5. `teardown` removes the test file from the file system.
6. Return the file path and the `teardown` function.

```go
func setup(t *testing.T) (string, func()) {
	t.Helper() 													// 1
	path := "test-config.json" 									// 2

	content := `{ 												// 3
		"name": "test-service",
		"port": 8080
	}`

	err := os.WriteFile(path, []byte(content), 0644) 			// 4
	if err != nil {
		t.Fatalf("failed to create config file: %v", err)
	}

	teardown := func() { 										// 5
		t.Logf("Tearing down: removing %s", path)
		if err := os.Remove(path); err != nil {
			t.Fatalf("teardown failed: %v", err)
		}
	}

	return path, teardown 										// 6
}
```

In the test function, use `setup` and `teardown` to create and cleanup resources:
1. Call `setup` to get the file path for the test file, and the `teardown` function. Make sure you pass `t` to setup because it uses logging functions from the test package.
2. Defer the `teardown` function call to the end of the test function.
3. Do your test assertions.

```go
func TestLoadConfig(t *testing.T) {
	configPath, teardown := setup(t) 					// 1
	defer teardown() 									// 2

	cfg, err := LoadConfig(configPath)
	if err != nil {
		t.Fatalf("Loading config failed: %v", err)
	}

	//  Test assertions 								// 3
	// if cfg.Name != "test-service" {
	// 	throw error
	// }

}
```

### Temporary directories

Rather than manually removing testing resources, you can create a temporary directory with `TempDir`. This test function automatically deletes the directory and its contents when testing completes:
1. Create a temporary directory.
2. Create a file path string in the temporary directory.
3. Define the JSON content as a raw string.
4. Write the content to the file path in the temporary directory.

```go 
func setup(t *testing.T)  {
	t.Helper()

	dir := t.TempDir() 											// 1
	path := filepath.Join(dir, "config.json") 					// 2

	content := `{ 												// 3
		"name": "test-service",
		"port": 8080
	}`

	err := os.WriteFile(path , []byte(content), 0644) 			// 4
	if err != nil {
		t.Fatalf("failed to create config file: %v", err )
	}
}
```

### Cleaning up 

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

## Fuzz tests

Fuzz tests generate random test data for your test functions. It automates input data into your test functions to identify unexpected test cases. Go has fuzzing built into the `test` package as a feature.

A fuzz test has two parts:
- Seeding the input to the fuzz function with `f.Add`. This is called creating a _seed corpus_.
- Running the fuzz function itself by calling the `f.Fuzz` function. Pass it a fuzz target, which is a funciton that has a pointer to the `testing.T` parameter and a set of fuzzing arguments.

To run a fuzz function, you need to pass your function name to the `-fuzz` flag, and optionally pass the `-fuzztime` parameter to indicate how long you want the fuzz test to run:

```bash
go test -v -fuzz=ReverseString -fuzztime=30s
```

Read this [Go blog](https://go.dev/doc/tutorial/fuzz) for more info about the following function and test:

```go
func Reverse(s string) (string, error) {
	if !utf8.ValidString(s) {
		return s, errors.New("input is not valid UTF-8")
	}
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r), nil
}
```

```go
func FuzzReverse(f *testing.F) {
	testcases := []string{"Hello, world", " ", "!12345"}
	for _, tc := range testcases {
		f.Add(tc) // Use f.Add to provide a seed corpus
	}
	f.Fuzz(func(t *testing.T, orig string) {
		rev, err1 := Reverse(orig)
		if err1 != nil {
			return
		}
		doubleRev, err2 := Reverse(rev)
		if err2 != nil {
			return
		}
		if orig != doubleRev {
			t.Errorf("Before: %q, after: %q", orig, doubleRev)
		}
		if utf8.ValidString(orig) && !utf8.ValidString(rev) {
			t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
		}
	})
}
```



## Test coverage

Go can generate a test report that details how much of your code is tested. This does not mean that your code is bug-free---it means that the code is executed during testing.

Use the `-cover` option to view your code coverage:

1. Basic coverage for one package.
2. Write coverage data to a file.
3. View coverage per function in the terminal.
4. View coverage data in browser. The green sections are covered, red is not covered, gray does not need to be tested.
5. View coverage data in browser. `covermode` controls how coverage is tracked. `count` counts how many times each statement executes.


```bash
go test -cover                          			# 1
go test -coverprofile=cover.out         			# 2
go tool cover -func=cover.out 						# 3
go tool cover -html=cover.out           			# 4
go tool cover -html cover.out           			# 5
go test -covermode=count -coverprofile=count.out 	# 6
```
