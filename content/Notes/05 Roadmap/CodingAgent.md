# Mini Devin — Complete Build Roadmap

**Approach:** Build While Learning. Every phase teaches one core skill and immediately ships a working component. Total estimated time: **9–13 weeks** (part-time pace).

---

## Phase 0: Toy Agent (De-risk the Core Loop)
**Duration:** 3–4 days
**Goal:** Prove the fundamental "LLM calls tools in a loop" pattern works *before* investing in infrastructure. This is the riskiest part of the whole project — validate it first, cheaply.

**What to Learn:**
- Basic tool-calling / function-calling API (OpenAI, Anthropic, etc.)
- The Observe → Plan → Act loop in its simplest form
- Why agents loop, stall, or hallucinate tool args

**What to Build:**
- A single Python script (no Docker, no WebSockets, no tree-sitter).
- Three tools implemented with plain `subprocess`/`open()`: `read_file`, `write_file`, `run_shell`.
- A `while` loop: send task → LLM picks a tool or finishes → execute → feed result back → repeat.
- Test it on one real task: "Fix this failing test in this small repo."

**Why this matters:** If your core reasoning loop is unreliable, no amount of infrastructure in later phases will fix that. You want to discover this on day 3, not week 8.

---

## Phase 1: Real-Time Async Engine & WebSockets
**Duration:** 1–2 weeks
**Goal:** Learn to manage async execution and push live events (token streaming, terminal output, status updates) to a client.

**What to Learn:**
- Async Python primitives: `asyncio.TaskGroup`, `asyncio.Queue`, non-blocking IO
- FastAPI `WebSocket` endpoints and connection management
- Pydantic `BaseModel` for structured event schemas (`ThoughtEvent`, `ToolOutputEvent`)

**What to Build:**
- A FastAPI WebSocket server that accepts a simulated multi-step task name.
- An `asyncio.Queue` background worker executing 5 mock "steps" with variable delays, streaming structured JSON events over the socket as each step completes.

**Improvisation — add a minimal frontend now, not later:**
- A single-page HTML/JS client (no framework needed) with:
  - A live event/thought stream pane
  - A raw terminal-output pane
- This makes every later phase visibly testable instead of "read JSON in a WebSocket debugger," and it's motivating to actually *see* the agent think.

---

## Phase 2: Secure Workspace & Execution Sandbox
**Duration:** 1–2 weeks
**Goal:** Isolate AI-generated code execution so it cannot affect your host OS — and can't be used to exfiltrate data or hammer external services either.

**What to Learn:**
- The `docker` Python SDK: creating, starting, stopping, binding volumes
- Programmatic git management via `GitPython`
- Capturing live `stdout`/`stderr` from containers
- **Container hardening:** resource limits, network policy, filesystem restrictions

**What to Build:**
- A `WorkspaceManager` module:
  1. Clone a sample GitHub repo into a temp host folder via `GitPython`.
  2. Mount that folder as a volume inside an **unprivileged** Docker container.
  3. Expose `execute_command(cmd: str)` that runs shell commands (`pytest`, `pip install`, `git status`) inside the container and yields output line-by-line via an async generator.

**Improvisation — harden the sandbox beyond "just Docker":**
- Set `mem_limit`, `cpu_quota`/`cpu_period`, and `pids_limit` so a runaway process can't take down your host.
- Mount the container filesystem **read-only** except the workspace volume.
- Restrict or fully disable network egress by default — an agent that can run arbitrary shell commands and also has open internet access is a serious risk (`curl | sh`, data exfiltration). Only allowlist specific hosts if a task genuinely needs them (e.g., `pypi.org` for installs).
- Note for later: if you want isolation closer to what Devin/production agents use, look into **gVisor** or **Firecracker** microVMs instead of plain Docker — optional, not required for the learning project.

---

## Phase 3: Codebase AST Indexing & Structural + Semantic Search
**Duration:** 1–2 weeks
**Goal:** Parse source code structurally using syntax trees instead of relying solely on raw text/regex chunking — and add fuzzy search for when the agent doesn't know exact symbol names.

**What to Learn:**
- AST concepts: nodes, leaves, parent-child relationships
- `tree-sitter` and `tree-sitter-python`
- Writing SCM queries to extract function defs, imports, class hierarchies
- **Embeddings basics** and a lightweight vector store

**What to Build:**
- A `CodebaseIndexer` that parses a target Python repo with `tree-sitter` and produces:
  - File trees and module imports
  - Class definitions, function signatures, line spans
- Exact-match search functions: `get_symbol_definition(symbol_name)`, `list_file_structure()`

**Improvisation — add semantic search alongside structural search:**
- Chunk each function/class extracted by the AST indexer.
- Embed the chunks and store them in a lightweight local vector DB (e.g., `chromadb` or even `faiss`).
- Expose `semantic_search(query: str)` so the agent can ask "find code related to authentication" without knowing the exact function name — structural search alone can't answer that kind of query.

---

## Phase 4: State Machine & Agent Execution Loop
**Duration:** 2–3 weeks
**Goal:** Manage the cyclic Observe → Plan → Act → Reflect loop and recover from errors — this time for real, wired into your Phase 1–3 infrastructure.

**What to Learn:**
- **LangGraph**: nodes, state schemas, conditional edges, checkpoint persistence
- Structured Outputs: enforcing Pydantic schemas on LLM outputs (native structured-output APIs or `instructor`)
- Multi-agent separation: *Planner* (breaks down tasks) vs *Executor* (writes code, invokes tools)
- **Code-editing strategy: diffs vs whole-file rewrites**

**What to Build:**
- A LangGraph agent loop:
  1. **Planning Node** — breaks a user request into a step list.
  2. **Execution Node** — selects a tool (`write_file`, `read_file`, `run_docker_cmd`) via structured LLM calls.
  3. **Reflection Node** — analyzes `pytest` stdout/stderr; on failure, feeds the error back into agent state to generate a fix.

**Improvisation — decide your edit strategy explicitly:**
- Whole-file rewrites are simple but expensive and error-prone on large files.
- Most production agents use **unified diffs** or **search-replace blocks** for edits instead. Pick one and implement it as a dedicated tool (`apply_patch`) rather than letting the LLM rewrite entire files by default — this directly affects how reliably your Reflection node can apply fixes.

---

## Phase 5: Integration, Guardrails & Evaluation
**Duration:** 2 weeks
**Goal:** Connect all components into an end-to-end platform with real safety guardrails and measurable performance.

**What to Learn:**
- Input/output guardrails: path traversal (`../`) prevention, dangerous command filtering (`rm -rf`, `curl | sh`), token usage limits
- Evaluation & benchmarking: test-pass rate, cost per task, trajectory length
- Traceability: **Langfuse** or **LangSmith** for step-by-step debugging

**What to Build:**
- Combine Phases 1–4 into one application:
  - **Backend:** FastAPI orchestrating LangGraph runs over WebSockets, executing inside Docker sandboxes, querying codebase AST + semantic indexes.
  - **Evaluation Suite:** an automated test runner that gives Mini Devin 5 real GitHub issues (small bug-fix tasks with pre-written unit tests) and measures pass rate with no manual intervention.

**Improvisation — expand the guardrail list:**
- **Secrets/credential scanning** before the agent writes files or runs commands (catch accidental leaked API keys, `.env` contents, etc.)
- **Hard per-task timeout / watchdog** — agents can loop indefinitely without one.
- **Prompt-injection defense** for any content the agent ingests from tool outputs (web pages, issue trackers, file contents). Instructions embedded in that content should never be treated as trusted user instructions.

---

## Summary Timeline

| Phase | Focus | Duration |
|---|---|---|
| 0 | Toy agent — validate the core reasoning loop | 3–4 days |
| 1 | Async engine + WebSockets + minimal frontend | 1–2 weeks |
| 2 | Hardened Docker sandbox + GitPython | 1–2 weeks |
| 3 | AST + semantic codebase search | 1–2 weeks |
| 4 | LangGraph agent loop + diff-based editing | 2–3 weeks |
| 5 | Integration, guardrails, evaluation suite | 2 weeks |

**Total: ~9–13 weeks**, depending on pace and how deep you go into sandbox hardening (gVisor/Firecracker) and eval breadth.