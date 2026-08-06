---
title: "Event Streaming MOC"
description: "Map of Content for event-driven system design, JSON event schemas, envelopes, and type-safe protocols."
tags:
  - architecture/event-driven
  - index
---

# Event Streaming & JSON Protocol Design MOC

Event-driven architecture decouples producers from consumers by communicating state changes through **events**. This Map of Content outlines structured message design, event envelope patterns, and Pydantic validation schemas.

---

## Core Event Streaming Notes

- **[[Notes/03 Event Streaming/JSON Event Design|JSON Event Protocol Design]]**: Transitioning from arbitrary unstructured text strings to typed JSON event envelopes (`event`, `timestamp`, `session_id`, `data`).
- **[[Notes/03 Event Streaming/Event Models|Pydantic Event Models]]**: Using Pydantic v2 discriminated union schemas (`Field(discriminator='event')`) for type-safe event parsing and validation.

---

## Key Architectural Principles

> [!tip]
>
> 1. **Events express facts**: Name events after completed actions or status changes (`task_started`, `token_generated`, `error_occurred`), never UI instructions.
> 2. **Consistent Envelope**: Standardize top-level envelope fields across all event types for uniform routing.
> 3. **Type Safety**: Enforce strict serialization schemas on both Python backend and frontend client receivers.

---

## Related Resources

- [[Notes/02 FastAPI/WebSockets|FastAPI WebSockets]]
- [[Notes/01 AsyncIO/asyncio.Queue()|AsyncIO Event Queues]]
- [[Notes/04 Projects/Real-Time Event Streamer|Capstone Integration Project]]
- [[index|Digital Garden Home]]
