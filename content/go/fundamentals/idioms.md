+++
title = 'Idioms and language features'
linkTitle = 'Idioms'
date = '2025-08-12T23:23:49-04:00'
weight = 40
draft = false
+++


## Naked returns

A naked return omits the return values when the function uses named return values. Name the return values in the signature, assign them in the body, then return with no arguments:

```go
func nakedReturn() (a, b string) {
	a = "First string"      // named return vals
	b = "second string"
	return                  // naked return
}
```

## Comma-ok

The comma-ok idiom uses a second boolean return value to distinguish a missing key or failed assertion from a zero value.

### Maps

```go
m := map[string]int{"a": 1}
v, ok := m["a"] // v = 1, ok = true
x, ok := m["b"] // x = 0 (zero value), ok = false
```

### Type assertions

```go
var i interface{} = "hello"
s, ok := i.(string) // s = "hello", ok = true
n, ok := i.(int)    // n = 0, ok = false
```

### Channel receives

Channel receives use comma-ok to check whether a channel is closed:

- If the channel is open, you receive a value and `ok` is `true`.
- If the channel is closed, you receive the zero value for the type and `ok` is `false`.

```go
ch := make(chan int)
close(ch)
v, ok := <-ch // v = 0, ok = false (channel closed)
```