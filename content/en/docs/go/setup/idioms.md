---
title: "Idioms"
weight: 40
description: >
  Common patterns and strategies to write idiomatic Go.
---

## Short-if declaration

`if variable := value; condition`

For example:
```go
if err := json.Marshal(&val); err != nil {
    // handle error
}
```

## Named return values

Name the returned values of a function so other developers can see what it returns without having to read the code.

## ok return value

If you are writing a helper function, do NOT return an error from the helper--return `ok bool`. This allows you to check the `ok` value in the caller and return the error there. For example:

```go
func Parse(rawurl string) (*URL, error) {

	scheme, rest, ok := parseScheme(rawurl)
	if !ok {
		return nil, errors.New("missing scheme")
	}
    ...
}

func parseScheme(rawurl string) (scheme, rest string, ok bool) {
	i := strings.Index(rawurl, "://")
	if i < 1 {
		return "", "", false
	}
	return rawurl[:i], rawurl[i+3:], true
}
```

## Naked returns

You can return from a function with just the `return` keyword. This is called a _naked return_. A naked return returns the current state of the result values.

Generally, **do NOT** use naked returns because they impact readability.

## blank identifiers

You can use blank identifiers in function definitions:
```go
handler := func(_ http.ResponseWriter, _ *http.Request) {
	// logic
}
```


## Interfaces

When a type satisfies an interface, you say _type X is a Y_. For example, _URL is a Stringer_ or _Parser is a Reader_.

### Stringer

`Stringer` prints the string representation of the object. The `fmt.Print[x]` packages detect when a type has a `Stringer` method, so it calls that method for proper formatting.

The `Stringer` interface:
```go
type Stringer interface {
    String() string
}
```

Implementation example:
```go
func (u *URL) String() string {
	return fmt.Sprintf("%s://%s/%s", u.Scheme, u.Host, u.Path)
}
```
### testString

Create a `testString()` function to return a concise string representation value for tests. For example, the following is the test version of the `String()` function in the previous section:

```go
func (u *URL) testString() string {
	return fmt.Sprintf("scheme=%q, host=%q, path=%q", u.Scheme, u.Host, u.Path)
}
```

### sort.Interface

The Go `sort` package has a [`Slice` method](https://pkg.go.dev/sort#Slice) to sort the values in a slice. However, you can implement the `sort.Interface` interface to create a custom `sort` method.

`sort.Interface` has the following methods:
```go
// Len is the number of elements in the collection.
Len() int
 
// Less reports whether the element with index i 
// must sort before the element with index j.
Less(i, j int) bool
 
// Swap swaps the elements with indexes i and j.
Swap(i, j int)
```
`sort.Interface` operates on collection types, so you should create a custom collection type to implement the interface methods. Name the type in a way that describes the sorting strategy. For example, the following implementation sorts a slice of type `Book` by the `Author` field:

```go
// Define a custom type whose name describe the sorting strategy.
type byAuthor []Book
 
// Len returns the length of the collection.
func (b byAuthor) Len() int { return len(b) }
 
// Swap swaps two books.
func (b byAuthor) Swap(i, j int) { 
    b[i], b[j] = b[j], b[i]
}
 
// Less returns books sorted by Author and then Title.
func (b byAuthor) Less(i, j int) bool {
    if b[i].Author != b[j].Author {
        return b.LessByAuthor(i, j)
    }
    return b[i].Title < b[j].Title
}
```

### Empty interface

Go versions prior to 1.18 used the empty interface: `interface{}`. This is an interface that did not implement any methods, so any type satisfied it. In Go 1.18 and later, `interface{}` was replaced with `any`. 


## strconv.ParseInt

Prefer `strconv.ParseInt()` over `strconv.Atoi()` because the former can parse hex and numbers with underscores (`1_000`).

## Formatting verbs

| verb | Definition | Usage |
|------|:-----------|:------|
| %q   | Wraps the given string in double quotes. | |
| %#v  | Prints the Go syntax representation of the value. | t.Errorf("%#v.String()\ngot  %q\nwant %q", u, got, want) |
| %t   | Boolean values. |

## nil

You can execute a method on a `nil` type. A method is a function that takes the receiver as a hidden first parameter. So, when you have a `nil` type, Go can find the method function to run, but it does not have anything to execute it on. For example, the Go compiler does the following when calling the `String()` method on a `nil` typ:

```go
var u *URL
u.String() // (*url.URL).String(u)
```

### if found

Use `if found...` to determine if an element exists in a list:
```go
// returs a bool and int
func (hl *HostsList) search(host string) (bool, int) {
    ...
}

// found is a boolean. You can use a truncated syntax:
func (hl *HostsList) Add(host string) error {
	if found, _ := hl.search(host); found {
		return fmt.Errorf("%w: %s", ErrExists, host)
	}
    ...
}
```

## Fatal and Panic

You should call `Fatal()` and `Panic()` from the `main` method only. Return errors in other areas of the application.