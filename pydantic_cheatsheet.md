# Pydantic Cheatsheet (v2)

A practical reference to Pydantic's core concepts, each with a runnable example.

---

## 1. Basic Model

The foundation of Pydantic: define fields with type hints, get validation for free.

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    is_active: bool = True   # default value -> optional field

u = User(id="123", name="Alice")   # "123" is coerced to int
print(u)          # id=123 name='Alice' is_active=True
print(u.id)        # 123 (int, not str)
```

---

## 2. Required vs Optional Fields

```python
from typing import Optional
from pydantic import BaseModel

class Item(BaseModel):
    name: str                    # required
    price: float                 # required
    description: Optional[str] = None   # optional, defaults to None
    tags: list[str] = []         # optional, default empty list
```

> Use `Optional[X] = None` (or `X | None = None` in Python 3.10+) — a type hint alone doesn't make a field optional; it's the **default value** that does.

---

## 3. `Field()` — Extra Validation & Metadata

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    price: float = Field(gt=0, description="Price must be positive")
    quantity: int = Field(default=1, ge=0, le=1000)
    sku: str = Field(alias="SKU")  # accept "SKU" as input key
```

- `...` (Ellipsis) marks a field as required when using `Field()`.
- Common constraints: `gt`, `ge`, `lt`, `le`, `min_length`, `max_length`, `pattern`, `multiple_of`.
- `alias` lets the input JSON/dict use a different key name than the Python attribute.

---

## 4. Nested Models

Models can contain other models — validation cascades automatically.

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    zip_code: str

class Person(BaseModel):
    name: str
    address: Address

p = Person(name="Bob", address={"city": "NYC", "zip_code": "10001"})
print(p.address.city)   # NYC
```

---

## 5. Lists, Dicts, and Nested Collections

```python
from pydantic import BaseModel

class Order(BaseModel):
    items: list[str]
    quantities: dict[str, int]
    prices: list[float] = []

o = Order(items=["pen", "book"], quantities={"pen": 2, "book": 1})
```

---

## 6. Field Validators (`field_validator`)

Custom validation logic for a single field.

```python
from pydantic import BaseModel, field_validator

class Account(BaseModel):
    username: str
    password: str

    @field_validator("username")
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError("must be alphanumeric")
        return v

    @field_validator("password")
    @classmethod
    def password_strength(cls, v: str) -> str:
        if len(v) < 8:
            raise ValueError("password too short")
        return v
```

- Runs after Pydantic's built-in type coercion by default (`mode="before"` runs prior to it).
- Must return the (possibly transformed) value.

---

## 7. Model Validators (`model_validator`)

Validate relationships **between** fields.

```python
from pydantic import BaseModel, model_validator

class DateRange(BaseModel):
    start: int
    end: int

    @model_validator(mode="after")
    def check_range(self) -> "DateRange":
        if self.start >= self.end:
            raise ValueError("start must be before end")
        return self
```

- `mode="before"`: receives raw input (dict) before field validation — good for reshaping data.
- `mode="after"`: receives the fully-built model instance — good for cross-field checks.

---

## 8. Computed Fields

Derive a value from other fields and include it in serialization.

```python
from pydantic import BaseModel, computed_field

class Rectangle(BaseModel):
    width: float
    height: float

    @computed_field
    @property
    def area(self) -> float:
        return self.width * self.height

r = Rectangle(width=3, height=4)
print(r.model_dump())   # {'width': 3.0, 'height': 4.0, 'area': 12.0}
```

---

## 9. Serialization: `model_dump()` / `model_dump_json()`

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    password: str

u = User(id=1, name="Alice", password="secret")

u.model_dump()                          # -> dict
u.model_dump_json()                     # -> JSON string
u.model_dump(exclude={"password"})      # drop sensitive fields
u.model_dump(include={"id", "name"})    # only these fields
u.model_dump(by_alias=True)             # use field aliases as keys
u.model_dump(exclude_none=True)         # skip fields that are None
```

---

## 10. Parsing Input

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str

User.model_validate({"id": 1, "name": "Alice"})   # from dict
User.model_validate_json('{"id": 1, "name": "Alice"}')  # from JSON string
```

---

## 11. `model_config` — Model Configuration

Replaces the old `class Config:` inner class from Pydantic v1.

```python
from pydantic import BaseModel, ConfigDict

class Strict(BaseModel):
    model_config = ConfigDict(
        strict=True,          # disable type coercion (e.g. "1" won't become 1)
        extra="forbid",       # reject unknown fields (options: "ignore", "allow", "forbid")
        frozen=True,          # make instances immutable/hashable
        populate_by_name=True # allow both alias and field name as input
    )
    id: int
    name: str
```

---

## 12. Custom Types with `Annotated`

```python
from typing import Annotated
from pydantic import BaseModel, Field, AfterValidator

def must_be_even(v: int) -> int:
    if v % 2 != 0:
        raise ValueError("must be even")
    return v

EvenInt = Annotated[int, AfterValidator(must_be_even)]

class Config(BaseModel):
    batch_size: EvenInt
    port: Annotated[int, Field(gt=0, lt=65536)]
```

---

## 13. Enums

```python
from enum import Enum
from pydantic import BaseModel

class Status(str, Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

class Task(BaseModel):
    name: str
    status: Status = Status.ACTIVE

t = Task(name="Deploy", status="active")   # string coerced to Enum
```

---

## 14. Union Types & Discriminated Unions

```python
from typing import Union, Literal
from pydantic import BaseModel, Field

class Cat(BaseModel):
    pet_type: Literal["cat"]
    meow_volume: int

class Dog(BaseModel):
    pet_type: Literal["dog"]
    bark_volume: int

class Owner(BaseModel):
    # discriminator tells Pydantic which model to use based on "pet_type"
    pet: Union[Cat, Dog] = Field(discriminator="pet_type")

o = Owner(pet={"pet_type": "dog", "bark_volume": 5})
print(type(o.pet))   # <class '__main__.Dog'>
```

---

## 15. Error Handling (`ValidationError`)

```python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    id: int
    name: str

try:
    User(id="not-a-number", name="Alice")
except ValidationError as e:
    print(e.errors())
    # [{'type': 'int_parsing', 'loc': ('id',), 'msg': 'Input should be a valid integer', ...}]
    print(e.json())   # machine-readable error list
```

---

## 16. Generic Models

Reusable models parameterized by type — great for API response wrappers.

```python
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar("T")

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: T

class UserData(BaseModel):
    id: int
    name: str

resp = ApiResponse[UserData](success=True, data={"id": 1, "name": "Alice"})
```

---

## 17. Settings Management (`pydantic-settings`)

Load configuration from environment variables / `.env` files.

```python
# pip install pydantic-settings
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    debug: bool = False

    class Config:
        env_file = ".env"

settings = Settings()   # reads DATABASE_URL, DEBUG from environment
```

---

## 18. Private Attributes & Class Vars

```python
from pydantic import BaseModel, PrivateAttr
from typing import ClassVar

class Session(BaseModel):
    user_id: int
    _token: str = PrivateAttr(default="secret")   # not part of the schema/dump
    MAX_SESSIONS: ClassVar[int] = 5                # not a field at all
```

---

## 19. `TypeAdapter` — Validate Non-Model Types

Validate plain types (lists, dicts, primitives) without wrapping them in a `BaseModel`.

```python
from pydantic import TypeAdapter

adapter = TypeAdapter(list[int])
adapter.validate_python(["1", "2", "3"])   # -> [1, 2, 3]

adapter2 = TypeAdapter(dict[str, int])
adapter2.validate_python({"a": "1"})       # -> {'a': 1}
```

---

## 20. Dataclasses Integration

```python
from pydantic import field_validator
from pydantic.dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

    @field_validator("x")
    @classmethod
    def x_positive(cls, v):
        if v < 0:
            raise ValueError("x must be positive")
        return v

p = Point(x="3", y=4)   # still gets Pydantic validation + coercion
```

---

## 21. JSON Schema Generation

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str

print(User.model_json_schema())
# {'title': 'User', 'type': 'object', 'properties': {...}, 'required': [...]}
```

Useful for auto-generating OpenAPI docs (FastAPI does this automatically).

---

## Quick Reference Table

| Concept                   | v2 API                           | Purpose                                  |
| ------------------------- | -------------------------------- | ---------------------------------------- |
| Define model              | `class X(BaseModel)`             | Core schema definition                   |
| Field constraints         | `Field(gt=, min_length=, ...)`   | Per-field validation rules               |
| Single-field custom logic | `@field_validator`               | Custom checks/transforms                 |
| Cross-field logic         | `@model_validator`               | Validate relationships between fields    |
| Derived field             | `@computed_field`                | Include computed values in output        |
| Config                    | `model_config = ConfigDict(...)` | Model-wide behavior (was `class Config`) |
| To dict                   | `model_dump()`                   | Serialize to Python dict                 |
| To JSON                   | `model_dump_json()`              | Serialize to JSON string                 |
| From dict                 | `model_validate()`               | Parse + validate dict                    |
| From JSON                 | `model_validate_json()`          | Parse + validate JSON string             |
| Validate non-model types  | `TypeAdapter`                    | Validate lists/dicts/primitives directly |
| Errors                    | `ValidationError`                | Catch and inspect validation failures    |
| Env-based config          | `pydantic_settings.BaseSettings` | Load settings from env/`.env`            |

---

**Note:** This covers Pydantic v2 (current stable, released 2023+). If you're on v1 (`pydantic<2`), key differences: `.dict()`/`.json()` instead of `model_dump()`/`model_dump_json()`, `@validator` instead of `@field_validator`, `class Config:` instead of `model_config`, and `.parse_obj()`/`.parse_raw()` instead of `model_validate()`/`model_validate_json()`.
