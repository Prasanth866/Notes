---
title: "Futures in Python AsyncIO"
description: "Detailed breakdown of asyncio.Future, state machine transitions, callbacks, thread safety, and relation to Tasks."
tags:
  - python/asyncio
  - future
aliases:
  - Future
  - Futures
---

# Futures in Python AsyncIO

> [!summary]
> An `asyncio.Future` is a **low-level awaitable object** that acts as a **placeholder for a result that will become available in the future**.
>
> A Future itself performs no execution logic; it serves purely as a state container bridging a result producer and a result consumer.

---

## Conceptual Analogy

Imagine receiving a claim receipt at a dry cleaner:

```
                  Order Placed
                       │
                       ▼
              Receipt Issued (Future)
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
      Cleaner washes       Customer waits
      (Producer)           (Consumer: `await future`)
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
             Receipt Claimed (Result)
```

The claim receipt is not the cleaned clothes. It is a promise that clean clothes (or an exception notice) will be available later.

---

## Future Lifecycle & State Machine

A Future progresses through three distinct states:

```
                   create_future()
                          │
                          ▼
                       PENDING
                  ┌───────┴───────┐
    set_result()  │               │  set_exception()
                  ▼               ▼
              FINISHED        FINISHED (Failed)
                  ▲
   cancel()       │
  ────────────────┘ (CANCELLED)
```

1. **PENDING**: The result is not yet available. Awaiting a pending Future pauses the calling coroutine.
2. **FINISHED**: The producer has invoked `.set_result(val)` or `.set_exception(exc)`. Awaiting resumes immediately.
3. **CANCELLED**: The Future was cancelled via `.cancel()`. Awaiting raises `asyncio.CancelledError`.

---

## Manual Future Manipulation

While high-level application code typically uses [[Notes/01 AsyncIO/Tasks|Tasks]], library authors use Futures to bridge async and callback-based code:

```python
import asyncio

async def async_producer(fut: asyncio.Future):
    await asyncio.sleep(1)
    # Complete the Future with a result
    fut.set_result("Data loaded successfully!")

async def main():
    loop = asyncio.get_running_loop()
    # Create an empty, pending Future
    fut: asyncio.Future[str] = loop.create_future()

    print("Future status:", fut.done())  # False

    # Schedule producer
    loop.create_task(async_producer(fut))

    # Wait for completion
    result = await fut
    print("Result received:", result)
    print("Future status after await:", fut.done())  # True

if __name__ == "__main__":
    asyncio.run(main())
```

---

## ⚡ Callbacks & Thread Safety

### 1. Registering Done Callbacks
You can attach callbacks to trigger immediately when a Future completes:

```python
def on_complete(fut: asyncio.Future):
    print("Callback fired! Result:", fut.result())

fut.add_done_callback(on_complete)
```

### 2. Thread-Safe Completion (`call_soon_threadsafe`)
When completing a Future from a background OS thread, use `loop.call_soon_threadsafe()`:

```python
import threading

def background_thread_worker(loop: asyncio.AbstractEventLoop, fut: asyncio.Future):
    # Perform heavy blocking thread work
    data = "Result from OS Thread"
    # Thread-safe result assignment
    loop.call_soon_threadsafe(fut.set_result, data)
```

---

## Future vs Task

| Feature | `asyncio.Future` | `asyncio.Task` |
| :--- | :--- | :--- |
| **Inheritance** | Base class | Subclass of `asyncio.Future` |
| **Execution** | Performs NO execution | Wraps and executes a [[Notes/01 AsyncIO/Coroutine\|Coroutine]] |
| **Instantiation** | `loop.create_future()` | `asyncio.create_task(coro)` |
| **Purpose** | Passive result placeholder | Active concurrent execution wrapper |

---

## Related Notes
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Active sub-class of Future executing coroutines
- [[Notes/01 AsyncIO/Coroutine|Coroutine]] — Pausable functions that produce results for Futures
- [[Notes/01 AsyncIO/Event Loop|Event Loop]] — Manages Future resolution and callbacks
- [[Notes/01 AsyncIO/index|AsyncIO Map of Content]]