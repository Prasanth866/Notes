
> [!abstract]
> Designing structured JSON events is one of the most important architectural decisions in an event-driven application.
>
> Instead of sending arbitrary strings over a WebSocket, we send **typed events** with well-defined schemas.
>
> The WebSocket is only the transport layer.
> The JSON event is the application protocol.

---

# Why Event Design Matters

Many beginners start with something like:

```json
{
    "message": "Downloading..."
}
```

It works initially.

But after adding more features:

- AI token streaming
- Progress updates
- Tool execution
- Errors
- Notifications
- File uploads
- Logs

The frontend no longer knows what the message means.

Example:

```json
{
    "message": "Downloading..."
}
```

Questions immediately arise:

- Is this a chat message?
- Is this a log?
- Is this an error?
- Is this a status update?
- Should it appear in the terminal?
- Should it update a progress bar?
- Should it trigger a notification?

There is no context.

This is called an **unstructured message**.

---

# Event-Driven Thinking

Instead of sending messages...

Think in terms of:

> Something happened.

Everything that happens in the application is an **event**.

Examples:

```
User connected

↓

Repository cloned

↓

Agent started

↓

Tool executed

↓

Token generated

↓

Progress updated

↓

Error occurred

↓

Task completed
```

Each becomes a JSON event.

---

# Instead of Messages...

Don't think:

```
Send text
```

Think:

```
Emit Event
```

Instead of

```json
{
    "message": "Downloading..."
}
```

Design

```json
{
    "event": "status",
    "status": "running",
    "message": "Downloading repository"
}
```

The frontend immediately knows

```
Event Type

↓

Status Update

↓

Update status component
```

---

# Events Describe What Happened

A good event answers:

- What happened?
- When did it happen?
- Who generated it?
- What data belongs to it?

For example

```json
{
    "event": "repository_download_started",
    "repository": "FastAPI",
    "timestamp": "2026-08-01T12:00:00Z"
}
```

Notice the difference.

Instead of

```
Downloading...
```

we describe

```
Repository download started
```

Much more meaningful.

---

# Messages vs Events

## Message-based Design

```json
{
    "message": "Installing..."
}
```

Problems:

❌ No type

❌ No metadata

❌ No context

❌ Impossible to extend cleanly

---

## Event-based Design

```json
{
    "event": "status",
    "status": "running",
    "message": "Installing dependencies"
}
```

Now we know

```
Status Event

↓

Current State = Running

↓

Display in status panel
```

---

# The Event Envelope

Every event should have a consistent outer structure.

Think of it as an envelope.

```
┌──────────────────────────────┐
│ Metadata                     │
├──────────────────────────────┤
│ Event Type                   │
├──────────────────────────────┤
│ Payload                       │
└──────────────────────────────┘
```

Example

```json
{
    "event": "...",
    "data": { ... }
}
```

The outer fields remain consistent.

Only the payload changes.

---

# Recommended Base Event Structure

```json
{
    "event": "status",

    "timestamp": "2026-08-01T12:30:15Z",

    "request_id": "req_123",

    "session_id": "session_42",

    "sequence": 15,

    "data": {

    }
}
```

Every event should follow the same structure.

This consistency makes the frontend much simpler.

---

# Why an Event Field?

Imagine receiving

```json
{
    "event": "token"
}
```

The frontend instantly knows

```
Append token to chat window
```

If

```json
{
    "event": "status"
}
```

```
Update status badge
```

If

```json
{
    "event": "progress"
}
```

```
Update progress bar
```

If

```json
{
    "event": "tool_output"
}
```

```
Append to terminal
```

One field determines the entire handling logic.

---

# Event Names

Choose names that describe **facts**, not UI.

Good

```
status

progress

tool_started

tool_finished

token

error

warning

completed

file_uploaded
```

Bad

```
green_box

spinner

chat_text

terminal_line

button_update
```

Events describe the backend.

Not the frontend.

---

# Designing Event Payloads

Each event should contain only information relevant to itself.

Example

Status

```json
{
    "event":"status",

    "status":"running",

    "message":"Installing dependencies"
}
```

---

Tool Output

```json
{
    "event":"tool_output",

    "tool":"terminal",

    "content":"npm install"
}
```

---

Progress

```json
{
    "event":"progress",

    "progress":42,

    "current":"Downloading",

    "total":100
}
```

---

Token

```json
{
    "event":"token",

    "content":"Hello"
}
```

---

Completion

```json
{
    "event":"complete",

    "success":true
}
```

Each event has a different schema.

---

# Example AI Agent Stream

Imagine asking ChatGPT

```
Build a FastAPI project
```

Instead of waiting 30 seconds...

The backend streams events.

```
status

↓

thinking

↓

tool_started

↓

tool_output

↓

token

↓

token

↓

token

↓

tool_finished

↓

complete
```

The UI updates in real time.

---

## Event 1

```json
{
    "event":"status",

    "status":"running",

    "message":"Starting agent"
}
```

---

## Event 2

```json
{
    "event":"thought",

    "content":"Searching documentation..."
}
```

---

## Event 3

```json
{
    "event":"tool_started",

    "tool":"web_search"
}
```

---

## Event 4

```json
{
    "event":"tool_output",

    "tool":"terminal",

    "content":"npm install react"
}
```

---

## Event 5

```json
{
    "event":"token",

    "content":"To"
}
```

---

## Event 6

```json
{
    "event":"token",

    "content":" build"
}
```

---

## Event 7

```json
{
    "event":"token",

    "content":" a"
}
```

---

## Event 8

```json
{
    "event":"complete"
}
```

Notice

No event looks like another.

Each has its own meaning.

---

# Frontend Event Dispatcher

The frontend never checks

```
if message == ...
```

Instead

```typescript
switch(event.event){

    case "status":

    case "token":

    case "progress":

    case "tool_output":

    case "error":

}
```

Each event updates a different part of the UI.

```
token

↓

Chat

progress

↓

Progress Bar

tool_output

↓

Terminal

status

↓

Status Badge

error

↓

Toast Notification
```

---

# Benefits of Event-Based Design

✅ Strong separation of concerns

Backend emits events.

Frontend decides how to display them.

---

✅ Easy to extend

Adding a new feature is often just introducing a new event type.

---

✅ Self-documenting

The JSON itself explains what happened.

---

✅ Easier debugging

Logging events tells the complete story of a request.

---

✅ Better scalability

Different clients (web, mobile, CLI) can consume the same event stream differently.

---

# Common Mistakes

> [!warning]
> **Sending plain strings**

```json
{
    "message":"Done"
}
```

Instead

```json
{
    "event":"complete"
}
```

---

> [!warning]
> **One giant event for everything**

Bad

```json
{
    "event":"everything",

    "status":"...",

    "tool":"...",

    "token":"...",

    "progress":"..."
}
```

Split events into meaningful types.

---

> [!warning]
> **Frontend decides event meaning**

Avoid

```typescript
if(message.includes("Downloading"))
```

Instead

```typescript
if(event.event==="status")
```

---

# Best Practices

> [!tip]
>
> - Every event should represent **one fact**.
> - Keep event schemas consistent.
> - Use descriptive event names.
> - Prefer many small event types over one generic event.
> - Design the event protocol first, then implement the backend and frontend.
> - Treat your JSON events as a public API contract.

---

# Mental Model

Think of your application as a newsroom.

Every important action becomes a news report.

```
Repository cloned

↓

Agent started

↓

Tool executed

↓

Token generated

↓

Progress updated

↓

Error occurred

↓

Completed
```

The backend is the **news reporter**.

The WebSocket is the **broadcast channel**.

The frontend is the **news editor**, deciding where each story belongs.

The event itself is the **news article**.

**Don't stream text—stream events.**