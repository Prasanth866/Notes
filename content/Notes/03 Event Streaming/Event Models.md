
> Event Streaming is a pattern where the server continuously emits **events** over time instead of returning a single response.

Instead of:

```
Request
↓
Processing
↓
One Response
↓
Done
```

Event streaming looks like:

```
Client Connects
      │
      ▼
Server emits Event 1
      │
      ▼
Server emits Event 2
      │
      ▼
Server emits Event 3
      │
      ▼
...
```

The connection stays open.

---

# Why Event Streaming?

Many modern applications don't return a single result.

Instead they continuously send updates.

Examples:

## AI Chat

```
Thinking...

↓

Token: "Hel"

↓

Token: "lo"

↓

Token: "!"

↓

Completed
```

---

## Cryptocurrency App

```
BTC Price

↓

101000

↓

101020

↓

101050

↓

100980

↓

...
```

---

## File Upload

```
Uploading...

↓

10%

↓

35%

↓

72%

↓

100%
```

---

## Build System

```
Starting build

↓

Installing packages

↓

Compiling

↓

Running tests

↓

Finished
```

---

# Traditional HTTP vs Event Streaming

## HTTP

```
Client

↓

POST /chat

↓

Wait

↓

Wait

↓

Wait

↓

Entire response
```

---

## Streaming

```
Client

↓

POST /chat

↓

Token

↓

Token

↓

Token

↓

Finished
```

The user sees progress immediately.

---

# What is an Event?

An **event** is a message describing **something that happened**.

Examples:

```
User Connected

Price Updated

Agent Started

Token Generated

Tool Executed

File Uploaded

Job Completed

Error Occurred
```

Instead of sending random strings:

```python
await websocket.send_text("hello")
```

we usually send structured events.

---

# Event Structure

An event generally contains:

```
What happened?

↓

Data

↓

Metadata
```

Example:

```json
{
    "type": "price_update",
    "data": {
        "symbol": "BTC",
        "price": 101234
    }
}
```

The client immediately knows:

```
Event Type

↓

price_update
```

and how to process it.

---

# Why Not Send Plain Strings?

Suppose the server sends:

```
101200
```

What does it mean?

```
Price?

Temperature?

User ID?

Score?
```

Impossible to know.

Instead:

```json
{
    "type": "price_update",
    "price": 101200
}
```

Now the client knows exactly what it represents.

---

# Event Model

An **Event Model** is a structured representation of an event.

In FastAPI we typically use **Pydantic models**.

Example:

```python
from pydantic import BaseModel

class PriceEvent(BaseModel):
    type: str
    symbol: str
    price: float
```

Instead of dictionaries:

```python
{
    "symbol": "BTC",
    "price": 100000
}
```

we create:

```python
PriceEvent(
    type="price_update",
    symbol="BTC",
    price=100000
)
```

Benefits:

- Validation
- Type safety
- Autocomplete
- Documentation
- Easier refactoring

---

# Generic Event Envelope

Most production systems use a common wrapper.

```
Event

├── event
├── timestamp
├── id
└── payload
```

Example:

```json
{
    "id": "evt_123",
    "event": "price_update",
    "timestamp": "2026-08-01T12:30:00Z",
    "payload": {
        "symbol": "BTC",
        "price": 101000
    }
}
```

This is called an **Event Envelope**.

---

# Recommended Base Event Model

```python
from datetime import datetime
from uuid import UUID, uuid4

from pydantic import BaseModel, Field

class Event(BaseModel):
    id: UUID = Field(default_factory=uuid4)
    event: str
    timestamp: datetime = Field(default_factory=datetime.utcnow)
    payload: dict
```

Every event now has:

- unique id
- event type
- timestamp
- payload

---

# Specialized Events

Instead of one huge model:

```python
{
    "type": "...",
    "..."
}
```

create one model per event.

```python
class PriceUpdate(BaseModel):
    symbol: str
    price: float
```

```
class ChatToken(BaseModel):
    token: str
```

```
class ErrorEvent(BaseModel):
    code: str
    message: str
```

Each represents one event.

---

# Streaming AI Responses

Instead of:

```
Entire answer
```

stream:

```
Agent Started

↓

Thinking

↓

Token

↓

Token

↓

Token

↓

Finished
```

Possible events:

```text
agent_started

thinking

token

tool_started

tool_finished

completed

error
```

---

# Example AI Stream

Server emits:

```json
{
    "event":"agent_started"
}
```

↓

```json
{
    "event":"token",
    "payload":{
        "text":"Hel"
    }
}
```

↓

```json
{
    "event":"token",
    "payload":{
        "text":"lo"
    }
}
```

↓

```json
{
    "event":"completed"
}
```

---

# Client Handling

Instead of:

```python
if message == "done":
```

use:

```python
match event["event"]:

    case "token":
        ...

    case "error":
        ...

    case "completed":
        ...
```

Each event has a dedicated handler.

---

# FastAPI Example

Server:

```python
event = {
    "event": "price_update",
    "payload": {
        "symbol": "BTC",
        "price": 101200
    }
}

await websocket.send_json(event)
```

Client receives:

```json
{
    "event":"price_update",
    "payload":{
        "symbol":"BTC",
        "price":101200
    }
}
```

---

# Event Flow

```
Price Worker

↓

Creates PriceUpdate Event

↓

Redis Pub/Sub (optional)

↓

Connection Manager

↓

WebSocket

↓

Browser

↓

UI updates
```

Notice that the worker never talks directly to the browser.

Everything communicates through events.

---

# Why Event Models Matter

Imagine changing:

```python
{
    "price":101000
}
```

to

```python
{
    "current_price":101000
}
```

Without models:

Every consumer breaks silently.

With Pydantic:

Type errors appear immediately.

---

# Common Event Types

## Chat

```
message

typing

joined

left

error
```

---

## AI

```
agent_started

thinking

token

tool_started

tool_finished

completed

cancelled

error
```

---

## Trading

```
price_update

portfolio_update

order_created

order_filled

order_cancelled

alert_triggered
```

---

## Job Queue

```
job_started

job_progress

job_completed

job_failed
```

---

# Event-Driven Architecture

Instead of:

```
Worker

↓

WebSocket

↓

Client
```

use

```
Worker

↓

Event

↓

Broker (optional)

↓

Connection Manager

↓

WebSocket

↓

Client
```

Every component only knows about events.

This makes the system loosely coupled.

---

# Best Practices

## Always include an event type

Good:

```json
{
    "event":"price_update"
}
```

Bad:

```json
{
    "price":100000
}
```

---

## Version events

Useful for long-lived APIs.

```json
{
    "version":1,
    "event":"price_update"
}
```

---

## Keep payloads focused

Good:

```json
{
    "event":"price_update",
    "payload":{
        "symbol":"BTC",
        "price":100000
    }
}
```

Avoid huge payloads with unrelated data.

---

## Make events immutable

Treat an emitted event as a historical fact.

Don't mutate it after publishing.

---

# Relationship to Connection Manager

```
Price Worker

↓

Price Event

↓

Connection Manager

↓

Broadcast

↓

Clients
```

The Connection Manager doesn't care **what** the event is.

It simply delivers it.

---

# Relationship to Redis Pub/Sub

```
Worker

↓

Publish Event

↓

Redis Channel

↓

Connection Manager

↓

Broadcast

↓

Browser
```

Redis transports events between processes.

The Connection Manager transports events to clients.

---

# Mental Model

Think of an airport.

```
Flight takes off

↓

Airport creates an event

↓

Departure Board updates

↓

Passengers see the update
```

The airport doesn't call every passenger individually.

It publishes an event, and every interested system reacts.

Your FastAPI application should work the same way.

Workers produce events, infrastructure transports them, and clients react based on the event type.