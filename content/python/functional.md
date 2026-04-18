---
title: "Functional programming"
weight: 100
---

Python supports functional programming patterns: passing functions as arguments, applying functions to sequences, and composing small functions into pipelines. The built-in `map()`, `filter()`, and `reduce()` functions are the main tools — though list comprehensions often express the same ideas more clearly.

## map()

`map(function, iterable)` applies a function to every item in an iterable and returns a lazy iterator. Convert it to a `list` when you need all results at once.

The following example parses structured log lines into typed records. It shows both the `map()` version and the equivalent comprehension:

```python
import re
from typing import NamedTuple
from datetime import datetime

class LogEntry(NamedTuple):
    timestamp: datetime
    level: str
    message: str

RAW_LOGS = [
    "2024-03-15T10:23:01 ERROR  Disk quota exceeded on /var/log",
    "2024-03-15T10:23:05 INFO   Health check passed",
    "2024-03-15T10:23:09 ERROR  Connection pool exhausted",
    "2024-03-15T10:23:12 WARN   Retry attempt 3/5 for job-884",
]

def parse_log_line(line: str) -> LogEntry:
    ts_str, level, message = re.split(r"\s+", line.strip(), maxsplit=2)
    return LogEntry(datetime.fromisoformat(ts_str), level, message)

# With map() and a named function:
entries = list(map(parse_log_line, RAW_LOGS))

# Equivalent list comprehension — preferred when readability matters:
entries = [parse_log_line(line) for line in RAW_LOGS]
```

Apply `map()` with a lambda for simple one-line transformations:

```python
service_names = ["api-gateway", "auth-service", "data-pipeline"]

# Normalize names to uppercase environment variable keys:
config_keys = list(map(lambda s: s.upper().replace("-", "_"), service_names))
# ['API_GATEWAY', 'AUTH_SERVICE', 'DATA_PIPELINE']

# Comprehension equivalent:
config_keys = [s.upper().replace("-", "_") for s in service_names]
```

## filter()

`filter(function, iterable)` keeps only the items for which the function returns `True`. Like `map()`, it returns a lazy iterator:

```python
entries = [parse_log_line(line) for line in RAW_LOGS]

# With filter() and a lambda:
errors = list(filter(lambda e: e.level == "ERROR", entries))

# Equivalent comprehension:
errors = [e for e in entries if e.level == "ERROR"]

# filter() with a named function — useful when the predicate is complex:
def is_recent(entry: LogEntry, cutoff: datetime) -> bool:
    return entry.timestamp >= cutoff

recent_errors = [
    e for e in entries
    if e.level == "ERROR" and is_recent(e, datetime(2024, 3, 15, 10, 23, 5))
]
```

## reduce()

`reduce(function, iterable[, initializer])` from `functools` accumulates a sequence into a single value. It applies the function to the first two items, then to the result and the third item, and so on.

The following example builds a single string of all error messages:

```python
from functools import reduce

errors = [e for e in entries if e.level == "ERROR"]

# With reduce():
def append_message(acc: str, entry: LogEntry) -> str:
    return acc + "\n" + entry.message

error_summary = reduce(append_message, errors, "Error summary:")

# The join() pattern is clearer and more Pythonic for string accumulation:
error_summary = "Error summary:\n" + "\n".join(e.message for e in errors)
```

`reduce()` is most appropriate when the accumulation logic has no simpler built-in equivalent. The following example merges a list of partial configuration dictionaries, where later layers override earlier ones:

```python
from functools import reduce

config_layers = [
    {"host": "localhost", "port": 5432, "pool_size": 10},
    {"host": "db.prod.internal"},         # production host override
    {"pool_size": 25, "ssl": True},       # tuning override
]

# Each layer overwrites keys from the previous merged result:
final_config = reduce(lambda merged, layer: {**merged, **layer}, config_layers)
# {'host': 'db.prod.internal', 'port': 5432, 'pool_size': 25, 'ssl': True}
```

## Choosing the right tool

The following table summarizes when to apply each functional tool:

| Tool | Apply when |
|---|---|
| List comprehension | You need a list and the logic reads clearly inline |
| Generator expression | You only need to iterate once and memory matters |
| `map()` | You already have a named function and want to apply it across an iterable |
| `filter()` | You already have a named predicate and want to keep matching items |
| `reduce()` | You need to fold a sequence into a single value and no simpler built-in exists |

Python's preference is for comprehensions — they are more readable, slightly faster, and require no imports. Reach for `map()` and `filter()` when you already have a named function that would read awkwardly inside a comprehension. Avoid `reduce()` unless the fold operation has no simpler built-in equivalent — `sum()`, `max()`, `min()`, and `str.join()` cover the most common cases.
