# Week 4 — Phase IV: Submit & Iterate

**Course:** AI301 — AI Open Source Capstone (Summer 2026, Section 1A)
**Student:** Brian Bazurto (Member ID 76069)
**Target duration:** 3–5 days · **Project 4 due:** Mon, June 29, 2026, 2:59 AM EDT

Phase IV is the finish line: take the Phase III branch from draft to a review-ready
pull request, surface it to the maintainer, and document the feedback loop. A
review-ready PR is the program's core completion milestone — it counts whether or
not it gets merged.

**My issue:** [Kushaal-k/Tessera.io #39 — Write a JSON health endpoint for Python AI service](https://github.com/Kushaal-k/Tessera.io/issues/39)
**My PR:** [Kushaal-k/Tessera.io #66](https://github.com/Kushaal-k/Tessera.io/pull/66) (open against upstream `main`, ready for review)

## What I did this phase

No new code — Phase III already shipped a tested, lint-clean implementation matching
the Phase II plan. Phase IV was about meeting the project's submission bar and
opening the feedback loop.

| Action | Result |
|--------|--------|
| Pre-submission checks | Fix reproduces; `pytest` 5/5; diff scoped to 4 files; CI Pipeline green |
| PR description | Rewritten to the project's `pull_request_template.md` (`Fixes #39`) |
| Draft → ready | Marked PR #66 **ready for review** (was a draft) |
| Reviewer surfaced | Comment @-mentioning @Kushaal-k requesting review + restating the design question |
| README | *Pull Request* section + dated Maintainer Feedback log updated |

## Task tracker

- [x] **1. Read the Phase IV project + grading guides** — done.
- [x] **2. Final pre-submission checks** — fix still reproduces (`503` with DB down);
  `python -m pytest tests/` → 5/5; `git diff` vs upstream `main` shows only the four
  intended files (no debug code / stray changes); CI Pipeline (Lint, Typecheck,
  Test, Build) passing on the PR.
- [x] **3. Confirm the PR targets upstream `main`** — base is
  `Kushaal-k/Tessera.io:main`, head is
  `ba-00001/Tessera.io:feature/issue-39-health-endpoint` (a fork PR, not
  fork-internal, not opened from `main`).
- [x] **4. Fill in the project's PR template** — used the repo's
  [`.github/pull_request_template.md`](https://github.com/Kushaal-k/Tessera.io/blob/main/.github/pull_request_template.md):
  Description (why-first), Type of Change, How Has This Been Tested (automated +
  manual with before/after output), Checklist. Referenced the issue with `Fixes #39`.
- [x] **5. Mark the PR ready for review** — taken out of draft.
- [x] **6. Request a review** — comment posted @-mentioning @Kushaal-k (the repo
  owner / maintainer); restated the `503`-vs-always-`200` design question.
- [x] **7. Update the Contribution README** — *Pull Request* section (PR link,
  summary, **Status: Awaiting review**) and a dated Maintainer Feedback log; added a
  Week 4 Progress entry.
- [x] **8. Save submission artifacts** — final PR body + review-request comment in
  [fork-artifacts/](fork-artifacts/).
- [ ] **9. Submit the check-in form** marking **"Phase IV Complete"** in the course
  portal (link the Contribution README).
- [ ] **10. Announce in `#dts-su26-ai301-celebration`** (text drafted below).
- [ ] **11. (Ongoing) Respond to maintainer feedback within ~24h**; log each round
  (date + commit hash) in the README's Maintainer Feedback section.

## Pre-submission verification log

```text
# fix still reproduces (patched service, MongoDB down)
$ curl -s -o /dev/null -w "HTTP %{http_code}  time=%{time_total}s\n" http://127.0.0.1:8022/health
HTTP 503  time=3.04s

# tests
$ python -m pytest tests/ -q
5 passed

# lint/format
$ python -m ruff check src/ tests/        # All checks passed!
$ python -m ruff format --check src/ tests/   # already formatted

# scope — only the four intended files changed
$ git diff --stat upstream/main...feature/issue-39-health-endpoint
 apps/ai-service/src/config.py       |   ...
 apps/ai-service/src/db.py           |   ...
 apps/ai-service/src/main.py         |   ...
 apps/ai-service/tests/test_health.py|   ...
 4 files changed, +161 / -4
```

GitHub PR checks (`gh pr checks 66`): **CI Pipeline (Lint, Typecheck, Test, Build) —
pass**, triage — pass, welcome — pass.

## Ready to post

**Slack announcement for `#dts-su26-ai301-celebration`** (Step 10):

> 🚀 Phase IV Complete — PR submitted! Tessera.io #66 turns the placeholder
> `/health` route into a real check that pings MongoDB and reports `database` +
> `models` status (`200`/`503` by state), backed by a 5-test suite. Open against
> upstream `main`, ready for review, CI green.
> https://github.com/Kushaal-k/Tessera.io/pull/66

## Notes / log

- The repo has its own PR template (`Fixes #`, Type of Change, How Has This Been
  Tested, Checklist), so I filled **that** in rather than the generic program
  template — the grading guide says to use the project's template when one exists.
- The PR was first opened as a draft in Phase III (June 8); Phase IV (June 9) moved
  it to ready-for-review and surfaced it to the maintainer. The `github-actions`
  welcome bot fired on open; no human review yet.
- Submission counts as the completion milestone whether or not it merges; remaining
  open items (#9–#11) are the course-portal check-in, the celebration post, and the
  ongoing feedback loop — I'll keep the Maintainer Feedback log dated as rounds come.
