---
title: Python 資料模型與描述器
type: concept
tags: [python, data-model, descriptor, metaclass, dunder, property, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 資料模型與描述器

## Python 資料模型（Data Model）

Python 的「魔術方法」（dunder methods）定義了物件如何與語言內建操作互動。理解這層協定是 Python 高級開發的核心。

### 物件生命週期

```python
class MyClass:
    # __new__：分配記憶體，建立 instance（類方法）
    def __new__(cls, *args, **kwargs):
        print(f"__new__ called for {cls}")
        instance = super().__new__(cls)  # 分配記憶體
        return instance

    # __init__：初始化 instance（在 __new__ 之後）
    def __init__(self, value):
        print(f"__init__ called")
        self.value = value

    # __del__：instance 被回收時（不保證何時呼叫）
    def __del__(self):
        print(f"__del__ called")

# 執行順序：__new__ → __init__ → 使用 → __del__

# 使用 __new__ 的實際場景：Singleton、不可變型別繼承
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# 繼承 int（不可變型別必須在 __new__ 中設定值）
class PositiveInt(int):
    def __new__(cls, value):
        if value <= 0:
            raise ValueError(f"must be positive, got {value}")
        return super().__new__(cls, value)
```

### 字串表示

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        # 開發者用，目標：eval(repr(obj)) == obj
        return f"Vector({self.x!r}, {self.y!r})"

    def __str__(self):
        # 使用者用，print() 和 str() 呼叫
        return f"({self.x}, {self.y})"

    def __format__(self, fmt_spec):
        # 支援 f-string 格式化
        if fmt_spec == 'polar':
            import math
            r = math.hypot(self.x, self.y)
            theta = math.atan2(self.y, self.x)
            return f"|{r:.2f}∠{theta:.2f}|"
        return str(self)

v = Vector(3, 4)
print(repr(v))       # Vector(3, 4)
print(str(v))        # (3, 4)
print(f"{v:polar}")  # |5.00∠0.93|
```

### 運算子重載

```python
class Vector:
    def __init__(self, x, y): self.x, self.y = x, y

    # 算術運算子
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):   # v * 3
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):  # 3 * v（左側不是 Vector 時呼叫）
        return self.__mul__(scalar)

    # 比較運算子（配合 functools.total_ordering 只需實作 __eq__ 和一個比較）
    def __eq__(self, other):
        return isinstance(other, Vector) and self.x == other.x and self.y == other.y

    def __abs__(self):           # abs(v)
        import math
        return math.hypot(self.x, self.y)

    def __bool__(self):          # if v: ...
        return bool(abs(self))

    def __len__(self):           # len(v)
        return 2

    def __getitem__(self, index): # v[0], v[1], for x in v
        return (self.x, self.y)[index]
```

### Context Manager 協定

```python
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn  # with ... as conn 中的 conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            self.conn.rollback()
        else:
            self.conn.commit()
        self.conn.close()
        return False  # False = 不抑制異常（True = 吞掉異常）

# 等同於：
from contextlib import contextmanager

@contextmanager
def database_connection():
    conn = connect_to_db()
    try:
        yield conn   # __enter__ 的返回值
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()
```

---

## 描述器（Descriptor）

描述器是實作了 `__get__`、`__set__`、`__delete__` 之一的物件，放在 class 的 namespace 中，攔截屬性存取。

### 描述器協定

```python
# 非資料描述器（Non-data descriptor）：只有 __get__
# 資料描述器（Data descriptor）：有 __get__ 和 __set__（或 __delete__）
# 優先順序：資料描述器 > instance __dict__ > 非資料描述器

class Validator:
    """資料描述器：驗證並存取數值"""
    def __set_name__(self, owner, name):
        # Python 3.6+：class 定義時自動呼叫，注入屬性名
        self.name = name
        self.storage_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self  # 透過 class 存取時返回描述器本身
        return getattr(obj, self.storage_name, None)

    def __set__(self, obj, value):
        self.validate(value)
        setattr(obj, self.storage_name, value)

    def validate(self, value):
        raise NotImplementedError

class PositiveNumber(Validator):
    def validate(self, value):
        if not isinstance(value, (int, float)) or value <= 0:
            raise ValueError(f"{self.name} must be positive, got {value!r}")

class Circle:
    radius = PositiveNumber()  # 描述器放在 class 層級
    diameter = PositiveNumber()

    def __init__(self, radius):
        self.radius = radius   # 呼叫 PositiveNumber.__set__

c = Circle(5)
c.radius = -1   # raises ValueError: radius must be positive
```

### property：最常用的描述器

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        """getter"""
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @celsius.deleter
    def celsius(self):
        del self._celsius

    @property
    def fahrenheit(self):
        """唯讀 property（只有 getter）"""
        return self._celsius * 9/5 + 32

t = Temperature(25)
print(t.fahrenheit)   # 77.0
t.celsius = -300      # raises ValueError
```

### `__getattr__` vs `__getattribute__`

```python
class FlexObject:
    def __init__(self):
        self.data = {}

    def __getattribute__(self, name):
        # 攔截所有屬性存取（包含存在的屬性）
        # ⚠️ 危險：容易造成無限遞迴
        print(f"Accessing {name}")
        return super().__getattribute__(name)  # 必須呼叫 super()

    def __getattr__(self, name):
        # 只在正常查找失敗後才呼叫（屬性不存在時）
        # 比 __getattribute__ 安全
        if name in self.data:
            return self.data[name]
        raise AttributeError(f"No attribute '{name}'")

    def __setattr__(self, name, value):
        # 攔截所有屬性設定
        if name == 'data':
            super().__setattr__(name, value)  # 允許 data 本身被設定
        else:
            self.data[name] = value  # 其他屬性存到 data dict

obj = FlexObject()
obj.foo = "bar"         # 觸發 __setattr__
print(obj.foo)          # 觸發 __getattr__（因為 foo 不在 instance __dict__）
```

---

## Metaclass（元類別）

Metaclass 是「建立 class 的 class」。`type` 是所有 class 的預設 metaclass。

```python
# 普通 class 的建立過程：
class MyClass:
    x = 10

# 等同於：
MyClass = type('MyClass', (), {'x': 10})

# 自訂 metaclass：在 class 定義時插入行為
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        # 攔截 class 的「呼叫」（即 MyClass(...)）
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = create_connection()

db1 = Database()
db2 = Database()
print(db1 is db2)  # True（同一個 instance）

# 更常見的使用：自動注冊、API 驗證、ORM 欄位掃描
class ModelMeta(type):
    def __new__(mcs, name, bases, namespace):
        fields = {k: v for k, v in namespace.items()
                  if isinstance(v, Field)}
        namespace['_fields'] = fields
        return super().__new__(mcs, name, bases, namespace)

class Model(metaclass=ModelMeta):
    pass

class User(Model):
    name = CharField(max_length=100)
    age = IntField(min_value=0)
    # User._fields = {'name': CharField(...), 'age': IntField(...)}
```

**Metaclass vs Decorator vs `__init_subclass__`**：

```python
# 大多數 metaclass 的用途可以用更簡單的方式實現

# __init_subclass__（Python 3.6+）：子類被定義時呼叫
class PluginBase:
    _plugins = {}

    def __init_subclass__(cls, plugin_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if plugin_name:
            PluginBase._plugins[plugin_name] = cls

class CSVPlugin(PluginBase, plugin_name="csv"):
    pass

class JSONPlugin(PluginBase, plugin_name="json"):
    pass

print(PluginBase._plugins)  # {'csv': CSVPlugin, 'json': JSONPlugin}
```

---

## 相關頁面

- [[Python型別系統與類型提示]] — Protocol 如何用描述器協定定義結構型別
- [[Python效能調優]] — `__slots__` 減少記憶體、描述器的效能影響
- [[Python記憶體管理]] — `__del__` 與 GC 的互動
