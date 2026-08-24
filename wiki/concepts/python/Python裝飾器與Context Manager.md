---
title: Python 裝飾器與 Context Manager
type: concept
tags: [python, decorator, context-manager, functools, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 裝飾器與 Context Manager

## 裝飾器（Decorator）

裝飾器是**接收函數、返回函數的高階函數**，用 `@syntax` 語法糖套用。

### 基礎裝飾器

```python
import functools

def my_decorator(func):
    @functools.wraps(func)   # 必須！保留原函數的 __name__, __doc__, __annotations__
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@my_decorator
def greet(name: str) -> str:
    """Greet someone."""
    return f"Hello, {name}!"

# 等同於：greet = my_decorator(greet)

print(greet.__name__)  # "greet"（有 @wraps 才正確；沒有的話是 "wrapper"）
print(greet.__doc__)   # "Greet someone."
```

### 帶參數的裝飾器（Decorator Factory）

```python
def retry(max_attempts: int = 3, delay: float = 1.0):
    """裝飾器工廠：返回真正的裝飾器"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    import time
                    time.sleep(delay)
                    print(f"Retry {attempt + 1}/{max_attempts}: {e}")
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def fetch_data(url: str) -> dict:
    return requests.get(url).json()

# 等同於：fetch_data = retry(max_attempts=5, delay=0.5)(fetch_data)
```

### Class-based 裝飾器

```python
class RateLimit:
    """用 class 實作裝飾器（可以有狀態）"""
    def __init__(self, calls_per_second: float):
        self.min_interval = 1.0 / calls_per_second
        self.last_call = 0.0

    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import time
            elapsed = time.time() - self.last_call
            if elapsed < self.min_interval:
                time.sleep(self.min_interval - elapsed)
            self.last_call = time.time()
            return func(*args, **kwargs)
        return wrapper

@RateLimit(calls_per_second=5)
def api_call(endpoint: str) -> dict:
    return requests.get(endpoint).json()
```

### 實用裝飾器模式

```python
import time
import functools
import logging

# 1. 計時裝飾器
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

# 2. 快取（自己實作，理解 lru_cache 底層）
def simple_cache(func):
    cache = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    wrapper.cache = cache  # 暴露 cache 以便清除
    wrapper.cache_clear = cache.clear
    return wrapper

# 3. 型別驗證（實際應用用 pydantic）
def validate_types(func):
    hints = func.__annotations__
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        import inspect
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)
        for param_name, value in bound.arguments.items():
            if param_name in hints and not isinstance(value, hints[param_name]):
                raise TypeError(
                    f"{param_name} must be {hints[param_name].__name__}, "
                    f"got {type(value).__name__}"
                )
        return func(*args, **kwargs)
    return wrapper

@validate_types
def add(x: int, y: int) -> int:
    return x + y

add(1, 2)     # OK
add(1, "2")   # TypeError: y must be int, got str
```

### 裝飾器的堆疊順序

```python
@decorator_A
@decorator_B
@decorator_C
def func(): ...

# 等同於：func = decorator_A(decorator_B(decorator_C(func)))
# 執行時：A 的 wrapper → B 的 wrapper → C 的 wrapper → 原函數
#         ↑ 先套用外層（A）   ↑ 後套用（C 最先包）    ↑ 最後執行
```

---

## Context Manager

Context Manager 保證資源在使用後被正確釋放，即使發生異常。

### 基於 class 的 Context Manager

```python
class ManagedResource:
    def __enter__(self):
        self.resource = acquire_resource()
        return self.resource  # as 子句中的值

    def __exit__(self, exc_type, exc_val, exc_tb):
        # exc_type/val/tb 是異常資訊（無異常則都是 None）
        try:
            if exc_type is not None:
                self.resource.rollback()
            else:
                self.resource.commit()
        finally:
            self.resource.release()
        return False  # False/None = 不抑制異常；True = 吞掉異常

with ManagedResource() as res:
    res.do_work()
```

### 基於 generator 的 Context Manager（更常用）

```python
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource           # with ... as resource 中的值
    except SomeError as e:
        resource.rollback()
        raise                    # 重新拋出（或 return 來抑制）
    else:
        resource.commit()
    finally:
        resource.release()       # 無論如何都執行

# 計時 context manager
@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield                    # 沒有 as 時不需要 yield 值
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.4f}s")

with timer("processing"):
    heavy_computation()

# 暫時修改狀態
@contextmanager
def temp_directory():
    import tempfile, os, shutil
    tmpdir = tempfile.mkdtemp()
    old_cwd = os.getcwd()
    try:
        os.chdir(tmpdir)
        yield tmpdir
    finally:
        os.chdir(old_cwd)
        shutil.rmtree(tmpdir)
```

### contextlib 工具集

```python
from contextlib import (
    contextmanager,
    suppress,
    ExitStack,
    asynccontextmanager,
    AbstractContextManager,
)

# suppress：抑制特定異常
with suppress(FileNotFoundError):
    os.remove("maybe_exists.tmp")
# 等同於：
try:
    os.remove("maybe_exists.tmp")
except FileNotFoundError:
    pass

# ExitStack：動態數量的 context manager
with ExitStack() as stack:
    files = [stack.enter_context(open(path)) for path in file_paths]
    # 離開 with 時所有 file 都自動 close

# 動態決定是否用 context manager
@contextmanager
def maybe_transaction(use_transaction: bool):
    if use_transaction:
        with db.transaction():
            yield
    else:
        yield

# async context manager
@asynccontextmanager
async def async_managed():
    conn = await db.connect()
    try:
        yield conn
    finally:
        await conn.close()

async def main():
    async with async_managed() as conn:
        await conn.execute("SELECT 1")
```

---

## 進階：functools 工具

```python
import functools

# partial：固定函數的部分參數
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(4))   # 16
print(cube(3))     # 27

# reduce：累計操作
from functools import reduce
product = reduce(lambda x, y: x * y, [1, 2, 3, 4, 5])  # 120

# total_ordering：只需實作 __eq__ 和 __lt__，自動補齊其餘比較方法
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor, patch):
        self.version = (major, minor, patch)

    def __eq__(self, other):
        return self.version == other.version

    def __lt__(self, other):
        return self.version < other.version

# 自動獲得：__le__, __gt__, __ge__
v1 = Version(1, 2, 3)
v2 = Version(1, 3, 0)
print(v1 < v2)   # True
print(v1 >= v2)  # False（自動推導）

# singledispatch：函數重載（依第一個參數型別）
from functools import singledispatch

@singledispatch
def process(data):
    raise TypeError(f"Cannot process {type(data)}")

@process.register(int)
def _(data: int):
    return data * 2

@process.register(str)
def _(data: str):
    return data.upper()

@process.register(list)
def _(data: list):
    return [process(item) for item in data]

process(42)           # 84
process("hello")      # "HELLO"
process([1, "a", 2])  # [2, "A", 4]
```

## 相關頁面

- [[Python型別系統與類型提示]] — 裝飾器的 ParamSpec 型別保留
- [[Python資料模型與描述器]] — Context Manager 的 `__enter__` / `__exit__` 是資料模型的一部分
- [[Python非同步程式設計]] — asynccontextmanager 在 asyncio 中的使用
