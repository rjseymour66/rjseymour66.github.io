+++
title = 'Loops'
date = '2025-10-12T14:37:24-04:00'
weight = 20
draft = false
+++

In Go, every loop is a `for` loop.

## C-style loops

A C-style loop has three components separated by semicolons:

- **Initializer**: runs once before the loop starts
- **Condition**: evaluated before each iteration; the loop runs while it's true
- **Post statement**: runs at the end of each iteration

```go
for i := 0; i < 5; i++ {
    // do something
}
```

## While-style loops

Use the `for` keyword where other languages would use `while`. Omit the initializer and post statement and provide only a condition:

```go
i := 0
for i < 5 {
    i++
}
```

You can also loop on any boolean condition:

```go
for iterator.Next() {
    // do something
}

for line != lastLine {
    // do something
}

for !gotResponse || response.invalid() {
    // do something
}
```

## Infinite loops

Omit all three components to loop forever. Infinite loops are common in servers and background workers that run until explicitly stopped:

```go
for {
    // loop forever
}
```

## for range

{{< admonition "Operates on a copy" warning >}}
The `for range` loop operates on a copy of the value, so it cannot mutate the value.
{{< /admonition >}}

The `for range` loop iterates over an array, slice, map, or channel using an index and value:

```go
for index, value := range iterable {
    // do something
}
```

For maps, `range` yields the key and value:

```go
m := map[string]int{"a": 1, "b": 2}
for key, value := range m {
    // do something
}
```

If you don't need the index, use the blank identifier:

```go
for _, value := range iterable {
    // do something
}
```