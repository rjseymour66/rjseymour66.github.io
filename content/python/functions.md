---
title: "Functions"
weight: 70
---

Functions are the primary unit of reusable logic in Python. When you call a function with arguments, Python copies the argument values into the function's parameters. Because Python is dynamically typed, parameters accept any type — type hints make the expected types explicit without enforcing them at runtime.

## Basic functions

### Defining and calling functions

Python requires the `pass` statement in a function body that does nothing:

```python
def nothing():
    pass

nothing()
```

Positional argument values are copied to parameters in the order they are listed:

```python
def menu(wine: str, entree: str, dessert: str) -> dict[str, str]:
    return {'wine': wine, 'entree': entree, 'dessert': dessert}

menu('Pinot noir', 'Ribeye', 'Chocolate cake')
# {'wine': 'Pinot noir', 'entree': 'Ribeye', 'dessert': 'Chocolate cake'}
```

### Default parameter values

A default value is applied when the caller omits the corresponding argument:

```python
def menu(wine: str, entree: str, dessert: str = 'ice cream') -> dict[str, str]:
    return {'wine': wine, 'entree': entree, 'dessert': dessert}

menu('Chardonnay', 'Tilapia')
# {'wine': 'Chardonnay', 'entree': 'Tilapia', 'dessert': 'ice cream'}
```

Do not apply mutable values such as lists as default values. Python creates the default value once when it compiles the function definition, not on each call — so every call shares the same list object:

```python
# Problem: the list persists across calls
def todo(arg, tasks=[]):
    tasks.append(arg)
    print(tasks)

todo('shop')    # ['shop']
todo('clean')   # ['shop', 'clean'] — the list grew between calls

# Fix: use None and create a fresh list inside the function body
def todo(arg: str, result: list[str] | None = None) -> None:
    if result is None:
        result = []
    result.append(arg)
    print(result)

todo('shop')    # ['shop']
todo('clean')   # ['clean'] — fresh list each call
```

### None vs False

`None` represents the absence of a value. It is not the same as `False`, `0`, or an empty string — even though all of those are falsy in a boolean context. Apply `is` (not `==`) to test for `None`:

```python
test = None
if test is None:
    print('Use the is operator')
```

### Type hints

*Type hints* annotate function signatures with the expected types of parameters and return values. Python does not enforce them at runtime, but static analysis tools such as mypy and editors use them to catch type errors before the code runs. The following function parses a configuration entry and returns `None` if the entry is missing or the file does not exist:

```python
from pathlib import Path
from typing import Optional, Union

ConfigValue = Union[str, int, float, bool]

def get_config_value(
    config_path: Path,
    key: str,
    default: Optional[ConfigValue] = None,
) -> Optional[ConfigValue]:
    """Read a single key from a KEY=VALUE config file.

    Returns the value cast to the most specific type, or default if the key
    is absent. Returns None if the file does not exist.
    """
    if not config_path.exists():
        return None

    with config_path.open() as fh:
        for line in fh:
            line = line.strip()
            if not line or line.startswith("#"):
                continue
            if "=" not in line:
                continue
            k, _, raw_value = line.partition("=")
            if k.strip() == key:
                return _coerce(raw_value.strip())

    return default
```

## Variable arguments

### * parameter

The `*args` parameter captures a variable number of positional arguments into a tuple. The name `args` is conventional but not required:

```python
def print_args(*args) -> None:
    print('positional tuple:', args)

print_args('one', 1, ['two', 'three'], 2, 'four')
# positional tuple: ('one', 1, ['two', 'three'], 2, 'four')

# Named positional parameters are satisfied first — the rest go into args:
def multi_args(a1, a2, *args) -> None:
    print('First:\t', a1)
    print('Second:\t', a2)
    print('Rest:\t',  args)

multi_args('one', 'two', 3, 4, 5, 6, 7, 8, 9)
# First:   one
# Second:  two
# Rest:    (3, 4, 5, 6, 7, 8, 9)
```

### ** keyword argument parameter

The `**kwargs` parameter captures keyword arguments into a dictionary. The name `kwargs` is conventional:

```python
def print_kwargs(**kwargs) -> None:
    print('Keyword arguments:', kwargs)

print_kwargs()
# Keyword arguments: {}

print_kwargs(first=1, second=2, third=3)
# Keyword arguments: {'first': 1, 'second': 2, 'third': 3}
```

### Combining *args and **kwargs

The following example shows a realistic function that accepts both. It writes structured audit log entries and passes arbitrary key-value context to the log record — a pattern common in observability and security tooling:

```python
import logging
from typing import Any, Optional

logger = logging.getLogger(__name__)

def audit_log(
    event: str,
    *args: Any,
    level: str = "INFO",
    user_id: Optional[str] = None,
    **context: Any,
) -> None:
    """Write a structured audit log entry.

    Positional args are passed through to the message formatter, the same
    way logging.info("user %s logged in", username) works internally.
    Keyword context captures arbitrary key=value fields for structured logging.
    """
    log_level = getattr(logging, level.upper(), logging.INFO)
    message   = event % args if args else event
    extra: dict[str, Any] = {"context": context}
    if user_id is not None:
        extra["user_id"] = user_id
    logger.log(log_level, message, extra=extra)

audit_log("User %s logged in from %s", "alice", "192.168.1.1", user_id="u-42", region="us-east-1")
audit_log("Payment failed", level="ERROR", user_id="u-99", amount=49.99, currency="USD")
```

## Docstrings

Add docstrings to document your functions. Apply a single string for a one-line summary. For multi-line docstrings, place the opening and closing triple quotes on their own lines:

```python
def printer(arg):
    'printer prints any argument to the console'
    print(arg)

def print_if_true(arg, check):
    '''
    Prints the first argument if a second argument is true.
    The operation is:
        1. Check whether the *second* argument is true.
        2. If it is, print the *first* argument.
    '''
    if check:
        print(arg)

# Access the docstring through help() or the __doc__ attribute:
help(print_if_true)
print(print_if_true.__doc__)
```

## First-class functions

Every function in Python is an object. Assign functions to variables, pass them as arguments, and return them from other functions. Do not add `()` when passing a function as a value — `()` calls it immediately:

```python
def name():
    print('My name is Bob.')

def do_func(func):
    func()

do_func(name)      # My name is Bob.

test = name
do_func(test)      # My name is Bob.

type(name)         # <class 'function'>
```

## Inner functions and closures

Define a function inside another function to take advantage of lexical scoping.

A *closure* is a function generated by another function that remembers the values of variables from the enclosing scope — even after the outer function returns. The following example creates an announcer that captures a phrase at construction time:

```python
def cue_card(phrase: str):
    def announcer():
        return f'The first guest tonight is {phrase}'
    return announcer

a = cue_card('Elvis')
b = cue_card('Ozzy Osbourne')

a()   # 'The first guest tonight is Elvis'
b()   # 'The first guest tonight is Ozzy Osbourne'
```

## Lambdas

A *lambda* is an anonymous function expressed as a single statement. It accepts zero or more comma-separated arguments, followed by a colon and an expression. Apply lambdas when passing a short function as an argument to another function:

```python
# lambda arg1[, arg2, ...]: expression

def alter_list(alist: list[str], func) -> None:
    for item in alist:
        print(func(item))

names = ['john', 'paul', 'george', 'ringo']

# With a named function:
def upper(word: str) -> str:
    return word.capitalize()

alter_list(names, upper)

# With a lambda — same result, no separate function definition needed:
alter_list(names, lambda word: word.capitalize())
# John Paul George Ringo
```

## Generators

A *generator* is a function that produces a sequence of values lazily — one at a time — rather than building the entire sequence in memory. Apply `yield` instead of `return` to define a generator. Python suspends execution at each `yield` and resumes from that point on the next iteration:

```python
def new_range(first: int = 0, last: int = 10, step: int = 1):
    number = first
    while number < last:
        yield number
        number += step

test = new_range(1, 5)
type(test)   # <class 'generator'>

for n in test:
    print(n)
# 1 2 3 4
```

Python's built-in `range()` is itself a generator — it does not allocate a list. The same principle applies to generator expressions. For example, summing file sizes with a generator expression processes one file at a time rather than building a list of all sizes first:

```python
from pathlib import Path

# Generator expression — yields one file size at a time:
total_bytes = sum(
    f.stat().st_size
    for f in Path("/var/log").glob("*.log")
    if f.is_file()
)
```

## Decorators

A *decorator* is a function that takes another function as input and returns a modified version of it. Decorators apply cross-cutting behavior — logging, timing, retrying — without modifying the original function's logic.

The following example wraps a function to log its arguments and return value:

```python
def document_func(func):
    def new_function(*args, **kwargs):
        print('Running function:', func.__name__)
        print('Positional args:', args)
        print('Keyword args:', kwargs)
        result = func(*args, **kwargs)
        print('Result:', result)
        return result
    return new_function

def add_ints(a, b):
    return a + b

doc_add_ints = document_func(add_ints)
doc_add_ints(3, 5)
# Running function: add_ints
# Positional args: (3, 5)
# Keyword args: {}
# Result: 8
```

Apply the `@decorator_name` syntax to decorate a function at definition time:

```python
@document_func
def add_ints(a, b):
    return a + b

add_ints(4, 5)
# Running function: add_ints
# Result: 9

# Stack multiple decorators — Python applies them bottom-up:
@document_func
@another_decorator
def add_ints(a, b):
    return a + b
```

### functools.wraps

Always apply `@functools.wraps(func)` inside a decorator. Without it, the wrapper function replaces the original's `__name__`, `__doc__`, and other metadata — all decorated functions appear as `wrapper` in stack traces and `help()` output:

```python
import time
import logging
import functools
from typing import Callable, TypeVar, ParamSpec

P = ParamSpec("P")
R = TypeVar("R")
logger = logging.getLogger(__name__)

def timed(func: Callable[P, R]) -> Callable[P, R]:
    @functools.wraps(func)   # copies __name__, __doc__, etc. onto the wrapper
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        start   = time.perf_counter()
        result  = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.info("%s completed in %.3fs", func.__name__, elapsed)
        return result
    return wrapper
```

### Decorator factories

A *decorator factory* is a function that returns a decorator. Apply it when the decorator needs configuration parameters. The following example retries a function on network errors — a common pattern for API clients and database calls:

```python
def retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    exceptions: tuple[type[Exception], ...] = (Exception,),
):
    """Retry a function up to max_attempts times on specified exceptions."""
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @functools.wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            last_exc: Exception
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as exc:
                    last_exc = exc
                    if attempt < max_attempts:
                        logger.warning(
                            "%s failed (attempt %d/%d): %s",
                            func.__name__, attempt, max_attempts, exc,
                        )
                        time.sleep(delay)
            raise last_exc
        return wrapper
    return decorator

@timed
@retry(max_attempts=3, delay=0.5, exceptions=(ConnectionError, TimeoutError))
def fetch_config(url: str) -> dict[str, str]:
    import urllib.request, json
    with urllib.request.urlopen(url, timeout=5) as resp:
        return json.load(resp)
```

`@timed` measures the total wall-clock time including all retry attempts because Python applies stacked decorators from the bottom up.
