---
title: "Observability"
weight: 100
description: >
  Working with observability in Go.
---

### Benchmarking

Running Go benchmarks is similar to running tests:
1. Write the benchmark functions using the `testing.B` type.
2. Run the benchmarks using the `go test` tool with the `-bench` parameter.

You want to benchmark your tool according to the main use case, so create test files to replicate your workload.


### Example benchmark function

The following benchmark test runs a tool on all CSV test files:
```go
func BenchmarkRun(b *testing.B) {
	filenames, err := filepath.Glob("./testdata/benchmark/*.csv")
	if err != nil {
		b.Fatal(err)
	}

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		if err := run(filenames, "avg", 2, io.Discard); err != nil {
			b.Error(err)
		}
	}
}
```
In the preceding example:
- `filepath.Glob()` matches a pattern to find all files with the `.cvs` file extension.
- `b.ResetTimer()` resets the benchmark clock. This ignores any time used to prepare for the benchmark's execution.
- `b.N` is the upper limit of the loop, which is adjusted to last about one second.
- `io.Discard` implements the `io.Writer` interface, but does not write output to any resource.

### Benchmark run commands

Run the benchmark tool with the `-bench <regex>` parameter, where `regex` is a regular expression that matches the benchmark tests that you want to run. To skip regular tests in the test files, include `-run ^$` in the command. For example:
```go
$ go test -bench . -run ^$
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	       2	 543565503 ns/op
PASS
ok  	colstats	1.588s
```
Run additional executions of the benchmark test with the `-benchtime` parameter. This parameter accepts a duration in time to run the benchmark, or a fixed number of executions. 

Run and save benchmarks to a file with the `tee` command. For example:
```
$ go test -bench . -benchtime=10x -run ^$ | tee benchresults00.txt
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 546183149 ns/op
PASS
ok  	colstats	6.067s

$ cat benchresults00.txt 
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 546183149 ns/op
PASS
ok  	colstats	6.067s
```
Compare two benchmark files to measure improvements after optimizations:
```go

```

### Profiling tools

The Go profiler gives you a breakdown of where your program spends its time executing. You can determine which functions consume the most time and target them for optimization.

Profile your tools using one of the two following options:
1. Add code directly to your program. This requires that you manually maintain profiling code.
2. Run the profiler integrated with the testing and benchmarking tools
   > The second option is easiest

Use the CPU profiler to create a profile file and a .test file:
```go
$ go test -bench . -benchtime=10x -run ^$ -cpuprofile cpu00.pprof
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 555744851 ns/op
PASS
ok  	colstats	6.230s
```
In the preceding command, `cpu00.pprof` is the 'profile name'.

Analyze the profiling results with `go tool pprof` and the profile name:
```go
$ go tool pprof cpu00.pprof 
File: colstats.test
Type: cpu
Time: Dec 5, 2022 at 12:12am (EST)
Duration: 6.22s, Total samples = 7.18s (115.38%)
Entering interactive mode (type "help" for commands, "o" for options)
```
The `top` command shows where your program is spending the most time:
```go
(pprof) top
Showing nodes accounting for 4880ms, 67.97% of 7180ms total
Dropped 120 nodes (cum <= 35.90ms)
Showing top 10 nodes out of 87
      flat  flat%   sum%        cum   cum%
    1000ms 13.93% 13.93%     1090ms 15.18%  runtime.heapBitsSetType
     730ms 10.17% 24.09%     4340ms 60.45%  encoding/csv.(*Reader).readRecord
     620ms  8.64% 32.73%      620ms  8.64%  runtime.memmove
     520ms  7.24% 39.97%     2510ms 34.96%  runtime.mallocgc
     490ms  6.82% 46.80%      490ms  6.82%  indexbytebody
     420ms  5.85% 52.65%      420ms  5.85%  strconv.readFloat
     360ms  5.01% 57.66%      360ms  5.01%  runtime.memclrNoHeapPointers
     290ms  4.04% 61.70%      290ms  4.04%  runtime.nextFreeFast (inline)
     260ms  3.62% 65.32%      260ms  3.62%  runtime.procyield
     190ms  2.65% 67.97%      190ms  2.65%  syscall.Syscall
```
The `top` command does not sort by cumulative time. Add the `-cum` flag:
```go
(pprof) top -cum
Showing nodes accounting for 2.45s, 34.12% of 7.18s total
Dropped 120 nodes (cum <= 0.04s)
Showing top 10 nodes out of 87
      flat  flat%   sum%        cum   cum%
         0     0%     0%      6.24s 86.91%  colstats.BenchmarkRun
     0.01s  0.14%  0.14%      6.24s 86.91%  colstats.run
         0     0%  0.14%      6.24s 86.91%  testing.(*B).runN
         0     0%  0.14%      5.71s 79.53%  testing.(*B).launch
     0.13s  1.81%  1.95%      5.69s 79.25%  colstats.csv2float
     0.05s   0.7%  2.65%      4.73s 65.88%  encoding/csv.(*Reader).ReadAll
     0.73s 10.17% 12.81%      4.34s 60.45%  encoding/csv.(*Reader).readRecord
     0.52s  7.24% 20.06%      2.51s 34.96%  runtime.mallocgc
     0.01s  0.14% 20.19%      1.59s 22.14%  runtime.makeslice
        1s 13.93% 34.12%      1.09s 15.18%  runtime.heapBitsSetType
```
After all the testing functions, you can see that the csv2float function takes up most of the execution time. To investigate further, use the `list <regex>` command. This command shows the source code for any function that matches the `regex` and displays how much time is spent executing each line in the function:
```go
(pprof) list csv2float
Total: 7.18s
ROUTINE ======================== colstats.csv2float in /home/ryanseymour/Development/go-projects/command-line/colstats/csv.go
     130ms      5.69s (flat, cum) 79.25% of Total
         .          .     24:// any function that matches this signature is of this type
         .          .     25:type statsFunc func(data []float64) float64
         .          .     26:
         .          .     27:func csv2float(r io.Reader, column int) ([]float64, error) {
         .          .     28:	// Create the CSV Reader used to read in data from CSV files
         .       20ms     29:	cr := csv.NewReader(r)
         .          .     30:	// adjusting column arg for 0-based index
         .          .     31:	column--
         .          .     32:	// Read in all CSV data
         .      4.73s     33:	allData, err := cr.ReadAll()
         .          .     34:	if err != nil {
         .          .     35:		return nil, fmt.Errorf("Cannot read data from file: %w", err)
         .          .     36:	}
         .          .     37:
         .          .     38:	var data []float64
         .          .     39:	/*
         .          .     40:		convert [][]string to [][]float64
         .          .     41:	*/
         .          .     42:
         .          .     43:	// loop through all records
      60ms       60ms     44:	for i, row := range allData {
         .          .     45:		// skip the first row that contains the column headers
      10ms       10ms     46:		if i == 0 {
         .          .     47:			continue
         .          .     48:		}
         .          .     49:		// Checking number of cols in CSV file to verify flag value
         .          .     50:		if len(row) <= column {
         .          .     51:			// file does not have that many columns
         .          .     52:			return nil,
         .          .     53:				fmt.Errorf("%w: File has only %d columns", ErrInvalidColumn, len(row))
         .          .     54:		}
         .          .     55:		// Try to convert data read into a float number
      40ms      770ms     56:		v, err := strconv.ParseFloat(row[column], 64)
         .          .     57:		if err != nil {
         .          .     58:			return nil, fmt.Errorf("%w: %s", ErrNotNumber, err)
         .          .     59:		}
         .          .     60:
      20ms      100ms     61:		data = append(data, v)
         .          .     62:	}
         .          .     63:	return data, nil
         .          .     64:}
```
To view a relationship graph in a browser, use the `web` command:
```go
(pprof) web
(pprof) quit
```
### Memory profiling

This is similar to running a benchmark, but use the `-memprofile` option:
```go
$ go test -bench . -benchtime=10x -run ^$ -memprofile mem00.pprof
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 551000648 ns/op
PASS
ok  	colstats	6.049s
```
This command creates the `mem00.pprof` file in your current working directory.

View the results of the memory profile with `go tool pprof` and the `-alloc_space` option:
```go
$ go tool pprof -alloc_space mem00.pprof
File: colstats.test
Type: alloc_space
Time: Dec 5, 2022 at 12:35am (EST)
Entering interactive mode (type "help" for commands, "o" for options)
```
Use the `top` command with the and sort with the `-cum` flag to view the parts of the program that allocate the most memory:
```go
(pprof) top -cum
Showing nodes accounting for 5.10GB, 99.89% of 5.10GB total
Dropped 28 nodes (cum <= 0.03GB)
Showing top 10 nodes out of 11
      flat  flat%   sum%        cum   cum%
         0     0%     0%     5.10GB 99.92%  colstats.BenchmarkRun
    1.13GB 22.23% 22.23%     5.10GB 99.92%  colstats.run
         0     0% 22.23%     5.10GB 99.92%  testing.(*B).runN
         0     0% 22.23%     4.64GB 90.89%  testing.(*B).launch
    0.64GB 12.50% 34.73%     3.96GB 77.66%  colstats.csv2float
    2.05GB 40.11% 74.84%     3.29GB 64.43%  encoding/csv.(*Reader).ReadAll
    1.24GB 24.32% 99.16%     1.24GB 24.32%  encoding/csv.(*Reader).readRecord
         0     0% 99.16%     0.46GB  9.03%  testing.(*B).run1.func1
         0     0% 99.16%     0.04GB  0.74%  bufio.NewReader (inline)
    0.04GB  0.74% 99.89%     0.04GB  0.74%  bufio.NewReaderSize (inline)
(pprof) 
```
### Total memory allocation

Use the -benchmem flag to view the total memory allocation for a program. The following command uses `tee` to send the output to STDOUT and the file parameter:
```go
$ go test -bench . -benchtime=10x -run ^$ -benchmem | tee benchresults00m.txt
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 530825085 ns/op	495567618 B/op	 5041035 allocs/op
PASS
ok  	colstats	5.837s
```

### Tracing

Tracing shows you how your application manages resources, such as network connections or file reads. 

Add the `-trace` option to the benchmark command:

```go
$ go test -bench . -benchtime=10x -run ^$ -trace trace01.out
goos: linux
goarch: amd64
pkg: colstats
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkRun-12    	      10	 571258799 ns/op
PASS
ok  	colstats	6.253s
```

View the results with `go tool trace`. The following command opens the contents of the trace file on a random port in the browser:

```go
$ go tool trace trace01.out 
2022/12/05 18:48:49 Parsing trace...
2022/12/05 18:48:50 Splitting trace...
2022/12/05 18:48:51 Opening browser. Trace viewer is listening on http://127.0.0.1:46209
```