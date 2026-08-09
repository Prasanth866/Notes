---
title: "Python AsyncIO MOC"
description: "Map of Content for Python AsyncIO core concepts, event loop mechanics, task scheduling, and concurrency primitives."
tags:
  - python/asyncio
  - index
---

# Python AsyncIO Map of Content

`asyncio` is Python's standard library module for writing single-threaded concurrent code using the `async`/`await` syntax. This Map of Content organizes all foundational concepts, scheduling mechanics, and concurrency primitives.

---

## Core Architecture & Mechanics

Understanding how Python achieves non-blocking I/O on a single thread:

- **[[Notes/AsyncIO/Non-blocking IO|Non-blocking I/O]]**: How OS event notification mechanisms (`epoll`, `kqueue`, `select`) allow single-threaded event loops to handle thousands of open sockets.
- **[[Notes/AsyncIO/Event Loop|Event Loop]]**: The central scheduler managing task execution states (Ready vs Waiting), timer callbacks, and I/O event polling.
- **[[Notes/AsyncIO/Coroutine|Coroutine]]**: Pausable functions created via `async def` that cooperatively yield control via `await`.
- **[[Notes/AsyncIO/Future|Future]]**: Low-level placeholder objects representing eventual results of asynchronous operations.
- **[[Notes/AsyncIO/Tasks|Tasks]]**: Event-loop wrapped coroutines executing concurrently.

---

## Concurrency & Scheduling Primitives

Tools for orchestrating, timing, and communicating between asynchronous tasks:

- **[[Notes/AsyncIO/asyncio.create_task()|asyncio.create_task()]]**: Schedules a coroutine on the event loop immediately. Covers strong reference retention to prevent garbage collection bugs.
- **[[Notes/AsyncIO/asyncio.gather()|asyncio.gather()]]**: Runs multiple awaitables concurrently and aggregates results into a single tuple.
- **[[Notes/AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]]**: Python 3.11+ structured concurrency context manager with automatic cancellation and `ExceptionGroup` handling.
- **[[Notes/AsyncIO/asyncio.Queue()|asyncio.Queue()]]**: Bounded producer-consumer queue for decoupling producers and managing system backpressure.
- **[[Notes/AsyncIO/asyncio.sleep()|asyncio.sleep()]]**: Non-blocking delay function and cooperative yield point (`await asyncio.sleep(0)`).

---

## Concept Relationship Diagram

```
                             ┌────────────────────────┐
                             │    Non-blocking IO     │
                             └───────────┬────────────┘
                                         │
                                         ▼
                             ┌────────────────────────┐
                             │       Event Loop       │
                             └───────────┬────────────┘
                                         │
                  ┌──────────────────────┴──────────────────────┐
                  ▼                                             ▼
       ┌────────────────────┐                        ┌────────────────────┐
       │     Coroutine      │                        │       Future       │
       └──────────┬─────────┘                        └──────────┬─────────┘
                  │                                             │
                  └──────────────────────┬──────────────────────┘
                                         │
                                         ▼
                             ┌────────────────────────┐
                             │         Task           │
                             └───────────┬────────────┘
                                         │
      ┌──────────────────────────────────┼──────────────────────────────────┐
      ▼                                  ▼                                  ▼
┌───────────┐                     ┌───────────────┐                  ┌────────────┐
│  gather() │                     │  TaskGroup()  │                  │  Queue()   │
└───────────┘                     └───────────────┘                  └────────────┘
```

---

## Return to Garden Index

- [[index|Digital Garden Home]]
