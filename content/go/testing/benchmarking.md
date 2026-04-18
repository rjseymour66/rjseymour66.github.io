+++
title = 'Benchmarking'
date = '2025-11-19T21:48:20-05:00'
weight = 50
draft = false
+++


Benchmarking is a systematic method of measuring and comparing the performance of software. It creates a controlled environment where you can analyze the impact of changes in code, algorithms, or system architecture.

Benchmark tests are nonfunctional tests---they do test whether the software performs its intended purpose. They test how well the software performs in terms of stability, speed, and scalability.

Benchmark tests take `*testing.B` as a parameter. This type has many of the log and error functions available to the `*testing.T` type.


## Basic benchmarking

Benchmark functions are similar to regular testing functions. They use the `BenchmarkXxx(b *testing.B)`, and you place them in `<name>_test.go` file along with other test functions.

### `b.N` vs `b.Loop()`

The `N` integer lets you dynamically adjust the number of iterations that your code runs. There are two ways to use the `N` integer in your code: `b.N` and `b.Loop()`.

#### b.N
`b.N` is the "old way" to write loops. This was commonly called in a C-style `for` loop that repeats `b.N` times:
```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Add(1, 2)
	}
}
```
In this loop, the `N` integer is set by Go to a value that runs long enough to be accurate and reliable. You directly call the `N` integer.

The issue is that the compiler always tries to find optimizations. It sees the `Add` function and removes it and computes the value at compile time because it has no side effects (no return value, no change in inputs). To make the compiler "observe" `Add`, you have to assign its result to a variable.

#### b.Loop()

`b.Loop` is the "new way" to write loops:
```go
func BenchmarkAdd(b *testing.B) {
	for b.Loop() {
		Add(1, 2)
	}
}
```
This lets Go control the iteration count, and when the benchmark timing starts and stops. The compiler does not try to optimize code found within `b.Loop`, so you do not have to create a variable to store the result of `Add`.


### Example

To demonstrate, here is a simple function that adds two integers and returns the answer:

```go
func Add(a, b int) int {
	return a + b
}
```

Here is the benchmark test function:
1. Pass the function a pointer to `testing.B`. `B` is a struct that manages benchmark timing and specifies how many iterations to run.
2. Run a loop for the benchmark tests. The benchmark package determines how many times this loop runs, which is until the measurements are statistically meaninful.
3. The function you want to benchmark.

```go
func BenchmarkAdd(b *testing.B) { 		// 1
	for b.Loop() { 						// 2
		Add(1, 2) 						// 3
	}
}
```

To run the performance test, use the `-bench` flag. This command runs all benchmark functions in the package:
1. Command to run the tests.
2. Operating system.
3. Chip architecture.
4. Package name.
5. Details about the CPU.
6. Number of cores used in the test | Number of iterations | Average time per iteration
   
```bash
go test -bench=.                                            # 1
goos: linux                                                 # 2
goarch: amd64                                               # 3
pkg: testdir                                                # 4
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz               # 5
BenchmarkAdd-12    	1000000000	         0.2460 ns/op       # 6
PASS
ok  	testdir	0.275s
```

## Flags

### bench flag

`-bench=` lets you filter benchmark tests that you want to run. It accepts a regex that matches test names. To run all tests, use the `.` character:

```bash
go test -bench=.
```

### count flag

Use the `count` flag if you want to run the same performance test a specified number of times. This makes sure that the timing is not a one-off. Pass `count` the number of tests that you want to run:

```bash
go test -bench=. -count=5
goos: linux
goarch: amd64
pkg: testdir
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkAdd-12    	1000000000	         0.2404 ns/op
BenchmarkAdd-12    	1000000000	         0.2397 ns/op
BenchmarkAdd-12    	1000000000	         0.2387 ns/op
BenchmarkAdd-12    	1000000000	         0.2408 ns/op
BenchmarkAdd-12    	1000000000	         0.2440 ns/op
PASS
ok  	testdir	1.338s
```

### benchtime flag

Use the `-benchtime` flag to control the minimum duration that the benchmark tests run or increase the number of iterations:

1. Runs for 2 seconds.
2. Runs exactly 10 iterations.

```bash
go test -run=XXX -bench=BenchmarkAdd -benchtime=2s 	# 1
go test -bench=BenchmarkAdd -benchtime=10x 			# 2
```

### run flag

Benchmark tests share `x_test.go` files alongside regular test functions. To execute only the benchmark functions, use the `run` flag.

`run` takes a regular expression. Go runs only functions that match that regular expression. To run only benchmark tests, pass `run` a regular expression that matches no test functions.

The `^$` regex matches the beginning and end of a string, which matches an empty string. You could also pass something random that you know won't match your function names, such as `-run=XXX` or `-run=' '`. `-run=XXX` ensures that it does not match any regular tests and only runs performance tests:

```bash
go test -v -bench=. -run=^$
goos: linux
goarch: amd64
pkg: testdir
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkAdd
BenchmarkAdd-12    	1000000000	         0.2447 ns/op
PASS
ok  	testdir	0.274s
```

## Avoiding test fixtures

A test fixture is any data or setup that you need to perform within your tests. To avoid testing test fixtures, use these benchmarking functions:

- `b.StartTimer`: Start the benchmark timer.
- `b.StopTimer`: Stop the benchmark timer.
- `b.RestartTimer`: Restart the benchmark timer.

For example, here is a function that flips an image:

```go
func flip(grid [][]color.Color) {
	for x := 0; x < len(grid); x++ {
		col := grid[x]
		for y := 0; y < len(col)/2; y++ {
			k := len(col) - y - 1
			col[y], col[k] = col[k], col[y]
		}
	}
}
```

### ResetTimer

The benchmark test needs to load the image first, so you need to reset the benchmark timer:
1. Load the resource.
2. Reset the benchmark timer.
```go
func BenchmarkFlip(b *testing.B) {
	grid := load("image.png")           // 1
	b.ResetTimer()                      // 2
	for i := 0; i < b.N; i++ {
		flip(grid)
	}
}
```

### StopTimer and StartTimer

If you need to perform the task during every iteration of the loop, you can start and stop the timer:
1. Create the loop.
2. Stop the timer before you set up the resources.
3. Start the timer after the setup and before the function you want to benchmark.

```go
func BenchmarkFlip(b *testing.B) {
	for i := 0; i < b.N; i++ {          // 1
		b.StopTimer()                   // 2
		grid := load("image.png")
		b.StartTimer()                  // 3
		flip(grid)
	}
}
```


## Sub-benchmarks

Sub-benchmark tests let you run table-driven performance tests. This example demonstrates a more common usage. It tests the performance of a function with different inputs:

1. Create a slice of anonymous structs to store the test cases.
2. This holds the individual test case name.
3. Inputs for each test.
4. Test case definitions.
5. This `for...range` loops over each test case.
6. Within each loop, call `b.Run` to call each test case. It takes the name of the sub-benchmark test case and a function that contains the benchmarking code. The first input is always the `name` field of the test input, and the function is always an anonymous testing function provided below.
7. The benchmarking loop.

```go
func BenchmarkSum(b *testing.B) {
	cases := []struct {                          	// 1
		name string                               	// 2
		a, b int                                  	// 3
	}{
		{"small", 1, 2},                          	// 4
		{"large", 1000, 2000},                    
	}

	for _, c := range cases {                    	// 5
		b.Run(c.name, func(b *testing.B) {        	// 6
			for i := 0; i < b.N; i++ {             	// 7
				Sum(c.a, c.b)
			}
		})
	}
}
```

Run this test:

```bash
go test -bench=BenchmarkSum
```

### Specify sub-benchmark test

You can run a specific sub-benchmark test with the `-bench` flag. Specify the test case name after the benchmark test name, separated by a forward slash. For example, this command runs the `small` sub-benchmark test within the `BenchmarkSum` test:

```bash
go test -bench=BenchmarkSum/small
goos: linux
goarch: amd64
pkg: perf
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkSum/small-12         	1000000000	         0.3237 ns/op
PASS
ok  	perf	0.360s
```

## Comparing benchmarks

Use `benchstat` to compare results from performance tests before and after you make code changes. `benchstat` provides a statistical analysis of benchmark results. This helps you compare different test run output to understand the different versions of your code:

1. Install `benchstat`:
   ```bash
   go install golang.org/x/perf/cmd/benchstat@latest
   ```
2. Benchmark your code, and redirect the output to a file:
   ```bash
   go test -bench=BenchmarkFlip -run=XXX -count=10 > old-flip.txt
   ```
3. Refactor.
4. Run the benchmarks again, redirecting the output to a different file:
   ```bash
   go test -bench=BenchmarkFlip -run=XXX -count=10 > new-flip.txt
   ```
5. Compare the benchmark tests with `benchstat`:
   ```bash
   benchstat old-flip.txt new-flip.txt
   ```

## Memory allocations

The `-benchmem` flag measures memory allocations. It adds two new columns to the benchmark output:
- `allocs/op`: Number of memory allocations per operation.
- `B/op`: Number of heap bytes allocated per operation.

```bash
go test -bench=. -benchmem
goos: linux
goarch: amd64
pkg: perf
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkSum/small-12         	1000000000	         0.3446 ns/op	       0 B/op	       0 allocs/op
BenchmarkSum/large-12         	1000000000	         0.3229 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	perf	0.748s
```



## Profiling a program

Use `pprof` to profile how your program uses system resources. `pprof` is the performance profiling tool that shows you where your system is spending time and memory.

A _profile_ is a collection of stack traces that show the sequence of certain events like CPU use, memory allocation, etc. Profiling has two parts:
- Creating the profile and saving it to a file.
- Running `pprof` to analyze the profile.

### CPU profiling

The CPU profile helps you understand how much time is spent processing specific parts of your code. When you profile your CPU, the runtime interrupts itself every 10ms and records the stack trace. The amount of time the code appears in the profile indicates how much time is spent in that particular line of code.

1. When you have the code that you want to profile, run this command. It creates a binary file named `cpu.prof` that you can analyze with `pprof`:

   ```bash
   go test -cpuprofile cpu.prof -bench=Resize -run=XXX
   goos: linux
   goarch: amd64
   pkg: testdir
   cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
   BenchmarkResize-12    	       3	 335413698 ns/op
   PASS
   ok  	testdir	2.227s
   ```

2. Next, you need to analyze `cpu.prof`. The easiest way is in the web interface, which requires Graphviz. To install this on Linux:
   ```bash
   sudo apt install graphviz
   ```
3. Start the web interface with `go tool`:
   ```bash
   go tool pprof -http localhost:8080 cpu.prof
   ```

### Memory profiling


1. Generate a memory profile with the `memprofile` flag. This command writes the memory profile to `mem.out`:

   ```bash
   go test . -bench=BenchmarkURLStringLong -benchmem -memprofile=mem.out
   ```
2. Inspect the `String` method's allocations using the `list` flag. The `list` flag tells pprof to find all functions whose name matches `String`, open its source code, and annotate each line with allocation data.
   
   The output shows how much memory each line of the code allocates:
   - `flat`: Number of allocations on that line.
   - `cum`: Number of allocations including callees.
   ```bash
   go tool pprof -list String mem.out
   Total: 7.70GB
   ROUTINE ======================== urlcopy.(*URL).String in /path/to/gofile.go
       7.70GB     7.70GB (flat, cum)   100% of Total
            .          .     34:func (u *URL) String() string {
            .          .     35:	if u == nil {
            .          .     36:		return ""
            .          .     37:	}
            .          .     38:	var s string
            .          .     39:	if sc := u.Scheme; sc != "" {
            .          .     40:		s += sc
          1GB        1GB     41:		s += "://"
            .          .     42:	}
            .          .     43:	if h := u.Host; h != "" {
       1.83GB     1.83GB     44:		s += h
            .          .     45:	}
            .          .     46:	if p := u.Path; p != "" {
       1.92GB     1.92GB     47:		s += "/"
       2.95GB     2.95GB     48:		s += p
            .          .     49:	}
            .          .     50:	return s
            .          .     51:}
	```
