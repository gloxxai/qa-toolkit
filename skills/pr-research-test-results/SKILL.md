---
name: pr-research-test-results
description: Execute a test plan against a PR, iterate on failures, and produce a final test results report suitable for posting on the PR or ticket. Use after pr-research-test-plan has produced the plan.
user-invocable: true
argument-hint: "[PR or test plan reference]"
---

## MANDATORY PREPARATION

1. **What's being tested** — PR number, test plan doc, or ad-hoc?
2. **Test target** — local branch, deployed preview, staging?
3. **Which specs / manual steps** — the list from `pr-research-test-plan`?
4. **Reporting destination** — PR comment, ticket, Confluence page?

---

## Phase 1 — Set up

- Check out the PR branch locally, or identify the preview deploy
- Install deps, verify the app starts
- Run any test-data setup the plan requires

## Phase 2 — Execute

For each item in the plan:

- **Automated specs:** `npx cypress run --spec <path>` or `npx playwright test <path>`. Capture HTML report + screenshots on fail.
- **Manual steps:** execute, capture evidence (screenshot, console log, network trace). Note actual vs expected.
- **API checks:** run via curl / test framework, capture status + body.

On failure:
1. Repeat once to confirm not transient
2. Diagnose: real regression vs flaky vs environmental
3. Real regression → file bug (use `bug-triage` skill), link on PR, mark **Fail**
4. Flaky → note it, mark **Pass (flaky)**, file flake ticket separately
5. Environmental → fix env, re-run

## Phase 3 — Report

```
## Test Results — PR #{n}

**Environment:** {local / preview / staging}
**Executed by:** {name}
**Date:** {date}

**Automated results:**
| Spec | Pass / Fail | Notes |
|---|---|---|
| checkout.spec.js | Pass | 12/12 tests, 0 flakes |
| auth.spec.js | Fail | 3/5 tests — see bug #X |

**Manual results:**
| # | Step | Pass / Fail | Notes |
|---|---|---|---|
| 1 | Happy path checkout | Pass | — |
| 2 | Empty cart error | Fail | Error says "undefined" — bug #Y |

**Regressions found:** {count} — {linked bugs}
**Blockers:** {list or "None"}

**Recommendation:** Merge / Hold / Return to dev
```

## What you do NOT do

- You don't mark failing tests as pass without investigating
- You don't skip the re-run on failure — always confirm before blaming flake
- You don't silently retry until green — report the flake honestly
- You don't file bugs without triaging severity first
