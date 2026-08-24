---
title: Python 型別系統與類型提示
type: concept
tags: [python, typing, type-hints, Protocol, TypeVar, dataclass, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 型別系統與類型提示

Python 是動態型別語言，但自 3.5 起的 `typing` 模組讓靜態型別分析成為可能，搭配 mypy / pyright / pylance 可在執行前發現型別錯誤。

## 基本類型提示

```python
# 變數標注
name: str = "Alice"
age: int = 30
items: list[str] = []          # Python 3.9+ 直接用 list[str]
mapping: dict[str, int] = {}
pair: tuple[int, str] = (1, "a")
nullable: str | None = None    # Python 3.10+ 用 | 代替 Optional

# 函數標注
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}! " * times).strip()

# 複雜型別
from typing import Callable, Iterator, Generator

def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

def count_up(n: int) -> Iterator[int]:
    for i in range(n):
        yield i

# Generator[YieldType, SendType, ReturnType]
def accumulator() -> Generator[int, int, str]:
    total = 0
    while True:
        value = yield total
        if value is None:
            return f"Total: {total}"
        total += value
```

## TypeVar 與泛型

```python
from typing import TypeVar, Generic

T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

# 泛型函數
def first(items: list[T]) -> T:
    return items[0]

first([1, 2, 3])        # T = int，返回 int
first(["a", "b"])       # T = str，返回 str

# 有約束的 TypeVar
Numeric = TypeVar('Numeric', int, float)

def double(x: Numeric) -> Numeric:
    return x * 2

# 泛型類別
class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

s: Stack[int] = Stack()
s.push(42)

# Python 3.12+ 新語法（更簡潔）
def first[T](items: list[T]) -> T:  # type parameter syntax
    return items[0]

class Stack[T]:
    def push(self, item: T) -> None: ...
```

## Protocol（結構型別 / Duck Typing）

`Protocol` 定義「有哪些方法」的介面，不需要繼承（Structural Subtyping）。

```python
from typing import Protocol, runtime_checkable

# 定義 Protocol
class Drawable(Protocol):
    def draw(self) -> None: ...
    def resize(self, factor: float) -> None: ...

# 任何實作了 draw 和 resize 的類都符合 Drawable，不需要繼承
class Circle:
    def draw(self) -> None: print("Drawing circle")
    def resize(self, factor: float) -> None: self.radius *= factor

class Square:
    def draw(self) -> None: print("Drawing square")
    def resize(self, factor: float) -> None: self.side *= factor

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())   # ✅ mypy 接受（Circle 符合 Drawable）
render(Square())   # ✅ mypy 接受

# runtime_checkable：讓 isinstance 也能用
@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

import io
print(isinstance(io.FileIO("f"), Closeable))  # True
```

**Protocol vs ABC（Abstract Base Class）**：
```python
from abc import ABC, abstractmethod

# ABC：名義型別（Nominal），必須明確繼承
class Drawable(ABC):
    @abstractmethod
    def draw(self) -> None: ...

class Circle(Drawable):  # 必須繼承 Drawable
    def draw(self): print("Drawing")

# Protocol：結構型別（Structural），不需要繼承
# 選擇準則：
# - 自己控制的類別 → ABC（更明確的意圖）
# - 第三方類別的鴨子型別 → Protocol（不需要修改第三方代碼）
```

## dataclass

```python
from dataclasses import dataclass, field, KW_ONLY
from typing import ClassVar

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0  # 有預設值的欄位必須在後面

# 自動生成：__init__, __repr__, __eq__
p = Point(1.0, 2.0)
print(p)            # Point(x=1.0, y=2.0, z=0.0)
print(p == Point(1.0, 2.0))  # True

@dataclass(frozen=True)         # 不可變（自動生成 __hash__）
class ImmutablePoint:
    x: float
    y: float

@dataclass(order=True)          # 自動生成 __lt__, __le__ 等比較方法
class SortableItem:
    priority: int               # 比較按欄位聲明順序
    name: str

# 複雜預設值：用 field(default_factory=...)
@dataclass
class Config:
    tags: list[str] = field(default_factory=list)   # ❌ 不能用 tags: list = []（共用同一個 list！）
    metadata: dict = field(default_factory=dict)
    _internal: int = field(default=0, repr=False, compare=False)  # 不顯示、不比較

    # ClassVar：類別變數，不是 instance 欄位
    count: ClassVar[int] = 0

# __post_init__：初始化後的驗證/計算
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)  # 不在 __init__ 中設定

    def __post_init__(self):
        if self.width <= 0 or self.height <= 0:
            raise ValueError("Dimensions must be positive")
        self.area = self.width * self.height
```

## TypedDict

```python
from typing import TypedDict, Required, NotRequired

class Movie(TypedDict):
    title: str
    year: int

class MovieOptional(TypedDict, total=False):  # 所有欄位都是可選的
    rating: float
    genre: str

# Python 3.11+：混合 Required / NotRequired
class User(TypedDict):
    name: Required[str]      # 必須
    email: Required[str]     # 必須
    age: NotRequired[int]    # 可選

# 使用
def process_movie(movie: Movie) -> str:
    return f"{movie['title']} ({movie['year']})"

m: Movie = {"title": "Matrix", "year": 1999}
process_movie(m)  # mypy 通過
```

## 進階型別工具

```python
from typing import Literal, Union, Annotated, Final, overload

# Literal：限制只能是特定值
def set_direction(direction: Literal["north", "south", "east", "west"]) -> None: ...

# Final：常數（不能重新賦值）
MAX_SIZE: Final = 100

# Annotated：帶元資料的型別（供 pydantic, FastAPI 等使用）
from annotated_types import Gt, Le

PositiveInt = Annotated[int, Gt(0)]
Percentage = Annotated[float, Gt(0), Le(100)]

def process(value: PositiveInt) -> None: ...

# overload：同一函數的多個型別簽名
from typing import overload

@overload
def process(x: int) -> int: ...
@overload
def process(x: str) -> str: ...
def process(x):          # 實際實作（無型別標注）
    if isinstance(x, int):
        return x * 2
    return x.upper()

# ParamSpec（Python 3.10+）：裝飾器的型別傳遞
from typing import ParamSpec, Concatenate
import functools

P = ParamSpec('P')

def log_calls(func: Callable[P, T]) -> Callable[P, T]:
    @functools.wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(x: int, y: int) -> int:  # 型別資訊完整保留
    return x + y
```

## 執行時型別檢查（Pydantic）

```python
from pydantic import BaseModel, Field, validator, field_validator

class UserCreate(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    email: str = Field(pattern=r'^[\w.-]+@[\w.-]+\.\w+$')
    age: int = Field(ge=0, le=150)
    tags: list[str] = []

    @field_validator('name')
    @classmethod
    def name_must_not_contain_space(cls, v: str) -> str:
        if ' ' in v:
            raise ValueError('name must not contain spaces')
        return v.strip()

# 使用（自動驗證）
user = UserCreate(name="Alice", email="alice@example.com", age=30)
# UserCreate(name='Alice', email='alice@example.com', age=30, tags=[])

UserCreate(name="", email="bad", age=-1)
# ValidationError: 3 validation errors
```

## mypy / pyright 使用

```bash
# 安裝
pip install mypy pyright

# mypy 檢查
mypy my_module.py
mypy --strict my_module.py   # 嚴格模式（推薦新專案）

# pyright（更快，VSCode Pylance 底層）
pyright my_module.py

# 常用配置（mypy.ini 或 pyproject.toml）
[mypy]
strict = true
ignore_missing_imports = true
```

## 相關頁面

- [[Python資料模型與描述器]] — Protocol 和描述器協定的關係
- [[Python效能調優]] — dataclass vs namedtuple vs 普通 class 的效能
- [[Go泛型設計]] — 對比 Go 與 Python 泛型系統的設計哲學
