# Contribution #1: Add a JSON `/health` endpoint to the Tessera.io Python AI service

**Contribution Number:** 1
**Student:** Brian Bazurto
**Project:** [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io) — an open-source collaborative developer sandbox with real-time CRDT sync and secure remote code execution for human-AI pair programming
**Issue:** [#39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**Status:** Phase I — In Progress

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

No `/health` route exists. There is no built-in way to verify the service's status
short of exercising its real endpoints.

### Affected Components

- `apps/ai-service/src/main.py` — the FastAPI application where the route is added.
- `apps/ai-service/src/config.py` — likely source of the DB/model settings the
  health check needs to inspect.

---

## Reproduction Process

_(Phase II — Week 2: stand up the local environment, confirm there's no `/health`
route today, and document the setup.)_

### Environment Setup

_To be completed in Phase II._

### Steps to Reproduce

1. Run the `apps/ai-service` FastAPI app locally.
2. Request `GET /health`.
3. Observe that the route does not exist (404).

### Reproduction Evidence

_To be added in Phase II (commit link / logs / findings)._

---

## Solution Approach

_(Phase II — Week 2.)_

### Analysis

_Root-cause / design notes to be completed in Phase II._

### Proposed Solution

Add a `/health` route to the FastAPI app that returns JSON with database
connectivity stats and model statuses, returning an appropriate status code when a
dependency is unhealthy.

### Implementation Plan (UMPIRE)

**Understand:** Add a JSON `/health` endpoint reporting DB connectivity and model status.
**Match:** Follow the existing FastAPI route/structure conventions in `apps/ai-service`.
**Plan:** _Detailed step-by-step plan in Phase II._
**Implement:** _Branch/commit links during Phase III._
**Review:** _Self-review against `CONTRIBUTING.md` before opening the PR._
**Evaluate:** _Verify with a request to `/health` and an automated test._

---

## Testing Strategy

_(Phase III.)_

### Unit Tests

- [ ] `/health` returns 200 and the expected JSON shape when dependencies are healthy.
- [ ] `/health` reports an unhealthy DB/model state correctly.

### Integration Tests

- [ ] Endpoint reachable on the running service.

### Manual Testing

_To be completed in Phase III._

---

## Implementation Notes

### Week 1 Progress

Phase I — Issue Selection. Selected issue #39 on the Tessera.io Python AI service,
created this Contribution README, and worked through the issue-selection checklist.
Detailed tracker in [week-1/PHASE-1-CHECKLIST.md](week-1/PHASE-1-CHECKLIST.md).

### Code Changes

_None yet — code work begins in Phase III._

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
- [FastAPI documentation](https://fastapi.tiangolo.com/)
