+++
title = 'Arrays'
date = '2025-11-02T09:56:41-05:00'
weight = 20
draft = false
+++

An *array* in Go is a fixed-length sequence of elements of the same type. After you declare an array, neither the element type nor the length can change.

Storing elements contiguously in memory means each element follows the previous at a predictable offset. The processor loads cache lines sequentially, so iterating over an array is fast and cache-friendly.

Arrays are values, not references. When you pass an array to a function, Go copies the entire array regardless of its size. Assignment works the same way: assigning one array to another produces an independent copy of every element.

The following example shows array assignment creating a full copy:

```go
var array1 [5]string
array2 := [5]string{"Red", "Blue", "Green", "Yellow", "Pink"}
array1 = array2 // array1 is now an independent copy of array2
```

## Creating an array

The syntax for an array type is `[N]type`, where `N` is a fixed length known at compile time. Go provides four declaration forms:

1. **Standard declaration:** Go initializes every element to the zero value for the type.
2. **Array literal:** The idiomatic form when you know the values upfront.
3. **Ellipsis syntax:** Go infers the length from the number of elements provided.
4. **Index initialization:** Sets specific elements by index. Go zeroes any element not explicitly set.

The following example shows all four forms:

```go
func main() {
	var numbers [10]int                                         // 1
	beatles := [4]string{"john", "paul", "george", "ringo"}     // 2
	primes := [...]int{2, 3, 5, 7, 11}                          // 3
	sparse := [5]int{1: 10, 2: 20}                              // 4
}
```

### new

The `new` built-in allocates a zeroed array and returns a pointer to it. Unlike an array literal, `new` does not accept initial values.

The following example allocates a zeroed `[10]int` array and prints the pointer:

```go
func main() {
	p := new([10]int)
	fmt.Println(p) // &[0 0 0 0 0 0 0 0 0 0]
}
```

Passing a pointer to an array instead of the array itself copies only the pointer (8 bytes on 64-bit systems) rather than the full array. Any writes through the pointer modify the original array, so callers share the same memory.

### Pointer arrays

An array can hold pointers instead of values. This is useful when elements are optional or expensive to initialize upfront: unset elements are `nil` rather than a zero value.

Call `new` to allocate each element, then dereference to assign a value:

```go
func main() {
	array := [5]*int{0: new(int), 1: new(int)}
	// array[2], array[3], array[4] are nil

	*array[0] = 10
	*array[1] = 20
}
```

## Accessing elements

Access individual elements by index using `array[i]`. Indices are zero-based, and Go panics at runtime if you access an out-of-bounds index.

The following example accesses the first and last elements of a four-element array:

```go
func main() {
	beatles := [4]string{"john", "paul", "george", "ringo"}
	fmt.Println(beatles[0]) // john
	fmt.Println(beatles[3]) // ringo
}
```

### Iterate with range

To iterate over every element, use `range`. It returns the index and a copy of the value at each position. Call `len` to get the array length.

The following example prints each element with its index:

```go
func main() {
	beatles := [4]string{"john", "paul", "george", "ringo"}
	for i, name := range beatles {
		fmt.Printf("%d: %s\n", i, name)
	}
	// 0: john
	// 1: paul
	// 2: george
	// 3: ringo

	fmt.Println(len(beatles)) // 4
}
```

If you only need the values and not the indices, discard the index with `_`:

```go
for _, name := range beatles {
	fmt.Println(name)
}
```



### Convert to slice

The slice expression `array[low:high]` returns a slice backed by the array's memory. Omit both bounds to slice the full array. The resulting slice shares the underlying array, so writes through the slice modify the original.

The following example converts a full array to a slice:

```go
func main() {
	array := [5]int{1, 2, 3, 4, 5}
	slice := array[:] // slice shares array's memory
}
```

## Modifying an array

Arrays are fixed in length but mutable in content. Assign a new value to any element by index.

The following example replaces the element at index 4:

```go
func main() {
	numbers := [7]int{0, 1, 2, 3, 4, 5, 6}
	numbers[4] = 7 // [0 1 2 3 7 5 6]
}
```
