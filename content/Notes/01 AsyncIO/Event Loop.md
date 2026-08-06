---
title: "Event Loop in Python AsyncIO"
description: "Comprehensive guide on the AsyncIO Event Loop, state transitions, ready/waiting queues, single-threaded scheduling, and common pitfalls."
tags:
  - python/asyncio
  - event-loop
aliases:
  - Event Loop
---

# The AsyncIO Event Loop

> [!summary]
> The **Event Loop** is the core engine and scheduler of `asyncio`. It manages all asynchronous execution by deciding **which task runs, which task waits, and when paused tasks resume**.
>
> **Core Concept Hierarchy:**
> > **[[Notes/01 AsyncIO/Coroutine|Coroutines]] describe work → [[Notes/01 AsyncIO/Tasks|Tasks]] wrap work → The Event Loop schedules work.**

---

## Why is the Event Loop Needed?

In traditional synchronous programming, operations execute sequentially, blocking the thread during I/O delays:

```
Sequential Execution (Synchronous):
Download A (3s) ──> Download B (2s) ──> Download C (1s) ──> Total Time = 6s
```

With an asynchronous event loop, operations yield control during wait states:

```
Concurrent Execution (Event Loop):
Start A (3s) ──┐
Start B (2s) ──┼──> Event Loop switches during I/O wait ──> Total Time ≈ 3s
Start C (1s) ──┘
```

Instead of idling, the event loop immediately switches CPU execution to any task that is ready to run.

---

## How the Event Loop Works

The event loop is a single-threaded loop that continuously:
1. Executes tasks in the **Ready Queue**.
2. Polls OS I/O selectors (`epoll`, `kqueue`, `select`) for completed operations.
3. Moves completed waiting tasks back into the **Ready Queue**.
4. Repeats until all scheduled work is finished.

```python
# Conceptual loop representation (managed internally by Python)
while ready_queue or waiting_queue:
    run_ready_tasks()
    poll_io_and_timers()
    move_completed_to_ready()
```

> [!tip]
> In modern Python (3.7+), you rarely manage the loop directly. Calling `asyncio.run(main())` creates, runs, and closes the loop cleanly.

---

## Event Loop Lifecycle

When invoking `asyncio.run(main())`, Python executes the following workflow:

```
   asyncio.run(main())
            │
            ▼
    Create Event Loop
            │
            ▼
    Schedule main() Task
            │
            ▼
     Run Event Loop
            │
            ▼
    main() completes
            │
            ▼
 Cancel remaining tasks & shutdown executors
            │
            ▼
     Close Event Loop
```

> [!important]
> `asyncio.run()` should be invoked **exactly once** as the top-level entry point of your application.

---

## Task Queue State Machine

The event loop moves tasks dynamically between states:

```
┌───────────┐    create_task()    ┌───────────┐     Event Loop     ┌───────────┐
│ Coroutine │ ──────────────────> │   Ready   │ ─────────────────> │  Running  │
└───────────┘                     └───────────┘                    └─────┬─────┘
                                        ▲                                │
                     Operation Complete │           `await` Yield        │
                                  ┌─────┴─────┐                          │
                                  │  Waiting  │ <────────────────────────┘
                                  └───────────┘
```

- **Ready State**: Tasks waiting for CPU execution time.
- **Running State**: The currently active task executing on the thread.
- **Waiting State**: Tasks paused at an `await` boundary awaiting I/O, timers, or [[Notes/01 AsyncIO/Future|Futures]].

---

## Concurrency vs CPU Parallelism

> [!warning]
> Asyncio provides **single-threaded concurrency**, NOT multi-core CPU parallelism. Only **one coroutine executes Python code at any single instant**.

```python
# BAD: CPU-bound loop blocks the entire Event Loop
async def heavy_computation():
    for i in range(100_000_000):
        pass  # No await! Blocks loop completely.
```

### Offloading CPU-Bound Work
To prevent blocking the event loop during heavy CPU calculations, offload execution to a thread or process pool:

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

def cpu_heavy_task(n: int) -> int:
    return sum(i * i for i in range(n))

async def main():
    loop = asyncio.get_running_loop()
    # Offload CPU work to ProcessPoolExecutor
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_heavy_task, 10_000_000)
        print("Result:", result)
```

---

## Common Pitfalls

| Anti-Pattern | Issue | Solution |
| :--- | :--- | :--- |
| `time.sleep(2)` | Blocks the OS thread and halts the entire loop | `await asyncio.sleep(2)` |
| `requests.get(url)` | Synchronous socket I/O blocks loop | `httpx.AsyncClient()` or `aiohttp` |
| Un-awaited tasks | Orphaned tasks can swallow exceptions | Use [[Notes/01 AsyncIO/asyncio.TaskGroup()\|TaskGroup]] |

---

## Related Notes
- [[Notes/01 AsyncIO/Coroutine|Coroutine]] — Pausable functions executed by the loop
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Event loop scheduled tasks
- [[Notes/01 AsyncIO/Non-blocking IO|Non-blocking I/O]] — OS selector mechanics underlying the loop
- [[Notes/01 AsyncIO/index|AsyncIO Map of Content]]