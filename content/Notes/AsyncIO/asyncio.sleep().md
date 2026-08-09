---
title: "asyncio.sleep() in Python"
description: "Understanding asyncio.sleep(), non-blocking timers, cooperative yielding with sleep(0), and avoiding thread-blocking time.sleep()."
tags:
  - python/asyncio
  - timer
aliases:
  - asyncio.sleep()
  - sleep
---

# ⚡ `asyncio.sleep()` in Python

> [!summary]
> `asyncio.sleep(delay, result=None)` is a non-blocking coroutine that delays execution for a specified number of seconds while yielding control back to the **[[Notes/AsyncIO/Event Loop|Event Loop]]**.
>
> Unlike `time.sleep()`, which freezes the entire thread, `asyncio.sleep()` allows the event loop to execute other ready coroutines during the wait period.

---

## `time.sleep()` vs `asyncio.sleep()`

```python
# INCORRECT: Blocks the entire OS thread!
def bad_delay():
    time.sleep(2)  # Event Loop is frozen! No other task can run.

# CORRECT: Non-blocking cooperative delay!
async def good_delay():
    await asyncio.sleep(2)  # Event Loop immediately runs other tasks.
```

```
time.sleep(2):     Thread Frozen ───────────────────────> No tasks run
asyncio.sleep(2):  Task Paused ──> Event Loop runs B & C ──> Task Resumes
```

---

## Cooperative Yielding: `await asyncio.sleep(0)`

Passing a delay of `0` to `asyncio.sleep()` serves a special architectural purpose in `asyncio`:

> [!tip]
> `await asyncio.sleep(0)` acts as an **explicit cooperative yield point**. It places the current coroutine at the back of the event loop's **Ready Queue**, allowing other pending tasks of equal priority to run immediately before resuming.

```python
import asyncio

async def tight_cpu_loop():
    for i in range(1000):
        # Do work
        if i % 100 == 0:
            # Yield control back to event loop to let UI/web tasks process
            await asyncio.sleep(0)
```

---

## Return Values & High-Resolution Timers

### 1. Returning Results on Delay Completion

You can pass an optional `result` argument to `asyncio.sleep()`:

```python
data = await asyncio.sleep(1.5, result={"status": "cache_expired"})
print(data)  # {"status": "cache_expired"}
```

### 2. Timer Precision

`asyncio.sleep()` uses the OS high-resolution monotonic timer (`time.monotonic()`). However, timer resolution is subject to OS scheduling granularity and event loop workload.

---

## Related Notes

- [[Notes/AsyncIO/Event Loop|Event Loop]] — Manages timer callbacks and ready queue
- [[Notes/AsyncIO/Coroutine|Coroutine]] — Pauses execution during sleep
- [[Notes/AsyncIO/Non-blocking IO|Non-blocking I/O]] — Non-blocking systems foundation
- [[Notes/AsyncIO/index|AsyncIO Map of Content]]
