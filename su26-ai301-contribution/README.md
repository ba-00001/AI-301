# Contribution #1: Add a JSON `/health` endpoint to the Tessera.io Python AI service

**Contribution Number:** 1
**Student:** Brian Bazurto
**Project:** [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io) — an open-source collaborative developer sandbox with real-time CRDT sync and secure remote code execution for human-AI pair programming
**Issue:** [#39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**Status:** Phase II — Reproduce & Plan (Complete)

---

## Why I Chose This Issue

I chose issue #39 because it is a clean, self-contained task with clear acceptance
criteria, and it lines up directly with what I'm learning in AI301. The work lives
in the Python AI microservice of Tessera.io, an open-source collaborative coding
sandbox built for human-AI pair programming — exactly the kind of AI-adjacent
codebase I want experience contributing to.

The task is to add a JSON `/health` route that reports database connectivity and
model status. It's well-scoped for a first contribution: it touches a single file
(`apps/ai-service/src/main.py`), it lets me practice FastAPI routing and service
health-check patterns, and "done" is unambiguous — operators can hit one endpoint
and see whether the service and its dependencies are up. The project has a
`CONTRIBUTING.md`, a `README.md`, and an `.env.example`, so I can get set up, and
the maintainer is actively triaging and shipping (the repo was pushed to today).

---

## Understanding the Issue

### Problem Description

The Tessera.io Python AI microservice (`apps/ai-service`) has no health-check
endpoint, so there's no quick, programmatic way to confirm that the service, its
database connection, and its model integrations are actually up and working.

### Expected Behavior

A `GET /health` route returns a JSON payload describing the service's health —
database connectivity statistics and model statuses — making it easy for operators
and monitoring to check the microservice at a glance.

### Current Behavior

A `/health` route _does_ exist, but it is a placeholder. The `health_check()`
handler at the bottom of `apps/ai-service/src/main.py` returns a hard-coded
`{"status": "ok"}` and inspects nothing — no database connection, no model/MCP
status. It was added in the very first scaffold commit (`1edd56f`, 2026-05-27,
"feat: scaffold monorepo structure …"), _before_ issue #39 was filed, which is why
the issue explicitly asks for connectivity **statistics** and **model statuses**
rather than a route that merely exists. So the route reports `ok` even when its
dependencies are completely down — I confirmed this during reproduction (see below).

### Affected Components

- `apps/ai-service/src/main.py` — the FastAPI app; `health_check()` is the function
  to expand.
- `apps/ai-service/src/db.py` — the Motor (`AsyncIOMotorClient`) connection lives
  here (`connect_db`, `close_db`, `get_collection`); a connectivity check needs a
  helper here.
- `apps/ai-service/src/config.py` — `Settings` holds `MONGODB_URI`,
  `MONGODB_DB_NAME`, `MONGODB_COLLECTION`, `EMBEDDING_DIMENSIONS`, and
  `MCP_SERVER_NAME` — the values the health check should report.
- `apps/ai-service/src/mcp_server.py` / `rag.py` — the "model" side: the MCP server
  and the (currently placeholder) embedding pipeline whose status the check reports.

---

## Reproduction Process

### Environment Setup

I set up the Python AI microservice (`apps/ai-service`) following the project
`README.md` (Typical Case — README setup instructions). The README lists the steps:
create a venv, `pip install -r requirements.txt`, start MongoDB on `27017` via
Docker, and run the stack with `npm run dev` (Turborepo). I ran the AI service
directly with `uvicorn` so I could isolate the one endpoint I care about.

Friction I hit and how I resolved it:

- **Python version mismatch.** My system Python is 3.9.6, but the project requires
  **Python ≥ 3.11** (the README states this, and the code uses 3.10+ syntax like
  `dict[str, str]` and `X | None`). `pip install` failed on
  `mcp[cli]>=1.9.0` with `No matching distribution found` because that package
  needs Python ≥ 3.10. **Fix:** I installed [`uv`](https://docs.astral.sh/uv/)
  (`brew install uv`) and created the venv against a managed interpreter:
  `uv venv --python 3.12 .venv && uv pip install -r requirements.txt`. All
  dependencies (fastapi, uvicorn, motor, mcp, …) then installed cleanly.
- **No MongoDB needed to reproduce.** The bug is in a route that _doesn't_ touch
  the database, so I deliberately left MongoDB **not** running — that turns out to
  be the cleanest way to prove the defect (see Steps to Reproduce). Motor connects
  lazily, so the app still starts without a database.

Run command used:

```bash
cd apps/ai-service
uv venv --python 3.12 .venv && source .venv/bin/activate
uv pip install -r requirements.txt
uvicorn src.main:app --host 127.0.0.1 --port 8000
```

**Working branch:** https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint

### Steps to Reproduce

1. Start the AI service (no MongoDB running): `uvicorn src.main:app --host 127.0.0.1 --port 8000`.
2. Confirm MongoDB is **not** listening: `nc -z -w2 127.0.0.1 27017` → connection refused.
3. Request the endpoint: `curl -s -i http://127.0.0.1:8000/health`.
4. **Expected (per issue #39):** JSON reporting database connectivity statistics and
   model statuses — e.g. whether MongoDB is reachable, and the state of the
   model/MCP components.
5. **Actual:** `HTTP/1.1 200 OK` with body `{"status":"ok"}` — a static string with
   no DB or model information, returned even though MongoDB is unreachable. The
   route claims the service is healthy when its primary dependency is down.
6. Repeated the request a second time (and a third after restart): identical
   `{"status":"ok"}` every time — the behavior is consistent, not a fluke.

### Reproduction Evidence

- `GET /health` response (MongoDB down): `200 OK`, `content-type: application/json`,
  body `{"status":"ok"}`.
- Registered routes from `/openapi.json`: `['/rag/ingest', '/rag/search', '/health']`
  — confirms `/health` is the placeholder route and nothing reports dependency state.
- The handler is `health_check()` in `apps/ai-service/src/main.py` (the last route in
  the file): `async def health_check() -> dict[str, str]: return {"status": "ok"}`.
- `git log -S "def health_check" -- apps/ai-service/src/main.py` shows it was
  introduced in scaffold commit `1edd56f` (2026-05-27), predating the issue.
- Branch with these notes: https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint

---

## Solution Approach

### Analysis

**Root cause.** The `/health` route is a scaffold placeholder. `health_check()` in
`apps/ai-service/src/main.py` is hard-coded to `return {"status": "ok"}` and never
reads any dependency state, so it reports a healthy service unconditionally — even
with MongoDB down, as I reproduced. The fix is not "add a route" (one exists) but
"make the existing route actually inspect its dependencies and report their state,"
which is exactly what issue #39 asks for (connectivity statistics + model statuses).

### Proposed Solution

Expand `health_check()` so it pings MongoDB and reports component status as JSON:
an overall `status`, a `database` block (reachable? + which DB/collection from
settings + ping latency), and a `models` block (MCP server name, embedding
dimensions, and which model/MCP components are wired vs. placeholder). Return
HTTP `200` when healthy and `503` when a critical dependency (the database) is
unreachable, so monitoring can act on the status code as well as the body.

### Implementation Plan (UMPIRE)

**Understand:** `/health` exists but is a static stub returning `{"status":"ok"}`;
it never checks MongoDB or the model/MCP layer, so it lies about service health
when dependencies are down. Issue #39 wants `/health` to return DB connectivity
statistics and model statuses so operators can check the microservice at a glance.

**Match:** The codebase already has the patterns I need.

- DB access lives in `apps/ai-service/src/db.py` via the Motor client
  (`AsyncIOMotorClient`); `get_collection()` shows how the client is reached.
  MongoDB exposes a standard `await client.admin.command("ping")` for a cheap
  connectivity probe — the natural way to turn "is the DB up?" into a statistic.
- Component status is already represented as JSON dicts elsewhere — e.g.
  `execute_code()` in `mcp_server.py` returns `{"status": "not_implemented", …}`,
  and the timing middleware in `main.py` already inspects each request and writes
  structured data. I'll mirror that "status-as-dict" shape for the health payload.
- Config values to report come from `Settings` in `config.py` (`MONGODB_DB_NAME`,
  `MONGODB_COLLECTION`, `EMBEDDING_DIMENSIONS`, `MCP_SERVER_NAME`).

**Plan:**

1. In `apps/ai-service/src/db.py`, add an async helper (e.g. `check_connection()`)
   that runs `await _client.admin.command("ping")`, times it, and returns
   `{connected: bool, latency_ms: float | None, database, collection, error?}` —
   handling the "not connected yet" and "DB unreachable" cases without raising.
2. In `apps/ai-service/src/main.py`, rewrite `health_check()` to call that helper,
   assemble a `models`/MCP block from `settings` (`MCP_SERVER_NAME`,
   `EMBEDDING_DIMENSIONS`, embedding provider = placeholder for now), compute an
   overall `status` (`"ok"` / `"degraded"`), and return JSON. Set the HTTP status
   to `503` via `Response`/`JSONResponse` when the database is unreachable.
3. Keep the response a plain JSON object (optionally a small Pydantic
   `HealthResponse` model, matching the `BaseModel` style used in `rag.py`).

**Implement:** _(Phase III)_ — work happens on the branch:
https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint

**Review:** Self-review against the project `CONTRIBUTING.md` before the PR:
`/claim` issue #39, star + fork the repo, branch off `main`, **sign off every
commit** (`git commit -s` — CI rejects unsigned commits), use a Conventional
Commit message (`feat: …`), and run the project's Python lint
(`python3 -m ruff check . && python3 -m ruff format --check .`, per the
`lint` script in `apps/ai-service/package.json`).

**Evaluate:**

- _Manual:_ With MongoDB **up**, `GET /health` → `200` and `database.connected: true`
  with a latency figure; with MongoDB **down**, `GET /health` → `503` and
  `database.connected: false`. (Today's stub returns `200 {"status":"ok"}` in both
  cases — that difference is the proof the fix works.)
- _Automated:_ Add `pytest` tests using FastAPI's `TestClient` / `httpx`, mocking
  the Motor `ping` for both the healthy and unhealthy paths, asserting the status
  code and JSON shape. (`apps/ai-service` has no test suite yet, so I'll add a
  `tests/` folder; I'll confirm the expected test layout in the PR description.)

---

## Testing Strategy

_(Tests are written in Phase III; the plan below is set from the Phase II
investigation.)_

### Unit Tests

- [ ] `GET /health` returns `200` and the expected JSON shape (`status`, `database`,
      `models`) when the Motor `ping` succeeds (mocked healthy DB).
- [ ] `GET /health` returns `503` and `database.connected: false` when the `ping`
      raises / times out (mocked unreachable DB).
- [ ] The `models` block reports `MCP_SERVER_NAME` and `EMBEDDING_DIMENSIONS` from
      `Settings`.

### Integration Tests

- [ ] With a real MongoDB container up, the endpoint is reachable and reports
      `database.connected: true` with a latency value.

### Manual Testing

- [ ] Reproduce the baseline (MongoDB down → today's stub returns `200 {"status":"ok"}`).
- [ ] After the fix: MongoDB down → `503`; MongoDB up → `200` with live stats.

---

## Implementation Notes

### Week 1 Progress

Phase I — Issue Selection. Selected issue #39 on the Tessera.io Python AI service,
created this Contribution README, and worked through the issue-selection checklist.
Detailed tracker in [week-1/PHASE-1-CHECKLIST.md](week-1/PHASE-1-CHECKLIST.md).

### Week 2 Progress

Phase II — Reproduce & Plan. Stood up the `apps/ai-service` FastAPI service locally
(worked through a Python ≥ 3.11 version mismatch with `uv`), and reproduced the
issue: the `/health` route is a scaffold stub that returns `{"status":"ok"}` with no
DB or model information — it reports healthy even with MongoDB down. Wrote the
UMPIRE plan above (expand `health_check()` + add a Motor connectivity helper in
`db.py`). Detailed tracker in
[week-2/PHASE-2-CHECKLIST.md](week-2/PHASE-2-CHECKLIST.md).

### Code Changes

_None yet — code work begins in Phase III. Reproduction in Phase II was
investigation only (running the unmodified service)._

---

## Pull Request

_(Phase IV.)_

**PR Link:** _TBD_
**PR Description:** _TBD_
**Maintainer Feedback:** _TBD_
**Status:** Not yet opened

---

## Learnings & Reflections

### Technical Skills Gained

_To be filled in as I work._

### Challenges Overcome

_To be filled in as I work._

### What I'd Do Differently Next Time

_To be filled in as I work._

---

## Resources Used

- [Tessera.io repository](https://github.com/Kushaal-k/Tessera.io)
- [Tessera.io CONTRIBUTING.md](https://github.com/Kushaal-k/Tessera.io/blob/main/CONTRIBUTING.md)
- [Issue #39](https://github.com/Kushaal-k/Tessera.io/issues/39)
- [FastAPI documentation](https://fastapi.tiangolo.com/)
- [Motor (async MongoDB) — admin `ping`](https://motor.readthedocs.io/)
- [`uv` — Python package & version manager](https://docs.astral.sh/uv/)
