---
title: "Types and collections"
weight: 40
description: >
  Working with basic types and collections in Go.
---

## Type system

Go is statically typed, which means that the compiler wants to know the type for every value in the program. This creates more efficient and secure code that is safe from memory corruption and bugs.

Types tell the compiler how much memory to allocate (its size) and what the memory represents. Some types get their representation based on the machine architecture (64- vs 32-bit).

### xUser-defined types

There are 2 ways to declare a user-defined type in Go:
1. Use the keyword `struct` to create a composite type:
```go
type user struct {
    name    string
    age     int64
    email   string
}
```
After you define the struct, you can declare a variable of the type with a value or zero value:
```go
// Idiomatic: declare zero-values with var
var bill user
```

You can also declare a struct literal, which is the declaration and values all in one:

```go
// add a comma after the last value
ryan := user{
    name:   "Ryan",
    age:    38,
    email:  "email@email.com",
}

// without the field names
ryan := user{"Ryan", 38, "email@email.com"}

// embedding user-defined types
type superuser struct {
    person      user
    password    string
}

boss := superuser{
    person: user{
        name:   "Ryan",
        age:    38,
        email:  "email@email.com",
    },
    password:   "pword",
}
```

2. Use an existing type as the specification for a new type:

```go
type Distance int64

// lets you add methods to a slice. This is useful when you want to decalure behavior around a built-in or refernce type
type List []string
```

### xMethods

Methods add behavior to user-defined types:

```go
func (r receiver) fName(p params) {}
```

The receiver binds the function to the specified type to create a method for the type. How the receiver is declared determines how Go operates on the type value. There are 2 types of receivers:

1. value. When you declare a method using a value receiver, the method will always be operating against a copy of the value used to make the method call

If adding or removing something from a value of this type need to create a new value, then use value receivers for your methods.

```go
func (u user) sendEmail()
ryan := user{"Ryan", 38, "email@email.com"}
ryan.sendEmail()
```
You can also call methods that are declared with a value receiver using a pointer:
```go
func (u user) sendEmail()
ryan := &user{"Ryan", 38, "email@email.com"}
ryan.sendEmail()
```
Go adjusts (dereferences) the pointer value to comply with the method's receiver. So, `sendEmail()` is operating against a copy of the value that `ryan` points to.

2. pointer. When you call a method declared with a pointer receiver, the value used to make the call is shared with the method. Pointer receivers operate on the actual value, and any changes are reflected in the value after the call.

Generally, use pointers when you are using nonprimitive values.

If adding or removing something from a value of this type need to mutate the existing value, then use value receivers for your methods.

```go
func (u *user) changeEmail(email string) {
    u.email = email
}
ryan := &user{"Ryan", 38, "email@email.com"}
ryan.changeEmail("new@email.com")
```
You can also call methods that are declared with a pointer receiver using a value:
```go
func (u *user) changeEmail(email string) {
    u.email = email
}
ryan := user{"Ryan", 38, "email@email.com"}
ryan.changeEmail("new@email.com")
```
Go adjusts (dereferences) the pointer value to comply with the method's receiver. So, `sendEmail()` is operating against a copy of the value that `ryan` points to.

### Reference types
When you decalure a reference type, the value is a header value that contains a pointer to the underlying data structure. The header contains a pointer, so you can pass a copy of any reference type and share the underlying data structure.

The following are reference types:
- slice
- map
- channel
- function types

### xInterfaces

Interfaces are types that declare behavior. They are implemented by user-defined types through methods. If a type implements an interface with a method, then a value of that user defined type can be assigned to values of the interface type.

Go uses implicit interfaces, which means that you do not have to declare that the method implements the interface, you just need to use the contract.

When you call a method that accepts an interface value, Go looks at the method set for the user-defined type and tries to find a method that implements the interface. The user-defined type is called the 'concrete type' because it provides the interface concrete behavior. For example:

```go 
// io.Reader interface
type Reader interface {
    Read(b []byte) (n int, err error)
}

// Stringer interface. `Stringer` is the name of the interface, and its signature is within the curly brackets. 
type Stringer interface {
    String() string
}
```
You can implement the `io.Reader` interface for a user-defined type if the function is:
- named `Read`
- accepts a slice of bytes ([]byte is a nil slice)
- returns an integer and an error

You can implement the `Stringer` interface if the function is:
- named `Stringer`
- returns a string


Interface values are two-word data structures:
1. A pointer to an internal table called iTable. iTable contains information about the user-defined stored value that implements the interface: the value's type and its list of methods.
2. A pointer to the actual stored value.

So, if you have a user that implements the `Stringer` interface:

```go
// user type
type user struct {
    name    string
    age     int64
    email   string
}


// user-defined interface 
type birthdater interface {
    birthday()
}

// user implements the Stringer interface
func (u user) String() string {
    return fmt.Sprintf("My name is %s", u.name)
}

// method with ptr receiver to update the user's email
func (u *user) updateEmail(newEmail string) {
    u.email = newEmail
}

// implement the birthdater interface
func (u *user) birthday() {
    u.age++
}

// user struct literal without field names
ryan := user{"Ryan", 38, "email@email.com"}
```
The Stringer iTable contains information about the user type, which includes its list of methods. It also contains a pointer to the memory address that stores the actual user value. You cannot swap value and pointer receiver semantics with interface implementation.

If you implement an interface using a pointer receiver, then only pointers of that type implement the interface:

```go

// user type
type user struct {
    name        string
    password    string
}

// superuser type
type superuser struct {
    person      user
    password    string
}

// user-defined interface 
type passUpdater interface {
    updatePassword(p string)
}

// user

```
## xLoops

In Go, every loop is a `for` loop:

```go
for i := 0; i < 5; i++ {
    // do somthing
}
```

Use the `for` keyword where other languages would use `while`:

```go
for iterator.Next() {}
for line != lastLine {}
for !gotResponse || response.invalid() {}
```

Create an infinite loop with only the `for` keyword:

```go
for {
    // loop forever
}
```

The `for range` loop iterates over an array, slice, map, or channel using an index and value:

```go
for index, value := range iterable {
    // do something
}
```
If you do not need the index, use the blank identifier:

```go
for _, value := range iterable {
    // do something
}
```
> The `for range` loop operates on a copy of the value, so do not expect the values to mutate.

## xArrays

### Internals

An array in Go is a fixed-length data type that contains a contiguous block of elements of the same type

Having memory in a contiguous form can help to keep the memory you use stay loaded within CPU caches longer
Since each element is of the same type and follows each other sequentially, moving through the array is consistent and fast

An array is declared by specifying the type of data to be stored and the total number of elements required, also known as the array’s length.
The type of an array variable includes both the length and the type of data that can be stored in each element

When you pass variables between functions, they’re always passed by value. When your variable is an array, this means the entire array, regardless of its size, is copied and passed to the function.
You can pass a pointer to the array and only copy eight bytes, instead of eight megabytes of memory on the stack
You just need to be aware that because you’re now using a pointer, changing the value that the pointer points to will change the memory being shared

Once an array is declared, neither the type of data being stored nor its length can be changed
they’re always initialized to their zero value for their respective type

An array is a value in Go. This means you can use it in an assignment operation
```go
var array1 [5]string
array2 := [5]string{"Red", "Blue", "Green", "Yellow", "Pink"}
array1 = array2
```

```go
var array [5]int                        // standard declaration
array := [5]int{10, 20, 30, 40, 50}     // array literal declaration
array := [...]int{10, 20, 30, 40, 50}   // Go finds the length based on num of elements
array := [5]int{1: 10, 2: 20}           // initialize specific elements

// pointers
array := [5]*int{0: new(int), 1: new(int)}  // array of pointers
array2 := [3]*string{new(string), new(string), new(string)}
// dereference to assign values
*array[0] = 10
*array[1] = 20
*array2[0] = "Red"
*array2[1] = "Blue"
```

## xSlices

The slice type is a dynamic array that can grow and shrink as you see fit. 

> **Slice vs. Array**
> If you specify a value inside the [] operator ([4]varName), then you are creating an array. If you do not specify a value, you create a slice.
> `array := [3]int{10, 20, 30}`
> `slice := []int{10, 20, 30}`
> 
> An _array_ is a value. They are of a fixed size, and the elements in the array are initialized to their zero value.
> 
> A _slice_ is a pointer to an array. It has no specified length, and its zero value is `nil`.
> 'Slicing' does not copy the slice's data, it creates a new slice value that points to the original array. So `s = slice[2:]` creates slice `s` that begins with second element of `slice`, and they both point to the same underlying data structure. So, modifying elements of a slice changes the  


### Internals

The slice has three fields:
- Pointer to the underlying array
- Length. Number of elements the slice can access from the array.
- Capacity. Size of the underlying array, or number of elements that the slice has available for growth. It is the 

You cannot create a slice with a capacity that's smaller than the length.

### Create slices 

Name slices with plural words. Create slices using the following methods:
- `new()` function.
- `make([]T, len, cap)` function.
- _Slice literals_. This is the idiomatic way to create slices. It requires that you define the contents when you create the slice.
- _nil slice_. A nil slice is declared without any initialization.  The most common way to create slices, and can be used with many of the standard library and built-in functions that work with slices.
- _empty slice_. Useful when you want to represent an empty collection, such as when a database query returns zero results.

```go
// make
slice := make([]string, 5)          // create a slice of strings with 5 capacity
slice := make([]int, 3, 5)          // length 3, cap 5

// slice literals
slice := []int{10, 20, 30}          // slice literal
slice := []string{99: ""}           // initialize the index that represents the length and capacity you need

// nil slice
var slice []int                     // nil slice

// empty slice
slice := make([]int, 3)             // empty slice with make
slice := []int{}                    // slice literal to create empty slice of integers
```

### Functions

_copy(dest, src)_ 
: Copies the contents of the `src` slice into the `dest` slice. The `dest` and `src` slices must share the same underlying array. This is commonly used to increase the capacity of an existing slice.
: If `dest` and `src` are different lengths, it copies up to the smaller number of elements. 

_append(slice, value)_
: Appends `value` to the end of `slice`.
: When there’s no available capacity in the underlying array for a slice, the `append` function creates a new underlying array, copies the existing values that are being referenced, and assigns the new value. So, if you append to the 3rd index of a slice with length 2, you get a new underlying array of length 3 with a capacity doubled the original array.

_len(slice)_
: Returns the length of `slice`.

_cap(slice)_
: Returns the capacity of `slice`. When you expand a slice beyond its original capacity, the capacity is always doubled when the existing capacity of the slice is under 1,000 elements. 



```go
slice := []string{"Apple", "Orange", "Plum", "Banana", "Grape"}

capacity := cap(slice)              // 5
length := len(slice)                // 5


slice = append(slice, "Kiwi")
capacity = cap(slice)               // 10
length = len(slice)                 // 6

```

### Slicing (Working with indices)

Use the following syntax to 'slice' a slice into a new slice:

`slice`[_start_:_end_:[ _capacity_] ]

_start_
: Inclusive. The index position of the element that the new slice begins with. For example, `newTest := test[1:]` means "take everything from element at index 1 to the end of the slice, and place it in `newTest`.

_end_
: Non-inclusive. The index of the existing slice where you stop copying values to the new slice. The new slice ends with the value at `[end-1]`.

_capacity_
: The capacity for the new slice. When you append to a slice that goes beyond the slice capacity, the capacity is doubled when its length is less than 1,000.

#### Calculating slices

The start, end, and capacity values have a formula that you can use to correctly calculate slicing. For example, the following slice creates a new slice with 1 element, and the capacity of 2:

```go
newSlice := slice[2:3:4]
```
| Value | Description                                                                  | Formula                                                |
| :---- | :--------------------------------------------------------------------------- | :----------------------------------------------------- |
| 2     | start. The first element in the original slice, inclusive.                   |                                                        |
| 3     | end. The index of the existing slice where the slicing stops, non-inclusive. | start + number of elements you want in the slice.      |
| 4     | capacity. Size of the new slice.                                             | start + number of elements to include in the capacity. |

#### Examples

```go

slice := []int{10, 20, 30, 40, 50}
newSlice := slice[:4]               // newSlice == {slice[0], slice[1], slice[2], slice[3]}.
newSlice := slice[1:3]              // {20, 30}
newSlice[1] = 31                    // {20, 31}

// start and end indices
s := []int{0, 1, 2, 3, 4}
l := s[1:3]
fmt.Println(s, l)   // [0 1 2 3 4] [1 2]

l[0] = 8
fmt.Println(s, l)   // [0 8 2 3 4] [8 2]

// Slicing length and capacity
slice := []int{10, 20, 30, 40, 50}  // Length:   5
                                    // Capacity: 5

newSlice := slice[1:3]              // Length:   3 - 1 = 2
                                    // Capacity: 5 - 1 = 4



// Slice the third element and restrict the capacity.
// Contains a length of 1 element and capacity of 2 elements.
slice := source[2:3:4]
```

You can use the built-in function called append, which can grow a slice quickly with efficiency. You can also reduce the size of a slice by slicing out a part of the underlying memory

### Variadic slices:

The built-in function append is also a variadic function. Use the ... operator to append all the elements of one slice into another.

```go
s1 := []int{1, 2}
s2 := []int{3, 4}

// Append the two slices together and display the results.
fmt.Printf("%v\n", append(s1, s2...))

Output:
[1 2 3 4]
```

Use the `...` operator to expand a slice into a list of values:

```go
// accepts any variable number of string args
func getFile(r io.Reader, args ...string) {}
..
// the ... operator expands a slice into a list of values
t, err := getFile(os.Stdin, flag.Args()...) {}
```

### Iterating over slices

Use `for` with `range` to iterate over slices from the beginning.

> **IMPORTANT**
> Do not use pointers (`&value`) when you iterate with `range` because it returns the index and a copy of the value for each iteration, not a reference. A pointer is an address that contains the copy of the `value` that is being copied.

```go
for index, value := range <slice-name> {
    fmt.Printf("index: %d, value: %d", index, value)
}

// discard the index with a '_'
for -, value := range <slice-name> {
    fmt.Printf("value: %d", value)
}
```

To iterate over a slice from an index other than 0, use a traditional `for` loop:

```go 
for i := 2; i < len(slice); i++ {
    fmt.Printf("index: %d, value: %d", index, slice[i])
}
```

Here, we use `append` so we don't have to deal with capacity. In the first function, we create a slice with the capacity equal to the number of variadic arguments passed to the function. Then, we use a `for...range` loop to iterate over the arguments, assigning the sum of each argument to an index in the return slice.

If we use `append`, we can just declare a slice, then range over arguments and append the sum of each argument to the slice:

```go
// before refactor
func SumAll(numbersToSum ...[]int) []int {
	lengthOfNumbers := len(numbersToSum)
	sums := make([]int, lengthOfNumbers)

	for i, numbers := range numbersToSum {
		sums[i] += Sum(numbers)
	}
	return sums
}

// after
func SumAll(numbersToSum ...[]int) []int {
	var sums []int

	for _, numbers := range numbersToSum {
		sums = append(sums, Sum(numbers))
	}

	return sums
}
```


### Sorting slices 

The Go `sort` package has a [`Slice` method](https://pkg.go.dev/sort#Slice) to sort the values in a slice. It compares items a two indices, and returns whether the item at the first index should be placed before the item at the second index.

For example, the following function sorts a slice of type `Book {Author, Title}` first by `Author`, then by `Title`:

```go
func sortBooks(books []Book) []Book {
	sort.Slice(books, func(i, j int) bool {
		if books[i].Author != books[j].Author {
			return books[i].Author < books[j].Author
		}
		return books[i].Title < books[j].Title
	})
	return books
}
```

### Passing slices between functions

Pass slices by value, because slices only contain a pointer to the underlying array, its length, and capacity. This is why slices are great--no need for passing pointers.

On 64-bit machines, each component of the slice requires 8 bytes (24 total).

```go
bigSlice := make([]int, 1e9)

slice = fName(slice)

func fName(slice []int) []int {
    return slice
}
```

## xMaps

A map provides you with an unordered collection of key/value pairs. maps are unordered collections, and there’s no way to predict the order in which the key/value pairs will be returned because a map is implemented using a hash table
The map’s hash table contains a collection of buckets. When you’re storing, removing, or looking up a key/value pair, everything starts with selecting a bucket. This is performed by passing the key—specified in your map operation—to the map’s hash function. The purpose of the hash function is to generate an index that evenly distributes key/value pairs across all available buckets.

The strength of a map is its ability to retrieve data quickly based on the key. They do not have a capacity or a restriction on growth.
Use len() to get the length of the map
The map key can be a value from any built-in or struct type as long as the value can be used in an expression with the == operator. You CANNOT use:
- slices
- functions
- struct types that contain slices

### Creating and initializing

To create a map, use `make` or a map literal. The map literal is idiomatic:

```go
// create with make
dict := make(map[string]int)

// create and initialize as a literal IDIOTMATIC
dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}

// slice as the value
dict := map[int]string{}

// assigning values with a map literal
colors := map[string]string{}
colors["Red"] = "#da137"

// DO NOT create nil maps, they result in a compile error
var colors map[string]string{}

```

### Finding keys with ok

Maps return Boolean values that indicate whether a key exists in a map. The common Go idiom is to name this Boolean `ok`.

> Map keys must be comparable and hashable. This means you cannot use a slice, map, or function.

The following example searches a map and returns whether the key `"blue"` exists in the map:

```go
val, ok := mapname["blue"]
if ok {...}

// compact version
if val, ok := mapname["blue"]; ok {
    // ...
}
```

Some Go code uses the word `exists` or `found` in place of `ok`:
```go
value, exists := colors["Blue"]

if exists {
    fmt.Println(value)
}
```
Return the value and test for the zero value to determine if the key if found:
```go
value, found := colors["Blue"]

if value != "" {
    fmt.Println(value)
}
```

### Iterating over maps with the for range loop

This works the same as slices, except index/value -> key/value:

```go
// Create a map of colors and color hex codes.
colors := map[string]string{
    "AliceBlue":   "#f0f8ff",
    "Coral":       "#ff7F50",
    "DarkGray":    "#a9a9a9",
    "ForestGreen": "#228b22",
}

// Display all the colors in the map.
for key, value := range colors {
    fmt.Printf("Key: %s  Value: %s\n", key, value)
}
```

Use the built-in function `delete` to remove a value from the map:

```go
delete(colors, "Coral")
```

### Passing maps to functions

Functions do not make copies of the map. Any changes made to the map by the function are reflected by all references to the map:

```go
func main() {
    // Create a map of colors and color hex codes.
    colors := map[string]string{
       "AliceBlue":   "#f0f8ff",
       "Coral":       "#ff7F50",
       "DarkGray":    "#a9a9a9",
       "ForestGreen": "#228b22",
    }

    // Call the function to remove the specified key.
    removeColor(colors, "Coral")

    // Display all the colors in the map.
    for key, value := range colors {
        fmt.Printf("Key: %s  Value: %s\n", key, value)
    }
}

// removeColor removes keys from the specified map.
func removeColor(colors map[string]string, key string) {
    delete(colors, key)
}
```

```go
// create with make
dict := make(map[string]int)

// create and initialize as a literal IDIOTMATIC
dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}

// slice as the value
dict := map[int]string{}

// assigning values with a map literal
colors := map[string]string{}
colors["Red"] = "#da137"

// DO NOT create nil maps, they result in a compile error
var colors map[string]string{}

// map with a struct literal value
var testResp = map[string]struct {
	Status int 
	Body string 
} {
	//...
}
```

## Strings

Initialize a buffer with string contents using the bytes.NewBufferString("string") func. This simulates an input (like STDIN):
```go
b := bytes.NewBufferString("string")
```

Use `io.WriteString` to write a string to a writer as a slice of bytes:
```go
output, err := io.WriteString(os.Stdout, "Log to console")
if err != nil {
    log.Fatal(err)
}
```
This command seems to be used a lot with the `exec.Command` `os/exec` package?

`.TrimSpace()` removes whitespace, `\n`, `\t`:
```go
func main() {
	fmt.Println(strings.TrimSpace(" \t\n Hello, Gophers \n\t\r\n"))
}
```
You can build strings using `fmt.Sprintf()`:
```go
u := fmt.Sprintf("%s/todo/%d", apiRoot, id)
```

## Pointers

`*` either declares a pointer variable or dereferences a pointer. Dereferencing is basically following a pointer to the address and retrieving stored value.

`&` accesses the address of a variable. Use this for the same reasons that you use a pointer receiver: mutating the object or in place of passing a large object in memory.

Here are some bad examples:

```go
func main() {
	test := "test string"
	var ptr_addr *string
	ptr_addr = &test
	fmt.Printf("ptr_addr:\t%v\n", ptr_addr)
	fmt.Printf("*ptr_addr:\t%v\n", *ptr_addr)
	fmt.Printf("test:\t\t%v\n", test)
	fmt.Printf("&test:\t\t%v\n", &test)
}

// output
ptr_addr:	0xc00009e210
*ptr_addr:	test string
test:		test string
&test:		0xc00009e210
```

## Equality

You can't use `==` to check whether two objects are equal. For any type, use `reflect.DeepEqual`. For slices, use `slices.Equal`.

Here is `reflect.DeepEqual`:

```go
func TestSumAll(t *testing.T) {
	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```
`reflect.DeepEqual` is not type safe--you can compare variables of different types without compilation errors--so be careful.

