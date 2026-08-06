---
title: "Coroutines in Python AsyncIO"
description: "Detailed guide on Python coroutines, async/await mechanics, execution lifecycle, generator roots, and suspension points."
tags:
  - python/asyncio
  - coroutine
aliases:
  - Coroutine
  - Coroutines
---

# Coroutines in Python AsyncIO

> [!summary]
> A **coroutine** is a specialized, pausable function that can suspend its execution at explicit cooperative yield points (`await`) and resume execution later without blocking the OS thread.
>
> Coroutines are the fundamental building blocks of asynchronous programming in Python.

---

## Core Concepts

### 1. Definition & Declaration

Coroutines are declared using the `async def` keyword syntax:

```python
async def fetch_data(url: str) -> dict:
    # Performs asynchronous work
    return {"status": "ok"}
```

### 2. Coroutine Function vs Coroutine Object

Calling a coroutine function **does not execute its body immediately**. Instead, it instantiates and returns a **coroutine object**:

```python
async def greet():
    print("Hello")

# greet      -> Coroutine Function (<function greet at ...>)
# greet()    -> Coroutine Object (<coroutine object greet at ...>)

coro = greet()  # Nothing executes yet!
print(type(coro)) # <class 'coroutine'>
```

> [!warning]
> A common beginner bug is invoking `fetch_data()` without `await` or `asyncio.create_task()`. This triggers a runtime warning: `RuntimeWarning: Enable tracemalloc to get the object allocation traceback`.

---

## Execution & Suspension Lifecycle

Coroutines can only execute when scheduled on an active **[[Notes/01 AsyncIO/Event Loop|Event Loop]]**.

```
  async def worker()
          │
          ▼
   Coroutine Object
          │
          ├───────────────────────────────┐
          ▼                               ▼
    await worker()               asyncio.create_task()
  (Cooperative Yield)           (Scheduled on Event Loop)
          │                               │
          └───────────────┬───────────────┘
                          │
                          ▼
               Executed by Event Loop
```

### Methods of Execution

1. **Top-Level Entry**: `asyncio.run(main())` creates an event loop, runs `main()`, and closes the loop.
2. **Cooperative Awaiting**: `await coro` suspends the calling coroutine until `coro` finishes.
3. **Concurrent Scheduling**: `asyncio.create_task(coro)` wraps `coro` into a **[[Notes/01 AsyncIO/Tasks|Task]]** for background loop execution.

---

## What Happens at `await`?

When Python encounters `await awaitable`:

```python
async def process():
    print("Step 1: Starting")
    await asyncio.sleep(2)  # Yields control back to the Event Loop
    print("Step 2: Resumed")
```

1. `process()` pauses immediately at `await`.
2. It saves its local variable state and stack pointer.
3. Control returns to the **[[Notes/01 AsyncIO/Event Loop|Event Loop]]**.
4. The event loop executes other ready tasks during the 2-second delay.
5. Once the timer elapses, the event loop marks `process()` as **Ready** and resumes execution at Step 2.

---

## What Can Be Awaitable?

The `await` keyword only accepts **awaitable objects**, which implement the `__await__()` magic method:

| Awaitable Type        | Purpose                                 | Reference Note                            |
| :-------------------- | :-------------------------------------- | :---------------------------------------- |
| **Coroutine Objects** | Returned by `async def` function calls  | [[Notes/01 AsyncIO/Coroutine\|Coroutine]] |
| **Tasks**             | Event-loop scheduled coroutine wrappers | [[Notes/01 AsyncIO/Tasks\|Tasks]]         |
| **Futures**           | Low-level result placeholders           | [[Notes/01 AsyncIO/Future\|Future]]       |

---

## Practical Example

```python
import asyncio
import time

async def fetch_user(user_id: int) -> dict:
    print(f"[{time.strftime('%X')}] Fetching user {user_id}...")
    await asyncio.sleep(1)  # Non-blocking IO delay
    print(f"[{time.strftime('%X')}] User {user_id} fetched.")
    return {"id": user_id, "name": f"User_{user_id}"}

async def main():
    start = time.time()
    # Execute sequential coroutine awaits
    u1 = await fetch_user(1)
    u2 = await fetch_user(2)
    print(f"Total time elapsed: {time.time() - start:.2f}s")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Mental Model

> **Think of a coroutine as a pausable state machine.**
> It does not run continuously like a standard synchronous function. Instead, it yields control at every `await` boundary, allowing the **[[Notes/01 AsyncIO/Event Loop|Event Loop]]** to multiplex thousands of concurrent tasks on a single OS thread.

---

## Related Notes

- [[Notes/01 AsyncIO/Event Loop|Event Loop]] — The scheduler behind coroutine execution
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Wrapping coroutines into concurrent tasks
- [[Notes/01 AsyncIO/Future|Futures]] — Low-level result promises
- [[Notes/01 AsyncIO/index|AsyncIO Map of Content]]
