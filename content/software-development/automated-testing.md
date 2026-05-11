+++
title = 'Automated Testing'
date = '2026-05-10T20:33:54-04:00'
weight = 40
draft = false
+++

## Benefits of automated testing

Automated tests do three things for you directly.

### Enhance code quality

Writing a test forces you to think about how your code will be used before you write it. That pressure surfaces design problems early, when they are cheap to fix. Code that is hard to test is usually code that is too tightly coupled or doing too much. The tests make that visible before it becomes a maintenance problem.

Writing good, maintainable code is a skill that typically takes years to develop. It requires thinking before you speak: reasoning through the design before committing to an implementation. Writing tests develops that habit. They expose areas where the code could be simpler, better named, or more clearly structured, and they give you a concrete signal to act on rather than a vague sense that something is off.

Tests also act as documentation. A test suite describes what a module does, what inputs it accepts, and what behavior it guarantees. Unlike written documentation, tests stay honest: they fail when the code diverges from them. A developer new to a module can read the tests to understand its features and trace back to the relevant code sections, without relying on documentation that may be outdated.

### Boost your confidence

A comprehensive test suite gives you the freedom to code without fear. You can make changes, try new solutions, and refactor with confidence, knowing the tests will catch any issues you introduce. That freedom matters. Without it, engineers hesitate to improve code they didn't write, and codebases calcify around the fear of breaking something invisible.

A comprehensive test suite also helps you better understand the system's behavior, catches bugs early in the development process when they are simpler and cheaper to fix, reduces the likelihood of introducing regressions, ensures features meet their specifications, and maintains code quality as the system grows.

### Work more efficiently

A well-written test suite acts as a fast feedback loop. Instead of manually exercising your application after every change, you run the tests and know in seconds whether the code works as expected. That speed compounds over time. The hours saved across hundreds of changes are significant.

## Types of automated testing

The testing pyramid describes how to distribute tests across three levels. The base is wide because unit tests should be numerous. The top is narrow because end-to-end tests should be few.

```
            /\
           /  \
          / E2E\
         /------\
        /        \
       /Integration\
      /------------\
     /              \
    /  Unit Tests    \
   /------------------\
```

The pyramid's shape carries a practical constraint: as you move up, tests run slower. Unit tests finish in milliseconds. Integration tests take seconds. End-to-end tests can take minutes. A test suite that is top-heavy runs too slowly to provide useful feedback during development. The pyramid tells you where to invest: write many fast unit tests, fewer integration tests, and only as many end-to-end tests as necessary to verify the critical paths.

### Unit tests

A unit test examines an individual component or function in isolation. All dependencies are replaced with fakes or mocks so that the test exercises only the logic of the unit itself. Unit tests are fast, precise, and easy to diagnose. When one fails, you know exactly which piece of logic broke.

Because they run in milliseconds, you can run hundreds of them on every save. They are the foundation of a healthy test suite.

```go
// order_service.go
type OrderService struct {
    repo OrderRepository
}

func (s *OrderService) TotalFor(customerID string) (float64, error) {
    orders, err := s.repo.FindByCustomer(customerID)
    if err != nil {
        return 0, err
    }
    var total float64
    for _, o := range orders {
        total += o.Amount
    }
    return total, nil
}

// order_service_test.go
type mockRepo struct {
    orders []Order
}

func (m *mockRepo) FindByCustomer(id string) ([]Order, error) {
    return m.orders, nil
}

func TestTotalFor(t *testing.T) {
    svc := &OrderService{
        repo: &mockRepo{orders: []Order{{Amount: 10.0}, {Amount: 25.0}}},
    }

    total, err := svc.TotalFor("customer-1")
    if err != nil {
        t.Fatal(err)
    }
    if total != 35.0 {
        t.Errorf("got %.2f, want 35.00", total)
    }
}
```

The repository is replaced with a mock. The test verifies only the summation logic in `TotalFor`, not the database.

### Integration tests

An integration test is a detailed process that thoroughly verifies how different components or modules of a system work together as a cohesive unit. Where a unit test replaces the database with a mock, an integration test uses a real database. Where a unit test stubs an HTTP call, an integration test makes the real call or spins up a test server.

Integration tests catch issues that arise only when individually tested components are combined: mismatched interfaces, incorrect query logic, and misconfigured dependencies that unit tests, by design, cannot see. They run slower than unit tests because they exercise real infrastructure, so you write fewer of them and run them less frequently.

```go
func TestOrderRepository_FindByCustomer(t *testing.T) {
    db, err := sql.Open("postgres", os.Getenv("TEST_DATABASE_URL"))
    if err != nil {
        t.Fatal(err)
    }
    defer db.Close()

    repo := NewOrderRepository(db)
    _, err = db.Exec(`INSERT INTO orders (customer_id, amount) VALUES ($1, $2)`,
        "customer-1", 49.99)
    if err != nil {
        t.Fatal(err)
    }

    orders, err := repo.FindByCustomer("customer-1")
    if err != nil {
        t.Fatal(err)
    }
    if len(orders) != 1 || orders[0].Amount != 49.99 {
        t.Errorf("unexpected orders: %+v", orders)
    }
}
```

This test hits a real database. It verifies that the SQL query is correct, the connection is configured properly, and the result is mapped to the right struct fields.

### End-to-end tests

An end-to-end (E2E) test covers the entire application starting at the UI and follows the request all the way through every layer to the database and back. Nothing is mocked. For backend systems, E2E tests focus on the API: they send real HTTP requests and verify the responses produced by the fully assembled system.

Unlike unit and integration tests, E2E tests require a real environment to run. The application must be deployed, the database must be seeded, and any external dependencies must be available. That requirement makes them slower to set up and more expensive to maintain than other test types.

E2E tests give you the highest confidence that the system works as a whole. They are also the most brittle. A small infrastructure change, a network delay, or a timing issue can cause them to fail for reasons unrelated to your code, making them more susceptible to false negatives. Keep the set small and focused on the flows that matter most to your users.

```go
func TestCheckout_E2E(t *testing.T) {
    server := startTestServer(t) // boots the full application
    defer server.Close()

    body := strings.NewReader(`{"cart_id":"cart-123","customer_id":"customer-1"}`)
    resp, err := http.Post(server.URL+"/checkout", "application/json", body)
    if err != nil {
        t.Fatal(err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        t.Errorf("got status %d, want 200", resp.StatusCode)
    }

    var result map[string]any
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        t.Fatalf("decoding response: %v", err)
    }
    if result["order_id"] == nil {
        t.Error("expected order_id in response")
    }
}
```

The test boots the full application, sends a real HTTP request, and asserts on the response. Every layer from the HTTP handler to the database participates.

## What tests should not cover

Writing tests for the wrong things wastes time and produces a test suite that is expensive to maintain without adding confidence. Avoid these categories.

#### Language features and framework code

Don't test that Go's `json.Marshal` serializes a struct correctly, or that your HTTP router dispatches to the right handler. The language and framework are already tested by their maintainers. Your tests should verify your logic, not theirs.

#### Getters, setters, and builder methods

These contain no logic. A getter returns a field value. A setter assigns one. Testing them only adds noise to the suite and maintenance cost when the struct changes.

#### DTOs

A data transfer object carries data between layers. It has no behavior to verify. Testing that a struct holds the values you assigned to it tells you nothing useful.

#### Private methods

Test the public interface, not the implementation. Private methods are implementation details that should be exercised indirectly through the public methods that call them. Testing private methods directly couples your tests to the internal structure of the code, making refactoring harder without making the suite more reliable.

#### External services

Tests that depend on a live database, a third-party API, or any external system are slow, fragile, and non-deterministic. A network timeout or a rate limit will fail your test for reasons that have nothing to do with your code. Use mocks or fakes at these boundaries. Your tests should verify how your code behaves given a response, not whether the external service is reachable.

## Assertions

An *assertion* is a function or method that verifies specific conditions are met during the execution of a program. When the condition is false, the assertion fails the test. Assertions are how a test communicates that the code did not behave as expected.

Every test follows the same basic structure: arrange the inputs, act by calling the *system under test* (the specific function, method, or component being verified), and assert that the output matches what you expected. The assertion is the part that does the verifying.

### Assertions in Go

Go's standard library provides no assertion helpers. Instead, you call methods on `*testing.T` directly.

Use `t.Errorf` to record a failure and continue running the test:

```go
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    if got != 5 {
        t.Errorf("Add(2, 3) = %d, want 5", got)
    }
}
```

Use `t.Fatalf` to record a failure and stop the test immediately. Reach for it when continuing would cause a panic or produce misleading output:

```go
func TestGetUser(t *testing.T) {
    user, err := GetUser("user-1")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if user.Name != "Alice" {
        t.Errorf("got name %q, want %q", user.Name, "Alice")
    }
}
```

If `GetUser` returns an error, there is no `user` to inspect. `t.Fatalf` stops the test before the next assertion runs.

### Using testify

The [`testify`](https://github.com/stretchr/testify) package provides assertion helpers that reduce boilerplate and produce clearer failure messages.

```go
import "github.com/stretchr/testify/assert"

func TestCreateOrder(t *testing.T) {
    order, err := CreateOrder("customer-1", 49.99)

    assert.NoError(t, err)
    assert.Equal(t, "customer-1", order.CustomerID)
    assert.Equal(t, 49.99, order.Total)
    assert.NotEmpty(t, order.ID)
}
```

Use `require` instead of `assert` when a failed assertion should stop the test:

```go
import (
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestCreateOrder(t *testing.T) {
    order, err := CreateOrder("customer-1", 49.99)

    require.NoError(t, err)           // stops here if err != nil
    assert.Equal(t, 49.99, order.Total)
    assert.NotEmpty(t, order.ID)
}
```

`require` mirrors `t.Fatalf`: it stops the test immediately on failure. `assert` mirrors `t.Errorf`: it records the failure and continues. Use `require` for preconditions, `assert` for everything else.

## Mocking

*Mocking* is a technique for creating a simulated version of a dependency so you can test code in isolation. A *dependency* is another class or component that the system under test relies on to complete a task — a database repository, an email client, an external API.

In unit tests, you replace dependencies with mocks so that the test exercises only the logic of the system under test. In integration tests, you let the real dependencies participate to verify that they work together correctly.

A mock looks like the real dependency but isn't. It implements the same interface, so the system under test can't tell the difference. What it doesn't do is execute real business logic. Instead, it does exactly what you program it to do: return a specific value, simulate an error, or record that it was called.

### Mocking in Go

Go's interface system makes mocking straightforward. Define an interface for the dependency, write a mock that implements it, and inject the mock in your test.

```go
// The interface the system under test depends on
type EmailSender interface {
    Send(to, subject, body string) error
}

// The real implementation (not used in unit tests)
type SMTPSender struct{}

func (s *SMTPSender) Send(to, subject, body string) error {
    // sends a real email
    return nil
}

// The mock
type mockEmailSender struct {
    sentTo string
    called bool
    err    error // returned by Send when set
}

func (m *mockEmailSender) Send(to, subject, body string) error {
    m.called = true
    m.sentTo = to
    return m.err
}

// The system under test
type NotificationService struct {
    email EmailSender
}

func (s *NotificationService) NotifyUser(userEmail string) error {
    return s.email.Send(userEmail, "Welcome", "Thanks for signing up.")
}

// The test
func TestNotifyUser(t *testing.T) {
    sender := &mockEmailSender{}
    svc := &NotificationService{email: sender}

    err := svc.NotifyUser("alice@example.com")

    require.NoError(t, err)
    assert.True(t, sender.called)
    assert.Equal(t, "alice@example.com", sender.sentTo)
}
```

The test verifies that `NotifyUser` calls the email sender with the correct address. No real email is sent. The mock records what it received so the assertions can inspect it.

### Simulating errors

Mocks also let you test how your code handles failures — scenarios that are difficult or impossible to trigger with a real dependency.

```go
func TestNotifyUser_EmailFailure(t *testing.T) {
    sender := &mockEmailSender{err: errors.New("smtp timeout")}
    svc := &NotificationService{email: sender}

    err := svc.NotifyUser("alice@example.com")

    assert.Error(t, err)
}
```

Triggering an SMTP timeout in a real test environment is impractical. With a mock, it takes one line.

## How to improve your testing skills

Testing well is a skill that develops with deliberate practice. These approaches accelerate that development.

### Learn from open source libraries

Production-quality open source projects are some of the best testing resources available. Read the test suites of well-maintained Go projects to see how experienced engineers structure tests, name cases, and handle edge conditions. The standard library itself is a strong starting point.

### Use tests as documentation

When joining an unfamiliar codebase, read the tests before the implementation. They describe what the code is supposed to do, what inputs it handles, and how it behaves under failure conditions. This habit builds your ability to write tests that serve the same purpose for the next developer.

### Improve existing codebases

If you work in a codebase with weak test coverage, pick an untested module and write tests for it. The act of testing existing code teaches you where the design makes testing hard — tightly coupled dependencies, methods that do too much, missing interfaces. Each gap is a lesson.

### Start with tests for new features

Before writing implementation code for a new feature, write the test first. This forces you to think about the interface and expected behavior before committing to an implementation. It also guarantees the feature ships with coverage.

### Address bugs with tests

When you find a bug, write a test that reproduces it before fixing it. The test confirms you understand the failure, verifies your fix is correct, and prevents the bug from returning undetected.

### Advocate for testing

Testing culture is built incrementally. Review others' code with testing in mind. Ask whether edge cases are covered. Normalize writing tests as part of the definition of done, not as an afterthought. The team's habits shape the codebase as much as any individual's code.
