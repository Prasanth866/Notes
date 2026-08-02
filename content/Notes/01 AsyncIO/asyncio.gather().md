
> [!summary]
> `asyncio.gather()` is a high-level function that **runs multiple coroutines concurrently**, **waits for all of them to finish**, and **returns their results in the same order they were passed**.

> **Remember:**
>
> - `await` → Run one coroutine.
> - `create_task()` → Start one coroutine concurrently.
> - `gather()` → Start **multiple coroutines together** and collect all their results.

---

# Why use `asyncio.gather()`?

Suppose you need to fetch:

- User profile
- Orders
- Notifications

Sequential approach:

```python
await get_user()
await get_orders()
await get_notifications()
```

Execution:

```
User
 │
 ▼
Orders
 │
 ▼
Notifications
```

Everything waits.

---

Using `gather()`:

```python
await asyncio.gather(
    get_user(),
    get_orders(),
    get_notifications()
)
```

Execution:

```
User
Orders
Notifications
        │
        ▼
Run Together
        │
        ▼
Wait for All
```

The total runtime is approximately the time of the **slowest** coroutine.

---

# Syntax

```python
results = await asyncio.gather(
    coro1(),
    coro2(),
    coro3()
)
```

Or

```python
results = await asyncio.gather(*coroutines)
```

Returns

```python
list_of_results
```

where each result corresponds to the coroutine at the same position.

---

# Sequential vs Concurrent

## Sequential

```python
await work("A", 3)
await work("B", 2)
await work("C", 1)
```

Timeline

```
A starts
(wait 3)

A finishes

B starts
(wait 2)

B finishes

C starts
(wait 1)

C finishes
```

Total ≈ **6 seconds**

---

## Concurrent (`gather()`)

```python
await asyncio.gather(
    work("A", 3),
    work("B", 2),
    work("C", 1)
)
```

Timeline

```
A starts
B starts
C starts

(wait 1)

C finishes

(wait 1)

B finishes

(wait 1)

A finishes
```

Total ≈ **3 seconds**

---

# How `gather()` Works Internally

Conceptually,

```python
await asyncio.gather(
    work("A"),
    work("B"),
    work("C")
)
```

does something like:

```
Coroutine A
Coroutine B
Coroutine C
      │
      ▼
Create Tasks
      │
      ▼
Run Concurrently
      │
      ▼
Wait Until All Complete
      │
      ▼
Return Results
```

You don't need to call `create_task()` manually.

---

# Return Values

```python
async def square(x):
    await asyncio.sleep(1)
    return x * x
```

```python
results = await asyncio.gather(
    square(2),
    square(3),
    square(4)
)

print(results)
```

Output

```
[4, 9, 16]
```

All three calculations finish together.

---

# Results Preserve Input Order

Even if coroutines finish in different orders,

```python
results = await asyncio.gather(
    square(3),
    square(1),
    square(2)
)
```

Output

```
[9, 1, 4]
```

> [!important]
> Results are returned in the **order of the arguments**, **not** the order in which the coroutines complete.

---

# Execution Flow

```
gather()

      │

Start Task A
Start Task B
Start Task C

      │

Run Concurrently

      │

Task C finishes
Task B finishes
Task A finishes

      │

Return Results
```

---

# Example: Downloading Files

```python
files = await asyncio.gather(
    download("A", 3),
    download("B", 2),
    download("C", 1)
)

print(files)
```

Output

```
Downloading A
Downloading B
Downloading C

C downloaded
B downloaded
A downloaded

['A', 'B', 'C']
```

Downloads finish independently, but results remain ordered.

---

# Mixing Different Coroutines

```python
user, orders, notifications = await asyncio.gather(
    get_user(),
    get_orders(),
    get_notifications()
)
```

Each coroutine can return a different type.

Example:

```
User

["Book", "Phone"]

10
```

Perfect for fetching unrelated data simultaneously.

---

# Error Handling

Suppose

```python
await asyncio.gather(
    good(),
    bad()
)
```

where

```python
bad()
```

raises

```python
ValueError
```

Result

```
ValueError
```

The **first exception** is propagated to the caller.

---

# `return_exceptions=True`

Sometimes you want every coroutine to complete, even if some fail.

```python
results = await asyncio.gather(
    good(),
    bad(),
    return_exceptions=True
)
```

Output

```
[
    "Good",
    ValueError("Boom")
]
```

Now you can inspect each result individually.

```python
for result in results:
    if isinstance(result, Exception):
        ...
```

> [!tip]
> Useful when partial success is acceptable.

---

# Running Many Coroutines

```python
tasks = []

for i in range(10):
    tasks.append(square(i))

results = await asyncio.gather(*tasks)
```

The `*` operator expands the list into separate arguments.

---

# `gather()` vs `create_task()`

## `create_task()`

```python
task = asyncio.create_task(download())

# Do other work...

await task
```

You manually manage:

- Task references
- Awaiting
- Cancellation
- Monitoring

---

## `gather()`

```python
await asyncio.gather(
    download(),
    upload()
)
```

Everything is managed for you.

Best when:

- Start multiple coroutines together.
- Wait for all.
- Collect all results.

---

# `gather()` vs `TaskGroup`

## `gather()`

```python
results = await asyncio.gather(
    download(),
    upload()
)
```

Advantages

- Very concise
- Returns results directly
- Excellent for independent work

---

## `TaskGroup`

```python
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(download())
    t2 = tg.create_task(upload())
```

Advantages

- Structured concurrency
- Better lifecycle management
- Automatically cancels sibling tasks
- Safer cleanup

> [!tip]
> Use **TaskGroup** when tasks belong to the same logical operation.

---

# When Should You Use `gather()`?

Use it when:

- Multiple independent coroutines.
- You want everything to start immediately.
- You need all return values.
- No complex task management is required.

Common examples:

- Multiple API requests
- Reading several files
- Downloading multiple pages
- Database queries
- Independent computations

---

# Common Pitfalls

> [!warning]
> Avoid these mistakes

❌ Expecting results in completion order.

```python
gather(A, B, C)
```

Returns

```
[A_result, B_result, C_result]
```

even if C finishes first.

---

❌ Forgetting to unpack a list.

Wrong

```python
await asyncio.gather(tasks)
```

Correct

```python
await asyncio.gather(*tasks)
```

---

❌ Using `gather()` for long-running background tasks.

Use `create_task()` instead.

---

❌ Using `gather()` when tasks should automatically cancel together on failure.

Prefer `TaskGroup`.

---

# Mental Model

Imagine ordering food.

Without `gather()`

```
Order Pizza

Wait

Order Burger

Wait

Order Dessert
```

Total = Sum of all preparation times.

---

With `gather()`

```
Order Pizza
Order Burger
Order Dessert

All kitchens cook simultaneously

Wait

Everything arrives together
```

Total = Time of the slowest dish.

---

# Quick Summary

> [!tip]
> - Runs multiple coroutines concurrently.
> - Automatically schedules them.
> - Waits until all finish.
> - Returns results in input order.
> - Excellent for independent concurrent operations.
> - Supports `return_exceptions=True`.
> - Simpler than manually creating tasks.
> - Prefer `TaskGroup` for structured concurrency in Python 3.11+.

---

# `await` vs `create_task()` vs `gather()` vs `TaskGroup`

| Feature | `await` | `create_task()` | `gather()` | `TaskGroup` |
|---------|---------|-----------------|------------|-------------|
| Run one coroutine | ✅ | ✅ | ❌ | ❌ |
| Run multiple concurrently | ❌ | ✅ | ✅ | ✅ |
| Returns results directly | ✅ | Via `await task` | ✅ | Via `task.result()` |
| Starts immediately | When awaited | ✅ | ✅ | ✅ |
| Automatic task management | N/A | ❌ | Partial | ✅ |
| Cancels sibling tasks on failure | N/A | ❌ | ❌* | ✅ |
| Best for related concurrent work | ❌ | Sometimes | Good | ⭐ Best |

> *Exception propagation and cancellation behavior in `gather()` differ from `TaskGroup`.

---

# Cheat Sheet

| Need | Use |
|------|-----|
| Run one coroutine | ✅ `await` |
| Start one coroutine in background | ✅ `create_task()` |
| Run many coroutines and collect results | ✅ `gather()` |
| Run related concurrent tasks safely | ✅ `TaskGroup` |
| Get results in input order | ✅ `gather()` |
| Handle partial failures | ✅ `gather(return_exceptions=True)` |
| Manage individual task lifecycle | ✅ `create_task()` |
| Structured concurrency | ✅ `TaskGroup` |