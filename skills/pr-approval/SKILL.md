---
name: pr-approval
description: Given a PR that has passed QA testing, produce the structured approval comment — acceptance criteria checkboxes, test results summary table, regression watch-list, and approval/hold recommendation.
user-invocable: true
argument-hint: "[PR number]"
---

## MANDATORY PREPARATION

1. **PR number.**
2. **Test plan + test results** — from `pr-research-test-plan` / `pr-research-test-results` runs.
3. **Ticket acceptance criteria** — the source of truth for "is this done?"

---

## The approval comment structure

```
## QA Approval — PR #{n}

**Ticket:** {ticket ID — title}

**Acceptance criteria:**
- [x] {criterion 1} — verified via {spec or manual step}
- [x] {criterion 2} — ...
- [ ] {criterion 3} — **not verified because {reason}**

**Test results:**
| Surface | Coverage | Result |
|---|---|---|
| Automated (Cypress) | X specs added, Y passed | Pass |
| Manual smoke | Happy path + 3 negative paths | Pass |
| API schema | Contract verified | Pass |
| Regression sweep | {adjacent flows} | Pass |

**Regressions found:** {count} — {bug IDs, severity}
**Open bugs blocking approval:** {list or "None"}

**Approval:** Approved / Hold / Blocked
**Rationale:** {1-2 sentences}
```

## Rules

- Every acceptance criterion must be checked or have a documented reason it couldn't be verified
- If S1/S2 bugs were found, status is **Blocked** — no "approved with bugs"
- S3/S4 bugs can be logged as follow-ups without blocking approval — note them
- The approval *is* the QA sign-off — don't hedge

## What you do NOT do

- You don't approve without running the test plan first
- You don't skip acceptance criteria "because it obviously works"
- You don't merge the PR — approval is QA's role; merge is the author's or maintainer's
- You don't hand-wave regressions — each must be tracked

## Reference

- Pair with `pr-research-test-plan` + `pr-research-test-results` for the upstream data
- Pair with `jira-qa` skill to update the linked ticket status
