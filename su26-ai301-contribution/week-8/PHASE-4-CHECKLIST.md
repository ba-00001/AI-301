# Week 8 — Cycle 2 / Phase IV: Submit & Iterate

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)

📽️ **Lightning Talks slides:** [AI301 AI Open Source Capstone — Lightning Talks deck](https://docs.google.com/presentation/d/1__D9MDUaeAnVnixWGanx0xrwC9Nu3YLrlpN4m_GqT9A/edit?usp=sharing)

Phase IV for Contribution #2: open the PR, surface it to the maintainer, document the
loop. A review-ready PR is the completion milestone — merged or not.

**Issue:** [#46](https://github.com/Kushaal-k/Tessera.io/issues/46) · **PR:** [#78](https://github.com/Kushaal-k/Tessera.io/pull/78)

## What I did

| Action | Result |
|--------|--------|
| Pre-submission checks | Byte-level repro holds; `test`/`typecheck`/`build` green repo-wide; diff scoped to 3 files |
| PR description | Project `pull_request_template.md` (`Fixes #46`), Bug-fix type, How-tested, Checklist |
| Open PR | [#78](https://github.com/Kushaal-k/Tessera.io/pull/78) against upstream `main`, ready for review (not draft) |
| Reviewer surfaced | Comment @-mentioning @Kushaal-k + design question (separate `stderr` vs. appended) |
| CI | **CI Pipeline, triage, verify-sandbox, welcome — all pass** |

## Task tracker

- [x] **1. Final pre-submission checks** — repro holds; `npm run test` (exec-engine
  7/7, web 4/4), `typecheck` 10/10, `build` 7/7; `git diff` vs upstream `main` shows
  only the three intended files.
- [x] **2. Confirm PR targets upstream `main`** — base `Kushaal-k/Tessera.io:main`,
  head `ba-00001/Tessera.io:feature/issue-46-demux-docker-logs` (fork PR, not
  fork-internal, not from `main`).
- [x] **3. Fill the project PR template** — Description (why-first), Type of Change
  (Bug fix), How Has This Been Tested (with byte-level before/after), Checklist;
  `Fixes #46`.
- [x] **4. Open the PR ready for review** — [#78](https://github.com/Kushaal-k/Tessera.io/pull/78).
- [x] **5. Request a review** — comment @-mentioning @Kushaal-k + design question.
- [x] **6. Update the Contribution #2 README** — Pull Request section, dated Maintainer
  Feedback log, **Status: Awaiting review**.
- [x] **7. Confirm CI is green** — all four checks pass on PR #78.
- [ ] **8. Submit the check-in form** marking the reported phase ("Phase IV").
- [ ] **9. Announce in `#dts-su26-ai301-celebration`** (text below).
- [ ] **10. (Ongoing) Respond to maintainer feedback within ~24h**; log each round
  (date + commit hash) in the Contribution #2 README.

## Ready to post

**Slack announcement for `#dts-su26-ai301-celebration`:**

> 🚀 Cycle 2 Phase IV Complete — PR submitted! Tessera.io #78 fixes the garbage
> symbols in the execution output panel by demultiplexing Docker's framed log stream
> (and finally separating stderr), backed by a 7-test vitest suite. Open against
> upstream `main`, ready for review, all CI green.
> https://github.com/Kushaal-k/Tessera.io/pull/78

## Notes / log

- This is my second submitted PR (Contribution #1 = [#66](https://github.com/Kushaal-k/Tessera.io/pull/66)),
  which the grading guide notes as a strong signal — a Cycle 2 PR before program close.
- Raised the behavior-change question (separate `stderr` now populated) in the PR up
  front, since downstream display code may have assumed the old always-empty `stderr`.
