---
title: "asyncio.TaskGroup() in Python 3.11+"
description: "Detailed guide to structured concurrency with asyncio.TaskGroup, ExceptionGroup exception handling, and comparison with asyncio.gather."
tags:
  - python/asyncio
  - taskgroup
  - structured-concurrency
aliases:
  - asyncio.TaskGroup()
  - TaskGroup
---

# ⚡ `asyncio.TaskGroup()` in Python 3.11+

> [!summary]
> `asyncio.TaskGroup` is an asynchronous context manager introduced in **Python 3.11** that implements **Structured Concurrency**.
>
> It ensures that a group of concurrent tasks complete together. If any task within the group raises an unhandled exception, `TaskGroup` automatically cancels all remaining running sibling tasks.

---

## 🏛️ What is Structured Concurrency?

In traditional unstructured concurrency (like un-managed `create_task()` calls), tasks can outlive the scope that spawned them, causing resource leaks and silent unhandled exceptions.

Structured Concurrency enforces a strict control boundary:

```
                      Enter TaskGroup Block
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
           tg.create_task(A)             tg.create_task(B)
                 │                             │
                 └──────────────┬──────────────┘
                                │
                 Wait for ALL tasks to finish
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
       No Exceptions?                Any Task Fails?
       Exit cleanly.                 Cancel all siblings &
                                     raise ExceptionGroup.
```

---

## 💻 Production TaskGroup Pattern

```python
import asyncio

async def fetch_api(service: str, delay: int) -> str:
    await asyncio.sleep(delay)
    if service == "Auth":
        raise ValueError("Authentication service timeout!")
    return f"Response from {service}"

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            # Spawn concurrent tasks within the group
            t1 = tg.create_task(fetch_api("Database", 1))
            t2 = tg.create_task(fetch_api("Auth", 2))       # Raises exception!
            t3 = tg.create_task(fetch_api("Payments", 3))   # Will be cancelled automatically!

        print("Results:", t1.result(), t3.result())
    except ExceptionGroup as eg:
        print("TaskGroup caught ExceptionGroup!")
        for exc in eg.exceptions:
            print(f" - Sub-exception: {type(exc).__name__}: {exc}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## ⚡ Handling `ExceptionGroup` (`except*`)

Python 3.11 introduced `except*` syntax specifically for matching exception types inside an `ExceptionGroup`:

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task_a())
        tg.create_task(task_b())
except* ValueError as eg:
    print(f"Handled ValueErrors: {eg.exceptions}")
except* KeyError as eg:
    print(f"Handled KeyErrors: {eg.exceptions}")
```

---

## ⚖️ `asyncio.TaskGroup()` vs `asyncio.gather()`

| Feature | `asyncio.TaskGroup()` | `asyncio.gather()` |
| :--- | :--- | :--- |
| **Python Version** | Python 3.11+ | Python 3.4+ |
| **Concurrency Paradigm** | Structured Concurrency | Unstructured / Flexible |
| **Failure Behavior** | Cancels ALL sibling tasks on first error | Continues or returns results based on `return_exceptions` |
| **Exception Type** | Raises `ExceptionGroup` | Raises first exception or returns exceptions list |
| **GC Reference Safety** | Built-in context manager protection | Manual reference management required |

---

## 🔗 Related Notes
- [[Notes/01 AsyncIO/asyncio.gather()|⚡ asyncio.gather()]] — Legacy/Flexible concurrent execution
- [[Notes/01 AsyncIO/Tasks|📋 Tasks]] — Underlying task execution objects
- [[Notes/01 AsyncIO/Coroutine|⚡ Coroutine]] — Pausable code executed within task groups
- [[Notes/01 AsyncIO/index|⚡ AsyncIO Map of Content]]
