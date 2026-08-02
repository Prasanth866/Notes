
> [!summary]
> `asyncio.create_task()` schedules a **coroutine** to run **concurrently** as an `asyncio.Task` managed by the event loop.
>
> **Remember:**
>
> > A **coroutine** is just a plan. `create_task()` turns that plan into a **running task**.

---

# Why do we need `create_task()`?

Calling an `async` function **does not execute it**.

```python
async def worker():
    ...

coro = worker()
```

Output

```text
<coroutine object worker at 0x...>
```

Nothing runs yet.

```
worker()
    │
    ▼
Coroutine Object

(Not running)
```

A coroutine is simply an object describing work that *could* be executed.

---

# Running a Coroutine with `await`

```python
async def main():
    await worker()
```

Execution flow

```
main()
    │
    ▼
await worker()
    │
    ▼
Worker starts
    │
    ▼
await sleep()
    │
(main waits)
    │
    ▼
Worker resumes
    │
    ▼
Worker finishes
    │
    ▼
main continues
```

### Key Point

`await` is **blocking with respect to the current coroutine**.

While `worker()` runs, `main()` cannot continue past the `await`.

---

# What does `create_task()` do?

```python
async def main():
    task = asyncio.create_task(worker())

    print("Main continues")

    await task
```

Possible output

```
Main continues
Worker started
Worker finished
```

Instead of waiting immediately,

```python
await worker()
```

you tell the event loop:

> "Start running this coroutine in the background while I continue doing other work."

---

# Internal Workflow

```
worker()
    │
    ▼
Coroutine
    │
    ▼
asyncio.create_task()
    │
    ▼
Task
    │
    ▼
Registered with Event Loop
    │
    ▼
Runs concurrently
```

### Relationship

```
Coroutine
      │
      ▼
create_task()
      │
      ▼
Running Task
```

---

# Coroutine vs Task

| Coroutine | Task |
|------------|------|
| Just a plan | Running work |
| Doesn't execute automatically | Scheduled by event loop |
| Cannot run by itself | Managed by event loop |
| Created by calling `async def` | Created by `create_task()` |

---

# What does `create_task()` return?

```python
task = asyncio.create_task(worker())
```

Returns

```text
<Task pending name='Task-2' ...>
```

A Task is a subclass of `Future`.

A task can:

- ✅ Be awaited
- ✅ Be cancelled
- ✅ Return results
- ✅ Report exceptions
- ✅ Report its state

---

# `await` vs `create_task()`

## Using `await`

```python
await worker()

print("Done")
```

Timeline

```
Worker starts
      │
      ▼
Worker finishes
      │
      ▼
Done
```

Everything waits.

---

## Using `create_task()`

```python
task = asyncio.create_task(worker())

print("Doing other work")

await task
```

Timeline

```
Create Task
      │
      ▼
Main continues

Worker starts

Main doing work

Worker sleeping

Main continues

Worker finishes

await task
```

The worker runs **concurrently** with the rest of `main()`.

---

# Concurrent Example

```python
t1 = asyncio.create_task(worker("A", 3))
t2 = asyncio.create_task(worker("B", 1))

await t1
await t2
```

Output

```
A started
B started

B finished

A finished
```

Both tasks begin immediately.

---

# Sequential vs Concurrent

## Sequential

```python
await worker("A", 3)
await worker("B", 1)
```

Timeline

```
A starts
    │
(wait 3s)
    │
A finishes
    │
B starts
    │
(wait 1s)
    │
B finishes
```

Total ≈ **4 seconds**

---

## Concurrent

```python
t1 = asyncio.create_task(worker("A", 3))
t2 = asyncio.create_task(worker("B", 1))

await t1
await t2
```

Timeline

```
A starts
B starts

(wait 1s)

B finishes

(wait 2s)

A finishes
```

Total ≈ **3 seconds**

---

# Task Lifecycle

```
Created
    │
    ▼
Scheduled
    │
    ▼
Running
    │
    ▼
Waiting
(await)
    │
    ▼
Running
    │
    ▼
Finished
```

---

# Accessing Results

```python
async def square(x):
    await asyncio.sleep(1)
    return x * x
```

```python
task = asyncio.create_task(square(5))

result = await task

print(result)
```

Output

```
25
```

Or after completion:

```python
print(task.result())
```

> [!note]
> Only call `.result()` after the task has finished, otherwise it raises an error.

---

# Checking Task Status

```python
task = asyncio.create_task(worker())
```

Initially

```python
task.done()
```

```
False
```

After completion

```python
await task

task.done()
```

```
True
```

Useful methods:

| Method | Purpose |
|---------|---------|
| `done()` | Has the task completed? |
| `cancelled()` | Was it cancelled? |
| `exception()` | Retrieve exception (if any) |
| `result()` | Retrieve return value |

---

# Cancelling Tasks

```python
task.cancel()
```

Cancellation is **cooperative**.

The task receives an

```python
asyncio.CancelledError
```

at the next `await` point.

Example flow

```
Running Task
      │
task.cancel()
      │
      ▼
Cancellation Requested
      │
Next await
      │
      ▼
CancelledError raised
      │
      ▼
Task exits
```

Typical pattern

```python
try:
    await task
except asyncio.CancelledError:
    ...
```

---

# Fire-and-Forget Tasks

Sometimes you'll see

```python
asyncio.create_task(send_logs())
```

without storing the task.

This is called **fire-and-forget**.

Use it only when:

- You intentionally don't need the result.
- You have a strategy for handling exceptions.

Otherwise you may get

```
Task exception was never retrieved
```

because nobody checked the task's outcome.

> [!warning]
> Avoid fire-and-forget unless the task is truly independent (e.g., metrics collection or background logging).

---

# `create_task()` vs `TaskGroup`

Using `create_task()`

```python
t1 = asyncio.create_task(download())
t2 = asyncio.create_task(upload())

await t1
await t2
```

You must:

- Track tasks
- Await tasks
- Handle exceptions
- Cancel tasks if needed

---

Using `TaskGroup`

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(download())
    tg.create_task(upload())
```

`TaskGroup` automatically:

- Waits for all tasks
- Cancels siblings if one fails
- Cleans up properly
- Propagates exceptions

> [!tip]
> For **related tasks**, prefer `TaskGroup` (Python 3.11+).

---

# `create_task()` vs `gather()`

## `gather()`

```python
results = await asyncio.gather(
    task1(),
    task2(),
    task3()
)
```

Best when:

- Running multiple coroutines concurrently
- Collecting all results together

---

## `create_task()`

```python
task = asyncio.create_task(task1())
```

Best when you need to:

- Start work immediately
- Await later
- Cancel later
- Monitor status
- Keep long-lived background tasks

---

# When Should You Use `create_task()`?

Use it when:

- You want work to **start immediately**.
- You need concurrency inside a coroutine.
- You will await the task later.
- You need to cancel or monitor the task.
- You're creating long-running background workers.

Examples:

- WebSocket listeners
- Metrics collectors
- Background schedulers
- Cache refreshers
- Periodic health checks

---

# Common Pitfalls

> [!warning]
> Avoid these mistakes

❌ Calling an async function and forgetting to `await` or `create_task()`.

```python
worker()
```

Creates only a coroutine object.

---

❌ Creating a task and never awaiting or tracking it.

```python
asyncio.create_task(worker())
```

Can hide exceptions.

---

❌ Assuming `task.cancel()` stops the task immediately.

Cancellation only happens at the next suspension point (`await`).

---

❌ Using `create_task()` for related tasks that should succeed or fail together.

Use `TaskGroup` instead.

---

# Mental Model

Think of a restaurant.

Without `create_task()`

```
Chef

Cook Pizza

(wait)

Serve Pizza

(wait)

Cook Pasta
```

Everything happens one at a time.

---

With `create_task()`

```
Chef
│
├── Start Pizza
├── Assistant starts Pasta
└── Manager prepares bills
```

Everyone works simultaneously.

---

# Quick Summary

> [!tip]
> - `create_task()` schedules a coroutine immediately.
> - Returns an `asyncio.Task`.
> - Tasks run concurrently under the event loop.
> - Use `await task` to get the result.
> - Use `task.cancel()` to request cancellation.
> - Fire-and-forget tasks should be used carefully.
> - Prefer `TaskGroup` for related concurrent tasks.
> - Prefer `gather()` when you mainly want multiple return values.

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Start a coroutine concurrently | ✅ `asyncio.create_task()` |
| Wait for result | ✅ `await task` |
| Retrieve completed result | ✅ `task.result()` |
| Check completion | ✅ `task.done()` |
| Cancel task | ✅ `task.cancel()` |
| Run related concurrent tasks | ✅ `TaskGroup` |
| Collect multiple results | ✅ `asyncio.gather()` |
| Long-running background worker | ✅ `create_task()` |
| Independent fire-and-forget work | ✅ `create_task()` *(with proper exception handling)* |