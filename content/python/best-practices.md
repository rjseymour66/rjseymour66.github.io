---
title: "Best practices"
weight: 110
---

Writing a Python script that works is the first step. Writing one that you can hand to a colleague, run in a cron job six months from now, or debug at 2 a.m. is the goal. These practices close that gap.

## Project layout

### Single-file scripts

A script that does one thing well belongs in a single file. Keep it short enough to read in one sitting, typically under 300 lines. When the file grows beyond that, or when you need to share logic across scripts, convert to a package.

```
my-project/
├── process_logs.py      # the script
├── requirements.txt
└── README.md
```

### Packages

A *package* is a directory with an `__init__.py` file. Convert a single-file script to a package when you need multiple modules, a test suite, or distribution via pip.

```
my-project/
├── src/
│   └── log_processor/
│       ├── __init__.py
│       ├── __main__.py   # entry point: python -m log_processor
│       ├── parser.py
│       └── reporter.py
├── tests/
│   └── test_parser.py
├── pyproject.toml
└── README.md
```

Place source code under `src/` to prevent the package from being importable before installation. This avoids a common source of test/production divergence.

## Script header

### Shebang line

Add a shebang as the first line of any script you intend to run directly. Use `env` so the system finds the correct Python regardless of installation path:

```python
#!/usr/bin/env python3
```

Make the file executable before running it directly:

```bash
chmod +x process_logs.py
./process_logs.py
```

### Encoding declaration

Python 3 defaults to UTF-8. You only need an explicit encoding declaration if your source file uses a different encoding, which is rare. Skip it unless your toolchain requires it.

## The `__name__` guard

Every Python file has a `__name__` attribute. When you run a file directly, Python sets `__name__` to `"__main__"`. When you import the file as a module, Python sets `__name__` to the module name.

Wrap top-level execution code in an `if __name__ == "__main__"` guard. This lets other modules import your functions without triggering side effects:

```python
def process(path: str) -> int:
    """Process a log file and return the number of records parsed."""
    ...

if __name__ == "__main__":
    import sys
    count = process(sys.argv[1])
    print(f"Parsed {count} records")
```

Without the guard, importing this module runs `process()` immediately and breaks any test or script that tries to reuse your functions.

## CLI arguments with argparse

Do not parse `sys.argv` manually. Use `argparse` to define a self-documenting interface that validates inputs, generates `--help` output, and handles errors cleanly.

### Basic argument parsing

```python
import argparse

def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(
        description="Count log entries matching a pattern.",
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument("log_file", type=str, help="Path to the log file")
    parser.add_argument(
        "--pattern",
        type=str,
        default="ERROR",
        help="Pattern to search for (default: ERROR)",
    )
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",
        help="Enable debug logging",
    )
    return parser

if __name__ == "__main__":
    args = build_parser().parse_args()
    # args.log_file  → "app.log"
    # args.pattern   → "ERROR"
    # args.verbose   → True / False
```

Define `build_parser()` as a separate function so tests can call it without executing the script.

### Subcommands

Use subparsers when your script does more than one distinct thing:

```python
def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(description="Log analysis toolkit")
    sub = parser.add_subparsers(dest="command", required=True)

    count_cmd = sub.add_parser("count", help="Count matching entries")
    count_cmd.add_argument("log_file")
    count_cmd.add_argument("--pattern", default="ERROR")

    export_cmd = sub.add_parser("export", help="Export matches to CSV")
    export_cmd.add_argument("log_file")
    export_cmd.add_argument("--output", required=True)

    return parser
```

## Logging instead of print

Replace `print()` calls with the `logging` module. Logging gives you severity levels, timestamps, output routing, and lets you silence a noisy library without touching its source.

### Configure logging once at the entry point

```python
import logging
import sys

def configure_logging(verbose: bool = False) -> None:
    level = logging.DEBUG if verbose else logging.INFO
    logging.basicConfig(
        stream=sys.stderr,
        level=level,
        format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
        datefmt="%Y-%m-%dT%H:%M:%S",
    )
```

Call `configure_logging()` once, inside the `if __name__ == "__main__"` block. Never call it at module level. Doing so forces your configuration on every script that imports your module.

### Use module-level loggers

Each module gets its own logger named after the module. This lets callers filter or route log output by source:

```python
import logging

logger = logging.getLogger(__name__)   # e.g., "log_processor.parser"

def parse_line(line: str) -> dict | None:
    logger.debug("Parsing line: %s", line.rstrip())
    if not line.strip():
        logger.warning("Skipping empty line")
        return None
    ...
```

Use `%s` placeholders instead of f-strings in log calls. The logging module skips string interpolation entirely when the message is below the active log level, which is a meaningful speedup in hot paths.

## File paths with pathlib

Use `pathlib.Path` instead of `os.path` string manipulation. Path objects compose cleanly, expose the file system through methods, and make intent explicit.

```python
from pathlib import Path

log_dir = Path("/var/log/myapp")

# Compose paths with /
today_log = log_dir / "2024-01-15.log"

# Inspect path components
today_log.stem      # "2024-01-15"
today_log.suffix    # ".log"
today_log.parent    # PosixPath('/var/log/myapp')

# Test existence and type
if today_log.exists() and today_log.is_file():
    size = today_log.stat().st_size   # bytes

# Iterate a directory
for path in log_dir.glob("*.log"):
    print(path.name)

# Read and write text
content = today_log.read_text(encoding="utf-8")
output = Path("summary.txt")
output.write_text("Done.\n", encoding="utf-8")
```

Annotate function parameters as `Path` rather than `str`. Convert `str` arguments at the boundary, not deep in your logic:

```python
from pathlib import Path

def count_lines(log_file: Path) -> int:
    return sum(1 for _ in log_file.open(encoding="utf-8"))

if __name__ == "__main__":
    import sys
    result = count_lines(Path(sys.argv[1]))   # convert str → Path once
```

## Context managers

Open every resource inside a `with` block: files, network connections, and database cursors. The context manager closes the resource when the block exits, even if an exception is raised.

```python
# File handles
with open("data.csv", encoding="utf-8") as fh:
    for line in fh:
        process(line)

# Multiple resources in one statement
with open("input.txt") as src, open("output.txt", "w") as dst:
    dst.writelines(src)
```

### Write your own context manager

Use `contextlib.contextmanager` to turn a generator function into a context manager. The code before `yield` runs on entry. The code after `yield` runs on exit:

```python
import contextlib
import logging
import time
from collections.abc import Generator

logger = logging.getLogger(__name__)

@contextlib.contextmanager
def timed_operation(name: str) -> Generator[None, None, None]:
    """Log the wall-clock time of a block."""
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        logger.info("%s completed in %.3fs", name, elapsed)

with timed_operation("index build"):
    build_index(records)
# 2024-01-15T10:30:01 INFO     __main__: index build completed in 1.247s
```

## Type hints

Annotate every function signature. Type hints do not affect runtime behavior, but they let mypy and your editor catch type errors before the code runs.

### Basic annotations

```python
from pathlib import Path
from collections.abc import Iterator

def read_log(path: Path, pattern: str = "ERROR") -> Iterator[str]:
    """Yield lines from path that contain pattern."""
    with path.open(encoding="utf-8") as fh:
        for line in fh:
            if pattern in line:
                yield line.rstrip()
```

### TypedDict for structured dicts

When a dict has a fixed, known set of keys, use `TypedDict` instead of `dict[str, Any]`:

```python
from typing import TypedDict

class LogEntry(TypedDict):
    timestamp: str
    level: str
    message: str

def parse_entry(line: str) -> LogEntry | None:
    parts = line.split(maxsplit=2)
    if len(parts) < 3:
        return None
    return LogEntry(timestamp=parts[0], level=parts[1], message=parts[2])
```

### Dataclasses

Use `dataclasses.dataclass` for structured data that needs methods. It generates `__init__`, `__repr__`, and `__eq__` automatically:

```python
from dataclasses import dataclass, field
from pathlib import Path

@dataclass
class ScanResult:
    path: Path
    total_lines: int
    matches: list[str] = field(default_factory=list)

    @property
    def match_count(self) -> int:
        return len(self.matches)

    def summary(self) -> str:
        return f"{self.path.name}: {self.match_count}/{self.total_lines} lines matched"
```

## Example

The following script applies every practice above. It scans a log file for a pattern and writes matching lines to a report file.

```python
#!/usr/bin/env python3
"""Scan a log file for a pattern and write a report.

Usage:
    python scan_log.py app.log --pattern ERROR --output report.txt
"""

import argparse
import logging
import sys
from dataclasses import dataclass, field
from pathlib import Path

logger = logging.getLogger(__name__)


# ---------------------------------------------------------------------------
# Data structures
# ---------------------------------------------------------------------------

@dataclass
class ScanResult:
    source: Path
    pattern: str
    matches: list[str] = field(default_factory=list)
    total_lines: int = 0

    @property
    def match_count(self) -> int:
        return len(self.matches)

    def summary(self) -> str:
        pct = (self.match_count / self.total_lines * 100) if self.total_lines else 0
        return (
            f"Scanned {self.source.name}: "
            f"{self.match_count}/{self.total_lines} lines matched "
            f"'{self.pattern}' ({pct:.1f}%)"
        )


# ---------------------------------------------------------------------------
# Core logic
# ---------------------------------------------------------------------------

def scan(source: Path, pattern: str) -> ScanResult:
    """Scan source for lines containing pattern.

    Args:
        source: Path to the log file.
        pattern: Substring to match.

    Returns:
        A ScanResult with all matching lines.

    Raises:
        FileNotFoundError: If source does not exist.
    """
    result = ScanResult(source=source, pattern=pattern)
    logger.info("Scanning %s for %r", source, pattern)

    with source.open(encoding="utf-8") as fh:
        for line in fh:
            result.total_lines += 1
            if pattern in line:
                result.matches.append(line.rstrip())
                logger.debug("Match on line %d: %s", result.total_lines, line.rstrip())

    logger.info(result.summary())
    return result


def write_report(result: ScanResult, output: Path) -> None:
    """Write matching lines to output.

    Args:
        result: The completed scan result.
        output: Destination file path. Created or overwritten.
    """
    output.parent.mkdir(parents=True, exist_ok=True)
    with output.open("w", encoding="utf-8") as fh:
        fh.write(f"# Scan report: {result.source.name}\n")
        fh.write(f"# Pattern: {result.pattern}\n")
        fh.write(f"# {result.summary()}\n\n")
        for line in result.matches:
            fh.write(line + "\n")
    logger.info("Report written to %s", output)


# ---------------------------------------------------------------------------
# CLI
# ---------------------------------------------------------------------------

def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(
        description=__doc__,
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument("log_file", type=Path, help="Log file to scan")
    parser.add_argument("--pattern", default="ERROR", help="Pattern to match (default: ERROR)")
    parser.add_argument("--output", type=Path, default=None, help="Write report to this file")
    parser.add_argument("--verbose", "-v", action="store_true", help="Enable debug logging")
    return parser


def configure_logging(verbose: bool) -> None:
    logging.basicConfig(
        stream=sys.stderr,
        level=logging.DEBUG if verbose else logging.INFO,
        format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
        datefmt="%Y-%m-%dT%H:%M:%S",
    )


def main(argv: list[str] | None = None) -> int:
    """Entry point. Returns an exit code."""
    args = build_parser().parse_args(argv)
    configure_logging(args.verbose)

    if not args.log_file.is_file():
        logger.error("File not found: %s", args.log_file)
        return 1

    result = scan(args.log_file, args.pattern)

    if args.output:
        write_report(result, args.output)

    print(result.summary())
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

Notice the structure:

- `scan()` and `write_report()` are pure functions, testable without running the CLI.
- `main()` accepts an optional `argv` parameter so tests can call `main(["app.log", "--pattern", "WARN"])` without spawning a subprocess.
- `main()` returns an integer exit code and passes it to `sys.exit()`, the only place the process exits.
- Logging is configured once, inside `main()`, before any other code runs.
