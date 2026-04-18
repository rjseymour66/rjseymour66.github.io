+++
title = 'Goroutines'
date = '2025-08-30T12:08:53-04:00'
weight = 10
draft = false
+++

A goroutine is an independently running concurrent function that is invoked after the keyword `go`. Goroutines are executed and managed by the Go Scheduler. The Scheduler distributes the goroutines over multiple operating system threads that run on one or more processors. They run separate alongside other code with a thir own call stack that is a few KB.

Goroutines use a few features that manage goroutines:
- [Channels](../channels): Channels enable communication between goroutines.
- [WaitGroups](#waitgroups): A WaitGroup tracks the number of executing goroutines and blocks the calling process until all goroutines are complete.


## `main` goroutine

The `main` method is a goroutine---the main goroutine that comprises the lifetime of the application. To demonstrate, the `countToTen` function in the following example counts from 1 to 10 and logs the current number to the console each second. The main function calls `countToTen` as a goroutine, then sleeps for seven seconds. `countToTen` runs concurrently to the main method, logging numbers to the console, while the main method sleeps. After seven seconds, the main method resumes execution and exits. This means that `countToTen` does not have enough time to complete its loop before the program ends:

```go
func main() {
	fmt.Println("Count to 10 if you can!")
	go countToTen()
	time.Sleep(7 * time.Second)
	fmt.Println("Out of time!")
	os.Exit(0)

}

func countToTen() {
	for i := 1; i <= 10; i++ {
		time.Sleep(1 * time.Second)
		fmt.Println(i)
	}
}
```
You can prevent the program from exiting before the goroutines complete their work with [WaitGroups](#waitgroups).

## WaitGroups

A wait group is a message-passing facility that signals to a waiting goroutine when it is safe to proceed execution. The wait group doesn't need to know about what kind of work it is facilitating, only the number of goroutines that it needs to wait for. Think of it as a counter in the outer process (waiting goroutine) that keeps track of the number of concurrent tasks in process. You increment the counter for each concurrent task, perform the task, then decrement the counter when the work completes. The outer process blocks until the counter returns to 0. The general process is as follows:
1. Create a wait group.
2. Use the `.Add(int)` function to increment the number of goroutines to wait for.
3. Perform a task with the goroutine.
4. Call `.Done()` to signal to the wait group that the task is complete. This decrements the tasks registered with `.Add(int)` are complete.
5. In the outer goroutine, call `.Wait()`. The outer goroutine blocks until all goroutines in the wait group call `.Done()`.

### Example

The following example illustrates this process. It calls a `compress` function on the files passed on the command line:
1. Creates a wait group named `wg`.
2. Loops through the files passed on the command line.
3. For each file argument, we increment the number of tasks.
4. Launch a goroutine that performs a task.
5. When the task is complete, send a signal to the wait group to decrement the counter.
6. The outer process blocks until the wait group counter is 0, then proceeds execution.

```go
func main() {
	var wg sync.WaitGroup                   // 1
	for _, file := range os.Args[1:] {      // 2
		wg.Add(1)                           // 3
		go func(filename string) {          // 4
			compress(filename)
			wg.Done()                       // 5
		}(file)
	}
	wg.Wait()                               // 6
	fmt.Printf("Compressed %d files\n", len(os.Args[1:]))
}

func compress(filename string) error {
	in, err := os.Open(filename)
	if err != nil {
		return err
	}

	defer in.Close()

	out, err := os.Create(filename + ".gz")
	if err != nil {
		return err
	}
	defer out.Close()

	gzout := gzip.NewWriter(out)
	_, err = io.Copy(gzout, in)
	gzout.Close()
	return err
}
```

{{< admonition "Looping with goroutines" note >}}
In the preceding example, each goroutine runs in an IIFL that takes as an argument the `file` value from the outer `for range` loop. Using separate variables (`file` and `filename`) ensures that each goroutine operates on a different file, one for each iteration.

If the goroutine closure captured `file`, all goroutines would share that same variable. This variable changes with each iteration, and it might be the same value by the time the goroutines execute. In other words, every goroutine might try to compress the last file passed to the `for range` loop.

Passing `file` as an argument to the IIFL means that it is evaluated immediately, and each goroutine gets its a unique copy of the `file` variable in each iteration.
{{< /admonition >}}

### Concurrent functions

When you need to perform work within a function but also use WaitGroups to manage concurrency, pass the WaitGroup to the function as a parameter, and call `Done` within the function. All WaitGroup setup (declaration, `Add`) and cleanup (`Wait()`) needs to take place outside the functions:

1. Create the WaitGroup.
2. Add the number of goroutines you want to run.
3. Pass the WaitGroup as a parameter.
4. In the functions, call `Done`.
5. Wait for the goroutines to finish.

```go
func main() {
	var wg sync.WaitGroup 						// 1
	wg.Add(2) 									// 2
	go hello("hello", &wg)
	go world("world", &wg)
	wg.Wait() 									// 5
}

func hello(s string, wg *sync.WaitGroup) { 		// 3
	defer wg.Done() 							// 4
	for i := 0; i < 5; i++ {
		fmt.Println(s, i)
	}
}

func world(s string, wg *sync.WaitGroup) { 		// 3
	defer wg.Done()								// 4
	for i := 0; i < 5; i++ {
		fmt.Println(s, i)
	}
}
```

## Mutexes

_Mutex_ stands for “mutual exclusion”---it is a mechanism that handles concurrency through memory access synchronization. A mutex provies a concurrent-safe way to provide exclusive access to shared resources. They protect shared state.

Mutexes prevent _race conditions_, which occur when at least two processes are "racing" to read or write to the same data at the same time. One process might be in the middle of writing data while the other reads it, which can lead to data integrity problems and other errors.

`sync.Mutex` provides the `.Lock()` and `.Unlock()` methods to implement mutexes. First, you lock a resource, perform a task on the resource, then unlock it when the task is complete. Goroutines wait to access the resource until the lock is removed.

{{< admonition "Mutex vs RWMutex" note >}}
`Mutex` provides exclusive access to a resource to one goroutine at a time. `RWMutex` allows multiple goroutines to read from a resource, but only one can write, or change the state.
{{< /admonition>}}

### Example

This example reads one or more files, tallies how many times each word occurs in each file, and then adds the word and word count to the `words` struct. If you pass more than one file, the program reads files concurrently.

To make mutexes available for a type, declare an anonymous field that references `sync.Mutex`:

```go
type words struct {
    sync.Mutex
    found map[string]int
}
```

When you pass more than one file to this program, it reads the files concurrently and writes to the `found` map. This is a perfect circumstance for race conditions, because multiple goroutines might try to write to the map simultaneously.

The `add` method modifies the map. To protect against race conditions, add a lock to the `words` receiver and unlock it when the task is complete. When multiple goroutines try to access the `add` method, the first gets access and locks the others out. When the lock is released, other goroutines can get access to `add`. In addition, call the `Unlock()` method with `defer` so the lock is released when the method returns:

```go
func (w *words) add(word string) {
	w.Lock()                            // Start the Lock
	defer w.Unlock()                    // Release the Lock upon return
	count, ok := w.found[word]
	if !ok {
		w.found[word] = 1
		return
	}
	w.found[word] = count + 1
}
```

The `main` method safely reads from the file arguments. When the goroutines complete their work and the wait group stops blocking, words that occur more than once are logged to the console:

```go
func main() {
	var wg sync.WaitGroup                   // Create wait group
	w := newWords()
	if len(os.Args) < 2 {
		// handle arguments
	}

	for _, f := range os.Args[1:] {
		wg.Add(1)                           // Increment by 1 per file
		go func(file string) {              // Process file
			//do work
			wg.Done()                       // Decrement when file completes
		}(f)
	}
	wg.Wait()                               // Stop blocking main execution
	fmt.Println("Words that appear more than once:")
	for word, count := range w.found {
		if count > 1 {
			fmt.Printf("%s: %d\n", word, count)
		}
	}
}
```