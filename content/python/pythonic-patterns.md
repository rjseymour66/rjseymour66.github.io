---
title: "Pythonic patterns"
weight: 130
---

Every language has idioms — patterns experienced developers reach for instinctively. In Python, these idioms are called *Pythonic*: code that is clear, concise, and takes advantage of what the language does best.

This guide covers the patterns you will encounter most often in production Python code. For each pattern, you will find an explanation of what it does, when to apply it, and a concrete example.

## Comprehensions

A *list comprehension* builds a new list by applying an expression to each item in an iterable, with an optional filter. It replaces a `for` loop and an `append` call with a single readable expression.

Compare the two approaches for extracting prices from a list of items:

```python
from dataclasses import dataclass


@dataclass
class Item:
    name: str
    price: float


items = [
    Item("Notebook",  2.50),
    Item("Stapler",   8.99),
    Item("Pen",       1.00),
]

# Loop approach
prices = []
for item in items:
    prices.append(item.price)

# Comprehension approach
prices = [item.price for item in items]
```

Add a condition after the `for` clause to filter the results:

```python
cheap = [item for item in items if item.price < 5.00]
```

### Dict and set comprehensions

The same syntax works for dictionaries and sets:

```python
# Map name to price
price_map = {item.name: item.price for item in items}
# {"Notebook": 2.50, "Stapler": 8.99, "Pen": 1.00}

# Unique price values — duplicates are discarded automatically
unique_prices = {item.price for item in items}
# {2.50, 8.99, 1.00}
```

### When to avoid comprehensions

Comprehensions improve readability when the logic is simple. Avoid them when the expression spans more than two lines or requires nested conditions. A plain `for` loop is easier to read at that point.

## Generator expressions

A *generator expression* looks like a list comprehension but produces values one at a time instead of building the entire list in memory. Write it with parentheses instead of brackets.

Apply generator expressions when you pass results directly to `sum`, `max`, `min`, or `any`:

```python
from pathlib import Path

# List comprehension: builds a complete list in memory first
total = sum([f.stat().st_size for f in Path("/var/log").glob("*.log")])

# Generator expression: yields one size at a time
total = sum(f.stat().st_size for f in Path("/var/log").glob("*.log"))
```

The difference becomes meaningful when processing thousands or millions of items. A generator expression holds one item at a time. A list comprehension holds all items simultaneously.

## Unpacking

Python lets you assign multiple variables in one statement by *unpacking* a sequence. The number of variables on the left must match the number of items in the sequence, or you must account for the remainder with a starred variable.

### Basic unpacking

```python
point = (3, 7)
x, y = point
# x = 3, y = 7

# Swap two variables without a temporary variable
a, b = 10, 20
a, b = b, a
# a = 20, b = 10
```

### Starred unpacking

A starred variable captures the remaining items as a list:

```python
first, *rest = [1, 2, 3, 4, 5]
# first = 1, rest = [2, 3, 4, 5]

*head, last = [1, 2, 3, 4, 5]
# head = [1, 2, 3, 4], last = 5

first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5
```

### Unpacking in loops

Unpack tuples directly in a `for` loop to avoid indexing:

```python
pairs = [("Alice", 95), ("Bob", 87), ("Carol", 92)]

# Without unpacking
for pair in pairs:
    print(pair[0], "scored", pair[1])

# With unpacking — intent is clear
for name, score in pairs:
    print(name, "scored", score)
```

## enumerate and zip

### enumerate

`enumerate` adds an index to any iterable. Apply it whenever you need both the index and the value in a loop. Do not maintain a counter variable manually:

```python
items = ["Notebook", "Stapler", "Pen"]

# Manual counter — avoid this
i = 0
for item in items:
    print(i, item)
    i += 1

# enumerate — idiomatic
for i, item in enumerate(items):
    print(i, item)

# Start the index at 1
for i, item in enumerate(items, start=1):
    print(f"{i}. {item}")
# 1. Notebook
# 2. Stapler
# 3. Pen
```

### zip

`zip` pairs items from two or more iterables together. Apply it to iterate over parallel sequences without indexing:

```python
names  = ["Alice", "Bob", "Carol"]
scores = [95, 87, 92]

for name, score in zip(names, scores):
    print(f"{name}: {score}")
# Alice: 95
# Bob: 87
# Carol: 92
```

`zip` stops at the shortest iterable. To iterate to the end of the longest, apply `itertools.zip_longest`.

## Chaining comparisons

Python lets you chain comparison operators in a single expression. This reads like natural language and avoids repeating the middle value:

```python
discount_pct = 15

# Two separate comparisons joined with `and`
if discount_pct >= 0 and discount_pct <= 100:
    ...

# Chained comparison — clearer and more Pythonic
if 0 <= discount_pct <= 100:
    ...
```

Chaining works with any mix of comparison operators:

```python
a = 5
assert 1 < a < 10      # a is between 1 and 10 exclusive
assert 0 <= a <= 100   # a is a valid percentage
```

## Conditional expressions

A *conditional expression* selects between two values based on a condition. Write it as `value_if_true if condition else value_if_false`:

```python
score = 87

# If/else block
if score >= 90:
    grade = "A"
else:
    grade = "B"

# Conditional expression — one line
grade = "A" if score >= 90 else "B"
```

Apply conditional expressions for simple, two-branch assignments. Avoid nesting them — a nested conditional expression is harder to read than a plain `if`/`elif`/`else` block.

## EAFP

Python favors *EAFP*: Easier to Ask Forgiveness than Permission. Instead of checking in advance whether an operation will succeed, try it and handle the exception if it fails.

The contrasting style is *LBYL* (Look Before You Leap), which checks conditions before acting. Compare them when reading a value from a dictionary:

```python
data = {"price": 9.99}

# LBYL: check before access
if "price" in data:
    price = data["price"]
else:
    price = 0.0

# EAFP: try it and handle the failure
try:
    price = data["price"]
except KeyError:
    price = 0.0

# Pythonic shorthand for this specific case
price = data.get("price", 0.0)
```

EAFP produces cleaner code when failures are rare and the success path is the common case. It also avoids a race condition in concurrent code: checking and acting happen as one step instead of two that another thread could interrupt between.

## Sorting with key functions

`sorted()`, `min()`, and `max()` all accept a `key` argument — a function applied to each item before comparison. The key function tells Python what to compare, not how to compare it.

Sort items by price:

```python
items = [
    Item("Stapler",  8.99),
    Item("Pen",      1.00),
    Item("Notebook", 2.50),
]

# Sort by price ascending
by_price = sorted(items, key=lambda item: item.price)
# [Pen 1.00, Notebook 2.50, Stapler 8.99]

# Sort by name, descending
by_name_desc = sorted(items, key=lambda item: item.name, reverse=True)

# Most expensive item
priciest = max(items, key=lambda item: item.price)
# Item(name='Stapler', price=8.99)
```

For simple attribute access, `operator.attrgetter` is slightly faster than a lambda and makes the intent more explicit:

```python
from operator import attrgetter

by_price = sorted(items, key=attrgetter("price"))
```

## The collections module

Python's `collections` module provides specialized data structures that solve common problems cleanly.

### Counter

*Counter* counts the frequency of items in an iterable. Apply it when you need to tally occurrences:

```python
from collections import Counter

words = ["apple", "banana", "apple", "cherry", "banana", "apple"]

counts = Counter(words)
# Counter({'apple': 3, 'banana': 2, 'cherry': 1})

counts.most_common(2)
# [('apple', 3), ('banana', 2)]
```

### defaultdict

A *defaultdict* is a dictionary that automatically creates a default value for missing keys. Pass a callable that produces the default when constructing it. This eliminates the "check if key exists, then create" pattern:

```python
from collections import defaultdict

catalog = [
    ("Kitchen", "Espresso machine"),
    ("Office",  "Notebook"),
    ("Office",  "Stapler"),
    ("Kitchen", "Blender"),
]

# Without defaultdict
by_category: dict[str, list[str]] = {}
for category, name in catalog:
    if category not in by_category:
        by_category[category] = []
    by_category[category].append(name)

# With defaultdict — the empty list is created automatically
by_category = defaultdict(list)
for category, name in catalog:
    by_category[category].append(name)

# {"Kitchen": ["Espresso machine", "Blender"], "Office": ["Notebook", "Stapler"]}
```

### namedtuple

A *namedtuple* is an immutable record type. It behaves like a regular tuple but lets you access fields by name. Apply it for lightweight data containers when you do not need methods or mutability:

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])

p = Point(x=3, y=7)
p.x    # 3
p.y    # 7
p[0]   # 3 — still accessible by index

# Unpack like a regular tuple
x, y = p
```

For data containers that need methods, default values, or type annotations, prefer `dataclasses.dataclass` over `namedtuple`. For simple, immutable records with no behavior, `namedtuple` is lighter and communicates intent clearly.

## The walrus operator

The *walrus operator* (`:=`) assigns a value and evaluates it as an expression in the same step. Apply it when you need the result of an expression both as a condition and as a value inside the block.

A common case is reading chunks from a file:

```python
from pathlib import Path

path = Path("data.bin")

# Without walrus: the read call is duplicated
with path.open("rb") as fh:
    chunk = fh.read(4096)
    while chunk:
        process(chunk)
        chunk = fh.read(4096)

# With walrus: one read call per iteration
with path.open("rb") as fh:
    while chunk := fh.read(4096):
        process(chunk)
```

Another common case is testing a regex match and working with the match object:

```python
import re

lines = [
    "ERROR 2024-01-15: disk full",
    "INFO  2024-01-15: backup complete",
    "ERROR 2024-01-16: connection refused",
]

pattern = re.compile(r"ERROR.*: (.+)")

for line in lines:
    if match := pattern.match(line):
        print("Found error:", match.group(1))
# Found error: disk full
# Found error: connection refused
```

Apply the walrus operator when it removes a clear redundancy. Avoid it when it makes the expression harder to follow — clarity matters more than brevity.
