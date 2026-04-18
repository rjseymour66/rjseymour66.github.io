---
title: "Data structures"
weight: 50
---

Python provides four core sequence and collection types: tuples, lists, dictionaries, and sets. Choosing the right type affects both correctness and performance.

Tuples and lists both hold ordered sequences of any type. The key distinction is *mutability*:

- *Tuples* are immutable. Once created, their contents cannot change. Apply tuples for values that should not be modified — function return values, dictionary keys, or fixed configuration records.
- *Lists* are mutable. Apply lists when a collection's contents need to change over time — adding items, removing items, or sorting.

## Tuples

### Creating a tuple

Apply parentheses or a trailing comma to create a tuple. A trailing comma after a single value is required to distinguish a tuple from a grouped expression:

```python
empty_tuple = ()

# A trailing comma creates a single-item tuple:
one_stooge = 'Larry',
# ('Larry',)

# Multiple items — no trailing comma needed on the last item:
all_stooges = 'Larry', 'Curly', 'Moe'
# ('Larry', 'Curly', 'Moe')

# Parentheses make multi-item tuples easier to read:
parens_stooges = ('Larry', 'Curly', 'Moe')
```

### Tuple actions

Tuple unpacking assigns each element to a named variable in one statement:

```python
parens_stooges = ('Larry', 'Curly', 'Moe')
a, b, c = parens_stooges
# a = 'Larry', b = 'Curly', c = 'Moe'

# Swap two variables without a temporary variable:
one = 'one'
two = 'two'
one, two = two, one
# one = 'two', two = 'one'

# Build a tuple from a list:
stooge_list = ['Larry', 'Curly', 'Moe']
tuple(stooge_list)
# ('Larry', 'Curly', 'Moe')

# Combine tuples with +:
('Larry',) + ('Curly', 'Moe')
# ('Larry', 'Curly', 'Moe')

# Repeat items:
('howdy',) * 5
# ('howdy', 'howdy', 'howdy', 'howdy', 'howdy')

# Compare tuples — comparison proceeds element-by-element from left to right:
x = (1, 2, 3)
y = (2, 3, 4)
x == y   # False
x < y    # True
x <= y   # True

# Concatenating tuples always creates a new object:
first  = ('one', 'two', 'three')
second = ('four',)
third  = first + second
id(first) == id(third)   # False — different objects
```

### Extended unpacking

The `*` operator in an unpacking assignment captures remaining elements into a list. This pattern is useful when parsing structured records where the first few fields are fixed but the rest vary:

```python
# Parse a log line: "2024-03-15 ERROR db.py Connection refused"
parts = ("2024-03-15", "ERROR", "db.py", "Connection", "refused")
date, level, *message_parts = parts
message = " ".join(message_parts)
# date = "2024-03-15", level = "ERROR", message = "Connection refused"

# Take the first and last, capture everything between:
first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5
```

### Iterate through a tuple

Apply `for`/`in` to iterate through a tuple:

```python
nums = ('one', 'two', 'three', 'four')
for n in nums:
    print(n)
# one
# two
# three
# four
```

## Lists

Lists hold ordered, mutable collections of any type. Apply them when the contents need to change, when order matters, and when you need index-based access.

### Creating a list

The following examples show the most common ways to create a list:

```python
# Empty list with brackets:
empty_list = []

# Empty list with list():
empty_too = list()

# List with initial values:
beatles = ['John', 'Paul', 'George', 'Ringo']

# Create a list from an iterable — splits a string into individual characters:
list('individual')
# ['i', 'n', 'd', 'i', 'v', 'i', 'd', 'u', 'a', 'l']

# Create a list from a tuple:
tup = ('one', 'two', 'three')
list(tup)
# ['one', 'two', 'three']

# Create a list by splitting a delimited string:
adj = 'once-in-a-lifetime'
adj.split('-')
# ['once', 'in', 'a', 'lifetime']
```

### Getting list items

Retrieve items by offset or slice. Offsets are zero-based. Negative offsets count from the end:

```python
turtles = ['Leonardo', 'Donatello', 'Michaelangelo', 'Raphael']

# By offset:
turtles[2]    # 'Michaelangelo'
turtles[-1]   # 'Raphael' — the last item

# By slice:
turtles[0:2]  # ['Leonardo', 'Donatello']
```

### List functions

The following examples demonstrate the most common list operations:

```python
nums = ['one', 'two', 'three', 'four']

nums.append('five')              # add to the end
# ['one', 'two', 'three', 'four', 'five']

nums.insert(0, 'zero')           # insert at an index
# ['zero', 'one', 'two', 'three', 'four', 'five']

['blah'] * 3                     # repeat
# ['blah', 'blah', 'blah']

numeros = ['uno', 'dos', 'tres']
nums.extend(numeros)             # append all items from another list

nums[2] = 'too'                  # replace by index
nums[6:] = ['six', 'seven', 'eight']  # replace by slice

del nums[0]                      # delete by index
nums.remove('seven')             # delete the first occurrence of a value

nums.pop()                       # remove and return the last item
nums.pop(1)                      # remove and return by index

nums.clear()                     # delete all items

nums.index('two')                # find the index of a value
'three' in nums                  # True if value is present

nums.count('two')                # count occurrences of a value
', '.join(nums)                  # join to a delimited string
len(nums)                        # length

# sorted() returns a new sorted list — the original is unchanged:
alpha_nums = sorted(nums)

# .sort() sorts the list in place — the original is modified:
nums.sort()
```

### Copying lists

Assigning a list to a second variable does not create a copy — both names refer to the same object. Apply `.copy()`, `list()`, or a full slice to create an independent copy:

```python
a = ['one', 'two', 'three']
b = a          # b and a point to the same list — modifying b changes a
c = a.copy()   # independent copy
d = list(a)    # also an independent copy
e = a[:]       # also an independent copy

id(a) == id(b)   # True  — same object
id(a) == id(c)   # False — different objects

a == c           # True  — equal contents
```

### Iterating through lists

Apply `for`/`in` to iterate. Apply `zip()` to iterate two lists simultaneously — it pairs elements by position and stops at the shorter list:

```python
ls = ['one', 'two', 'three', 'four', 'five']
for i in ls:
    print(i)
# one two three four five

es = ['uno', 'dos', 'tres', 'quatro', 'cinco']
for english, spanish in zip(ls, es):
    print(english, ":\t", spanish)
# one :    uno
# two :    dos
# three :  tres
# four :   quatro
# five :   cinco
```

### Sorting with a key function

Pass a `key=` function to `sorted()` or `.sort()` to control sort order. The key function receives each element and returns the value to sort by. The following example sorts deployment records by timestamp:

```python
deployments = [
    {"service": "api",      "deployed_at": "2024-03-15T14:30:00"},
    {"service": "worker",   "deployed_at": "2024-03-15T09:15:00"},
    {"service": "frontend", "deployed_at": "2024-03-15T16:45:00"},
]

# Sort by the deployed_at field — the key function extracts the sort value:
by_time = sorted(deployments, key=lambda d: d["deployed_at"])
# worker (09:15), api (14:30), frontend (16:45)

# Sort in reverse — most recent first:
most_recent = sorted(deployments, key=lambda d: d["deployed_at"], reverse=True)
```

### List comprehensions

A *list comprehension* builds a list from an iterable in a single expression. The format is:

`[expression for item in iterable]`

This is equivalent to a `for` loop that appends to a list, but is more concise:

```python
ls = ['one', 'two', 'three']

# Capitalize each item:
upper = [item.capitalize() for item in ls]
# ['One', 'Two', 'Three']

# Generate a range of numbers:
num_ls = [n for n in range(0, 10)]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Apply an expression to each item:
index_adjusted = [n + 1 for n in num_ls]
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

Add a condition after the iterable to filter which items are included:

`[expression for item in iterable if condition]`

```python
animals = ['cat', 'dog', 'mouse', 'rat', 'bird']

three    = [a for a in animals if len(a) == 3]
# ['cat', 'dog', 'rat']

long_cap = [a.capitalize() for a in animals if len(a) != 3]
# ['Mouse', 'Bird']
```

Nest multiple iterables by listing them one after the other — Python evaluates them from left to right, like nested `for` loops:

```python
nums  = range(1, 4)
alpha = ['x', 'y']

# Equivalent to two nested for loops:
ls_comp = [(num, a) for num in nums for a in alpha]
# [(1, 'x'), (1, 'y'), (2, 'x'), (2, 'y'), (3, 'x'), (3, 'y')]

# Unpack tuples directly in the for clause:
for (num, a) in ls_comp:
    print(num, a)
```

### Generator expressions

A *generator expression* has the same syntax as a list comprehension but with parentheses instead of brackets. It produces values one at a time rather than building the entire list in memory. Apply generator expressions when you only need to iterate over results once — for example, passing them directly to `sum()`, `max()`, or `any()`:

```python
from pathlib import Path

# A list comprehension builds the full list before sum() runs:
total = sum([f.stat().st_size for f in Path("/var/log").glob("*.log")])

# A generator expression yields one size at a time — no list is built:
total = sum(f.stat().st_size for f in Path("/var/log").glob("*.log"))
```

For thousands of log files, the generator expression uses a fixed amount of memory regardless of how many files exist. The list comprehension allocates memory proportional to the number of files.

### Lists of lists

A list can contain other lists. Access nested items with chained bracket notation:

```python
evens = [2, 4, 6, 8, 10]
odds  = [1, 3, 5, 7, 9]
prime = [1, 7, 13, 19, 23]
nums  = [evens, odds, prime]

nums[0]      # [2, 4, 6, 8, 10]
nums[0][2]   # 6
```
