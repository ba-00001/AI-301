# Week 10 — Program Closeout

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Project 10 due:** Thu, August 6, 2026, 2:59 AM EDT · **Final week.**

Week 10 is the program's final self-paced check-in — graded against the phase I report,
not the calendar. I finished both contribution cycles ahead of schedule, so this week
is the closeout: confirm everything is healthy, document the program honestly, and set
the plan for keeping both feedback loops alive after the course ends.

## Program completion summary

The core milestone is **one review-ready PR to a real open-source project**. I shipped
**two**, both to [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io),
across two different packages.

| Cycle | Issue | PR | Area | Language | Tests | CI | Status |
|-------|-------|----|------|----------|-------|----|--------|
| 1 | [#39](https://github.com/Kushaal-k/Tessera.io/issues/39) — JSON `/health` endpoint | [#66](https://github.com/Kushaal-k/Tessera.io/pull/66) | `apps/ai-service` | Python / FastAPI | 5 (pytest) | green | Open, ready, **awaiting review** |
| 2 | [#46](https://github.com/Kushaal-k/Tessera.io/issues/46) — demux Docker logs | [#78](https://github.com/Kushaal-k/Tessera.io/pull/78) | `apps/execution-engine` | TypeScript | 7 (vitest) | green | Open, ready, **awaiting review** |

Both PRs: opened against upstream `main` (not fork-internal), use the project's PR
template, link their issue with `Fixes #`, have the maintainer @-mentioned, pass all CI
checks, and are `MERGEABLE` (no conflicts). Per the project guide, *"a PR that's
actively being reviewed at program close still counts as a completion"* — and these are
review-ready and surfaced, awaiting first maintainer response.

## Honest status at program close (as of June 9, 2026)

**No human-maintainer review has been received on either PR.** I have not invented any
feedback. Both were opened today (working ahead of the calendar), so the course's
5–7 business-day window before a polite follow-up has not elapsed — leaving a nudge now
would be premature, so I haven't. Each PR carries one open design question already
surfaced to the maintainer:

- **#66:** `503` on an unhealthy DB vs. always-`200` with a status field.
- **#78:** populate `stderr` separately vs. append it to `stdout` for display.

## Deliverables map (the full paper trail)

- **Contribution #1:** [su26-ai301-contribution/README.md](../README.md) — Cycle 1 living doc.
- **Contribution #2:** [contribution-2/README.md](../contribution-2/README.md) — Cycle 2 living doc.
- **Per-week / per-phase trackers:**
  - Cycle 1: [week-1](../week-1/PHASE-1-CHECKLIST.md) · [week-2](../week-2/PHASE-2-CHECKLIST.md) · [week-3](../week-3/PHASE-3-CHECKLIST.md) · [week-4](../week-4/PHASE-4-CHECKLIST.md)
  - Cycle 2: [week-5](../week-5/PHASE-1-CHECKLIST.md) · [week-6](../week-6/PHASE-2-CHECKLIST.md) · [week-7](../week-7/PHASE-3-CHECKLIST.md) · [week-8](../week-8/PHASE-4-CHECKLIST.md)
  - [week-9](../week-9/WEEK-9-ITERATE-AND-REFLECT.md) — iterate + reflection · **week-10** — this closeout
- **Reproducible artifacts:** [Cycle 1](../week-3/fork-artifacts/) · [Cycle 2](../contribution-2/fork-artifacts/) (git bundle, per-commit patches, combined diff).

## Task tracker

- [x] **1. Confirm both PRs healthy** — open vs upstream `main`, ready, CI green, mergeable.
- [x] **2. Document program completion honestly** — two review-ready PRs; both awaiting review.
- [x] **3. Final capstone reflection** (below).
- [x] **4. Verify the Contribution READMEs are internally consistent** — Phase II plans
  match what was built and submitted in both cycles.
- [ ] **5. Submit the final check-in form** (Project 10) in the course portal.
- [ ] **6. Post the program-completion note** in `#dts-su26-ai301-celebration` (below).
- [ ] **7. Post-program: monitor both PRs**, respond to feedback within ~24h, log each
  round (date + commit hash), and leave one polite follow-up after 5–7 business days of
  no review.

## Final Capstone Reflection

**What I shipped.** Two review-ready pull requests to the same real project, in two
languages and two subsystems: a Python/FastAPI `/health` endpoint that actually probes
MongoDB and reports model/MCP status (#66), and a TypeScript fix that demultiplexes
Docker's framed log stream so the execution output panel stops rendering header bytes as
garbage and finally separates `stderr` (#78). Each is backed by a real test suite (5
pytest, 7 vitest) and passes the project's CI gates.

**The technique that carried both cycles: test the logic, fake the I/O.** Both fixes
sit against an external dependency that's painful in CI (MongoDB; the Docker daemon). In
both I isolated the real logic into a pure, testable unit — a connectivity probe
returning a plain dict, and a `Buffer`-in/`{stdout,stderr}`-out demuxer — and drove it
with a fake client or synthetic frames. Deterministic, daemon-free, fast. It's the first
tool I'll reach for the next time a bug lives behind I/O.

**Reading the project before writing.** My biggest avoidable cost in Cycle 1 was writing
a PR description in my own structure, then rewriting it once I found the project's
template. Cycle 2 went faster because I read `.github/` first — the PR template, the
Gatekeeper sign-off requirement, the `verify-sandbox` workflow — *before* the first
commit. Conventions are discoverable; find them early.

**Communication is part of the work.** Both PRs lead with *why* before *what*, reference
the issue with a close keyword, include before/after evidence, and pose a specific design
question to the maintainer rather than presenting the change as final. The READMEs
document the loop — dated status, open questions — so a reviewer (or a future me) can see
the state at a glance.

**Submission ≠ acceptance, and that's the lesson.** Neither PR is merged yet; that's a
normal open-source outcome, not a failure. The program's milestone is a professional,
review-ready contribution and an honest record of the process — both of which exist,
twice. If changes are requested, the feedback logs are ready to capture each round.

**What I'd do differently.** Run typechecks through the monorepo task runner (turbo) from
the root from the start, to avoid the workspace build-ordering red herring I chased in
Cycle 2; and add a quick live smoke test (a `curl -w "%{time_total}"`, a real container
run) to my manual-test habit, since dependency-mocking unit tests can hide latency and
integration issues — exactly the 30s `/health` hang I only caught live in Cycle 1.

## Ready to post

**Program-completion note for `#dts-su26-ai301-celebration`:**

> 🎓 AI301 complete — two review-ready PRs to Tessera.io: #66 (Python `/health` that
> really probes MongoDB) and #78 (demux Docker logs in the TS execution engine). Both
> open against upstream `main`, template-filled, tested, CI green, maintainer pinged.
> Now iterating on review. Thanks to the cohort and staff!
> https://github.com/Kushaal-k/Tessera.io/pull/66 · https://github.com/Kushaal-k/Tessera.io/pull/78

## Notes / log

- Both PRs `MERGEABLE`; `mergeStateStatus: BLOCKED` only reflects the pending required
  review (no conflicts, CI green) — expected for an unreviewed community PR.
- I'll keep both Contribution READMEs updated as the only source of truth as feedback
  arrives after program close.
