---
title: Python 記憶體管理
type: concept
tags: [python, memory, GC, reference-counting, weak-reference, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 記憶體管理

## 一、引用計數（Reference Counting）

CPython 的主要記憶體管理機制：每個物件有一個 `ob_refcnt` 欄位，記錄有多少個引用指向它。**計數歸零時立即釋放**。

```python
import sys

a = []             # ob_refcnt = 1
b = a              # ob_refcnt = 2
del a              # ob_refcnt = 1
del b              # ob_refcnt = 0 → 立即釋放

# sys.getrefcount 回傳引用計數（但本身也會+1）
x = [1, 2, 3]
print(sys.getrefcount(x))  # 2（一個是 x，一個是 getrefcount 的參數）

# 引用計數的增加來源：
# - 變數賦值
# - 放入容器（list/dict/set）
# - 函數參數傳遞
# - 閉包捕捉
```

**優點**：物件釋放即時、確定性（RAII 風格）
**缺點**：無法處理循環引用

## 二、循環引用（Cyclic References）與分代 GC

```python
# ❌ 循環引用：引用計數永遠不會歸零
class Node:
    def __init__(self):
        self.next = None

a = Node()
b = Node()
a.next = b  # a 引用 b
b.next = a  # b 引用 a

del a
del b
# a 和 b 的引用計數都是 1（互相引用），但沒有任何外部引用
# → 記憶體洩漏（沒有 GC 的話）
```

### 分代垃圾回收（Generational GC）

CPython 用「分代回收」處理循環引用，補充引用計數的不足：

```python
import gc

# 三代（generation 0, 1, 2）
# - 新物件進入 gen 0
# - 存活過一次 GC 進入 gen 1
# - 再存活進入 gen 2（最老的物件）
# - 各代有不同的 GC 觸發閾值

print(gc.get_threshold())  # (700, 10, 10) 預設值
# gen0：分配次數 - 釋放次數 > 700 時觸發
# gen1：gen0 GC 次數 > 10 時觸發
# gen2：gen1 GC 次數 > 10 時觸發

# 手動觸發（生產環境不建議）
gc.collect()       # 回收所有代
gc.collect(0)      # 只回收 gen 0

# 查看 GC 統計
print(gc.get_count())    # 各代當前物件計數
print(gc.get_stats())    # 詳細統計（Python 3.4+）

# 追蹤循環垃圾
gc.set_debug(gc.DEBUG_LEAK)  # 打印無法回收的物件（有 __del__ 的循環引用）
```

### 哪些物件會被 GC 追蹤？

```python
# GC 只追蹤「容器型別」（可能形成循環）：
# list, dict, set, tuple（含可變元素）, class instances, functions, ...

# 以下不被 GC 追蹤（純 leaf 物件）：
# int, float, str, bytes, None, bool
# → 這些型別只靠引用計數就夠了

import gc
a = [1, 2, 3]
print(gc.is_tracked(a))     # True
print(gc.is_tracked(42))    # False
print(gc.is_tracked("hi"))  # False
```

## 三、`__del__` 的陷阱

```python
# ❌ 有 __del__ 的循環引用，GC 在 Python 3.4 之前無法回收
class Resource:
    def __del__(self):
        print("cleaning up")  # 可能永遠不被呼叫！

a = Resource()
b = Resource()
a.other = b
b.other = a
del a, b
# Python 3.4+：GC 會正確回收，__del__ 最終會被呼叫
# Python 3.3 之前：循環引用含 __del__ 的物件進入 gc.garbage，永不回收

# ✅ 用 contextlib.contextmanager 或 __enter__/__exit__ 取代 __del__
# 或用 weakref.finalize 注冊清理函數（更安全）
import weakref

class Connection:
    pass

def cleanup(obj_ref, resource):
    print(f"Cleaning up {resource}")

conn = Connection()
weakref.finalize(conn, cleanup, weakref.ref(conn), "db connection")
```

## 四、弱引用（Weak References）

弱引用不增加引用計數，讓 GC 可以回收物件。被回收後 `weakref.ref()` 返回 `None`。

```python
import weakref

class BigObject:
    def __init__(self, data):
        self.data = data

obj = BigObject("important data")

# 建立弱引用
weak = weakref.ref(obj)

print(weak())        # <__main__.BigObject object>（物件還活著）
print(weak().data)   # "important data"

del obj              # 引用計數歸零，立即回收

print(weak())        # None（物件已被回收）

# weakref.WeakValueDictionary：value 是弱引用，物件被 GC 後自動移除 entry
cache = weakref.WeakValueDictionary()
obj = BigObject("cached data")
cache["key"] = obj

del obj
import gc; gc.collect()
print(dict(cache))   # {}（entry 已自動清除）

# weakref.WeakSet：成員是弱引用
listeners = weakref.WeakSet()
```

## 五、記憶體池（pymalloc）

CPython 有自己的記憶體分配器，針對小物件（≤ 512 bytes）做優化：

```
OS 記憶體
└── CPython Arena（256KB 塊）
    └── Pool（4KB）
        └── Block（固定大小，如 8/16/24/.../512 bytes）

分配小物件時：
1. 找對應大小的 pool
2. 從 pool 取出 block（O(1)，無需呼叫 malloc）
3. 釋放時 block 歸還給 pool（不一定還給 OS）
```

```python
# 這解釋了為何 Python 進程的 RSS 不會隨物件刪除立即下降
# Pool 仍然持有這些記憶體，以備後用

# 可用 tracemalloc 追蹤記憶體分配
import tracemalloc

tracemalloc.start()

# ... 你的代碼 ...

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

for stat in top_stats[:5]:
    print(stat)
# output: my_file.py:42: 1.5 KiB (size=1536, count=3)
```

## 六、常見記憶體洩漏場景

```python
# 1. 無限增長的快取（沒有 eviction）
_cache = {}
def expensive_compute(key):
    if key not in _cache:
        _cache[key] = compute(key)  # _cache 只進不出
    return _cache[key]
# ✅ 修法：用 functools.lru_cache 或 cachetools.LRUCache

from functools import lru_cache

@lru_cache(maxsize=1000)
def expensive_compute(key):
    return compute(key)

# 2. 全域 list/dict append（event listener 忘記移除）
_listeners = []
def register(callback):
    _listeners.append(callback)  # 如果 callback 持有大物件，記憶體無法釋放
# ✅ 修法：用 weakref.WeakSet 或提供 unregister

# 3. 閉包意外捕捉大物件
def make_handler(large_data):
    def handler(event):
        # large_data 被閉包捕捉，handler 活著時 large_data 不會被 GC
        return process(event)
    return handler
# ✅ 只傳入需要的欄位，不傳整個大物件

# 4. Thread-local 存放了大物件
import threading
local = threading.local()
local.session = create_large_session()  # thread 結束後 session 被回收嗎？
# → 是的，但如果用 thread pool（thread 重用），thread 不會結束，local 一直留著！
```

## 七、`__slots__` 減少記憶體佔用

```python
# 普通 class：每個 instance 有一個 __dict__（dict 本身很耗記憶體）
class PointNormal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# 有 __slots__：不建立 __dict__，用固定 offset 存屬性
class PointSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
p1 = PointNormal(1, 2)
p2 = PointSlots(1, 2)
print(sys.getsizeof(p1))           # ~48 bytes（不含 __dict__）
print(p1.__dict__)                 # {'x': 1, 'y': 2}（~240 bytes）
print(sys.getsizeof(p2))           # ~56 bytes（slots 直接在 object header）
# p2 沒有 __dict__，省下 ~232 bytes

# 大量 instance 時效果顯著（如 10M 個 Point）
# ⚠️ 代價：不能動態增加屬性，繼承需要明確聲明 slots
```

## 相關頁面

- [[Python GIL與並發模型]] — GIL 如何影響引用計數
- [[Python效能調優]] — tracemalloc、memory_profiler 實戰
- [[Go記憶體洩漏排查]] — 對比 Go GC 與 Python GC 的差異
