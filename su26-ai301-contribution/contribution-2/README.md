# Contribution #2: Demultiplex Docker logs in the Tessera.io execution engine

**Contribution Number:** 2 (Cycle 2)
**Student:** Brian Bazurto
**Project:** [Kushaal-k/Tessera.io](https://github.com/Kushaal-k/Tessera.io) — an open-source collaborative developer sandbox with real-time CRDT sync and secure remote code execution for human-AI pair programming
**Issue:** [#46 — Parse multiplexed Docker container logs to fix unknown symbols](https://github.com/Kushaal-k/Tessera.io/issues/46)
**Pull Request:** [#78](https://github.com/Kushaal-k/Tessera.io/pull/78)
**Status:** Phase IV — Submit & Iterate (PR open & ready for review; awaiting maintainer review)

> This is my **second** contribution to Tessera.io, started after Contribution #1
> ([#39 / PR #66](../README.md)) reached Phase IV. The course recommends a second
> cycle on the same project, so I stayed in Tessera.io but moved from the Python AI
> service to the TypeScript **execution engine** to broaden the surface I've worked
> in.

---

## Why I Chose This Issue

After submitting Contribution #1 (a `/health` endpoint on the Python `apps/ai-service`),
I wanted a second issue in the same repo but a different package, to show range. Issue
#46 is a `type:bug` with **written reproduction steps** and a precise, well-scoped fix
in a single file (`apps/execution-engine/src/sandbox.ts`). It scored 6/6 on my issue
checklist: I can state the problem in one sentence (Docker stream headers leak into
the output), the scope is bounded, it's learnable (the Docker multiplexed-stream format
is documented), it was open/unassigned with no linked PR, the issue links the exact
file, and the project has setup docs. It's also genuinely satisfying: a visible bug
(garbage symbols in the output panel) with a clean, testable root cause.

---

## Understanding the Issue

### Problem Description

When a code-execution task runs, the browser output panel shows **unknown replacement
symbols** (boxed question marks) at the start of each output line. Standard error is
also never shown separately from standard output.

### Expected Behavior

The output panel should display clean, printable text with no leading control bytes,
and `stdout` and `stderr` should be reported separately.

### Current (buggy) Behavior

In `apps/execution-engine/src/sandbox.ts`, containers are created **without a TTY**
(`Tty` is not set, so Docker defaults it to `false`). With no TTY, the Docker daemon
**multiplexes** `stdout` and `stderr` into one stream and prefixes **every** payload
chunk with an 8-byte frame header:

```
byte 0      : stream type (0 = stdin, 1 = stdout, 2 = stderr)
bytes 1–3   : zero padding
bytes 4–7   : payload length, big-endian uint32
bytes 8…    : payload
```

The handler read that buffer straight as text:

```ts
const logs = await container.logs({ stdout: true, stderr: true, follow: false });
const logOutput = typeof logs === "string" ? logs : logs.toString("utf-8");
// ...
return { /* ... */ stdout: logOutput, stderr: "" /* always empty */ };
```

So the 8-byte binary headers were decoded as text (the replacement symbols), and
`stderr` was hard-coded to `""` — error output was folded into `stdout` behind the
header noise.

### Affected Components

- `apps/execution-engine/src/sandbox.ts` — `executeInSandbox()` reads and returns the
  container logs; this is where the parsing must happen.
- `packages/shared-types` — `ExecutionResult` already declares both `stdout` and
  `stderr: string`, so separating them needs no type change.

---

## Reproduction Process

### Environment Setup

I cloned my fork of Tessera.io, added the upstream remote, and synced `main`
(`git fetch upstream`; the fork was even with upstream). The repo is a Turborepo
monorepo (Node ≥ 20, npm workspaces); `npm install` at the root wired up all
workspaces. The execution engine talks to the Docker socket at runtime, but the
bug — and its fix — is in **pure stream parsing**, which is reproducible and testable
without a live Docker daemon (mirroring how Contribution #1 tested the DB probe with a
fake client).

```bash
git clone https://github.com/ba-00001/Tessera.io.git && cd Tessera.io
git remote add upstream https://github.com/Kushaal-k/Tessera.io.git && git fetch upstream
npm install
```

**Working branch:** `feature/issue-46-demux-docker-logs` (in my fork, `ba-00001/Tessera.io`)

### Steps to Reproduce

1. The bug appears whenever code prints output: e.g. `print("Hello World")`.
2. With `Tty: false`, Docker returns a multiplexed buffer; a `stdout` frame for
   `Hello World\n` is `\x01\x00\x00\x00\x00\x00\x00\x0c` + `Hello World\n`.
3. The old code returns `buffer.toString("utf-8")`, so the leading `\x01` + length
   header are decoded as text and render as the boxed replacement symbols.
4. I captured this at the byte level in a unit test: `frame(1, "Hello World\n").toString("utf-8")`
   is **not** equal to `"Hello World\n"` and its first char code is `1` (the stream-type
   byte) — exactly the leaked header the issue describes.

### Reproduction Evidence

- Byte-level: a synthetic `stdout` frame decoded the old way leaks the `\x01…` header;
  decoded with the new `demuxDockerStream()` it yields a clean `{ stdout: "Hello World\n", stderr: "" }`.
- `git blame`/read of `apps/execution-engine/src/sandbox.ts` shows the raw
  `logs.toString("utf-8")` read and the hard-coded `stderr: ""`.

---

## Solution Approach

### Analysis

**Root cause:** the code treats a *framed, multiplexed* byte stream as plain UTF-8
text. The fix is to parse the frames per Docker's documented format, strip the headers,
and route payloads to `stdout`/`stderr` by stream-type byte.

### Proposed Solution

Add a pure, exported helper `demuxDockerStream(buffer): { stdout, stderr }` that walks
the 8-byte frames. Guard against a non-multiplexed (raw/TTY) buffer by falling back to
treating the bytes as `stdout`, so output is never corrupted or dropped. Wire it into
`executeInSandbox()` so the returned `ExecutionResult` carries clean, separated streams.

### Implementation Plan (UMPIRE)

**Understand:** Output panel shows garbage bytes because multiplexed Docker stream
headers are read as text; `stderr` is never separated. Fix the parsing.

**Match:** Docker's multiplexed-stream header format is documented (8-byte header,
stream type + big-endian length). The repo already tests pure functions with **vitest**
(`apps/web/src/services/aiService.test.ts`) and `ExecutionResult` already has `stdout`
and `stderr` fields — no new types needed.

**Plan:**
1. Add `demuxDockerStream(buffer: Buffer): { stdout: string; stderr: string }` in
   `sandbox.ts` — walk frames, strip headers, split by stream type, fall back to
   stdout for raw buffers, never drop bytes.
2. Replace the raw `toString("utf-8")` read in `executeInSandbox()` with the demuxer
   and return both streams.
3. Add a vitest suite from synthetic frames (no Docker), and a `test` script to the
   execution-engine package (it had none).

**Implement:** _(Phase III)_ — branch `feature/issue-46-demux-docker-logs`.

**Review:** Self-review against `CONTRIBUTING.md`: claim the issue, branch off `main`,
**sign off every commit** (`-s`), Conventional Commits, keep the diff scoped.

**Evaluate:** Unit tests assert header stripping, stdout/stderr separation, multi-frame
concatenation, multi-byte UTF-8, the raw/TTY fallback, and truncated trailing frames;
plus repo-wide `npm run test` / `typecheck` / `build`.

---

## Testing Strategy

Added `apps/execution-engine/src/sandbox.test.ts` (vitest), built from synthetic Docker
frames so it needs no live Docker. The package had no `test` script, so I added one
(`vitest run`) matching `apps/web`.

### Unit Tests (7, all passing)

- [x] Empty buffer → `{ stdout: "", stderr: "" }`.
- [x] **Reproduction + fix:** a `stdout` frame read raw leaks the header (first char
      code `1`, not equal to the clean text); `demuxDockerStream` returns clean stdout.
- [x] Interleaved `stdout`/`stderr` frames are separated correctly.
- [x] Multiple same-stream frames concatenate in order (`"a"+"b"+"c"` → `"abc"`).
- [x] Multi-byte UTF-8 payload (`"café — 日本語\n"`) is preserved.
- [x] Non-multiplexed (raw/TTY) buffer falls back to `stdout` intact.
- [x] A truncated trailing frame is not dropped.

### How to run

```bash
npm install
npm run test       # turbo: execution-engine 7/7, web 4/4
npm run typecheck  # 10/10
npm run build      # 7/7
```

---

## Implementation Notes

### What I built

| File | Change | Commit |
|------|--------|--------|
| `apps/execution-engine/src/sandbox.ts` | `demuxDockerStream()` + wire into `executeInSandbox()` (returns clean `stdout`/`stderr`) | `e9681d4` |
| `apps/execution-engine/src/sandbox.test.ts` | new vitest suite, 7 tests | `40fe40f` |
| `apps/execution-engine/package.json` | add `test` script (`vitest run`) | `40fe40f` |

Diff: 3 files, +142/−3, scoped to the issue. All commits signed off (`-s`).

### Challenges faced

- **Strict `noUncheckedIndexedAccess`.** `buffer[offset]` is typed `number | undefined`
  in this repo's TS config, so my first `typecheck` failed on the stream-type read.
  Fixed by guarding `streamType === undefined` in the same branch that handles a
  malformed/raw frame — which also documents the fallback path.
- **Workspace build ordering.** Running `tsc --noEmit` in the package before building
  `@tessera/shared-types` reported "cannot find module" for the shared types (and in
  untouched files like `worker.ts`). Building the dependency first (`npm run build
  --workspace=@tessera/shared-types`), or running `npm run typecheck` through turbo
  from the root, resolves it — turbo orders dependencies for you.
- **No Docker in CI.** Rather than depend on a live daemon, I made the demuxer a pure
  function over a `Buffer` and tested it with synthetic frames — the same "test the
  logic, fake the I/O" approach I used for the Mongo probe in Contribution #1.

### Code Changes

- **Branch:** https://github.com/ba-00001/Tessera.io/tree/feature/issue-46-demux-docker-logs
- **Artifacts:** combined diff, per-commit patches, and a git bundle preserving the SHAs
  are in [fork-artifacts/](fork-artifacts/).

---

## Pull Request

**PR Link:** [#78](https://github.com/Kushaal-k/Tessera.io/pull/78) — open against
upstream `Kushaal-k/Tessera.io:main`, from
`ba-00001/Tessera.io:feature/issue-46-demux-docker-logs`.

**PR Description:** Used the project's
[`pull_request_template.md`](https://github.com/Kushaal-k/Tessera.io/blob/main/.github/pull_request_template.md):
why-first Description (header bytes leak because the multiplexed stream is read as
text), `Type of Change: Bug fix`, How Has This Been Tested (automated + a byte-level
before/after), and the project Checklist. Linked with `Fixes #46`.

**Maintainer Feedback:**

- **June 9, 2026** — Claimed #46 via the `/claim` bot (assigned to me). Opened PR #78
  ready for review against upstream `main`, filled the project template, and left a
  comment @-mentioning @Kushaal-k with the summary and one design question (separate
  `stderr` vs. appended to `stdout`). _Awaiting first maintainer review._

**Status:** **Awaiting review** (ready-for-review, not draft; reviewer @-mentioned
June 9, 2026).

---

## Learnings & Reflections

### Technical Skills Gained

- The Docker Engine **multiplexed stream protocol** (8-byte frame header: stream type +
  big-endian length) and how to demultiplex a log `Buffer` by hand.
- Testing pure logic in a **Turborepo + vitest** TypeScript monorepo: co-located
  `*.test.ts`, ESM `.js` import specifiers, and `vitest run` per workspace.
- Why workspace **build ordering** matters (`tsc` needs `@tessera/shared-types` built
  first; turbo handles the dependency graph), and how `noUncheckedIndexedAccess`
  changes the way you read from a `Buffer`/array.

### Challenges Overcome

- Turned a vague visual bug ("boxed question marks") into a precise, byte-level
  reproduction and a unit test that pins the exact leaked header — without ever
  starting Docker.

### What I'd Do Differently Next Time

- Run `npm run typecheck` through **turbo from the repo root** from the start, instead
  of `tsc` inside one package — it builds workspace dependencies in order and avoids the
  "cannot find module" red herring I chased first.
- When a fix changes a returned shape (here, `stderr` going from always-empty to real),
  raise the behavior question in the PR up front (I did), since downstream display code
  may assume the old behavior.

---

## Resources Used

- [Issue #46](https://github.com/Kushaal-k/Tessera.io/issues/46) and [PR #78](https://github.com/Kushaal-k/Tessera.io/pull/78)
- [Docker Engine API — attach/logs stream format](https://docs.docker.com/engine/api/v1.43/#tag/Container/operation/ContainerAttach)
- [dockerode](https://github.com/apocas/dockerode)
- [Vitest](https://vitest.dev/) · [Turborepo](https://turbo.build/repo/docs)
- My Contribution #1: [#39 / PR #66](../README.md)
