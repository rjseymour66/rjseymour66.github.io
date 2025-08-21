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

## Dependency injection

Dependency injection means thta you can inject (pass in) a dependency at runtime. The dependencies--like a DB connection or printing to STDOUT--is passed into the function rather than being hardcoded. Dependency injection lets you write general-purpose functions, and it faciliates testing.

One way to do this is to pass an interface rather than a concrete type:

```go
func Greet(writer io.Writer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}
```

Use `bytes.Buffer{}` to test anything with the `io.Writer` interface.

## Mocking

### Test spies

A testing spy is a test double (like a mock or stub) that records information about how it's used, so your test can make assertions about:

- What methods were called
- How many times
- With what arguments
- And in what order
- A spy is a kind of test double that records what happened.
- Use it when you need to assert behavior, not just results.
- It’s ideal for testing side effects and interactions in Go, especially when paired with interfaces and dependency injection.

What makes it a spy:

- Behaves like the real dependency (implements the same interface)
- Tracks internal state or call history
- Is inspected after the test to verify behavior

Unlike mocks, spies typically don’t enforce expectations upfront (e.g., "this method must be called once"). You test after the fact.

Use a spy to test side effects and interactions.

- For example:
- Did we call Sleep() 3 times?
- Did we write to a file before sleeping?
- Did a logger get used properly?

## Concurrency

Use concurrency for the part of the code that you want to run faster. The other parts should run linearly.

### Race condition

A race condition is a bug that occurs when software is dependent on the timing and sequence of events. Go has a built-in race detector:

```bash
go test -race
```

### Channels

Channels can help solve data race condiitons.

- send expressions send values to a channel
- receive expressions assign a value in a channel to a variable

```go
// send expression
resultChannel <- result{u, wc(u)}

// receive expression
r := <-resultChannel
```
## Templates

[Calhoun.io blogs](https://www.calhoun.io/intro-to-templates-p1-contextual-encoding/) have helpful information about templates.

The basic format of all these examples:

- the program parses the template
- uses the template to render a post to any `io.Writer`

Basic template:

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.New("blog").Parse(postTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, p); err != nil {
        return err
    }

    return nil
}
```

Embedded in FS. This lets you load multiple templates and combine them.

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return err
    }

    if err := templ.Execute(w, p); err != nil {
        return err
    }

    return nil
}
```

With multiple templates, where you import into other files:

```go
func Render(w io.Writer, p Post) error {

    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return err
    }

    if err := templ.ExecuteTemplate(w, "blog.gohtml", p); err != nil {
        return err
    }

    return nil
}
```

### Benchmarking templates

A program must parse the template files repeatedly, and this affects performance. Here is the benchmark test before we refactor:

```bash
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: learngotests/renderer
cpu: Apple M3 Pro
BenchmarkRender-11        149312          8087 ns/op
PASS
ok      learngotests/renderer   1.455s
```

To improve this, we create a type that holds the parsed template and has a method to rerender the template:

```go
var (
    //go:embed "templates/*"
    postTemplates embed.FS
)

type Post struct {
    Title       string
    Body        string
    Description string
    Tags        []string
}

type PostRenderer struct {
    templ *template.Template
}

func NewPostRenderer() (*PostRenderer, error) {
    templ, err := template.ParseFS(postTemplates, "templates/*.gohtml")
    if err != nil {
        return nil, err
    }

    return &PostRenderer{templ: templ}, nil
}

func (r *PostRenderer) Render(w io.Writer, p Post) error {

    if err := r.templ.ExecuteTemplate(w, "blog.gohtml", p); err != nil {
        return err
    }

    return nil
}
```

Here are the tests:

```go

func TestRender(t *testing.T) {
    var (
        aPost = Post{
            Title:       "hello world",
            Body:        "This is a post",
            Description: "This is a description",
            Tags:        []string{"go", "tdd"},
        }
    )

    postRenderer, err := NewPostRenderer()

    if err != nil {
        t.Fatal(err)
    }

    t.Run("it converts a single post into HTML", func(t *testing.T) {
        buf := bytes.Buffer{}

        if err := postRenderer.Render(&buf, aPost); err != nil {
            t.Fatal(err)
        }

        approvals.VerifyString(t, buf.String())
    })
}

func BenchmarkRender(b *testing.B) {
    var (
        aPost = Post{
            Title:       "hello world",
            Body:        "This is a post",
            Description: "This is a description",
            Tags:        []string{"go", "tdd"},
        }
    )

    postRenderer, err := NewPostRenderer()

    if err != nil {
        b.Fatal(err)
    }

    for b.Loop() {
        postRenderer.Render(io.Discard, aPost)
    }

}
```

Here are the post-refactor benchmarks:

```go
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: learngotests/renderer
cpu: Apple M3 Pro
BenchmarkRender-11       1847904           629.1 ns/op
PASS
ok      learngotests/renderer   1.434s
```

### FuncMaps

Before you parse a template, you can use a `FuncMap` to define functions that are called within your template. Here, we define a `sanitizeTitle` function between the `New` and `Parse` template methods. Our template string calls the function on the first title occurence:

```go
func (r *PostRenderer) RenderIndex(w io.Writer, posts []Post) error {
    indexTemplate := `<ol>{{range .}}<li><a href="/post/{{sanitizeTitle .Title}}">{{.Title}}</a></li>{{end}}</ol>`

    templ, err := template.New("index").Funcs(template.FuncMap{
        "sanitizeTitle": func(title string) string {
            return strings.ToLower(strings.Replace(title, " ", "-", -1))
        },
    }).Parse(indexTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, posts); err != nil {
        return err
    }

    return nil
}
```

The problem with this approach is that you can only test the sanitize the function by generating HTML output. You can't test the actual function itself. Also, you should avoid logic in templates at all costs.

The solution is to use a ["view model"](https://stackoverflow.com/questions/11064316/what-is-viewmodel-in-mvc/11074506#11074506), which is a type that represents the data that you want to display on the page and can be saved to a database. A view model is different from the domain model because it contains only information that you want to display in the UI.

To fix this, we create a view model that contains testable data and logic for our view:

```go
type PostViewModel struct {
    Title, SanitizedTitle, Description, Body string
    Tags                                     []string
}
```

Here, we call the `.SanitizedTitle` function in place of the URL title:

```go
func (r *PostRenderer) RenderIndex(w io.Writer, posts []Post) error {
    indexTemplate := `<ol>{{range .}}<li><a href="/post/{{.SanitizedTitle}}">{{.Title}}</a></li>{{end}}</ol>`

    templ, err := template.New("index").Parse(indexTemplate)
    if err != nil {
        return err
    }

    if err := templ.Execute(w, posts); err != nil {
        return err
    }

    return nil
}

func (p Post) SanitizedTitle() string {
    return strings.ToLower(strings.Replace(p.Title, " ", "-", -1))
}
```

## Approval tests

[Approval tests](https://github.com/approvals/go-approval-tests)

```bash
go get github.com/approvals/go-approval-tests
```

An approval tool compares program output with an approved file that you created. Use this instead of using golden files. A golden file is the master file that you test your development code against.

## Generics

Types exist so you can tell the compiler what form of data to look for: a string, int, etc. Generics let you design functions that do not requrethat accept types that do not require concrete types, but rather types that have the behavior you need.

Generic functions need a type parameter that includes a description of your generic type and a label. Here, `T` is the label, and `comparable` is the description:

```go
func AssertEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

Alternately, you can use the empty interface (`interface{}` or `any`), which lets you pass any type. Make sure you use the `%+v` format in formatted strings.

The problem with `any` is that we are not telling the compiler anything about the types we pass to the function. There are absolutely NO constraints, which can lead to runtime errors. Generics let you provide some guidance (constraings) to the compiler. Also, you don't have to make type assertions if your function returns a generic type, the caller can use the type as it is returned.

Here is an implementation using generics:

```go
type Stack[T any] struct {
    values []T
}

func NewStack[T any]() *Stack[T] {
    return new(Stack[T])
}

func (s *Stack[T]) Push(value T) {
    s.values = append(s.values, value)
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.values) == 0
}

func (s *Stack[T]) Pop() (T, bool) {
    if s.IsEmpty() {
        var zero T
        return zero, false
    }

    index := len(s.values) - 1
    el := s.values[index]
    s.values = s.values[:index]
    return el, true
}
```

Here are the tests:

```go
type Stack[T any] struct {
    values []T
}

func NewStack[T any]() *Stack[T] {
    return new(Stack[T])
}

func (s *Stack[T]) Push(value T) {
    s.values = append(s.values, value)
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.values) == 0
}

func (s *Stack[T]) Pop() (T, bool) {
    if s.IsEmpty() {
        var zero T
        return zero, false
    }

    index := len(s.values) - 1
    el := s.values[index]
    s.values = s.values[:index]
    return el, true
}
```