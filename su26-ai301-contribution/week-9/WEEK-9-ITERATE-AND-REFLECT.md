# Week 9 — Iterate & Final Reflection

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Project 9 due:** Thu, July 30, 2026, 2:59 AM EDT

📽️ **Lightning Talks slides:** [AI301 AI Open Source Capstone — Lightning Talks deck](https://docs.google.com/presentation/d/1__D9MDUaeAnVnixWGanx0xrwC9Nu3YLrlpN4m_GqT9A/edit?usp=sharing)

Module 2 weeks are self-paced: you submit whatever phase you're on for whatever cycle,
graded against the four phase rubrics. By Week 9 I have **two** review-ready PRs open,
so this week is about keeping both feedback loops alive and writing a portfolio-grade
reflection across the whole program — not opening anything new.

## Where both contributions stand

| Cycle | Issue | PR | Base | State | CI |
|-------|-------|----|------|-------|----|
| 1 | [#39](https://github.com/Kushaal-k/Tessera.io/issues/39) — JSON `/health` endpoint (Python AI service) | [#66](https://github.com/Kushaal-k/Tessera.io/pull/66) | upstream `main` | Open, ready, **awaiting review** | green |
| 2 | [#46](https://github.com/Kushaal-k/Tessera.io/issues/46) — demux Docker logs (execution engine) | [#78](https://github.com/Kushaal-k/Tessera.io/pull/78) | upstream `main` | Open, ready, **awaiting review** | green |

## Maintainer feedback status (as of June 9, 2026)

**No human-maintainer review has been received on either PR yet.** Both were opened
ready-for-review against upstream `main`, use the project's PR template, link their
issues with `Fixes #`, and have a comment @-mentioning the maintainer (@Kushaal-k). All
CI checks pass on both. Per the course guidance, if there's no review after **5–7
business days** I'll leave one polite follow-up comment ("Hi! Is there anything I can
adjust to help move this forward?") rather than re-pinging sooner. Each PR also carries
one open **design question** for the maintainer:

- **#66:** return `503` on an unhealthy DB vs. always-`200` with a status field.
- **#78:** populate `stderr` separately vs. append it to `stdout` for display.

I'll log any response here and in each Contribution README with the date and the commit
hash that addresses it.

## What I did this week (while waiting)

- **Self-review pass** on both branches: re-read each diff against the project's
  `CONTRIBUTING.md`, confirmed every commit is signed off, and re-ran the checks
  (`pytest` 5/5 for #66; `npm run test`/`typecheck`/`build` green repo-wide for #78).
- Confirmed neither branch has drifted from upstream `main` (both still merge cleanly).
- Prepared concise follow-up comment drafts for the 5–7 business-day mark.

## Task tracker

- [x] **1. Confirm both PRs are healthy** — open against upstream `main`, ready, CI green.
- [x] **2. Document feedback status honestly** — "awaiting review," dated, with the two
  open design questions surfaced to the maintainer.
- [x] **3. Re-verify branches** — re-ran tests/checks; branches merge cleanly.
- [x] **4. Write the program-level reflection** (below).
- [ ] **5. Submit the check-in form** for the reported phase.
- [ ] **6. Follow up politely on each PR** after 5–7 business days with no review.
- [ ] **7. Respond to any maintainer feedback within ~24h**; log date + commit hash.

## Final Reflection (across both cycles)

**Two PRs, two packages, one project.** I deliberately kept both contributions in
Tessera.io but moved across its stack — Cycle 1 in the Python `apps/ai-service`
(FastAPI + Motor/MongoDB), Cycle 2 in the TypeScript `apps/execution-engine` (Docker
stream parsing). Working a second time in a repo I already understood let me move much
faster on setup and conventions in Cycle 2.

**The repeated pattern that worked: test the logic, fake the I/O.** Both fixes touch an
external dependency that's painful to run in CI (MongoDB; the Docker daemon). In both, I
isolated the real logic into a pure, testable unit — a connectivity probe returning a
plain dict, and a `Buffer`-in/`{stdout,stderr}`-out demuxer — and drove it with a fake
client / synthetic frames. That made the suites deterministic and daemon-free, and it's
the single technique I'll reach for first next time.

**Reading `.github/` early pays off.** The biggest avoidable cost in Cycle 1 was
writing a PR description in my own structure and rewriting it once I found the project's
template. In Cycle 2 I read the PR template, CI workflows (Gatekeeper sign-off,
`verify-sandbox`), and contributing rules *before* the first commit, and submission was
much smoother.

**Submission ≠ acceptance, and that's the point.** Neither PR is merged yet, and that's
a normal open-source outcome. The deliverable was a professional, review-ready
contribution and an honest, dated record of the loop — both of which I have. If a
maintainer requests changes, the README feedback logs are ready to capture each round.

**What I'd do differently going forward:** run typechecks through the monorepo's task
runner (turbo) from the root rather than per-package to avoid build-ordering red
herrings; and add a quick live smoke test (a `curl -w "%{time_total}"`, a real
container run) to my manual-test habit, since unit tests that mock the dependency can
hide latency/integration issues.

## Notes / log

- Cycle 2 reaching a submitted PR before program close is exactly the "strong signal"
  the grading guide calls out; I'll keep both loops active through Week 10.
- Honest status: I have not invented maintainer feedback. Both PRs are awaiting first
  review as of this writing.
