---
title: "asyncio.gather() in Python"
description: "Detailed breakdown of asyncio.gather, batch concurrent awaitable execution, error handling with return_exceptions, and comparison with TaskGroup."
tags:
  - python/asyncio
  - concurrency
aliases:
  - asyncio.gather()
  - gather
---

# `asyncio.gather()` in Python

> [!summary]
> `asyncio.gather(*aws, return_exceptions=False)` takes multiple **awaitable objects** (coroutines, tasks, futures), runs them concurrently on the **[[Notes/01 AsyncIO/Event Loop|Event Loop]]**, and returns a list of their aggregated results in the exact order passed.

---

## How `asyncio.gather()` Works

```python
results = await asyncio.gather(
    fetch_user(1),    # 2 seconds
    fetch_orders(1),  # 1 second
    fetch_avatar(1)   # 3 seconds
)
# Overall time elapsed ≈ 3 seconds (Longest task)
# results -> [user_data, orders_data, avatar_data]
```

```
fetch_user(1)    ─────── (2s) ───────┐
fetch_orders(1)  ── (1s) ──┐         │
fetch_avatar(1)  ──────────┼─ (3s) ──┴──> All Complete ──> Return Ordered List
```

Even though `fetch_orders` completes first, `gather()` preserves the original positional order of the inputs in the returned list.

---

## Exception Handling & `return_exceptions`

The `return_exceptions` boolean parameter completely changes how `gather()` handles runtime errors.

### 1. `return_exceptions=False` (Default)

The first exception raised by any awaitable immediately bubbles up to the caller. However, **other pending tasks are NOT automatically cancelled** and continue running in the background!

```python
import asyncio

async def bad_task():
    await asyncio.sleep(0.5)
    raise ValueError("Database connection failed!")

async def slow_task():
    await asyncio.sleep(2)
    print("Slow task finished!")

async def main():
    try:
        results = await asyncio.gather(bad_task(), slow_task(), return_exceptions=False)
    except ValueError as exc:
        print("Gather caught error:", exc)

if __name__ == "__main__":
    asyncio.run(main())
```

> [!warning]
> With `return_exceptions=False`, an error does not cancel other tasks. If you require automatic sibling cancellation on failure, prefer **[[Notes/01 AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]]**.

### 2. `return_exceptions=True` (Safe Result Aggregation)

Exceptions are treated as valid return values and placed directly into the results list at their corresponding index:

```python
async def main():
    results = await asyncio.gather(
        bad_task(),
        slow_task(),
        return_exceptions=True
    )
    # results -> [ValueError('Database connection failed!'), None]
    for res in results:
        if isinstance(res, Exception):
            print("Captured exception:", res)
        else:
            print("Success result:", res)
```

---

## `asyncio.gather()` vs `asyncio.TaskGroup()`

| Feature                  | `asyncio.gather()`              | `asyncio.TaskGroup()`                        |
| :----------------------- | :------------------------------ | :------------------------------------------- |
| **Python Compatibility** | Python 3.4+                     | Python 3.11+                                 |
| **Sibling Cancellation** | No (Pending tasks keep running) | Yes (Automatic cancellation of all siblings) |
| **Return Format**        | Ordered `list` of results       | Task objects via `task.result()`             |
| **Exception Handling**   | `return_exceptions=True/False`  | `ExceptionGroup` via `except*`               |

---

## Related Notes

- [[Notes/01 AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]] — Python 3.11+ structured concurrency alternative
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Concurrent execution objects
- [[Notes/01 AsyncIO/Coroutine|Coroutine]] — Pausable code executed by gather
- [[Notes/01 AsyncIO/index|AsyncIO Map of Content]]
