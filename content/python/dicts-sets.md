---
title: "Dictionaries and sets"
weight: 60
---

## Dictionaries

Dictionaries store key-value pairs. Keys must be hashable — strings, numbers, or tuples of hashable types. Values can be any type. Python 3.7 and later preserve insertion order.

### Creating dictionaries

The following examples show the standard ways to create dictionaries:

```python
# With curly braces:
empty_dict = {}
beatles = {
    "John": "Lennon",
    "Paul": "McCartney",
    "George": "Harrison",
    "Ringo": "Starr",
}

# With dict() — keyword arguments become string keys:
poet = dict(first='Edgar', middle='Alan', last='Poe')
# {'first': 'Edgar', 'middle': 'Alan', 'last': 'Poe'}
```

Convert two-value sequences into a dictionary by passing a list of pairs to `dict()`:

```python
# Two-item tuples:
tups = [('a', 'z'), ('b', 'y'), ('c', 'x')]
dict(tups)
# {'a': 'z', 'b': 'y', 'c': 'x'}

# Tuple of two-item lists:
tup_of_lis = (['a', 'z'], ['b', 'y'], ['c', 'x'])
dict(tup_of_lis)
# {'a': 'z', 'b': 'y', 'c': 'x'}

# Two-character strings — first character becomes the key, second the value:
two_char_str = ['ab', 'cd', 'ef']
dict(two_char_str)
# {'a': 'b', 'c': 'd', 'e': 'f'}
```

### Adding and changing items

If the key does not exist, the assignment adds it. If it does, the assignment replaces the value:

```python
beatles = {
    'Lennon': 'John',
    'McCartney': 'Paul',
    'Harrison': 'George',
    'Starr': 'Ringo',
}

# Add a new key:
beatles['Preston'] = 'Billy'

# Add, then correct a value:
beatles['Clapton'] = 'Erik'
beatles['Clapton'] = 'Eric'   # replaces the typo
```

### Getting items

The following examples cover the common ways to read from a dictionary:

```python
# Get by key — raises KeyError if missing:
beatles['Lennon']   # 'John'

# Get with a default — returns the default instead of raising:
beatles.get('Martin')                    # None
beatles.get('Martin', 'Not a Beatle')   # 'Not a Beatle'
beatles.get('Harrison')                  # 'George'

# Get all keys:
beatles.keys()   # dict_keys(['Lennon', 'McCartney', 'Harrison', 'Starr', ...])

# Get all values:
beatles.values()   # dict_values(['John', 'Paul', 'George', 'Ringo', ...])

# Store keys or values in a list:
list(beatles.values())   # ['John', 'Paul', 'George', 'Ringo']

# Get as a list of (key, value) tuples:
list(beatles.items())
# [('Lennon', 'John'), ('McCartney', 'Paul'), ...]

# Length:
len(beatles)   # 4

# Combine two dicts with ** — later keys override earlier ones:
stones     = {'Jagger': 'Mick', 'Richards': 'Keith', 'Watts': 'Charlie', 'Wyman': 'Bill'}
supergroup = {**beatles, **stones}

# Combine with update() — modifies the dict in place:
ruttles = {}
ruttles.update(beatles)

# Delete by key:
del beatles['Preston']

# Remove and return with pop():
beatles.pop('Clapton')   # returns 'Eric'

# Delete all items:
ruttles.clear()

# Check for a key:
'Lennon' in beatles   # True

# Copy:
ruttles = beatles.copy()

# Compare:
beatles == ruttles   # True
```

### Iterating through dictionaries

A `for`/`in` loop over a dictionary iterates over keys by default. Apply `.values()` to iterate over values, and `.items()` to iterate over key-value pairs:

```python
for guy in beatles:
    print(guy)
# Lennon McCartney Harrison Starr

for guy in beatles.values():
    print(guy)
# John Paul George Ringo

# Unpack each item into named variables:
for last, first in beatles.items():
    print(f"{first}'s last name is {last}")
# John's last name is Lennon
# Paul's last name is McCartney
# George's last name is Harrison
# Ringo's last name is Starr
```

### Building grouped structures with defaultdict

A *defaultdict* from the `collections` module creates missing keys automatically using a factory function. This eliminates the `KeyError` that occurs when you append to a key that does not yet exist. Apply it when grouping items by a category:

```python
from collections import defaultdict

# Group log entries by severity level:
raw_logs = [
    ("ERROR",   "Disk quota exceeded"),
    ("INFO",    "Health check passed"),
    ("ERROR",   "Connection pool exhausted"),
    ("WARNING", "Retry attempt 3/5"),
    ("INFO",    "Scheduled backup completed"),
]

by_level: defaultdict[str, list[str]] = defaultdict(list)
for level, message in raw_logs:
    by_level[level].append(message)  # no KeyError — missing keys get an empty list

# {'ERROR': ['Disk quota exceeded', 'Connection pool exhausted'],
#  'INFO':  ['Health check passed', 'Scheduled backup completed'],
#  'WARNING': ['Retry attempt 3/5']}
```

### Counting with Counter

`Counter` from the `collections` module counts occurrences of each element in an iterable. It returns a dictionary subclass where keys are elements and values are counts:

```python
from collections import Counter

# Count word frequencies in a document:
text = "the quick brown fox jumps over the lazy dog the fox"
word_counts = Counter(text.split())
# Counter({'the': 3, 'fox': 2, 'quick': 1, ...})

word_counts.most_common(3)
# [('the', 3), ('fox', 2), ('quick', 1)]

# Count HTTP status codes from a request log:
status_codes = [200, 404, 200, 500, 200, 404, 200]
code_counts  = Counter(status_codes)
# Counter({200: 4, 404: 2, 500: 1})
```

### Dictionary comprehensions

*Dictionary comprehensions* build a dictionary from an iterable in a single expression. The format is:

`{key_expression: value_expression for expression in iterable}`

The following example counts character occurrences in a word:

```python
word = 'better'
better_count = {letter: word.count(letter) for letter in word}
# {'b': 1, 'e': 2, 't': 2, 'r': 1}
```

Add a condition after the iterable to filter which items are included:

`{key_expression: value_expression for expression in iterable if condition}`

```python
vowels = 'aeiou'
word   = 'superpower'
vowel_counts = {letter: word.count(letter) for letter in set(word) if letter in vowels}
# {'o': 1, 'u': 1, 'e': 2}
```

## Sets

A *set* is an unordered collection of unique, hashable values. Sets are useful for membership testing, deduplication, and computing relationships between groups — union, intersection, and difference.

### Creating sets

Apply curly braces to create a set, or pass any iterable to `set()`. Sets eliminate duplicates automatically:

```python
empty_set = set()   # {} creates an empty dict, not an empty set
evens = {2, 4, 6, 8}
odds  = {1, 3, 5, 7, 9}

# Convert other data structures to sets — duplicates are dropped:
set('letters')
# {'e', 's', 't', 'r', 'l'}

set(['Leonardo', 'Donatello', 'Raphael', 'Michaelangelo'])
# {'Leonardo', 'Michaelangelo', 'Donatello', 'Raphael'}

set(('dog', 'cat', 'fish'))
# {'cat', 'fish', 'dog'}

# From a dict — only keys are included:
set({'John': 'Lennon', 'Paul': 'McCartney', 'George': 'Harrison', 'Ringo': 'Starr'})
# {'John', 'George', 'Ringo', 'Paul'}
```

### Set functions

The following examples cover adding, removing, and measuring set contents:

```python
evens = {2, 4, 6, 8}
len(evens)           # 4
evens.add(10)        # {2, 4, 6, 8, 10}
evens.remove(10)     # raises KeyError if not present
evens.discard(99)    # no error if not present
```

### Deduplication and set comparison

Converting a list to a set removes duplicates. The following example finds unique signups and identifies which users already exist in the system:

```python
new_signups    = ["alice@example.com", "bob@example.com", "alice@example.com", "carol@example.com"]
existing_users = ["bob@example.com", "dave@example.com"]

unique_signups = list(set(new_signups))
# ['alice@example.com', 'bob@example.com', 'carol@example.com']

already_exists = set(new_signups) & set(existing_users)
# {'bob@example.com'}

truly_new = set(new_signups) - set(existing_users)
# {'alice@example.com', 'carol@example.com'}
```

### Iterating and filtering with sets

Sets are often nested inside dictionaries when values represent groups. The following pizza menu example demonstrates iteration and set-based filtering:

```python
menu = {
    'classic': {'pepperoni', 'cheese'},
    'italian': {'sausage', 'peppers', 'onions'},
    'veggie':  {'peppers', 'onions', 'mushrooms', 'olives'},
    'supreme': {'pepperoni', 'ham', 'beef', 'sausage', 'peppers', 'onions', 'mushrooms', 'olives'}
}

# Find pizzas that include onions:
for pizza, toppings in menu.items():
    if 'onions' in toppings:
        print(pizza)
# italian, veggie, supreme

# Find pizzas with onions but without sausage:
for pizza, toppings in menu.items():
    if 'onions' in toppings and not ('sausage' in toppings):
        print(pizza)
# veggie
```

### Set operators

The following examples demonstrate the set operators. Each operator has both a symbolic form and an equivalent method:

```python
fave  = menu['italian']   # {'peppers', 'onions', 'sausage'}
worst = menu['veggie']    # {'peppers', 'mushrooms', 'onions', 'olives'}
best  = menu['supreme']

# Intersection (&) — items in both sets:
fave & worst              # {'peppers', 'onions'}
fave.intersection(worst)  # {'peppers', 'onions'}

# Union (|) — all items from both sets:
fave | worst              # {'peppers', 'olives', 'onions', 'sausage', 'mushrooms'}
fave.union(worst)         # same result

# Difference (-) — items in the left set but not the right:
fave - worst              # {'sausage'}

# Symmetric difference (^) — items in one set but not both:
fave ^ worst              # {'olives', 'sausage', 'mushrooms'}

# Intersection with the & operator to filter a dict loop:
for pizza, toppings in menu.items():
    if toppings & {'peppers', 'onions'}:   # any overlap triggers True
        print(pizza)
# italian, veggie, supreme

# Subset (<=) — True if all items in fave are also in best:
fave <= best              # True
fave.issubset(best)       # True

# Proper subset (<) — True if fave is a subset and not equal to best:
fave < best               # True

# Superset (>=):
fave.issuperset(worst)    # False
```

### Set comprehensions

*Set comprehensions* build a set from an iterable. The format is:

`{expression for expression in iterable}`

Add a condition after the iterable to filter items:

`{expression for expression in iterable if condition}`

```python
even_set = {number for number in range(1, 20) if number % 2 == 0}
# {2, 4, 6, 8, 10, 12, 14, 16, 18}
```

### Immutable sets

Apply `frozenset()` to create a set whose contents cannot change after creation. Because they are hashable, frozensets can be used as dictionary keys:

```python
frozen = frozenset([1, 2, 3])
# frozenset({1, 2, 3})

frozen.remove(3)   # AttributeError: 'frozenset' object has no attribute 'remove'
frozen.add(4)      # AttributeError: 'frozenset' object has no attribute 'add'

# Frozensets as dictionary keys — useful for representing unordered groups:
topology = {
    frozenset({"web", "api"}):   "front-tier",
    frozenset({"db", "cache"}):  "data-tier",
}
```
