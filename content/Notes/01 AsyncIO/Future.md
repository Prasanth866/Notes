> **Essential Reference | asyncio Internals | Python 3.11+**

---

## What is a Future?

`asyncio.Future` is a **low-level awaitable object** that represents **a value that will become available later**.

It acts as a bridge between:

- the code **producing** a result
- the code **waiting** for that result

A Future itself **does not perform any work**.

It is simply a **placeholder for a future result**.

```text
Producer
    │
    ▼
 Future
    │
    ▼
Consumer
```

---

## Mental Model

Imagine ordering food.

```text
Order Pizza
      │
      ▼
Receipt (Future)
      │
      ▼
Kitchen cooks
      │
      ▼
Pizza Ready
      │
      ▼
Receive Pizza
```

The receipt isn't the pizza.

It only promises:

> "You'll receive the pizza later."

A Future is exactly that promise.

---

# Why does asyncio need Futures?

Many asynchronous operations take time.

Examples:

- HTTP requests
- Database queries
- Reading files
- Waiting for another coroutine
- Waiting for an external event

Instead of blocking,

the coroutine waits on a Future.

```text
Coroutine

↓

await Future

↓

Paused

↓

Event Loop

↓

Future completed

↓

Resume coroutine
```

---

# Important

A Future **does not execute code**.

It only stores:

- a result
- an exception
- cancellation state

Something else must complete it.

---

# Future Lifecycle

```text
           create_future()

                 │

                 ▼

             PENDING

       ┌────────┴────────┐

       ▼                 ▼

set_result()     set_exception()

       │                 │

       ▼                 ▼

            FINISHED
```

Possible states:

- Pending
- Finished
- Cancelled

---

# Creating a Future

```python
import asyncio

async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()

    print(future)

asyncio.run(main())
```

Output

```text
<Future pending>
```

At this point,

- no work is running
- no value exists
- nothing will happen until someone completes it

---

# Completing a Future

```python
future.set_result("Hello")
```

Now

```python
result = await future
```

returns immediately.

Example

```python
future.set_result("Done")

print(await future)
```

Output

```text
Done
```

---

# Setting an Exception

Instead of returning a value,

a Future may fail.

```python
future.set_exception(ValueError("Boom"))
```

Later

```python
await future
```

raises

```text
ValueError("Boom")
```

---

# Cancelling a Future

```python
future.cancel()
```

Now

```python
await future
```

raises

```python
asyncio.CancelledError
```

Useful when shutting down tasks or timing out operations.

---

# Waiting for a Future

Suppose

```python
future = loop.create_future()

await future
```

Nothing resumes it.

The coroutine waits forever.

```text
Future

Pending

Pending

Pending

...
```

Someone must call

```python
future.set_result(...)
```

or

```python
future.set_exception(...)
```

---

# Producer–Consumer Example

```python
import asyncio

async def producer(future):
    await asyncio.sleep(2)
    future.set_result("Finished")

async def consumer(future):
    result = await future
    print(result)

async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()

    await asyncio.gather(
        producer(future),
        consumer(future)
    )

asyncio.run(main())
```

Timeline

```text
Consumer

↓

await future

↓

Paused

Producer

↓

sleep(2)

↓

set_result()

↓

Consumer resumes

↓

Finished
```

---

# Relationship with Task

A Task is built on top of Future.

```text
Awaitable

│

├── Coroutine

├── Future

└── Task
```

More precisely

```text
Task
    │
inherits
    ▼
Future
```

That means every Task is also a Future.

```python
task = asyncio.create_task(work())
```

You can

```python
await task
```

because Task inherits Future.

---

# Future vs Task

## Future

Represents

> "A result will exist later."

It does **not** run code.

```python
future = loop.create_future()
```

---

## Task

Represents

> "A coroutine is currently running."

```python
task = asyncio.create_task(worker())
```

The Task executes the coroutine and completes itself automatically.

---

Comparison

| Future | Task |
|----------|------|
| Placeholder | Running coroutine |
| Doesn't execute code | Executes coroutine |
| Someone else completes it | Completes automatically |
| Low-level primitive | High-level API |
| Rarely created manually | Used constantly |

---

# How the Event Loop Uses Futures

Suppose

```python
await future
```

Internally

```text
Coroutine

↓

Checks Future

↓

Pending?

↓

Yes

↓

Pause coroutine

↓

Register callback

↓

Run other tasks

↓

Future completed

↓

Move coroutine to Ready Queue

↓

Resume coroutine
```

The event loop watches the Future.

Once it finishes,

the coroutine continues.

---

# Futures Power Async Libraries

Most async libraries internally create Futures.

Examples

```python
await websocket.receive_text()

await database.fetch()

await http_client.get()

await asyncio.sleep()
```

Internally,

they suspend the coroutine until a Future is completed by the event loop or an I/O callback.

---

# When should you create a Future manually?

Almost never.

Typical application code uses

```python
await

create_task()

gather()

TaskGroup()
```

Manual Futures are mainly useful for:

- building async libraries
- wrapping callback-based APIs
- integrating external event systems
- implementing synchronization primitives

---

# Common Methods

## Create

```python
loop.create_future()
```

---

## Complete

```python
future.set_result(value)
```

---

## Raise Error

```python
future.set_exception(exc)
```

---

## Cancel

```python
future.cancel()
```

---

## Await

```python
await future
```

---

## Check State

```python
future.done()

future.cancelled()
```

---

## Get Result

```python
future.result()
```

Raises an exception if the Future failed.

---

# Future State Machine

```text
                create_future()

                       │

                       ▼

                 Pending Future

             ┌─────────┼──────────┐

             ▼         ▼          ▼

     set_result   set_exception   cancel

             │         │          │

             ▼         ▼          ▼

          Finished  Finished   Cancelled

             │

             ▼

      await returns result
```

---

# When You Encounter Futures

You'll indirectly use Futures whenever you write:

```python
await asyncio.sleep()

await websocket.receive_text()

await redis.get()

await database.fetch()

await asyncio.gather(...)

await task
```

Although you rarely create them yourself,

they are one of the core building blocks of asyncio.

---

# Key Takeaways

- `Future` represents a **result that will exist later**.
- It **does not execute work**.
- Coroutines **await Futures**.
- Another task or the event loop **completes the Future**.
- A **Task is a subclass of Future** that executes a coroutine.
- Most application developers **rarely create Futures manually**.
- Futures are primarily used by **async frameworks and library authors**.

---

# Mental Model

```text
                    Event Loop
                         │
          ┌──────────────┴──────────────┐
          │                             │
          ▼                             ▼

    Producer Task                 Consumer Task

          │                             │
          │ set_result()                │ await future
          ▼                             │
      +-------------------------------+  │
      |           Future              |◄─┘
      |                               |
      |  Pending → Finished           |
      +-------------------------------+
                    │
                    ▼
          Resume Waiting Coroutine
```

> **Remember:**
>
> - **Coroutine** → defines work.
> - **Task** → runs work.
> - **Future** → stores the eventual result of that work.
> - **Event Loop** → coordinates everything.