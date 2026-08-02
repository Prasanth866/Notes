## Definition

A **Task** is an object that wraps a **coroutine** and schedules it to run on the **asyncio event loop**.

Unlike a coroutine object, which does nothing until awaited, a Task begins executing as soon as the event loop gets a chance.

A Task is an instance of `asyncio.Task`, which is a subclass of `asyncio.Future`.

---

# Why do we need Tasks?

Without Tasks, coroutines execute sequentially.

```python
async def main():
    await task1()
    await task2()
```

Execution:

```
task1 starts
task1 finishes
task2 starts
task2 finishes
```

Total time = `time(task1) + time(task2)`

---

Tasks allow multiple coroutines to make progress concurrently.

```python
async def main():
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())

    await t1
    await t2
```

Execution:

```
task1 starts
task2 starts

task2 finishes
task1 finishes
```

Total time = `max(time(task1), time(task2))`

---

# What does create_task() do?

```python
task = asyncio.create_task(coro())
```

Internally it:

1. Creates the coroutine object.
2. Wraps it in an `asyncio.Task`.
3. Registers the task with the event loop.
4. Returns immediately.
5. The event loop starts executing it as soon as possible.

Flow:

```
Coroutine Function
        │
        ▼
Coroutine Object
        │
create_task()
        ▼
	  Task
        │
	Event Loop
        ▼
Runs Concurrently
```

---

# Coroutine vs Task

## Coroutine

```python
coro = fetch_data()
```

- Just an object.
- Not scheduled.
- Doesn't execute by itself.

---

## Task

```python
task = asyncio.create_task(fetch_data())
```

- Scheduled immediately.
- Managed by the event loop.
- Executes concurrently with other tasks.

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
Waiting (await)
    │
    ▼
Runnable
    │
    ▼
Running
    │
    ▼
Completed
```

Possible alternative states:

```
Running
    │
Cancelled
```

or

```
Running
    │
Exception Raised
```

---

# State Example

```python
async def work():
    print("Start")

    await asyncio.sleep(2)

    print("Resume")

    return 42
```

Timeline:

```
Running
↓

await sleep()

↓

Paused

↓

Other tasks execute

↓

Sleep completes

↓

Running again

↓

Finished
```

---

# Why Tasks enable concurrency

Suppose:

```python
await asyncio.sleep(2)
```

The task isn't using the CPU.

Instead of wasting those 2 seconds, the event loop runs another ready task.

```
Task A
↓

await sleep()

↓

Paused

↓

Task B runs

↓

Task C runs

↓

Sleep finishes

↓

Task A resumes
```

This is **cooperative multitasking**.

Tasks voluntarily give control back to the event loop whenever they reach an `await`.

---

# create_task() vs await

## await

```python
await download()
```

- Starts the coroutine.
- Waits until completion.
- Caller is suspended until finished.

---

## create_task()

```python
task = asyncio.create_task(download())
```

- Starts immediately.
- Returns a Task.
- Caller continues executing.
- Can await later.

---

# Example

```python
async def main():

    task = asyncio.create_task(download())

    print("Doing other work")

    await asyncio.sleep(1)

    print("Still working")

    await task

    print("Done")
```

Possible output

```
Doing other work
Downloading...

Still working

Download complete

Done
```

---

# Awaiting a Task

Tasks are awaitable.

```python
task = asyncio.create_task(work())

result = await task
```

The returned value is whatever the coroutine returns.

```python
async def square(x):
    return x * x

task = asyncio.create_task(square(5))

result = await task

print(result)
```

Output

```
25
```

---

# Task Exceptions

If the coroutine raises an exception,

```python
async def fail():
    raise ValueError("Boom")
```

then

```python
task = asyncio.create_task(fail())

await task
```

raises

```
ValueError: Boom
```

Tasks don't hide exceptions.

---

# Cancelling Tasks

Tasks can be cancelled.

```python
task.cancel()
```

The coroutine receives

```python
asyncio.CancelledError
```

Example

```python
async def worker():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("Cleaning up...")
        raise
```

---

# Checking Task State

Useful methods:

```python
task.done()
```

Returns

```
True
False
```

---

```python
task.cancelled()
```

Returns

```
True
False
```

---

```python
task.result()
```

Returns the coroutine's result.

Raises an exception if the task failed.

---

```python
task.exception()
```

Returns the exception raised by the task.

---

# Naming Tasks

Python 3.8+

```python
task = asyncio.create_task(worker(), name="worker-1")
```

Later

```python
task.get_name()
```

Useful for debugging.

---

# Waiting for Multiple Tasks

## asyncio.gather()

```python
results = await asyncio.gather(
    task1(),
    task2(),
    task3()
)
```

Runs all concurrently.

Returns results in order.

```
[
 result1,
 result2,
 result3
]
```

---

## asyncio.wait()

```python
done, pending = await asyncio.wait(tasks)
```

Useful when you need lower-level control.

---

## asyncio.as_completed()

```python
for task in asyncio.as_completed(tasks):
    result = await task
```

Processes results as soon as each task finishes.

Very useful for downloading many files or making many API requests.

---

# Fire-and-Forget Tasks

Sometimes you don't immediately await a task.

```python
asyncio.create_task(send_logs())
```

The task runs in the background.

However, you should usually keep a reference:

```python
background = asyncio.create_task(send_logs())
```

Otherwise, it may be harder to manage or observe exceptions.

---

# Real-world Uses

Tasks are used everywhere in asynchronous applications:

- FastAPI background processing
- WebSocket connection handlers
- Chat servers
- Download managers
- Concurrent API requests
- Agent frameworks
- Streaming LLM responses
- Event processing pipelines

---

# Mental Model

Think of a restaurant.

```
Coroutine
```

A recipe.

---

```
Task
```

A chef cooking the recipe.

---

```
Event Loop
```

The head chef deciding which cook should work next.

Whenever one chef has to wait (oven, timer, delivery), the head chef assigns another chef to work.

---

# Key Points

- A Task wraps a coroutine.
- Tasks are scheduled on the event loop.
- Tasks begin executing automatically.
- Tasks enable concurrency.
- Tasks are awaitable.
- Tasks can return values.
- Tasks can raise exceptions.
- Tasks can be cancelled.
- Multiple tasks can run concurrently on a single thread.
- A Task is a subclass of `asyncio.Future`.

---

# Summary

| Concept | Description |
|----------|-------------|
| Coroutine | A pausable async function. |
| Task | A scheduled coroutine managed by the event loop. |
| await | Suspends until an awaitable completes. |
| create_task() | Schedule a coroutine to run concurrently. |
| gather() | Run multiple awaitables concurrently and wait for all. |
| wait() | Lower-level waiting API for tasks. |
| as_completed() | Process tasks in completion order. |