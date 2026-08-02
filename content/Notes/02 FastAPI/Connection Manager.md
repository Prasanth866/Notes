
> [!abstract]
> **Goal:** Maintain and manage all active WebSocket connections in one place.
>
> A Connection Manager is responsible for:
>
> - Accepting new WebSocket connections
> - Tracking connected clients
> - Sending messages to individual clients
> - Broadcasting messages
> - Cleaning up disconnected clients
> - Providing a central place for connection-related logic

---

# What is a Connection Manager?

A **Connection Manager** is **not provided by FastAPI**.

It is an application-level class that stores and manages active WebSocket connections.

Think of it as the **registry of connected clients**.

```text
                    FastAPI

                       │

              WebSocket Endpoint

                       │

                Connection Manager

                       │

      ┌────────────────┼────────────────┐

      ▼                ▼                ▼

  WebSocket A     WebSocket B     WebSocket C
```

Without it, every WebSocket endpoint only knows about **its own client**.

---

# Why Do We Need One?

Suppose three users connect.

```
Alice
Bob
Charlie
```

FastAPI creates

```
Alice  → websocket_a

Bob    → websocket_b

Charlie→ websocket_c
```

Alice sends

```
Hello everyone
```

Inside Alice's endpoint you only have

```python
websocket
```

which is

```text
websocket_a
```

There is no built-in way to access Bob's or Charlie's sockets.

Therefore we store them.

---

# Responsibility of a Connection Manager

A good Connection Manager answers questions like:

- Who is connected?
- How many clients are connected?
- Send to one client?
- Send to everyone?
- Send to a room?
- Disconnect a client?
- Is this user already connected?
- Which websocket belongs to user X?

---

# Basic Implementation

```python
from fastapi import WebSocket

class ConnectionManager:

    def __init__(self):
        self.active_connections: list[WebSocket] = []
```

Initially

```text
active_connections

[]
```

---

# Connect

```python
async def connect(self, websocket: WebSocket):

    await websocket.accept()

    self.active_connections.append(websocket)
```

Flow

```text
Client

↓

Connection Request

↓

accept()

↓

Append websocket

↓

Ready
```

After three users

```text
[
    websocket_a,
    websocket_b,
    websocket_c
]
```

---

# Disconnect

```python
def disconnect(self, websocket: WebSocket):

    self.active_connections.remove(websocket)
```

Before

```text
Alice

Bob

Charlie
```

After Bob disconnects

```text
Alice

Charlie
```

Always remove disconnected clients.

Otherwise you'll attempt to send messages to dead sockets.

---

# Personal Messages

```python
async def send_personal_message(
    self,
    websocket: WebSocket,
    message: str
):
    await websocket.send_text(message)
```

Example

```python
await manager.send_personal_message(
    websocket,
    "Hello!"
)
```

---

# Broadcasting

```python
async def broadcast(self, message: str):

    for connection in self.active_connections:

        await connection.send_text(message)
```

Flow

```text
broadcast()

↓

Alice

↓

Bob

↓

Charlie
```

Everyone receives

```
Hello
```

---

# Full Minimal Manager

```python
from fastapi import WebSocket

class ConnectionManager:

    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(
        self,
        message: str,
        websocket: WebSocket
    ):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)
```

---

# Using the Manager

Create a single instance.

```python
manager = ConnectionManager()
```

Endpoint

```python
@app.websocket("/chat")
async def chat(websocket: WebSocket):

    await manager.connect(websocket)

    try:

        while True:

            message = await websocket.receive_text()

            await manager.broadcast(message)

    except WebSocketDisconnect:

        manager.disconnect(websocket)
```

The endpoint now delegates connection management.

---

# Internal Lifecycle

Client connects

```text
↓

Endpoint

↓

manager.connect()

↓

accept()

↓

Store websocket
```

Client sends

```text
↓

receive_text()

↓

Business Logic

↓

manager.broadcast()
```

Client disconnects

```text
↓

WebSocketDisconnect

↓

manager.disconnect()
```

---

# Why Use a Class Instead of a Global List?

Instead of

```python
connections.append(ws)

connections.remove(ws)

for ws in connections:
    ...
```

spread across many files,

encapsulate everything.

```python
await manager.connect()

manager.disconnect()

await manager.broadcast()
```

Benefits

- Cleaner endpoints
- Easier testing
- Easier extension
- Single responsibility

---

# Production Problem #1
## Sending to Closed Connections

Imagine

```
Alice
Bob
Charlie
```

Charlie disconnects unexpectedly.

Manager

```text
Alice

Bob

Charlie (dead)
```

Next broadcast

```python
await ws.send_text(...)
```

raises an exception.

Good implementation

```python
async def broadcast(self, message: str):

    disconnected = []

    for ws in self.active_connections:

        try:
            await ws.send_text(message)

        except Exception:
            disconnected.append(ws)

    for ws in disconnected:
        self.disconnect(ws)
```

Always clean up dead sockets.

---

# Production Problem #2
## Slow Clients

Suppose

```
Alice ← fast

Bob ← fast

Charlie ← very slow network
```

Naive broadcasting

```python
for ws in connections:
    await ws.send_json(data)
```

Timeline

```text
Alice

↓

Bob

↓

Wait...

↓

Wait...

↓

Charlie

↓

Continue
```

One slow client delays everyone.

---

### Better Design

Give every client its own outgoing queue.

```text
Producer

↓

Queue A

↓

Sender A

↓

Alice
```

```text
Producer

↓

Queue B

↓

Sender B

↓

Bob
```

Each client has an independent sender coroutine.

Slow clients no longer block fast ones.

---

# Production Problem #3
## Multiple Coroutines Sending

This is a very common bug.

Imagine

```
Redis Listener

↓

send_json()
```

and

```
Chat Endpoint

↓

send_json()
```

Both write to the same socket simultaneously.

Result

```
Race Condition
```

Recommended design

```text
Many Producers

↓

asyncio.Queue

↓

Single Sender Coroutine

↓

send_json()
```

Only one coroutine writes to the socket.

---

# Tracking Users

Instead of

```python
list[WebSocket]
```

store

```python
dict[str, WebSocket]
```

Example

```python
self.connections = {

    "alice": websocket_a,

    "bob": websocket_b,

    "charlie": websocket_c
}
```

Now

```python
await self.connections["bob"].send_json(...)
```

is possible.

---

# Mapping User IDs

Even better

```python
dict[int, WebSocket]
```

```python
connections = {

    1: websocket,

    2: websocket,

    3: websocket
}
```

This matches database user IDs.

---

# Multiple Connections Per User

Modern apps often allow one user to connect from:

- Laptop
- Phone
- Tablet

Instead of

```python
dict[user_id, WebSocket]
```

use

```python
dict[user_id, set[WebSocket]]
```

Example

```text
42

↓

Laptop

Phone

Tablet
```

Broadcast to user

```python
for ws in connections[user_id]:
    await ws.send_json(message)
```

---

# Rooms / Topics

Instead of

```text
All Clients
```

store

```python
rooms = {

    "crypto": set(),

    "sports": set(),

    "chat": set()
}
```

Joining

```python
rooms["crypto"].add(websocket)
```

Broadcast

```python
for ws in rooms["crypto"]:
    await ws.send_json(message)
```

---

# Connection Metadata

Sometimes store more than a WebSocket.

```python
@dataclass
class ClientConnection:

    websocket: WebSocket

    user_id: int

    username: str

    connected_at: datetime

    subscriptions: set[str]
```

Instead of

```python
list[WebSocket]
```

store

```python
dict[user_id, ClientConnection]
```

Much easier for production systems.

---

# Scaling Beyond One Process

Simple manager

```text
FastAPI Instance

↓

Memory

↓

Connections
```

Works only for one process.

If using

```
uvicorn --workers 4
```

Each worker has

```text
Worker 1

Connections A
```

```text
Worker 2

Connections B
```

Worker 1 cannot directly send to Worker 2's clients.

---

## Solution

```text
Redis Pub/Sub

↓

Worker A

↓

Connections
```

```text
Redis Pub/Sub

↓

Worker B

↓

Connections
```

Broadcast flow

```text
Producer

↓

Redis

↓

Every FastAPI Worker

↓

Each Connection Manager

↓

Connected Clients
```

---

# Suggested Project Structure

```text
app/

websocket/

    manager.py

    router.py

    schemas.py

    events.py

services/

redis/

auth/
```

Manager only manages connections.

Business logic belongs elsewhere.

---

# Best Practices

✅ Keep endpoints thin.

```text
Receive

↓

Service

↓

Manager

↓

Send
```

---

✅ Never let business logic manipulate connection lists directly.

---

✅ Store user IDs, not just sockets.

---

✅ Always clean up on disconnect.

---

✅ Use JSON messages.

---

✅ Use Redis Pub/Sub when running multiple workers.

---

✅ Give each client a dedicated sender queue for high-throughput systems.

---

# Common Mistakes

❌ Forgetting to remove disconnected clients.

---

❌ Broadcasting directly from every endpoint.

---

❌ Multiple coroutines writing to one WebSocket simultaneously.

---

❌ Using a global list across multiple worker processes.

---

❌ Putting business logic inside the Connection Manager.

The manager should only manage connections—not validate messages, query databases, or implement application rules.

---

# Cheat Sheet

| Method | Purpose |
|---------|---------|
| `connect(ws)` | Accept and register a client |
| `disconnect(ws)` | Remove a client |
| `send_personal_message()` | Send to one client |
| `broadcast()` | Send to every client |
| `active_connections` | Track all connected sockets |

---

# Mental Model

Think of the Connection Manager as the **network switch** inside your application.

```text
                  Connection Manager

             (Application Network Switch)

                      │

        ┌─────────────┼─────────────┐

        ▼             ▼             ▼

     WebSocket A   WebSocket B   WebSocket C

        │             │             │

      Alice          Bob         Charlie
```

Endpoints don't communicate with clients directly—they ask the Connection Manager to route messages. As your application grows, the Connection Manager evolves from a simple list of sockets into a routing layer that understands users, rooms, subscriptions, permissions, and eventually multiple server instances via Redis Pub/Sub.