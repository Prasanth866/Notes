---
title: "Non-blocking I/O Mechanics"
description: "Explaining non-blocking file descriptors, OS I/O multiplexing primitives (epoll, kqueue, select), and single-threaded event loop throughput."
tags:
  - python/asyncio
  - networking
  - system
aliases:
  - Non-blocking IO
  - Non-blocking I/O
---

# Non-blocking I/O Mechanics

> [!summary]
> **Non-blocking I/O** is an OS-level mechanism where system calls (like `read()` or `write()`) on sockets or file descriptors return immediately rather than stalling the calling thread until data arrives.
>
> Non-blocking I/O is the core foundation enabling single-threaded event loops to handle tens of thousands of concurrent connections.

---

## Synchronous Blocking I/O vs ⚡ Non-blocking I/O

### 1. Blocking I/O (Traditional)

In blocking mode, calling `socket.recv()` halts thread execution completely until data arrives over the network interface card (NIC):

```
Thread  ─────> socket.recv() ─────> [BLOCKED WAITING ON OS NETWORK BUFFER] ─────> Data Returns
```

To handle multiple clients, synchronous servers must spawn thousands of OS threads, incurring severe context-switching overhead and memory consumption (typically ~1MB to 8MB per thread stack).

### 2. Non-blocking I/O (Event Loop)

In non-blocking mode (`socket.setblocking(False)`), system calls return immediately with an `EWOULDBLOCK` or `EAGAIN` status if no data is available:

```
Event Loop  ───> socket.recv()  ───> Returns EWOULDBLOCK immediately
Event Loop  ───> Polls OS Selector (epoll/kqueue) ───> Registers event callback
Event Loop  ───> Executes other tasks ───> OS signals socket READY ───> Reads data
```

---

## OS I/O Multiplexing Primitives

Operating systems provide kernel-level event notification interfaces to monitor thousands of file descriptors simultaneously:

| OS Platform        | Multiplexing Primitive        | Algorithmic Complexity            |
| :----------------- | :---------------------------- | :-------------------------------- |
| **Linux**          | `epoll`                       | $O(1)$ event notifications        |
| **macOS / BSD**    | `kqueue`                      | $O(1)$ event notifications        |
| **Windows**        | `IOCP` (I/O Completion Ports) | Asynchronous I/O Overlapped Model |
| **POSIX Fallback** | `select` / `poll`             | $O(N)$ scanning cost              |

The **[[Notes/AsyncIO/Event Loop|Event Loop]]** selects the optimal OS primitive automatically using Python's `selectors` module.

---

## Low-Level Python Example

Here is how non-blocking sockets and OS selectors work under the hood without high-level `asyncio`:

```python
import selectors
import socket

selector = selectors.DefaultSelector()

def accept_connection(server_sock: socket.socket):
    conn, addr = server_sock.accept()
    print(f"Accepted client connection from {addr}")
    conn.setblocking(False)  # Configure non-blocking socket mode
    # Register connection with selector for READ events
    selector.register(conn, selectors.EVENT_READ, read_callback)

def read_callback(conn: socket.socket):
    data = conn.recv(1024)
    if data:
        conn.send(b"HTTP/1.1 200 OK\r\nContent-Length: 2\r\n\r\nOK")
    selector.unregister(conn)
    conn.close()

# Initialize non-blocking server socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('127.0.0.1', 8080))
server.listen()
server.setblocking(False)

selector.register(server, selectors.EVENT_READ, accept_connection)

print("Non-blocking server running on port 8080...")
# Event Loop polling loop
while True:
    events = selector.select()
    for key, mask in events:
        callback = key.data
        callback(key.fileobj)
```

---

## How AsyncIO Builds on Non-blocking I/O

`asyncio` wraps OS selectors into friendly `async`/`await` interfaces:

```python
import asyncio

async def handle_client(reader: asyncio.StreamReader, writer: asyncio.StreamWriter):
    data = await reader.read(100)  # Non-blocking yield to event loop
    writer.write(b"HTTP/1.1 200 OK\r\nContent-Length: 2\r\n\r\nOK")
    await writer.drain()
    writer.close()

async def main():
    server = await asyncio.start_server(handle_client, '127.0.0.1', 8080)
    async with server:
        await server.serve_forever()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Related Notes

- [[Notes/AsyncIO/Event Loop|Event Loop]] — The scheduler driven by non-blocking OS selectors
- [[Notes/AsyncIO/Coroutine|Coroutine]] — Pausable user code running atop non-blocking I/O
- [[Notes/FastAPI/WebSockets|FastAPI WebSockets]] — High-concurrency socket handling
- [[Notes/AsyncIO/index|AsyncIO Map of Content]]
