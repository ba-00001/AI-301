# Week 2 — Phase II: Reproduce & Plan

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Target duration:** 3–7 days · **Project 2 due:** Mon, June 15, 2026, 2:59 AM EDT

Phase II is about proving I understand issue #39 by reproducing it locally and
writing a concrete plan. No pull request and no finished code this phase — just a
reproduction and a plan that makes sense.

**My issue:** [Kushaal-k/Tessera.io #39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**Working branch:** `feature/issue-39-health-endpoint` (in my fork, `ba-00001/Tessera.io`)

## Headline finding

The `/health` route already exists, but it's a scaffold **placeholder**:
`health_check()` in `apps/ai-service/src/main.py` returns a hard-coded
`{"status": "ok"}` and inspects nothing. It reports the service healthy **even with
MongoDB down** — which I confirmed. Issue #39 asks for real DB connectivity
statistics and model statuses, so the work is to make that route actually inspect
its dependencies, not to add a new route. Documented in full in the
[Contribution README](../README.md).

## Task tracker

- [x] **1. Read the Phase II project + grading guides** — done.
- [x] **2. Set up the local dev environment** — `apps/ai-service` running under
  `uvicorn` (Python 3.12 via `uv` after a 3.9 version-mismatch).
- [x] **3. Reproduce the issue at least twice** — `GET /health` returns
  `{"status":"ok"}` consistently, even with MongoDB not running.
- [x] **4. Document numbered reproduction steps** — in the README under
  *Reproduction Process → Steps to Reproduce*.
- [x] **5. Write the UMPIRE solution plan** — in the README under *Solution
  Approach → Implementation Plan (UMPIRE)*; names the root cause and the specific
  files (`main.py`, `db.py`, `config.py`).
- [x] **6. Update the Contribution README** — Environment Setup, Steps to
  Reproduce, branch link, and Implementation Plan all filled in.
- [x] **7. Star + fork the repo** on GitHub — done ([ba-00001/Tessera.io](https://github.com/ba-00001/Tessera.io)).
- [x] **8. `/claim` issue #39** — comment posted; the auto-claim bot assigned #39 to me.
- [x] **9. Create and push the working branch** — pushed; link resolves:
  [feature/issue-39-health-endpoint](https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint).
- [ ] **10. Submit the check-in form** marking **"Phase II Complete"** in the course
  portal.
- [ ] **11. (Recommended) Announce in `#dts-su26-ai301-celebration`** (text drafted
  below).

## Environment setup notes

- Followed the project `README.md` (Typical Case: README setup instructions).
- **Python version mismatch.** System Python is 3.9.6; project needs ≥ 3.11.
  `pip install -r requirements.txt` failed on `mcp[cli]` (needs ≥ 3.10). Fixed with
  `uv`: `uv venv --python 3.12 .venv && uv pip install -r requirements.txt`.
- Reproduced **without** MongoDB on purpose — Motor connects lazily, so the app
  starts fine, and a down database is the clearest way to expose the stub.

## Commands to create + push the branch (Step 9)

The project's `CONTRIBUTING.md` requires **signed-off** commits (`-s`), or CI
rejects the PR. After forking on GitHub:

```bash
git clone git@github.com:ba-00001/Tessera.io.git
cd Tessera.io
git checkout main && git pull origin main
git checkout -b feature/issue-39-health-endpoint
git push origin feature/issue-39-health-endpoint
```

Branch URL once pushed:
`https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint`

## Reproduction (exact commands used)

```bash
cd apps/ai-service
uv venv --python 3.12 .venv && source .venv/bin/activate
uv pip install -r requirements.txt
uvicorn src.main:app --host 127.0.0.1 --port 8000   # MongoDB intentionally NOT running

nc -z -w2 127.0.0.1 27017        # -> connection refused (no DB)
curl -s -i http://127.0.0.1:8000/health
#   HTTP/1.1 200 OK
#   {"status":"ok"}              <- reports healthy with the DB down: the defect
curl -s http://127.0.0.1:8000/openapi.json   # paths: /rag/ingest, /rag/search, /health
```

## Ready to post

**Comment to leave on issue #39** (Step 8, after `/claim`):

> /claim
>
> Following up on my earlier note — I've reproduced this locally. The `/health`
> route currently returns a static `{"status": "ok"}` and doesn't inspect MongoDB
> or the model/MCP layer (it still reports `ok` with the database down). My plan is
> to expand `health_check()` in `apps/ai-service/src/main.py` to ping MongoDB and
> report a `database` block (connectivity + latency) and a `models` block
> (`MCP_SERVER_NAME`, `EMBEDDING_DIMENSIONS`), returning `503` when the database is
> unreachable. Does returning `503` on an unhealthy dependency match how you'd like
> health reported, or would you prefer always-`200` with a status field?

**Slack announcement for `#dts-su26-ai301-celebration`** (Step 11):

> ✅ Phase II Complete — Tessera.io #39. Reproduced locally: the `/health` route is
> a scaffold stub returning `{"status":"ok"}` that reports healthy even with
> MongoDB down. Plan: expand `health_check()` to ping the DB + report model/MCP
> status, with a `503` when the DB is unreachable. Issue:
> https://github.com/Kushaal-k/Tessera.io/issues/39

## Notes / log

- `/health` stub was added in scaffold commit `1edd56f` (2026-05-27), before the
  issue was filed — confirmed with `git log -S "def health_check"`.
- Issue #39 is OPEN, unassigned, no `/claim`, no open PR addressing it (the one
  "health"-matching PR, #52, is an unrelated TS API client). Safe to claim.
- `apps/ai-service` has no Python test suite yet; I'll add a `tests/` folder in
  Phase III (pytest + FastAPI `TestClient`).
