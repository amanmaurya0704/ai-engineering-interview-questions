# Python Interview Concepts

This guide covers fundamental Python topics that frequently appear in interviews. Each section gives a short explanation and points out what interviewers commonly test.

## 1. Mutable vs. Immutable Objects

**Mutable** objects can be changed after creation, such as `list`, `dict`, and `set`. **Immutable** objects cannot be changed in place, such as `int`, `float`, `str`, `tuple`, `frozenset`, and `bool`.

Changing a mutable object through one reference is visible through every reference to that same object. With an immutable object, an apparent “change” creates a new object instead.

## 2. Shallow Copy vs. Deep Copy

A **shallow copy** creates a new outer container but keeps references to the same nested objects. A **deep copy** recursively creates copies of nested objects as well.

Use `copy.copy(value)` for a shallow copy and `copy.deepcopy(value)` for a deep copy. This matters most with nested lists and dictionaries.

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
deep = copy.deepcopy(original)

original[0].append(99)
# shallow[0] also changes; deep[0] does not
```

## 3. Hashable Objects

An object is **hashable** if it has a hash value that does not change during its lifetime and can be compared for equality. Hashable objects can be dictionary keys and set elements.

Examples: `str`, `int`, `float`, and immutable tuples containing only hashable values. Lists, dictionaries, and sets are not hashable because they can change.

```python
scores = {"Aman": 95}     # valid: string key
locations = {(10, 20)}    # valid: tuple is hashable here
# locations = {[10, 20]}  # TypeError: list is unhashable
```

## 4. `==` vs. `is`

`==` checks whether two values are equal. `is` checks whether two names refer to the exact same object in memory.

Use `is` mainly for singleton checks such as `value is None`, not for ordinary string or number comparison.

## 5. List, Tuple, Set, and Dictionary

- A **list** is ordered, mutable, and allows duplicates.
- A **tuple** is ordered, immutable, and allows duplicates.
- A **set** contains unique hashable values and provides fast membership checks; it has no index-based access.
- A **dictionary** maps unique hashable keys to values and preserves insertion order in modern Python.

Choose the collection based on whether you need ordering, mutation, uniqueness, or key-based lookup.

## 6. Dictionary Internals

Python dictionaries are implemented using hash tables. A key’s hash helps Python find the likely storage location quickly, giving average-case `O(1)` lookup, insertion, and deletion.

Two different keys can have the same hash; this is called a **collision**. Python resolves collisions while still checking actual key equality.

## 7. Scope and the LEGB Rule

When resolving a name, Python searches scopes in this order:

1. **Local** — inside the current function.
2. **Enclosing** — inside an outer function, for nested functions.
3. **Global** — at module level.
4. **Built-in** — names such as `len` and `print`.

Use `global` to rebind a module-level name and `nonlocal` to rebind a variable from an enclosing function scope.

## 8. Mutable Default Arguments

Default parameter values are evaluated once, when the function is defined—not every time it is called. Therefore, a mutable default like `[]` can accidentally be shared across calls.

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

This `None` pattern creates a fresh list for each call that does not supply one.

## 9. Closures and Late Binding

A **closure** is a function that remembers values from its enclosing scope. **Late binding** means a closure typically looks up captured variables when it runs, not when it was created.

This often surprises people when creating lambdas in a loop. Capture the current loop value with a default parameter if needed: `lambda x=i: x`.

## 10. Iterables vs. Iterators

An **iterable** is anything you can loop over, such as a list, string, or dictionary. An **iterator** produces values one at a time and keeps track of its position.

`iter(value)` obtains an iterator and `next(iterator)` retrieves its next item. Most iterators are exhausted after one complete pass.

## 11. Generators

A **generator** is a special iterator that produces values lazily, usually using `yield` or a generator expression.

Generators are memory-efficient for large or potentially infinite sequences because they do not build every result in memory at once.

## 12. Decorators

A **decorator** takes a function or class and returns a wrapped or modified version. Decorators are commonly used for logging, authorization, timing, caching, and validation.

Use `functools.wraps` inside function decorators to preserve the original function’s name and documentation.

## 13. Comprehensions

Comprehensions provide a compact way to create collections.

```python
squares = [n * n for n in range(5)]
unique_lengths = {len(word) for word in ["cat", "dog", "python"]}
word_lengths = {word: len(word) for word in ["cat", "python"]}
```

Python supports list, set, dictionary, and generator comprehensions. Prefer them when they remain readable.

## 14. `*args` and `**kwargs`

`*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary.

They can also unpack values in a call: `func(*items, **options)`.

## 15. Instance, Class, and Static Methods

- An **instance method** receives `self` and works with a particular object.
- A **class method** receives `cls` and commonly implements alternate constructors or class-wide behavior.
- A **static method** receives neither automatically and is a utility function logically grouped with the class.

## 16. Inheritance, `super()`, and MRO

Inheritance lets a class reuse or extend another class. `super()` calls an implementation according to Python’s **method resolution order (MRO)**.

MRO is especially important in multiple inheritance because it determines which parent implementation is selected and in what order cooperative `super()` calls proceed.

## 17. Dunder (Magic) Methods

Double-underscore methods customize how objects behave with Python syntax and built-ins.

Common examples are `__init__` (initialization), `__repr__` (developer representation), `__str__` (user-friendly text), `__eq__` (equality), `__hash__` (hashing), and `__len__` (length).

## 18. Dataclasses

`@dataclass` reduces boilerplate for classes mainly used to hold data. It can automatically create methods such as `__init__`, `__repr__`, and `__eq__`.

Use `field(default_factory=list)` for mutable fields, and consider `@dataclass(frozen=True)` for immutable value-like objects.

## 19. Exception Handling

Use `try` for code that may fail and `except` to handle expected exceptions. `else` runs only if no exception occurs; `finally` always runs, often for cleanup.

Catch specific exception types where possible, and create custom exception classes when an application needs meaningful domain errors.

## 20. Context Managers

A context manager handles setup and cleanup reliably around a block of code, normally using `with`.

```python
with open("data.txt", encoding="utf-8") as file:
    content = file.read()
# The file is closed even if reading raises an exception.
```

Classes can implement `__enter__` and `__exit__`; `contextlib` offers simpler helpers for custom context managers.

## 21. GIL, Threading, Multiprocessing, and Asyncio

The **Global Interpreter Lock (GIL)** in CPython allows only one thread to execute Python bytecode at a time. Threads are still useful for many I/O-bound tasks, where they wait on network or file operations.

For CPU-bound parallel work, use `multiprocessing` or native libraries that release the GIL. For high-concurrency I/O, use `asyncio` with non-blocking libraries.

## 22. Async Python

`async def` defines a coroutine, and `await` pauses it without blocking the entire event loop while another asynchronous operation progresses.

Async code is useful for many concurrent I/O operations, but putting blocking code inside a coroutine can freeze the event loop.

## 23. Memory Management

CPython primarily manages memory through **reference counting**. Its garbage collector also detects many circular-reference cases that reference counting alone cannot free.

Avoid retaining unnecessary references to large objects, and be cautious with cycles involving objects that manage external resources.

## 24. Imports and Modules

A module is a Python file that can be imported. Python caches imported modules in `sys.modules`, so top-level module code usually runs only on the first import in a process.

Use `if __name__ == "__main__":` for code that should run only when a file is executed directly, not when imported.

## 25. Type Hints

Type hints document expected types and enable static analysis tools such as mypy and pyright. Python normally does not enforce them at runtime.

Examples include `str | None`, `list[int]`, generic types, and `Protocol` for structural typing.

## 26. `@property`

`@property` lets a method be accessed like an attribute. It is useful for validation, computed values, or preserving an attribute-style API while changing the implementation.

Properties may include a setter with `@name.setter` when controlled assignment is needed.

## 27. Sorting

`sorted(iterable)` returns a new sorted list, while `list.sort()` sorts an existing list in place and returns `None`.

Both accept `key=` and `reverse=`. Python sorting is stable, so items with equal sort keys retain their original order.

## 28. Common Python Pitfalls

- **Aliasing:** assigning one list to another name does not copy it.
- **Changing a list while iterating:** can skip or repeat elements; iterate over a copy or build a new list instead.
- **Floating-point precision:** compare approximate values with `math.isclose()` rather than exact equality when appropriate.
- **Shadowing built-ins:** avoid variable names such as `list`, `dict`, or `id`.
- **Using `is` for values:** reserve it primarily for identity checks such as `is None`.

# Practice Interview Questions

1. Why can’t a list be a dictionary key, but a tuple sometimes can?
2. What happens when two variables reference the same mutable object and one changes it?
3. Explain `is` versus `==` with an example.
4. Why is a mutable default argument dangerous? How do you fix it?
5. What happens when a generator is iterated over twice?
6. Explain the GIL and when multiprocessing is preferable.
7. How does Python resolve `super()` in multiple inheritance?
8. What must `__hash__` guarantee relative to `__eq__`?
9. Compare a shallow copy and a deep copy of a nested dictionary.
10. What is the difference between `@classmethod`, `@staticmethod`, and an instance method?
11. How do decorators work internally?
12. What are closures, and what is late binding in Python?
13. Why should you use `default_factory=list` in a dataclass?
14. When would you choose a set instead of a list?
15. Explain `try`, `except`, `else`, and `finally` with a practical use case.

# Python OOP Interview Guide

Object-Oriented Programming (OOP) organizes code around **objects**: values that combine state (data) and behavior (methods). Python supports OOP but also encourages simple functions and composition when they fit better.

## 1. Classes and Objects

A **class** is a blueprint; an **object** (or instance) is a concrete value created from that blueprint. Attributes store data and methods define behavior.

```python
class Car:
    def __init__(self, brand, speed=0):
        self.brand = brand
        self.speed = speed

    def accelerate(self, amount):
        self.speed += amount

car = Car("Tesla")
car.accelerate(20)
print(car.brand, car.speed)  # Tesla 20
```

`self` refers to the current instance. It is passed automatically when calling an instance method.

## 2. Constructor: `__init__`

`__init__` initializes an instance after it is created. It is commonly called a constructor, although object creation itself is performed by `__new__`.

Use `__init__` to validate and assign initial object state.

```python
class User:
    def __init__(self, name):
        if not name:
            raise ValueError("name is required")
        self.name = name
```

## 3. Instance, Class, and Static Attributes

An **instance attribute** belongs to one object. A **class attribute** belongs to the class and is shared unless an instance shadows it. Static attributes are usually just class attributes used as constants.

```python
class Employee:
    company = "Acme"  # class attribute

    def __init__(self, name):
        self.name = name  # instance attribute

first = Employee("Asha")
second = Employee("Ravi")
first.company = "Other"  # shadows the class attribute only for first
```

Avoid mutable class attributes for per-instance data because all instances would share the same object.

## 4. Instance Methods, Class Methods, and Static Methods

- **Instance methods** receive `self` and operate on an object’s state.
- **Class methods** receive `cls` and operate on the class; they are useful for alternate constructors.
- **Static methods** receive neither automatically; they are utility functions placed inside a class for logical grouping.

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32

    @classmethod
    def from_fahrenheit(cls, value):
        return cls((value - 32) * 5 / 9)

    @staticmethod
    def is_valid(value):
        return value >= -273.15
```

## 5. Encapsulation

**Encapsulation** means keeping related data and behavior together and controlling how state is accessed or changed.

Python does not enforce truly private instance attributes in the way some languages do. Instead it uses conventions:

- `name`: public attribute.
- `_name`: internal-use convention; callers should not depend on it.
- `__name`: name mangling, intended to avoid accidental clashes in subclasses.

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("amount must be positive")
        self.__balance += amount

    @property
    def balance(self):
        return self.__balance
```

## 6. Properties: `@property`

A property exposes a method through attribute syntax. It is useful for validation, read-only values, and computed values without changing the public API.

```python
class Product:
    def __init__(self, price):
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")
        self._price = value
```

## 7. Inheritance

**Inheritance** lets a child class reuse and extend a parent class. It models an “is-a” relationship when that relationship is genuinely appropriate.

```python
class Animal:
    def speak(self):
        return "some sound"

class Dog(Animal):
    def speak(self):
        return "woof"

print(Dog().speak())  # woof
```

The child can inherit methods, override them, or add new behavior.

## 8. Method Overriding and Polymorphism

**Method overriding** occurs when a subclass supplies its own implementation of an inherited method. **Polymorphism** means code can work with objects through a common interface even when their concrete types differ.

```python
class Circle:
    def area(self):
        return 3.14159 * self.radius ** 2

    def __init__(self, radius):
        self.radius = radius

class Square:
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

def total_area(shapes):
    return sum(shape.area() for shape in shapes)
```

Python often uses **duck typing**: if an object provides the required methods, its exact class may not matter.

## 9. `super()`

Use `super()` to access parent behavior according to Python’s method resolution order (MRO). It avoids hard-coding a parent class name and works correctly with cooperative multiple inheritance.

```python
class Vehicle:
    def __init__(self, wheels):
        self.wheels = wheels

class Bike(Vehicle):
    def __init__(self, brand):
        super().__init__(wheels=2)
        self.brand = brand
```

## 10. Multiple Inheritance and MRO

Python allows a class to inherit from more than one parent. The **MRO** determines where Python searches for a method, following the C3 linearization algorithm.

```python
class A:
    def identify(self):
        return "A"

class B(A):
    pass

class C(A):
    pass

class D(B, C):
    pass

print(D.mro())  # D, B, C, A, object
```

Inspect it with `ClassName.mro()` or `ClassName.__mro__`. In multiple inheritance, classes should generally use `super()` consistently.

## 11. Abstraction and Abstract Base Classes

**Abstraction** exposes an essential interface while hiding implementation details. The `abc` module lets you define abstract methods that child classes must implement.

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount):
        """Process a payment."""

class CardProcessor(PaymentProcessor):
    def pay(self, amount):
        return f"Charged {amount} by card"

# PaymentProcessor()  # TypeError: abstract class cannot be instantiated
```

## 12. Composition vs. Inheritance

**Composition** means building an object from other objects, a “has-a” relationship. It is often more flexible than inheritance because behavior can be replaced or combined without a deep class hierarchy.

```python
class Engine:
    def start(self):
        return "engine started"

class Car:
    def __init__(self, engine):
        self.engine = engine

    def start(self):
        return self.engine.start()
```

Prefer composition when the relationship is not clearly “is-a,” or when behavior should be swappable.

## 13. Special (Dunder) Methods

Special methods let custom objects work with Python syntax and built-ins.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __eq__(self, other):
        return isinstance(other, Vector) and (self.x, self.y) == (other.x, other.y)
```

Common methods include:

- `__init__`: initialize an object.
- `__new__`: create an object before initialization.
- `__repr__` / `__str__`: developer-friendly / user-friendly text representation.
- `__eq__`, `__lt__`: comparison behavior.
- `__hash__`: hash value for dictionary keys and sets.
- `__len__`, `__iter__`, `__getitem__`: collection-like behavior.
- `__call__`: make an instance callable like a function.
- `__enter__`, `__exit__`: context-manager behavior.

## 14. `__str__` vs. `__repr__`

`__str__` produces a readable string for end users and is used by `str()` and `print()`. `__repr__` produces an unambiguous developer representation and is used in the interactive interpreter and by `repr()`.

When practical, make `__repr__` look like valid code that could recreate the object.

## 15. Equality and Hashing: `__eq__` and `__hash__`

If two objects compare equal, they must have the same hash value. This is necessary for correct dictionary and set behavior.

Mutable objects should usually not be hashable, because changing data that affects the hash after using the object as a dictionary key corrupts lookups. Defining `__eq__` without an appropriate `__hash__` normally makes instances unhashable.

## 16. Dataclasses

`@dataclass` is ideal for data-oriented classes. It can generate `__init__`, `__repr__`, and `__eq__` automatically.

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Point:
    x: int
    y: int

@dataclass
class Team:
    name: str
    members: list[str] = field(default_factory=list)
```

Use `default_factory` for mutable defaults. `frozen=True` makes normal attribute reassignment unavailable and can support hashable value objects when fields are hashable.

## 17. `__slots__`

Normally, instances store attributes in a dictionary called `__dict__`. `__slots__` restricts allowed instance attributes and can reduce memory use for many small objects.

```python
class Coordinate:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Trade-off: instances generally cannot receive arbitrary new attributes, and inheritance can be more complex.

## 18. Class Relationships

- **Association:** objects know or use each other, such as a teacher teaching a student.
- **Aggregation:** a whole contains parts that can exist independently, such as a team containing employees.
- **Composition:** a whole owns parts whose lifecycle is tied to it, such as an order containing order lines.

Python does not enforce these relationships; they are design concepts expressed through references between objects.

## 19. Method Overloading and Operator Overloading

Python does not support traditional method overloading by defining multiple same-named methods with different signatures; later definitions replace earlier ones. Use default values, `*args`, or `functools.singledispatchmethod` when needed.

**Operator overloading** is supported through dunder methods such as `__add__` for `+` and `__lt__` for `<`.

## 20. Object Lifecycle and Garbage Collection

Object creation usually involves `__new__` followed by `__init__`. CPython mainly uses reference counting and also has a garbage collector for many reference cycles.

Avoid relying on `__del__` for important cleanup. Prefer context managers (`with`) and explicit `close()` methods for files, network connections, and other external resources.

## 21. Type Hints, Protocols, and OOP

Type hints document an object’s expected interface. A `Protocol` defines structural typing: an object is acceptable if it provides the required members, regardless of inheritance.

```python
from typing import Protocol

class HasArea(Protocol):
    def area(self) -> float: ...

def print_area(shape: HasArea) -> None:
    print(shape.area())
```

This formalizes duck typing for static type checkers.

## 22. Design Principles for Interviews

- Prefer **single responsibility**: each class should have one focused reason to change.
- Favor **composition over inheritance** when it keeps behavior flexible.
- Depend on interfaces/abstractions rather than concrete implementations when practical.
- Keep public APIs small and clear; hide internal implementation details.
- Use inheritance only for a valid substitutable “is-a” relationship.

# Common OOP Interview Questions

1. What is the difference between a class and an object?
2. Explain encapsulation, inheritance, polymorphism, and abstraction in Python.
3. What are `self` and `cls`?
4. Compare instance methods, class methods, and static methods.
5. What is method overriding? Does Python support method overloading?
6. Explain `super()` and why it is better than directly naming a parent class.
7. What is the MRO, and how do you inspect it?
8. What problems can multiple inheritance cause, and how does Python address them?
9. Explain composition versus inheritance. When would you choose each?
10. What are dunder methods? Name several useful examples.
11. Compare `__str__` and `__repr__`.
12. What is the contract between `__eq__` and `__hash__`?
13. Why can mutable objects be problematic as dictionary keys?
14. What does `@property` solve, compared with directly exposing an attribute?
15. What is an abstract base class, and when would you use one?
16. What is duck typing? How do `Protocol` types relate to it?
17. What are dataclasses, and why is `default_factory` important?
18. What does `__slots__` do, and what are its trade-offs?
19. What are class attributes, and how can they accidentally be shared?
20. Why should external-resource cleanup use context managers rather than `__del__`?

