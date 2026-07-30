# Week 7 — Cycle 2 / Phase III: Build

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)

📽️ **Lightning Talks slides:** [AI301 AI Open Source Capstone — Lightning Talks deck](https://docs.google.com/presentation/d/1__D9MDUaeAnVnixWGanx0xrwC9Nu3YLrlpN4m_GqT9A/edit?usp=sharing)

Phase III for Contribution #2: implement the plan, add tests, push working commits.

**Issue:** [Kushaal-k/Tessera.io #46](https://github.com/Kushaal-k/Tessera.io/issues/46)
**Branch:** [feature/issue-46-demux-docker-logs](https://github.com/ba-00001/Tessera.io/tree/feature/issue-46-demux-docker-logs)

## What I built

| File | Change | Commit |
|------|--------|--------|
| `apps/execution-engine/src/sandbox.ts` | `demuxDockerStream()` parses the 8-byte frames, strips headers, splits stdout/stderr, falls back to stdout for raw buffers; `executeInSandbox()` returns clean separated streams | `e9681d4` |
| `apps/execution-engine/src/sandbox.test.ts` | new vitest suite, 7 tests (synthetic frames, no Docker) | `40fe40f` |
| `apps/execution-engine/package.json` | add `test` script (`vitest run`) — package had none | `40fe40f` |

Diff: 3 files, **+142/−3**, scoped to the issue. Both commits **signed off** (`-s`).

## Task tracker

- [x] **1. Review `CONTRIBUTING.md`** — `/claim`, fork, Conventional Commits,
  **sign-off required** (Gatekeeper CI), keep diffs scoped.
- [x] **2. Implement in small commits** — two signed-off commits above.
- [x] **3. Write tests** — `sandbox.test.ts`, 7 tests, all passing.
- [x] **4. Run the suite + checks** — repo-wide via turbo (log below).
- [x] **5. Update the Contribution #2 README** — Testing Strategy + Implementation Notes.
- [x] **6. Save reproducible artifacts** — bundle + patches + combined diff in
  [contribution-2/fork-artifacts/](../contribution-2/fork-artifacts/).
- [x] **7. Push the branch to my fork.**
- [ ] **8. Submit the check-in form** for the reported phase.

## Test / check run

```text
$ npm run test        # turbo
@tessera/execution-engine:test  Test Files 1 passed (1)   Tests 7 passed (7)
@tessera/web:test               Test Files 1 passed (1)   Tests 4 passed (4)
Tasks: 5 successful, 5 total

$ npm run typecheck   # 10 successful, 10 total
$ npm run build       # 7 successful, 7 total
```

## Challenges faced

- **Strict `noUncheckedIndexedAccess`:** `buffer[offset]` is `number | undefined` in
  this repo's TS config, so the first `typecheck` failed. Fixed by folding a
  `streamType === undefined` guard into the same branch that handles a malformed/raw
  frame.
- **Workspace build order:** `tsc` in the package failed with "cannot find module
  `@tessera/shared-types`" (also in untouched `worker.ts`) until I built the dependency
  first / ran `typecheck` through turbo from the root, which orders the graph.

## Notes / log

- Kept the demuxer a **pure function** over a `Buffer` so the suite needs no Docker —
  same "test the logic, fake the I/O" approach as Contribution #1's Mongo probe.
