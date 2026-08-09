---
title: "Digital Garden Home"
description: "Comprehensive notes and practical guides on Python AsyncIO, FastAPI Asynchronous Web Systems, Event-Driven Architecture, and Systems Engineering Roadmaps."
tags:
  - index
  - digital-garden
  - python
  - fastapi
  - event-streaming
  - systems-programming
  - roadmap
---

# Welcome to My Digital Garden

Welcome! This digital garden serves as a structured, living repository of knowledge focused on **high-performance Python asynchronous systems**, **FastAPI web backends**, **event-driven streaming protocols**, and **systems engineering projects** (C++ networking, autonomous coding agents).

---

## Knowledge Map & Index Pages

Explore topic clusters through dedicated Maps of Content (MOCs):

| Topic Cluster       | Core Focus                                                                          | Quick Link                                            |
| :------------------ | :---------------------------------------------------------------------------------- | :---------------------------------------------------- |
| **AsyncIO**         | Python event loop mechanics, coroutines, tasks, futures, and concurrency primitives | [[Notes/AsyncIO/index\|AsyncIO MOC]]                  |
| **FastAPI**         | Real-time WebSockets, connection managers, and non-blocking background tasks        | [[Notes/FastAPI/index\|FastAPI Systems MOC]]          |
| **Event Streaming** | JSON event schemas, envelope design, and event-driven architecture                  | [[Notes/Event Streaming/index\|Event Streaming MOC]]  |
| **Projects**        | Practical capstone architectures & real-world streaming implementations             | [[Notes/Projects/index\|Projects MOC]]                |
| **Roadmap**         | Multi-week systems engineering projects: C++ HTTP server, autonomous coding agent   | [[Notes/Roadmap/index\|Roadmap MOC]]                  |
| **Meta**            | Garden structure, tagging taxonomies, and note-taking principles                    | [[Notes/Meta/index\|Meta & Taxonomy]]                 |

---

## Core Learning Path

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

## Quick Concept Index

### Python AsyncIO Fundamentals

- [[Notes/AsyncIO/Coroutine|Coroutines]] — Pausable async functions (`async def` / `await`)
- [[Notes/AsyncIO/Event Loop|Event Loop]] — Core single-threaded scheduler
- [[Notes/AsyncIO/Future|Futures]] — Low-level result placeholders
- [[Notes/AsyncIO/Tasks|Tasks]] — Event-loop scheduled coroutine wrappers
- [[Notes/AsyncIO/Non-blocking IO|Non-blocking I/O]] — OS multiplexing (`epoll`/`kqueue`)

### Concurrency & Synchronization

- [[Notes/AsyncIO/asyncio.create_task()|asyncio.create_task()]] — Concurrent task dispatch & GC reference safety
- [[Notes/AsyncIO/asyncio.gather()|asyncio.gather()]] — Batch awaitable execution
- [[Notes/AsyncIO/asyncio.TaskGroup()|asyncio.TaskGroup()]] — Python 3.11+ structured concurrency & exception groups
- [[Notes/AsyncIO/asyncio.Queue()|asyncio.Queue()]] — Producer-consumer queue & backpressure control
- [[Notes/AsyncIO/asyncio.sleep()|asyncio.sleep()]] — Non-blocking pauses & cooperative yield points

### FastAPI Real-Time Architecture

- [[Notes/FastAPI/WebSockets|WebSockets]] — Full-duplex real-time communication
- [[Notes/FastAPI/Connection Manager|Connection Manager]] — Client connection pooling & room broadcast strategies
- [[Notes/FastAPI/Background Tasks|Background Tasks]] — In-process post-response task execution

### Event-Driven Design

- [[Notes/Event Streaming/JSON Event Design|JSON Event Design]] — Structured envelopes vs plain text streams
- [[Notes/Event Streaming/Event Models|Event Models]] — Pydantic v2 discriminated union models for event parsing

### Systems Engineering Roadmaps

- [[Notes/Roadmap/ForgeHTTP|ForgeHTTP]] — 40-day roadmap: build an HTTP/1.1 server from scratch in C++ (TCP, epoll, Reactor, reverse proxy, observability)
- [[Notes/Roadmap/CodingAgent|CodingAgent (Mini Devin)]] — 30-day roadmap: production-grade autonomous coding agent (FastAPI, LangGraph, Docker sandbox, AST indexing, guardrails)
