
> [!summary]
> `asyncio.Queue` is an **asynchronous FIFO (First-In, First-Out) queue** that enables safe communication between coroutines without blocking the event loop. It is one of the core synchronization primitives in `asyncio`.

---

# Why use `asyncio.Queue`?

When building asynchronous applications, different coroutines often run at different speeds.

Example:

- A coroutine downloads web pages.
- Another parses HTML.
- Another stores data in a database.

Instead of directly calling each other, they communicate through a queue.

```
Producer
    │
    ▼
┌─────────────────┐
│ asyncio.Queue   │
└─────────────────┘
    │
    ▼
Consumer
```

This **decouples producers and consumers**, allowing each to work independently.

---

# FIFO (First In, First Out)

Items leave the queue in the same order they entered.

```
Producer

Apple
Banana
Orange

↓

Consumer

Apple
Banana
Orange
```

---

# Creating a Queue

```python
import asyncio

queue = asyncio.Queue()
```

By default:

- Unlimited capacity
- FIFO ordering

To limit memory usage:

```python
queue = asyncio.Queue(maxsize=10)
```

---

# Queue Operations

| Method | Purpose |
|---------|---------|
| `await put(item)` | Add an item |
| `await get()` | Remove next item |
| `task_done()` | Mark item as processed |
| `await join()` | Wait until every queued item is processed |
| `qsize()` | Current queue size |
| `empty()` | Is queue empty? |
| `full()` | Is queue full? |

---

# Basic Example

```python
import asyncio

async def main():
    queue = asyncio.Queue()

    await queue.put("Apple")
    await queue.put("Banana")
    await queue.put("Orange")

    print(await queue.get())
    print(await queue.get())
    print(await queue.get())

asyncio.run(main())
```

Output

```
Apple
Banana
Orange
```

---

# Why are `put()` and `get()` awaitable?

## When queue is empty

```python
item = await queue.get()
```

If no item exists:

```
Consumer
    │
queue.get()
    │
Queue Empty
    │
Coroutine pauses
    │
Producer adds item
    │
Coroutine resumes
```

The coroutine **sleeps efficiently** instead of wasting CPU.

---

## When queue is full

```python
await queue.put(item)
```

If the queue has reached `maxsize`:

```
Producer
    │
Queue Full
    │
Producer waits
    │
Consumer removes item
    │
Producer resumes
```

This is called **backpressure**, which prevents unlimited memory growth.

---

# Producer–Consumer Pattern

This is the primary use case.

```python
Producer
    │
Produces data
    │
    ▼
asyncio.Queue
    │
    ▼
Consumer
Processes data
```

Example:

```python
async def producer(queue):
    for i in range(5):
        await queue.put(i)

async def consumer(queue):
    while True:
        item = await queue.get()

        print(item)

        queue.task_done()
```

Benefits:

- Producers and consumers run concurrently.
- Speeds can differ safely.
- No manual synchronization needed.

---

# Understanding `queue.join()`

One of the most important concepts.

Every time:

```python
await queue.put(item)
```

the queue increments an internal counter.

```
put(A)
put(B)
put(C)

Unfinished Tasks = 3
```

Each consumer eventually calls:

```python
queue.task_done()
```

Counter becomes

```
3
↓
2
↓
1
↓
0
```

Only when it reaches zero does

```python
await queue.join()
```

return.

---

# Relationship Between Queue Methods

```
Producer

put()
  │
  ▼
Unfinished Tasks +1

Consumer

get()
  │
Process Item
  │
task_done()
  │
  ▼
Unfinished Tasks -1

join()

Wait until
Unfinished Tasks == 0
```

---

# What Happens if `task_done()` is Forgotten?

```python
item = await queue.get()

# Forgot task_done()
```

Now:

```python
await queue.join()
```

will wait forever because the queue still believes work is pending.

> [!warning]
> Every successful `get()` that represents a completed unit of work should eventually be paired with exactly one `task_done()`.

---

# Multiple Producers

Several coroutines can safely produce items.

```
Producer A
      │
Producer B
      │
Producer C
      │
      ▼
 asyncio.Queue
```

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(producer("A", queue))
    tg.create_task(producer("B", queue))
```

The scheduling order is nondeterministic, but every produced item is queued.

---

# Multiple Consumers

Several workers can process items simultaneously.

```
            Queue
              │
     ┌────────┼────────┐
     ▼        ▼        ▼
 Worker1  Worker2  Worker3
```

Example:

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(consumer("Worker1", queue))
    tg.create_task(consumer("Worker2", queue))
    tg.create_task(consumer("Worker3", queue))
```

Work is automatically distributed among available consumers.

---

# Queue Lifecycle

```
Producer
    │
put(item)
    │
    ▼
Queue
    │
get()
    │
Consumer
    │
Process item
    │
task_done()
    │
join() unblocks
```

---

# Backpressure

Without a maximum size:

```
Producer >>>>>>>>>>>>

Queue keeps growing

Memory usage increases
```

With

```python
asyncio.Queue(maxsize=100)
```

```
Producer
    │
Queue Full
    │
Producer pauses
    │
Consumer catches up
    │
Producer resumes
```

Benefits:

- Prevents memory exhaustion.
- Automatically balances producer and consumer speeds.
- Essential in high-throughput systems.

---

# Real-World Pipeline Example

```
Upload Images
      │
      ▼
 asyncio.Queue
      │
      ▼
 Resize Workers
      │
      ▼
 Store Database
```

Or a web scraper:

```
Fetch URLs
      │
      ▼
 asyncio.Queue
      │
      ▼
 Parse HTML
      │
      ▼
 Save Results
```

Queues separate each processing stage, making pipelines scalable and fault-tolerant.

---

# Queue vs Python List

## Using a list (bad)

```python
items = []

while not items:
    pass
```

Problems:

- Busy waiting
- Wastes CPU
- No synchronization
- Doesn't cooperate with the event loop

---

## Using `asyncio.Queue`

```python
item = await queue.get()
```

Advantages:

- Efficient waiting
- Event-loop friendly
- Built-in synchronization
- Safe producer–consumer communication

---

# Related Queue Types

## `PriorityQueue`

Smallest priority first.

```python
await queue.put((1, "High"))
await queue.put((5, "Low"))

print(await queue.get())
```

Output

```
(1, "High")
```

---

## `LifoQueue`

Stack behavior (Last-In, First-Out).

```python
await queue.put("A")
await queue.put("B")

print(await queue.get())
```

Output

```
B
```

---

# Queue + TaskGroup (Recommended Pattern)

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(producer(queue))

    worker1 = tg.create_task(consumer("Worker-1", queue))
    worker2 = tg.create_task(consumer("Worker-2", queue))

    await queue.join()

    worker1.cancel()
    worker2.cancel()
```

This combines:

- Structured concurrency (`TaskGroup`)
- Work distribution (`Queue`)
- Automatic synchronization (`join`)
- Graceful worker shutdown

---

# Common Use Cases

✅ Producer–consumer pipelines

✅ Web scraping

✅ Background job processing

✅ AI agent event pipelines

✅ WebSocket message queues

✅ Image/video processing

✅ Task scheduling

✅ Log processing

✅ Streaming data

---

# Common Pitfalls

> [!warning]
> Avoid these mistakes

❌ Forgetting `task_done()`

❌ Calling `join()` without consumers

❌ Using an unlimited queue for unbounded workloads

❌ Polling with `empty()` instead of awaiting `get()`

```python
# Bad
while queue.empty():
    pass
```

Instead:

```python
item = await queue.get()
```

❌ Sharing `asyncio.Queue` between different event loops or threads (it's designed for coroutines within a **single event loop**).

---

# Mental Model

Imagine a **restaurant order counter**.

```
Customers
(Producers)
      │
      ▼
 Order Counter
(asyncio.Queue)
      │
      ▼
Chefs
(Consumers)
```

- Customers place orders (`put()`).
- Chefs pick up orders (`get()`).
- Chefs finish cooking (`task_done()`).
- The manager (`join()`) waits until every order has been completed before closing the restaurant.

---

# Quick Summary

> [!tip]
> - Asynchronous FIFO queue for coroutines
> - Non-blocking communication between producers and consumers
> - `put()` waits if full
> - `get()` waits if empty
> - `task_done()` marks work complete
> - `join()` waits until all work is finished
> - Supports multiple producers and consumers
> - `maxsize` provides backpressure
> - Works seamlessly with `TaskGroup`
> - Ideal for pipelines, workers, streaming, and async applications

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Pass work between coroutines | ✅ `asyncio.Queue` |
| Wait for available work | ✅ `await queue.get()` |
| Add work | ✅ `await queue.put(item)` |
| Wait for all work to finish | ✅ `await queue.join()` |
| Mark work complete | ✅ `queue.task_done()` |
| Limit memory usage | ✅ `maxsize` |
| Multiple workers | ✅ Multiple consumers |
| Prioritized processing | ✅ `PriorityQueue` |
| Stack behavior | ✅ `LifoQueue` |