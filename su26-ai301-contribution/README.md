# Contribution #1: Add a JSON `/health` endpoint to the Tessera.io Python AI service

**Contribution Number:** 1
**Student:** Brian Bazurto
**Project:** [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io) — an open-source collaborative developer sandbox with real-time CRDT sync and secure remote code execution for human-AI pair programming
**Issue:** [#39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**Status:** Phase IV — Submit & Iterate (PR open & ready for review; awaiting maintainer review)

📽️ **Lightning Talks slides:** [AI301 AI Open Source Capstone — Lightning Talks deck](https://docs.google.com/presentation/d/1__D9MDUaeAnVnixWGanx0xrwC9Nu3YLrlpN4m_GqT9A/edit?usp=sharing)

> **Two contributions in this repo.** This README documents **Contribution #1**
> (Cycle 1, issue #39 → [PR #66](https://github.com/Kushaal-k/Tessera.io/pull/66)).
> After reaching Phase IV I started a second cycle —
> **[Contribution #2](contribution-2/README.md)** (issue #46, the Docker-log demuxer →
> [PR #78](https://github.com/Kushaal-k/Tessera.io/pull/78)). Per-week trackers:
> weeks [1](week-1/PHASE-1-CHECKLIST.md)–[4](week-4/PHASE-4-CHECKLIST.md) cover Cycle 1
> Phases I–IV; weeks [5](week-5/PHASE-1-CHECKLIST.md)–[8](week-8/PHASE-4-CHECKLIST.md)
> cover Cycle 2 Phases I–IV; [week 9](week-9/WEEK-9-ITERATE-AND-REFLECT.md) is iteration
> and the final reflection; [week 10](week-10/WEEK-10-PROGRAM-CLOSEOUT.md) is the program
> closeout. **Program milestone met twice** — two review-ready PRs (#66, #78), both open
> against upstream `main` and awaiting maintainer review.

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

I added a `pytest` suite at `apps/ai-service/tests/test_health.py` (the service
had no test suite before), using a small fake Motor client so the tests need no
live MongoDB. All 5 tests pass and `ruff check` / `ruff format --check` are clean.

### Unit Tests

- [x] `db.check_connection()` returns `connected: false` (with an `error`) when the
      client isn't initialized — `test_check_connection_no_client`.
- [x] `db.check_connection()` returns `connected: true` with a float `latency_ms`
      when the ping succeeds — `test_check_connection_success`.
- [x] `db.check_connection()` reports the error and `connected: false` when the ping
      raises — `test_check_connection_failure`.
- [x] `GET /health` returns `200`, `status: "ok"`, and the `database`/`models`
      blocks when the DB is up — `test_health_returns_200_and_stats_when_db_up`.
- [x] `GET /health` returns `503` and `status: "degraded"` when the DB is down —
      `test_health_returns_503_when_db_down`.

### Integration / Manual Testing

- [x] **Baseline reproduced** (Phase II): MongoDB down → the old stub returns
      `200 {"status":"ok"}`.
- [x] **After the fix, DB down:** ran `uvicorn` with no MongoDB and `curl`'d
      `/health` → `HTTP 503`, `status: "degraded"`, `database.connected: false`
      with the connection-refused error and the `models` block. Response came back
      in ~3s (the new `serverSelectionTimeoutMS`), not Motor's 30s default.
- [x] **After the fix, DB up:** asserted in `test_health_returns_200_and_stats_when_db_up`
      (`200` + `database.connected: true` + `latency_ms`). The live container run
      uses the project README's `docker run … mongo:7` step.

### How to run

```bash
cd apps/ai-service && source .venv/bin/activate
python -m pytest tests/ -v
python -m ruff check src/ tests/ && python -m ruff format --check src/ tests/
```

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

### Week 3 Progress (June 8, 2026)

Phase III — Build. Implemented the Phase II plan on the branch
`feature/issue-39-health-endpoint`, in three small signed-off commits, and added a
test suite. All tests pass; lint/format clean.

**What I built:**

- `apps/ai-service/src/db.py` — added `check_connection()`, which pings MongoDB
  (`await _client.admin.command("ping")`), times it, and returns
  `{connected, latency_ms, database, collection, error?}` without raising. Also
  set `serverSelectionTimeoutMS` on the client in `connect_db()` so an unreachable
  DB fails in ~3s instead of Motor's 30s default (commit `6c2f3ce`).
- `apps/ai-service/src/config.py` — added `MONGODB_TIMEOUT_MS` (default `3000`) to
  drive that timeout (commit `6c2f3ce`).
- `apps/ai-service/src/main.py` — replaced the static `{"status":"ok"}` stub in
  `health_check()` with a JSON payload reporting a `database` block (connectivity +
  latency) and a `models` block (`MCP_SERVER_NAME`, `EMBEDDING_DIMENSIONS`,
  `embedding_provider`), returning `200` when healthy and `503` when the DB is down
  (commit `cb1d8de`).
- `apps/ai-service/tests/test_health.py` — new `pytest` suite, 5 tests, covering
  the probe and both endpoint paths with a fake Motor client (commit `fda55d2`).

**Challenges faced:**

- My first attempt bounded the ping with `asyncio.wait_for(..., timeout=2.0)`, but a
  live run with no MongoDB still took **30s** to respond. Motor bridges the ping to
  a worker thread that doesn't honor asyncio cancellation, so `wait_for` never
  fired. I traced this from the response body — the error text was pymongo's own
  `Timeout: 30s` server-selection message, not a `wait_for` timeout. **Fix:**
  configure the client with `serverSelectionTimeoutMS` so pymongo itself fails fast;
  a re-run returned `503` in ~3s. (Verified with a `curl -w "%{time_total}"`.)
- The service had no Python test suite, so there was no neighboring test to model.
  I followed the repo's existing style instead (`pydantic` `BaseModel` shapes in
  `rag.py`, type hints, `ruff` formatting) and used FastAPI's `TestClient` with a
  fake client to avoid a real DB dependency.

**Commits this week (branch `feature/issue-39-health-endpoint`):**

- `6c2f3ce` — feat(ai-service): add MongoDB connectivity probe for health checks
- `cb1d8de` — feat(ai-service): report DB and model status from /health
- `fda55d2` — test(ai-service): cover /health endpoint and connectivity probe

### Week 4 Progress (June 9, 2026)

Phase IV — Submit & Iterate. Took the working Phase III branch from draft to a
review-ready pull request. No new code this phase: the work was submission quality
and surfacing the PR to the maintainer.

**What I did:**

- Ran the pre-submission checklist: confirmed the fix still reproduces correctly,
  `pytest` is 5/5 green, and the diff is scoped to the issue (`git diff` against
  upstream `main` shows only the four intended files, no debug code or stray
  changes). The PR's **CI Pipeline (Lint, Typecheck, Test, Build)** check is passing.
- **Rewrote the PR description to use the project's own
  [`.github/pull_request_template.md`](https://github.com/Kushaal-k/Tessera.io/blob/main/.github/pull_request_template.md)**
  rather than my earlier ad-hoc structure: a "why-first" Description, the
  `Type of Change` selection, `How Has This Been Tested` (automated + manual, with
  before/after `/health` output), and the project Checklist. Used `Fixes #39` (the
  project's close keyword) so the issue auto-closes on merge.
- **Marked the PR ready for review** (it had been a draft) and left a comment
  @-mentioning the maintainer (@Kushaal-k) introducing myself as a first-time
  contributor, summarizing the change, and re-raising the `503`-vs-always-`200`
  design question.
- Updated this README's _Pull Request_ section with the final PR state and a dated
  Maintainer Feedback log.

**Why no code changed this phase:** Phase III already shipped a tested, lint-clean
implementation that matched the Phase II plan, so Phase IV was about meeting the
project's submission bar (template, ready-for-review, reviewer surfaced) and opening
the feedback loop — not rewriting the solution.

### Code Changes

- **Branch:** https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint
- **Diff:** 4 files, +161/−4, scoped to the issue (`config.py`, `db.py`, `main.py`,
  `tests/test_health.py`). Reviewable copy of the diff and the exact commits are in
  [week-3/fork-artifacts/](week-3/fork-artifacts/) (combined diff, per-commit
  patches, and a git bundle that preserves the commit SHAs above).
- All commits are signed off (`git commit -s`) per the project `CONTRIBUTING.md`.

---

## Pull Request

**PR Link:** https://github.com/Kushaal-k/Tessera.io/pull/66 (open against upstream
`Kushaal-k/Tessera.io:main`, from `ba-00001/Tessera.io:feature/issue-39-health-endpoint`).
**Issue:** [#39](https://github.com/Kushaal-k/Tessera.io/issues/39) — claimed and
assigned to me via the project's `/claim` bot; the PR body links it with `Fixes #39`
so it auto-closes on merge.

**PR Description:** Rewrote the PR using the project's own
[`pull_request_template.md`](https://github.com/Kushaal-k/Tessera.io/blob/main/.github/pull_request_template.md)
— a "why-first" Description (the placeholder route reported healthy with MongoDB
down), Type of Change, How Has This Been Tested (automated + manual, with
before/after `/health` output), and the project Checklist (signed-off commits,
read `CONTRIBUTING.md`, self-review). The change makes `/health` ping MongoDB and
report `database` + `models` blocks, returning `200`/`503` by DB state.

**Maintainer Feedback:**

- **June 8, 2026** — Opened the PR as a draft for early feedback (Phase III). The
  repo's `github-actions` bot posted the first-PR welcome and confirmed the
  Gatekeeper CI gate. CI Pipeline (Lint, Typecheck, Test, Build) passed.
- **June 9, 2026** — Phase IV: rewrote the description to the project's PR template,
  **marked the PR ready for review** (out of draft), and left a comment
  @-mentioning the maintainer (@Kushaal-k) to request review, restating the
  `503`-vs-always-`200` design question. _No human-maintainer review yet — awaiting
  first response._

**Status:** **Awaiting review** (ready-for-review, not draft; reviewer @-mentioned
June 9, 2026; CI green).

---

## Learnings & Reflections

### Technical Skills Gained

- FastAPI route + response handling (`JSONResponse` with an explicit status code),
  and async MongoDB health probing with Motor (`admin.command("ping")`).
- Why `asyncio.wait_for` can't bound a thread-bridged driver call, and that the
  right lever is the client's `serverSelectionTimeoutMS`.
- Testing an ASGI app with FastAPI's `TestClient` and a hand-rolled fake client so
  the suite needs no live database; matching a project's `ruff` style.
- The submission mechanics: opening a fork→upstream PR, filling a project's own
  `pull_request_template.md`, using the project's close keyword (`Fixes #`) so the
  issue auto-closes on merge, and converting a draft to ready-for-review once CI is
  green.

### Challenges Overcome

- Diagnosed the 30s `/health` hang from the response body (pymongo's own timeout
  message gave it away) and fixed it at the client-config level rather than papering
  over it with an async timeout that didn't actually fire.
- Resisted the urge to keep polishing before submitting. The Phase III code was
  tested and lint-clean, so the right Phase IV move was to ship it for review and
  open the feedback loop — not to gold-plate a PR no maintainer had seen yet.

### What I'd Do Differently Next Time

- Run a quick `curl -w "%{time_total}"` on a new endpoint earlier — the latency
  problem was invisible to the unit tests (which mock the client) and only showed up
  in a live run. I'll add a timing check to my manual-test habit from the start.
- Check for the project's PR template **before** writing any PR prose. I drafted a
  Phase III description in my own structure and rewrote it in Phase IV once I read
  the repo's `.github/pull_request_template.md`. Reading `.github/` first (template,
  CODEOWNERS, workflows) tells you both how to submit and who reviews — do it before
  the first commit, not at submission time.
- Lead the PR description with the _why_ before the _what_. My first draft opened
  with the change; reviewers need the problem and the investigation first, then the
  diff. I reordered it in Phase IV to open with "the route reported healthy with the
  DB down."

---

## Resources Used

- [Tessera.io repository](https://github.com/Kushaal-k/Tessera.io)
- [Tessera.io CONTRIBUTING.md](https://github.com/Kushaal-k/Tessera.io/blob/main/CONTRIBUTING.md)
- [Issue #39](https://github.com/Kushaal-k/Tessera.io/issues/39)
- [FastAPI documentation](https://fastapi.tiangolo.com/)
- [Motor (async MongoDB) — admin `ping`](https://motor.readthedocs.io/)
- [`uv` — Python package & version manager](https://docs.astral.sh/uv/)
