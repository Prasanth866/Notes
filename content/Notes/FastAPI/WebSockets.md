---
title: "WebSockets in FastAPI"
description: "Comprehensive guide to FastAPI WebSockets, frame transport, lifecycle management, ping/pong heartbeats, and authentication."
tags:
  - fastapi/websockets
  - python/asyncio
  - networking
aliases:
  - WebSockets
  - WebSocket
---

# WebSockets in FastAPI

> [!summary]
> **WebSockets** provide a persistent, full-duplex TCP communication channel between client browsers and server backends over a single socket connection.
>
> Unlike stateless HTTP request-response cycles, WebSockets allow servers to push real-time data to connected clients without polling overhead.

---

## HTTP vs WebSocket Protocol Lifecycle

```
HTTP Protocol:
Client ─────────────── Request (GET /api/data) ───────────────> Server
Client <────────────── Response (200 OK + JSON) <───────────── Server  (Connection Closed)

WebSocket Protocol:
Client ─────────────── HTTP Upgrade Request ──────────────────> Server
Client <────────────── 101 Switching Protocols <────────────── Server  (Connection Upgraded)
                               │
               ┌───────────────┴───────────────┐
               ▼                               ▼
       Bi-Directional Real-Time Frames (Text / Binary / Ping / Pong)
```

---

## Production FastAPI WebSocket Endpoint

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, status, Query, HTTPException
import asyncio

app = FastAPI()

async def authenticate_token(token: str | None) -> str:
    """Validate query token during handshake."""
    if not token or token != "secret_auth_token":
        raise HTTPException(status_code=403, detail="Invalid token")
    return "user_123"

@app.websocket("/ws/notifications")
async def websocket_notifications(
    websocket: WebSocket,
    token: str = Query(None)
):
    # 1. Perform Handshake & Token Validation
    if not token or token != "secret_auth_token":
        await websocket.close(code=status.WS_1008_POLICY_VIOLATION)
        return

    # 2. Accept WebSocket Connection
    await websocket.accept()
    print("WebSocket connection accepted.")

    try:
        # 3. Message Loop
        while True:
            # Receive text frame (Yields to Event Loop)
            data = await websocket.receive_text()
            print(f"Received message: {data}")

            # Echo response back
            await websocket.send_json({
                "type": "ack",
                "received": data
            })
    except WebSocketDisconnect:
        # 4. Graceful Cleanup on Connection Drop
        print("Client disconnected cleanly.")
```

---

## Heartbeat & Ping/Pong Protocols

> [!important]
> Firewalls, reverse proxies (Nginx/Cloudflare), and load balancers often close idle TCP connections after 60 seconds of inactivity.
>
> Modern WebSocket backends maintain connection health using **Ping/Pong Heartbeat frames**.

```
Server ─────── Ping Frame ───────> Client
Server <────── Pong Frame <─────── Client
```

If a client fails to respond to 3 consecutive Pings, the server terminates the dead connection to free system file descriptors.

```python
async def heartbeat_ping(websocket: WebSocket, interval: float = 30.0):
    """Periodically sends ping frames to keep connection alive."""
    try:
        while True:
            await asyncio.sleep(interval)
            await websocket.send_bytes(b"PING")
    except Exception:
        pass  # Connection closed
```

---

## Security & Authentication Strategies

| Auth Method          | Transmission                             | Best Practice                                 |
| :------------------- | :--------------------------------------- | :-------------------------------------------- |
| **Query Parameters** | `wss://api.domain.com/ws?token=JWT_HERE` | Useful for standard browser `WebSocket()` API |
| **HTTP Headers**     | Sec-WebSocket-Protocol or Cookie         | More secure against access log leaks          |
| **Handshake Ticket** | Temporary single-use ticket exchange     | Safest enterprise pattern                     |

---

## Related Notes

- [[Notes/FastAPI/Connection Manager|Connection Manager Pattern]] — Pooling and room broadcasting
- [[Notes/Event Streaming/JSON Event Design|JSON Event Design]] — Designing structured WebSocket event payloads
- [[Notes/AsyncIO/Non-blocking IO|Non-blocking I/O]] — Underpinning socket mechanics
- [[Notes/FastAPI/index|FastAPI Systems MOC]]
