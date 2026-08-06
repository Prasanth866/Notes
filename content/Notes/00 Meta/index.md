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
    └── 04 Projects/                # Capstone implementations
        └── Real-Time Event Streamer.md
```

---

## Bidirectional Linking Standard

All notes adhere to strict wikilink hygiene:
1. Every conceptual note must link back to its parent **Map of Content (MOC)** (`[[Notes/01 AsyncIO/index|01 AsyncIO]]`, etc.).
2. Cross-references link technical dependencies (e.g. `Tasks` linking to `Future` and `Coroutine`).
