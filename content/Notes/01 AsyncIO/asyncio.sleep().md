
> [!summary]
> `asyncio.sleep()` is an **asynchronous sleep function** that pauses **only the current coroutine** without blocking the event loop.
>
> **Remember:**
>
> > `time.sleep()` blocks the **entire thread**, while `asyncio.sleep()` only pauses the **current coroutine**, allowing other coroutines to continue running.

---

# Why use `asyncio.sleep()`?

In asynchronous programs, waiting should **not stop the entire application**.

Instead of blocking:

```python
time.sleep(2)
```

use

```python
await asyncio.sleep(2)
```

This allows the event loop to schedule other coroutines while waiting.

---

# Syntax

```python
await asyncio.sleep(seconds)
```

Example

```python
import asyncio

async def main():
    print("Start")

    await asyncio.sleep(2)

    print("End")

asyncio.run(main())
```

Output

```
Start

(wait 2 seconds)

End
```

---

# `time.sleep()` vs `asyncio.sleep()`

## `time.sleep()`

```python
time.sleep(2)
```

Execution

```
Program
   │
Start
   │
time.sleep(2)
   │
Everything Stops
   │
End
```

During those 2 seconds:

- No other code runs.
- The thread is completely blocked.

---

## `asyncio.sleep()`

```python
await asyncio.sleep(2)
```

Execution

```
Current Coroutine
        │
Start
        │
await sleep()
        │
Coroutine Pauses
        │
Event Loop Runs Other Tasks
        │
Resume After Delay
        │
End
```

Only the current coroutine pauses.

---

# What Happens Internally?

When Python reaches

```python
await asyncio.sleep(3)
```

Conceptually

```
Current Coroutine

"I have nothing to do
for the next 3 seconds."

        │
        ▼
Event Loop

Runs Other Ready Tasks

        │
        ▼
3 Seconds Later

Resume Coroutine
```

The CPU isn't wasted waiting.

---

# Why is it Important?

`asyncio.sleep()` is an **await point**.

Whenever a coroutine reaches

```python
await asyncio.sleep(...)
```

it **yields control** back to the event loop.

The event loop can now execute:

- Other coroutines
- Network requests
- Database operations
- WebSocket handlers
- Background tasks

---

# Example: Two Coroutines

```python
await asyncio.gather(
    worker("A"),
    worker("B")
)
```

Output

```
A started
B started

(wait 2 seconds)

A finished
B finished
```

Timeline

```
Worker A

sleep(2)

───────────────

Worker B

sleep(2)

───────────────

Both Resume Together
```

The sleeps overlap.

---

# What if We Use `time.sleep()`?

```python
async def worker():
    time.sleep(2)
```

Timeline

```
Worker A

time.sleep()

(Event Loop Blocked)

Worker A Finishes

Worker B Starts

time.sleep()

Worker B Finishes
```

Everything becomes sequential.

Total time doubles.

> [!warning]
> Never use `time.sleep()` inside asynchronous code.

---

# Event Loop Scheduling

Imagine two workers.

```
Worker A

Work

await sleep()

Pause

────────────

Worker B

Runs

await sleep()

Pause

────────────

Worker A resumes

Worker B resumes
```

Every `await` gives the event loop an opportunity to switch tasks.

---

# Countdown Example

```python
async def countdown():
    for i in range(5, 0, -1):
        print(i)
        await asyncio.sleep(1)

    print("Done!")
```

Output

```
5
4
3
2
1
Done!
```

Between every second, other coroutines may execute.

---

# Multiple Countdowns

```python
await asyncio.gather(
    countdown("A", 3),
    countdown("B", 5)
)
```

Possible output

```
A 3
B 5

A 2
B 4

A 1
B 3

A Finished

B 2
B 1

B Finished
```

The event loop switches between coroutines whenever they reach `await asyncio.sleep()`.

---

# `await asyncio.sleep(0)`

One special case is

```python
await asyncio.sleep(0)
```

This **does not actually sleep**.

It means:

> "I'm willing to pause now. Let another ready coroutine run."

Execution

```
Current Coroutine

Step 1

sleep(0)

Yield

Other Tasks Run

Resume

Step 2
```

Useful for:

- Long-running loops
- Preventing event-loop starvation
- Improving responsiveness

---

# Simulating Slow I/O

Many tutorials use

```python
await asyncio.sleep(3)
```

to simulate

- Network requests
- Database queries
- File I/O

Example

```python
async def fetch_user():
    await asyncio.sleep(3)
```

Real applications instead use

```python
await http_client.get(...)
```

or

```python
await database.fetch(...)
```

The important idea is the same:

**Waiting without blocking.**

---

# Using `sleep()` with `TaskGroup`

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(worker("A", 3))
    tg.create_task(worker("B", 1))
    tg.create_task(worker("C", 2))
```

Output

```
A started
B started
C started

B finished

C finished

A finished
```

Each coroutine resumes after its own delay.

---

# Common Use Cases

Use `asyncio.sleep()` for:

- Retry delays
- Polling APIs
- Rate limiting
- Countdown timers
- Simulating slow I/O
- Background loops
- Periodic cleanup jobs
- Yielding control (`sleep(0)`)

---

# Common Mistakes

> [!warning]
> Avoid these mistakes

❌ Using

```python
time.sleep()
```

inside `async def`.

This blocks the entire event loop.

---

❌ Forgetting `await`

```python
asyncio.sleep(2)
```

Creates a coroutine but doesn't execute it.

Correct

```python
await asyncio.sleep(2)
```

---

❌ Using long sleeps when waiting for events.

If you're waiting for:

- Queue items
- Network responses
- Locks
- Events

Prefer awaiting those primitives directly instead of repeatedly sleeping (polling).

---

# Returning a Value

`asyncio.sleep()` supports

```python
result=
```

```python
value = await asyncio.sleep(
    1,
    result="Finished"
)

print(value)
```

Output

```
Finished
```

Useful occasionally in tests or helper code.

---

# Mental Model

Imagine a teacher supervising students.

```
Student A

"I'm waiting 3 seconds."

        │
        ▼
await sleep(3)

Teacher

"While you're waiting,
I'll help Student B and Student C."

3 seconds later...

Teacher

"Student A,
continue working."
```

The student pauses.

The teacher keeps everyone else productive.

---

# Quick Summary

> [!tip]
> - `asyncio.sleep()` pauses only the current coroutine.
> - The event loop continues running other coroutines.
> - Never blocks the thread.
> - Always use `await`.
> - `sleep(0)` simply yields control.
> - Commonly used for retries, polling, timers, rate limiting, and simulating I/O.
> - Never replace it with `time.sleep()` inside async code.

---

# `time.sleep()` vs `asyncio.sleep()`

| Feature | `time.sleep()` | `asyncio.sleep()` |
|---------|----------------|-------------------|
| Blocks thread | ✅ | ❌ |
| Blocks event loop | ✅ | ❌ |
| Must be awaited | ❌ | ✅ |
| Allows other coroutines to run | ❌ | ✅ |
| Used inside async code | ❌ | ✅ |
| Supports `result=` | ❌ | ✅ |

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Pause current coroutine | ✅ `await asyncio.sleep()` |
| Pause without blocking event loop | ✅ `asyncio.sleep()` |
| Yield to other tasks immediately | ✅ `await asyncio.sleep(0)` |
| Retry after delay | ✅ `asyncio.sleep()` |
| Simulate slow I/O | ✅ `asyncio.sleep()` |
| Rate limiting / polling | ✅ `asyncio.sleep()` |
| Pause synchronous code | ✅ `time.sleep()` *(non-async only)* |
| Pause inside async code | ❌ Never use `time.sleep()` |