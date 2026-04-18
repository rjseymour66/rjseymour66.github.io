+++
title = 'Interfaces'
date = '2025-08-23T17:46:10-04:00'
weight = 40
draft = false
+++

Interfaces are types that declare behavior with a set of methods. Types do not have to explicitly declare that they implement an interface. You can pass types to a function based only on its methods, and Go checks at compile time whether the type matches the interface definition.

To implement an interface in Go, a user-defined type must a method set with signatures that exactly match the methods declared in the interface. If a type implements an interface, then a value of that type can be assigned to values of the interface type.

{{< admonition "" note >}}
If a type implements an interface, then it can be referred to as an _interface_-er. For example, if a type implements the `Reader` method, you might see it called a "Reader". If it implements the `Writer` method, it is called a "Writer".
{{< /admonition >}}

When you call a method that accepts an interface value, Go looks at the method set for the user-defined type and tries to find a method that implements the interface. The user-defined type is called the 'concrete type' because it provides the interface concrete behavior. For example, here are the `io.Reader` and `Stringer` interfaces with their signatures between the curly braces:

```go 
type Reader interface {
    Read(b []byte) (n int, err error)
}

type Stringer interface {
    String() string
}
```

A type implements the `io.Reader` interface if the following conditions are met:
- It has a method named `Read`
- This `Read` method accepts a slice of bytes (`[]byte` is a nil slice)
- The function returns an `integer` and an `error`

A type implements the `Stringer` interface if the following conditions are met:
- It has a method named `Stringer`
- The function returns a `string`

The following example implements the `Stringer` interface:

```go
type user struct {
    name    string
    age     int64
    email   string
}

func (u user) String() string {
    return fmt.Sprintf("My name is %s", u.name)
}
```

## Implementing an interface

To implement an interface, you only need to add a method with the same signature as the interface. For example, define a `Worker` interface with a single `Work` method:

```go
type Worker interface {
	Work()
}
```

Any type that implements this interface is called a "Worker". The `Pay` function takes a `Worker`, calls the Worker's `Work` method, and prints a message:

```go
func Pay(w Worker) {
	w.Work()
	fmt.Println("and getting paid!")
}
```

If you want to call the `Pay` method on a custom type, you need to implement the `Worker` interface. Here, the `Person` type has a `Work` method, which means that `Person` implements the `Worker` interface---`Person` is a `Worker`:

```go
type Person struct {
	Id    int
	Email string
}

func (person Person) Work() {
	fmt.Println("Working hard...")
}
```

Now, you can pass a `Person` to the `Pay` function:

```go
func main() {
	person := Person{1, "example@email.com"}
	Pay(person)
}
```

## Internals

Interface values are two-word data structures:
1. A pointer to an internal table called iTable. iTable contains information about the user-defined stored value that implements the interface---the value's type and its list of methods.
2. A pointer to the actual stored value.

Using the example in the previous section, the `Stringer` iTable contains information about the user type, which includes its list of methods. It also contains a pointer to the memory address that stores the actual user value. You cannot swap value and pointer receiver semantics with interface implementation.

If you implement an interface using a pointer receiver, then only pointers of that type implement the interface.

## Testing with interfaces

Interfaces help you decouple your code from internal or external libraries so you can test without interacting with the library.

You need to test how your code interacts with an external API without calling the API during testing. You can create an interface with a subset of the API's methods and test against the interface. For example, if the API has a `Charge` method, do the following:
1. Declare a `Charger` interface with a `Charge` method.
2. Declare a test-only type `FakeCharger` with a `Charge` method.
3. Update your code to accept the `Charger` interface. This is a benefit of Go's interface system: you can declare an interface as needed.
4. During testing, pass the fake concrete type to the code.


## Standard interfaces

When possible, it is practical to implement interfaces in the standard library to make it easier to integrate our code with the existing functionality of the standard library and third-party software. 

### Stringer

Implement the Stringer interface to give a type a human-readable string representation that the standard library can use. For example, you can use a Stringer interface for logging, errors, and testing.

The Stringer interface has a single `String` method that returns a `string`:

```go
type Stringer interface {
	String() string
}
```

Here, we implement Stringer for a `URL` type:
```go
// A URL represents a parsed URL.
type URL struct {
	Scheme string
	Host   string
	Path   string
}

func (u *URL) String() string {
	return fmt.Sprintf("%s://%s/%s", u.Scheme, u.Host, u.Path)
}
```