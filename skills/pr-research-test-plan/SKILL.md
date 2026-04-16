---
name: pr-research-test-plan
description: Given a PR number or ticket link, research the change in depth and produce a manual test plan with impact analysis and automation recommendations. Use when a ticket or PR needs a detailed test plan before QA engages.
user-invocable: true
argument-hint: "[PR number or ticket link]"
---

## MANDATORY PREPARATION

1. **The PR number or ticket link.**
2. **Related tickets / PRs** — parent epic, linked bugs, dependent work.
3. **Access method** — `gh` CLI available? Ticket tracker CLI available?
4. **Goal** — manual plan only, automation recs, or both?

---

## Phase 1 — Gather

- `gh pr view <n>` + `gh pr diff <n>` for the PR
- Read the linked ticket in full (description, acceptance criteria, comments)
- Identify all files changed and group by layer (UI / API / data / infra)
- Find existing test specs that cover the changed surface (`grep -r` in test dirs)

## Phase 2 — Analyze

- What *behaviors* changed, not just what files?
- What's the blast radius? (Use the `pr-risk-reviewer` agent's rubric if needed)
- Which existing test specs cover vs miss this change?
- What's newly uncovered?

## Phase 3 — Produce the plan

```
## Test Plan — PR #{n} / {ticket}

**Summary of change:** {1-2 sentences — what the user can now do, or what bug was fixed}
**Blast radius:** S{1-4}
**Files changed (by layer):**
- UI: ...
- API: ...
- Data: ...

**What to verify manually:**
| # | Step | Expected result |
|---|---|---|
| 1 | ... | ... |

**Negative paths:**
- {condition → expected error}

**Regression watch:**
- {existing flow adjacent to this change}

**Automation recommendations:**
- [ ] {concrete spec to add — framework, file path, scenario}
- [ ] ...

**Sign-off criteria:**
- [ ] All manual steps pass
- [ ] All regression checks pass
- [ ] Automation coverage added for priority items
```

## What you do NOT do

- You don't write the automation specs — delegate to `cypress-test-creation` or `api-testing`
- You don't approve the PR — that's the `pr-approval` skill
- You don't skip reading the diff because "the title is clear"
- You don't produce plans longer than 2 pages — break into sub-plans

## Reference

- Pair with `pr-risk-reviewer` agent for blast-radius scoring
- Pair with `pr-research-test-results` skill after execution
