---
title: "asyncio.Queue() in Python"
description: "Detailed guide on asyncio.Queue, producer-consumer patterns, managing backpressure, PriorityQueue, and graceful worker shutdown."
tags:
  - python/asyncio
  - concurrency
  - queue
aliases:
  - asyncio.Queue()
  - Queue
---

# `asyncio.Queue()` in Python

> [!summary]
> `asyncio.Queue` is a FIFO (First-In, First-Out) synchronization primitive designed for transferring data between **[[Notes/AsyncIO/Coroutine|Coroutines]]** running on the same **[[Notes/AsyncIO/Event Loop|Event Loop]]**.
>
> It provides thread-safe coroutine methods (`put()` and `get()`) that pause execution when the queue is full or empty, managing system backpressure naturally.

---

## Producer-Consumer Architecture

```
                    ┌─────────────────────────┐
                    │    Producer Coroutine   │
                    └────────────┬────────────┘
                                 │
                            await put()
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │    asyncio.Queue        │
                    │   (maxsize Bounded)     │
                    └────────────┬────────────┘
                                 │
                            await get()
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │    Consumer Coroutine   │
                    └────────────┬────────────┘
                                 │
                         task_done() & join()
```

- **`await queue.put(item)`**: Suspends the producer if `queue.qsize() == maxsize` (Backpressure Control).
- **`await queue.get()`**: Suspends the consumer if `queue.empty()`.
- **`queue.task_done()`**: Signals that an extracted item was fully processed.
- **`await queue.join()`**: Blocks until every enqueued item has been marked completed via `task_done()`.

---

## Complete Producer-Consumer Example

```python
import asyncio
import random

async def producer(name: str, queue: asyncio.Queue[int], total_items: int):
    for i in range(total_items):
        item = random.randint(1, 100)
        await queue.put(item)  # Suspends if queue is full
        print(f"[{name}] Produced item {item} (Queue size: {queue.qsize()})")
        await asyncio.sleep(0.1)

async def consumer(name: str, queue: asyncio.Queue[int]):
    while True:
        item = await queue.get()  # Suspends if queue is empty
        try:
            print(f"  [{name}] Consumed item {item}")
            await asyncio.sleep(0.3)  # Simulate processing work
        finally:
            queue.task_done()  # Signal completion for queue.join()

async def main():
    # Bounded queue with maxsize=3 to enforce backpressure
    queue: asyncio.Queue[int] = asyncio.Queue(maxsize=3)

    # Launch consumers
    consumers = [
        asyncio.create_task(consumer(f"Worker-{i}", queue))
        for i in range(2)
    ]

    # Run producer to completion
    await producer("Producer-1", queue, total_items=6)

    # Wait for all queue items to be processed
    await queue.join()

    # Cancel background consumer tasks
    for c in consumers:
        c.cancel()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Managing Backpressure & Sizing

> [!tip]
> Always specify `maxsize` in production queues (`asyncio.Queue(maxsize=100)`). Unbounded queues (`maxsize=0`) will consume unlimited RAM if producers generate data faster than consumers process it.

| Queue Method | Behavior when Full/Empty          | Non-blocking Alternative                |
| :----------- | :-------------------------------- | :-------------------------------------- |
| `put(item)`  | Suspends until space is available | `put_nowait(item)` (Raises `QueueFull`) |
| `get()`      | Suspends until item arrives       | `get_nowait()` (Raises `QueueEmpty`)    |

---

## Queue Variants in `asyncio`

1. **`asyncio.Queue`**: Standard FIFO queue.
2. **`asyncio.PriorityQueue`**: Retrieves lowest-valued items first (`tuple(priority_number, data)`).
3. **`asyncio.LifoQueue`**: LIFO (Last-In, First-Out) stack structure.

---

## Related Notes

- [[Notes/AsyncIO/Tasks|Tasks]] — Worker tasks consuming from queues
- [[Notes/AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]] — Managing worker pools
- [[Notes/Projects/Real-Time Event Streamer|Capstone Project]] — Using Queue for streaming event dispatch
- [[Notes/AsyncIO/index|AsyncIO Map of Content]]
