---
title: "Testing"
weight: 120
---

Writing code that works is the first goal. Knowing it still works after you change something is the second. *Automated tests* give you that confidence without manually running the program after every edit.

A test is a function that calls your code, checks the result, and either passes or fails. When a test fails, you know exactly what broke and where. When all tests pass, you can ship with confidence.

This guide builds a small invoicing module as the subject under test. Every section adds new testing techniques to the same module so you can see how a real test suite grows.

## Install pytest

*pytest* is the standard testing framework for Python. Install it in your virtual environment:

```bash
pip install pytest
```

Verify the installation:

```bash
pytest --version
```

## The module under test

Save the following module as `invoice.py`. It calculates order totals, applies discounts, and validates items. All examples in this guide test these three functions.

```python
from dataclasses import dataclass


@dataclass
class Item:
    name: str
    price: float
    quantity: int


def calculate_total(items: list[Item]) -> float:
    """Return the sum of price * quantity for all items."""
    return round(sum(item.price * item.quantity for item in items), 2)


def apply_discount(total: float, discount_pct: float) -> float:
    """Return total after applying a percentage discount.

    Raises:
        ValueError: If discount_pct is not between 0 and 100.
    """
    if not 0 <= discount_pct <= 100:
        raise ValueError(f"Discount must be between 0 and 100, got {discount_pct}")
    return round(total * (1 - discount_pct / 100), 2)


def validate_item(item: Item) -> None:
    """Raise ValueError if the item has invalid fields."""
    if item.price < 0:
        raise ValueError(f"Price cannot be negative: {item.price}")
    if item.quantity < 1:
        raise ValueError(f"Quantity must be at least 1: {item.quantity}")
```

## Write your first test

Create a file named `test_invoice.py` in the same directory as `invoice.py`. pytest discovers test files automatically by looking for names that start with `test_`.

Add the following test:

```python
from invoice import Item, calculate_total


def test_calculate_total_single_item():
    items = [Item(name="Notebook", price=2.50, quantity=4)]
    assert calculate_total(items) == 10.00
```

Run the suite from the command line:

```bash
pytest test_invoice.py
```

pytest prints a dot for each passing test and an `F` for each failure. A failure includes the full assertion output so you can see the expected and actual values side by side.

### Naming conventions

pytest discovers tests by looking for functions whose names start with `test_`. Follow this naming pattern consistently:

- `test_<function>_<scenario>`: names like `test_calculate_total_empty_list` or `test_apply_discount_full_discount`
- One assertion per test when possible — failures are easier to diagnose when each test checks one thing
- Names that read like sentences: `test_validate_item_rejects_negative_price`

### The assert statement

pytest intercepts `assert` statements and shows a detailed diff when they fail. You do not need a special assertion library. The following tests show `assert` in practice:

```python
def test_calculate_total_multiple_items():
    items = [
        Item(name="Pen",      price=1.00, quantity=3),
        Item(name="Notebook", price=2.50, quantity=2),
    ]
    assert calculate_total(items) == 8.00


def test_calculate_total_empty_list():
    assert calculate_total([]) == 0.00
```

## Fixtures

A *fixture* is a function that provides data or resources to a test. Define fixtures with the `@pytest.fixture` decorator and pass them as parameters to test functions. pytest injects the fixture's return value automatically.

Fixtures solve a common problem: when many tests need the same setup, repeating it in every function adds noise and maintenance burden. Define it once and reuse it everywhere.

The following fixture creates a standard set of invoice items:

```python
import pytest
from invoice import Item, calculate_total, apply_discount


@pytest.fixture
def sample_items() -> list[Item]:
    return [
        Item(name="Pen",      price=1.00, quantity=3),
        Item(name="Notebook", price=2.50, quantity=2),
        Item(name="Stapler",  price=8.99, quantity=1),
    ]


def test_calculate_total_with_fixture(sample_items):
    assert calculate_total(sample_items) == 16.99


def test_total_before_discount(sample_items):
    total = calculate_total(sample_items)
    assert apply_discount(total, 10) == 15.29
```

pytest creates a fresh `sample_items` list for each test. Tests do not share state.

### Fixtures with teardown

When a fixture creates a resource that needs cleanup, split setup and teardown around a `yield` statement. Code before `yield` runs before the test. Code after `yield` runs after the test, even if the test fails.

The following fixture writes a temporary CSV file and returns its path:

```python
import csv
import pytest
from pathlib import Path


@pytest.fixture
def invoice_csv(tmp_path: Path) -> Path:
    path = tmp_path / "invoice.csv"
    with path.open("w", newline="") as fh:
        writer = csv.writer(fh)
        writer.writerow(["name", "price", "quantity"])
        writer.writerow(["Pen", 1.00, 3])
        writer.writerow(["Notebook", 2.50, 2])
    yield path
    # Any cleanup code goes here, after the yield.
    # In this case tmp_path handles deletion automatically.


def test_invoice_csv_exists(invoice_csv: Path):
    assert invoice_csv.exists()
    assert invoice_csv.stat().st_size > 0
```

`tmp_path` is a built-in pytest fixture that provides a temporary directory unique to each test. pytest deletes it after the test completes.

## Parameterized tests

*Parameterized tests* run the same test function with multiple inputs. Instead of writing a separate function for each case, define the inputs in a list and let pytest run them all.

Apply `@pytest.mark.parametrize` to test `apply_discount` across several discount rates:

```python
import pytest
from invoice import apply_discount


@pytest.mark.parametrize("total, discount_pct, expected", [
    (100.00,  0,  100.00),
    (100.00, 10,   90.00),
    (100.00, 50,   50.00),
    (100.00, 100,   0.00),
    ( 49.99, 20,   39.99),
])
def test_apply_discount(total, discount_pct, expected):
    assert apply_discount(total, discount_pct) == expected
```

pytest runs five separate tests, one per row. When a test fails, pytest identifies it by its parameter values so you know exactly which case broke.

## Testing exceptions

Some tests verify that a function raises the right exception when given invalid input. Apply `pytest.raises` as a context manager to assert that a specific exception is raised:

```python
import pytest
from invoice import apply_discount, validate_item, Item


def test_apply_discount_rejects_negative_discount():
    with pytest.raises(ValueError):
        apply_discount(100.00, -5)


def test_apply_discount_rejects_over_100():
    with pytest.raises(ValueError):
        apply_discount(100.00, 110)


def test_validate_item_rejects_negative_price():
    item = Item(name="Pen", price=-1.00, quantity=1)
    with pytest.raises(ValueError, match="negative"):
        validate_item(item)
```

The `match` parameter checks that the exception message contains the given string. This is useful when a function raises `ValueError` for several different reasons and you want to confirm the correct one fired.

## Mocking dependencies

A *mock* replaces a real dependency with a controlled substitute during a test. Mocks are valuable when your code calls something slow, unreliable, or external — a database, an HTTP API, or the system clock.

Python's standard library provides `unittest.mock`. The most common tool is `patch`, which temporarily replaces an object for the duration of a single test.

Suppose `invoice.py` gains a function that fetches tax rates from an external API:

```python
import json
import urllib.request


def get_tax_rate(region: str) -> float:
    """Fetch the current tax rate for a region from the tax API."""
    url = f"https://api.example.com/tax/{region}"
    with urllib.request.urlopen(url) as resp:
        return json.loads(resp.read())["rate"]
```

Patch `urlopen` to return a controlled response instead of making a real network call:

```python
import json
from unittest.mock import patch, MagicMock
from invoice import get_tax_rate


def test_get_tax_rate_returns_float():
    mock_resp = MagicMock()
    mock_resp.__enter__.return_value = mock_resp
    mock_resp.__exit__.return_value = False
    mock_resp.read.return_value = json.dumps({"rate": 0.08}).encode()

    with patch("invoice.urllib.request.urlopen", return_value=mock_resp):
        rate = get_tax_rate("us-ca")

    assert rate == 0.08
```

### When to mock

Mocks make tests fast and isolated, but a mock that does not accurately reflect the real dependency can hide bugs. Apply these guidelines:

- Mock external systems: HTTP APIs, databases, email services
- Mock the system clock when your code calls `datetime.now()`
- Do not mock your own internal modules — test them directly
- If a function is hard to test without mocking, consider splitting it into a pure logic layer and a thin I/O layer so you can test the logic directly

## Organize your test suite

As your project grows, a single test file becomes hard to navigate. Mirror your source layout with a parallel test layout. A typical structure looks like this:

```
my-project/
├── src/
│   └── invoice/
│       ├── __init__.py
│       ├── core.py
│       └── tax.py
├── tests/
│   ├── conftest.py
│   ├── test_core.py
│   └── test_tax.py
└── pyproject.toml
```

### conftest.py

`conftest.py` is a special file that pytest loads automatically before running tests. Define shared fixtures here so every test file in the directory can access them without any import.

Move `sample_items` to `tests/conftest.py`:

```python
# tests/conftest.py
import pytest
from invoice.core import Item


@pytest.fixture
def sample_items() -> list[Item]:
    return [
        Item(name="Pen",      price=1.00, quantity=3),
        Item(name="Notebook", price=2.50, quantity=2),
    ]
```

Every test file under `tests/` can now receive `sample_items` as a parameter without importing it.

### Running subsets

Run only the tests whose names match a keyword with `-k`:

```bash
pytest -k "discount"     # runs all tests with "discount" in the name
pytest -k "not slow"     # skips any test with "slow" in the name
```

Mark tests with custom labels and filter by label with `-m`:

```python
@pytest.mark.slow
def test_large_invoice():
    ...
```

Run the suite filtering by mark:

```bash
pytest -m slow        # runs only marked-slow tests
pytest -m "not slow"  # skips marked-slow tests
```

Register custom marks in `pyproject.toml` to avoid warnings:

```toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow (deselect with -m 'not slow')",
]
```

## Measure coverage

*Test coverage* measures which lines of your source code the test suite executes. Lines that no test reaches could contain undetected bugs.

Install the coverage plugin:

```bash
pip install pytest-cov
```

Run the suite with coverage enabled:

```bash
pytest --cov=invoice --cov-report=term-missing
```

The `term-missing` option prints the line numbers that no test touched. The output looks like this:

```
Name         Stmts   Miss  Cover   Missing
------------------------------------------
invoice.py      24      3    88%   41-43
```

Lines 41 through 43 are not covered. Open the file, read those lines, and write a test that exercises them.

### What coverage does not tell you

100% coverage does not mean your code is correct. It means every line ran at least once during the test suite. A line can run and still produce the wrong result. Treat coverage as a minimum baseline: aim for 90% or higher, and focus energy on testing logic branches and edge cases rather than chasing the last few percentage points.
