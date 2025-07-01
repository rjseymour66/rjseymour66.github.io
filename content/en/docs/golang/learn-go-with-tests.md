---
title: "Learn with tests"
# linkTitle: ""
weight: 1
description: >
  [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests)
---

## Install go

This script will get the lastest version of Go and install it in `/usr/local`:

```bash
#!/bin/bash

# Upgrades the Go binary to the version specified as the first argument passed
# to this script.

# Check if the version argument was passed
if [ -z "$1" ]; then
	echo "Usage: $0 <go-version>"
	echo "Example: $0 1.24.4"
	exit 1
fi

VERSION="$1"
TARBALL="go${VERSION}.linux-amd64.tar.gz"
URL="https://go.dev/dl/${TARBALL}"

echo "Removing previous installation from /usr/local/go..."
rm -rf /usr/local/go

echo "Downloading go tarball from $URL..."
wget "$URL"

echo "Extracting the tarball to /usr/local..."
tar -C /usr/local -xzf "${TARBALL}"

echo "Cleaning up (deleting the tarball)..."
rm -v ${TARBALL}

echo
echo
echo "Verify the installation with 'go version'"
```

## Big ideas

- Each function gets a test.
- Use subtests in each test to check different cases.
- Use helpers in tests to reduce duplicated code.
- Benchmark loops with `-bench` and `-benchmem`
- Use `go test -cover` to check your test coverage.

## go mod

https://go.dev/doc/modules/gomod-ref

If you plan to distribute your application, you need to tell others where your code is available for download. Thats what `go mod` does--it gives the name of the module and the download URL:

```bash
go mod init <path/to/module-name.com>
```

## Writing Tests

When you write tests, you are using the compiler as a feedback mechanism. Here is the feedback loop:
1. Write a test.
2. Write code to make the compiler pass.
3. Write another test.
4. Run the test, see that it fails and make sure the error message is meaningful.
5. Write code to make the compiler pass.
6. Refactor.

This makes sure that you are writing tested code with relative tests that are easier to debug when they fail.

### Placeholder strings

https://pkg.go.dev/fmt#hdr-Printing


## Structs

A method is a function with a reciever. Its declaration binds the method name (the identifier) to a method and associates that method with the receiver's base type.

A method has to be invoked on an instance of this base type. When you call the method on an instance, you get a reference to the instance's data through the receiver variable. The receiver variable is similar to `this` in other programming languages. By convention, the receiver variable is the first letter of the Type.

```go
// base type
type Rectangle struct {
    Width  float64
    Height float64
}

// receiver gives access to its data
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}
```

You can also make type from an existing type. This helps make the code more domain-speific. You can also add methods and interfaces to these new types:

```go
type Bitcoin int

bc := Bitcoin(10)

// Stringer method
func (b Bitcoin) String() string {
    return fmt.Sprintf("%d BTC", b)
}
```

> The `Stringer` method lets you define what your type looks like when its output as a string.

## Interfaces

An interface _decouples_ functions from concrete types. By decoupling types from behavior, an interface helps you declare the _behavior that you need_ rather than the type you need.

Table tests are useful for testing intefaces. For example, when you implement an interface with a new type, you can add the new type as a test case to the table test.

## Table tests

When you create a table test, name the fields in the anonymous struct, and include a `name` field so you can name each test. This helps identify specific tests in the output. The anonymous struct should include the following:

- name for each subtest
- what you want to test
- "want" field

When you run the tests with a `for...range` loop, use `name` field as the test name for each `t.Run` subtest:

```go
func TestArea(t *testing.T) {
    areaTests := []struct {
        name    string
        shape   Shape
        hasArea float64
    }{
        {name: "Rectangle", shape: Rectangle{12, 6}, hasArea: 72.0},
        {name: "Circle", shape: Circle{10}, hasArea: 314.1592653589793},
        {name: "Triangle", shape: Triangle{12, 6}, hasArea: 36.0},
    }

    for _, tt := range areaTests {
        t.Run(tt.name, func(t *testing.T) {
            got := tt.shape.Area()
            if got != tt.hasArea {
                t.Errorf("#%#v got %g want %g", tt.shape, got, tt.hasArea)
            }
        })
    }
}
```

## Pointers

- Pointers help you manage state.
- Keep your method receiver types consistent. If one method uses a pointer type, use a pointer type for all, even if they do not need it.
- When a function returns a pointer, you need to check whether its nil

When you use pointers, you don't have to dereference the pointer in the function. For example:

```go
func (w *Wallet) Balance() int {
    return w.balance                // not return (*w).balance
}
```

Here, you can return the correct wallet instance without dereferencing. (you can also dereference the pointer, but it is not necessary.) The creators of Go didn't like the syntax, so they don't make us dereference (they are [automatically dereferenced](https://go.dev/ref/spec#Method_values)).

## Errors

Helpful error linter:

```go
go install github.com/kisielk/errcheck@latest
// run in working dir
errcheck .
```

Link: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

This creates an error with your custom error message:

```go
errors.New("error msg")
```

You can also convert an error to a string message to confirm that it is the error that you want:

```go
// tested function
func (w *Wallet) Withdraw(amount Bitcoin) error {
    if amount > w.balance {
        return errors.New("cannot withdraw, insufficient funds")
    }
    w.balance -= amount
    return nil
}

assertError := func(t testing.TB, got error, want string) {
    t.Helper()

    if got.Error() != want {    // compare got string to want string
        t.Errorf("got %q, want %q", got, want)
    }
}
```

Handling errors like this is tedious. If you want to change the error message, you have to change it in multiple places.

Its much easier to define a meaningful error value (errors are values in Go) that you can reference throughout your codebase:

```go
var ErrInsufficientFunds = errors.New("cannot withdraw, insufficient funds")

func (w *Wallet) Withdraw(amount Bitcoin) error {
    if amount > w.balance {
        return ErrInsufficientFunds
    }
    ...
}
```

Create a type for your errors and implement the `error` interface. Then, you can create `constant` errors, which makes them more reusable and immutable:

```go
const (
    ErrNotFound   = DictionaryErr("could not find the word you were looking for")
    ErrWordExists = DictionaryErr("cannot add word because it already exists")
)

type DictionaryErr string

func (e DictionaryErr) Error() string {
    return string(e)
}
```

An idiomatic way to check your errors is with the `.Is()` or `.As()` error methods, but you can also use a `switch` statement:

```go
func (d Dictionary) Delete(word string) error {
    _, err := d.Search(word)

    switch err {
    case ErrNotFound:
        return ErrWordDoesNotExist
    case nil:
        delete(d, word)
    default:
        return err
    }

    return nil
}
```

## ok

Maps can return two values. Idiomatically, you can check if a map contains a value with the `ok` keyword:

```go
func (d Dictionary) Search(word string) (string, error) {
    definition, ok := d[word]
    if !ok {
        return "", errors.New("could not find the word you were looking for")
    }
    return definition, nil
}
```

## Maps

You can mutate a map without passing their address. This is because a map is a pointer to a [runtime.hmap structure](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it). When you copy a map, you aren't copying the data structure, you're copying the pointer to the data structure.

All this means that you can initialize a `nil` map, but you DO NOT want to do that because it results in a runtime panic. Initialize an empty map or use the `make` keyword:

```go
var dictionary = map[string]string{}

// OR

var dictionary = make(map[string]string)
```

If you add a value with a key that already exists, the map does not create duplicate entries. It overwrites the old value with the new value.

You can delete items from a map with the built-in function `delete`. It takes the map and the key to remove, and it returns nothing:

```go
delete(mapName, key)
```