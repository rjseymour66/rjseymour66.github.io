---
title: "Fundamentals"
weight: 30
---

This page covers Python's core building blocks: operators, control flow, and loops. Understanding where Python differs from other languages — chained comparisons, loop `else` clauses, `match`/`case` — pays dividends throughout the rest of the language.

## Operators

### Comparison and assignment operators

The following operators form the foundation of Python expressions:

```python
=       # assignment — binds a name to a value
==      # equality — compares values, not memory location

# Identity — compares memory location, not value
is      # True if both names refer to the same object
is not  # True if both names refer to different objects

# Membership
in      # True if x is in y
not in  # True if x is not in y

# Logical
and
or
not

# Boolean literals
True
False
```

To test equality between values, apply `==`. To test whether two names refer to the same object in memory, apply `is`. `None` checks should always apply `is` rather than `==`:

```python
value = None
if value is None:
    print("no value provided")
```

### Augmented assignments

*Augmented assignment operators* combine an arithmetic operation with assignment. They update a variable in place rather than creating a new binding:

```python
# Without augmented assignments:
count = count + 1
total = total - refund

# With augmented assignments — shorter and reads as "add to," "subtract from":
count += 1
total -= refund
total *= 1.08   # apply sales tax
rate  /= 100    # convert percentage to decimal
```

The following example tallies HTTP response codes. Updating counters this way is idiomatic Python and appears frequently in metrics collection, log analysis, and retry logic:

```python
from dataclasses import dataclass, field

@dataclass
class RequestStats:
    total: int = 0
    success: int = 0
    client_errors: int = 0
    server_errors: int = 0

def tally_response(stats: RequestStats, status: int) -> None:
    stats.total += 1

    if 200 <= status <= 299:
        stats.success += 1
    elif 400 <= status <= 499:
        stats.client_errors += 1
    elif 500 <= status <= 599:
        stats.server_errors += 1

responses = [200, 201, 404, 500, 200, 403, 502]
stats = RequestStats()
for code in responses:
    tally_response(stats, code)

print(stats)
# RequestStats(total=7, success=3, client_errors=2, server_errors=2)
```

### Chained comparisons

Python allows chained comparisons that read like mathematical range notation. The expression `200 <= status <= 299` is equivalent to `status >= 200 and status <= 299`, but more readable and evaluates the middle operand only once. Most other languages do not support this syntax:

```python
status = 201

# Chained comparison — unambiguous range check:
if 200 <= status <= 299:
    print("success")

# Works for any comparable type — useful for grading, thresholds, or validation:
score = 85
if 60 <= score < 70:
    grade = "D"
elif 70 <= score < 80:
    grade = "C"
elif 80 <= score < 90:
    grade = "B"
else:
    grade = "A"
```

## Control flow

### if, elif, else

Python evaluates `if`/`elif`/`else` branches in order and runs the first branch whose condition is `True`. Only one branch runs:

```python
name = 'Smith'

if name == 'Nelson':
    print('This is wrong')
elif name == 'Douglas':
    print('This is wrong')
elif name == 'Smith':
    print('This is right')
else:
    print('No answer')
```

### Ternary expressions

A *ternary expression* (also called a conditional expression) returns one of two values depending on a condition. Apply it when the entire expression fits on one readable line and there are exactly two outcomes:

```python
import os

env = os.getenv("APP_ENV", "development")

# Without ternary — four lines for a simple label:
# if env == "development":
#     log_level = "DEBUG"
# else:
#     log_level = "WARNING"

# With ternary — one line, reads naturally:
log_level = "DEBUG" if env == "development" else "WARNING"

db_url = os.getenv("DATABASE_URL") if env == "production" else "sqlite:///dev.db"
```

Avoid chaining ternary expressions. When a condition has more than two outcomes, an `if`/`elif`/`else` block is clearer.

### match / case

The `match`/`case` statement (Python 3.10 and later) replaces long `if`/`elif` chains and supports structural pattern matching — matching on values, types, and the shape of objects. The following example routes incoming HTTP requests to handler functions:

```python
from typing import TypedDict

class RouteResult(TypedDict):
    handler: str
    allow_body: bool

def route_request(method: str, path: str) -> RouteResult:
    match method.upper():
        case "GET" | "HEAD":
            return {"handler": f"read_{path.strip('/').replace('/', '_')}", "allow_body": False}
        case "POST":
            return {"handler": f"create_{path.strip('/').replace('/', '_')}", "allow_body": True}
        case "PUT" | "PATCH":
            return {"handler": f"update_{path.strip('/').replace('/', '_')}", "allow_body": True}
        case "DELETE":
            return {"handler": f"delete_{path.strip('/').replace('/', '_')}", "allow_body": False}
        case _:
            raise ValueError(f"Unsupported HTTP method: {method}")

print(route_request("GET", "/users"))    # {'handler': 'read_users', 'allow_body': False}
print(route_request("POST", "/orders"))  # {'handler': 'create_orders', 'allow_body': True}
```

The `_` case is the default — it matches anything not matched above. The `|` operator matches multiple values in a single case.

## Loops

### while loop

A `while` loop runs as long as its condition is `True`. The following example counts from 0 to 4:

```python
count = 0
while count < 5:
    print(count)
    count += 1
# 0 1 2 3 4
```

### break, continue, and else in loops

`break` exits a loop immediately. `continue` skips the rest of the current iteration and moves to the next one. The `else` clause on a loop runs only when the loop completes without hitting a `break` — making it useful for "search, and report not found" patterns.

The following example searches a list of database servers for the first one that accepts a connection. The `else` block runs only when every server failed:

```python
import socket
from typing import Optional

SERVERS = [
    ("db-primary.internal", 5432),
    ("db-replica-1.internal", 5432),
    ("db-replica-2.internal", 5432),
]

def find_healthy_server(timeout: float = 1.0) -> Optional[tuple[str, int]]:
    for host, port in SERVERS:
        try:
            with socket.create_connection((host, port), timeout=timeout):
                print(f"Connected to {host}:{port}")
                break  # found a healthy server — stop searching
        except (socket.timeout, ConnectionRefusedError, OSError):
            continue   # this server is down — try the next one
    else:
        # Runs only when the loop exhausted all servers without hitting break.
        return None

    return (host, port)
```

### for and in with iterators

The `for`/`in` loop iterates over any *iterable* — an object that produces values one at a time. Strings, lists, tuples, and dictionaries are all iterables:

```python
sentence = 'A string is an iterable'
for letter in sentence:
    print(letter)
```

### enumerate()

When you need both the position and the value during iteration, apply `enumerate()` rather than tracking an index variable manually. It yields `(index, value)` pairs. The following example builds a side-by-side diff of two configuration files:

```python
def build_diff_lines(original: list[str], revised: list[str]) -> None:
    # enumerate(iterable, start=1) yields (line_number, value) pairs.
    # zip() pairs the two lists by position, stopping at the shorter one.
    for lineno, (old, new) in enumerate(zip(original, revised), start=1):
        if old != new:
            print(f"  line {lineno:>4}: - {old.rstrip()}")
            print(f"  line {lineno:>4}: + {new.rstrip()}")

v1 = ["host: localhost\n", "port: 5432\n", "pool_size: 10\n"]
v2 = ["host: db.prod.internal\n", "port: 5432\n", "pool_size: 25\n"]
build_diff_lines(v1, v2)
# line    1: - host: localhost
# line    1: + host: db.prod.internal
# line    3: - pool_size: 10
# line    3: + pool_size: 25
```

### range() to generate numbers

`range()` returns a stream of integers within a specified range without allocating a sequence in memory. It returns an iterable object you can step through with a `for`/`in` loop or convert to a list:

```python
# range(start, stop, step)

# Standard iteration from 0 to 4:
for x in range(0, 5):
    print(x)
# 0 1 2 3 4

# Convert to a list:
list(range(0, 5))
# [0, 1, 2, 3, 4]

# Count down with a negative step:
for x in range(5, -1, -1):
    print(x)
# 5 4 3 2 1 0
```
