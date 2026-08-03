---
title: "Digital Garden Home"
description: "Comprehensive notes and practical guides on Python AsyncIO, FastAPI Asynchronous Web Systems, and Event-Driven Architecture."
tags:
  - index
  - digital-garden
  - python
  - fastapi
  - event-streaming
---

# 🌿 Welcome to My Digital Garden

Welcome! This digital garden serves as a structured, living repository of knowledge focused on **high-performance Python asynchronous systems**, **FastAPI web backends**, and **event-driven streaming protocols**.

---

## 🗺️ Knowledge Map & Index Pages

Explore topic clusters through dedicated Maps of Content (MOCs):

| Topic Cluster | Core Focus | Quick Link |
| :--- | :--- | :--- |
| **01 AsyncIO** | Python event loop mechanics, coroutines, tasks, futures, and concurrency primitives | [[Notes/01 AsyncIO/index\|⚡ AsyncIO MOC]] |
| **02 FastAPI** | Real-time WebSockets, connection managers, and non-blocking background tasks | [[Notes/02 FastAPI/index\|🚀 FastAPI Systems MOC]] |
| **03 Event Streaming** | JSON event schemas, envelope design, and event-driven architecture | [[Notes/03 Event Streaming/index\|📡 Event Streaming MOC]] |
| **04 Projects** | Practical capstone architectures & real-world streaming implementations | [[Notes/04 Projects/Real-Time Event Streamer\|🛠️ Capstone Project]] |
| **00 Meta** | Garden structure, tagging taxonomies, and note-taking principles | [[Notes/00 Meta/index\|⚙️ Meta & Taxonomy]] |

---

## 🚀 Core Learning Path

```
                    ┌─────────────────────────┐
                    │     Non-blocking IO     │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │      Event Loop &       │
                    │       Coroutines        │
                    └────────────┬────────────┘
                                 │
            ┌────────────────────┴────────────────────┐
            ▼                                         ▼
┌───────────────────────┐                 ┌───────────────────────┐
│ Tasks & Concurrency   │                 │ FastAPI WebSockets    │
│ (Queue, TaskGroup)    │                 │ & Connection Manager  │
└───────────┬───────────┘                 └───────────┬───────────┘
            │                                         │
            └────────────────────┬────────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │  JSON Event Streaming   │
                    │      Architecture       │
                    └─────────────────────────┘
```

---

## ⚡ Quick Concept Index

### Python AsyncIO Fundamentals
- [[Notes/01 AsyncIO/Coroutine|Coroutines]] — Pausable async functions (`async def` / `await`)
- [[Notes/01 AsyncIO/Event Loop|Event Loop]] — Core single-threaded scheduler
- [[Notes/01 AsyncIO/Future|Futures]] — Low-level result placeholders
- [[Notes/01 AsyncIO/Tasks|Tasks]] — Event-loop scheduled coroutine wrappers
- [[Notes/01 AsyncIO/Non-blocking IO|Non-blocking I/O]] — OS multiplexing (`epoll`/`kqueue`)

### Concurrency & Synchronization
- [[Notes/01 AsyncIO/asyncio.create_task()|asyncio.create_task()]] — Concurrent task dispatch & GC reference safety
- [[Notes/01 AsyncIO/asyncio.gather()|asyncio.gather()]] — Batch awaitable execution
- [[Notes/01 AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]] — Python 3.11+ structured concurrency & exception groups
- [[Notes/01 AsyncIO/asyncio.Queue()|asyncio.Queue()]] — Producer-consumer queue & backpressure control
- [[Notes/01 AsyncIO/asyncio.sleep()|asyncio.sleep()]] — Non-blocking pauses & cooperative yield points

### FastAPI Real-Time Architecture
- [[Notes/02 FastAPI/WebSockets|WebSockets]] — Full-duplex real-time communication
- [[Notes/02 FastAPI/Connection Manager|Connection Manager]] — Client connection pooling & room broadcast strategies
- [[Notes/02 FastAPI/Background Tasks|Background Tasks]] — In-process post-response task execution

### Event-Driven Design
- [[Notes/03 Event Streaming/JSON Event Design|JSON Event Design]] — Structured envelopes vs plain text streams
- [[Notes/03 Event Streaming/Event Models|Event Models]] — Pydantic v2 discriminated union models for event parsing
