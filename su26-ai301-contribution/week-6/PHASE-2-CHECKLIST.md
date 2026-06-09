# Week 6 — Cycle 2 / Phase II: Reproduce & Plan

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)

Phase II for Contribution #2: stand up the project locally, reproduce issue #46, and
write the plan. Full detail lives in the [Contribution #2 README](../contribution-2/README.md);
this is the tracker.

**Issue:** [Kushaal-k/Tessera.io #46](https://github.com/Kushaal-k/Tessera.io/issues/46)
**Branch:** `feature/issue-46-demux-docker-logs`

## Headline finding

`apps/execution-engine/src/sandbox.ts` runs containers **without a TTY**, so
`container.logs({ follow: false })` returns Docker's **multiplexed** stream — each
chunk is prefixed with an 8-byte header (`[stream_type, 0,0,0, size_be_u32]`). The code
read that buffer straight as UTF-8, so the header bytes rendered as the boxed
replacement symbols in the output panel, and `stderr` was hard-coded to `""`.

## Task tracker

- [x] **1. Set up the local dev environment** — cloned the fork, added `upstream`,
  `git fetch upstream` (fork even with upstream), `npm install` (Turborepo workspaces).
- [x] **2. Reproduce the issue** — captured it at the **byte level**: a synthetic
  `stdout` frame for `"Hello World\n"` decoded as raw UTF-8 is *not* the clean string
  and its first char code is `1` (the stream-type byte) — exactly the leaked header the
  issue describes. (Full browser repro needs a live Docker daemon; the byte-level repro
  is deterministic and CI-friendly.)
- [x] **3. Document the reproduction** — in the README under *Reproduction Process*.
- [x] **4. Write the UMPIRE plan** — add a pure `demuxDockerStream(buffer)` helper,
  wire it into `executeInSandbox()`, cover with vitest from synthetic frames.
- [x] **5. Update the Contribution #2 README** — Understanding, Reproduction, Solution
  Approach all filled in.
- [x] **6. Confirm the working branch is pushed.**
- [ ] **7. Submit the check-in form** for the reported phase.

## Why no live Docker was needed

The bug and its fix are pure **stream parsing**. Like Contribution #1 (which tested the
Mongo probe with a fake client), I made the fix a pure function over a `Buffer` and
reproduced the defect with synthetic Docker frames — deterministic, fast, and runnable
in CI with no daemon. The Docker multiplexed-stream header format is documented in the
Docker Engine API.

## Notes / log

- `ExecutionResult` in `packages/shared-types` already declares `stdout` and
  `stderr: string`, so separating the streams needs no type change.
- The repo's existing test (`apps/web/src/services/aiService.test.ts`) confirmed the
  test runner is **vitest** with co-located `*.test.ts` files — the pattern I'll follow.
