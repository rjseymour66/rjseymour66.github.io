+++
title = 'Functions'
date = '2025-08-30T10:07:00-04:00'
weight = 30
draft = false
+++

## Variadic functions

A variadic function accepts a variable number of arguments of the same type. Place `...` before the type in the parameter list to mark it as variadic. Go converts the arguments into a slice, so you can use any slice operation on them.

The variadic parameter must be last if the function has multiple parameters.

This function accepts zero or more strings and prints each one:

```go
func varString(str ...string) {
	for _, s := range str {
		fmt.Printf("%s ", s)
	}
	fmt.Println()
}
```

Call it with any number of arguments:

```go
varString("one", "two", "three")
```

## Closures

A closure is a function that captures variables from its surrounding scope. It retains access to those variables even after the outer function has returned.

When a normal function returns, Go pops it off the stack and frees its local variables. A closure changes this: when Go detects that an inner function references a variable from an outer function, it allocates that variable on the heap instead. The inner function holds a pointer to it.

This `counter` function returns an anonymous function that increments and returns `count`. Each call to the returned function advances the same `count` variable:

```go
func counter() func() int {
	count := 0
	return func() int {
		count++
		return count
	}
}

c := counter()
fmt.Println(c()) // 1
fmt.Println(c()) // 2
fmt.Println(c()) // 3
```

When `counter` returns, `count` is not freed. The returned function holds a pointer to it on the heap.

### Middleware

The closure pattern is common in HTTP middleware. The middleware function wraps a handler and adds behavior around it without modifying the handler itself.

This `logger` middleware closes over `f`, the handler passed to it, so the returned function can call `f` after the outer function has returned:

1. `logger` accepts an `http.HandlerFunc` and returns one.
2. The returned function is the actual handler. It records the start time, calls `f`, then logs the elapsed duration.
3. Wrap any handler with `logger` at registration time.

```go
func logger(f http.HandlerFunc) http.HandlerFunc {             // 1
	return func(w http.ResponseWriter, r *http.Request) {      // 2
		start := time.Now()
		f(w, r)
		log.Printf("%s (%v)", r.URL.Path, time.Since(start))
	}
}

http.HandleFunc("/hello", logger(hello))                       // 3
```

## Immediately invoked function literals (IIFL)

An IIFL is an anonymous function called immediately after its closing brace. Goroutines commonly use this pattern:

```go
for _, file := range os.Args[1:] {
    wg.Add(1)
    go func(filename string) {
        compress(filename)
        wg.Done()
    }(file)
}
```

`file` is passed as an argument to the IIFL rather than captured from the loop variable. Each goroutine receives its own copy of `file`, evaluated at the moment the goroutine is launched.

If the goroutine captured `file` directly, all goroutines would share the same variable. Because the loop advances that variable before the goroutines run, every goroutine would likely operate on the last value the loop assigned. Each would compress the same file rather than a unique one.

## Function types

A function type declares a function signature as a named type. You can assign it to a variable, pass it as an argument, return it from another function, or store it in a struct. Function types are a lightweight alternative to an interface when you need to pass a single behavior.

### Strategy pattern

The strategy pattern lets you pass different behavior into the same function by accepting a function type as a parameter:

1. Declare the function type:
   ```go
   type Operation func(int, int) int
   ```
2. Write a function that accepts it:
   ```go
   func calculate(a, b int, op Operation) int {
       return op(a, b)
   }
   ```
3. Define functions that match the signature:
   ```go
   func add(a, b int) int      { return a + b }
   func subtract(a, b int) int { return a - b }
   func multiply(a, b int) int { return a * b }
   ```
4. Pass them to `calculate`:
   ```go
   a := calculate(5, 4, add)
   b := calculate(5, 4, subtract)
   c := calculate(9, 2, multiply)
   ```

### Dependency injection

A function type can serve as a dependency. The struct doesn't know what `send` does, only its signature. You swap implementations without changing the service:

1. Define the function type:
   ```go
   type SendFunc func(to, message string) error
   ```
2. Build a service that depends on it:
   - `NotificationService` holds a `SendFunc` field. It doesn't know whether `send` prints, emails, or does nothing.
   - `NewNotificationService` is the constructor. Pass the implementation here. This is the injection point.
   - `NotifyUser` calls `send` with the recipient and message. It delegates entirely to whatever was injected.
   ```go
   type NotificationService struct {
       send SendFunc
   }

   func NewNotificationService(send SendFunc) *NotificationService {
       return &NotificationService{send: send}
   }

   func (s *NotificationService) NotifyUser(user string) error {
       return s.send(user, "Welcome!")
   }
   ```
3. Define a `SendFunc` implementation:
   ```go
   func testSend(to, message string) error {
       fmt.Printf("To: %s\nMessage: %s\n", to, message)
       return nil
   }
   ```
4. Wire them together:
   ```go
   func main() {
       svc := NewNotificationService(testSend)
       if err := svc.NotifyUser("alice@example.com"); err != nil {
           log.Fatal(err)
       }
   }
   ```

#### Production-grade

To make this testable, move the logic into a `run` function and inject `stderr`:

```go
type env struct {
    stderr io.Writer
}

func run(e *env) error {
    svc := NewNotificationService(testSend)
    if err := svc.NotifyUser("alice@example.com"); err != nil {
        fmt.Fprintf(e.stderr, "%v\n", err)
        return err
    }
    return nil
}

func main() {
    if err := run(&env{stderr: os.Stderr}); err != nil {
        os.Exit(1)
    }
}
```