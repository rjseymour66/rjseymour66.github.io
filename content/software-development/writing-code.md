+++
title = 'Writing Code'
date = '2026-05-10T20:07:27-04:00'
weight = 30
draft = false
+++



## The zeroth law of computer science

Every decision in software design reduces to a single principle: high cohesion and low coupling. Practitioners call it the zeroth law because it underlies every good design pattern, architecture decision, and refactoring technique you'll encounter.

*Cohesion* measures how closely related the responsibilities within a single module are. High cohesion means a module does one thing and does it completely. Every function, method, and data structure it contains belongs there. Low cohesion means a module does many unrelated things. A class that handles user authentication, formats dates, and sends emails has low cohesion. Nothing inside it is there for the same reason.

*Coupling* measures how much one module depends on another. Low coupling means modules can change independently. A change to one module doesn't ripple through the rest of the codebase. High coupling means the opposite: touch one thing, break another.

### Example: a user service

A high-cohesion, low-coupling user service handles exactly one domain: users. It creates accounts, validates credentials, and manages profile data. It depends on an abstraction (an interface or contract) to send email, not on a specific email library. When the email provider changes, the user service doesn't change.

A low-cohesion, high-coupling alternative puts user management, email delivery, and payment processing in the same module and calls the payment library directly. Every change to payment logic risks breaking user registration.

High cohesion makes code easier to understand. You know where to look because related things live together. Low coupling makes code easier to change. You can modify one module without triggering a cascade of changes elsewhere.

## Composition over inheritance

*Inheritance* lets one class derive behavior from another. *Composition* builds behavior by combining smaller, focused pieces. The Gang of Four established "favor composition over inheritance" as a foundational principle, and the reasoning holds up across every language and codebase.

Inheritance creates tight coupling between a parent class and its children. When you change the parent, every subclass is affected. Deep hierarchies become especially brittle: a change three levels up can break behavior five levels down in ways that are hard to trace. The child also inherits everything from the parent, including behavior it doesn't need and can't easily remove.

Composition avoids these problems. Instead of extending a class to gain behavior, you give an object a reference to another object that provides that behavior. You assemble what you need from small, focused parts.

### Example: logging

With inheritance, a service extends a `BaseLogger` class to gain logging behavior. To swap the logger for a different implementation, you change the base class or restructure the entire hierarchy.

With composition, the service holds a reference to a `Logger` interface. You pass in any implementation at construction time. In tests, you pass a no-op logger. In production, you pass the real one. The service never changes.

### Example: notifications

With inheritance, an `AlertService` base class spawns subclasses for each channel: `EmailAlertService`, `SMSAlertService`, `PushAlertService`. When a critical event needs to trigger all three channels at once, the hierarchy has no clean answer. You can't inherit from three classes simultaneously.

With composition, an `AlertService` holds a list of `Notifier` implementations. You inject whichever notifiers apply at runtime. Adding a Slack notifier means implementing the `Notifier` interface, not restructuring the hierarchy or touching existing code.

### When inheritance is appropriate

Inheritance models genuine "is-a" relationships well. A `SavingsAccount` truly is a `BankAccount`, and sharing that contract through inheritance is reasonable. Reach for composition instead when you find yourself overriding methods to disable inherited behavior, or when your hierarchy grows beyond two levels.

## Methods should do one thing

The Unix philosophy, articulated by Doug McIlroy in 1978, says to write programs that do one thing and do it well, and write programs that work together. The same principle applies to every method you write.

A method that does one thing is easy to name, easy to test, and easy to reuse. When a method does several things, its name either lies (it says one thing but does more) or becomes so vague it tells you nothing: `processData`, `handleRequest`, `doStuff`.

### Names should speak for themselves

Ask yourself: how long does it take to understand what a function does? You should be able to read the name and know immediately, without opening the body. A well-named method is self-documenting. If you have to read the implementation to understand the intent, the name has failed.

Think of a class name as the headline of an article. It tells you the subject. From there, you should be able to skim the method signatures and get a clear picture of what the class does and how it behaves. If skimming the signatures leaves you confused, the class is doing too much.

Short methods are also easier to reason about. When a method fits on one screen, you understand it completely. When it runs to hundreds of lines, you must hold the entire thing in your head to change any part of it safely.

### Measuring "one thing"

Describe what the method does without using the word "and." If you say "this method validates the input and writes to the database and sends a confirmation email," you have three methods pretending to be one.

Another signal: if a block of code inside a method needs a comment to explain what it does, that block is a candidate for extraction. The new method's name becomes the comment.

### Composing small methods

Small, focused methods combine to accomplish larger goals, just as Unix commands pipe together. Consider a data processing pipeline:

```
records := load(filename)
valid   := filter(records, isValid)
results := transform(valid, normalize)
save(results, outputPath)
```

Each method does exactly one thing. You can test each step in isolation, swap any implementation without touching the others, and read the top-level function as a plain description of the overall process.

### How long is too long?

There is no universal rule, but a useful heuristic: if a method exceeds one screen (roughly 20 to 30 lines), look for extraction opportunities. Length is not the problem by itself. A method that does one thing cleanly at 40 lines is better than a method that does three things in 15.

## Code comments

Liberally commenting your code is a code smell. Comments that explain *what* the code does start diverging from the code the moment you make a change without updating them. A stale comment is worse than no comment. It misleads anyone who reads it.

Your time is better spent making the code itself easier to read. Clear names, short methods, and good structure communicate intent without narration. If you feel the urge to write a comment explaining what a block of code does, that's a signal to simplify or extract the code instead.

### Comments should explain why, not what

The code already says what it does. A good comment explains the reasoning behind a decision that the code can't express on its own: a constraint, a workaround, a non-obvious tradeoff.

*What* comments add noise:

```go
// increment the counter
count++

// check if the user is an admin
if user.Role == "admin" {
```

*Why* comments add value:

```go
// The payment gateway returns 429s under sustained load, so we retry up to 3 times.
for attempt := range 3 {

// Use UTC here to avoid DST ambiguity in the scheduling logic.
createdAt := time.Now().UTC()
```

The first set tells you nothing the code doesn't already tell you. The second set records decisions a future reader couldn't reconstruct from the code alone.

### When comments are appropriate

Write a comment when:

- A workaround exists because of an external bug or constraint
- A non-obvious algorithm is in use and a reference explains it better than the code can
- A decision was made deliberately against the obvious approach

If none of these apply, the comment probably belongs in a commit message, not the source file.

## Tests as documentation

Tests are the most reliable documentation in a codebase. Unlike comments, they can't silently drift from the code they describe. When the behavior changes, the tests either change with it or fail. That constraint keeps them honest.

A well-written test suite tells you what the system does, what inputs it accepts, what outputs it produces, and how it behaves when things go wrong. Read the tests before the implementation when working in an unfamiliar codebase. They give you a verified description of intent that the source code alone can't provide.

### Consumer-driven contracts

*Consumer-driven contracts* take this further for distributed systems. Instead of relying on shared documentation or assumptions, the consumer of a service writes tests that describe exactly what it needs from the provider: which endpoints it calls, what request shape it sends, and what response shape it expects. The provider runs those tests as part of its own build.

This approach does two things at once. It communicates what your service does in executable, verifiable form. It also gives you the confidence to iterate. When you change the provider, you know immediately whether any consumer breaks, without deploying both services and running end-to-end tests.

Tools like [Pact](https://docs.pact.io/) implement this pattern. The consumer generates a contract file during its test run. The provider verifies against that contract. The contract becomes the shared understanding between teams, and the test suite enforces it continuously.

### The practical result

A service with a good contract test suite is self-describing. A new team member can read the contracts to understand what the service promises to its callers. A developer making changes can refactor freely, knowing the contracts will catch any regression that matters to a real consumer.

## Avoid clever code

*Essential complexity* is inherent in the problem you're solving. A payment system is complex because payments are complex. You can't simplify that away. *Accidental complexity* is complexity developers introduce themselves through the choices they make: clever abstractions, obscure language features, and code written to impress rather than communicate.

Clever code is a liability. It takes longer to read, harder to debug, and often surprises the next developer in ways the author didn't anticipate. That next developer is frequently you, six months later.

Write code that a tired colleague could understand at the end of a long day. The measure of good code isn't how sophisticated it looks. It's how quickly someone can understand what it does and change it safely.

### Examples of accidental complexity

A one-liner that chains three higher-order functions may be technically correct, but three named steps are easier to follow, easier to debug, and easier to test individually.

```go
// Clever
result := lo.Map(lo.Filter(orders, func(o Order, _ int) bool { return o.Active }), func(o Order, _ int) float64 { return o.Total })

// Clear
active := filterActiveOrders(orders)
result := extractTotals(active)
```

Deep nesting is another common form. Each extra level of indentation is a cognitive tax on the reader. Early returns flatten the structure and eliminate the tax.

```go
// Clever (nested)
func process(order Order) error {
    if order.Valid {
        if order.Active {
            if order.Total > 0 {
                return save(order)
            }
        }
    }
    return nil
}

// Clear (early returns)
func process(order Order) error {
    if !order.Valid || !order.Active || order.Total <= 0 {
        return nil
    }
    return save(order)
}
```

### The right use of sophistication

This doesn't mean avoiding advanced language features. It means using them when they reduce complexity, not when they perform it. The question to ask is: does this make the code easier to understand, or does it make me look clever?

## Code reviews

Code reviews catch bugs, but their deeper value is ensuring code is understandable and maintainable long after the author has moved on. To get that value, focus your attention at the bottom of the code review pyramid.

The pyramid places style and formatting at the top: the smallest, least important concerns. Linters and formatters handle most of that automatically. The bottom holds the substantive questions about design, readability, and correctness. That's where your time belongs.

### Readability and naming

Start with the basics. Are method and variable names clear and concise? Can you read the code without stopping to decode it? Good names eliminate the need for explanation. If you find yourself re-reading a line to understand it, the name or structure has failed.

Look for duplication. Duplicated logic is a maintenance trap. When the logic changes, every copy has to change with it, and one is always missed.

### Observability

Ask whether the code has the appropriate logging, tracing, and metrics. A feature that ships without observability is a feature you can't diagnose in production. Check that log messages are meaningful, that trace spans cover the right boundaries, and that metrics exist for behavior that matters to the system.

### Design and correctness

These are the hardest questions and the most important ones.

- Are the interfaces consistent with the rest of the codebase? Inconsistent interfaces create cognitive overhead for every developer who follows.
- Did the developer use any error-prone forms? Patterns that look correct but fail under specific conditions deserve scrutiny.
- Is the model correct? Does the code accurately represent the domain it operates in?
- Did the developer choose the right abstraction? An abstraction that fits well today should also accommodate the changes that are likely to come.

A review that catches a wrong abstraction early saves far more time than one that catches a formatting inconsistency.

## The lazy programmer ethos

The lazy programmer ethos is a philosophy built on efficiency. Strategic laziness isn't about avoiding work. It's about choosing the right work.

When you resist the urge to immediately start coding, you create space to think. That thinking time pays off. You write less code, avoid dead ends, and build solutions that hold up over time.

In practice, the lazy programmer:

- Automates repetitive tasks rather than doing them manually
- Reuses existing code before writing new code
- Asks "do I actually need this?" before starting any task
- Writes simple code that's easy to change, not code that anticipates every future requirement

The goal isn't to do less. It's to eliminate unnecessary work so you can focus on what matters.

## Shift left

"Shifting left" means moving testing and quality checks earlier in the development cycle. The name comes from project timelines where earlier stages appear on the left. The further right a bug travels, the more expensive it becomes to fix.

In practice, shifting left means:

- Write tests before or alongside your code, not after
- Run linters and static analysis tools on every commit
- Review code early and often, not just before release
- Include security scanning in your CI pipeline from the start
- Define acceptance criteria before writing a single line of code

A bug caught in development costs minutes to fix. The same bug caught in production can cost hours, damage users, and erode trust. Shifting left keeps quality work close to where the code is written.

## The five whys

The five whys is a problem-solving method for finding the root cause of an issue. Instead of treating symptoms, you ask "why" five times, each time directing the question at the previous answer. By the fifth why, you've usually uncovered the underlying cause.

### Example

1. Why did the site go down? The server ran out of memory.
2. Why did the server run out of memory? A process leaked memory over time.
3. Why did the process leak memory? The connection pool was never closed after use.
4. Why was the connection pool never closed? The error-handling path skipped the cleanup code.
5. Why did the error-handling path skip cleanup? There were no tests covering the error path.

The root cause isn't the outage or even the leak. It's a gap in test coverage. Fixing only the symptom (restarting the server) guarantees the problem returns.

Use the five whys when a bug, incident, or failure keeps recurring. The method forces you past the obvious fix and toward the change that actually prevents recurrence.
