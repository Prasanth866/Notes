## Definition
- A **coroutine** is a special kind of function that can **pause (suspend)** its execution and later **resume** from where it left off.
- Coroutines are the fundamental building blocks of asynchronous programming in Python.

## Creating a Coroutine
- A coroutine function is defined using `async def`.

```python
async def fetch_data():
    ...
```

## Calling a Coroutine
- Calling a coroutine function **does not execute it immediately**.
- Instead, it returns a **coroutine object**, which represents the pending computation.

```python
coro = fetch_data()   # Nothing runs yet
print(type(coro))
```

## Running a Coroutine
A coroutine executes only when it is:

- Awaited using `await`
- Scheduled as a Task using `asyncio.create_task()`
- Passed to `asyncio.run()` (typically the program's entry point)

```python
async def main():
    await fetch_data()

asyncio.run(main())
```

## Suspending Execution
- A coroutine can suspend its execution using the `await` keyword.
- While suspended, the event loop can run other coroutines.

```python
await asyncio.sleep(1)
```

## What Can Be Awaited?
`await` works with **awaitables**, including:
- Coroutine objects
- `asyncio.Task`
- `asyncio.Future`
- Any object implementing `__await__()`

## Coroutine Function vs Coroutine Object

```python
async def greet():
    print("Hello")
```

- `greet` → **Coroutine Function**

```python
coro = greet()
```

- `coro` → **Coroutine Object**

## Key Points
- Defined using `async def`
- Calling it returns a coroutine object
- Does **not** execute immediately
- Can pause using `await`
- Resumes after the awaited operation completes
- Runs only when awaited or scheduled on an event loop

## Example

```python
import asyncio

async def hello():
    print("Start")
    await asyncio.sleep(2)
    print("End")

asyncio.run(hello())
```

### Execution Flow
1. `hello()` returns a coroutine object.
2. `asyncio.run()` creates an event loop.
3. The coroutine starts executing.
4. It pauses at `await asyncio.sleep(2)`.
5. The event loop runs other ready tasks (if any).
6. After 2 seconds, the coroutine resumes.
7. The coroutine finishes and the event loop exits.

## Mental Model
>A coroutine is a **pausable function** whose execution is managed by the **asyncio event loop**.