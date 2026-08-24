---
title: Python 效能調優
type: concept
tags: [python, performance, profiling, cProfile, numpy, cython, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 效能調優

> 原則：先量測，後優化。Python 的效能陷阱很反直覺，沒有 profiling 數據的優化是猜測。

## 一、量測工具

### cProfile（函數層級）

```python
import cProfile
import pstats
from pstats import SortKey

# 方法 1：命令列
# python -m cProfile -s cumtime my_script.py

# 方法 2：程式碼內
profiler = cProfile.Profile()
profiler.enable()
my_function()
profiler.disable()

stats = pstats.Stats(profiler)
stats.sort_stats(SortKey.CUMULATIVE)
stats.print_stats(20)  # 印出前 20 個函數

# 輸出解讀：
# ncalls  tottime  percall  cumtime  percall filename:lineno(function)
# 1000    0.523    0.001    2.341    0.002   my_module.py:42(process)
# ↑       ↑                ↑
# 呼叫次數 函數本身耗時       累計耗時（含子函數）
```

### line_profiler（行層級）

```python
# pip install line_profiler

# 在函數上加 @profile 裝飾器（執行時注入，不需要 import）
@profile
def slow_function(data):
    result = []
    for item in data:
        processed = item.strip().lower()  # 哪行慢？
        result.append(processed)
    return result

# 執行
# kernprof -l -v my_script.py

# 輸出：
# Line #  Hits      Time   Per Hit   % Time  Line Contents
# 9       1000    1234.5     1.2      45.2    processed = item.strip().lower()
# 10      1000     567.8     0.6      20.8    result.append(processed)
```

### memory_profiler（記憶體）

```python
# pip install memory_profiler

@profile
def memory_heavy():
    big_list = list(range(1_000_000))    # 多少 MB？
    filtered = [x for x in big_list if x % 2 == 0]
    return filtered

# python -m memory_profiler my_script.py

# tracemalloc（標準庫，追蹤分配來源）
import tracemalloc

tracemalloc.start()
heavy_operation()
snapshot = tracemalloc.take_snapshot()
top = snapshot.statistics('lineno')
for stat in top[:10]:
    print(stat)
# my_module.py:15: 38.1 MiB (size=39980000, count=1000000)
```

### timeit（微基準測試）

```python
import timeit

# 比較兩種實作
t1 = timeit.timeit(
    stmt='result = [x**2 for x in range(1000)]',
    number=10_000
)
t2 = timeit.timeit(
    stmt='result = list(map(lambda x: x**2, range(1000)))',
    number=10_000
)
print(f"list comp: {t1:.3f}s, map: {t2:.3f}s")
# list comp: 0.847s, map: 1.234s（list comp 通常更快）

# 命令列
# python -m timeit "sum(range(1000))"
```

---

## 二、常見效能陷阱與修法

### 字串拼接

```python
# ❌ O(n²)：每次 + 建立新字串
result = ""
for item in items:
    result += str(item)

# ✅ O(n)：join 一次分配
result = "".join(str(item) for item in items)

# ✅ 大量拼接：io.StringIO
import io
buf = io.StringIO()
for item in items:
    buf.write(str(item))
result = buf.getvalue()
```

### 列表操作

```python
# ❌ list 的 O(n) 操作
if item in my_list:    # O(n)
    my_list.remove(item)  # O(n)

# ✅ 改用 set/dict
my_set = set(my_list)
if item in my_set:     # O(1)
    my_set.discard(item)

# ❌ 頭部插入 O(n)
my_list.insert(0, item)

# ✅ 用 deque
from collections import deque
d = deque()
d.appendleft(item)     # O(1)

# ✅ list comprehension vs for + append（前者通常快 1.5-2x）
squares = [x**2 for x in range(1000)]   # 快
squares = []
for x in range(1000):
    squares.append(x**2)                # 慢（append 的查找開銷）
```

### 屬性查找優化

```python
# ❌ 在熱循環中重複的屬性查找
import math
for i in range(1_000_000):
    result = math.sqrt(i)   # 每次都要查找 math.sqrt

# ✅ 本地化（local lookup 比 global 快 ~10-20%）
sqrt = math.sqrt
for i in range(1_000_000):
    result = sqrt(i)

# 同樣適用於方法
append = my_list.append
for item in data:
    append(item)            # 比 my_list.append(item) 快
```

### 生成器 vs 列表

```python
# 不需要全部結果時，用生成器節省記憶體
# ❌ 建立完整列表再取前 N 個
first_10 = [process(x) for x in million_items][:10]

# ✅ 生成器：惰性求值
import itertools
first_10 = list(itertools.islice(
    (process(x) for x in million_items), 10
))
```

---

## 三、NumPy 向量化

NumPy 的核心優化：用 C 實作的向量運算取代 Python 的 for 迴圈。

```python
import numpy as np

# ❌ Python 迴圈（慢）
data = list(range(1_000_000))
result = [x ** 2 + 2 * x + 1 for x in data]

# ✅ NumPy 向量化（快 50-100x）
arr = np.arange(1_000_000)
result = arr ** 2 + 2 * arr + 1

# 廣播（Broadcasting）：不同形狀的 array 運算
matrix = np.ones((1000, 1000))
row = np.arange(1000)       # shape (1000,)
result = matrix + row        # 自動廣播到 (1000, 1000)

# ❌ 逐元素用 Python 函數（np.vectorize 不快！）
np.vectorize(my_python_func)(arr)  # 只是語法糖，底層仍是 Python 迴圈

# ✅ 用 NumPy 的 ufunc 或 where
result = np.where(arr > 0, np.log(arr), 0)  # 向量化條件

# 記憶體視圖（避免複製）
a = np.array([1, 2, 3, 4, 5])
b = a[1:4]          # view，不複製
b[0] = 99           # a[1] 也變成 99！
c = a[1:4].copy()   # 明確複製，獨立
```

---

## 四、functools 快取

```python
from functools import lru_cache, cache

# lru_cache：有大小限制（LRU 驅逐）
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(100)                    # 第一次計算
fibonacci(100)                    # 直接從快取返回
print(fibonacci.cache_info())     # CacheInfo(hits=..., misses=..., maxsize=128, currsize=...)
fibonacci.cache_clear()           # 清除快取

# cache（Python 3.9+）：無大小限制（適合純計算且參數有限）
@cache
def parse_config(path: str) -> dict:
    with open(path) as f:
        return json.load(f)

# ⚠️ 注意：lru_cache 的參數必須可 hash
# list, dict 不能作為參數 → 用 tuple 或 frozenset
@lru_cache
def process_items(items: tuple[int, ...]) -> int:
    return sum(items)

process_items(tuple(my_list))  # ✅
```

---

## 五、__slots__ 效能影響

```python
import sys
from dataclasses import dataclass

# 普通 class：每個 instance 有 __dict__
class NormalPoint:
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

# __slots__ class
class SlottedPoint:
    __slots__ = ('x', 'y', 'z')
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

# dataclass + slots（Python 3.10+）
@dataclass(slots=True)
class SlottedDataclass:
    x: float
    y: float
    z: float

n = NormalPoint(1, 2, 3)
s = SlottedPoint(1, 2, 3)
print(sys.getsizeof(n) + sys.getsizeof(n.__dict__))  # ~280 bytes
print(sys.getsizeof(s))                               # ~64 bytes（少 4x）

# 屬性存取速度：__slots__ 比 __dict__ 快 ~15%
# 記憶體佔用：__slots__ 少 ~75%（大量 instance 時顯著）
```

---

## 六、Cython / C Extension

當 Python 本身不夠快時的逃生口：

```python
# Cython：Python-like 語法編譯成 C extension
# my_module.pyx

def fibonacci(int n) -> int:       # 靜態型別聲明
    cdef int a = 0, b = 1, i      # C 語言的本地變數
    for i in range(n):
        a, b = b, a + b
    return a

# 編譯後：速度接近 C，自動釋放 GIL（nogil 修飾器）
```

```python
# ctypes：呼叫現有 C library
import ctypes

lib = ctypes.CDLL('./my_lib.so')
lib.add.argtypes = [ctypes.c_int, ctypes.c_int]
lib.add.restype = ctypes.c_int
result = lib.add(3, 4)

# cffi：更 Pythonic 的 C 介面
# numba：JIT 編譯 NumPy 代碼（GPU 也支援）
from numba import jit

@jit(nopython=True)
def fast_sum(arr):
    total = 0
    for x in arr:
        total += x
    return total
```

---

## 七、效能快速參考

```
操作                           大約耗時
────────────────────────────────────────
Python function call           ~100ns
dict lookup                    ~50ns
list append                    ~60ns
list indexing                  ~40ns
attribute lookup (dict-based)  ~50ns
attribute lookup (__slots__)   ~40ns
int + int                      ~20ns
float + float                  ~30ns
isinstance check               ~50ns
try/except (no exception)      ~30ns
try/except (with exception)    ~10μs（有異常時貴！）

建議：
- 避免在熱迴圈中 raise/catch exception
- 屬性查找本地化（把 obj.method 存到本地變數）
- 用 in 在 set/dict 查找，避免用 list
- 字串用 join，不用 +
- 大量數值計算用 NumPy
- 考慮用 PyPy（替代 CPython，JIT 加速）
```

## 相關頁面

- [[Python GIL與並發模型]] — 並行策略對效能的影響
- [[Python記憶體管理]] — 記憶體分配的 profiling
- [[Go pprof實戰完整指南]] — 對比 Go 的效能分析方法
