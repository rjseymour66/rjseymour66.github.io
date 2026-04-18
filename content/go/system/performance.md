+++
title = 'Performance'
date = '2026-01-02T11:56:25-05:00'
weight = 80
draft = false
+++

Performance optimization in Go requires that you understand its garbage collector, and the distinctions between stack and heap memory allocations. You need to optimize code to reduce memory usage and minimize garbage collection, which increases the scalability and responsiveness of Go applications.

## Garbage collection

[Blog post about Go's garbage collector](https://go.dev/blog/ismmkeynote)

Garbage collection prevents memory links, dangling pointers, and double frees. Go's garbage collector (GC) handles memory inference---which memory to free---by tracking allocations on the heap, freeing unneeded allocations, and keeping allocations in use. The GC does this with tracing and reference counting.

### Tracing and reference counting

Go's tracing GC traces objects that are reachable by a chain of references from root objects, and it considers all other objects as garbage. Roots are objects that are always considered "alive" because the program can access them directly:

- Global variables
- Local variables on goroutine stacks
- Function arguments
- CPU registers
- Runtime-internal references

### GC algorithm

Go uses a concurrent, tri-color mark-and-sweep algorithm. Concurrent means it runs alongside your code and doesn't stop the runtime to clean up garbage. Tri-color is how the GC views objects for cleanup. Mark and sweep is split into two parts:


#### 1: Mark

This phase runs concurrently with the program execution. The first part of the phase is _stop-the-world_ (STW), where the program stops and Go identifies the root set to figure out what is in use.

Next, variables are labeled by color to determine whether they are garbage collected:
1. **white**: All variables are initially marked white, which means there is no decision made.
2. **gray**: Variables that are identified by their roots, but need to be explored further.
3. **black**: Variables that are in use.

During marking, the Go runtime allocates about 25% of CPU resources so that the GC is efficient enough to keep memory usage in check without overwhelming the system. It also allocates about 5% of CPU for mark assists, where goroutines help with marking if the GC is lagging due to memory allocations during the GC cycle.


#### 2: Sweep

Objects still marked white are garbage collected, and their memory is deallocated. This helps avoid memory leaks, but it can cause latency spikes on apps with large heaps.

### Stack

The stack stores variables that are tied to the function calls that create them:
- Local variables
- Function parameters
- Return values

The stack is efficient because it uses Last In, First Out (LIFO), and it pops objects off the stack using a pointer. The stack is small, and putting too much on the stack results in stack overflow.

### Heap

The heap stores variables that are less predictable and not explicitly tied to where they were created. These variables must live beyond the scope of the function they were created in. The heap is more flexible and dynamic, but this means it is slower in allocating memory. The GC manages heap cleanup, which adds overhead and reduces performance.

## GC Tooling

### GOGC

The `GOGC` environment variable is how you tune the GC. It accepts any integer greater than `0`. By default, it is set to `100`. This means that the GC tries to leave at least 100% of the initial heap memory available after a new GC cycle. If you set it to `50`, the GC runs more frequently and keeps the heap size smaller, which uses more CPU time. Setting it higher---for example, to `200`---means the GC runs less frequently, leading to apps that use large amounts of memory. You can also set it to `off` for programs that do not need the GC.

### GC pacer

The GC pacer regulates the timing of the GC cycles to balance the need to reclaim memory with the need to keep the program running efficiently. It decides when to run the GC based on the heap size and the allocation rate. It uses the `GOGC` setting as a guideline to determine thresholds that trigger a GC cycle.

GC is adaptive---if an app consumes lots of memory, then it will run the GC more often.

### GODEBUG

`GODEBUG` is an environment variable that gives insight into the Go runtime. Use this when you want to find memory leaks or optimize the GC overhead.

The following setting is often used to gain detailed information about garbage collection processes. It outputs detailed information for each GC cycle so you can understand the GC's impact on your application performance:

```bash
GODEBUG=gctrace=1 go run .
gc 1 @0.004s 2%: 0.058+0.49+0.035 ms clock, 0.70+0.29/0.64/0+0.42 ms cpu, 3->4->0 MB, 4 MB goal, 0 MB stacks, 0 MB globals, 12 P
```
- `gc 1`: Sequence number of the GC
- `@0.004s`: Time (seconds) since program started to when the GC ran
- `2%:`: Percentage of total program runtime spent on GC.
- `0.058+0.49+0.035 ms clock`: Breakdown of GC cycle time:
  - `0.058`: STW sweep mark start
  - `+0.49`: Concurrent mark and scan phase time
  - `+0.035 ms clock`: STW mark end
- `0.70`: CPU time for sweep mark start
- `+0.29/0.64/0`: CPU time for concurrent phases
- `+0.42 ms`: CPU time for STW mark end
- `3->4->0 MB`: Heap size at start, midpoint, and end of GC cycle
- `4 MB goal`: Next GC cycle target heap size
- `0 MB stacks`: Amount of memory reachable from stacks accounts for this amount of the heap.
- `0 MB globals`: Amount of memory reachable from global vars accounts for this amount of the heap.
- `12 P`: Number of processors

### Memory ballast

{{< admonition "Deprecated" warning >}}
Memory ballasts are relevant only for Go version 1.19 and earlier. Newer versions manage heap size with the `GOMEMLIMIT` environment variable.
{{< /admonition >}}

A memory balast is a large allocation of memory that is used only to influence the behavior of the GC. The GC runs when based on the heap size. If the heap is double what it was when the last GC cycle ended, then it begins. A memory ballast artificially increases the heap size, which increases the GC threshold and causes the GC to run less often. Here is an example implementation:

```go
var ballast []byte

func init() {
    ballast = make([]byte, 10<<30) // 10 GB (example)
}
```

### GOMEMLIMIT

`GOMEMLIMIT` is a soft cap on the memory usage of the Go runtime. It gives your application a memory budget and removes the need for manual tweaks like memory ballasts. It is a numeric value measured in bytes. You can add a unit suffix for clarity, such as `B`, `KiB`, `MiB`, or `GiB`. See IEC 80000-13 for more suffixes.



## Performance analysis

### Escape analysis

Escape analysis is how Go's compiler decides whether a variable goes to the stack or the heap:
- If the lifetime of the variable doesn't "escape" the function that its in, it goes to the stack. For example, does not have a return value, any variables within the function do not escape, and they are stored on the stack. If it returns a pointer to an integer, that pointer does escape the function.
- If the variable is passed around or returned from a function, then it "escapes" to the heap.

The goal is to improve memory usage by allocating variables on the stack when possible because the stack is faster and more CPU-cache friendly.

In Go, no goroutine can have a pointer to another goroutine's stack. This influences where the compiler stores variables. Here is a sample program and its escape analysis:

{{< admonition "Inlining functions" note >}}
`//go:noinline` is a compiler directive that tells the Go compiler to always create a function call with a stack frame for the function. Otherwise, the compiler might replace simple functions with the actual code of the function to enhance performance, which can change the stack frames. `//go:noinline` is helpful for benchmarking and escape analysis.
{{< /admonition >}}

```go
type person struct {
	name string
	age  int
}

func main() {
	p := createPerson
	fmt.Println(p)
}

//go:noinline
func createPerson() *person {
	p := person{
		name: "Steve Stevens",
		age:  100,
	}
	return &p
}
```

To view how Go performs escape analysis, use the `-gcflags "-m -m"` option. Ignore the lines about inlining and cost:
1. Go assigns each function a complexity cost to determine whether inlining is beneficial. Here, it is not beneficial to inline `main`.
2. `p` is allocated on the heap

```bash
go run -gcflags "-m -m" .
# perf
./main.go:16:6: cannot inline createPerson: marked go:noinline
./main.go:10:6: cannot inline main: function too complex: cost 83 exceeds budget 80     # 1
./main.go:12:13: inlining call to fmt.Println
./main.go:17:2: p escapes to heap:                                                      # 2
./main.go:17:2:   flow: ~r0 = &p:
./main.go:17:2:     from &p (address-of) at ./main.go:21:9
./main.go:17:2:     from return &p (return) at ./main.go:21:2
./main.go:17:2: moved to heap: p
./main.go:12:13: ... argument does not escape
0x491b20
```

### Pointers

A pointer is a variable that holds the address of another variable. They "point" to where the value lives, whether that is on the stack, heap, memory, etc. Pointers let you change the value of a variable without passing around the variable itself.

Declaring and getting the value from a pointer requires that you understand the following syntax:
1. Declare a pointer with an asterisk (`*`) before the type.
2. Store the address of a variable with the address-of operator (`&`).
3. Get the value that a pointer points to by dereferncing the pointer. Use the asterisk (`*`) to dereference. This is like saying, "Follow the pointer to the address and get the value."

```go
var pointer *int        // 1
x := 10
pointer = &x            // 2
fmt.Println(*pointer)   // 3
```

{{< admonition "Best practices" tip >}}
Follow these best practices when using pointers:
- **Null pointer checks**: Check whether a pointer is `nil` before dereferencing. This avoids runtime panics.
- **Passing pointers**: Use pointers when passing large structs to functions to avoid copying the entire structure.
{{< /admonition >}}

### Stack

The stack is where all local variables live until their function calls end. The stack is a LIFO data structure, where the data for a function is placed on the stack in the order it is , then popped off the stack from top to bottom and cleared when the function returns. Go has a limited stack size, but it is also dynamic and can resize as needed.

When a function is called, Go allocates space on the stack for its local variables. Each function call creates a stack frame that contains all the necessary information for the function, which includes the local variables, arguments, and return address.

### Heap

The heap is a much larger, more flexible space to store data. It is a less structured area of memory, which means it is slower than the stack.

### Pointers, stack, and heap

To illustrate, create a simple program that uses a linked list. The code consists of the following:
- `Node`: Struct that models each node in the list
- `buildList`: Function that creates a three-node list, and returns a pointer to the first node in the list.
- `main`: Entrypoint that builds the list and prints all its values.

The storage location for each value is determined by how it is called. To summarize, the pointer to the head of the linked list lives on the stack, the nodes live on the heap, and the pointers within the nodes live on the heap:
1. `Node` is a type declaration that lives in the program's compiled metadata. It does not live on either the stack or the heap.
2. `main` calls `buildList`, a function that returns a pointer to a `Node` instance. `head` is stored as a local variable in the stack, on `main`'s stack frame. 
3. `buildList` allocates all variables on the heap, which allows it to grow and shrink as needed. Its return value is an indicator:
   1. `return &n1` escapes the function and survives after `buildList` returns. That means that `n2` and `n3` must also be allocated on the heap.
   2. Because the `n*` variables are on the heap, `n*.next` values are stored on the heap because thats where they are created. `n*.next` is a pointer in a heap object that points to another heap object.  


```go
type Node struct {                  // 1
	value int
	next  *Node
}

func main() {
	head := buildList()             // 2
	fmt.Println(
        head.value, 
        head.next.value, 
        head.next.next.value
    )
}

func buildList() *Node {
	n1 := Node{value: 1}            // 3.1
	n2 := Node{value: 2}            // 3.1
	n3 := Node{value: 3}            // 3.1

	n1.next = &n2                   // 3.2
	n2.next = &n3

	return &n1                      // 3.1
}

```