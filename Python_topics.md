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
