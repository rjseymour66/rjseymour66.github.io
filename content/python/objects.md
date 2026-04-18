---
title: "Objects"
weight: 80
---

An object is a data structure that combines data (stored as *attributes*) and behavior (defined as *methods*). In Python, everything is an object — numbers, strings, functions, and modules all follow the same object model.

## Simple objects

Define a class with the `class` keyword. Instantiate it by calling the class name as a function:

```python
class Dog():
    pass

fido  = Dog()
pluto = Dog()
# Each instantiation creates a distinct object at a different memory address.
```

## Attributes

### Adding attributes with __init__

`__init__()` initializes a new object's attributes. Python calls it automatically after creating the object. Its first parameter must be `self` — a reference to the object being initialized:

```python
class Teacher:
    def __init__(self):
        pass
```

In the following example, `__init__()` stores the `name` argument as an attribute on the object:

```python
class Teacher:
    def __init__(self, name: str) -> None:
        self.name = name

mr_smith = Teacher('Smith')
mr_smith.name   # 'Smith'
```

When Python evaluates `Teacher('Smith')`, it:

1. Looks up the `Teacher` class definition
2. Creates a new object in memory
3. Calls `__init__()` with the new object as `self` and `'Smith'` as `name`
4. Stores `'Smith'` as `self.name` on the object
5. Returns the new object
6. Binds the name `mr_smith` to it

### Initializers vs constructors

`__init__()` is an *initializer*, not a constructor. Python creates the object before calling `__init__()` — the initializer populates it with the attributes that distinguish one instance from another. Not every class needs an `__init__()`.

## Inheritance

*Inheritance* creates a new class from an existing one. The *child class* inherits all methods and attributes from the *parent class*, and can add or override them. Pass the parent class name in parentheses when defining a child class:

```python
class Guitar():
    def strum(self):
        print('raaaaannnngggg')

class Fender(Guitar):
    pass

issubclass(Fender, Guitar)   # True

blackie = Fender()
blackie.strum()   # 'raaaaannnngggg' — inherited from Guitar

# Override the strum method in the child class:
class Fender(Guitar):
    def strum(self):
        print('brrroooooonnnnngggggg')

blackie = Fender()
blackie.strum()   # 'brrroooooonnnnngggggg'
```

Override `__init__()` in a child class to add new attributes:

```python
class Person():
    def __init__(self, name: str) -> None:
        self.name = name

class MDPerson(Person):
    def __init__(self, name: str) -> None:
        self.name = "Doctor " + name

class JDPerson(Person):
    def __init__(self, name: str) -> None:
        self.name = name + ", Esquire"

person = Person('Jack')     # name = 'Jack'
doctor = MDPerson('Jack')   # name = 'Doctor Jack'
lawyer = JDPerson('Jack')   # name = 'Jack, Esquire'
```

### super()

`super()` calls the parent class's method from inside the child class. Apply it in `__init__()` to initialize the parent's attributes before adding new ones:

```python
class Person():
    def __init__(self, name: str) -> None:
        self.name = name

class EmailPerson(Person):
    def __init__(self, name: str, email: str) -> None:
        super().__init__(name)   # initialize the parent's name attribute
        self.email = email       # add the attribute unique to this child class
```

Always apply `super()` rather than calling the parent class by name — it correctly handles inheritance chains and remains robust when the class hierarchy changes.

## Getters and setters

Python does not enforce private attributes at the language level. The convention for a "hidden" attribute is a double-underscore prefix (`__attribute`). Expose it through `@property` and `@<name>.setter` decorators instead of accessing it directly.

Apply `@property` for the getter and `@<name>.setter` for the setter. If you define no setter, the attribute is effectively read-only from outside the class:

```python
class Person():
    def __init__(self, name: str) -> None:
        self.__name = name   # double underscore — name-mangled by Python

    @property
    def name(self) -> str:
        print('getter method')
        return self.__name

    @name.setter
    def name(self, name: str) -> None:
        print('setter method')
        self.__name = name

musician = Person('Paul')
musician.name           # getter method → 'Paul'
musician.name = 'John'  # setter method
musician.name           # getter method → 'John'
```

Python stores the hidden attribute under a mangled name to prevent accidental collisions in subclasses. For example, `__name` on a `Person` object is stored as `_Person__name`:

```python
musician._Person__name
# 'John'
```

## Class and object attributes

Attributes assigned directly on the class (outside any method) are *class attributes* — shared across all instances. Attributes assigned on `self` inside a method are *instance attributes* — unique to each instance. Assigning to an instance attribute shadows the class attribute for that object without changing the class:

```python
class Boots:
    color = 'brown'   # class attribute

red_wing = Boots()
red_wing.color          # 'brown' — reads the class attribute
red_wing.color = 'tan'  # creates an instance attribute on red_wing only
red_wing.color          # 'tan'
Boots.color             # 'brown' — class attribute unchanged
```

## Method types

Python supports three method types. The following table describes each:

| Method type | First parameter | Decorator | Affects |
|---|---|---|---|
| Instance | `self` (the object) | None | The individual instance |
| Class | `cls` (the class) | `@classmethod` | The class and all instances |
| Static | None | `@staticmethod` | Neither — it is a utility function |

The following example applies a class method to track how many objects have been created from a class:

```python
class ChildCounter():
    count = 0

    def __init__(self) -> None:
        ChildCounter.count += 1

    def exclaim(self) -> None:
        print("I'm in the ChildCounter() class!")

    @classmethod
    def children(cls) -> None:
        print('ChildCounter has', cls.count, 'child objects.')

first  = ChildCounter()
second = ChildCounter()
third  = ChildCounter()
ChildCounter.children()   # ChildCounter has 3 child objects.
```

The following example demonstrates a static method — a utility that belongs to the class namespace but requires no access to the class or any instance:

```python
class Nike():
    @staticmethod
    def slogan() -> str:
        return 'Just Do It'

Nike.slogan()   # 'Just Do It'
```

## Magic methods

*Magic methods* (also called dunder methods) are special methods Python recognizes by their double-underscore names. Define them on your classes to integrate with Python's built-in operators and functions. The full list is in the [Python data model documentation](https://docs.python.org/3/reference/datamodel.html#special-method-names).

The following table covers the most commonly implemented magic methods:

| Method | Triggered by | Purpose |
|---|---|---|
| `__eq__(self, other)` | `self == other` | Test equality |
| `__str__(self)` | `str(self)`, `print()`, f-strings | Human-readable string representation |
| `__repr__(self)` | `repr(self)`, REPL output | Unambiguous developer-facing representation |
| `__len__(self)` | `len(self)` | Return the length |

### __repr__ vs __str__

`__str__` produces the output a user reads — in logs, print statements, and displayed text. `__repr__` produces unambiguous output a developer reads — in the REPL, debuggers, and stack traces. When only one is defined, `__repr__` serves as the fallback.

The following example implements both on a configuration entry class:

```python
from dataclasses import dataclass

@dataclass
class ConfigEntry:
    key: str
    value: str
    source: str
    secret: bool = False

    def __str__(self) -> str:
        # Hide secret values in user-facing output — logs, dashboards, CLI:
        display_value = "***" if self.secret else self.value
        return f"{self.key}={display_value}  (from {self.source})"

    def __repr__(self) -> str:
        # Show full detail for debugging — mask nothing:
        return (
            f"ConfigEntry(key={self.key!r}, value={self.value!r}, "
            f"source={self.source!r}, secret={self.secret!r})"
        )

db_pass = ConfigEntry("DB_PASSWORD", "s3cr3t!", source="env", secret=True)

print(str(db_pass))    # DB_PASSWORD=***  (from env)
print(repr(db_pass))   # ConfigEntry(key='DB_PASSWORD', value='s3cr3t!', ...)
print(f"{db_pass}")    # DB_PASSWORD=***  (from env)   — f-string calls __str__
print(f"{db_pass!r}")  # ConfigEntry(...)              — !r forces __repr__
```

## Aggregation and composition

Build objects from other objects by storing instances as attributes. *Composition* means the parent object creates and owns its parts. *Aggregation* means the parts exist independently and are passed in:

```python
class Sail():
    def __init__(self, description: str) -> None:
        self.description = description

class Tiller():
    def __init__(self, length: str) -> None:
        self.length = length

class Boat():
    def __init__(self, sail: Sail, tiller: Tiller) -> None:
        self.sail   = sail
        self.tiller = tiller

    def about(self) -> None:
        print(f'This boat has a {self.sail.description} sail and a {self.tiller.length} tiller')

sail   = Sail('blue and white')
tiller = Tiller('5 ft')
boat   = Boat(sail, tiller)
boat.about()
# This boat has a blue and white sail and a 5 ft tiller
```

## Named tuples

*Named tuples* are tuples whose fields have names. Access values by `.name` or by position. They are immutable and hashable — like regular tuples — but easier to read. Import `namedtuple` from the `collections` module:

```python
from collections import namedtuple

Boat = namedtuple('Boat', 'sail tiller')
boat = Boat('blue', 'rudder')

boat.sail     # 'blue'
boat.tiller   # 'rudder'
boat[0]       # 'blue' — positional access still works
```

Apply named tuples for lightweight, readable record types that need no methods or validation. Apply dataclasses when you need default values, computed fields, or methods.

## Dataclasses

*Dataclasses* (introduced in Python 3.7) generate `__init__()`, `__repr__()`, and `__eq__()` automatically from annotated class attributes. Apply the `@dataclass` decorator and declare attributes with `name: type` or `name: type = default_value`:

```python
from dataclasses import dataclass

@dataclass
class GuitarClass:
    brand: str
    color: str
    price: int = 0   # 0 is the default value

strat    = GuitarClass('Fender', 'Black', 1000)
les_paul = GuitarClass(price=2500, brand='Gibson', color='Honeyburst')
```

### field(), __post_init__, and frozen dataclasses

`field()` configures per-attribute behavior: factory functions for mutable defaults, and flags that control whether a field appears in `__repr__` or participates in equality comparisons. `__post_init__()` runs after `__init__()` for cross-field validation. Setting `frozen=True` makes the dataclass immutable — every attribute is read-only after creation.

The following example models an immutable API (application programming interface) response:

```python
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any

@dataclass(frozen=True)
class APIResponse:
    """Immutable record of a single API call result.

    frozen=True makes every field read-only after __init__ runs.
    Python enforces this by raising FrozenInstanceError on any assignment.
    frozen=True also generates __hash__ automatically, so instances can be
    stored in sets or used as dictionary keys.
    """
    status_code: int
    url: str
    body: dict[str, Any] = field(default_factory=dict)
    headers: dict[str, str] = field(default_factory=dict, repr=False)
    received_at: datetime = field(default_factory=datetime.utcnow, compare=False)
    _elapsed_ms: float = field(default=0.0, repr=False)

    def __post_init__(self) -> None:
        if not (100 <= self.status_code <= 599):
            raise ValueError(f"Invalid HTTP status code: {self.status_code}")

    @property
    def ok(self) -> bool:
        """Return True for 2xx status codes."""
        return 200 <= self.status_code <= 299

    @property
    def elapsed_seconds(self) -> float:
        return self._elapsed_ms / 1000

resp = APIResponse(
    status_code=200,
    url="https://api.example.com/v1/users",
    _elapsed_ms=143.7,
)

resp.ok               # True
resp.elapsed_seconds  # 0.1437
# resp.status_code = 404  # raises FrozenInstanceError
```

The following table describes the most common `field()` options:

| Option | Purpose | Example |
|---|---|---|
| `default_factory` | Call a function to produce the default value (required for mutable types like `list` and `dict`) | `field(default_factory=list)` |
| `repr=False` | Exclude this field from `__repr__` output | `field(default=0.0, repr=False)` |
| `compare=False` | Exclude this field from `__eq__` and ordering comparisons | `field(default_factory=datetime.utcnow, compare=False)` |
