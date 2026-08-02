
> [!summary]
> `asyncio.TaskGroup` is the **recommended way** to create and manage multiple concurrent tasks in Python (introduced in **Python 3.11**). It provides **structured concurrency**, ensuring tasks are managed safely and predictably.

---

## Why use `TaskGroup`?

### Problems with `asyncio.create_task()`

When using `create_task()` directly:

- Tasks start immediately.
- You **must keep references** to every task.
- You **must manually await** them.
- If the parent exits, unfinished tasks are **cancelled**.
- Failed tasks require manual cleanup.
- Can lead to **orphaned background tasks**.

```python
import asyncio

async def worker(name, delay):
    await asyncio.sleep(delay)
    print(f"{name} finished")

async def main():
    asyncio.create_task(worker("A", 2))
    asyncio.create_task(worker("B", 3))

    print("Main finished")

asyncio.run(main())
```

Output:

```
Main finished
```

The event loop ends before the workers finish.

---

## Solution: `TaskGroup`

```python
import asyncio

async def worker(name, delay):
    await asyncio.sleep(delay)
    print(f"{name} finished")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(worker("A", 2))
        tg.create_task(worker("B", 1))
        tg.create_task(worker("C", 3))

    print("All tasks completed")

asyncio.run(main())
```

Output:

```
B finished
A finished
C finished
All tasks completed
```

### Benefits

- ✅ Starts all tasks immediately.
- ✅ Waits for every task automatically.
- ✅ Prevents orphaned tasks.
- ✅ Automatically handles cancellation on failure.
- ✅ Cleaner and more maintainable code.

---

# Structured Concurrency

> [!info]
> **Structured Concurrency** means child tasks **cannot outlive their parent scope**.

```
Parent Task
│
└── TaskGroup
    ├── Task A
    ├── Task B
    └── Task C
```

When execution leaves the `async with` block:

```
Exit TaskGroup
      │
      ▼
Wait for all child tasks
      │
      ▼
Continue execution
```

This guarantees:

- Every task finishes
- Or gets cancelled
- Before execution continues

---

# Lifecycle of a TaskGroup

```
Create TaskGroup
        │
        ▼
Create child tasks
        │
        ▼
Tasks run concurrently
        │
        ▼
One fails?
   │           │
  No          Yes
   │           │
Wait all   Cancel remaining tasks
   │           │
   └──────► Raise exception
```

---

# How it works internally

```python
async with asyncio.TaskGroup() as tg:
```

Creates a task manager.

Every call to

```python
tg.create_task(...)
```

Registers the task with the group.

When the block exits:

1. Wait for every child task.
2. Cancel remaining tasks if one failed.
3. Raise exceptions (if any).
4. Continue execution.

---

# Error Handling (Biggest Advantage)

```python
import asyncio

async def good():
    await asyncio.sleep(3)
    print("Finished")

async def bad():
    await asyncio.sleep(1)
    raise ValueError("Something went wrong")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(good())
        tg.create_task(bad())

asyncio.run(main())
```

Timeline

```
Time 0
│
├── good()
├── bad()
│
Time 1
│
bad() raises exception
│
▼
TaskGroup cancels good()
│
▼
Raises exception
```

### Key Points

- If **one child fails**, all sibling tasks are cancelled.
- Prevents wasted computation.
- Avoids inconsistent application state.
- Parent receives the exception **after cleanup**.

---

# Exception Groups (Python 3.11+)

If multiple tasks fail around the same time, `TaskGroup` raises an **ExceptionGroup**.

```python
try:
    async with asyncio.TaskGroup() as tg:
        ...
except* ValueError as eg:
    print(eg.exceptions)
```

> [!note]
> `except*` is new in Python 3.11 and is designed for handling multiple concurrent exceptions.

---

# Returning Results

Unlike `asyncio.gather()`, results are obtained from the returned `Task` objects.

```python
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(square(2))
    t2 = tg.create_task(square(5))

print(t1.result())
print(t2.result())
```

Output

```
4
25
```

---

# Dynamic Task Creation

Tasks can be created while other tasks are running.

```python
async with asyncio.TaskGroup() as tg:
    for i in range(5):
        tg.create_task(worker(i))
```

Useful when:

- Crawling websites
- Processing queue items
- Handling client requests
- Recursive async workflows

---

# Nested TaskGroups

```
Main Group
│
├── Group A
│   ├── Task A1
│   └── Task A2
│
└── Group B
    ├── Task B1
    └── Task B2
```

Useful for organizing large concurrent workflows into logical units.

---

# TaskGroup vs `asyncio.gather()`

`gather()` is still useful for collecting results.

```python
results = await asyncio.gather(
    task1(),
    task2(),
    task3()
)
```

### Differences

| TaskGroup | gather() |
|-----------|----------|
| Structured concurrency | No structured concurrency |
| Cancels sibling tasks on failure | Different behavior depending on parameters |
| Better resource cleanup | Older API |
| Preferred for new code | Great for simple result collection |

---

# TaskGroup vs gather() vs create_task()

| Feature | TaskGroup | gather() | create_task() |
|----------|-----------|----------|---------------|
| Concurrent execution | ✅ | ✅ | ✅ |
| Automatically waits | ✅ | ✅ | ❌ |
| Structured concurrency | ✅ | ❌ | ❌ |
| Cancels sibling tasks | ✅ | ❌ | ❌ |
| Automatic cleanup | ✅ | ❌ | ❌ |
| Returns results directly | ❌ (use `.result()`) | ✅ | ❌ |
| Best for new code | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

---

# When to Use

## Use `TaskGroup`

- Multiple related concurrent tasks.
- Child tasks belong to the same operation.
- Failure of one should stop the others.
- Building modern async applications.
- FastAPI request handlers.
- Parallel API calls.
- Concurrent database queries.
- AI agents running multiple tools simultaneously.

---

## Use `asyncio.gather()`

- You primarily need **all return values**.
- Tasks are independent.
- Fan-out → fan-in workflows.

---

## Use `create_task()`

- Long-running background jobs.
- Fire-and-forget tasks.
- Periodic schedulers.
- WebSocket listeners.
- Cache refresh workers.
- Tasks intentionally outlive the current function.

> [!warning]
> If you use `create_task()`, **keep a reference** to the task or ensure something manages its lifecycle. Otherwise, it may be cancelled unexpectedly or leak resources.

---

# Performance

`TaskGroup` has **minimal overhead** compared to `create_task()`.

Benefits outweigh the tiny management cost:

- Better safety
- Cleaner code
- Predictable cancellation
- Easier debugging

---

# Common Pitfalls

> [!warning]
> **Don't forget these**

❌ Creating tasks with `create_task()` and never awaiting them.

❌ Assuming `TaskGroup` returns results like `gather()`.

❌ Calling `.result()` **before** the `TaskGroup` exits.

❌ Catching exceptions inside child tasks when you actually want the whole group to fail.

❌ Using `TaskGroup` for long-lived daemon/background tasks.

---

# Mental Model

Imagine a **team lead** managing developers.

```
Team Lead
│
├── Developer A
├── Developer B
└── Developer C
```

The team lead:

- Assigns work.
- Waits for everyone.
- If one developer encounters a critical issue,
  everyone else stops.
- Reports the issue only after cleanup.

This is exactly how `TaskGroup` manages concurrent tasks.

---

# Quick Summary

> [!tip]
> - Python **3.11+**
> - Implements **structured concurrency**
> - Automatically waits for all tasks
> - Cancels sibling tasks on failure
> - Prevents orphaned tasks
> - Safer than manual `create_task()`
> - Preferred over `gather()` for new concurrent code
> - Use `.result()` to retrieve task results after the group exits
> - Ideal for FastAPI, async APIs, WebSockets, AI agents, and concurrent I/O

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Related concurrent tasks | ✅ `TaskGroup` |
| Collect return values easily | ✅ `gather()` |
| Long-running background task | ✅ `create_task()` |
| Automatic cancellation | ✅ `TaskGroup` |
| Structured concurrency | ✅ `TaskGroup` |
| Fire-and-forget | ✅ `create_task()` |
