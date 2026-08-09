---
title: "Meta & Garden Structure"
description: "Overview of garden organization, taxonomy, tagging conventions, and note structures."
tags:
  - meta
  - documentation
---

# Digital Garden Meta & Taxonomy

This meta page details the structural rules, tagging taxonomy, and design conventions used across this digital garden.

---

## Tagging Hierarchy

Every note in this garden includes structured YAML frontmatter. Tags follow a categorized namespace:

- `#python/asyncio` — AsyncIO mechanics, event loop, futures, tasks
- `#python/concurrency` — Synchronization primitives (`Queue`, `Lock`, `Semaphore`)
- `#fastapi/websockets` — WebSocket protocols and endpoint handlers
- `#fastapi/background` — Starlette & FastAPI background worker processing
- `#architecture/event-driven` — Streaming protocol design, JSON envelopes, Pydantic models
- `#project/capstone` — End-to-end integration implementations
- `#cpp` / `#networking` / `#linux` — C++ systems programming, TCP/IP, Linux OS internals
- `#roadmap` — Multi-week structured project plans with daily tasks and checkboxes

---

## Directory Structure

```
content/
├── index.md                        # Garden Home & Master Index
└── Notes/
    ├── 00 Meta/                    # Garden architecture & guidelines
    │   └── index.md
    ├── 01 AsyncIO/                 # Core Python AsyncIO primitives & loop mechanics
    │   ├── index.md
    │   ├── Coroutine.md
    │   ├── Event Loop.md
    │   ├── Future.md
    │   ├── Non-blocking IO.md
    │   ├── Tasks.md
    │   ├── asyncio.Queue().md
    │   ├── asyncio.TaskGroup().md
    │   ├── asyncio.create_task().md
    │   ├── asyncio.gather().md
    │   └── asyncio.sleep().md
    ├── 02 FastAPI/                 # Real-time WebSockets & background tasks
    │   ├── index.md
    │   ├── Background Tasks.md
    │   ├── Connection Manager.md
    │   └── WebSockets.md
    ├── 03 Event Streaming/        # Event-driven protocol design & schemas
    │   ├── index.md
    │   ├── Event Models.md
    │   └── JSON Event Design.md
    ├── 04 Projects/                # Capstone implementations
    │   └── Real-Time Event Streamer.md
    └── 05 Roadmap/                 # Systems engineering project roadmaps
        ├── ForgeHTTP.md            # 40-day: HTTP server in C++ (TCP, epoll, Reactor, proxy)
        ├── CodingAgent.md          # 30-day: Production coding agent (LangGraph, Docker, FastAPI)
        └── DBMS.md                 # Database systems notes
```

---

## Bidirectional Linking Standard

All notes adhere to strict wikilink hygiene:

1. Every conceptual note must link back to its parent **Map of Content (MOC)** (`[[Notes/01 AsyncIO/index|01 AsyncIO]]`, etc.).
2. Cross-references link technical dependencies (e.g. `Tasks` linking to `Future` and `Coroutine`).
3. Roadmap notes in `05 Roadmap/` use Obsidian internal links (`[[TCP]]`, `[[epoll]]`, `[[LangGraph]]`) to connect concepts — create stub notes for these as you learn them.

---

## Roadmaps

| Roadmap | Description | Link |
|---|---|---|
| ForgeHTTP | 40-day C++ HTTP server: TCP sockets, epoll event loop, Reactor pattern, static files, reverse proxy, performance profiling | [[Notes/05 Roadmap/ForgeHTTP\|ForgeHTTP]] |
| CodingAgent | 30-day production coding agent: FastAPI, LangGraph, Docker sandbox with security tests, AST indexing, semantic search, guardrails | [[Notes/05 Roadmap/CodingAgent\|CodingAgent]] |
