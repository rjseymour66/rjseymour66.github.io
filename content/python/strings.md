---
title: "Strings"
weight: 40
---

Strings in Python are immutable sequences of Unicode characters. Every string method returns a new string — modifying a string always creates a new object rather than changing the original.

## Creating strings

Apply triple quotes for multi-line content such as SQL queries, email templates, or configuration blocks:

```python
# Single or double quotes for short strings:
greeting = "Hello, world"
name = 'Alice'

# Three single or double quotes for multi-line strings:
query = '''
SELECT id, email, created_at
FROM users
WHERE active = true
ORDER BY created_at DESC
'''

# Convert another data type to a string with str():
version = str(3.11)   # "3.11"
port    = str(5432)   # "5432"

# Multiply strings to repeat them:
name = 'Jack'
name * 4    # 'JackJackJackJack'

# Retrieve a single character by position (zero-based):
name[1]     # 'a'
```

## String methods

Python provides built-in methods for searching, testing, and transforming strings. The following examples operate on a common `song` variable:

```python
song = '''Happy birthday to you,
Happy birthday to you,
Happy birthday dear person,
Happy birthday to you!'''

song.startswith('Happy')    # True
song.endswith('everyone!')  # False

# Find the offset of the first occurrence — returns -1 if not found:
word = 'to'
song.find(word)    # 15

# Find the last occurrence — returns -1 if not found:
song.rfind(word)   # 89

# Count occurrences:
song.count(word)   # 3

# Test whether all characters are letters or numbers:
song.isalnum()     # False
```

### Case and whitespace methods

Apply these methods to normalize user input before storing or comparing it:

```python
phrase = 'you win some ...'

phrase.strip(' .')   # 'you win some'  — strips leading/trailing spaces and dots
phrase.capitalize()  # 'You win some ...'
phrase.title()       # 'You Win Some ...'
phrase.upper()       # 'YOU WIN SOME ...'
phrase.lower()       # 'you win some ...'
phrase.swapcase()    # 'YOU WIN SOME ...'
```

## Substring slicing

*Slicing* extracts a portion of a string by specifying start and end offsets. Offsets are zero-based, and the end offset is non-inclusive:

```python
offsets = '0123456789'

offsets[:]      # '0123456789' — the entire string
offsets[5:]     # '56789'      — from offset 5 to the end
offsets[:5]     # '01234'      — from the start up to (not including) offset 5
offsets[2:7]    # '23456'      — from offset 2 up to (not including) offset 7
offsets[2:7:2]  # '246'        — every second character from offset 2 to 6
```

## Built-in string functions

### len()

Apply `len()` to get the number of characters in a string:

```python
offsets = '0123456789'
len(offsets)   # 10
```

### split()

`split()` divides a string into a list on a separator. Without a separator, it splits on any whitespace:

```python
tasks = 'one,two,three,four'
tasks.split(',')   # ['one', 'two', 'three', 'four']
tasks.split()      # ['one,two,three,four']
```

### join()

`join()` collapses a list of strings into a single string. The string it is called on becomes the separator between items. Call it with the separator on the left and the list on the right:

```python
str_list = ['one', 'two', 'three', 'four']
'\n'.join(str_list)   # 'one\ntwo\nthree\nfour'
' '.join(str_list)    # 'one two three four'
```

### replace()

`replace()` performs simple substring substitution. It returns a new string — the original is unchanged:

```python
sub = 'This is the dumbest string ever'
sub.replace('dumb', 'cool')
# 'This is the coolest string ever'
```

## Formatting strings

Python provides two main approaches for string interpolation: `str.format()` and f-strings. F-strings are the standard choice in modern Python because they are more readable and are evaluated at runtime.

### str.format()

Apply `str.format()` when the template string is stored separately from the values — for example, in a configuration file or a translated message catalog:

```python
animal = 'dog'
place  = 'house'

# Positional placeholders:
'The {} is in the {}.'.format(animal, place)
# 'The dog is in the house.'

# Named placeholders:
'The {a} is in the {b}.'.format(a='cat', b='litter box')
# 'The cat is in the litter box.'
```

### f-strings

F-strings prefix the string literal with `f` and embed expressions directly inside `{}`. They are the standard choice for all inline string construction:

```python
animal = 'dog'
place  = 'house'

f'The {animal} is in the {place}'
# 'The dog is in the house'

# Expressions and method calls work inside the braces:
f'The {animal.capitalize()} is in the {place.rjust(20)}'
# 'The Dog is in the                house'
```

### f-string format specifiers

Format specifiers control alignment, padding, numeric precision, and display style. They follow the expression inside `{}` after a colon. The following example formats an invoice row for a fixed-width financial report:

```python
from dataclasses import dataclass

@dataclass
class LineItem:
    item_id: int
    name: str
    quantity: int
    unit_price: float

def format_invoice_row(item: LineItem) -> str:
    total = item.quantity * item.unit_price

    # :05d  — zero-pad an integer to 5 digits
    id_col    = f"{item.item_id:05d}"

    # :<30  — left-align in a 30-character field (pad right with spaces)
    name_col  = f"{item.name:<30}"

    # :>6   — right-align in a 6-character field
    qty_col   = f"{item.quantity:>6}"

    # :>12,.2f — right-align in 12 chars, thousands separator, 2 decimal places
    price_col = f"{item.unit_price:>12,.2f}"
    total_col = f"{total:>14,.2f}"

    return f"{id_col}  {name_col}  {qty_col}  {price_col}  {total_col}"

items = [
    LineItem(1,   "Cloud Storage (TB)",   12,    9.99),
    LineItem(42,  "API Calls (millions)",  3,  150.00),
    LineItem(107, "Support Tier Pro",      1, 1299.00),
]

header = f"{'ID':<5}  {'Description':<30}  {'Qty':>6}  {'Unit Price':>12}  {'Total':>14}"
print(header)
print("-" * len(header))
for item in items:
    print(format_invoice_row(item))
```

The following table summarizes the most common format specifiers:

| Specifier | Effect | Example | Output |
|---|---|---|---|
| `:<20` | Left-align in 20 chars | `f"{'name':<20}"` | `"name                "` |
| `:>20` | Right-align in 20 chars | `f"{'name':>20}"` | `"                name"` |
| `:^20` | Center in 20 chars | `f"{'name':^20}"` | `"        name        "` |
| `:05d` | Zero-pad integer to 5 digits | `f"{42:05d}"` | `"00042"` |
| `:,.2f` | Float with thousands separator and 2 decimals | `f"{1234.5:,.2f}"` | `"1,234.50"` |
| `:.2%` | Percentage with 2 decimals | `f"{0.875:.2%}"` | `"87.50%"` |
| `!r` | Call `repr()` on the value | `f"{val!r}"` | Quoted string with escape characters |
| `!s` | Call `str()` on the value | `f"{val!s}"` | Human-readable string |

Apply `!r` when debugging — it shows quotes and escape sequences that make the value unambiguous. For example, `f"{path!r}"` makes it clear whether a path contains a trailing slash.
