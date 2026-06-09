## Description

Issue #39 reported that the Python AI service (`apps/ai-service`) had no real health endpoint. Investigation showed the `/health` route already existed but was a scaffold placeholder added in the initial monorepo commit (`1edd56f`), *before* the issue was filed: `health_check()` returned a hard-coded `{"status": "ok"}` and inspected nothing. The route reported the service healthy even when MongoDB was completely unreachable — I confirmed this locally by hitting `/health` with no database running and still getting `200 {"status":"ok"}`.

This PR turns that placeholder into a real health check that reports MongoDB connectivity statistics and model/MCP status, so operators and monitoring can tell at a glance whether the service and its dependencies are actually up.

What changed:
- `apps/ai-service/src/db.py`: add `check_connection()`, which pings MongoDB (`await _client.admin.command("ping")`), times it, and returns `{connected, latency_ms, database, collection, error?}` without raising. Also set a bounded `serverSelectionTimeoutMS` on the client so an unreachable DB fails fast instead of hanging on Motor's 30s default.
- `apps/ai-service/src/config.py`: add `MONGODB_TIMEOUT_MS` (default `3000`) to drive that timeout.
- `apps/ai-service/src/main.py`: `health_check()` now returns a `database` block (connectivity + latency) and a `models` block (`MCP_SERVER_NAME`, `EMBEDDING_DIMENSIONS`, `embedding_provider`), returning HTTP `200` when healthy and `503` when the database is unreachable.
- `apps/ai-service/tests/test_health.py`: new pytest suite (5 tests) covering the probe and both endpoint paths with a fake Motor client, so the suite needs no live MongoDB.

One open design question for you: I return `503` when the database is unreachable so monitoring can act on the status code as well as the body. If you'd prefer the endpoint always return `200` with a status field instead, I'm happy to switch.

Fixes #39

## Type of Change
- [x] New feature (non-breaking change which adds functionality)

## How Has This Been Tested?

### Automated Verification
- [x] Existing/new tests pass — `python -m pytest tests/` → **5 passed**
- [x] Python lint/format clean — `ruff check src/ tests/` and `ruff format --check src/ tests/` → clean (the `apps/ai-service` package lints with `ruff`/`pytest`; these Python-only changes don't touch the TS workspaces, and the repo's **CI Pipeline (Lint, Typecheck, Test, Build)** check is green on this PR)

### Manual Verification
- [x] With MongoDB **down**: `GET /health` → `503 degraded` in ~3s (was 30s before the timeout fix)
- [x] With MongoDB **up**: `GET /health` → `200`, `database.connected: true` with a `latency_ms` figure

**Before** (placeholder route, MongoDB down):
```
HTTP/1.1 200 OK
{"status":"ok"}
```

**After** (this PR, MongoDB down):
```
HTTP/1.1 503 Service Unavailable
{
  "status": "degraded",
  "database": { "connected": false, "database": "tessera", "collection": "code_chunks",
                "latency_ms": null, "error": "localhost:27017: [Errno 61] Connection refused ..." },
  "models": { "mcp_server": "tessera-ai", "embedding_dimensions": 1536, "embedding_provider": "placeholder" }
}
```

## Checklist
- [x] My commits are **signed off** using `git commit -s` (`Signed-off-by: BRIAN BAZURTO <...>`).
- [x] I have read the [CONTRIBUTING.md](CONTRIBUTING.md) guidelines.
- [x] My code follows the style guidelines of this project.
- [x] I have performed a self-review of my own code.
- [x] I have commented my code, particularly in hard-to-understand areas.
- [ ] I have made corresponding changes to the documentation. (No docs change needed — the endpoint is self-describing via its JSON payload; happy to add a note to the service README if you'd like one.)
- [x] My changes generate no new warnings or console errors.
