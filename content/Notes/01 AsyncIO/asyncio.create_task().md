---
title: "asyncio.create_task() in Python"
description: "Detailed guide to scheduling coroutines with asyncio.create_task, task naming, exception handling, and GC reference safety."
tags:
  - python/asyncio
  - task
aliases:
  - asyncio.create_task()
  - create_task
---

# `asyncio.create_task()` in Python

> [!summary]
> `asyncio.create_task()` wraps a **[[Notes/01 AsyncIO/Coroutine|Coroutine]]** into a **[[Notes/01 AsyncIO/Tasks|Task]]** and immediately schedules its background execution on the **[[Notes/01 AsyncIO/Event Loop|Event Loop]]**.
>
> It enables concurrent non-blocking execution without pausing the calling coroutine immediately.

---

## Execution Flow

Calling `create_task()` submits the coroutine to the event loop's **Ready Queue**:

```python
async def main():
    # Schedules job() to run concurrently in background
    task = asyncio.create_task(job(), name="BackgroundWorker")

    # main() continues running without waiting for job()!
    print("Main continues immediately...")

    # Await when result is needed
    result = await task
```

```
main() Execution:
create_task(job()) ──> Submits job() to Event Loop ──> main() continues immediately
                                                                │
                                                                ▼
                                                        `await task` yields
                                                                │
                                                                ▼
                                                        Result returned
```

---

## The Garbage Collection (GC) Bug

> [!danger]
> **Critical Requirement:** Maintain a strong reference to any `Task` returned by `create_task()`.
>
> The event loop holds only **weak references** to tasks. If you create a task without storing a reference in a variable or collection, Python's garbage collector may delete the task mid-execution!

### Dangerous (Un-referenced background task):
```python
async def main():
    # Bug! GC may destroy this task while it sleeps!
    asyncio.create_task(send_analytics_ping())
```

### Safe (Strong reference set):
```python
# Set for storing active background task references
active_tasks = set()

def fire_and_forget(coro):
    task = asyncio.create_task(coro)
    active_tasks.add(task)
    # Remove from set when complete to allow natural GC cleanup
    task.add_done_callback(active_tasks.discard)
```

---

## Named Tasks & Error Handling

### 1. Naming Tasks (Python 3.8+)
Assign human-readable names to tasks for clean debugging and tracebacks:

```python
task = asyncio.create_task(fetch_user(42), name="FetchUser-42")
print(f"Executing task: {task.get_name()}")
```

### 2. Handling Task Exceptions
If a background task raises an exception and is never awaited, the event loop logs an unhandled exception warning when destroyed. Handle exceptions explicitly:

```python
async def safe_worker():
    task = asyncio.create_task(unstable_api_call())
    try:
        await task
    except Exception as exc:
        print(f"Task failed with error: {exc}")
```

---

## When to Use `create_task()` vs `await`

| Operation | Use `await coro()` | Use `asyncio.create_task(coro())` |
| :--- | :--- | :--- |
| **Execution Order** | Sequential (Pauses caller until done) | Concurrent (Fires immediately in background) |
| **Use Case** | Dependent data operations | Independent background jobs, analytics, notifications |

---

## Related Notes
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Deep dive into task lifecycle and states
- [[Notes/01 AsyncIO/Coroutine|Coroutine]] — The underlying pausable function
- [[Notes/01 AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]] — Structured alternative for managing groups of tasks
- [[Notes/01 AsyncIO/index|AsyncIO Map of Content]]