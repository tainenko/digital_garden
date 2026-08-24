---
title: Python 非同步程式設計（asyncio）
type: concept
tags: [python, asyncio, coroutine, event-loop, async-await, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Python 非同步程式設計（asyncio）

## Event Loop 的本質

asyncio 的核心是**單 thread 的事件循環**：不等待 I/O，而是在等待時切換執行其他 coroutine。

```
事件循環的工作：
1. 從 ready queue 取出一個 coroutine 執行
2. coroutine 遇到 await（如 await asyncio.sleep()）→ 暫停，控制權歸還給 event loop
3. Event loop 繼續執行其他 ready 的 coroutine
4. 當 I/O 完成（OS 通知），原 coroutine 重新加入 ready queue
5. 重複

相比 threading：coroutine 在明確的 await 點切換（協作式），
threading 由 OS 強制切換（搶佔式）。
協作式的優點：無 race condition（切換點已知），開銷極低（無 context switch）。
```

## 基本語法

```python
import asyncio

# async def 定義 coroutine function
async def fetch_data(url: str) -> str:
    # await 讓出控制權，等待 I/O 完成
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()

# 執行 coroutine
async def main():
    result = await fetch_data("https://api.example.com/data")
    print(result)

asyncio.run(main())  # Python 3.7+，建立 event loop 並執行到完成

# ❌ 常見錯誤：直接呼叫 async function 不會執行
result = fetch_data("https://...")  # 這只是建立了一個 coroutine object！
# ✅ 必須 await 或用 asyncio.run/create_task
```

## 並發執行：asyncio.gather 與 Task

```python
import asyncio
import aiohttp

# 方法 1：asyncio.gather（並發執行，等全部完成）
async def fetch_all(urls: list[str]) -> list[str]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_one(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

# 方法 2：asyncio.create_task（立即排程，不需要立刻 await）
async def main():
    # create_task 立即把 coroutine 排程到 event loop
    task1 = asyncio.create_task(fetch_data(url1))
    task2 = asyncio.create_task(fetch_data(url2))

    # 做其他事情...
    await asyncio.sleep(0)  # 讓 event loop 有機會執行 task

    result1 = await task1
    result2 = await task2

# 方法 3：asyncio.TaskGroup（Python 3.11+，更安全）
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_data(url1))
        task2 = tg.create_task(fetch_data(url2))
    # 離開 with 時等所有 task 完成
    # 任一 task 失敗 → 其他 task 被 cancel，異常被重新拋出
    print(task1.result(), task2.result())
```

## gather vs TaskGroup vs wait

```python
# asyncio.gather：並發，返回結果列表
# - return_exceptions=True：某個 task 失敗不影響其他
results = await asyncio.gather(*tasks, return_exceptions=True)
for r in results:
    if isinstance(r, Exception):
        handle_error(r)

# asyncio.wait：更細粒度的控制
done, pending = await asyncio.wait(
    tasks,
    timeout=5.0,                          # 超時
    return_when=asyncio.FIRST_COMPLETED,  # 有一個完成就返回
)
for task in done:
    print(task.result())
for task in pending:
    task.cancel()  # 取消未完成的

# asyncio.as_completed：哪個先完成就先處理
for coro in asyncio.as_completed(tasks, timeout=10):
    try:
        result = await coro
        process(result)
    except asyncio.TimeoutError:
        pass
```

## 取消（Cancellation）與 Timeout

```python
# asyncio.timeout（Python 3.11+）
async def main():
    try:
        async with asyncio.timeout(5.0):
            result = await slow_operation()
    except TimeoutError:
        print("timed out")

# 舊版：asyncio.wait_for
try:
    result = await asyncio.wait_for(slow_operation(), timeout=5.0)
except asyncio.TimeoutError:
    print("timed out")

# Task 取消
task = asyncio.create_task(long_running())
await asyncio.sleep(2)
task.cancel()  # 發送 CancelledError 到 coroutine 的當前 await 點

# 在 coroutine 內處理取消
async def graceful_task():
    try:
        await do_work()
    except asyncio.CancelledError:
        await cleanup()  # 在取消時做清理
        raise  # 必須重新拋出，讓 task 正確標記為 cancelled
```

## asyncio.Queue（Coroutine 間通訊）

```python
async def producer(queue: asyncio.Queue):
    for item in data_source:
        await queue.put(item)
    await queue.put(None)  # sentinel

async def consumer(queue: asyncio.Queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        process(item)
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=100)  # 有界 buffer
    await asyncio.gather(
        producer(queue),
        consumer(queue),
        consumer(queue),  # 多個 consumer
    )
```

## 同步原語

```python
import asyncio

# asyncio.Lock（不可在同一 coroutine 重入）
lock = asyncio.Lock()
async def critical_section():
    async with lock:
        await modify_shared_state()

# asyncio.Semaphore（限制並發數）
sem = asyncio.Semaphore(10)  # 最多 10 個並發

async def limited_fetch(url):
    async with sem:
        return await fetch(url)

# asyncio.Event（等待某個條件）
ready = asyncio.Event()

async def waiter():
    await ready.wait()  # 阻塞直到 ready.set()
    print("ready!")

async def setter():
    await asyncio.sleep(1)
    ready.set()
```

## 與同步代碼整合

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

# 在 async 代碼中執行阻塞的同步函數
async def main():
    loop = asyncio.get_event_loop()

    # run_in_executor：把同步函數放到 thread pool 執行，不阻塞 event loop
    result = await loop.run_in_executor(
        None,            # None = 預設 ThreadPoolExecutor
        blocking_func,   # 同步函數
        arg1, arg2       # 參數
    )

    # 或用自訂 executor（CPU-bound 用 ProcessPoolExecutor）
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_intensive, data)

# asyncio.to_thread（Python 3.9+，更簡潔）
result = await asyncio.to_thread(blocking_func, arg1, arg2)
```

## 常見陷阱

```python
# ❌ 陷阱 1：在 coroutine 中呼叫阻塞函數
async def bad():
    time.sleep(1)           # ← 阻塞整個 event loop！所有 coroutine 都停住
    requests.get(url)       # ← 同樣問題

# ✅ 修法：用 asyncio 版本的函數，或 run_in_executor
async def good():
    await asyncio.sleep(1)
    async with aiohttp.ClientSession() as s:
        await s.get(url)

# ❌ 陷阱 2：忘記 await
async def bad():
    result = fetch_data(url)   # coroutine object，沒有執行！
    return result

# ❌ 陷阱 3：在非 async 函數中 await
def sync_func():
    result = await coroutine()  # SyntaxError

# ❌ 陷阱 4：多個 event loop
asyncio.run(main())
asyncio.run(main())  # 第二個 run 可以，但不能巢狀
# → 在 Jupyter 等已有 event loop 的環境用 nest_asyncio 或 asyncio.get_event_loop().run_until_complete()

# ❌ 陷阱 5：未 await 的 Task 被 GC 回收
async def main():
    asyncio.create_task(background())  # 沒有保存 task 引用
    # task 可能在 main 結束前被 GC，Python 3.12+ 會發出警告

# ✅ 保存引用
background_tasks = set()
task = asyncio.create_task(background())
background_tasks.add(task)
task.add_done_callback(background_tasks.discard)
```

## 效能特性

```python
# asyncio 的效能優勢：
# - 單 thread，無 race condition，無 mutex 開銷
# - 切換開銷極低（coroutine switch ≈ 函數調用）
# - 適合大量 I/O 並發（數千個並發連線）

# asyncio vs threading 的 I/O 並發比較：
# threading：10,000 thread → 10,000 * 8KB stack ≈ 80MB，OS scheduling 開銷大
# asyncio：10,000 coroutine → 每個 coroutine frame 幾百 bytes，無 OS 介入

# 實際測試：10,000 個 HTTP 請求
# threading: ~5s, 高 CPU（context switch）
# asyncio + aiohttp: ~1s, 低 CPU
```

## 相關頁面

- [[Python GIL與並發模型]] — threading / multiprocessing / asyncio 的選擇
- [[Python效能調優]] — asyncio 的 profiling 與瓶頸分析
