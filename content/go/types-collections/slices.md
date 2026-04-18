+++
title = 'Slices'
date = '2025-11-02T09:54:27-05:00'
weight = 20
draft = false
+++

A *slice* in Go is a dynamically sized view into an array. A slice has three fields:

*Pointer*
: Points to the element in the underlying array where the slice begins.

*Length*
: The number of elements the slice can access.

*Capacity*
: The number of elements between the start of the slice and the end of the underlying array.

You cannot create a slice with a capacity smaller than its length.

{{< admonition "slice vs array" note >}}
If you specify a value inside the `[]` operator (`[4]varName`), you are creating an array. If you do not specify a value, you create a slice:

```go
array := [3]int{10, 20, 30}
slice := []int{10, 20, 30}
```

An *array* is a value. It has a fixed size, and Go initializes its elements to their zero value.

A *slice* is a reference to an underlying array. It has no specified length, and its zero value is `nil`. Slicing does not copy data: it creates a new slice header pointing to the original array. Modifying elements through a slice modifies the underlying array, which any other slice pointing to the same array will also see.
{{< /admonition >}}


## Creating a slice

The slice type syntax is `[]type` with no length specified. Go provides three declaration forms:

1. **Nil slice:** The zero value for a slice. Its length is zero and it has no underlying array. Most standard library functions that accept slices also accept nil slices.
2. **Slice literal:** The idiomatic form when you know the values upfront.
3. **Empty slice:** Represents an empty but non-nil collection. Prefer this when you need to distinguish "no results" from "uninitialized," for example when marshaling to JSON (`null` vs `[]`).

The following example shows all three forms:

```go
func main() {
	var integers []int                                              // 1
	var stones = []string{"jagger", "richards", "wyman", "watts"}  // 2
	slice := []int{}                                               // 3
}
```
### make

`make` allocates a slice with a given type, length, and optional capacity. Pass a capacity when you know the approximate final size upfront. Pre-allocating avoids repeated reallocation as the slice grows.

`make` zeroes every element in the allocated backing array:

1. **Length only:** Capacity defaults to the given length.
2. **Length and capacity:** Allocates a backing array of size 10, but the slice can only access the first 8 elements until you append past the length.

```go
func main() {
	eight := make([]int, 8)       // 1
	tenCap := make([]int, 8, 10)  // 2
}
```

### new

`new` allocates a zeroed slice header and returns a pointer to it. The pointer points to a nil slice: length 0, capacity 0, and no underlying array.

The following example shows the zero state of a `*[]int` allocated with `new`:

```go
func main() {
	var zeroes *[]int = new([]int)
	fmt.Println(zeroes) // &[]
}
```

`new` is rarely used with slices in practice. Reach for a nil slice declaration or `make` instead.
## As function arguments

Pass slices by value. A slice header is only 24 bytes on 64-bit systems (8 bytes each for the pointer, length, and capacity), so copying it is cheap regardless of how large the underlying array is.

The following example passes a large slice without allocating a pointer:

```go
func main() {
	bigSlice := make([]int, 1e9)
	bigSlice = process(bigSlice)
}

func process(slice []int) []int {
	return slice
}
```

The callee receives its own copy of the slice header, but both headers point to the same underlying array. Writes to existing elements in the callee are visible to the caller. Only appends that exceed capacity are invisible, because `append` allocates a new array in that case.


## Variadic slices

The `...` operator has two related uses in Go. In a function signature, it declares a *variadic parameter* that accepts any number of arguments of the same type. At the call site, it unpacks a slice into individual arguments, which is also called *slice unpacking notation*.

The following example shows both uses, plus unpacking with `append`:

1. **Variadic parameter:** `args ...string` accepts zero or more string arguments.
2. **Slice unpacking at the call site:** `flag.Args()...` passes each element of the slice as a separate argument.
3. **Unpacking with `append`:** Appends all elements of `s2` into `s1`.

```go
func someFunc(r io.Reader, args ...string) {} // 1

func main() {
	someFunc(os.Stdin, flag.Args()...) // 2

	s1 := []int{1, 2}
	s2 := []int{3, 4}
	combined := append(s1, s2...) // 3 [1 2 3 4]
}
```

## Functions

### len

`len(slice)` returns the number of elements in the slice:

```go
func main() {
	var stones = []string{"jagger", "richards", "wyman", "watts"}
	fmt.Println(len(stones)) // 4
}
```

### cap

`cap(slice)` returns the capacity of the slice’s underlying array. When a slice grows beyond its capacity, the runtime allocates a new, larger array. For small slices, the runtime typically doubles the capacity; the exact growth factor depends on the Go version and element size.

```go
func main() {
	tenCap := make([]int, 8, 10)
	fmt.Println(cap(tenCap)) // 10
}
```

### copy

`copy(dst, src)` copies elements from `src` into `dst` and returns the number of elements copied. If `dst` and `src` have different lengths, `copy` copies up to the shorter length. Use `copy` to duplicate a slice into an independent backing array, for example to avoid mutating shared memory.

### append

`append(slice, value)` appends one or more values to the end of a slice and returns the updated slice. Always assign the return value back to the slice variable.

When the slice has no remaining capacity, `append` allocates a new underlying array, copies existing elements, and appends the new value. The original slice is unaffected. The following example shows this growth in action in the "Append" subsection under Modifying.

## Accessing elements

Access individual elements by index using `slice[i]`. Indices are zero-based, and Go panics at runtime if you access an out-of-bounds index.

The following example accesses a single element:

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}
	third := numbers[2] // 2
}
```

### for loop

A classic C-style loop works well when you need the index to compute something or modify elements in place:

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}

	for i := 0; i < len(numbers); i++ {
		fmt.Println(i * 2)
	}
}
```

### for ... range

`range` returns the index and a copy of the value at each position. It is the idiomatic way to iterate over a slice when you do not need to modify elements:

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}

	for i, v := range numbers {
		fmt.Printf("index: %d, value: %v\n", i, v)
	}
}
```

If you only need the values, discard the index with `_`:

```go
for _, v := range numbers {
	fmt.Println(v)
}
```

## Slicing

A *slice expression* extracts a portion of a slice without copying the underlying data. The result shares the same backing array as the original.

The syntax is:

`slice[start:end:capacity]`

*start*
: Inclusive. The index of the first element in the new slice. Defaults to `0` if omitted.

*end*
: Non-inclusive. The index where the slice stops. The new slice ends at `end - 1`. Defaults to `len(slice)` if omitted.

*capacity*
: The capacity of the new slice. Defaults to `cap(slice) - start` if omitted.

The following example shows common slice expressions:

```go
func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6}

	firstSecond := slice[1:3] // [1 2]
	noStart := slice[:4]      // [0 1 2 3]
	noEnd := slice[2:]        // [2 3 4 5 6]
	full := slice[:]          // [0 1 2 3 4 5 6]
}
```

### Calculating slices

The three-index form `slice[start:end:capacity]` lets you control the capacity of the resulting slice, which limits how far `append` can grow before allocating a new array. The values follow these formulas:

| Value | Role | Formula |
| :---- | :--- | :------ |
| `start` | First element, inclusive | |
| `end` | Last element, non-inclusive | `start + number of elements you want` |
| `capacity` | Size of the new slice's backing window | `start + number of elements to include in capacity` |

The following example creates a slice with length 1 and capacity 2 from a larger slice:

```go
source := []int{0, 1, 2, 3, 4}
newSlice := source[2:3:4]
// Length:   3 - 2 = 1
// Capacity: 4 - 2 = 2
```


## Modifying

A slice is mutable. You can assign to existing elements by index or grow the slice with `append`.

### Index

Assign a new value to any element by index. This replaces the existing element in the underlying array:

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}
	numbers[4] = 7 // [0 1 2 3 7 5 6]
}
```

### Append

`append` adds one or more elements of the same type to the end of a slice:

1. **Single element:** Appends one value.
2. **Multiple elements:** Appends several values in one call.
3. **Another slice:** Create a second slice, then unpack it with `...` to append all its elements.

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}

	numbers = append(numbers, 7)          // 1 [0 1 2 3 4 5 6 7]
	numbers = append(numbers, 8, 9)       // 2 [0 1 2 3 4 5 6 7 8 9]
	newNums := []int{10, 11, 12, 13}      // 3
	numbers = append(numbers, newNums...) //   [0 1 2 3 4 5 6 7 8 9 10 11 12 13]
}
```

### Insert

#### In the middle

To insert an element without replacing an existing one, combine two `append` calls:

1. Declare the original slice.
2. Open a gap at the target index by appending the tail onto a truncated head. `numbers[:2+1]` reserves one extra slot at index 2, giving `[0 1 2 2]`. Appending `numbers[2:]...` fills in the tail, producing `[0 1 2 2 3 4 5 6]`.
3. Assign the new value into the reserved slot at index 3.

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}          // 1
	numbers = append(numbers[:2+1], numbers[2:]...) // 2
	numbers[3] = 99                                 // 3 [0 1 2 99 3 4 5 6]
}
```

#### At the beginning

To prepend a value, create a one-element slice literal and unpack the original slice into it:

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}
	numbers = append([]int{99}, numbers...) // [99 0 1 2 3 4 5 6]
}
```

#### Multiple elements

Inserting multiple elements requires saving the tail first, because `append` modifies the underlying array in place and would overwrite elements you still need:

1. Declare the original slice.
2. Declare the values to insert.
3. Save the tail — the elements that should follow the inserted values — into a new independent slice.
4. Truncate the original to the insertion point, then append the inserted values.
5. Append the saved tail.

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}     // 1
	inserted := []int{100, 101, 103}           // 2

	tail := append([]int{}, numbers[3:]...)    // 3
	numbers = append(numbers[:3], inserted...) // 4
	numbers = append(numbers, tail...)         // 5 [0 1 2 100 101 103 3 4 5 6]
}
```

### Remove elements

#### At the beginning or end

Removing the first or last element is a reslice operation:

1. Reslice from index `1` to drop the first element.
2. Reslice to `len(numbers) - 1` to drop the last element.

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}
	rmFirst := numbers[1:]             // 1 [1 2 3 4 5 6]
	rmLast := numbers[:len(numbers)-1] // 2 [0 1 2 3 4 5]
}
```

#### From the middle

To remove elements from the middle, append the head of the slice to the tail, skipping the elements to remove:

{{< admonition "" warning >}}
The following example is for demonstration purposes only. Running it produces unexpected output:

```go
[0 1 6 6 5 6]
[0 1 6 6]
```

`append` modifies the underlying array in place. Both `rmTwo` and `rmMore` share `numbers` as their backing array, so the second `append` overwrites memory that `rmTwo` is still pointing to.
{{< /admonition >}}

```go
func main() {
	numbers := []int{0, 1, 2, 3, 4, 5, 6}
	rmTwo := append(numbers[:2], numbers[3:]...)  // [0 1 3 4 5 6]
	rmMore := append(numbers[:2], numbers[5:]...) // [0 1 5 6]
}
```

## Concurrency safety

Slices are not safe for concurrent reads and writes. If multiple goroutines modify a slice simultaneously, protect access with a `sync.Mutex`:

1. Declare the mutex alongside the shared slice.
2. Lock the mutex at the start of any function that writes to the slice.
3. Perform the work.
4. Unlock the mutex when the work is complete.

```go
var shared []int = []int{1, 2, 3, 4, 5, 6}
var mutex sync.Mutex // 1

func increase(num int) {
	mutex.Lock() // 2
	fmt.Printf("[+%d a] : %v\n", num, shared) // 3
	for i := 0; i < len(shared); i++ {
		time.Sleep(20 * time.Microsecond)
		shared[i] = shared[i] + 1
	}
	fmt.Printf("[+%d b] : %v\n", num, shared)
	mutex.Unlock() // 4
}
```

With the mutex in place, only one goroutine can access the slice at a time:

```go
func main() {
	for i := 0; i < 5; i++ {
		go increase(i)
	}

	time.Sleep(2 * time.Second)
}
```

## Sorting

The `sort` package provides functions to sort slices of common types and custom types.

### int, float64, string

Sort these types with `sort.Ints`, `sort.Float64s`, and `sort.Strings`:

```go
func main() {
	integers := []int{2, 4, 1, 8, 3, 4}
	floats := []float64{3.14, 6.54, 33.4, 9.1}
	strs := []string{"zebra", "elephant", "giraffe", "cat", "apple"}

	sort.Ints(integers)   // [1 2 3 4 4 8]
	sort.Float64s(floats) // [3.14 6.54 9.1 33.4]
	sort.Strings(strs)    // [apple cat elephant giraffe zebra]
}
```

### Reverse sort

Sort the slice first, then reverse it with a loop that swaps elements from the outside in. The loop runs for `len/2` iterations, and each iteration swaps index `i` with its mirror index `len - 1 - i`:

```go
func main() {
	integers := []int{2, 4, 1, 8, 3, 6}

	sort.Ints(integers)

	for i := len(integers)/2 - 1; i >= 0; i-- {
		opp := len(integers) - 1 - i
		integers[i], integers[opp] = integers[opp], integers[i]
	}
}
```

### Slice method

`sort.Slice` sorts a slice using a `less` function you provide. The function receives two indices `i` and `j` and returns `true` if the element at `i` should come before the element at `j`. For ascending order, return `slice[i] < slice[j]`. For descending order, return `slice[i] > slice[j]`:

```go
func main() {
	floats := []float64{3.14, 6.54, 33.4, 9.1}

	sort.Slice(floats, func(i, j int) bool {
		return floats[i] > floats[j] // descending
	})
}
```

### Sort interface

Implementing `sort.Interface` on a custom slice type is more performant than `sort.Slice` and makes the sort reusable. The interface requires three methods:

```go
Len() int
Less(i, j int) bool
Swap(i, j int)
```

Because the methods must be attached to a named type, define a type alias for your slice:

1. Define the struct you want to sort.
2. Define a named slice type. This is named `ByAge` because it sorts by the `Age` field.

```go
type Person struct { // 1
	Name string
	Age  int
}

type ByAge []Person // 2
```

Implement the three interface methods on `ByAge`:

```go
func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
```

Pass a `ByAge`-cast slice to `sort.Sort`. Wrap it in `sort.Reverse` to invert the order:

```go
func main() {
	people := []Person{
		{"Sally", 21},
		{"Billy", 35},
		{"Rick", 10},
		{"Mufasa", 68},
		{"Luke", 44},
	}

	sort.Sort(ByAge(people))               // [{Rick 10} {Sally 21} {Billy 35} {Luke 44} {Mufasa 68}]
	sorted := sort.IsSorted(ByAge(people)) // true

	sort.Sort(sort.Reverse(ByAge(people))) // [{Mufasa 68} {Luke 44} {Billy 35} {Sally 21} {Rick 10}]
	sorted = sort.IsSorted(ByAge(people))  // false
}
```