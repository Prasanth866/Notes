---
title: "Capstone: Real-Time Event Streamer"
description: "A production-grade architecture and implementation connecting AsyncIO queues, FastAPI WebSockets, and Pydantic discriminated union events."
tags:
  - project/capstone
  - python/asyncio
  - fastapi/websockets
  - architecture/event-driven
---

# Capstone Project: Real-Time Event Streaming Engine

> [!abstract]
> This capstone project integrates concepts from across the digital garden:
> - **[[Notes/01 AsyncIO/index|AsyncIO]]**: Non-blocking `asyncio.Queue` and `asyncio.TaskGroup` for asynchronous task execution and event dispatching.
> - **[[Notes/02 FastAPI/index|FastAPI]]**: High-concurrency WebSocket endpoint and `ConnectionManager` pool broadcasting.
> - **[[Notes/03 Event Streaming/index|Event Streaming]]**: Pydantic v2 discriminated union event models (`status`, `progress`, `token`, `complete`, `error`).

---

## System Architecture

```
                  ┌──────────────────────────────────────────────┐
                  │              FastAPI Client                  │
                  └──────────────────────┬───────────────────────┘
                                         │ WebSocket Connection
                                         ▼
                  ┌──────────────────────────────────────────────┐
                  │       FastAPI Connection Manager             │
                  └──────────────────────┬───────────────────────┘
                                         │
                                 Broadcast Event
                                         │
                  ┌──────────────────────┴───────────────────────┐
                  │          AsyncIO Event Queue                 │
                  └──────────────────────▲───────────────────────┘
                                         │ Put Event
                                         │
                  ┌──────────────────────┴───────────────────────┐
                  │       AsyncIO Background Task Worker         │
                  └──────────────────────────────────────────────┘
```

---

## Full Implementation

```python
import asyncio
import json
import uuid
from datetime import datetime, timezone
from typing import Annotated, Literal, Union
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from pydantic import BaseModel, Field

app = FastAPI(title="Real-Time Event Streamer")

# --- 1. Event Schemas (Discriminated Unions) ---

class BaseEvent(BaseModel):
    timestamp: str = Field(default_factory=lambda: datetime.now(timezone.utc).isoformat())
    session_id: str

class StatusEvent(BaseEvent):
    event: Literal["status"] = "status"
    state: str
    message: str

class ProgressEvent(BaseEvent):
    event: Literal["progress"] = "progress"
    percentage: int
    step_name: str

class TokenEvent(BaseEvent):
    event: Literal["token"] = "token"
    token: str

class ErrorEvent(BaseEvent):
    event: Literal["error"] = "error"
    error_code: str
    details: str

StreamEvent = Annotated[
    Union[StatusEvent, ProgressEvent, TokenEvent, ErrorEvent],
    Field(discriminator="event")
]

# --- 2. Connection Manager ---

class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        if websocket in self.active_connections:
            self.active_connections.remove(websocket)

    async def broadcast_event(self, event: StreamEvent):
        payload = event.model_dump_json()
        disconnected = []
        for connection in self.active_connections:
            try:
                await connection.send_text(payload)
            except Exception:
                disconnected.append(connection)
        for conn in disconnected:
            self.disconnect(conn)

manager = ConnectionManager()

# --- 3. Event Producer Worker ---

async def simulate_background_job(session_id: str, queue: asyncio.Queue[StreamEvent]):
    """Simulates background execution pushing structured events into an AsyncIO queue."""
    await queue.put(StatusEvent(session_id=session_id, state="starting", message="Job initialized"))
    await asyncio.sleep(1)

    for progress in range(25, 101, 25):
        await queue.put(ProgressEvent(session_id=session_id, percentage=progress, step_name=f"Step {progress // 25}"))
        await asyncio.sleep(0.5)

    tokens = ["Generated ", "real-time ", "streaming ", "output!"]
    for tok in tokens:
        await queue.put(TokenEvent(session_id=session_id, token=tok))
        await asyncio.sleep(0.2)

    await queue.put(StatusEvent(session_id=session_id, state="completed", message="Job finished successfully"))

# --- 4. WebSocket Endpoint ---

@app.websocket("/ws/stream/{session_id}")
async def websocket_endpoint(websocket: WebSocket, session_id: str):
    await manager.connect(websocket)
    queue: asyncio.Queue[StreamEvent] = asyncio.Queue()

    # Start background producer
    producer_task = asyncio.create_task(simulate_background_job(session_id, queue))

    try:
        while not producer_task.done() or not queue.empty():
            try:
                event = await asyncio.wait_for(queue.get(), timeout=0.1)
                await websocket.send_text(event.model_dump_json())
                queue.task_done()
            except asyncio.TimeoutError:
                continue
    except WebSocketDisconnect:
        producer_task.cancel()
    finally:
        manager.disconnect(websocket)
```

---

## Verification & Next Steps
- Combine this backend with a client frontend event switcher (`switch (event.event)`).
- See [[Notes/02 FastAPI/WebSockets|FastAPI WebSockets]] and [[Notes/03 Event Streaming/JSON Event Design|JSON Event Design]] for deep protocol specifications.
- Return to [[index|Digital Garden Home]].
