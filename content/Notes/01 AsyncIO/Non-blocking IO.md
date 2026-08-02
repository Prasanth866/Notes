> [!summary]
> **Non-blocking I/O** allows a program to **start an I/O operation, pause only the current coroutine while waiting, and continue executing other tasks** instead of blocking the entire thread.
>
> **Remember:**
>
> > **Blocking I/O waits. Non-blocking I/O works while waiting.**

---

# What is I/O?

**I/O (Input/Output)** is any operation where your program communicates with something outside the CPU.

Common examples:

- Reading/Writing files
- HTTP requests
- Database queries
- Sending emails
- WebSocket communication
- DNS lookups
- Socket communication

These operations are **much slower than CPU operations** because they depend on disks, networks, or external services.

---

# Why is I/O Slow?

CPU operations happen in **nanoseconds**.

External operations take much longer.

| Operation | Approximate Time |
|-----------|-----------------:|
| CPU instruction | ~1 ns |
| RAM access | ~100 ns |
| SSD read | ~100 μs |
| Network request | 10–500 ms |
| Internet API | 100 ms–seconds |

> [!important]
> Most backend applications spend **far more time waiting for I/O than executing CPU instructions**.

---

# Blocking I/O

Example

```python
import requests

response = requests.get("https://example.com")

print(response.text)
```

Timeline

```
Program

↓

Send Request

↓

Wait

↓

Wait

↓

Receive Response

↓

Continue
```

During the wait:

- CPU is mostly idle.
- Current thread cannot perform other work.

This is **blocking I/O**.

---

# Real-World Analogy

Imagine a waiter.

Blocking behavior

```
Take Order

↓

Stand In Kitchen

↓

Wait

↓

Food Ready

↓

Deliver

↓

Next Customer
```

The waiter spends most of the time doing nothing.

---

# Non-blocking I/O

Instead

```
Take Order

↓

Kitchen Starts Cooking

↓

Serve Other Tables

↓

Take More Orders

↓

Kitchen Bell Rings

↓

Deliver Food
```

The waiter stays productive.

This is exactly how `asyncio` works.

---

# Async Example

```python
import asyncio

async def fetch():
    print("Sending request...")

    await asyncio.sleep(3)

    print("Response received")

asyncio.run(fetch())
```

Execution

```
Coroutine

↓

Send Request

↓

await

↓

Pause Coroutine

↓

Event Loop Runs Other Tasks

↓

Response Ready

↓

Resume Coroutine
```

Only the coroutine pauses.

The application continues working.

---

# Blocking vs Non-blocking

## Blocking

```python
import time

print("Start")

time.sleep(3)

print("End")
```

Execution

```
Start

↓

Everything Stops

↓

3 Seconds

↓

End
```

---

## Non-blocking

```python
import asyncio

async def main():
    print("Start")

    await asyncio.sleep(3)

    print("End")
```

Execution

```
Start

↓

Current Coroutine Pauses

↓

Other Coroutines Run

↓

Resume

↓

End
```

The output looks similar, but the behavior is completely different.

---

# Why is Non-blocking Faster?

Suppose each download takes **3 seconds**.

Blocking

```
Download A

↓

3 Seconds

↓

Download B

↓

3 Seconds

↓

Download C

↓

3 Seconds

Total = 9 Seconds
```

---

Non-blocking

```python
await asyncio.gather(
    download_a(),
    download_b(),
    download_c()
)
```

Execution

```
Start A
Start B
Start C

↓

All Waiting Together

↓

Responses Arrive

↓

Done

Total ≈ 3 Seconds
```

The waiting overlaps.

---

# Example: Multiple HTTP Requests

Blocking

```python
requests.get(site1)
requests.get(site2)
requests.get(site3)
```

Execution

```
Request 1

↓

Wait

↓

Request 2

↓

Wait

↓

Request 3

↓

Wait
```

---

Async

```python
await asyncio.gather(
    fetch(site1),
    fetch(site2),
    fetch(site3)
)
```

Execution

```
Request 1
Request 2
Request 3

↓

Wait Together

↓

Responses Arrive
```

Huge performance improvement for I/O-heavy workloads.

---

# What Happens Inside the Event Loop?

Suppose

```python
await network_request()
```

The coroutine tells the event loop

> "I'm waiting for the network."

Execution

```
Task A

↓

Waiting

↓

Run Task B

↓

Task B Waiting

↓

Run Task C

↓

OS Reports Task A Ready

↓

Resume Task A
```

The event loop keeps the CPU busy.

---

# What Does the Operating System Do?

The coroutine **does not monitor the network itself**.

Instead:

```
Coroutine

↓

Starts Network Request

↓

Operating System Handles I/O

↓

Coroutine Suspended

↓

Event Loop Runs Others

↓

OS Signals Completion

↓

Event Loop Resumes Coroutine
```

The operating system notifies the event loop when data is available.

---

# Common Async I/O Operations

Examples

```python
await asyncio.sleep()

await http_client.get(url)

await websocket.recv()

await database.fetch()

await file.read()

await socket.recv()
```

Each one spends most of its time waiting for external resources.

---

# CPU-bound vs I/O-bound

## CPU-bound

```python
for i in range(1_000_000_000):
    ...
```

Characteristics

- Heavy calculations
- CPU always busy
- Few or no `await` points

Asyncio provides little benefit.

---

## I/O-bound

```python
await socket.recv()

await database.fetch()

await http_client.get()

await asyncio.sleep()
```

Characteristics

- Mostly waiting
- CPU frequently idle
- Many `await` points

Ideal for asyncio.

---

# CPU vs I/O Timeline

## CPU-bound

```
CPU

Working

Working

Working

Working

Working
```

Nothing else can run until an `await` occurs.

---

## Blocking I/O

```
CPU

Working

Idle

Idle

Idle

Working
```

Lots of wasted CPU time.

---

## Non-blocking I/O

```
Task A Waiting

↓

Task B Running

↓

Task C Running

↓

Task A Ready

↓

Task A Running
```

CPU stays productive.

---

# How Modern Operating Systems Help

Operating systems provide scalable I/O notification systems.

Examples

- Linux → `epoll`
- macOS / BSD → `kqueue`
- Windows → I/O Completion Ports (IOCP)

Instead of repeatedly checking every socket, the event loop asks:

> "Notify me when this socket is ready."

```
Event Loop

↓

Register Socket

↓

Sleep Efficiently

↓

OS Sends Notification

↓

Resume Waiting Task
```

This is why asyncio can efficiently handle **thousands of simultaneous connections**.

---

# Common Mistakes

> [!warning]
> Avoid these mistakes

### ❌ Using blocking libraries

```python
requests.get(...)
```

inside async code blocks the event loop.

Prefer asynchronous libraries.

---

### ❌ Using `time.sleep()`

```python
time.sleep(2)
```

Blocks the thread.

Use

```python
await asyncio.sleep(2)
```

---

### ❌ Long CPU loops

```python
for _ in range(...):
    ...
```

Without `await`, the event loop cannot switch tasks.

---

### ❌ Assuming asyncio speeds up CPU work

Asyncio improves **I/O concurrency**, not heavy computation.

---

# When Should You Use Non-blocking I/O?

Ideal for:

- 🌐 Web servers (FastAPI, aiohttp)
- 🔌 WebSockets
- 🤖 Chat applications
- 📡 REST APIs
- 🗄️ Database access
- ☁️ Cloud services
- 📁 Async file operations
- 🕷️ Web scraping
- 📨 Message queues

Not ideal for:

- Scientific computing
- Image processing
- Video rendering
- Machine learning training
- Cryptography
- Heavy numerical algorithms

Use threads or processes for those workloads.

---

# Blocking vs Non-blocking

| Feature | Blocking I/O | Non-blocking I/O |
|---------|--------------|------------------|
| Waits for operation | ✅ | ❌ |
| Thread blocked | ✅ | ❌ |
| CPU idle during wait | ✅ | ❌ |
| Other tasks can run | ❌ | ✅ |
| Efficient resource usage | ❌ | ✅ |
| Best for async applications | ❌ | ✅ |

---

# Mental Model

Imagine a librarian.

### Blocking

```
Student A

↓

Find Book

↓

Wait

↓

Return Book

↓

Student B
```

Everyone waits in line.

---

### Non-blocking

```
Student A Requests Book

↓

Student B Requests Book

↓

Student C Requests Book

↓

Librarian Helps Whoever Is Ready

↓

Books Arrive

↓

Deliver Books
```

Nobody wastes time waiting.

---

# Quick Summary

> [!tip]
> - I/O means communicating with external resources.
> - Blocking I/O pauses the entire thread.
> - Non-blocking I/O pauses only the current coroutine.
> - The event loop runs other coroutines while one waits.
> - Waiting operations overlap, dramatically improving throughput.
> - Asyncio is designed for **I/O-bound**, not **CPU-bound**, workloads.

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Read/write files asynchronously | Async file APIs |
| Make HTTP requests | Async HTTP client |
| Query database | Async database driver |
| WebSocket communication | Async WebSocket library |
| Delay without blocking | `await asyncio.sleep()` |
| Run multiple I/O tasks | `asyncio.gather()` / `TaskGroup` |
| CPU-intensive work | Threads / Processes |

---

# Mental Model

```
External Resource
(Network, Disk, Database)

          │
          ▼

Coroutine Starts I/O

          │

await

          ▼

Coroutine Suspended

          │

Event Loop Runs Other Tasks

          │

OS Signals Completion

          ▼

Coroutine Resumes
```

> **Non-blocking I/O is the foundation of `asyncio`.** Instead of wasting CPU time waiting for external resources, the event loop keeps the application productive by switching to other ready coroutines until the waiting operation completes.