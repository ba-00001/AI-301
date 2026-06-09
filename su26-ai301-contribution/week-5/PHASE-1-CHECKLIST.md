# Week 5 — Cycle 2 / Phase I: Issue Selection

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Target duration:** 1–3 days · **Project 5 due:** Thu, July 2, 2026, 2:59 AM EDT

Module 2 has no new per-week requirements — from Week 5 on, you move through
additional contribution cycles at your own pace and submit whichever phase you're on.
Having reached Phase IV on Contribution #1 ([#39 / PR #66](../README.md)), I started
**Cycle 2** with a new issue in the same project, as the course recommends.

**My Cycle 2 issue:** [Kushaal-k/Tessera.io #46 — Parse multiplexed Docker container logs to fix unknown symbols](https://github.com/Kushaal-k/Tessera.io/issues/46)
**Contribution #2 README:** [contribution-2/README.md](../contribution-2/README.md)
**Working branch:** `feature/issue-46-demux-docker-logs` (fork `ba-00001/Tessera.io`)

## Task tracker

- [x] **1. Re-run the Issue Selection Checklist** on fresh candidates — my original
  Cycle-2 shortlist (#38, #40) was taken/closed, so I re-surveyed open, unassigned
  issues with no linked PR.
- [x] **2. Pick one and commit to it** — chose #46 (`type:bug`, `good first issue`,
  `help wanted`): a single-file fix in `apps/execution-engine/src/sandbox.ts` with
  written reproduction steps. Scores 6/6 (see below).
- [x] **3. Fork + star the repo** — already done in Cycle 1 ([ba-00001/Tessera.io](https://github.com/ba-00001/Tessera.io)).
- [x] **4. `/claim` issue #46** — intro comment + `/claim`; the bot replied
  "successfully claimed" and assigned #46 to me.
- [x] **5. Create the Contribution #2 README** — [contribution-2/README.md](../contribution-2/README.md)
  (links, why-I-chose, understanding, repro, plan).
- [x] **6. Create the working branch** — `feature/issue-46-demux-docker-logs`.
- [ ] **7. Submit the check-in form** marking the reported phase in the course portal.
- [ ] **8. Announce in `#dts-su26-ai301-celebration`** (text below).

## Issue Selection Checklist (#46 → 6/6)

1. **Understand the problem** — Docker multiplexed-stream headers leak into the output
   as garbage symbols; `stderr` is never separated. ✅
2. **Scope fits the time** — one file, one helper + tests. ✅
3. **Matches my skills** — TypeScript/Node; Docker stream format is documented. ✅
4. **Active & claimable** — open, unassigned, no linked PR, `help wanted`. ✅
5. **Helpful context** — issue includes location, repro steps, and a fix outline. ✅
6. **Clear setup docs** — repo `README.md` + `CONTRIBUTING.md`. ✅

## Candidates I considered (all open, unassigned, no linked PR)

| Issue | Area | Why / why not |
|-------|------|---------------|
| **#46 (chosen)** | `execution-engine/sandbox.ts` | Real bug, written repro, testable demux helper |
| #35 | `execution-engine/sandbox.ts` | Env-var RAM override; clean but smaller, less of a "bug" |
| #37 | `execution-engine/sandbox.ts` | Container-ID logging; thinner on testable behavior |
| #44 | `execution-engine` | Cleanup npm script; harder to unit-test |

## Ready to post

**Slack announcement for `#dts-su26-ai301-celebration`:**

> 🎯 Cycle 2 Phase I — Selected Tessera.io #46: parse Docker's multiplexed container
> logs so the output panel stops showing garbage header bytes (and separate stderr).
> Same project, new package (the TS execution engine).
> https://github.com/Kushaal-k/Tessera.io/issues/46

## Notes / log

- Module 2 is self-paced cycles graded against the four phase rubrics (Week 5 Grading
  Guide: "grade against the phase the student reported, not the calendar week").
- Kept Contribution #1 ([PR #66](https://github.com/Kushaal-k/Tessera.io/pull/66))
  open and awaiting review while starting Cycle 2.
