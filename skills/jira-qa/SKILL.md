---
name: jira-qa
description: Centralized skill for QA-specific Jira operations — filing bugs with the right fields, creating test-case tickets, linking tests to stories, searching the QA ticket queue. Use when the Jira interaction is QA-driven.
user-invocable: true
argument-hint: "[operation: file-bug | search | link | create-test]"
---

## MANDATORY PREPARATION

1. **Project key** — which Jira project?
2. **Issue type** — Bug, Test Case, Task?
3. **Auth** — Jira CLI authenticated? Token in env var?
4. **Required custom fields** — does this team require severity, sprint, component?

---

## Operations

### File a bug

Required fields at minimum:
- **Summary** (one-line title)
- **Description** (markdown: repro, expected, actual, env, evidence)
- **Severity** (custom field; S1–S4)
- **Component** (which part of the product)
- **Reporter** (you)
- **Priority** (maps from severity)

Pair with the `bug-triage` skill for severity/impact classification first.

### Search the QA queue

Common JQL patterns:

| Need | JQL |
|---|---|
| Open bugs assigned to me | `project = "{KEY}" AND type = Bug AND status in ("Open", "In Progress") AND assignee = currentUser()` |
| Bugs filed this week | `project = "{KEY}" AND type = Bug AND created >= -7d` |
| Regression candidates | `project = "{KEY}" AND type = Bug AND labels = "regression"` |
| Stale (30+ days no update) | `project = "{KEY}" AND type = Bug AND updated <= -30d AND status != Closed` |

### Create a test-case ticket

For teams tracking test cases in Jira:
- Type: Test Case (or equivalent)
- Linked to: parent Story or Epic
- Pre-conditions, steps, expected result populated
- Owner: the QA engineer who'll execute it

### Link a test to a story

Use "tests" / "is tested by" link type. Ensures traceability from story → test case → execution.

## Field hygiene rules

- Always populate severity + impact — bugs without these are unactionable
- Use components, not freeform labels, when the team has them
- Don't re-file dupes — search first
- Close-with-reason: fixed / won't fix / not reproducible / duplicate / expected behavior

## What you do NOT do

- You don't file bugs without triaging severity first
- You don't set severity sentimentally ("it annoyed me" ≠ S1)
- You don't skip the search-for-dupes step
- You don't leave old tickets stale — update, close, or escalate

## Reference

- Pair with `bug-triage` skill before filing
- Pair with `jira` skill for generic Jira CLI operations
- Pair with `xray` skill if team uses Xray for test management
