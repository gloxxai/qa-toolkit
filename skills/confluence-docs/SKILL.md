---
name: confluence-docs
description: Create, update, and maintain Confluence documentation for epics, features, QA processes, and runbooks. Use when writing up a test plan, documenting a feature for QA handoff, or publishing a process doc the team will reference.
user-invocable: true
argument-hint: "[operation: create | update | publish-test-plan]"
---

## MANDATORY PREPARATION

1. **Space key** — which Confluence space?
2. **Parent page** — where in the page tree does this go?
3. **Audience** — QA team, dev team, product, exec?
4. **Doc type** — test plan, runbook, process, retrospective?

---

## Doc types + templates

### Test plan (per feature or epic)

```
# Test Plan — {feature / epic}

## Scope
What this plan covers. What it doesn't.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Test strategy
Automated vs manual split. Test data approach. Environments.

## Test cases
| ID | Description | Steps | Expected | Priority |
|---|---|---|---|---|
| TC-01 | ... | ... | ... | High |

## Risks + assumptions
- ...

## Sign-off
- [ ] Product · [ ] Dev · [ ] QA
```

### Runbook (operational procedure)

```
# Runbook — {procedure}

## When to use
Trigger conditions. Preconditions.

## Steps
1. ...

## Verification
How to confirm success.

## Rollback
If it goes wrong.

## Contacts
On-call, escalation path.
```

### Retrospective (post-release or post-incident)

```
# Retro — {release or incident}

## What happened
Timeline, events, impact.

## What went well
- ...

## What didn't
- ...

## Action items
- [ ] Owner — action — due date
```

## Creation via REST API

```bash
curl -X POST -u "user:token" \
  -H "Content-Type: application/json" \
  "https://{site}.atlassian.net/wiki/rest/api/content" \
  -d '{
    "type": "page",
    "title": "{title}",
    "space": {"key": "{SPACEKEY}"},
    "ancestors": [{"id": "{parentPageId}"}],
    "body": {"storage": {"value": "<h1>Content</h1>", "representation": "storage"}}
  }'
```

## Rules

- Every doc has an owner + last-reviewed date
- Docs not reviewed in >6 months get a staleness banner
- Don't duplicate content — link to the source of truth
- For QA test plans: version per release, archive old versions

## What you do NOT do

- You don't scatter content across multiple pages when one would do
- You don't leave draft pages unpublished for weeks
- You don't skip sign-off checkboxes on test plans (they're the audit trail)
- You don't use Confluence as a tracker — tickets belong in Jira

## Reference

- Pair with `jira-qa` skill to link Confluence plans to Jira tickets
- Pair with `pr-research-test-plan` skill to generate content, then publish via this skill
