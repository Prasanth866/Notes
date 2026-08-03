---
title: "JSON Event Protocol Design"
description: "Principles of event-driven protocol design, consistent JSON envelopes, domain event naming, and TypeScript client-side switching."
tags:
  - architecture/event-driven
  - json
aliases:
  - JSON Event Design
---

# 📡 JSON Event Protocol Design

> [!summary]
> Designing a structured event protocol is a critical architectural decision in real-time streaming systems.
>
> The transport layer (WebSockets, SSE, gRPC) handles bytes delivery, but the **JSON Event Protocol** defines the application domain contract.

---

## ❓ Messages vs Events

### ❌ Unstructured Message Design (Anti-Pattern)
```json
{
  "message": "Downloading repository..."
}
```
*Problems:*
- No explicit event type.
- The frontend client cannot determine whether this string represents a log message, a UI progress label, an error, or a chat output.
- Forces brittle frontend string parsing (`if (msg.includes("Downloading")) ...`).

---

### ✅ Structured Event Design (Best Practice)
```json
{
  "event": "status",
  "timestamp": "2026-08-03T15:30:00Z",
  "session_id": "sess_8912",
  "data": {
    "state": "running",
    "message": "Downloading repository"
  }
}
```
*Benefits:*
- Self-documenting domain fact.
- Frontend switches directly on `event: "status"`.
- Clean separation between transport envelope and event data payload.

---

## ✉️ The Standard Event Envelope

Every event in your system should share a top-level **Envelope**:

```
┌──────────────────────────────────────────────┐
│  event: string       (Discriminator)         │
│  timestamp: string   (ISO-8601 UTC)          │
│  session_id: string  (Client Request Trace)  │
│  sequence: number    (Message Counter)       │
├──────────────────────────────────────────────┤
│  data: { ... }       (Typed Payload)         │
└──────────────────────────────────────────────┘
```

---

## 🖥️ TypeScript Client Event Dispatcher

On the client frontend, process incoming WebSocket frames using a type-safe `switch` block:

```typescript
// TypeScript Types matching backend Pydantic models
type StatusEvent = {
  event: "status";
  timestamp: string;
  data: { state: string; message: string };
};

type ProgressEvent = {
  event: "progress";
  timestamp: string;
  data: { progress: number; step: string };
};

type TokenEvent = {
  event: "token";
  timestamp: string;
  data: { content: string };
};

type ErrorEvent = {
  event: "error";
  timestamp: string;
  data: { error_code: string; details: str };
};

type StreamEvent = StatusEvent | ProgressEvent | TokenEvent | ErrorEvent;

// Client WebSocket Message Handler
function handleServerEvent(rawMessage: string) {
  const event: StreamEvent = JSON.parse(rawMessage);

  switch (event.event) {
    case "status":
      updateStatusBadge(event.data.state, event.data.message);
      break;
    case "progress":
      updateProgressBar(event.data.progress);
      break;
    case "token":
      appendTokenToChat(event.data.content);
      break;
    case "error":
      showErrorToast(event.data.error_code, event.data.details);
      break;
    default:
      console.warn("Unhandled event type:", event);
  }
}
```

---

## 🎯 Best Practices Checklist

> [!tip]
> - **Describe facts, not UI**: Name events after domain facts (`repository_cloned`, `token_generated`), never UI rendering instructions (`show_red_box`).
> - **Sequence Numbers**: Include a incremental `sequence` integer to detect out-of-order delivery over networks.
> - **Granular Events**: Prefer multiple lightweight, specific event types (`tool_started`, `tool_finished`) over a giant monolithic state payload.

---

## 🔗 Related Notes
- [[Notes/03 Event Streaming/Event Models|📡 Pydantic Event Models]] — Server-side Python implementation
- [[Notes/02 FastAPI/WebSockets|🚀 FastAPI WebSockets]] — Transporting JSON event streams
- [[Notes/04 Projects/Real-Time Event Streamer|🛠️ Capstone Integration Project]] — End-to-end engine
- [[Notes/03 Event Streaming/index|📡 Event Streaming MOC]]