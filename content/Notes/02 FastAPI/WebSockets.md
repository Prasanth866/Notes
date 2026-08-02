
> [!abstract]
> **Goal:** Build persistent, bidirectional communication between clients and the FastAPI server.
>
> Unlike HTTP, a WebSocket connection remains open after the initial handshake, allowing both client and server to send messages at any time.

---

# Where WebSockets Fit

```text
HTTP

Client ----Request----> Server
Client <---Response---- Server

Connection Closed


WebSocket

Client <===============> Server

Connection remains open

Client and Server can send messages independently.
```

Typical use cases:

- Chat applications
- Live dashboards
- Crypto price streaming
- Multiplayer games
- AI token streaming
- Terminal output streaming
- Notifications

---

# FastAPI WebSocket Endpoint

HTTP endpoint:

```python
@app.get("/users")
async def get_users():
    ...
```

WebSocket endpoint:

```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    ...
```

Unlike HTTP endpoints:

- No Request object
- No Response object
- Communication happens through a persistent `WebSocket` object.

---

# Endpoint Lifecycle

```text
Client

↓

Connect

↓

FastAPI creates WebSocket object

↓

Endpoint starts

↓

await websocket.accept()

↓

while connected

↓

receive

↓

process

↓

send

↓

repeat

↓

disconnect

↓

cleanup

↓

endpoint exits
```

---

# The WebSocket Object

Each connected client receives its own `WebSocket` instance.

```python
@app.websocket("/ws")
async def ws(websocket: WebSocket):
    ...
```

Five clients →

```text
Client A → WebSocket A

Client B → WebSocket B

Client C → WebSocket C

Client D → WebSocket D

Client E → WebSocket E
```

Each endpoint execution is independent.

---

# Accepting Connections

```python
await websocket.accept()
```

Until `accept()` is called:

- cannot receive
- cannot send
- connection is not established

Flow:

```text
Client

↓

Handshake Request

↓

FastAPI

↓

accept()

↓

Connection Established
```

---

# Rejecting Connections

Sometimes you don't want to accept the connection.

Example:

```python
if token is None:
    await websocket.close(code=1008)
    return

await websocket.accept()
```

Common reasons:

- invalid authentication
- missing permissions
- banned users
- invalid origin

---

# Receiving Messages

## Text

```python
message = await websocket.receive_text()
```

Returns:

```python
str
```

Example:

Client:

```text
hello
```

Server:

```python
message == "hello"
```

---

## JSON

```python
payload = await websocket.receive_json()
```

Client:

```json
{
    "type":"chat",
    "message":"Hello"
}
```

Server:

```python
payload["type"]
payload["message"]
```

Much more common in production.

---

## Binary

```python
data = await websocket.receive_bytes()
```

Useful for:

- images
- audio
- protobuf
- video

---

# Sending Messages

## Text

```python
await websocket.send_text("hello")
```

---

## JSON

```python
await websocket.send_json({
    "event":"price",
    "price":103456
})
```

Recommended for APIs.

---

## Binary

```python
await websocket.send_bytes(binary_data)
```

---

# Message Schema

Production WebSockets almost always use structured messages.

Instead of

```python
await websocket.send_text("BTC")
```

prefer

```python
await websocket.send_json(
    {
        "event":"price",
        "symbol":"BTC",
        "price":103450
    }
)
```

Advantages:

- extensible
- versionable
- easier frontend handling

---

# Keeping the Connection Alive

Typical endpoint:

```python
@app.websocket("/ws")
async def ws(websocket: WebSocket):

    await websocket.accept()

    while True:

        message = await websocket.receive_text()

        await websocket.send_text(message)
```

Without the loop:

```python
message = await websocket.receive_text()

await websocket.send_text(message)
```

the endpoint ends after one message.

---

# Why receive_* Uses await

```python
message = await websocket.receive_text()
```

If the client sends nothing:

```text
Coroutine pauses

↓

Event loop switches

↓

Other clients continue

↓

Message arrives

↓

Coroutine resumes
```

No CPU is wasted waiting.

---

# Disconnect Handling

When the client disconnects:

```python
await websocket.receive_text()
```

raises

```python
WebSocketDisconnect
```

Always catch it.

```python
try:

    while True:

        message = await websocket.receive_text()

except WebSocketDisconnect:

    print("Disconnected")
```

Without cleanup you'll leak resources.

---

# Closing Connections

Close manually:

```python
await websocket.close()
```

Example:

```python
await websocket.close(
    code=1000
)
```

Possible reasons:

- logout
- idle timeout
- invalid token
- maintenance

---

# ConnectionManager

Never store connected clients directly inside the endpoint.

Instead create:

```text
ConnectionManager
```

Example:

```python
class ConnectionManager:

    def __init__(self):
        self.connections = []

    async def connect(self, websocket):
        await websocket.accept()
        self.connections.append(websocket)

    def disconnect(self, websocket):
        self.connections.remove(websocket)

    async def send(self, websocket, message):
        await websocket.send_json(message)

    async def broadcast(self, message):
        for ws in self.connections:
            await ws.send_json(message)
```

Benefits:

- reusable
- testable
- easier scaling

---

# Broadcast

FastAPI **does not** broadcast automatically.

Wrong assumption:

```text
Alice

↓

Server

↓

Everyone receives
```

Reality:

Each endpoint only knows its own connection.

To broadcast:

```python
for ws in connections:
    await ws.send_json(message)
```

---

# Multiple Clients

Suppose

```
Alice
Bob
Charlie
```

connect.

FastAPI starts three endpoint coroutines.

```text
Endpoint 1

↓

Alice
```

```text
Endpoint 2

↓

Bob
```

```text
Endpoint 3

↓

Charlie
```

Each coroutine is paused independently whenever it awaits.

---

# Authentication

Typical approach:

```text
ws://localhost/ws?token=JWT
```

or

Headers

or

Cookies

Example:

```python
token = websocket.query_params.get("token")
```

Validate before:

```python
await websocket.accept()
```

---

# Dependencies

FastAPI dependencies work with WebSockets too.

Example:

```python
@app.websocket("/ws")
async def ws(
    websocket: WebSocket,
    user=Depends(get_current_user)
):
```

Useful for:

- JWT auth
- permissions
- database session
- tenant lookup

---

# Sending from Multiple Coroutines

This is a common mistake.

```python
await websocket.send_json(...)
```

should generally not be called concurrently by multiple coroutines on the same `WebSocket`.

Instead:

```text
Producer A

↓

Queue

↓

Single sender coroutine

↓

send_json()
```

This avoids race conditions and interleaved writes.

---

# Pattern for Large Applications

Instead of

```text
Producer

↓

WebSocket
```

use

```text
Redis

↓

Pub/Sub

↓

ConnectionManager

↓

WebSocket
```

Example:

```text
CoinGecko Worker

↓

Redis

↓

Broadcast Task

↓

All WebSocket Clients
```

This scales much better.

---

# Handling Slow Clients

Problem:

```text
Broadcast

↓

One client is slow

↓

Everyone waits
```

Solutions:

- send queues per client
- disconnect slow clients
- bounded queues
- timeout writes

---

# Heartbeats

Long-lived connections may need heartbeats.

Typical approaches:

- ping/pong (handled by WebSocket implementation)
- application heartbeat messages

Example:

```json
{
  "event":"ping"
}
```

Useful for detecting dead connections.

---

# Error Handling

Never allow an exception inside the receive loop to silently kill the connection.

Example:

```python
try:
    while True:
        ...
except WebSocketDisconnect:
    ...
except Exception:
    ...
finally:
    manager.disconnect(websocket)
```

Always clean up.

---

# Typical Project Structure

```text
app/

├── main.py
├── websocket/
│   ├── router.py
│   ├── manager.py
│   ├── schemas.py
│   └── auth.py
├── services/
├── redis/
└── api/
```

---

# Production Flow

```text
Browser

↓

WebSocket Connect

↓

FastAPI

↓

Authenticate

↓

accept()

↓

ConnectionManager

↓

receive_json()

↓

Business Logic

↓

Redis Pub/Sub

↓

Broadcast

↓

send_json()

↓

Disconnect

↓

Cleanup
```

---

# Common Mistakes

❌ Forgetting `accept()`

```python
await websocket.receive_text()
```

Raises an error because the connection was never accepted.

---

❌ Forgetting disconnect cleanup

```python
connections.append(websocket)
```

but never removing it.

Leads to memory leaks.

---

❌ Using global lists across multiple server processes

Works only on a single process.

For multiple FastAPI workers:

```
Redis Pub/Sub
```

is required.

---

❌ Broadcasting directly inside business logic

Better:

```text
Business Logic

↓

Publish Event

↓

Redis

↓

Broadcaster

↓

Clients
```

Keeps components decoupled.

---

❌ Sending plain strings everywhere

Prefer structured JSON.

Instead of

```text
"BTC 100000"
```

send

```json
{
  "event":"price_update",
  "symbol":"BTC",
  "price":100000
}
```

---

# Cheat Sheet

| Task | Method |
|------|--------|
| Accept connection | `await websocket.accept()` |
| Receive text | `await websocket.receive_text()` |
| Receive JSON | `await websocket.receive_json()` |
| Receive bytes | `await websocket.receive_bytes()` |
| Send text | `await websocket.send_text()` |
| Send JSON | `await websocket.send_json()` |
| Send bytes | `await websocket.send_bytes()` |
| Close | `await websocket.close()` |

---

# Best Practices

- ✅ Always accept or reject immediately.
- ✅ Catch `WebSocketDisconnect`.
- ✅ Use JSON messages instead of plain text.
- ✅ Separate connection management into a `ConnectionManager`.
- ✅ Validate authentication before accepting.
- ✅ Keep one dedicated sender per connection if multiple producers exist.
- ✅ Use Redis Pub/Sub for multi-instance deployments.
- ✅ Remove disconnected clients immediately.
- ✅ Design versioned message schemas (`event`, `type`, `payload`) for future compatibility.

---

# Mental Model

```text
             FastAPI

                │

        WebSocket Endpoint

                │

        ConnectionManager

                │

       ┌────────┼────────┐

       ▼        ▼        ▼

    Client A Client B Client C

       ▲        ▲        ▲

       └────────┼────────┘

         Business Logic

                │

             Redis Pub/Sub
```

Think of the WebSocket endpoint as the **doorway**, the `WebSocket` object as **one client's dedicated communication channel**, the `ConnectionManager` as the **traffic controller**, and your business logic as the **producer of events**. The endpoint should stay thin—receive messages, delegate work, and send responses—while connection management, authentication, and broadcasting are handled by dedicated components.