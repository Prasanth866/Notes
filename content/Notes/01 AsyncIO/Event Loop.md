
> [!summary]
> The **Event Loop** is the **core scheduler of `asyncio`**. It manages all asynchronous execution by deciding **which task runs, which task waits, and when paused tasks resume**.
>
> **Remember:**
>
> > **Coroutines describe work → Tasks execute work → The Event Loop schedules work.**

---

# Why is the Event Loop Needed?

Without an event loop, every operation runs sequentially.

```
Download A (3s)

↓

Download B (2s)

↓

Download C (1s)

Total = 6s
```

With an event loop:

```
Start A
Start B
Start C

↓

A waiting

↓

Run B

↓

B waiting

↓

Run C

↓

Resume C

↓

Resume B

↓

Resume A

Total ≈ 3s
```

Instead of waiting idly, the event loop switches to another ready task.

---

# What is an Event Loop?

The event loop is a scheduler that continuously:

1. Executes ready tasks.
2. Pauses tasks waiting for I/O or timers.
3. Monitors external events.
4. Resumes tasks when they become ready.
5. Repeats until no work remains.

Conceptually:

```python
while True:
    run_ready_tasks()
    wait_for_events()
    move_ready_tasks_back()
```

You never write this loop yourself.

`asyncio.run()` creates and manages it automatically.

---

# The Core Components

```
async def
      │
      ▼
Coroutine
      │
create_task()
      ▼
Task
      │
Managed By
      ▼
Event Loop
      │
Runs Until await
      ▼
Waiting
      │
Ready Again
      ▼
Running
```

Everything in `asyncio` revolves around this relationship.

---

# Event Loop Lifecycle

When you write

```python
asyncio.run(main())
```

Python roughly performs

```
Create Event Loop

        │

Schedule main()

        │

Run Event Loop

        │

main() completes

        │

Cancel remaining tasks (if any)

        │

Shutdown async generators

        │

Shutdown default executor

        │

Close Event Loop
```

> [!important]
> `asyncio.run()` should normally be called **once** at the top level of your application.

---

# How a Task Executes

Example

```python
async def worker():
    print("Start")

    await asyncio.sleep(2)

    print("End")
```

Execution

```
Event Loop

↓

Run worker()

↓

print("Start")

↓

await sleep()

↓

Worker Pauses

↓

Run Other Ready Tasks

↓

2 Seconds Later

↓

Resume Worker

↓

print("End")
```

Notice:

> [!note]
> The **event loop never sleeps**.
>
> Only the coroutine pauses.

---

# What Happens at `await`?

This is the most important concept in asyncio.

When Python reaches

```python
await something()
```

The coroutine says:

> "I can't continue until this operation finishes."

The event loop then:

```
Current Task

↓

Suspended

↓

Save Current State

↓

Run Another Ready Task

↓

Operation Completes

↓

Restore State

↓

Continue Execution
```

> [!tip]
> `await` is a **cooperative yield point**.
>
> It voluntarily gives control back to the event loop.

---

# Task States

```
Coroutine

↓

Task Created

↓

Ready Queue

↓

Running

↓

await

↓

Waiting Queue

↓

Operation Completes

↓

Ready Queue

↓

Running

↓

Finished
```

The event loop constantly moves tasks between **Ready** and **Waiting**.

---

# Ready Queue vs Waiting Queue

## Ready Queue

```
Task A

Task B

Task C
```

Task A executes

```python
await asyncio.sleep(5)
```

Now

```
Waiting Queue

Task A
```

Ready Queue becomes

```
Task B

Task C
```

The event loop immediately switches to Task B.

Five seconds later

```
Task A

↓

Ready Queue
```

It becomes eligible to run again.

---

# Example: Multiple Tasks

```python
await asyncio.gather(
    task("A", 3),
    task("B", 2),
    task("C", 1)
)
```

Timeline

```
A Starts
B Starts
C Starts

↓

All Reach await

↓

1 Second

Resume C

↓

2 Seconds

Resume B

↓

3 Seconds

Resume A
```

Tasks are resumed based on **when they become ready**, not in creation order.

---

# Event Loop Scheduling

Example

```python
async def A():
    print("A1")
    await asyncio.sleep(2)
    print("A2")

async def B():
    print("B1")
    await asyncio.sleep(1)
    print("B2")
```

Execution

```
Run A

↓

A1

↓

await

↓

Run B

↓

B1

↓

await

↓

1 Second

↓

B2

↓

2 Seconds

↓

A2
```

The event loop always picks the next ready task.

---

# Event Loop and `create_task()`

```python
t1 = asyncio.create_task(download())
t2 = asyncio.create_task(upload())
```

Internally

```
Event Loop

Ready Queue

├── Download
├── Upload
```

Scheduler loop

```
Pick Ready Task

↓

Execute

↓

Reached await?

↓

Yes

↓

Pause

↓

Pick Next Ready Task
```

---

# Event Loop and `gather()`

```python
await asyncio.gather(
    task1(),
    task2(),
    task3()
)
```

Conceptually

```
Create Tasks

↓

Register With Event Loop

↓

Run Concurrently

↓

Wait Until All Finish

↓

Return Results
```

The event loop manages all created tasks.

---

# Event Loop and `TaskGroup`

```python
async with asyncio.TaskGroup() as tg:
    tg.create_task(A())
    tg.create_task(B())
```

Execution

```
Register Tasks

↓

Run Concurrently

↓

One Task Fails?

├── No → Continue
└── Yes → Cancel Siblings

↓

Exit TaskGroup

↓

Continue
```

The event loop enforces the scheduling while `TaskGroup` manages task lifetime.

---

# What Does the Event Loop Manage?

The event loop keeps track of

```
Ready Tasks

Waiting Tasks

Timers

Socket Events

File I/O Events

Callbacks

Futures

Completed Tasks
```

Everything asynchronous flows through it.

---

# Single-Threaded Scheduling

A common misconception is that asyncio runs multiple coroutines simultaneously.

Reality

```
Task A

↓

Task B

↓

Task C

↓

Task A

↓

Task B
```

Only **one coroutine executes Python code at a time** within a single event loop.

> [!important]
> Asyncio provides **concurrency**, not CPU parallelism.

---

# Event Loop vs Threads

| Event Loop | Threads |
|------------|----------|
| One thread | Multiple OS threads |
| Cooperative (`await`) | OS preemptive scheduling |
| Lightweight | Heavier |
| Excellent for I/O | Better for CPU-intensive work |
| Thousands of tasks | Limited by thread overhead |

---

# Event Loop vs CPU Work

Bad

```python
async def heavy():
    for _ in range(100_000_000):
        pass
```

There is no

```python
await
```

Result

```
Heavy Task

↓

CPU Busy

↓

Everything Else Waits
```

The event loop cannot switch tasks.

For CPU-intensive work use:

- `asyncio.to_thread()`
- `loop.run_in_executor()`
- `ProcessPoolExecutor`
- Multiprocessing

---

# Common Mistakes

> [!warning]
> Avoid these mistakes

### ❌ Using `time.sleep()`

```python
time.sleep(2)
```

Blocks the entire event loop.

Use

```python
await asyncio.sleep(2)
```

---

### ❌ Long CPU loops without `await`

They block every other coroutine.

---

### ❌ Thinking `await` blocks the whole program

It only pauses the current coroutine.

---

### ❌ Creating tasks but never awaiting or managing them

This can leave orphaned tasks or unhandled exceptions.

Prefer `TaskGroup` for related tasks.

---

# Real-World Analogy

Imagine an airport control tower.

```
                 Event Loop
          (Air Traffic Controller)

                      │
      ┌───────────────┼───────────────┐
      ▼               ▼               ▼

   Flight A       Flight B       Flight C
  (Coroutine)    (Coroutine)    (Coroutine)

      │               │               │

   Waiting         Ready          Waiting

      │               │               │

Controller asks continuously:

"Who is ready?"

↓

Assign Runway

↓

Next Flight
```

The controller never flies the planes.

It simply decides who gets the runway next.

The event loop works exactly the same way.

---

# Complete Flow

```
asyncio.run()

        │

Create Event Loop

        │

Schedule main()

        │

main()

        │

gather() / TaskGroup

        │

Create Tasks

        │

Register Tasks

        │

Run Task

        │

Task reaches await

        │

Pause Task

        │

Run Next Ready Task

        │

Operation Completes

        │

Resume Task

        │

All Tasks Complete

        │

Shutdown Remaining Resources

        │

Close Event Loop
```

---

# Quick Summary

> [!tip]
> - The Event Loop is the scheduler behind every asyncio program.
> - It runs tasks until they hit an `await`.
> - Waiting tasks are paused; ready tasks continue.
> - `await` yields control back to the event loop.
> - Only one coroutine executes Python code at a time.
> - Concurrency comes from switching between waiting tasks.
> - Ideal for I/O-bound workloads, not CPU-bound computation.

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Start async program | `asyncio.run()` |
| Schedule coroutine | `asyncio.create_task()` |
| Run many coroutines | `asyncio.gather()` |
| Structured concurrency | `asyncio.TaskGroup()` |
| Pause without blocking | `await asyncio.sleep()` |
| Yield immediately | `await asyncio.sleep(0)` |
| CPU-intensive work | `asyncio.to_thread()` / executors / processes |
| Scheduler | **Event Loop** |

---

# Mental Model

```
Coroutine
     │
     ▼
Task
     │
     ▼
Event Loop
     │
     ├── Run Ready Tasks
     ├── Pause Waiting Tasks
     ├── Watch I/O
     ├── Resume Completed Tasks
     └── Repeat
```

> **Think of the Event Loop as an operating system scheduler for asynchronous code.** It doesn't perform the work itself—it continuously decides **who gets CPU time next**, allowing thousands of I/O-bound operations to be handled efficiently on a single thread.