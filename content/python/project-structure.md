---
title: "Project structure"
weight: 20
---

A Python project that starts as a single script will eventually need a test suite, configuration, and shared utilities. Adding those without a plan creates a tangle that slows every change. Starting with the right structure means later growth follows a clear path.

This guide walks through a reference project called `pricemon`, a command-line tool that reads a product catalog from a JSON file and reports items below a target price. The complete directory tree and all source files appear at the end.

## When a single file is enough

Not every Python program needs a package. A script that does one clear task belongs in a single file. Keep it short enough to read in one sitting — under 300 lines is a useful guideline. When the file grows beyond that, or when you need to share logic across scripts or write a test suite, convert it to a package.

For simple scripts, the following layout is sufficient:

```
pricemon/
├── pricemon.py
├── requirements.txt
└── README.md
```

## The src layout

For a package that includes tests, a command-line entry point, or distribution via pip, the *src layout* is the recommended standard. Place all source code under a `src/` directory:

```
pricemon/
├── src/
│   └── pricemon/
│       ├── __init__.py
│       ├── __main__.py
│       ├── catalog.py
│       └── report.py
├── tests/
│   ├── conftest.py
│   ├── test_catalog.py
│   └── test_report.py
├── pyproject.toml
├── .env.example
└── README.md
```

The `src/` wrapper prevents the package from being importable before installation. Without it, Python's import system can find the local source directory and silently shadow an installed version, creating subtle differences between development and production.

### What each file does

The following files are standard in every package:

- `__init__.py`: Marks the directory as a Python package. Keep it minimal — define the public API or leave it empty
- `__main__.py`: Makes the package runnable with `python -m pricemon`. Put your `main()` call here
- Module files (`catalog.py`, `report.py`): The actual logic, split by responsibility
- `tests/conftest.py`: Shared pytest fixtures, loaded automatically
- `pyproject.toml`: Project metadata, dependencies, and tool configuration

## pyproject.toml

`pyproject.toml` is the single configuration file for modern Python projects. It replaces `setup.py`, `setup.cfg`, and scattered `.ini` files. The following is a complete example for `pricemon`:

```toml
[build-system]
requires      = ["setuptools>=68", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name            = "pricemon"
version         = "0.1.0"
description     = "Monitor product prices from a catalog file"
requires-python = ">=3.11"
dependencies    = [
    "rich>=13.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "mypy>=1.9",
    "ruff>=0.4",
]

[project.scripts]
pricemon = "pricemon.__main__:main"

[tool.setuptools.packages.find]
where = ["src"]

[tool.pytest.ini_options]
testpaths = ["tests"]
markers   = [
    "slow: marks tests as slow (deselect with -m 'not slow')",
]

[tool.mypy]
strict         = true
python_version = "3.11"

[tool.ruff.lint]
select = ["E", "F", "I"]
```

The key sections serve distinct purposes:

- `[project]`: The package name, version, Python version requirement, and runtime dependencies. The `requires-python` field prevents accidental installation on an incompatible interpreter
- `[project.optional-dependencies]`: Development tools installed with `pip install -e ".[dev]"` — kept separate from runtime dependencies
- `[project.scripts]`: Registers the `pricemon` shell command. After installation, running `pricemon` in the terminal calls `pricemon.__main__:main`
- `[tool.setuptools.packages.find]`: Tells setuptools to look for packages inside `src/` rather than the project root

## Manage dependencies

Python's built-in tools are sufficient for most projects. The following workflow covers the full lifecycle.

### Create and activate a virtual environment

Every project gets its own isolated environment. Run the following from the project root:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Install in editable mode

Install the project and its dependencies in one step:

```bash
pip install -e ".[dev]"
```

The `-e` flag installs in *editable mode*: changes to source files take effect immediately without reinstalling. The `[dev]` extra installs the development tools defined in `pyproject.toml`.

### Pin dependencies for production

For applications deployed to a server or container, pin exact versions so every deployment is identical. Generate a locked requirements file:

```bash
pip freeze > requirements.lock
```

Install from the lock file in CI or production:

```bash
pip install -r requirements.lock
```

Libraries published to PyPI should not pin their dependencies. Only pin in applications and services.

## Configuration

Applications need configuration: file paths, API keys, and environment-specific settings. Store configuration in environment variables, not in source files. Source files belong in version control; secrets do not.

### Read from environment variables

Read configuration at startup with `os.environ`:

```python
import os
from pathlib import Path


def load_config() -> dict[str, str | Path]:
    catalog_path = os.environ.get("PRICEMON_CATALOG", "catalog.json")
    return {
        "catalog_path": Path(catalog_path),
        "currency":     os.environ.get("PRICEMON_CURRENCY", "USD"),
    }
```

### .env files for local development

In development, load environment variables from a `.env` file instead of setting them in the shell. Install `python-dotenv`:

```bash
pip install python-dotenv
```

Load it at the entry point before anything else runs:

```python
from dotenv import load_dotenv

load_dotenv()
```

Create a `.env.example` file in the project root and commit it to version control. List every required variable with placeholder values so new contributors know what to configure:

```
PRICEMON_CATALOG=catalog.json
PRICEMON_CURRENCY=USD
```

Add `.env` itself to `.gitignore`. The real values stay on the developer's machine.

## Full example

The following shows all source files for the complete `pricemon` project.

### src/pricemon/\_\_init\_\_.py

The `__init__.py` file declares the package's public API:

```python
from pricemon.catalog import load_catalog, Item
from pricemon.report  import find_below_price

__all__ = ["load_catalog", "Item", "find_below_price"]
```

### src/pricemon/catalog.py

`catalog.py` owns all data loading and validation:

```python
import json
from dataclasses import dataclass
from pathlib import Path


@dataclass
class Item:
    name:     str
    price:    float
    category: str


def load_catalog(path: Path) -> list[Item]:
    """Load items from a JSON catalog file.

    Args:
        path: Path to the catalog JSON file.

    Returns:
        A list of Item objects.

    Raises:
        FileNotFoundError: If path does not exist.
        ValueError: If the file contains invalid JSON or missing fields.
    """
    if not path.exists():
        raise FileNotFoundError(f"Catalog not found: {path}")

    try:
        raw = json.loads(path.read_text(encoding="utf-8"))
    except json.JSONDecodeError as exc:
        raise ValueError(f"Invalid JSON in {path}: {exc}") from exc

    return [
        Item(name=entry["name"], price=entry["price"], category=entry["category"])
        for entry in raw
    ]
```

### src/pricemon/report.py

`report.py` contains pure filtering logic with no I/O:

```python
from pricemon.catalog import Item


def find_below_price(items: list[Item], threshold: float) -> list[Item]:
    """Return items whose price is strictly below threshold, sorted by price.

    Args:
        items:     The catalog to search.
        threshold: The maximum price, exclusive.

    Returns:
        A filtered list sorted by price ascending.
    """
    return sorted(
        [item for item in items if item.price < threshold],
        key=lambda item: item.price,
    )
```

### src/pricemon/\_\_main\_\_.py

`__main__.py` is the entry point. It wires the CLI together but contains no business logic:

```python
import argparse
import sys
from pathlib import Path

from pricemon.catalog import load_catalog
from pricemon.report  import find_below_price


def build_parser() -> argparse.ArgumentParser:
    parser = argparse.ArgumentParser(
        description="Find catalog items below a price threshold."
    )
    parser.add_argument("catalog", type=Path, help="Path to the JSON catalog file")
    parser.add_argument("--below", type=float, required=True, help="Price threshold")
    return parser


def main(argv: list[str] | None = None) -> int:
    args = build_parser().parse_args(argv)
    items = load_catalog(args.catalog)
    results = find_below_price(items, args.below)

    if not results:
        print(f"No items below {args.below}")
        return 0

    for item in results:
        print(f"{item.name:<30} {item.price:>8.2f}  {item.category}")

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

Notice the structure:

- `load_catalog` and `find_below_price` are pure functions, testable without running the CLI
- `main()` accepts an optional `argv` parameter so tests can call `main(["catalog.json", "--below", "50"])` without spawning a subprocess
- `main()` returns an integer exit code passed to `sys.exit()`, the only place the process exits

### tests/conftest.py

`conftest.py` defines shared fixtures for all tests:

```python
import json
import pytest
from pathlib import Path
from pricemon.catalog import Item


@pytest.fixture
def sample_items() -> list[Item]:
    return [
        Item(name="Espresso machine", price=149.99, category="Kitchen"),
        Item(name="Notebook",         price=  2.50, category="Office"),
        Item(name="Desk lamp",        price= 34.95, category="Office"),
    ]


@pytest.fixture
def catalog_file(tmp_path: Path) -> Path:
    data = [
        {"name": "Espresso machine", "price": 149.99, "category": "Kitchen"},
        {"name": "Notebook",         "price":   2.50, "category": "Office"},
    ]
    path = tmp_path / "catalog.json"
    path.write_text(json.dumps(data), encoding="utf-8")
    return path
```

### tests/test\_catalog.py

Tests for loading and validation:

```python
import pytest
from pathlib import Path
from pricemon.catalog import load_catalog


def test_load_catalog_returns_items(catalog_file: Path):
    items = load_catalog(catalog_file)
    assert len(items) == 2
    assert items[0].name == "Espresso machine"


def test_load_catalog_raises_for_missing_file(tmp_path: Path):
    with pytest.raises(FileNotFoundError):
        load_catalog(tmp_path / "missing.json")


def test_load_catalog_raises_for_invalid_json(tmp_path: Path):
    bad = tmp_path / "bad.json"
    bad.write_text("not json", encoding="utf-8")
    with pytest.raises(ValueError, match="Invalid JSON"):
        load_catalog(bad)
```

### tests/test\_report.py

Tests for the filtering logic:

```python
from pricemon.catalog import Item
from pricemon.report  import find_below_price


def test_find_below_price_filters_correctly(sample_items: list[Item]):
    results = find_below_price(sample_items, 50.00)
    names = [item.name for item in results]
    assert "Notebook" in names
    assert "Desk lamp" in names
    assert "Espresso machine" not in names


def test_find_below_price_returns_sorted(sample_items: list[Item]):
    results = find_below_price(sample_items, 200.00)
    prices = [item.price for item in results]
    assert prices == sorted(prices)


def test_find_below_price_returns_empty_when_none_match(sample_items: list[Item]):
    results = find_below_price(sample_items, 1.00)
    assert results == []
```
