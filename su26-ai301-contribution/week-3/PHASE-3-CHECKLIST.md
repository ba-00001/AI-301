# Week 3 — Phase III: Build

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Target duration:** ~2 weeks · **Phase III check-in:** submit when the solution is
ready to become a PR.

Phase III is where I write the code: implement the Phase II plan, add tests, and push
working commits. No PR yet — that's Phase IV.

**My issue:** [Kushaal-k/Tessera.io #39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**Working branch:** `feature/issue-39-health-endpoint` (in my fork, `ba-00001/Tessera.io`)

## What I built

Turned the placeholder `/health` route into a real health check that reports
MongoDB connectivity and model/MCP status.

| File | Change | Commit |
|------|--------|--------|
| `apps/ai-service/src/db.py` | `check_connection()` pings MongoDB + reports stats; `connect_db()` sets a bounded `serverSelectionTimeoutMS` | `6c2f3ce` |
| `apps/ai-service/src/config.py` | `MONGODB_TIMEOUT_MS` setting (default 3000) | `6c2f3ce` |
| `apps/ai-service/src/main.py` | `health_check()` returns `database` + `models` blocks; `200`/`503` by DB state | `cb1d8de` |
| `apps/ai-service/tests/test_health.py` | new pytest suite, 5 tests | `fda55d2` |

Diff: 4 files, +161/−4, scoped to the issue. All commits signed off (`-s`).

## Task tracker

- [x] **1. Review `CONTRIBUTING.md`** — confirmed: `/claim`, fork+star,
  Conventional Commits, **sign-off required** (`git commit -s`, CI rejects
  unsigned), Python lint via `ruff`.
- [x] **2. Implement the plan in small commits** — three signed-off commits above.
- [x] **3. Write tests** — `tests/test_health.py`, 5 tests, all passing.
- [x] **4. Run the suite + linters** — `pytest` 5/5 pass; `ruff check` and
  `ruff format --check` clean.
- [x] **5. Manual verification** — live `curl` of `/health` with MongoDB down →
  `503 degraded` in ~3s (see log below).
- [x] **6. Update the Contribution README** — Implementation Progress, Challenges,
  Testing notes, branch link, Code Changes.
- [x] **7. Save reproducible artifacts** — bundle + patches + combined diff in
  [fork-artifacts/](fork-artifacts/).
- [x] **8. Push the branch to my fork** — pushed from the bundle, so the commit
  SHAs match the README:
  [feature/issue-39-health-endpoint](https://github.com/ba-00001/Tessera.io/tree/feature/issue-39-health-endpoint).
- [x] **9. Open a draft PR** referencing #39 — [PR #66](https://github.com/Kushaal-k/Tessera.io/pull/66) (draft).
- [ ] **10. Submit the check-in form** marking **"Phase III Complete."**
- [ ] **11. Post a scrum update** in `#dts-su26-ai301-build-support` (twice-weekly
  Mon/Fri) and announce in `#dts-su26-ai301-celebration` (text below).

## Push these exact commits to my fork (Step 8)

After forking `Kushaal-k/Tessera.io` on GitHub, from a clone of my fork:

```bash
# from the root of a clone of ba-00001/Tessera.io
git fetch /path/to/su26-ai301-contribution/week-3/fork-artifacts/issue-39-health-endpoint.bundle \
  feature/issue-39-health-endpoint:feature/issue-39-health-endpoint
git push origin feature/issue-39-health-endpoint
```

Alternative (re-applies the patches; commit SHAs will differ):

```bash
git checkout -b feature/issue-39-health-endpoint
git am --signoff week-3/fork-artifacts/patches/*.patch
git push origin feature/issue-39-health-endpoint
```

## Manual verification log

```text
# MongoDB NOT running, patched service on :8022
$ curl -s -o out.json -w "HTTP %{http_code}  time=%{time_total}s\n" http://127.0.0.1:8022/health
HTTP 503  time=3.044054s
# out.json:
{ "status": "degraded",
  "database": { "connected": false, "database": "tessera",
                "collection": "code_chunks", "latency_ms": null,
                "error": "localhost:27017: [Errno 61] Connection refused ..." },
  "models": { "mcp_server": "tessera-ai", "embedding_dimensions": 1536,
              "embedding_provider": "placeholder" } }
```

Healthy path (`200` + `database.connected: true` + `latency_ms`) is asserted in
`test_health_returns_200_and_stats_when_db_up`; a full live run uses the project
README's `docker run -d -p 27017:27017 mongo:7`.

## Test run

```text
$ python -m pytest tests/ -v
tests/test_health.py::test_check_connection_no_client PASSED
tests/test_health.py::test_check_connection_success PASSED
tests/test_health.py::test_check_connection_failure PASSED
tests/test_health.py::test_health_returns_200_and_stats_when_db_up PASSED
tests/test_health.py::test_health_returns_503_when_db_down PASSED
5 passed
$ python -m ruff check src/ tests/        # All checks passed!
$ python -m ruff format --check src/ tests/   # 7 files already formatted
```

## Ready to post

**Scrum update for `#dts-su26-ai301-build-support`:**

> **Did:** Implemented #39 — `/health` now pings MongoDB and reports a `database`
> block (connectivity + latency) and a `models` block, returning `503` when the DB
> is down. Added a 5-test pytest suite (all green) and fixed a 30s hang by setting
> `serverSelectionTimeoutMS`.
> **Next:** Open a draft PR and self-review against `CONTRIBUTING.md`.
> **Blocked:** No.

**Slack announcement for `#dts-su26-ai301-celebration`:**

> ✅ Phase III Complete — Tessera.io #39. `/health` now reports real MongoDB
> connectivity + model/MCP status with proper `200`/`503` codes, backed by tests.
> https://github.com/Kushaal-k/Tessera.io/issues/39

## Notes / log

- Key debugging win: an `asyncio.wait_for` around the Motor ping did **not** bound
  the call (Motor bridges to a thread that ignores asyncio cancellation). The real
  fix was the client's `serverSelectionTimeoutMS`. Documented in the README.
- The `apps/ai-service` had no existing Python test suite, so I added `tests/` and
  modeled style on the repo's existing code rather than a neighboring test.
- Design point to confirm with the maintainer in the PR: returning `503` on an
  unhealthy DB vs. always `200` with a status field.
