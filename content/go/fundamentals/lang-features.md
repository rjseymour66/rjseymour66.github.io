+++
title = 'Language features'
date = '2025-08-29T08:30:12-04:00'
weight = 10
draft = false
+++

## Type system

Go has no type hierarchies. There are no classes, no parent-child relationships, and no inheritance. Polymorphism is achieved with interfaces.

### Composition over inheritance

Go uses struct embedding instead of inheritance. Embedding places one struct type inside another, and the outer type can access the inner type's fields and methods directly, as if they were its own.

The `hlog.Interceptor` from the `linkd` project is a practical example. It needs to behave like an `http.ResponseWriter` — forwarding all standard methods — but intercept `WriteHeader` to capture the status code:

```go
type Interceptor struct {
    http.ResponseWriter          // all ResponseWriter methods promoted to Interceptor
    OnWriteHeader func(code int)
}

func (ic *Interceptor) WriteHeader(code int) {
    if ic.OnWriteHeader != nil {
        ic.OnWriteHeader(code)   // capture the status code
    }
    ic.ResponseWriter.WriteHeader(code) // then delegate to the original
}
```

`Interceptor` embeds `http.ResponseWriter` and gets `Header()`, `Write()`, and all other methods for free. It only defines `WriteHeader` to add behavior before delegating. No inheritance is involved. `Interceptor` is not a subtype of `http.ResponseWriter`. It satisfies the same interface because it has all the required methods.

## Address space

The address space is the set of virtual memory addresses your program can use, managed by the OS and the Go runtime. Every process gets its own virtual address space from the OS, so addresses in one program are completely isolated from addresses in another.

### Memory regions

The Go runtime manages memory inside your program's address space:

| Memory region    | Description                                         | Growth direction |
| ---------------- | --------------------------------------------------- | ---------------- |
| Kernel space     | Reserved for OS, not accessible by Go               | n/a              |
| Shared libs      | System libraries, Go runtime, cgo libs              | n/a              |
| Heap             | Dynamically allocated vars (`make`, `new`, escapes) | up               |
| Free / mmap      | Unused virtual memory, used when heap expands       | n/a              |
| Goroutine stacks | Each goroutine has its own stack                    | down             |
| Globals / data   | Package-level vars, constants, initialized data     | n/a              |
| Code segment     | Compiled Go instructions, read-only                 | n/a              |
| Reserved / NULL  | Protects invalid access at address `0x0`            | n/a              |

The garbage collector traces pointers across the heap and frees memory that is no longer reachable.

### Pointers and memory safety

A pointer (`*T`) is an address within your program's address space. Go prevents you from constructing arbitrary addresses and dereferencing them. Unlike C, you cannot cast an integer to a pointer and read memory at will. If your program accesses memory outside its address space, the OS terminates it with a segmentation fault.
