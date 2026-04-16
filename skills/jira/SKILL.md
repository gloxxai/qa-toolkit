---
name: jira
description: General Jira CLI reference — create, view, search, edit, transition, comment on, and manage tickets using the Atlassian CLI or REST API. Use for any Jira operation from the command line.
user-invocable: true
argument-hint: "[operation]"
---

## MANDATORY PREPARATION

1. **CLI or API?** — acli, jira-cli, or REST via curl?
2. **Auth** — API token set in env var? Token scopes correct?
3. **Site URL** — e.g., `https://{team}.atlassian.net`.
4. **Project key** — operation-dependent.

---

## Common operations

### Create

```bash
# Atlassian CLI
acli issue create --project "{KEY}" --type Bug \
  --summary "Login fails on Safari" \
  --description "See steps in comment"

# REST API
curl -X POST -u "user:token" \
  -H "Content-Type: application/json" \
  "https://{site}.atlassian.net/rest/api/3/issue" \
  -d '{"fields":{"project":{"key":"KEY"},"summary":"...","issuetype":{"name":"Bug"}}}'
```

### View

```bash
acli issue view KEY-123
# or
curl -u "user:token" "https://{site}.atlassian.net/rest/api/3/issue/KEY-123"
```

### Search (JQL)

```bash
acli issue search --jql 'project = "KEY" AND status = Open'
```

### Edit fields

```bash
acli issue edit KEY-123 --summary "New summary"
acli issue edit KEY-123 --add-label "regression"
```

### Transition status

```bash
acli issue transition KEY-123 "In Progress"
acli issue transition KEY-123 "Closed" --resolution "Fixed"
```

### Comment

```bash
acli issue comment KEY-123 "Verified on staging. Closing."
```

### Link issues

```bash
acli issue link KEY-123 KEY-456 --type "blocks"
acli issue link KEY-789 KEY-456 --type "is tested by"
```

## JQL cookbook

| Need | JQL |
|---|---|
| My open tickets | `assignee = currentUser() AND status = Open` |
| Bugs this sprint | `project = "KEY" AND type = Bug AND sprint in openSprints()` |
| Unassigned bugs | `project = "KEY" AND type = Bug AND assignee is EMPTY` |
| Recent bugs | `project = "KEY" AND type = Bug AND created >= -7d` |
| Critical open | `project = "KEY" AND priority = Highest AND status != Closed` |
| Stale | `updated <= -30d AND status != Closed` |

## Rules

- Store tokens in env vars, never hardcode
- JQL is case-insensitive for field names; some custom fields need exact match
- Test JQL in the Jira UI first, then copy to CLI — syntax errors are easier to debug visually
- Back up your JQL filters — they're team knowledge

## What you do NOT do

- You don't commit tokens
- You don't run bulk edits without a dry-run first
- You don't skip the JQL preview — bulk mistakes are hard to undo
