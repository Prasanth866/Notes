---
title: "ForgeHTTP — 40 Day Systems Engineering Roadmap"
tags:
  - cpp
  - networking
  - linux
  - operating-systems
  - systems-programming
  - http
  - portfolio
status: active
duration: 40-days
---

# ForgeHTTP

> [!ABSTRACT]
> A high-performance HTTP server and networking engine built from scratch in modern C++. This is a serious 40-day systems-engineering project designed to build deep, practical understanding of how networking and operating systems work underneath high-level web frameworks.

---

## 🗺 Project Vision

\`\`\`text
TCP Socket
    ↓
HTTP Server
    ↓
HTTP Parser
    ↓
Static File Server
    ↓
Concurrent Server
    ↓
Non-Blocking Server
    ↓
epoll Event Loop
    ↓
Event-Driven HTTP Server
    ↓
Router + Middleware
    ↓
Reverse Proxy
    ↓
Observability
    ↓
Performance Engineering
    ↓
Unique Innovation
\`\`\`

> [!NOTE]
> The goal is not to finish the fastest. The goal is to understand deeply — then implement — then experiment — then benchmark — then document.

---

## 🎯 Learning Objectives

- [ ] Understand [[TCP/IP]] from the socket layer upward
- [ ] Build an HTTP/1.1 parser from scratch, handling partial reads, framing, and edge cases
- [ ] Implement concurrency from single-threaded to thread pool to event-driven [[epoll]]
- [ ] Use modern [[C++]] features: RAII, smart pointers, move semantics, `std::string_view`, atomics
- [ ] Understand [[Linux]] internals: file descriptors, system calls, kernel/user space boundary
- [ ] Build a production-inspired reverse proxy with load balancing
- [ ] Profile, benchmark, and optimize using `wrk`, `perf`, and flamegraphs
- [ ] Package the project professionally for a strong GitHub portfolio

---

## 🏗 Final Feature Set

| Feature | Status |
|---|---|
| TCP server with socket lifecycle | - [ ] |
| HTTP/1.1 parser (FSM, incremental) | - [ ] |
| Static file server with MIME, ETag, Range | - [ ] |
| Thread pool concurrency model | - [ ] |
| epoll-based non-blocking event loop | - [ ] |
| Router with params and wildcards | - [ ] |
| Middleware pipeline | - [ ] |
| Reverse proxy with round-robin LB | - [ ] |
| Observability: /metrics, /debug/stats | - [ ] |
| Security: Slowloris, path traversal, limits | - [ ] |
| Performance profiling + flamegraphs | - [ ] |
| Unit + integration + fuzz tests | - [ ] |
| Dockerfile + GitHub Actions CI | - [ ] |

---

## 🧱 Architecture Evolution

\`\`\`mermaid
flowchart TD
    A[Day 1-2: C++ and Linux Foundations] --> B[Day 3-5: TCP Echo Server]
    B --> C[Day 6-10: HTTP/1.1 Parser]
    C --> D[Day 11-13: Robust FSM Parser]
    D --> E[Day 14-16: Static File Server]
    E --> F[Day 17-19: Thread Pool Concurrency]
    F --> G[Day 20-23: Non-Blocking + epoll]
    G --> H[Day 24-27: Reactor Event Loop]
    H --> I[Day 28-29: Production HTTP Features]
    I --> J[Day 30: Router + Middleware]
    J --> K[Day 31-33: Reverse Proxy]
    K --> L[Day 34: Observability]
    L --> M[Day 35-36: Performance Engineering]
    M --> N[Day 37-38: Innovation / Backpressure]
    N --> O[Day 39-40: Testing + Productionization]
\`\`\`

---

## 📅 40-Day Roadmap Overview

| Phase | Days | Title |
|---|---|---|
| 0 | 1–2 | Foundations: C++ & Linux |
| 1 | 3–5 | TCP Socket Programming |
| 2 | 6–10 | HTTP/1.1 Parsing |
| 3 | 11–13 | Robust FSM Parser |
| 4 | 14–16 | Static File Server |
| 5 | 17–19 | Concurrency Models |
| 6 | 20–23 | Non-Blocking I/O & epoll |
| 7 | 24–27 | Event-Driven Reactor Architecture |
| 8 | 28–29 | Production HTTP Features |
| 9 | 30 | Router + Middleware |
| 10 | 31–33 | Reverse Proxy |
| 11 | 34 | Observability |
| 12 | 35–36 | Performance Engineering |
| 13 | 37–38 | Innovation: Backpressure |
| 14 | 39–40 | Testing + Productionization |

---

## Phase 0 — Foundations

### Day 1 — C++ Systems Foundations

#### 🎯 Goal
Build a minimal but solid working knowledge of the C++ features you will use throughout the project. Write small experiments, not just read.

#### 📚 Concepts

- [[RAII]] — Resource Acquisition Is Initialization
- [[Smart Pointers]] — `unique_ptr`, `shared_ptr`, `weak_ptr`
- [[Move Semantics]] — rvalue references, `std::move`
- [[std::string_view]] — zero-copy string references
- [[STL Containers]] — `vector`, `unordered_map`, `deque`, `array`
- [[Lambdas]] — closures, captures
- [[Error Handling]] — exceptions vs error codes vs `std::expected`
- [[CMake]] — modern build system

#### 💻 Implementation

- [ ] Set up CMake project with `CMakeLists.txt`, build directory, and `.clang-format`
- [ ] Write a `Connection` class that wraps a file descriptor using RAII (`close()` in destructor)
- [ ] Write a `Buffer` class using `std::vector<char>` with move semantics
- [ ] Write a function that accepts `std::string_view` and does zero-copy string slicing
- [ ] Write a lambda-based callback dispatcher (preview of the event loop)
- [ ] Compile with `-Wall -Wextra -fsanitize=address`

#### 🧪 Experiment

Write two versions of a `Buffer` class: one that copies strings, one that uses `std::string_view`. Measure the performance difference with a micro-benchmark using `std::chrono`.

#### 🔬 What to Observe

- How much time is spent on memory allocation with copying?
- What does `std::move` actually do at the assembly level? (Use Compiler Explorer / godbolt.org)
- What happens if you access a `string_view` after the underlying string is destroyed?

#### 🧠 Questions

- [ ] What is RAII and why does it matter for networking code?
- [ ] What is the difference between `unique_ptr` and `shared_ptr`?
- [ ] Why is `std::string_view` preferred for parsing?
- [ ] What does `std::move` actually do at the hardware level?

#### 🧪 Tests

- [ ] Test that `Connection` destructor calls `close()` exactly once (check with `strace`)
- [ ] Test that the `Buffer` correctly handles move construction

#### 📊 Benchmark

Compare copy-based string slicing vs `string_view` slicing for 100,000 iterations.

#### ✅ Definition of Done

- [ ] CMake project compiles cleanly with sanitizers enabled
- [ ] RAII `Connection` class exists and is tested
- [ ] `Buffer` class supports move semantics
- [ ] You can explain RAII, move semantics, and `string_view` without notes

#### 📝 Git Commit

`feat: project scaffold, RAII Connection, Buffer with move semantics`

---

### Day 2 — Linux Internals & File Descriptors

#### 🎯 Goal
Understand the OS substrate on which all networking code runs. Every socket is a file descriptor. Every `recv()` is a system call. You need to know this.

#### 📚 Concepts

- [[File Descriptors]] — integers that represent kernel resources
- [[System Calls]] — the boundary between user space and kernel space
- [[Linux Processes]] — fork, exec, PID, address space
- [[Linux Threads]] — POSIX threads, kernel threads, `clone()`
- [[Virtual Memory]] — address space, pages, page faults
- [[Context Switching]] — when and why the kernel preempts
- [[User Space vs Kernel Space]] — privilege levels, system call overhead

#### 💻 Implementation

- [ ] Write a C++ program that opens a file, reads it, and closes it — trace with `strace`
- [ ] Write a program that forks a child process and shares a pipe between them
- [ ] Write a `std::thread` producer/consumer using `std::mutex` and `std::condition_variable`
- [ ] Use `/proc/self/fd` to inspect open file descriptors at runtime
- [ ] Use `ulimit -n` to observe the system's file descriptor limit

#### 🧪 Experiment

Run `strace -c ./your_program` on a simple file-reading program. Count how many system calls happen. Then open a socket and count again.

#### 🔬 What to Observe

- What is the overhead of a system call?
- How many system calls does `printf()` trigger?
- What file descriptors does every process start with? (0, 1, 2)
- What does `/proc/<pid>/maps` show?

#### 🧠 Questions

- [ ] Why is a socket represented as a file descriptor?
- [ ] What happens at the CPU level when you call `read()`?
- [ ] What is the difference between a process and a thread at the kernel level?
- [ ] What is a context switch and what does it cost?

#### 🧪 Tests

- [ ] Verify that your producer/consumer works without deadlock under 10,000 iterations
- [ ] Verify that file descriptors are correctly closed (use `lsof -p <pid>`)

#### 📊 Benchmark

Measure the time cost of one `getpid()` system call using `clock_gettime(CLOCK_MONOTONIC)` in a tight loop (1,000,000 iterations).

#### ✅ Definition of Done

- [ ] You can read `/proc/self/fd` and explain every entry
- [ ] `strace` output for a simple program is readable and understandable
- [ ] Producer/consumer implementation works without data races (verified with ThreadSanitizer)
- [ ] You can explain user/kernel space boundary confidently

#### 📝 Git Commit

`feat: linux fundamentals experiments, producer-consumer, strace analysis`

---

## Phase 1 — TCP Socket Programming

### Day 3 — TCP Echo Server

#### 🎯 Goal
Write a real TCP server and client from scratch using POSIX socket APIs. Understand the full socket lifecycle.

#### 📚 Concepts

- [[TCP]] — Transmission Control Protocol
- [[Socket Programming]] — `socket()`, `bind()`, `listen()`, `accept()`
- [[TCP Handshake]] — SYN, SYN-ACK, ACK
- [[IP Addressing]] — `sockaddr_in`, `inet_pton`, `htons`
- [[Berkeley Sockets]] — the POSIX socket API

#### 💻 Implementation

- [ ] Create `tcp_server.cpp` that: `socket()` → `setsockopt(SO_REUSEADDR)` → `bind()` → `listen()` → `accept()` → `recv()/send()` loop
- [ ] Wrap each system call in a checked function that throws on error
- [ ] Create `tcp_client.cpp` that connects and sends data
- [ ] Test with `nc localhost 8080`
- [ ] Observe with `ss -tnp` and `lsof -i :8080`

#### 🧪 Experiment

Use `tcpdump -i lo port 8080` to capture packets while connecting a client. Observe the 3-way handshake (SYN, SYN-ACK, ACK) and the FIN-ACK teardown.

#### 🔬 What to Observe

- What does the socket file descriptor number look like? (usually 3 or 4)
- What is in the `accept()` return value?
- What happens in `ss` before and after a client connects?
- Can you see the TCP state machine: LISTEN → SYN_RCVD → ESTABLISHED → TIME_WAIT?

#### 🧠 Questions

- [ ] What happens inside the kernel when `socket()` is called?
- [ ] What does `listen(backlog)` do and what is the backlog queue?
- [ ] Why does TCP use a 3-way handshake?
- [ ] What is `SO_REUSEADDR` and why do you always need it?

#### 🧪 Tests

- [ ] Connect 5 clients simultaneously and verify all receive echo responses
- [ ] Kill client without closing — observe server behavior

#### 📊 Benchmark

Measure round-trip latency for echo with `time echo "hello" | nc localhost 8080`.

#### ✅ Definition of Done

- [ ] TCP echo server accepts multiple sequential connections
- [ ] TCP client sends and receives data correctly
- [ ] You can read the `tcpdump` output and identify the handshake
- [ ] `ss` output is fully understood

#### 📝 Git Commit

`feat: TCP echo server and client with full socket lifecycle`

---

### Day 4 — Multi-Client TCP Server

#### 🎯 Goal
Handle multiple clients simultaneously using `fork()` and then threads. Understand the difference.

#### 📚 Concepts

- [[TCP Concurrency]] — handling multiple connections
- [[fork()]] — process-per-connection model
- [[Linux Threads]] — thread-per-connection model
- [[SIGCHLD]] — zombie processes
- [[Socket Options]] — `SO_KEEPALIVE`, `TCP_NODELAY`

#### 💻 Implementation

- [ ] Implement process-per-connection using `fork()` (handle `SIGCHLD` to prevent zombies)
- [ ] Implement thread-per-connection using `std::thread`
- [ ] Implement a version that uses `std::jthread` (C++20)
- [ ] Add `TCP_NODELAY` to disable Nagle's algorithm
- [ ] Test both versions with 10 simultaneous `nc` clients

#### 🧪 Experiment

Open 50 connections simultaneously with a small shell script (`for i in {1..50}; do nc localhost 8080 & done`). Observe with `ss` and `top`. Compare memory usage of `fork` vs threads.

#### 🔬 What to Observe

- How much memory does each forked process use vs each thread?
- What happens when you don't handle `SIGCHLD`? (zombie processes in `ps aux`)
- What does `TCP_NODELAY` change in `tcpdump`?

#### 🧠 Questions

- [ ] What is the C10K problem?
- [ ] Why does fork-per-connection not scale?
- [ ] Why does thread-per-connection also not scale?
- [ ] What is Nagle's algorithm and why might you disable it?

#### 🧪 Tests

- [ ] Test that no zombie processes accumulate after 100 connections
- [ ] Test that all 50 simultaneous clients receive correct responses

#### 📊 Benchmark

Use `wrk -t4 -c50 -d10s http://localhost:8080/` against thread-per-connection. Record baseline req/sec.

#### ✅ Definition of Done

- [ ] Multi-client server handles 50 concurrent connections without crashing
- [ ] No zombie processes
- [ ] You can explain why neither fork nor thread-per-connection is the final answer

#### 📝 Git Commit

`feat: multi-client server with fork and thread-per-connection models`

---

### Day 5 — Hardcoded HTTP Response + Tools Mastery

#### 🎯 Goal
Send a minimal hardcoded HTTP/1.1 response so `curl` and a browser can display it. Master the debugging tools you will use every day.

#### 📚 Concepts

- [[HTTP]] — Hypertext Transfer Protocol basics
- [[HTTP Response Format]] — status line, headers, body
- [[curl]] — HTTP client for testing
- [[tcpdump]] — packet capture
- [[Wireshark]] — visual packet analysis
- [[ss]] / [[lsof]] — socket inspection tools

#### 💻 Implementation

- [ ] Modify the TCP server to send a hardcoded HTTP/1.1 200 OK response when any data arrives
- [ ] Test with `curl -v http://localhost:8080/`
- [ ] Test with a real browser
- [ ] Run `tcpdump -A -i lo port 8080` to see raw HTTP bytes in the terminal
- [ ] Capture a `.pcap` file and open in Wireshark

#### 🧪 Experiment

Use `curl -v` to see the full request/response. Use `tcpdump -A` to see the raw bytes. Find the exact HTTP request bytes that curl sends and the response bytes your server sends.

#### 🔬 What to Observe

- What exactly does curl send in the request?
- What are the exact bytes of your HTTP response?
- What is `\r\n` and why does HTTP use it?
- What is the blank line separating headers from body?

#### 🧠 Questions

- [ ] What is the minimal valid HTTP/1.1 response?
- [ ] Why does HTTP use `\r\n` (CRLF) as a line separator?
- [ ] What does `HTTP/1.1 200 OK\r\n\r\n` mean exactly?
- [ ] Why can a browser connect but then show a broken page?

#### 🧪 Tests

- [ ] `curl -v http://localhost:8080/` returns 200 OK
- [ ] Browser displays the response body

#### 📊 Benchmark

Not applicable today — focus on observation and tooling.

#### ✅ Definition of Done

- [ ] curl receives a valid HTTP/1.1 response
- [ ] You can use `tcpdump`, `ss`, `lsof`, and `curl` fluently
- [ ] You can open and read a `.pcap` file in Wireshark

#### 📝 Git Commit

`feat: hardcoded HTTP response, tools mastery (tcpdump, curl, wireshark)`

---

## Phase 2 — HTTP/1.1 Parsing

### Day 6 — HTTP Request Parser (Part 1)

#### 🎯 Goal
Write a basic HTTP request parser that can extract the method, URI, version, and headers from a raw byte buffer.

#### 📚 Concepts

- [[HTTP Parser]] — request line, headers, body
- [[HTTP Methods]] — GET, POST, HEAD, PUT, DELETE
- [[HTTP Headers]] — name: value pairs
- [[URI]] — Uniform Resource Identifier
- [[TCP Byte Stream]] — why `recv()` does not give you one clean HTTP request

#### 💻 Implementation

- [ ] Create `HttpRequest` struct: method, uri, version, headers (unordered_map), body
- [ ] Create `HttpParser` class with a `parse(std::string_view)` method
- [ ] Parse the request line: `METHOD URI HTTP/1.1\r\n`
- [ ] Parse headers: `Name: Value\r\n` repeated until `\r\n\r\n`
- [ ] Extract `Content-Length` header to know body size
- [ ] Return a `ParseResult` enum: `Complete`, `Incomplete`, `Error`

#### 🧪 Experiment

Send a POST request with a body using `curl -X POST -d "hello=world" http://localhost:8080/`. Print every byte your server receives, count the calls to `recv()`, and observe that one request may arrive in multiple `recv()` calls.

#### 🔬 What to Observe

- Does one `recv()` always give you the complete HTTP request?
- How does a 1MB POST body arrive in `recv()` chunks?
- What happens if the request line arrives in two separate `recv()` calls?

#### 🧠 Questions

- [ ] Why is TCP a byte stream and not a message stream?
- [ ] Why can `recv()` return partial data?
- [ ] Why does HTTP need `Content-Length` or chunked encoding?
- [ ] What is message framing and why is it the parser's job?

#### 🧪 Tests

- [ ] Unit test: parse a complete HTTP GET request from a string
- [ ] Unit test: parse a POST request with `Content-Length` and body
- [ ] Unit test: partial request returns `Incomplete`

#### 📊 Benchmark

Not applicable — focus on correctness.

#### ✅ Definition of Done

- [ ] Parser correctly handles GET with query params
- [ ] Parser correctly handles POST with body
- [ ] `ParseResult::Incomplete` returned for partial requests
- [ ] All unit tests pass

#### 📝 Git Commit

`feat: basic HTTP/1.1 request parser with method, URI, headers, body`

---

### Day 7 — HTTP Response Builder

#### 🎯 Goal
Build a response builder that generates valid HTTP/1.1 responses and integrate it with the server.

#### 📚 Concepts

- [[HTTP Response]] — status line, headers, body
- [[HTTP Status Codes]] — 200, 301, 400, 404, 500
- [[Content-Type]] — MIME types
- [[Content-Length]] — required for non-chunked responses
- [[HTTP Keep-Alive]] — `Connection: keep-alive`

#### 💻 Implementation

- [ ] Create `HttpResponse` class with: status code, reason phrase, headers map, body
- [ ] Create `HttpResponseBuilder` with fluent API: `.status(200).header("Content-Type", "text/html").body("...")`
- [ ] Implement `serialize()` that produces the full HTTP response string
- [ ] Integrate with server: parse request → build response → send
- [ ] Test with `curl -v` to verify response headers are correct

#### 🧪 Experiment

Intentionally send a response without `Content-Length`. Observe how curl and Firefox handle it differently. Then send with `Connection: close` and observe when the TCP connection closes.

#### 🔬 What to Observe

- What does curl do when `Content-Length` is missing?
- What does a browser do when `Content-Length` is missing?
- When does the TCP connection close with `Connection: close` vs `keep-alive`?

#### 🧠 Questions

- [ ] What is the difference between `Connection: close` and `Connection: keep-alive`?
- [ ] Why must `Content-Length` be exact?
- [ ] What happens if `Content-Length` is wrong?

#### 🧪 Tests

- [ ] Unit test: serialize a 200 OK response with body
- [ ] Unit test: 404 response with correct status line
- [ ] Integration test: curl receives correct Content-Length

#### 📊 Benchmark

Not applicable today.

#### ✅ Definition of Done

- [ ] Fluent `HttpResponseBuilder` works correctly
- [ ] curl shows correct status, headers, and body
- [ ] Server can respond to GET and POST with correct responses

#### 📝 Git Commit

`feat: HTTP response builder, fluent API, Content-Length, Connection headers`

---

### Day 8 — Query Parameters, Multiple Routes

#### 🎯 Goal
Parse query strings from URIs and serve different content on different paths.

#### 📚 Concepts

- [[URI Components]] — scheme, host, path, query string, fragment
- [[Query Parameters]] — `?key=value&key2=value2`
- [[URL Encoding]] — percent-encoding
- [[Basic Routing]] — path-based dispatch

#### 💻 Implementation

- [ ] Implement URI parser: extract path and query string
- [ ] Implement query parameter parser: `?a=1&b=2` → map
- [ ] Implement percent-decoding for query values
- [ ] Add basic dispatch: `if path == "/hello"` respond differently
- [ ] Test: `curl "http://localhost:8080/hello?name=Prasanth&age=23"`

#### 🧪 Experiment

Send a request with a query parameter containing special characters: `curl "http://localhost:8080/?q=hello%20world"`. Verify your decoder handles `%20` → space correctly.

#### 🔬 What to Observe

- What does the raw request line look like for a URL with query params?
- What happens with `+` in query strings vs `%20`?

#### 🧠 Questions

- [ ] What is percent-encoding and why does HTTP need it?
- [ ] What is the difference between the path and the query string?
- [ ] How do you handle duplicate query parameter keys?

#### 🧪 Tests

- [ ] Unit test: parse `/path?a=1&b=2` into path `/path` and params map
- [ ] Unit test: percent-decode `%20` → space, `%2B` → `+`
- [ ] Integration test: `curl "http://localhost:8080/?name=World"` returns correct response

#### 📊 Benchmark

Not applicable.

#### ✅ Definition of Done

- [ ] URI parser correctly separates path and query string
- [ ] Percent-decoding works for common escape sequences
- [ ] Server dispatches to different handlers based on path

#### 📝 Git Commit

`feat: URI parser, query params, percent-decode, basic path dispatch`

---

### Day 9 — Persistent Connections (Keep-Alive)

#### 🎯 Goal
Implement HTTP keep-alive so that one TCP connection can serve multiple HTTP requests. Understand the performance implications.

#### 📚 Concepts

- [[HTTP Keep-Alive]] — persistent connections
- [[HTTP Pipelining]] — sending multiple requests before responses
- [[Connection Lifecycle]] — when to close a connection
- [[recv() Loop]] — reading multiple requests from one socket

#### 💻 Implementation

- [ ] Modify server to loop: parse request → send response → check `Connection: keep-alive` → loop or close
- [ ] Implement connection timeout: close connection after N seconds of inactivity
- [ ] Implement max requests per connection limit (e.g., 100 requests)
- [ ] Test with `curl --http1.1 -v http://localhost:8080/ http://localhost:8080/hello` (single connection, two requests)

#### 🧪 Experiment

Use `tcpdump` to compare: (1) two requests on two separate connections vs (2) two requests on one persistent connection. Count the TCP handshakes.

#### 🔬 What to Observe

- How many TCP handshakes does keep-alive save?
- What happens in `ss` when a connection is in `TIME_WAIT`?
- How long does `TIME_WAIT` last?

#### 🧠 Questions

- [ ] What is `TIME_WAIT` and why does it exist?
- [ ] What is the performance cost of a TCP handshake?
- [ ] How does HTTP/2 improve on HTTP/1.1 keep-alive?
- [ ] What is the `SO_LINGER` socket option?

#### 🧪 Tests

- [ ] Single connection successfully serves 10 sequential requests
- [ ] Connection correctly closes after idle timeout
- [ ] Connection correctly closes when client sends `Connection: close`

#### 📊 Benchmark

Compare latency for 100 requests: (1) new TCP connection each time vs (2) keep-alive persistent connection.

#### ✅ Definition of Done

- [ ] Keep-alive correctly serves multiple requests on one connection
- [ ] Idle timeout works
- [ ] `tcpdump` confirms single handshake for multiple requests

#### 📝 Git Commit

`feat: HTTP keep-alive persistent connections with idle timeout`

---

### Day 10 — Buffering, Partial Reads, Partial Writes

#### 🎯 Goal
Handle the realities of TCP byte streams: data arrives in fragments, and `send()` may not send all bytes at once.

#### 📚 Concepts

- [[TCP Buffering]] — kernel send/receive buffers
- [[Partial Reads]] — `recv()` may return fewer bytes than requested
- [[Partial Writes]] — `send()` may not send all bytes
- [[Read Buffer]] — accumulate bytes until a complete message is formed
- [[Write Buffer]] — queue bytes until they are fully sent

#### 💻 Implementation

- [ ] Implement `ReadBuffer` class: accumulates incoming bytes, supports scanning for `\r\n\r\n`
- [ ] Implement `WriteBuffer` class: queues outgoing bytes, tracks how many have been sent
- [ ] Modify `recv()` loop to feed bytes into `ReadBuffer` and attempt parse after each chunk
- [ ] Modify `send()` loop to drain `WriteBuffer` until fully sent
- [ ] Simulate partial reads by reducing `recv()` buffer size to 16 bytes

#### 🧪 Experiment

Reduce the `recv()` buffer to 1 byte at a time. Send a normal HTTP request and verify your parser still works correctly. This simulates a very slow/fragmented network.

#### 🔬 What to Observe

- How many `recv()` calls does a typical HTTP GET require when reading 1 byte at a time?
- Does your parser produce the correct result regardless of how bytes arrive?

#### 🧠 Questions

- [ ] What happens if you call `send()` on a full kernel socket buffer?
- [ ] What is the difference between `send()` returning 0 and returning -1?
- [ ] What does `EAGAIN` mean on a blocking socket?

#### 🧪 Tests

- [ ] Unit test: `ReadBuffer` correctly identifies `\r\n\r\n` when added one byte at a time
- [ ] Unit test: `WriteBuffer` correctly tracks partial send progress
- [ ] Integration test: server handles requests fragmented into 16-byte chunks

#### 📊 Benchmark

Measure overhead of 1-byte-at-a-time reads vs 4096-byte reads for 10,000 requests.

#### ✅ Definition of Done

- [ ] Server correctly handles requests split across any number of `recv()` calls
- [ ] `WriteBuffer` handles partial `send()` correctly
- [ ] All unit tests pass

#### 📝 Git Commit

`feat: ReadBuffer and WriteBuffer for robust partial read/write handling`

---

## Phase 3 — Robust FSM Parser

### Day 11 — Finite State Machine HTTP Parser

#### 🎯 Goal
Rewrite the HTTP parser as a proper finite state machine (FSM) that processes bytes incrementally without backtracking.

#### 📚 Concepts

- [[Finite State Machine]] — states, transitions, events
- [[Incremental Parsing]] — process one byte or chunk at a time
- [[Parser States]] — RequestLine, HeaderName, HeaderValue, Body, Complete, Error
- [[std::string_view]] — zero-copy slice into the buffer

#### 💻 Implementation

- [ ] Define parser states as an enum class: `RequestLine`, `HeaderName`, `HeaderColon`, `HeaderValue`, `HeaderEnd`, `Body`, `Complete`, `Error`
- [ ] Implement `HttpFSMParser::feed(std::string_view chunk)` that transitions state on each byte
- [ ] Use `std::string_view` to reference slices of the input buffer without copying
- [ ] Handle `\r\n` CRLF line endings correctly in the FSM
- [ ] Draw the state machine diagram before coding (pencil + paper or Mermaid)

#### 🧪 Experiment

\`\`\`mermaid
stateDiagram-v2
    [*] --> RequestLine
    RequestLine --> HeaderName : CRLF
    HeaderName --> HeaderColon : colon
    HeaderColon --> HeaderValue : space
    HeaderValue --> HeaderName : CRLF
    HeaderName --> Body : CRLF empty line
    Body --> Complete : Content-Length bytes read
    RequestLine --> Error : invalid char
    HeaderName --> Error : invalid char
\`\`\`

Feed the FSM one character at a time from a real HTTP request and log every state transition.

#### 🔬 What to Observe

- How many state transitions does a simple `GET / HTTP/1.1` request require?
- What state does an empty header name trigger?
- How does the FSM know when the body is complete?

#### 🧠 Questions

- [ ] Why is an FSM parser more robust than a line-splitting approach?
- [ ] What is the difference between a push parser and a pull parser?
- [ ] How does an FSM prevent buffer overruns?

#### 🧪 Tests

- [ ] Unit test: FSM processes complete GET request correctly
- [ ] Unit test: FSM processes input one byte at a time and still succeeds
- [ ] Unit test: FSM enters Error state on `GET\x00 / HTTP/1.1`
- [ ] Unit test: FSM correctly handles POST with body

#### 📊 Benchmark

Compare FSM parser vs naive line-splitting parser for 100,000 parse operations.

#### ✅ Definition of Done

- [ ] FSM parser passes all unit tests
- [ ] FSM parser handles partial input correctly
- [ ] No string copies inside the hot path (verified by profiling)

#### 📝 Git Commit

`feat: FSM HTTP parser with incremental byte-level processing`

---

### Day 12 — Parser Security & Limits

#### 🎯 Goal
Harden the parser against malformed, oversized, and malicious inputs.

#### 📚 Concepts

- [[HTTP Security]] — parser attacks
- [[Slowloris]] — slow-sending attack
- [[Request Limits]] — max header size, max body size
- [[Header Injection]] — CRLF injection
- [[Request Smuggling]] — ambiguous framing

#### 💻 Implementation

- [ ] Add max request line length limit (e.g., 8192 bytes)
- [ ] Add max number of headers limit (e.g., 100 headers)
- [ ] Add max header name length and header value length limits
- [ ] Add max body size limit (e.g., 1MB for now)
- [ ] Detect and reject `\r\n` injection in header values
- [ ] Add request timeout: if a complete request is not received within 10 seconds, close the connection

#### 🧪 Experiment

Simulate a Slowloris attack: connect to the server and send headers one byte per second, never completing the request. Without a timeout, how many connections can you hold open? With a timeout, what happens?

#### 🔬 What to Observe

- How many slow connections can exhaust the server before timeout is added?
- What error does curl report when the server closes a timed-out connection?

#### 🧠 Questions

- [ ] What is the Slowloris attack and how does it work?
- [ ] What is request smuggling and why is it dangerous?
- [ ] What is CRLF injection?
- [ ] Why must limits be applied before parsing, not after?

#### 🧪 Tests

- [ ] Unit test: request line > 8192 bytes returns Error
- [ ] Unit test: more than 100 headers returns Error
- [ ] Unit test: body > 1MB returns Error
- [ ] Integration test: connection closed after 10-second timeout

#### 📊 Benchmark

Not applicable — focus on security correctness.

#### ✅ Definition of Done

- [ ] All limit checks are in place and tested
- [ ] Timeout mechanism works correctly
- [ ] Slowloris simulation confirms the server handles it gracefully

#### 📝 Git Commit

`feat: parser security limits, request timeout, Slowloris protection`

---

### Day 13 — Parser Comparison & Benchmarking

#### 🎯 Goal
Compare three parser implementations and benchmark them. Understand the real-world performance difference.

#### 📚 Concepts

- [[Zero-Copy Parsing]] — `string_view` vs copying
- [[Memory Allocation]] — impact on throughput
- [[Benchmark Methodology]] — warm-up, steady state, variance
- [[Profiling]] — `perf`, `gprof`, flamegraphs

#### 💻 Implementation

- [ ] Implement three parsers side-by-side: (1) copy-based, (2) `string_view`-based, (3) FSM
- [ ] Write a benchmark harness using `std::chrono` that runs each parser 1,000,000 times
- [ ] Profile with `valgrind --tool=callgrind` to count instruction counts
- [ ] Generate a flamegraph using `perf record` + `perf script` + `FlameGraph` scripts

#### 🧪 Experiment

Run all three parsers with AddressSanitizer enabled. Do any allocate memory in the hot path?

#### 🔬 What to Observe

- Which parser is fastest? By how much?
- Where does each parser spend most of its time?
- How many heap allocations does each parser make per request?

#### 🧠 Questions

- [ ] What is zero-copy and why does it improve performance?
- [ ] What is the cost of heap allocation in a tight loop?
- [ ] What is a flamegraph and how do you read it?

#### 🧪 Tests

- [ ] All three parsers produce identical output for the same input
- [ ] Benchmark results are reproducible across 5 runs

#### 📊 Benchmark

| Parser | 1M ops time | Allocations per parse |
|---|---|---|
| Copy-based | ? ms | ? |
| string_view | ? ms | ? |
| FSM | ? ms | ? |

#### ✅ Definition of Done

- [ ] All three parsers benchmarked and results documented
- [ ] Flamegraph generated and analyzed
- [ ] FSM chosen as the canonical parser going forward
- [ ] Results committed to the repo as `docs/parser-benchmark.md`

#### 📝 Git Commit

`feat: parser benchmark, flamegraph analysis, FSM selected as canonical`

---

## Phase 4 — Static File Server

### Day 14 — Static File Serving

#### 🎯 Goal
Serve real files from disk with correct MIME types, 404 handling, and directory index support.

#### 📚 Concepts

- [[MIME Types]] — Content-Type for files
- [[Filesystem API]] — `std::filesystem` (C++17)
- [[Path Traversal]] — directory traversal attack (`../../../etc/passwd`)
- [[File Descriptors]] — `open()`, `read()`, `close()`

#### 💻 Implementation

- [ ] Implement `StaticFileHandler` that maps URI paths to filesystem paths
- [ ] Implement MIME type lookup table (at least: html, css, js, json, png, jpg, svg, ico, txt)
- [ ] Implement 404 response for missing files
- [ ] Implement directory index: if URI is a directory, serve `index.html`
- [ ] Implement path traversal protection: canonicalize path and verify it is inside the document root
- [ ] Test: `curl http://localhost:8080/index.html`

#### 🧪 Experiment

Try a path traversal attack: `curl http://localhost:8080/../../../etc/passwd`. Verify that your server returns 403 or 404, not the actual file contents.

#### 🔬 What to Observe

- What does `std::filesystem::canonical()` return for `/../../../etc/passwd`?
- What does the raw URI look like for a traversal attempt?

#### 🧠 Questions

- [ ] How does path traversal work and how is it prevented?
- [ ] What is the difference between relative and canonical paths?
- [ ] What MIME type should you use for unknown file types?

#### 🧪 Tests

- [ ] Unit test: path traversal attempt returns 403
- [ ] Unit test: missing file returns 404
- [ ] Unit test: directory URI returns `index.html` if it exists
- [ ] Integration test: `curl http://localhost:8080/index.html` returns file content

#### 📊 Benchmark

Serve a 1MB file with `wrk` — measure throughput in MB/s.

#### ✅ Definition of Done

- [ ] Static files served with correct Content-Type
- [ ] Path traversal protection in place and tested
- [ ] 404 responses correct

#### 📝 Git Commit

`feat: static file server with MIME types, 404, path traversal protection`

---

### Day 15 — HTTP Caching Headers

#### 🎯 Goal
Implement `ETag`, `Last-Modified`, and `If-None-Match` so browsers can cache static files efficiently.

#### 📚 Concepts

- [[ETag]] — entity tag for cache validation
- [[Last-Modified]] — timestamp-based cache validation
- [[If-None-Match]] — conditional request with ETag
- [[If-Modified-Since]] — conditional request with timestamp
- [[304 Not Modified]] — cache hit response
- [[Cache-Control]] — cache directives

#### 💻 Implementation

- [ ] Compute `ETag` from file size + last modified time (or a fast hash)
- [ ] Add `Last-Modified` header using file stat mtime
- [ ] Handle `If-None-Match` request header: compare client ETag vs server ETag → return 304 if match
- [ ] Handle `If-Modified-Since`: parse timestamp, compare, return 304 if not modified
- [ ] Add `Cache-Control: max-age=3600` for static assets
- [ ] Test: first `curl` gets 200, second `curl -H 'If-None-Match: "etag"'` gets 304

#### 🧪 Experiment

Open a browser DevTools Network tab. Load a static file. Reload the page. Observe how the browser uses ETags and how the server returns 304 instead of 200.

#### 🔬 What to Observe

- What is the bandwidth saved by a 304 response vs a 200 response?
- Does the browser still make a network request for a 304?

#### 🧠 Questions

- [ ] What is the difference between a strong ETag and a weak ETag?
- [ ] What does `Cache-Control: no-cache` mean vs `no-store`?
- [ ] When should you NOT use ETags?

#### 🧪 Tests

- [ ] Unit test: ETag is consistent for the same file
- [ ] Unit test: `If-None-Match` with correct ETag returns 304
- [ ] Unit test: `If-None-Match` with wrong ETag returns 200

#### 📊 Benchmark

Measure bandwidth saved by 304 responses under 1000 requests for the same file.

#### ✅ Definition of Done

- [ ] ETag and Last-Modified headers present on all file responses
- [ ] 304 Not Modified works correctly
- [ ] Browser DevTools confirms cache behavior

#### 📝 Git Commit

`feat: ETag, Last-Modified, If-None-Match, 304 Not Modified caching`

---

### Day 16 — sendfile() vs mmap() vs read/write

#### 🎯 Goal
Explore three approaches to sending file data over a socket and benchmark them to understand zero-copy I/O.

#### 📚 Concepts

- [[sendfile()]] — zero-copy file to socket transfer
- [[mmap()]] — memory-mapped file I/O
- [[Zero Copy]] — avoiding kernel/user space data copies
- [[DMA]] — Direct Memory Access
- [[Page Cache]] — kernel file cache

#### 💻 Implementation

- [ ] Implement `send_file_read_write()`: `open()` → `read()` into buffer → `send()` (2 copies)
- [ ] Implement `send_file_mmap()`: `mmap()` the file → `send()` from mapped memory (1 copy)
- [ ] Implement `send_file_sendfile()`: `sendfile()` syscall (0 copies in theory)
- [ ] Benchmark all three with a 1MB, 10MB, and 100MB file using `wrk`
- [ ] Use `strace -c` to count system calls for each approach

#### 🧪 Experiment

Run `strace -c ./server` while serving a large file with each method. Count `read()`, `write()`, `mmap()`, and `sendfile()` syscalls.

#### 🔬 What to Observe

- How many data copies happen with each approach?
- What is the throughput difference for a 100MB file?
- What is the CPU usage difference?

#### 🧠 Questions

- [ ] What does zero-copy actually mean at the DMA level?
- [ ] Why can't `sendfile()` be used with SSL/TLS?
- [ ] What is the Linux page cache and how does it affect file serving performance?
- [ ] When is `mmap()` actually slower than `read()`?

#### 🧪 Tests

- [ ] All three methods serve the file with identical content (byte-for-byte comparison with `md5sum`)
- [ ] All three handle large files (> 2GB) correctly

#### 📊 Benchmark

| Method | 1MB req/s | 10MB MB/s | CPU% |
|---|---|---|---|
| read/write | ? | ? | ? |
| mmap | ? | ? | ? |
| sendfile | ? | ? | ? |

#### ✅ Definition of Done

- [ ] All three implementations working and producing identical output
- [ ] Benchmark results documented with analysis
- [ ] `sendfile()` chosen as default for production path

#### 📝 Git Commit

`feat: sendfile vs mmap vs read/write benchmark, sendfile adopted`

---

## Phase 5 — Concurrency

### Day 17 — Thread Pool Implementation

#### 🎯 Goal
Implement a proper thread pool that avoids creating a new thread per connection. This is the foundation of a scalable server.

#### 📚 Concepts

- [[Thread Pool]] — fixed pool of worker threads
- [[Work Queue]] — producer/consumer queue of tasks
- [[std::mutex]] — mutual exclusion
- [[std::condition_variable]] — thread coordination
- [[Atomic Operations]] — `std::atomic` for lock-free counters
- [[False Sharing]] — cache line contention

#### 💻 Implementation

- [ ] Implement `ThreadPool` class: fixed number of threads, `std::deque<std::function<void()>>` work queue
- [ ] Implement `submit(task)` that pushes to the queue and notifies a worker
- [ ] Implement graceful shutdown: drain the queue before joining all threads
- [ ] Integrate with the HTTP server: `accept()` → `pool.submit([fd]{ handle(fd); })`
- [ ] Test with 4 worker threads and 100 simultaneous connections

#### 🧪 Experiment

Run the server with `wrk -t8 -c200 -d30s`. Measure throughput with thread pool sizes of 1, 2, 4, 8, 16, 32. Plot the results.

#### 🔬 What to Observe

- At what thread pool size does throughput peak?
- What happens to CPU usage as thread count increases beyond CPU core count?
- What does `top -H` show about thread usage?

#### 🧠 Questions

- [ ] What is the optimal thread pool size for CPU-bound vs I/O-bound work?
- [ ] What is false sharing and how does it cause cache line contention?
- [ ] What is a work-stealing thread pool?
- [ ] What is the difference between a thread pool and a coroutine pool?

#### 🧪 Tests

- [ ] Thread pool correctly handles 10,000 sequential tasks with 4 threads
- [ ] Graceful shutdown completes without deadlock
- [ ] ThreadSanitizer reports no data races

#### 📊 Benchmark

| Threads | req/sec | CPU% |
|---|---|---|
| 1 | ? | ? |
| 4 | ? | ? |
| 8 | ? | ? |
| 16 | ? | ? |

#### ✅ Definition of Done

- [ ] Thread pool implemented and integrated
- [ ] Benchmark results show optimal thread count for your machine
- [ ] ThreadSanitizer clean

#### 📝 Git Commit

`feat: thread pool with work queue, graceful shutdown, benchmark`

---

### Day 18 — Mutex, Atomics, and Race Conditions

#### 🎯 Goal
Deeply understand concurrency primitives and deliberately introduce and then fix race conditions.

#### 📚 Concepts

- [[Mutex]] — `std::mutex`, `std::lock_guard`, `std::unique_lock`
- [[Atomic Operations]] — `std::atomic<int>`, `fetch_add`, memory ordering
- [[Race Conditions]] — undefined behavior from unsynchronized access
- [[Deadlocks]] — and how to detect them
- [[Memory Ordering]] — `relaxed`, `acquire`, `release`, `seq_cst`

#### 💻 Implementation

- [ ] Implement a connection counter using `std::atomic<int>` — no mutex needed
- [ ] Implement a request statistics struct using mutexes (total requests, bytes sent, error count)
- [ ] Deliberately introduce a race condition (non-atomic counter), run with ThreadSanitizer, observe the report
- [ ] Fix the race condition, rerun ThreadSanitizer
- [ ] Implement a `SpinLock` using `std::atomic_flag` for comparison

#### 🧪 Experiment

Write a program that has a classic race condition on a shared counter. Run it with `-fsanitize=thread`. Read and understand the ThreadSanitizer report. Then fix it.

#### 🔬 What to Observe

- What does the ThreadSanitizer report look like?
- What is the performance difference between `std::mutex` and `std::atomic` for a simple counter?
- What is `memory_order_relaxed` and when can you use it safely?

#### 🧠 Questions

- [ ] What is the C++ memory model?
- [ ] What is acquire/release semantics and when do you need it?
- [ ] What is a deadlock and how do you avoid it?
- [ ] Why is `volatile` not sufficient for thread synchronization?

#### 🧪 Tests

- [ ] ThreadSanitizer clean on all concurrency code
- [ ] Connection counter is always accurate under 100 concurrent connections

#### 📊 Benchmark

Compare `std::mutex` vs `std::atomic<int>` for a shared counter: 8 threads, 10M increments each.

#### ✅ Definition of Done

- [ ] All concurrency code is ThreadSanitizer clean
- [ ] You understand `memory_order_relaxed` vs `seq_cst`
- [ ] SpinLock implemented and benchmarked

#### 📝 Git Commit

`feat: atomic connection counter, mutex stats, race condition experiment`

---

### Day 19 — Concurrency Comparison & Benchmark

#### 🎯 Goal
Rigorously benchmark all three concurrency models: single-threaded, thread-per-connection, thread pool.

#### 📚 Concepts

- [[Context Switching]] — cost of switching between threads
- [[Contention]] — mutex contention under high load
- [[Throughput vs Latency]] — tradeoffs
- [[wrk]] — HTTP benchmarking tool
- [[perf stat]] — performance counters

#### 💻 Implementation

- [ ] Ensure all three server versions are buildable from the same codebase with a compile flag
- [ ] Write a benchmark script that runs `wrk` against each version with identical settings
- [ ] Use `perf stat` to measure context switches per second for each model
- [ ] Document findings in `docs/concurrency-benchmark.md`

#### 🧪 Experiment

Run `perf stat -e context-switches,cpu-migrations,cache-misses ./server` for each concurrency model under `wrk` load. Compare context switches per second.

#### 🔬 What to Observe

- How do context switches scale with connection count?
- At what point does thread-per-connection perform worse than thread pool?
- How does CPU cache miss rate differ between models?

#### 🧠 Questions

- [ ] Why does thread-per-connection fail at 10,000 connections?
- [ ] What is the C10K problem?
- [ ] Why is event-driven I/O the answer to C10K?

#### 🧪 Tests

- [ ] All three models produce identical HTTP responses
- [ ] Thread pool handles 1,000 concurrent connections without OOM

#### 📊 Benchmark

| Model | 100 conns req/s | 1000 conns req/s | Context switches/s |
|---|---|---|---|
| Single-threaded | ? | ? | ? |
| Thread/conn | ? | ? | ? |
| Thread pool | ? | ? | ? |

#### ✅ Definition of Done

- [ ] All three models benchmarked and documented
- [ ] Conclusion clear: why the event loop (Phase 6) is needed
- [ ] `docs/concurrency-benchmark.md` committed

#### 📝 Git Commit

`docs: concurrency model comparison benchmark results`

---

## Phase 6 — Non-Blocking I/O & epoll

### Day 20 — Non-Blocking Sockets

#### 🎯 Goal
Understand what `O_NONBLOCK` actually does and how `EAGAIN` works.

#### 📚 Concepts

- [[Non-Blocking I/O]] — `O_NONBLOCK` flag
- [[EAGAIN]] / `EWOULDBLOCK` — would block error code
- [[fcntl()]] — file descriptor control
- [[Blocking vs Non-Blocking]] — the fundamental difference

#### 💻 Implementation

- [ ] Set a socket to non-blocking: `fcntl(fd, F_SETFL, O_NONBLOCK)`
- [ ] Implement a `recv()` loop that handles `EAGAIN` correctly (retry later, not error)
- [ ] Implement a `send()` loop that handles `EAGAIN` correctly
- [ ] Observe what happens when you call `recv()` on a non-blocking socket with no data available

#### 🧪 Experiment

Create a non-blocking socket. Call `recv()` before any data arrives. Observe the return value and `errno`. Then call `recv()` again after data arrives.

#### 🔬 What to Observe

- What is the difference between `recv()` returning 0 vs -1 with `EAGAIN`?
- What does `EWOULDBLOCK` mean?
- How does non-blocking mode change the mental model of I/O?

#### 🧠 Questions

- [ ] What does `O_NONBLOCK` actually change in the kernel?
- [ ] What is the difference between `EAGAIN` and `EWOULDBLOCK`?
- [ ] When is `recv()` returning 0 vs -1?
- [ ] How would you build a server using only non-blocking sockets and a busy loop?

#### 🧪 Tests

- [ ] Non-blocking `recv()` returns `EAGAIN` immediately when no data available
- [ ] Non-blocking `send()` returns `EAGAIN` when the send buffer is full

#### 📊 Benchmark

Compare CPU usage: blocking server (idle) vs non-blocking busy-poll server (idle). Measure CPU% with `top`.

#### ✅ Definition of Done

- [ ] Non-blocking sockets working correctly
- [ ] `EAGAIN` handled without treating it as a fatal error
- [ ] You understand why busy-polling is not the answer

#### 📝 Git Commit

`feat: non-blocking sockets, EAGAIN handling, O_NONBLOCK experiment`

---

### Day 21 — select() and poll()

#### 🎯 Goal
Understand the predecessors to epoll. Implement `select()` and `poll()` based multiplexing servers to understand the problem they solve — and their limitations.

#### 📚 Concepts

- [[select()]] — the original I/O multiplexing syscall
- [[poll()]] — improved select with dynamic fd sets
- [[fd_set]] — fixed-size bitmask
- [[pollfd]] — poll file descriptor structure
- [[O(n) Scanning]] — why select/poll don't scale

#### 💻 Implementation

- [ ] Implement a `select()`-based server that handles multiple non-blocking sockets
- [ ] Implement a `poll()`-based server that handles multiple non-blocking sockets
- [ ] Observe the O(n) scan over all file descriptors in each iteration
- [ ] Test with 10, 100, 500 connections

#### 🧪 Experiment

Run `select()`-based server with 500 connections where only 1 is active. Use `perf stat` to count CPU instructions. Observe the O(n) scan cost even when nothing is happening.

#### 🔬 What to Observe

- How does CPU usage change as the number of idle connections grows?
- What is the `FD_SETSIZE` limit in `select()`?
- What happens when a fd > 1023 is used with `select()`?

#### 🧠 Questions

- [ ] Why does `select()` have an `FD_SETSIZE` (1024) limit?
- [ ] Why does `poll()` require scanning all fds even when none are ready?
- [ ] What is the fundamental algorithmic problem with both `select()` and `poll()`?
- [ ] How does `epoll` solve the O(n) problem?

#### 🧪 Tests

- [ ] `select()` server handles 10 simultaneous connections
- [ ] `poll()` server handles 100 simultaneous connections

#### 📊 Benchmark

| Mechanism | 100 conns CPU% | 500 conns CPU% |
|---|---|---|
| select | ? | ? |
| poll | ? | ? |

#### ✅ Definition of Done

- [ ] Both `select()` and `poll()` implementations working
- [ ] You can explain why neither scales to 10,000 connections
- [ ] The stage is set for `epoll` tomorrow

#### 📝 Git Commit

`feat: select and poll based servers, scalability limitations documented`

---

### Day 22 — epoll Fundamentals

#### 🎯 Goal
Implement a proper `epoll`-based event loop. Understand level-triggered vs edge-triggered modes.

#### 📚 Concepts

- [[epoll]] — scalable Linux I/O event notification
- [[epoll_create1()]] — create epoll instance
- [[epoll_ctl()]] — add/modify/delete watched fds
- [[epoll_wait()]] — wait for events
- [[Level-Triggered]] — `EPOLLIN` fires as long as data available
- [[Edge-Triggered]] — `EPOLLIN | EPOLLET` fires only on state change

#### 💻 Implementation

- [ ] Implement `epoll_create1(EPOLL_CLOEXEC)` → `epoll_ctl(EPOLL_CTL_ADD)` → `epoll_wait()` event loop
- [ ] Add listening socket to epoll in level-triggered mode
- [ ] Accept new connections and add client sockets to epoll
- [ ] Handle `EPOLLIN` events: read data from ready sockets
- [ ] Handle `EPOLLOUT` events: send buffered data when socket is writable
- [ ] Handle `EPOLLHUP`/`EPOLLERR` events: close connection

#### 🧪 Experiment

Implement the same server in both level-triggered (LT) and edge-triggered (ET) mode. For ET mode, deliberately do not read all available data in one call. Observe what happens (you will miss events). Fix it by reading until `EAGAIN`.

#### 🔬 What to Observe

- What happens with ET mode if you don't read until `EAGAIN`?
- What does the `events` bitmask look like in the `epoll_event` struct?
- How many `epoll_wait()` syscalls per second does a busy server make?

#### 🧠 Questions

- [ ] Why does epoll scale to millions of connections while select cannot?
- [ ] What is the difference between level-triggered and edge-triggered epoll?
- [ ] When should you use ET vs LT?
- [ ] What is `EPOLLONESHOT` and when is it useful?

#### 🧪 Tests

- [ ] epoll server handles 100 simultaneous connections correctly
- [ ] LT mode: correct behavior even when reading partial data
- [ ] ET mode: correct behavior by reading until `EAGAIN`

#### 📊 Benchmark

| Mechanism | 1000 conns req/s | 5000 conns req/s | 10000 conns req/s |
|---|---|---|---|
| select | ? | N/A | N/A |
| poll | ? | ? | N/A |
| epoll LT | ? | ? | ? |

#### ✅ Definition of Done

- [ ] epoll event loop handling multiple connections
- [ ] LT and ET modes both implemented and understood
- [ ] Benchmark confirms epoll's scaling advantage

#### 📝 Git Commit

`feat: epoll event loop, level-triggered and edge-triggered modes`

---

### Day 23 — epoll Deep Dive & Connection State

#### 🎯 Goal
Implement proper per-connection state tracking inside the epoll loop. This is the foundation of the Reactor pattern.

#### 📚 Concepts

- [[Connection State Machine]] — states per connection
- [[epoll User Data]] — `epoll_event.data.ptr` for per-connection context
- [[Read/Write Readiness]] — toggling EPOLLOUT based on write buffer state
- [[Connection Lifecycle]] — CONNECTING → READING → WRITING → CLOSING

#### 💻 Implementation

- [ ] Create `Connection` struct: fd, read buffer, write buffer, state, last activity time
- [ ] Store `Connection*` in `epoll_event.data.ptr`
- [ ] Implement state transitions: READING → WRITING when response is ready
- [ ] Toggle `EPOLLOUT` on the fd when write buffer is non-empty, off when drained
- [ ] Implement idle timeout: scan connections for last activity > 60 seconds

#### 🧪 Experiment

Add logging to every state transition. Connect 5 clients and observe the full lifecycle of each connection from ACCEPT through READING, WRITING, to CLOSE.

#### 🔬 What to Observe

- How many epoll events does one complete HTTP request/response cycle generate?
- What happens when a client closes the connection unexpectedly?
- What does `EPOLLHUP` vs `EPOLLERR` indicate?

#### 🧠 Questions

- [ ] What is the difference between `EPOLLHUP` and `EPOLLERR`?
- [ ] What does it mean when `recv()` returns 0?
- [ ] Why do you need to track per-connection state explicitly?

#### 🧪 Tests

- [ ] 1000 connections all complete HTTP request/response cycle correctly
- [ ] Idle timeout correctly closes inactive connections
- [ ] Server handles unexpected client disconnection gracefully

#### 📊 Benchmark

Measure events per second in `epoll_wait()` under 1000 active connections.

#### ✅ Definition of Done

- [ ] Per-connection state machine working correctly
- [ ] Idle timeout implemented
- [ ] Graceful handling of unexpected disconnections
- [ ] Foundation ready for the Reactor pattern (Phase 7)

#### 📝 Git Commit

`feat: per-connection state machine, idle timeout, epoll lifecycle`

---

## Phase 7 — Event-Driven Reactor Architecture

### Day 24 — Reactor Pattern Design

#### 🎯 Goal
Redesign the server around the Reactor pattern. This is the architectural breakthrough of the project.

#### 📚 Concepts

- [[Reactor Pattern]] — event loop + handlers + dispatcher
- [[Event Loop]] — the central polling mechanism
- [[Handler]] — callback for a specific event type
- [[Dispatcher]] — routes events to handlers
- [[Single Responsibility]] — each component has one job

#### 💻 Implementation

- [ ] Design the architecture on paper first
- [ ] Implement `EventLoop` class with `run()`, `stop()`, `add_fd()`, `remove_fd()` methods
- [ ] Implement `Acceptor` class that handles new connections on the listening socket
- [ ] Wire everything together with clean interfaces

\`\`\`mermaid
flowchart TD
    Client --> Listener
    Listener --> EventLoop
    EventLoop --> ConnectionManager
    ConnectionManager --> HttpParser
    HttpParser --> Router
    Router --> Middleware
    Middleware --> Handler
    Handler --> Response
    Response --> WriteBuffer
    WriteBuffer --> EventLoop
\`\`\`

#### 🧪 Experiment

Draw the sequence diagram for a single HTTP request/response on paper before coding. Trace: client connects → epoll fires EPOLLIN on listen fd → accept → new fd added → epoll fires EPOLLIN on client fd → read → parse → route → handle → write.

#### 🔬 What to Observe

- How clean is the separation between the event loop and the HTTP logic?
- What is the single responsibility of each class?

#### 🧠 Questions

- [ ] What is the Reactor pattern and how does it differ from thread-per-connection?
- [ ] What is the Proactor pattern?
- [ ] What is the difference between `io_uring` and `epoll`?

#### 🧪 Tests

- [ ] EventLoop starts and stops cleanly
- [ ] Acceptor correctly adds new connections to the EventLoop
- [ ] Integration test: complete HTTP request/response through the reactor

#### 📊 Benchmark

Baseline: how many req/sec does the raw Reactor handle with a hardcoded response?

#### ✅ Definition of Done

- [ ] Reactor architecture implemented with clean class boundaries
- [ ] EventLoop, Acceptor, ConnectionManager separated
- [ ] Architecture diagram committed to docs

#### 📝 Git Commit

`feat: Reactor pattern architecture, EventLoop, Acceptor, ConnectionManager`

---

### Day 25 — Connection Manager & Lifecycle

#### 🎯 Goal
Build a robust connection manager that tracks all active connections and handles their full lifecycle.

#### 📚 Concepts

- [[Connection Lifecycle]] — full lifecycle from accept to close
- [[RAII]] — automatic cleanup of connections
- [[Graceful Shutdown]] — drain connections before exit
- [[Backpressure]] — limit accepted connections when overloaded

#### 💻 Implementation

- [ ] Implement `ConnectionManager` with `std::unordered_map<int, std::unique_ptr<Connection>>`
- [ ] Implement `accept_connection()`, `close_connection()`, `cleanup_idle()`
- [ ] Implement connection limit: if active connections > MAX, stop accepting
- [ ] Implement graceful shutdown signal handler (`SIGTERM`, `SIGINT`): stop accepting, drain existing connections, exit cleanly
- [ ] Add connection stats: active count, total served, total errors

#### 🧪 Experiment

Send `SIGTERM` while 50 connections are active. Verify all in-flight requests complete before the server exits.

#### 🔬 What to Observe

- What happens if the server exits while a `send()` is in progress?
- How long does graceful shutdown take with 100 in-flight requests?

#### 🧠 Questions

- [ ] What is the difference between SIGTERM and SIGKILL?
- [ ] How does a reverse proxy achieve zero-downtime restarts?
- [ ] What happens to a TCP connection when the server process exits?

#### 🧪 Tests

- [ ] Graceful shutdown completes all in-flight responses
- [ ] Connection limit correctly stops accepting when at max
- [ ] RAII ensures no file descriptor leaks (verify with `lsof`)

#### 📊 Benchmark

Measure time from `SIGTERM` to process exit under 100, 1000 in-flight connections.

#### ✅ Definition of Done

- [ ] ConnectionManager tracks all connections with RAII
- [ ] Graceful shutdown works correctly
- [ ] Connection limit enforced
- [ ] No file descriptor leaks

#### 📝 Git Commit

`feat: ConnectionManager with lifecycle, graceful shutdown, connection limits`

---

### Day 26 — Write Path & Backpressure

#### 🎯 Goal
Build a robust write path: buffer responses, handle partial sends, and implement write backpressure.

#### 📚 Concepts

- [[Write Buffer]] — queue outgoing data
- [[Backpressure]] — slow down producers when consumers are slow
- [[EPOLLOUT]] — toggle based on write buffer state
- [[Partial Write]] — `send()` may not send all bytes

#### 💻 Implementation

- [ ] Implement `WriteBuffer` with a `std::deque<std::string>` of pending writes
- [ ] Implement `flush()` method that sends as much as possible and handles `EAGAIN`
- [ ] Toggle `EPOLLOUT` on the fd: add when write buffer is non-empty, remove when drained
- [ ] Implement max write buffer size: if write buffer > MAX, close connection (client too slow)
- [ ] Implement per-connection write timeout

#### 🧪 Experiment

Use `tc qdisc add dev lo root netem delay 500ms` to simulate a very slow client. Observe how write buffers grow. Verify the max buffer limit triggers correctly.

#### 🔬 What to Observe

- How quickly does the write buffer grow for a slow client?
- What happens to the event loop when a write blocks?
- Does `EPOLLOUT` toggle correctly?

#### 🧠 Questions

- [ ] What is backpressure in networking?
- [ ] What happens if you don't implement write buffering?
- [ ] How does TCP's own flow control relate to application-level backpressure?

#### 🧪 Tests

- [ ] Write buffer correctly handles partial `send()` across multiple events
- [ ] EPOLLOUT toggled correctly based on buffer state
- [ ] Slow client connection closed after write buffer exceeds limit

#### 📊 Benchmark

Measure max memory usage under 1000 slow clients each receiving a 1MB response.

#### ✅ Definition of Done

- [ ] Write path is correct and robust under all partial-write scenarios
- [ ] Backpressure via connection close works
- [ ] EPOLLOUT correctly managed

#### 📝 Git Commit

`feat: write buffer, EPOLLOUT management, write backpressure`

---

### Day 27 — Integration: Full Reactor HTTP Server

#### 🎯 Goal
Complete the Reactor HTTP server integration. All components working together for the first time as a cohesive event-driven system.

#### 📚 Concepts

- [[Integration]] — assembling all components
- [[End-to-End Testing]] — full request/response cycle
- [[Regression Testing]] — ensure previous features still work

#### 💻 Implementation

- [ ] Wire together: EventLoop → Acceptor → ConnectionManager → HttpParser → Router → StaticFileHandler → WriteBuffer
- [ ] Ensure keep-alive works with the Reactor (loop on parse after each complete request)
- [ ] Ensure idle timeout is enforced by the event loop timer mechanism
- [ ] Verify all Phase 4 (static files) and Phase 3 (parser security) features still work
- [ ] Load test with `wrk -t4 -c1000 -d30s http://localhost:8080/`

#### 🧪 Experiment

Run `wrk` with 1000 connections for 60 seconds. Monitor with `htop`, `ss -s`, and `watch -n1 'cat /proc/$(pidof server)/status | grep VmRSS'`.

#### 🔬 What to Observe

- Memory usage over 60 seconds — is it stable or growing?
- What does `ss -s` show about socket states?
- Are there any file descriptor leaks?

#### 🧠 Questions

- [ ] Is your server memory-stable under sustained load?
- [ ] Does the reactor handle a burst of 1000 connections cleanly?

#### 🧪 Tests

- [ ] All existing tests pass on the Reactor architecture
- [ ] 1000 concurrent connections handled correctly
- [ ] Memory stable over 60 seconds

#### 📊 Benchmark

Full `wrk` benchmark: req/sec, latency p50/p95/p99.

#### ✅ Definition of Done

- [ ] Full Reactor HTTP server integrated
- [ ] All previous features working
- [ ] Memory stable under sustained load
- [ ] Benchmark recorded as new baseline

#### 📝 Git Commit

`feat: full reactor HTTP server integration, 1000-conn benchmark`

---

## Phase 8 — Production HTTP Features

### Day 28 — Chunked Transfer Encoding

#### 🎯 Goal
Implement chunked transfer encoding for streaming responses where the total length is unknown.

#### 📚 Concepts

- [[Chunked Transfer Encoding]] — `Transfer-Encoding: chunked`
- [[Streaming Response]] — send body in chunks
- [[Chunk Format]] — `hex-size\r\ndata\r\n ... 0\r\n\r\n`
- [[Content-Length vs Chunked]] — when to use each

#### 💻 Implementation

- [ ] Implement `ChunkedEncoder` that wraps a body chunk in chunked format
- [ ] Implement `ChunkedDecoder` for chunked request bodies
- [ ] Add `Transfer-Encoding: chunked` support in response builder
- [ ] Test with `curl -v --chunked -T file.txt http://localhost:8080/upload`
- [ ] Stream a large file using chunked encoding without loading it all into memory

#### 🧪 Experiment

Use `tcpdump -A` to observe the raw bytes of a chunked response. Find the chunk size in hex, the data, and the terminal `0\r\n\r\n`.

#### 🔬 What to Observe

- What does a chunked response look like on the wire?
- How does a browser know when chunked encoding is done?
- Can you stream a 1GB file without using 1GB of memory?

#### 🧠 Questions

- [ ] When should you use chunked encoding vs `Content-Length`?
- [ ] Can you use both `Content-Length` and `Transfer-Encoding: chunked`?
- [ ] How does HTTP/2 handle streaming differently?

#### 🧪 Tests

- [ ] Unit test: chunked encoder produces correct format
- [ ] Unit test: chunked decoder handles multi-chunk body
- [ ] Integration test: streaming response received correctly by curl

#### 📊 Benchmark

Stream a 100MB file: memory usage with chunked vs loading entire file.

#### ✅ Definition of Done

- [ ] Chunked encoding works for both requests and responses
- [ ] Large files streamed without loading into memory
- [ ] curl correctly decodes chunked responses

#### 📝 Git Commit

`feat: chunked transfer encoding, streaming responses`

---

### Day 29 — Security Hardening

#### 🎯 Goal
Add a comprehensive set of security protections to make the server production-worthy.

#### 📚 Concepts

- [[HTTP Security]] — production security patterns
- [[Header Limits]] — prevent header attacks
- [[Connection Limits]] — prevent resource exhaustion
- [[Rate Limiting]] — basic per-IP request limiting
- [[Slowloris]] — slow request attack

#### 💻 Implementation

- [ ] Implement per-IP connection limit (e.g., max 100 connections per IP)
- [ ] Implement global connection limit (e.g., max 10,000 connections)
- [ ] Implement request header size limit (8KB)
- [ ] Implement request body size limit (configurable, default 1MB)
- [ ] Implement read timeout (10 seconds to send complete headers)
- [ ] Implement write timeout (30 seconds to drain write buffer)
- [ ] Add security response headers: `X-Content-Type-Options`, `X-Frame-Options`
- [ ] Verify path traversal protection from Day 14 is still in place

#### 🧪 Experiment

Simulate Slowloris. Verify the read timeout closes the connection.

#### 🔬 What to Observe

- How many slow connections can you open before the read timeout kicks in?
- What does the server log show for a timed-out connection?

#### 🧠 Questions

- [ ] What is the difference between a Slowloris attack and a DDoS?
- [ ] What is connection multiplexing and how does it change attack surface?
- [ ] Why is `X-Content-Type-Options: nosniff` important?

#### 🧪 Tests

- [ ] Read timeout correctly closes slow clients
- [ ] Per-IP connection limit enforced
- [ ] Path traversal returns 403
- [ ] Request size limit returns 413

#### 📊 Benchmark

Measure performance overhead of all security checks: req/sec with vs without.

#### ✅ Definition of Done

- [ ] All security limits in place and tested
- [ ] Slowloris simulation handled correctly
- [ ] Security headers added to all responses

#### 📝 Git Commit

`feat: security hardening, per-IP limits, timeouts, size limits, security headers`

---

## Phase 9 — Router + Middleware

### Day 30 — Router and Middleware Pipeline

#### 🎯 Goal
Implement a clean router with path parameters and a middleware pipeline.

#### 📚 Concepts

- [[HTTP Router]] — matches requests to handlers
- [[Path Parameters]] — `/users/:id`
- [[Middleware]] — composable request/response interceptors
- [[Trie]] — efficient prefix matching data structure

#### 💻 Implementation

- [ ] Implement `Router` class with `add_route(method, pattern, handler)` and `match(method, path)`
- [ ] Support exact routes: `/about`
- [ ] Support parameter routes: `/users/:id` → extracts `id`
- [ ] Support wildcard routes: `/static/*` → matches any suffix
- [ ] Implement `MiddlewarePipeline` with `use(fn)` and chaining
- [ ] Implement example middleware: request logger, timing, CORS headers
- [ ] Test: `GET /users/42` matches route and extracts `id = "42"`

#### 🧪 Experiment

Implement the router as both `unordered_map` (exact only) and a simple trie. Benchmark match time for 1000 routes with `std::chrono`.

#### 🔬 What to Observe

- How does trie lookup compare to linear scan for 1000 routes?
- What is the memory overhead of a trie vs a flat map?

#### 🧠 Questions

- [ ] What is a radix tree and how does it differ from a regular trie?
- [ ] How does Express.js implement routing?
- [ ] What is the difference between router middleware and route-specific middleware?

#### 🧪 Tests

- [ ] Unit test: exact route matches correctly
- [ ] Unit test: parameter route extracts correct values
- [ ] Unit test: wildcard route matches any suffix
- [ ] Integration test: middleware executes in correct order
- [ ] Unit test: no route match returns 404

#### 📊 Benchmark

Route match time for 10 routes vs 1000 routes: unordered_map vs trie.

#### ✅ Definition of Done

- [ ] Router handles exact, parameter, and wildcard routes
- [ ] Middleware pipeline works correctly
- [ ] Logger middleware logs all requests with method, path, status, time

#### 📝 Git Commit

`feat: router with path params and wildcards, middleware pipeline`

---

## Phase 10 — Reverse Proxy

### Day 31 — Reverse Proxy: Upstream Connection

#### 🎯 Goal
Turn ForgeHTTP into a reverse proxy. Forward incoming HTTP requests to upstream backend servers.

#### 📚 Concepts

- [[Reverse Proxy]] — forwards client requests to backends
- [[Upstream]] — the backend server
- [[Connection Pool]] — reuse TCP connections to upstream
- [[HTTP Forwarding]] — `X-Forwarded-For`, `Host` header rewriting

#### 💻 Implementation

- [ ] Implement `UpstreamConnector` that establishes TCP connections to backend servers
- [ ] Implement `ProxyHandler`: receive client request → forward to upstream → stream response back to client
- [ ] Add `X-Forwarded-For: <client IP>` header when forwarding
- [ ] Add `X-Forwarded-Proto`, `X-Real-IP` headers
- [ ] Rewrite `Host` header to upstream host
- [ ] Test: run `python3 -m http.server 8081` as backend, ForgeHTTP as proxy on 8080

#### 🧪 Experiment

Use `tcpdump` on both ports simultaneously: observe the client→proxy request and the proxy→backend request side by side.

#### 🔬 What to Observe

- How do the two TCP connections (client→proxy and proxy→backend) relate?
- What headers change between the client request and the forwarded request?

#### 🧠 Questions

- [ ] What is the difference between a forward proxy and a reverse proxy?
- [ ] What is `X-Forwarded-For` and why does it matter for security?
- [ ] What is hop-by-hop vs end-to-end headers?

#### 🧪 Tests

- [ ] Proxy correctly forwards GET requests to backend
- [ ] `X-Forwarded-For` header contains the correct client IP
- [ ] Response from backend is correctly relayed to client

#### 📊 Benchmark

Baseline: latency added by the proxy (direct vs proxied).

#### ✅ Definition of Done

- [ ] Reverse proxy forwards requests and relays responses
- [ ] Forwarded headers correctly set
- [ ] `tcpdump` confirms two TCP connections per proxied request

#### 📝 Git Commit

`feat: reverse proxy upstream connector, request forwarding, proxy headers`

---

### Day 32 — Load Balancer: Round Robin

#### 🎯 Goal
Implement round-robin load balancing across multiple upstream backends.

#### 📚 Concepts

- [[Load Balancing]] — distributing traffic across backends
- [[Round Robin]] — simple sequential distribution
- [[Upstream Pool]] — collection of backend servers
- [[Health Check]] — detect and remove failed backends

#### 💻 Implementation

- [ ] Implement `UpstreamPool` with a list of backends and a round-robin selector
- [ ] Use `std::atomic<size_t>` for the round-robin counter (lock-free)
- [ ] Implement basic health check: periodic TCP connect to each backend, mark unhealthy if failed
- [ ] Skip unhealthy backends in round-robin selection
- [ ] Test: start 3 backends on ports 8081, 8082, 8083 — verify requests distribute evenly

#### 🧪 Experiment

Kill one backend while the load balancer is serving traffic. Verify that the health check detects the failure and stops sending requests to it.

#### 🔬 What to Observe

- How long does it take for the health check to detect a backend failure?
- What happens to in-flight requests when a backend dies mid-response?

#### 🧠 Questions

- [ ] What is the difference between round-robin, least-connections, and weighted round-robin?
- [ ] What is a connection pool and how does it improve proxy performance?
- [ ] How do production load balancers (nginx, HAProxy) implement health checks?

#### 🧪 Tests

- [ ] Requests distributed across 3 backends in round-robin order
- [ ] Failed backend correctly removed from rotation
- [ ] Recovered backend correctly re-added to rotation

#### 📊 Benchmark

Load test through the proxy: compare latency direct vs 1 backend vs 3 backends.

#### ✅ Definition of Done

- [ ] Round-robin across 3+ backends working
- [ ] Health checks detect backend failure within 5 seconds
- [ ] Atomic round-robin counter is lock-free

#### 📝 Git Commit

`feat: round-robin load balancer, health checks, upstream pool`

---

### Day 33 — Upstream Connection Pool & Retry

#### 🎯 Goal
Implement upstream connection pooling to avoid creating a new TCP connection for every proxied request.

#### 📚 Concepts

- [[Connection Pooling]] — reuse upstream TCP connections
- [[Connection Pool Management]] — idle connections, max pool size
- [[Retry Logic]] — retry failed upstream requests
- [[Timeout Handling]] — upstream connect and read timeouts

#### 💻 Implementation

- [ ] Implement `ConnectionPool` per upstream: pool of idle connections, `acquire()`, `release()`
- [ ] Reuse idle connections instead of creating new ones per request
- [ ] Implement max pool size (e.g., 10 connections per upstream)
- [ ] Implement upstream connect timeout (e.g., 5 seconds)
- [ ] Implement upstream read timeout (e.g., 30 seconds)
- [ ] Implement one retry on upstream failure (not for POST — idempotency)

#### 🧪 Experiment

Run `wrk` against the proxy with and without connection pooling. Measure the throughput difference.

#### 🔬 What to Observe

- How many TCP connections does `tcpdump` show without pooling vs with pooling?
- What is the latency reduction from connection pooling?

#### 🧠 Questions

- [ ] Why should POST requests not be retried on upstream failure?
- [ ] What is idempotency and why does it matter for retry logic?
- [ ] How does HTTP/2 multiplexing make connection pooling less necessary?

#### 🧪 Tests

- [ ] Connection pool reuses connections (verify with `tcpdump` — fewer TCP handshakes)
- [ ] Pool respects max size limit
- [ ] Upstream timeout correctly handled and error returned to client

#### 📊 Benchmark

| Config | req/sec | TCP handshakes/s |
|---|---|---|
| No pooling | ? | ? |
| Pool size 5 | ? | ? |
| Pool size 10 | ? | ? |

#### ✅ Definition of Done

- [ ] Connection pool working and measurably reducing TCP handshakes
- [ ] Timeouts and retries implemented
- [ ] Benchmark shows clear throughput improvement

#### 📝 Git Commit

`feat: upstream connection pool, retry logic, connect/read timeouts`

---

## Phase 11 — Observability

### Day 34 — Metrics, Logging, and Debug Endpoints

#### 🎯 Goal
Add structured observability: logging, metrics, and debug HTTP endpoints. Make the server's internal state visible.

#### 📚 Concepts

- [[Structured Logging]] — machine-readable log format
- [[Metrics]] — counters, gauges, histograms
- [[Percentiles]] — p50, p95, p99 latency
- [[Request ID]] — unique ID per request for tracing
- [[Histogram]] — latency distribution

#### 💻 Implementation

- [ ] Implement `Logger` with structured JSON-like output: `timestamp`, `level`, `request_id`, `method`, `path`, `status`, `duration_ms`, `bytes`
- [ ] Implement `MetricsCollector` with atomic counters: `requests_total`, `requests_in_flight`, `bytes_sent_total`, `bytes_received_total`, `errors_total`, `connections_total`
- [ ] Implement latency histogram with buckets: 1ms, 5ms, 10ms, 25ms, 50ms, 100ms, 250ms, 500ms, 1s
- [ ] Compute p50/p95/p99 from the histogram
- [ ] Implement `GET /metrics` endpoint that returns all metrics as JSON
- [ ] Implement `GET /debug/stats` that returns human-readable server status
- [ ] Generate a unique `X-Request-Id` UUID for each request

#### 🧪 Experiment

Run `wrk -t4 -c100 -d30s`. While it runs, poll `curl http://localhost:8080/metrics` every second. Watch the counters change in real time.

#### 🔬 What to Observe

- Does the p99 latency change during a burst of 1000 req/s vs 100 req/s?
- Are the atomic counters always accurate even under 1000 concurrent connections?

#### 🧠 Questions

- [ ] What is the difference between a counter and a gauge?
- [ ] How is a histogram different from a simple average?
- [ ] What is the four golden signals of observability?
- [ ] How does Prometheus scrape metrics?

#### 🧪 Tests

- [ ] `GET /metrics` returns valid JSON with correct counters
- [ ] `requests_total` increments correctly under concurrent load
- [ ] p99 latency is computed correctly from histogram

#### 📊 Benchmark

Measure overhead of metrics collection under 10,000 req/s. Compare with/without metrics.

#### ✅ Definition of Done

- [ ] Structured logging in place
- [ ] `/metrics` endpoint returns accurate counters and latency percentiles
- [ ] Request IDs present in all log lines
- [ ] Metrics collection overhead < 1% of request time

#### 📝 Git Commit

`feat: structured logging, metrics collector, /metrics endpoint, request IDs`

---

## Phase 12 — Performance Engineering

### Day 35 — Profiling and Flamegraphs

#### 🎯 Goal
Profile ForgeHTTP under load using perf and flamegraphs. Identify the actual bottlenecks — not guesses.

#### 📚 Concepts

- [[Profiling]] — measuring where time is spent
- [[Flamegraph]] — visualization of call stacks
- [[perf]] — Linux performance analysis tool
- [[Hotspot]] — code that consumes disproportionate CPU
- [[Cache Miss]] — performance killer

#### 💻 Implementation

- [ ] Install FlameGraph scripts: `git clone https://github.com/brendangregg/FlameGraph`
- [ ] Run server under `wrk` load: `wrk -t4 -c500 -d30s http://localhost:8080/`
- [ ] Simultaneously profile: `perf record -g -p $(pidof server) -o perf.data sleep 20`
- [ ] Generate flamegraph: `perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > flamegraph.svg`
- [ ] Identify top 5 hotspots from the flamegraph
- [ ] Use `perf stat` to measure: instructions, cache-misses, branch-misses, context-switches

#### 🧪 Experiment

Run profiling with two different workloads: (1) many small requests (100 bytes) and (2) large file serving (1MB). Compare the flamegraphs. Are the bottlenecks the same?

#### 🔬 What to Observe

- Where does the server spend most of its time?
- Is the parser a hotspot?
- Is memory allocation a hotspot?
- Is `epoll_wait()` dominating? (if so, the server is I/O-bound, not CPU-bound)

#### 🧠 Questions

- [ ] What is the difference between CPU-bound and I/O-bound?
- [ ] What is a cache miss and how does it affect performance?
- [ ] How do you read a flamegraph?
- [ ] What is `perf stat` measuring?

#### 🧪 Tests

- [ ] Flamegraph generated successfully
- [ ] Top 5 hotspots identified and documented

#### 📊 Benchmark

Record baseline: req/sec, p50/p95/p99, CPU%, memory usage (RSS).

#### ✅ Definition of Done

- [ ] Flamegraph generated and analyzed
- [ ] Top 5 hotspots documented in `docs/profiling-results.md`
- [ ] Baseline benchmark recorded for Day 36 comparison

#### 📝 Git Commit

`docs: flamegraph analysis, profiling results, performance baseline`

---

### Day 36 — Optimization + Final Benchmark

#### 🎯 Goal
Implement the most impactful optimizations identified in Day 35. Benchmark the improvement.

> [!WARNING]
> Never optimize without first measuring. Every optimization must be validated by a benchmark that shows improvement. Premature optimization is the enemy of readable code.

#### 📚 Concepts

- [[Optimization Methodology]] — measure → hypothesize → optimize → measure again
- [[Memory Pool]] — preallocate objects to avoid heap fragmentation
- [[Cache Locality]] — arrange data for CPU cache efficiency
- [[SO_REUSEPORT]] — multiple listen sockets for multi-core

#### 💻 Implementation

Based on Day 35 findings, implement optimizations:
- [ ] Pre-allocate `Connection` objects from a pool to avoid per-connection `new`/`delete`
- [ ] Use `SO_REUSEPORT` to allow multiple threads to accept on the same port (lock-free accept)
- [ ] Reduce `std::string` copies in the hot path using `std::string_view`
- [ ] Preallocate HTTP response headers for common responses (200 OK, 404)
- [ ] Tune `epoll_wait()` timeout value

#### 🧪 Experiment

For each optimization, benchmark before and after. Use the exact same `wrk` command and conditions.

#### 🔬 What to Observe

- Did each optimization actually improve things or make things worse?
- Did the flamegraph change after the optimization?

#### 🧠 Questions

- [ ] What is `SO_REUSEPORT` and how does it scale across cores?
- [ ] What is a memory pool and when should you use one?
- [ ] What is the cost of `std::string` allocation in a tight loop?

#### 🧪 Tests

- [ ] All existing tests still pass after each optimization
- [ ] No regressions in correctness

#### 📊 Benchmark

| Metric | Before | After | Delta |
|---|---|---|---|
| req/sec | ? | ? | +?% |
| p50 latency | ? ms | ? ms | -?ms |
| p99 latency | ? ms | ? ms | -?ms |
| CPU% | ? | ? | -?% |
| Memory RSS | ? MB | ? MB | -?MB |

#### ✅ Definition of Done

- [ ] At least 2 optimizations implemented and benchmarked
- [ ] Benchmark results documented with before/after comparison
- [ ] Flamegraph after optimization shows the hotspot reduction
- [ ] `docs/optimization-results.md` committed

#### 📝 Git Commit

`perf: connection pool, SO_REUSEPORT, string_view hot path, benchmark`

---

## Phase 13 — Innovation: Intelligent Backpressure

### Day 37 — Backpressure Design & Implementation

#### 🎯 Goal
Design and implement an intelligent backpressure system that detects server overload and automatically reduces incoming traffic.

#### 📚 Concepts

- [[Backpressure]] — flow control mechanism
- [[Load Shedding]] — reject requests when overloaded
- [[Circuit Breaker]] — automatically open/close based on error rate
- [[Admission Control]] — regulate inflow based on capacity
- [[Little's Law]] — L = λW (queue length = arrival rate × wait time)

#### 💻 Implementation

- [ ] Implement `LoadMonitor` that tracks:
  - Active connection count
  - Request queue depth
  - Average request latency (rolling 5-second window)
  - CPU usage (via `/proc/stat`)
  - Write buffer sizes
- [ ] Implement `BackpressureController` with three states: `Normal`, `Throttled`, `Overloaded`
- [ ] In `Throttled` state: accept connections but return 503 for new requests
- [ ] In `Overloaded` state: stop accepting new connections, send 503 with `Retry-After` header
- [ ] Implement hysteresis: must stay below threshold for 5 seconds before returning to `Normal`

#### 🧪 Experiment

Create an artificial overload: add a 100ms sleep in a handler. Run `wrk -t16 -c1000`. Watch the backpressure controller activate and deactivate.

#### 🔬 What to Observe

- At what connection count does the controller enter `Throttled` state?
- How does the 503 rate change over time?
- Does the server stabilize rather than crash under overload?

#### 🧠 Questions

- [ ] What is Little's Law and how does it apply to web servers?
- [ ] What is the difference between load shedding and rate limiting?
- [ ] What is a circuit breaker pattern?
- [ ] How does backpressure propagate through a distributed system?

#### 🧪 Tests

- [ ] Controller enters `Throttled` state when connection count exceeds threshold
- [ ] Controller returns to `Normal` after load drops for 5 seconds
- [ ] `Retry-After` header correctly set in 503 responses

#### 📊 Benchmark

Compare server stability (request error rate) under extreme load: with and without backpressure.

#### ✅ Definition of Done

- [ ] Backpressure controller implemented with three states
- [ ] Hysteresis working correctly
- [ ] Server remains stable under extreme load with backpressure

#### 📝 Git Commit

`feat: intelligent backpressure controller, load monitor, 503 with Retry-After`

---

### Day 38 — Backpressure Tuning & Documentation

#### 🎯 Goal
Tune the backpressure system, add a `/debug/backpressure` endpoint, and document the design as a portfolio piece.

#### 📚 Concepts

- [[Adaptive Systems]] — self-tuning based on observed behavior
- [[Control Theory]] — feedback loops
- [[Technical Writing]] — documenting complex systems clearly

#### 💻 Implementation

- [ ] Add `/debug/backpressure` endpoint showing: current state, thresholds, metrics, time in current state
- [ ] Add configurable thresholds (via config file or environment variables)
- [ ] Experiment with different threshold values under `wrk` load
- [ ] Write `docs/backpressure-design.md`: the problem, the design, the tradeoffs, the results
- [ ] Add backpressure metrics to `/metrics`: `backpressure_state`, `requests_shed_total`

#### 🧪 Experiment

Run the server at exactly its saturation point (find it with `wrk`). Enable backpressure. Observe the system oscillating between Normal and Throttled states. Tune thresholds to achieve stability.

#### 🔬 What to Observe

- Does the system oscillate or stabilize?
- What is the impact on p99 latency with backpressure active?

#### 🧠 Questions

- [ ] What is the relationship between backpressure and p99 latency?
- [ ] How do production systems like NGINX implement admission control?

#### 🧪 Tests

- [ ] All backpressure states visible in `/debug/backpressure`
- [ ] Thresholds configurable without recompilation
- [ ] `docs/backpressure-design.md` complete

#### 📊 Benchmark

Final system benchmark: req/sec, error rate, p99 latency — at 50%, 100%, 150% of saturation point.

#### ✅ Definition of Done

- [ ] `/debug/backpressure` endpoint working
- [ ] Design document written and committed
- [ ] Backpressure metrics in `/metrics`
- [ ] System behaves stably at 150% of saturation point

#### 📝 Git Commit

`feat: backpressure debug endpoint, configurable thresholds, design doc`

---

## Phase 14 — Testing & Productionization

### Day 39 — Testing Suite

#### 🎯 Goal
Build a comprehensive test suite: unit tests, integration tests, stress tests, and sanitizer-clean runs.

#### 📚 Concepts

- [[GoogleTest]] — C++ testing framework
- [[AddressSanitizer]] — memory error detection
- [[ThreadSanitizer]] — data race detection
- [[UndefinedBehaviorSanitizer]] — UB detection
- [[Fuzz Testing]] — automated input generation

#### 💻 Implementation

- [ ] Add GoogleTest or Catch2 to CMake
- [ ] Write unit tests for: `HttpParser`, `HttpFSMParser`, `Router`, `WriteBuffer`, `ReadBuffer`, `MetricsCollector`, `BackpressureController`
- [ ] Write integration tests: start server in-process, send real HTTP requests, verify responses
- [ ] Write stress test: 10,000 requests with 100 concurrent connections, verify 0 errors
- [ ] Write malformed request tests: fuzzy inputs, truncated requests, binary garbage
- [ ] Run all tests with AddressSanitizer: `cmake -DCMAKE_CXX_FLAGS="-fsanitize=address"`
- [ ] Run all tests with ThreadSanitizer: `cmake -DCMAKE_CXX_FLAGS="-fsanitize=thread"`
- [ ] Run all tests with UndefinedBehaviorSanitizer: `cmake -DCMAKE_CXX_FLAGS="-fsanitize=undefined"`
- [ ] If time permits: add libFuzzer target for the HTTP parser

#### 🧪 Experiment

Run the fuzz target for 5 minutes. How many unique inputs does it find? Does it crash the parser?

#### 🔬 What to Observe

- Do any sanitizers find issues that unit tests missed?
- Does ThreadSanitizer find any races that your manual testing missed?

#### 🧠 Questions

- [ ] What types of bugs does AddressSanitizer catch that unit tests don't?
- [ ] What is the difference between fuzz testing and unit testing?
- [ ] What is a corpus in fuzz testing?

#### 🧪 Tests

- [ ] All unit tests passing
- [ ] All integration tests passing
- [ ] AddressSanitizer clean
- [ ] ThreadSanitizer clean
- [ ] UndefinedBehaviorSanitizer clean

#### 📊 Benchmark

Run full test suite and record total test count and execution time.

#### ✅ Definition of Done

- [ ] All three sanitizers clean
- [ ] > 80% code coverage (estimated)
- [ ] Fuzz target runs without crashes
- [ ] CI configuration includes sanitizer runs

#### 📝 Git Commit

`test: full test suite, sanitizer clean, integration tests, fuzz target`

---

### Day 40 — Production Packaging & Portfolio

#### 🎯 Goal
Package ForgeHTTP as a professional, portfolio-ready project. Write the README, Dockerfile, and GitHub Actions CI.

#### 📚 Concepts

- [[Docker]] — containerization
- [[GitHub Actions]] — CI/CD
- [[CMake Install]] — `cmake --install`
- [[Technical Documentation]] — README, architecture docs
- [[Portfolio]] — presenting work to interviewers

#### 💻 Implementation

- [ ] Write comprehensive `README.md` (see Portfolio section for structure)
- [ ] Create `Dockerfile` using multi-stage build: builder (gcc/cmake) + runtime (minimal debian)
- [ ] Create `.github/workflows/ci.yml`: build, test with all three sanitizers
- [ ] Add `config/forgehttp.conf` configuration file with all tunables
- [ ] Add `.clang-format` with Google C++ style
- [ ] Add `examples/` directory with: basic server, reverse proxy, static file server
- [ ] Ensure `cmake --install` works correctly
- [ ] Write `docs/architecture.md` with the full Mermaid diagram
- [ ] Write `docs/design-decisions.md` documenting the 5 most important design choices
- [ ] Tag v0.1.0 release

#### 🧪 Experiment

Clone the repository into a fresh directory and follow the README steps exactly. Does it build and run without any prior knowledge? Fix anything that breaks.

#### 🔬 What to Observe

- Is the README truly self-contained?
- Does the Docker build complete successfully?
- Do GitHub Actions pass on all platforms?

#### 🧠 Questions

- [ ] What makes a GitHub project stand out to an interviewer?
- [ ] What design decisions are most interesting to discuss in a technical interview?
- [ ] How would you explain the epoll event loop in 2 minutes?

#### 🧪 Tests

- [ ] Docker build succeeds
- [ ] GitHub Actions CI passes
- [ ] Fresh clone builds and runs following README

#### 📊 Benchmark

Final published benchmark in README: req/sec, p99 latency, concurrent connections tested.

#### ✅ Definition of Done

- [ ] README complete and self-contained
- [ ] Dockerfile builds and runs
- [ ] GitHub Actions CI passing
- [ ] All documentation committed
- [ ] v0.1.0 tagged on GitHub

#### 📝 Git Commit

`release: v0.1.0 — ForgeHTTP production packaging, README, Dockerfile, CI`

---

## 🔬 Research Questions

Answer these questions during the project. Write an Obsidian note for each.

### TCP & Sockets

- [ ] What happens inside the kernel when `socket()` is called?
- [ ] Why is a socket represented by a file descriptor?
- [ ] What does `listen(backlog)` do and what exactly is the backlog queue?
- [ ] What happens during `accept()` — which side sends the ACK?
- [ ] Why does `bind()` fail with `EADDRINUSE` and how does `SO_REUSEADDR` fix it?
- [ ] What is the difference between `SO_REUSEADDR` and `SO_REUSEPORT`?
- [ ] What is `TIME_WAIT` and why does it last 2×MSL?

### HTTP & Protocol

- [ ] Why is TCP a byte stream and not a message stream?
- [ ] Why can `recv()` return partial data?
- [ ] Why does HTTP need message framing (Content-Length or chunked)?
- [ ] What exactly is `\r\n` and why does HTTP use CRLF?
- [ ] What is the difference between HTTP/1.0 and HTTP/1.1 for connection handling?
- [ ] What is HTTP/2 multiplexing and how does it differ from HTTP/1.1 keep-alive?

### Non-Blocking I/O & epoll

- [ ] What does `O_NONBLOCK` actually change in the kernel?
- [ ] What is the difference between `EAGAIN` and `EWOULDBLOCK`?
- [ ] Why does `epoll` scale to millions of connections while `select()` cannot?
- [ ] What is the difference between level-triggered and edge-triggered epoll?
- [ ] What is `EPOLLONESHOT` and when should you use it?
- [ ] What is `io_uring` and how does it differ from `epoll`?
- [ ] What is an event loop and what is the Reactor pattern?

### Concurrency

- [ ] What is a race condition and how do you detect one?
- [ ] What is the difference between `std::mutex` and `std::atomic`?
- [ ] What is false sharing and how does it hurt performance?
- [ ] What is the C++ memory model?
- [ ] What is `memory_order_acquire` vs `memory_order_release`?
- [ ] What causes deadlocks and how do you prevent them?
- [ ] What is the C10K problem?

### Performance & OS

- [ ] What does zero-copy actually mean at the DMA level?
- [ ] What happens inside `sendfile()`?
- [ ] What is the Linux page cache?
- [ ] What is a context switch and what does it cost?
- [ ] What is a CPU cache miss and how does it affect throughput?
- [ ] What is a flamegraph and how do you read one?

### Distributed Systems

- [ ] What is backpressure and why is it essential?
- [ ] What is the circuit breaker pattern?
- [ ] What is Little's Law?
- [ ] What is the difference between rate limiting and load shedding?
- [ ] How does graceful shutdown work in production systems?
- [ ] What happens when a client disconnects unexpectedly during a response?

---

## 🗺 Learning Map

| Feature | Networking | OS/Linux | C++ | Concurrency | Systems Design |
|---|---|---|---|---|---|
| TCP Server | TCP, 3-way handshake | Sockets, FDs, syscalls | RAII, smart pointers | — | Client/Server model |
| HTTP Parser | HTTP/1.1, framing | Buffers, kernel buffers | `string_view`, FSM, enum class | — | Protocol design |
| Thread Pool | — | Scheduling, context switch | `mutex`, `condition_variable` | Producer/Consumer | Work queues |
| epoll | I/O readiness, events | Kernel I/O, event queues | Event abstractions, callbacks | Event Loop | Reactor pattern |
| Static Files | HTTP caching, ETags | `mmap`, `sendfile`, page cache | `std::filesystem` | — | I/O optimization |
| Reverse Proxy | TCP/HTTP forwarding | Multiple sockets | Interfaces, polymorphism | Connection pooling | Distributed systems |
| Metrics | HTTP endpoints | `/proc/stat`, atomics | `std::atomic`, histograms | Concurrent counters | Observability |
| Backpressure | Flow control | CPU monitoring | State machines | Atomic state | Admission control |

---

## 🏛 Final Architecture

\`\`\`mermaid
flowchart TD
    Client["Client Browser/curl/wrk"]
    Listener["TCP Listener SO_REUSEPORT"]
    EventLoop["epoll Event Loop edge-triggered"]
    ConnMgr["Connection Manager RAII idle timeout"]
    Parser["HTTP FSM Parser zero-copy string_view"]
    Router["Router trie path params"]
    MW["Middleware Pipeline logger auth CORS"]
    Static["Static File Handler sendfile ETag Range"]
    AppHandler["Application Handler custom routes"]
    Proxy["Reverse Proxy round-robin LB"]
    Upstream["Upstream Pool connection pool health checks"]
    Metrics["Observability /metrics /debug/stats"]
    Response["HTTP Response Builder chunked keep-alive"]
    WriteBuf["Write Buffer EPOLLOUT backpressure"]
    BPCtrl["Backpressure Controller 503 Retry-After"]

    Client --> Listener
    Listener --> BPCtrl
    BPCtrl --> EventLoop
    EventLoop --> ConnMgr
    ConnMgr --> Parser
    Parser --> Router
    Router --> MW
    MW --> Static
    MW --> AppHandler
    MW --> Proxy
    Proxy --> Upstream
    Static --> Response
    AppHandler --> Response
    Upstream --> Response
    Response --> WriteBuf
    WriteBuf --> EventLoop
    ConnMgr --> Metrics
\`\`\`

---

## 📁 Suggested Obsidian Vault Structure

\`\`\`text
00 - Index
01 - C++
    RAII
    Smart Pointers
    Move Semantics
    std::string_view
    Lambdas
    Atomics
    std::mutex
    std::condition_variable
    Memory Model
02 - Linux
    File Descriptors
    System Calls
    User Space vs Kernel Space
    Linux Processes
    Linux Threads
    Virtual Memory
    Context Switching
    proc filesystem
03 - Networking
    TCP
    TCP Handshake
    Socket Programming
    Berkeley Sockets
    TIME_WAIT
    TCP Buffering
    Nagle Algorithm
04 - HTTP
    HTTP Parser
    HTTP Methods
    HTTP Status Codes
    HTTP Keep-Alive
    Chunked Transfer Encoding
    MIME Types
    ETag
    Cache-Control
05 - Concurrency
    Thread Pool
    Mutex
    Atomic Operations
    Race Conditions
    Deadlocks
    False Sharing
    Memory Ordering
06 - Performance
    Profiling
    Flamegraph
    Zero Copy
    sendfile
    mmap
    SO_REUSEPORT
    Benchmarking
07 - Security
    HTTP Security
    Path Traversal
    Slowloris
    Header Injection
    Request Limits
08 - Systems Design
    Reactor Pattern
    Event Loop
    Backpressure
    Load Balancing
    Circuit Breaker
    Connection Pooling
    Observability
09 - ForgeHTTP Project
    Architecture
    Design Decisions
    Benchmark Results
    Profiling Results
    Backpressure Design
\`\`\`

---

## 💼 Portfolio / GitHub Strategy

### GitHub README Structure

\`\`\`text
# ForgeHTTP

High-performance HTTP/1.1 server built from scratch in modern C++20.
Implements epoll-based event-driven I/O, FSM HTTP parsing, reverse proxy
with load balancing, and intelligent backpressure.

## Overview
## Features
## Architecture (Mermaid diagram)
## How It Works
## Performance
## Benchmarks (wrk results table)
## Security
## Design Decisions
## What I Learned
## Networking Concepts
## OS Concepts
## C++ Concepts
## Testing
## Running Locally
## Docker
## Configuration
## Future Work
\`\`\`

### Key Design Decisions to Document

1. **Why FSM parser** — Explain incremental parsing, partial reads, and why a simple `split("\r\n")` is wrong
2. **Why epoll over select/poll** — Explain O(1) readiness notification vs O(n) scanning
3. **Why edge-triggered with read-until-EAGAIN** — Explain the correctness requirement
4. **Why RAII for connections** — Explain how resource leaks are prevented by design
5. **Why backpressure** — Explain the stability property under overload

---

## 📄 Resume Bullets

> Engineered ForgeHTTP, a production-inspired HTTP/1.1 server in C++20 using a custom epoll-based Reactor event loop, serving 85,000 req/s at sub-2ms p99 latency under 10,000 concurrent connections on a 4-core Linux machine.

> Implemented a zero-copy finite-state-machine HTTP parser using std::string_view, eliminating heap allocations in the hot path and achieving 3× throughput improvement over a naive copy-based implementation (benchmarked with wrk and perf).

> Built a reverse proxy with round-robin load balancing, upstream connection pooling, health checks, and configurable timeouts; profiled with Linux perf and flamegraphs to identify and resolve bottlenecks, reducing per-request CPU overhead by 40%.

> Designed an intelligent backpressure controller using atomic load monitoring across CPU, connection count, and write buffer depth — automatically shedding load via 503 responses to maintain server stability at 150% of saturation point.

> Hardened the server against adversarial HTTP inputs using AddressSanitizer, ThreadSanitizer, and libFuzzer, achieving sanitizer-clean builds under sustained concurrent load; packaged with Dockerfile and GitHub Actions CI with multi-sanitizer test matrix.

---

## 🚀 Day 40 — Final Checklist

### Core Networking

- [ ] TCP server with full socket lifecycle (socket → bind → listen → accept → recv/send → close)
- [ ] Socket lifecycle observable with `ss`, `lsof`, `tcpdump`
- [ ] HTTP/1.1 parser with FSM, incremental parsing, partial reads
- [ ] Non-blocking I/O with correct `EAGAIN` handling
- [ ] epoll event loop (level-triggered and edge-triggered understood)
- [ ] Keep-alive persistent connections

### C++

- [ ] RAII for all resources (sockets, file descriptors, connections)
- [ ] Move semantics in Buffer and Connection classes
- [ ] `std::string_view` used in parser hot path (zero-copy)
- [ ] `std::atomic` for all shared counters (no mutex on hot path)
- [ ] Smart pointers for all heap-allocated objects
- [ ] CMake build system with modern practices

### OS / Linux

- [ ] File descriptors understood (every resource is an FD)
- [ ] System calls traced with `strace` and understood
- [ ] `epoll_create1`, `epoll_ctl`, `epoll_wait` used correctly
- [ ] `sendfile()` used for static file serving
- [ ] `/proc/stat` read for CPU metrics
- [ ] Kernel/user-space boundary understood

### Server Features

- [ ] HTTP parser: method, URI, headers, body, query params
- [ ] HTTP responses: status codes, headers, Content-Length, Connection
- [ ] Static file server: MIME types, ETag, Last-Modified, 304, Range
- [ ] Router: exact routes, path parameters, wildcards
- [ ] Middleware pipeline: logger, CORS, timing
- [ ] Reverse proxy: upstream forwarding, round-robin, health checks
- [ ] Connection pooling for upstream
- [ ] Chunked transfer encoding
- [ ] Observability: `/metrics`, `/debug/stats`, request IDs
- [ ] Security: limits, timeouts, path traversal, Slowloris protection
- [ ] Backpressure controller with three states

### Performance

- [ ] `wrk` load testing at 10, 100, 1000, 10000 connections
- [ ] `perf stat` profiling under load
- [ ] Flamegraph generated and analyzed
- [ ] At least 2 optimizations implemented and benchmarked
- [ ] Optimization report: before/after comparison documented

### Portfolio

- [ ] README: overview, features, architecture, benchmarks, design decisions
- [ ] Architecture Mermaid diagram in README
- [ ] Benchmark table in README (req/sec, p99 at various concurrency levels)
- [ ] All tests passing (unit, integration, stress)
- [ ] AddressSanitizer clean
- [ ] ThreadSanitizer clean
- [ ] Dockerfile builds and runs
- [ ] GitHub Actions CI passing on all sanitizer configurations
- [ ] v0.1.0 tagged
- [ ] `docs/` directory: architecture, design decisions, profiling, backpressure design
- [ ] Technical write-up: "How ForgeHTTP Works"

---

## 🔮 Future Improvements (Post Day 40)

- [ ] **HTTP/2** — HPACK header compression, stream multiplexing, server push
- [ ] **TLS/SSL** — integrate with OpenSSL or BoringSSL, ALPN for HTTP/2
- [ ] **io_uring** — replace epoll with io_uring for async I/O with fewer syscalls
- [ ] **WebSocket** — upgrade handshake and frame parsing
- [ ] **gRPC** — binary protocol support over HTTP/2
- [ ] **Zero-downtime reload** — hot-swap configuration without dropping connections
- [ ] **Dynamic upstream discovery** — consul/etcd integration
- [ ] **Prometheus metrics** — native Prometheus text format on `/metrics`
- [ ] **Distributed tracing** — W3C TraceContext header propagation
- [ ] **QUIC/HTTP/3** — UDP-based transport

---

#cpp #networking #linux #systems #http #portfolio #project #roadmap
