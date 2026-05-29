# Week 1 — Phase I: Issue Selection

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Target duration:** 1–3 days · **Project 1 due:** Mon, June 8, 2:59 AM EDT

Phase I is about getting set up, creating my Contribution README, and choosing an
issue well. No code or local environment setup yet — that starts in Phase II.

**My selected issue:** [Kushaal-k/Tessera.io #39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)

## Task tracker

- [ ] **1. Log into GitHub and create my CodePath Student Slack account.**
- [ ] **1b. Join the Slack `#dts-su26-ai301-contribute` channel.**
- [ ] **1c. (Optional) Run through the First Contributions tutorial.**
- [x] **2. Create my Contribution README** — done, at the repo root ([../README.md](../README.md)).
- [x] **3. Browse the issue lists and shortlist candidates** — shortlisted 3 (see table below).
- [x] **4. Read my top candidates carefully** — read the issue bodies, labels, and confirmed no linked PRs.
- [x] **5. Run the Issue Selection Checklist** — scored below; #39 scores highest.
- [x] **6. Pick one and commit to it** — chose #39.
- [ ] **7. Fork the Tessera.io repo on GitHub.**
- [ ] **8. Comment on issue #39 (text drafted below) and note it on the course Google Sheet.**
- [ ] **9. Set README Status to "Phase I Complete" and submit my repo link in the course portal.**
- [ ] **10. Announce in Slack `#dts-su26-ai301-celebration` (text drafted below).**

## Issue Selection Checklist (≥4 of 6 to claim)

1. **I understand the problem** — can explain it in one sentence.
2. **Scope fits 3–4 weeks** — specific and bounded.
3. **Matches my skills** (or learnable quickly with AI).
4. **Active and claimable** — recent maintainer activity, unassigned, no claim, no open PR, no blocking labels.
5. **Helpful context** — maintainer notes, links to the relevant code.
6. **Clear setup docs** — README.md + CONTRIBUTING.md I can follow.

Decision rule: 6/6 → claim · 4–5 → ask for a gut check · <4 → skip.

## My candidates

All three are in [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io)
(open-source collaborative dev sandbox for human-AI pair programming; TypeScript
monorepo with a Python `apps/ai-service`). All three are unassigned, have 0
comments (no one has claimed them), no linked PR, and carry `good first issue` +
`help wanted`. The repo has `README.md`, `CONTRIBUTING.md`, and `.env.example`.

| Issue | Summary | File | Score | Notes |
|-------|---------|------|-------|-------|
| [#39](https://github.com/Kushaal-k/Tessera.io/issues/39) **(chosen)** | Add a JSON `/health` endpoint reporting DB connectivity + model status | `apps/ai-service/src/main.py` | 6/6 | Cleanest "done"; single file; FastAPI routing practice |
| [#38](https://github.com/Kushaal-k/Tessera.io/issues/38) | Enforce env checks in Python AI config via Pydantic validators (fail fast on bad `MONGODB_URI`) | `apps/ai-service/src/config.py` | 5/6 | Slightly more involved; good Pydantic practice |
| [#40](https://github.com/Kushaal-k/Tessera.io/issues/40) | Add FastAPI timing-log middleware for AI/RAG search tasks | `apps/ai-service/src/main.py` | 5/6 | Touches middleware; useful metrics work |

**Why #39:** scores 6/6 — I can explain the problem in one sentence, the scope is
small and bounded, it's Python/FastAPI (learnable for an AI capstone), it's active
and unclaimed, the issue names the exact file, and the project has setup docs.

## Ready to post

**Comment to leave on issue #39** (Step 8):

> Hi! I'm a student in the CodePath AI301 program and I'd like to work on this
> issue. My plan is to add a `GET /health` route in `apps/ai-service/src/main.py`
> that returns JSON with database connectivity and model status. I'll start by
> setting up the service locally this week. Any pointers on which DB/model client
> objects you'd like the health check to inspect would be appreciated!

**Slack announcement for `#dts-su26-ai301-celebration`** (Step 10):

> 🎯 Phase I Complete — Selected Tessera.io #39: add a JSON `/health` endpoint to
> the Python AI service. Link: https://github.com/Kushaal-k/Tessera.io/issues/39

## Notes / log

- Verified all three candidates are open, unassigned, 0 comments, no linked PR.
- Repo created 2026-05-27, pushed to 2026-05-29 — maintainer is actively shipping.
- I can swap to #38 or #40 with no penalty if I change my mind before building.
