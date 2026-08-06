---
title: "Pydantic Event Models"
description: "Designing type-safe event schemas using Pydantic v2 discriminated unions, JSON schema generation, and strict payload validation."
tags:
  - architecture/event-driven
  - python/pydantic
aliases:
  - Event Models
  - Event Model
---

# Pydantic Event Models

> [!summary]
> In an event-driven streaming system, incoming and outgoing JSON payloads must adhere to strict schemas.
>
> Using **Pydantic v2 Discriminated Unions**, Python applications can automatically parse, validate, and serialize polymorphically typed JSON events based on a single `event` discriminator field.

---

## Polymorphic Event Architecture

```
                                 Incoming Raw JSON String
                                            │
                                            ▼
                              TypeAdapter(StreamEvent)
                                            │
                                  Discriminator Match:
                                    `"event": "..."`
                                            │
         ┌────────────────────────┬─────────┴──────────────┬────────────────────────┐
         ▼                        ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   StatusEvent    │    │  ProgressEvent   │    │    TokenEvent    │    │    ErrorEvent    │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
```

---

## Pydantic v2 Discriminated Union Code

```python
from datetime import datetime, timezone
from typing import Annotated, Literal, Union
from pydantic import BaseModel, Field, TypeAdapter, ValidationError

# Base Envelope Metadata
class BaseEvent(BaseModel):
    timestamp: str = Field(default_factory=lambda: datetime.now(timezone.utc).isoformat())
    session_id: str

# 1. Status Change Event
class StatusEvent(BaseEvent):
    event: Literal["status"] = "status"
    state: Literal["idle", "running", "completed", "failed"]
    message: str

# 2. Progress Percentage Event
class ProgressEvent(BaseEvent):
    event: Literal["progress"] = "progress"
    progress: int = Field(ge=0, le=100)
    current_step: str

# 3. Streaming Text Token Event
class TokenEvent(BaseEvent):
    event: Literal["token"] = "token"
    content: str

# 4. Error Event
class ErrorEvent(BaseEvent):
    event: Literal["error"] = "error"
    error_code: str
    message: str

# Discriminated Union definition using "event" field
StreamEvent = Annotated[
    Union[StatusEvent, ProgressEvent, TokenEvent, ErrorEvent],
    Field(discriminator="event")
]

# TypeAdapter for parsing raw JSON into the polymorphic Union
event_adapter = TypeAdapter(StreamEvent)

# --- Example Usage ---

def parse_incoming_json(raw_json: str) -> StreamEvent:
    try:
        # Automatically parses into the correct model instance (e.g. ProgressEvent)
        parsed_event: StreamEvent = event_adapter.validate_json(raw_json)
        return parsed_event
    except ValidationError as e:
        print("Invalid event schema:", e)
        raise

# Testing
raw_input = '{"event": "progress", "session_id": "s_123", "progress": 75, "current_step": "Downloading weights"}'
event_obj = parse_incoming_json(raw_input)
print(type(event_obj))  # <class '__main__.ProgressEvent'>
print(f"Step: {event_obj.current_step}, Progress: {event_obj.progress}%")
```

---

## Benefits of Pydantic v2 Event Schemas

> [!tip]
>
> 1. **Zero-Boilerplate Parsing**: No nested `if payload['event'] == 'status'` chains required.
> 2. **Automatic Data Validation**: Rejects invalid field types (e.g., negative progress percentages or invalid state strings) before business logic executes.
> 3. **OpenAPI / JSON Schema Export**: Easily export schemas for client code generation (`event_adapter.json_schema()`).

---

## Related Notes

- [[Notes/03 Event Streaming/JSON Event Design|JSON Event Protocol Design]] — Principles of envelope design
- [[Notes/02 FastAPI/WebSockets|FastAPI WebSockets]] — Transporting Pydantic events over sockets
- [[Notes/04 Projects/Real-Time Event Streamer|Capstone Integration Project]] — Real-world engine
- [[Notes/03 Event Streaming/index|Event Streaming MOC]]
