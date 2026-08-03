---
title: "Tasks in Python AsyncIO"
description: "Comprehensive guide to asyncio.Task, lifecycle states, task cancellation, shielding, and garbage collection reference safety."
tags:
  - python/asyncio
  - task
aliases:
  - Tasks
  - Task
---

# 📋 Tasks in Python AsyncIO

> [!summary]
> An `asyncio.Task` is a high-level, concurrent object that wraps a **[[Notes/01 AsyncIO/Coroutine|Coroutine]]** and schedules its execution on the **[[Notes/01 AsyncIO/Event Loop|Event Loop]]**.
>
> `Task` is a subclass of **[[Notes/01 AsyncIO/Future|Future]]**, allowing coroutines to run concurrently in the background and yield execution results or exceptions.

---

## ⚙️ Coroutine vs Task

```
                  async def my_func()
                           │
                           ▼
                    Coroutine Object
             (Pausable, but inactive)
                           │
                 asyncio.create_task()
                           │
                           ▼
                      asyncio.Task
         (Scheduled & executing on Event Loop)
```

- A **Coroutine** is inert code until awaited or scheduled.
- A **Task** immediately registers the coroutine with the event loop's ready queue, running it concurrently alongside other tasks.

---

## 🔄 Task Lifecycle States

A task progresses through several operational states managed by the loop:

```
    create_task(coro)
           │
           ▼
        PENDING ──────────────> RUNNING ──────────────> FINISHED (Success)
           │                        │                       ▲
           │                        │                       │
           │                  await / yield                 │
           │                        │                       │
           └──────────────────> WAITING ────────────────────┘
                                    │
                                 cancel()
                                    │
                                    ▼
                                CANCELLED (Raises CancelledError)
```

---

## 🛡️ Task Cancellation & Shielding

### 1. Cancelling a Task
Invoke `.cancel()` to request task cancellation. This injects an `asyncio.CancelledError` at the task's next `await` boundary:

```python
import asyncio

async def worker():
    try:
        print("Worker starting long operation...")
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("Worker caught CancelledError! Cleaning up resources...")
        raise  # Re-raise to finalize cancellation state

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(1)  # Let worker run briefly
    task.cancel()  # Request cancellation
    try:
        await task
    except asyncio.CancelledError:
        print("Main confirmed task was cancelled.")

if __name__ == "__main__":
    asyncio.run(main())
```

> [!important]
> Always re-raise `asyncio.CancelledError` when catching it in a coroutine, unless you explicitly intend to suppress cancellation.

### 2. Shielding Tasks (`asyncio.shield`)
To prevent an outer cancellation request from aborting a critical background operation, wrap it in `asyncio.shield()`:

```python
# The outer caller can be cancelled, but save_db_record will complete!
await asyncio.shield(save_db_record(data))
```

---

## ⚠️ Task Garbage Collection (GC) Bug

> [!danger]
> **Critical Python 3.8+ Gotcha:** The event loop only keeps **weak references** to tasks created by `asyncio.create_task()`.
>
> If you do not retain a strong reference to the returned `Task` object, Python's garbage collector may delete the task mid-execution!

### ❌ Incorrect (Vulnerable to GC destruction):
```python
async def main():
    # Danger! Un-referenced background task may be garbage collected mid-flight
    asyncio.create_task(background_worker())
```

### ✅ Correct (Retaining strong reference):
```python
background_tasks = set()

async def main():
    task = asyncio.create_task(background_worker())
    # Save strong reference to prevent GC collection
    background_tasks.add(task)
    task.add_done_callback(background_tasks.discard)
```

---

## 🔗 Related Notes
- [[Notes/01 AsyncIO/asyncio.create_task()|⚡ asyncio.create_task()]] — Function for creating background tasks
- [[Notes/01 AsyncIO/asyncio.TaskGroup()|⚡ asyncio.TaskGroup()]] — Structured concurrency for task management
- [[Notes/01 AsyncIO/Coroutine|⚡ Coroutine]] — The underlying code wrapped by a Task
- [[Notes/01 AsyncIO/Future|🔮 Future]] — The parent class of Task
- [[Notes/01 AsyncIO/index|⚡ AsyncIO Map of Content]]