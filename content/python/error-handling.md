---
title: "Error handling"
weight: 90
---

Python signals errors and unusual conditions through *exceptions*. When code raises an exception, Python unwinds the call stack looking for a matching `except` clause. If it finds none, the program terminates and prints a traceback.

## try and except

Wrap code that might fail in a `try` block. Provide one `except` clause for each exception type you expect:

```python
alist     = [0, 1, 2, 3, 4]
bad_index = 6

# Without error handling:
alist[bad_index]
# IndexError: list index out of range

# With error handling:
try:
    alist[bad_index]
except:
    print('Requires an index between 0 and', len(alist) - 1, 'but got', bad_index)
```

Capture the exception object by name to inspect its details. Provide a specific `except` for each error type and a general `except Exception` as a fallback:

```python
alist     = [1, 2, 3]
bad_index = 4

def exception_func(the_list: list, index: int) -> None:
    try:
        the_list[index]
    except IndexError as err:
        print('Requires index between 0 and', len(the_list) - 1, 'but got', index)
    except Exception as other:
        print('There was a different error:', other)

exception_func(alist, 5)      # IndexError path
exception_func(alist, 'dog')  # Exception path: list indices must be integers or slices, not str
```

### Common exception types

The following table describes the most frequently encountered built-in exceptions:

| Exception | Raised when | Typical response |
|---|---|---|
| `ValueError` | Argument has the right type but a wrong value | Validate input before calling |
| `TypeError` | Argument has the wrong type | Check types or add type hints |
| `KeyError` | Dictionary key does not exist | Apply `.get()` or check `in` first |
| `IndexError` | Sequence index is out of range | Check `len()` before indexing |
| `AttributeError` | Object has no such attribute | Verify the object type |
| `FileNotFoundError` | File or directory does not exist | Check existence or wrap in `try` |
| `OSError` | System-level failure (permissions, disk full) | Wrap in `try`; log and alert |
| `ConnectionError` | Network connection failed | Retry with backoff |
| `TimeoutError` | Operation exceeded its time limit | Retry or fail fast |

## try / except / else / finally

Python's full exception-handling construct has four clauses:

- `try`: the code that might fail
- `except`: handles a specific failure mode
- `else`: runs only when the `try` block succeeded without raising an exception
- `finally`: runs unconditionally — cleanup, logging, and resource release go here

The `else` clause is important: it keeps success logic out of the `try` block, making it clear that the code inside `else` only runs on a clean success path. The following example loads a remote config file with fallback and caching:

```python
import json
import urllib.request
import urllib.error
from pathlib import Path
from typing import Any

CACHE_PATH = Path("/tmp/remote_config_cache.json")

def load_remote_config(url: str) -> dict[str, Any]:
    config: dict[str, Any] = {}

    try:
        with urllib.request.urlopen(url, timeout=10) as response:
            raw    = response.read()
            config = json.loads(raw)

    except urllib.error.HTTPError as exc:
        # The server responded with an error status (4xx or 5xx).
        print(f"HTTP {exc.code} fetching config: {exc.reason}")
        if CACHE_PATH.exists():
            config = json.loads(CACHE_PATH.read_text())

    except urllib.error.URLError as exc:
        # Network failure: DNS error, refused connection, or timeout.
        print(f"Network error: {exc.reason}")
        if CACHE_PATH.exists():
            config = json.loads(CACHE_PATH.read_text())

    except json.JSONDecodeError as exc:
        # The server responded 200 but the body was not valid JSON.
        raise ValueError(f"Config server returned invalid JSON at offset {exc.pos}") from exc

    else:
        # Runs only when the try block completed without raising.
        # Write the fresh config to cache so the next failure has a fallback.
        CACHE_PATH.write_text(json.dumps(config, indent=2))
        print(f"Config loaded and cached ({len(config)} keys).")

    finally:
        # Always runs — a reliable place for metrics, audit logs, and cleanup.
        print(f"Config load attempt complete for {url}")

    return config
```

## Context managers

A *context manager* automates setup and teardown around a block of code. The `with` statement calls `__enter__()` when the block starts and `__exit__()` when it ends — whether or not an exception occurred. This guarantees resource cleanup without requiring manual `try`/`finally` blocks.

The most common context managers are file handles:

```python
# The file is closed automatically when the with block exits,
# even if an exception occurs inside it.
with open("config.json") as fh:
    data = json.load(fh)
```

### contextlib.contextmanager

The `@contextmanager` decorator from `contextlib` lets you write a generator function as a context manager. The `yield` statement marks the boundary between setup and teardown. Code before `yield` runs on entry; code after `yield` runs on exit:

```python
import os
import tempfile
import shutil
from contextlib import contextmanager
from pathlib import Path
from typing import Generator

@contextmanager
def temp_working_directory(prefix: str = "job_") -> Generator[Path, None, None]:
    """Create a temporary directory, change into it, and clean up on exit.

    The try/finally guarantees cleanup even when the caller raises an exception.
    This is the primary advantage of context managers over manual setup/teardown.
    """
    original_dir = Path.cwd()
    tmp_dir = Path(tempfile.mkdtemp(prefix=prefix))
    try:
        os.chdir(tmp_dir)
        yield tmp_dir       # execution pauses here while the caller's with-block runs
    finally:
        os.chdir(original_dir)
        shutil.rmtree(tmp_dir, ignore_errors=True)

@contextmanager
def managed_transaction(conn) -> Generator[None, None, None]:
    """Commit on clean exit; roll back on exception."""
    try:
        yield
        conn.commit()
    except Exception:
        conn.rollback()
        raise   # re-raise so the caller knows the transaction failed
```

Apply context managers whenever a resource must be released or a state must be restored — file handles, database connections, locks, temporary directories, and HTTP sessions.

## Defining custom exceptions

Python's standard library provides a wide range of built-in exceptions. Define your own by inheriting from `Exception`. Custom exceptions let callers catch your code's errors specifically without accidentally catching unrelated exceptions:

```python
class UppercaseException(Exception):
    pass

words = ['one', 'two', 'three', 'FOUR']

for word in words:
    if word.isupper():
        raise UppercaseException(word)

# Traceback (most recent call last):
# __main__.UppercaseException: FOUR
```

For larger projects, build an exception hierarchy with a shared base class. Callers can then catch the base class to handle all your errors, or catch specific subclasses for finer control:

```python
class AppError(Exception):
    """Base class for all application exceptions."""

class ConfigError(AppError):
    """A configuration value is missing or invalid."""

class NetworkError(AppError):
    """A network operation failed."""

# Catch all application errors:
try:
    load_remote_config(url)
except AppError as exc:
    print(f"Application error: {exc}")

# Or catch specific errors:
try:
    load_remote_config(url)
except ConfigError as exc:
    print(f"Bad config: {exc}")
except NetworkError as exc:
    print(f"Network unavailable: {exc}")
```
