---
title: "FastAPI Background Tasks"
description: "Detailed guide to FastAPI BackgroundTasks, post-response execution, thread pool offloading, and Celery task queue boundaries."
tags:
  - fastapi/background
  - python/asyncio
aliases:
  - Background Tasks
---

# FastAPI Background Tasks

> [!summary]
> FastAPI provides a `BackgroundTasks` class (inherited from Starlette) that allows endpoints to **return an HTTP response immediately** while scheduling small, post-response tasks to run in the background.

---

## Execution Timeline Comparison

### 1. Synchronous Request Handling (Traditional)

```
Client ──> [HTTP Request] ──> Create User ──> Send Email (5s) ──> Return Response (Total: 5.5s)
```

The HTTP client must wait for every operation (including email delivery) to complete before receiving a status response.

### 2. Post-Response Background Task (FastAPI)

```
Client ──> [HTTP Request] ──> Create User ──> Return Response Immediately (0.05s)
                                                    │
                                                    ▼
                                           [Background Tasks]
                                                    │
                                                    ▼
                                            Send Email (5s)
```

The response is sent to the client **before** the email task starts execution!

---

## Implementation Pattern

FastAPI injects an instance of `BackgroundTasks` automatically via dependency injection:

```python
from fastapi import FastAPI, BackgroundTasks
import asyncio
import time

app = FastAPI()

def write_audit_log(user_id: int, action: str):
    """Synchronous task running in Starlette threadpool."""
    time.sleep(1)  # Simulate file I/O
    with open("audit.log", "a") as f:
        f.write(f"[{time.strftime('%X')}] User {user_id}: {action}\n")

async def send_welcome_email(email: str):
    """Asynchronous coroutine task running directly on Event Loop."""
    await asyncio.sleep(2)  # Non-blocking async email dispatch
    print(f"Welcome email sent to {email}")

@app.post("/users/{user_id}")
async def create_user(user_id: int, email: str, background_tasks: BackgroundTasks):
    # Enqueue tasks for execution AFTER response is returned
    background_tasks.add_task(write_audit_log, user_id, "ACCOUNT_CREATED")
    background_tasks.add_task(send_welcome_email, email)

    return {"status": "success", "message": "User created. Notifications enqueued."}
```

---

## Async vs Sync Task Execution

When passing functions to `background_tasks.add_task(func)`:

| Function Type  | Execution Strategy                                                      | Reference Note                                         |
| :------------- | :---------------------------------------------------------------------- | :----------------------------------------------------- |
| `async def`    | Executed directly on the **[[Notes/AsyncIO/Event Loop                | Event Loop]]**                                         | [[Notes/AsyncIO/Coroutine\|Coroutine]] |
| Standard `def` | Offloaded to Starlette's background **Thread Pool** (`anyio.to_thread`) | [[Notes/AsyncIO/Non-blocking IO\|Non-blocking I/O]] |

---

## When to Use FastAPI BackgroundTasks vs Celery/Task Queues

> [!warning]
> FastAPI `BackgroundTasks` run **in-process** inside the ASGI web server worker. If the web server process crashes, uncompleted background tasks are lost!

```
┌────────────────────────────────────────────────────────────────────────┐
│                   In-Process: FastAPI BackgroundTasks                 │
│  - Ideal for: Audit logging, analytics pings, simple email dispatches  │
│  - Duration: < 10 seconds                                              │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│               Distributed Task Queue: Celery / Dramatiq / RQ           │
│  - Ideal for: Heavy AI inference, video encoding, multi-minute jobs   │
│  - Guarantees: Redis/RabbitMQ persistence, retries, worker scaling     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Related Notes

- [[Notes/AsyncIO/asyncio.create_task()|asyncio.create_task()]] — In-loop background task creation
- [[Notes/FastAPI/WebSockets|FastAPI WebSockets]] — Real-time event streaming alternative
- [[Notes/FastAPI/index|FastAPI Systems MOC]]
