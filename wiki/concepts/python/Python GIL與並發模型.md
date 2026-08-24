---
title: Python GIL 與並發模型
type: concept
tags: [python, GIL, threading, multiprocessing, asyncio, concurrency, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python GIL 與並發模型

> GIL（Global Interpreter Lock）是 Python 面試最高頻的考點。理解它的本質，才能正確選擇 threading / multiprocessing / asyncio。

## 什麼是 GIL

GIL 是 **CPython 直譯器的一把互斥鎖**，確保任一時刻只有一個 Python thread 在執行 Python bytecode。

```
為什麼 CPython 需要 GIL？

CPython 用引用計數（reference counting）管理記憶體。
如果多個 thread 同時修改 ob_refcnt（引用計數），會有 race condition，
導致物件提早釋放（use-after-free）或記憶體洩漏。

GIL 讓引用計數的修改變成序列化操作，從而保證記憶體安全。
代價：Python thread 無法真正並行執行 Python 代碼。
```

## GIL 的實際影響

```python
import threading
import time

# ❌ CPU-bound 任務：多 thread 不會加速
def count_up(n):
    total = 0
    for _ in range(n):
        total += 1
    return total

# 單 thread
start = time.time()
count_up(50_000_000)
count_up(50_000_000)
print(f"single: {time.time() - start:.2f}s")  # ~4.0s

# 兩個 thread（因 GIL，和單 thread 一樣慢甚至更慢）
start = time.time()
t1 = threading.Thread(target=count_up, args=(50_000_000,))
t2 = threading.Thread(target=count_up, args=(50_000_000,))
t1.start(); t2.start()
t1.join(); t2.join()
print(f"2 threads: {time.time() - start:.2f}s")  # ~4.5s（略慢，context switch 開銷）

# ✅ I/O-bound 任務：多 thread 有效（I/O 時會釋放 GIL）
def fetch_url(url):
    import urllib.request
    urllib.request.urlopen(url)  # I/O 等待期間 GIL 釋放，其他 thread 可以跑

# fetch 10 個 URL：multi-thread 快 ~10x
```

**GIL 的釋放時機**：
- 執行 I/O 操作（網路、磁碟）
- 呼叫 C extension（如 numpy 的計算，內部自行釋放 GIL）
- 每執行一定 bytecode 數量（`sys.getswitchinterval()`，預設 5ms）

## 三種並發模型比較

```python
# ┌────────────────┬───────────────────┬──────────────────┬─────────────────┐
# │                │ threading         │ multiprocessing  │ asyncio         │
# ├────────────────┼───────────────────┼──────────────────┼─────────────────┤
# │ 適合場景       │ I/O-bound         │ CPU-bound        │ I/O-bound（大量）│
# │ 受 GIL 限制    │ 是（CPU 無法並行）│ 否（獨立進程）   │ 否（單線程）    │
# │ 記憶體         │ 共享（省記憶體）  │ 各自獨立（較多） │ 共享            │
# │ 通訊           │ 直接（但需鎖）    │ Queue/Pipe       │ 直接（協程間）  │
# │ 開銷           │ 低               │ 高（fork/spawn） │ 極低            │
# │ 除錯難度       │ 高（race condition）│ 中              │ 中              │
# └────────────────┴───────────────────┴──────────────────┴─────────────────┘
```

## threading 模組

```python
import threading
from concurrent.futures import ThreadPoolExecutor

# 基本用法
lock = threading.Lock()
shared = []

def producer(lock, shared):
    for i in range(5):
        with lock:             # Context manager 自動 acquire/release
            shared.append(i)

# ThreadPoolExecutor（推薦，比直接用 Thread 更方便）
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(fetch_url, url) for url in urls]
    results = [f.result() for f in futures]

# threading.Event：goroutine 的 channel done 替代品
stop_event = threading.Event()

def background_worker():
    while not stop_event.is_set():
        do_work()
        stop_event.wait(timeout=1.0)  # 等 1 秒或直到 stop

worker = threading.Thread(target=background_worker, daemon=True)
worker.start()
# ... 需要停止時
stop_event.set()
worker.join()
```

## multiprocessing 模組

```python
from multiprocessing import Pool, Process, Queue
from concurrent.futures import ProcessPoolExecutor

# CPU-bound 並行（真正利用多核）
def cpu_intensive(data):
    return sum(x ** 2 for x in data)

# 方法 1：Pool（最常用）
with Pool(processes=4) as pool:
    results = pool.map(cpu_intensive, [large_data_1, large_data_2, ...])

# 方法 2：ProcessPoolExecutor（和 ThreadPoolExecutor API 一致）
with ProcessPoolExecutor(max_workers=4) as executor:
    futures = {executor.submit(cpu_intensive, data): data for data in dataset}
    for future in concurrent.futures.as_completed(futures):
        result = future.result()

# 進程間通訊：Queue
q = Queue()
def worker(q, data):
    result = process(data)
    q.put(result)

p = Process(target=worker, args=(q, data))
p.start()
p.join()
result = q.get()

# ⚠️ 注意：multiprocessing 的 pickle 限制
# 傳入/傳出的物件必須可以 pickle
# Lambda、本地函數、未 pickle 的 socket/file 會失敗
```

## GIL 的未來：PEP 703（No-GIL Python）

Python 3.13（2024）正式引入「自由執行緒（free-threaded）」模式：

```bash
# 安裝支援 no-GIL 的 Python（CPython 3.13t）
# 在 3.13+ 可選擇 GIL-free 模式：
PYTHON_GIL=0 python3.13 my_script.py

# 或在代碼中：
import sys
print(sys._is_gil_enabled())  # 檢查 GIL 狀態
```

```python
# No-GIL 模式下：CPU-bound 多 thread 真正並行
# 但：現有的 C extension 可能不是 thread-safe
# 目前（3.13）作為實驗功能，主流生產環境尚未採用
```

**面試常問**：「No-GIL 之後是否就用 threading 做 CPU 並行？」

答：No-GIL 是趨勢，但 3.13 仍是實驗性，且很多 C extension 還不支援。現在仍應視任務選擇：CPU-bound 用 multiprocessing，I/O-bound 用 asyncio 或 threading。

---

## 常見面試題

**Q：為什麼 threading 對 I/O-bound 有效？**

A：執行 I/O syscall 時，OS 讓 thread 等待；CPython 在此時釋放 GIL，讓其他 thread 跑 Python 代碼。所以 10 個 thread 同時等待 I/O，實際上是並發的。

**Q：如果一個 C extension 不釋放 GIL，會怎樣？**

A：其他 thread 會被完全阻塞，直到該 C 代碼返回。好的 C extension（如 numpy）在計算時手動釋放 GIL（`Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS`）。

**Q：threading.Lock vs multiprocessing.Lock 的差異？**

A：`threading.Lock` 在同一進程的 thread 間共用，存在同一記憶體空間。`multiprocessing.Lock` 透過 OS 的命名 semaphore 實作，跨進程有效但開銷更大。

## 相關頁面

- [[Python非同步程式設計]] — asyncio event loop、coroutine、Task
- [[Python記憶體管理]] — 引用計數與 GC 的細節
- [[Python效能調優]] — 從 GIL 角度選擇優化方向
