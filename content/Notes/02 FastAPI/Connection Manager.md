---
title: "FastAPI WebSocket Connection Manager"
description: "Designing stateful connection managers in FastAPI for client connection pooling, room routing, and multi-worker Redis Pub/Sub broadcasting."
tags:
  - fastapi/websockets
  - python/asyncio
aliases:
  - Connection Manager
---

# FastAPI Connection Manager Pattern

> [!summary]
> A **Connection Manager** is a centralized state manager in FastAPI that maintains active **[[Notes/02 FastAPI/WebSockets|WebSocket]]** client connections, handles registration/disconnection lifecycles, and routes targeted or room-wide event broadcasts.

---

## System Architecture

```
                  ┌──────────────────────────────────────────────┐
                  │          ConnectionManager Pool              │
                  │   active_connections: dict[str, WebSocket]   │
                  └──────────────────────┬───────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │ Broadcast                      │ Room Target                    │ Direct User
        ▼                                ▼                                ▼
┌───────────────┐                ┌───────────────┐                ┌───────────────┐
│ Client 1 (WS) │                │ Client 2 (WS) │                │ Client 3 (WS) │
└───────────────┘                └───────────────┘                └───────────────┘
```

---

## In-Memory Connection Manager Pattern

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
import json

class ConnectionManager:
    def __init__(self):
        # Store active connections mapped by client ID
        self.active_connections: dict[str, WebSocket] = {}
        # Store connections grouped by chat room ID
        self.rooms: dict[str, set[WebSocket]] = {}

    async def connect(self, client_id: str, websocket: WebSocket, room_id: str = "default"):
        await websocket.accept()
        self.active_connections[client_id] = websocket

        if room_id not in self.rooms:
            self.rooms[room_id] = set()
        self.rooms[room_id].add(websocket)

    def disconnect(self, client_id: str, room_id: str = "default"):
        if client_id in self.active_connections:
            ws = self.active_connections.pop(client_id)
            if room_id in self.rooms and ws in self.rooms[room_id]:
                self.rooms[room_id].remove(ws)

    async def send_personal_message(self, message: dict, client_id: str):
        if client_id in self.active_connections:
            await self.active_connections[client_id].send_text(json.dumps(message))

    async def broadcast_to_room(self, room_id: str, message: dict):
        if room_id in self.rooms:
            payload = json.dumps(message)
            # Iterate over a copy to handle concurrent disconnects safely
            for connection in list(self.rooms[room_id]):
                try:
                    await connection.send_text(payload)
                except Exception:
                    # Stale connection handle cleanup
                    pass

manager = ConnectionManager()
app = FastAPI()

@app.websocket("/ws/{room_id}/{client_id}")
async def websocket_endpoint(websocket: WebSocket, room_id: str, client_id: str):
    await manager.connect(client_id, websocket, room_id)
    try:
        while True:
            data = await websocket.receive_text()
            # Broadcast received message to room
            await manager.broadcast_to_room(room_id, {
                "sender": client_id,
                "content": data
            })
    except WebSocketDisconnect:
        manager.disconnect(client_id, room_id)
        await manager.broadcast_to_room(room_id, {
            "sender": "SYSTEM",
            "content": f"Client {client_id} disconnected."
        })
```

---

## Scaling Beyond a Single Process (Redis Pub/Sub)

> [!important]
> An in-memory `ConnectionManager` only tracks connections on a **single Uvicorn process**.
>
> In multi-worker or multi-server deployments (e.g. `uvicorn main:app --workers 4`), client connections are distributed across processes. To broadcast events across workers, integrate a **Redis Pub/Sub** message broker:

```
Uvicorn Worker 1 ─── (Client A) ──┐
                                  ├──> [Redis Pub/Sub Channel] ──> Broadcast Event
Uvicorn Worker 2 ─── (Client B) ──┘
```

```python
# Redis Pub/Sub Listener Loop inside Uvicorn Worker startup
async def redis_broadcast_listener(manager: ConnectionManager, redis_pubsub):
    async for message in redis_pubsub.listen():
        if message["type"] == "message":
            event_data = json.loads(message["data"])
            await manager.broadcast_to_room(event_data["room_id"], event_data["payload"])
```

---

## Related Notes

- [[Notes/02 FastAPI/WebSockets|FastAPI WebSockets]] — Core WebSocket endpoint mechanics
- [[Notes/03 Event Streaming/JSON Event Design|JSON Event Design]] — Message payload protocols
- [[Notes/04 Projects/Real-Time Event Streamer|Capstone Project]] — Full implementation
- [[Notes/02 FastAPI/index|FastAPI Systems MOC]]
