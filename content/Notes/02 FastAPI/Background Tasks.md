
> **Essential Reference | FastAPI | Background Processing**
>
> Background tasks allow FastAPI to **return an HTTP response immediately** while scheduling **small, non-critical work** to run **after the response has been sent**.

---

# What are Background Tasks?

Normally, an HTTP request waits until **all processing is complete** before sending a response.

Example:

```python
@app.post("/signup")
async def signup():
    create_user()
    send_email()
    write_log()

    return {"status": "success"}
```

Timeline

```text
Client

↓

Create User

↓

Send Email

↓

Write Log

↓

Return Response
```

If sending the email takes 5 seconds,

the client waits 5 seconds.

---

Background tasks change this behavior.

```python
@app.post("/signup")
async def signup(background_tasks: BackgroundTasks):

    create_user()

    background_tasks.add_task(send_email)

    return {"status": "success"}
```

Timeline

```text
Client

↓

Create User

↓

Return Response Immediately

↓

Background Task

↓

Send Email
```

The response is sent **before** the email starts.

---

# When Should You Use Background Tasks?

Good use cases:

- Sending emails
- Writing logs
- Updating analytics
- Cleaning temporary files
- Generating thumbnails
- Sending notifications
- Cache invalidation
- Recording audit logs

Not suitable for:

- Long-running AI jobs
- Video processing
- Machine learning training
- Heavy report generation
- Multi-minute tasks

Those should use task queues like:

- Celery
- Dramatiq
- RQ
- Arq

---

# How Background Tasks Work

FastAPI provides the `BackgroundTasks` class.

```python
from fastapi import BackgroundTasks
```

Inject it into your endpoint.

```python
@app.post("/send")
async def send(background_tasks: BackgroundTasks):
    ...
```

FastAPI creates an instance automatically.

---

# Adding a Task

```python
background_tasks.add_task(function, *args, **kwargs)
```

Example

```python
background_tasks.add_task(send_email, user.email)
```

Equivalent conceptually to

```text
After response:

send_email(user.email)
```

---

# Simple Example

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def write_log(message):
    with open("log.txt", "a") as f:
        f.write(message + "\n")

@app.post("/log")
async def log(background_tasks: BackgroundTasks):

    background_tasks.add_task(
        write_log,
        "User logged in"
    )

    return {"status": "accepted"}
```

Flow

```text
Request

↓

Schedule write_log()

↓

Return Response

↓

Execute write_log()
```

---

# Background Task Lifecycle

```text
HTTP Request

        │

        ▼

Endpoint Executes

        │

        ▼

background_tasks.add_task()

        │

        ▼

Response Returned

        │

        ▼

Background Task Starts

        │

        ▼

Task Completes
```

---

# Multiple Background Tasks

You can register multiple tasks.

```python
background_tasks.add_task(send_email)

background_tasks.add_task(write_log)

background_tasks.add_task(update_metrics)
```

Execution order:

```text
Response

↓

Task 1

↓

Task 2

↓

Task 3
```

Tasks execute **in the order they were added**.

---

# Passing Arguments

```python
def send_email(email, subject):
    ...

background_tasks.add_task(
    send_email,
    "alice@example.com",
    "Welcome"
)
```

Equivalent to

```python
send_email(
    "alice@example.com",
    "Welcome"
)
```

after the response.

---

# Async vs Sync Tasks

Background tasks can call:

## Normal Functions

```python
def write_log():
    ...
```

or

## Async Functions

```python
async def notify():
    ...
```

FastAPI handles both.

---

# Complete Example

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def send_email(email):
    print(f"Sending email to {email}")

@app.post("/signup")
async def signup(background_tasks: BackgroundTasks):

    user_email = "alice@example.com"

    background_tasks.add_task(
        send_email,
        user_email
    )

    return {
        "message": "User created"
    }
```

Timeline

```text
Client

↓

POST /signup

↓

Create User

↓

Register Background Task

↓

Return JSON

↓

Run send_email()
```

---

# Dependency Injection Support

Background tasks work inside dependencies.

```python
from fastapi import Depends

def log_dependency(
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(write_log)

@app.get("/")
async def home(
    _: None = Depends(log_dependency)
):
    return {"message": "Hello"}
```

FastAPI collects all tasks

↓

Runs them after sending the response.

---

# Exception Handling

If a background task fails,

the client has already received the response.

Example

```python
background_tasks.add_task(send_email)

return {"status": "ok"}
```

Later

```python
send_email()

↓

Exception
```

Client still receives

```json
{
    "status": "ok"
}
```

The exception is logged by the server but **cannot change the already-sent response**.

---

# Background Tasks are Request-Scoped

Every HTTP request gets its own BackgroundTasks object.

```text
Request A

↓

BackgroundTasks A

↓

Tasks
```

```text
Request B

↓

BackgroundTasks B

↓

Tasks
```

Tasks are **not shared** between requests.

---

# Background Tasks vs asyncio.create_task()

Many beginners confuse these.

## BackgroundTasks

```python
background_tasks.add_task(send_email)
```

- Runs **after the response**
- Managed by FastAPI
- Tied to the current request
- Best for small post-response work

---

## asyncio.create_task()

```python
asyncio.create_task(worker())
```

- Starts immediately
- Runs concurrently while the request is still active
- Managed by the event loop
- Useful for concurrent async workflows

---

Timeline

### BackgroundTasks

```text
Request

↓

Response

↓

Background Task
```

---

### create_task()

```text
Request

↓

Task Starts

↓

Response

↓

Task Continues
```

---

# BackgroundTasks vs Task Queue

## BackgroundTasks

```text
FastAPI Process

↓

Background Task
```

If the process crashes,

the task is lost.

---

## Celery / Dramatiq / RQ

```text
FastAPI

↓

Redis / RabbitMQ

↓

Worker Process

↓

Task
```

Advantages

- Retries
- Scheduling
- Persistence
- Multiple workers
- Survives server restarts
- Horizontal scaling

---

# Limitations

BackgroundTasks:

- Execute in the same application process
- No retries
- No scheduling
- Lost if server crashes
- Not distributed
- Suitable only for lightweight work

---

# Internal Flow

```text
HTTP Request

        │

        ▼

FastAPI Endpoint

        │

        ▼

background_tasks.add_task()

        │

        ▼

Store Task

        │

        ▼

Return Response

        │

        ▼

ASGI Response Finished

        │

        ▼

Execute Stored Tasks

        │

        ▼

Task Completed
```

---

# Typical Use Cases

| Task | Good Candidate |
|--------|----------------|
| Send welcome email | ✅ |
| Write audit log | ✅ |
| Analytics tracking | ✅ |
| Push notification | ✅ |
| Delete temp files | ✅ |
| Generate thumbnail | ✅ |
| Export large PDF | ❌ |
| AI inference | ❌ |
| ML training | ❌ |
| Video rendering | ❌ |

---

# Comparison

| Feature | BackgroundTasks | `asyncio.create_task()` | Celery/RQ/Dramatiq |
|----------|----------------|------------------------|--------------------|
| Managed by | FastAPI | Event Loop | Worker Queue |
| Starts | After response | Immediately | Worker process |
| Same process | ✅ | ✅ | ❌ |
| Retry support | ❌ | ❌ | ✅ |
| Persistent | ❌ | ❌ | ✅ |
| Distributed | ❌ | ❌ | ✅ |
| Best for | Small post-response work | Concurrent async work | Long-running jobs |

---

# Best Practices

✅ Use for lightweight work (< a few seconds)

✅ Keep tasks idempotent when possible

✅ Handle exceptions inside the task

✅ Log failures

❌ Don't perform CPU-intensive work

❌ Don't rely on BackgroundTasks for critical business operations

❌ Don't use for long-running workflows

---

# Mental Model

Imagine a receptionist at a hotel.

Without Background Tasks:

```text
Guest arrives

↓

Check in

↓

Print receipt

↓

Call housekeeping

↓

Prepare welcome gift

↓

Send welcome email

↓

Guest leaves
```

The guest waits for everything.

---

With Background Tasks:

```text
Guest arrives

↓

Check in

↓

Hand room key

↓

Guest leaves

↓

Receptionist later:

• Send email
• Notify housekeeping
• Record analytics
• Write logs
```

The guest gets a fast response, while the receptionist finishes the non-essential work afterward.

---

# Key Takeaways

- `BackgroundTasks` schedules work **after the HTTP response is sent**.
- It is ideal for **small, non-critical, post-response operations**.
- Tasks run in the **same FastAPI process**.
- If the application crashes, queued background tasks are **lost**.
- Background tasks are **not** a replacement for Celery or other distributed task queues.
- Use `BackgroundTasks` for lightweight operations like emails, logging, and notifications, not for heavy or long-running jobs.