---
name: xray
description: Create and manage Xray (Jira test-management plugin) test cases, test steps, test sets, test plans, and test executions via the Xray Cloud GraphQL API. Use when your team tracks test cases in Xray.
user-invocable: true
argument-hint: "[operation: create-test | add-execution | link-to-plan]"
---

## MANDATORY PREPARATION

1. **Xray Cloud GraphQL endpoint** — `https://xray.cloud.getxray.app/api/v2/graphql`
2. **Auth** — client ID + client secret → bearer token (1-hour expiry)
3. **Project key** — the Jira project Xray is scoped to
4. **Entity type** — Test / Test Set / Test Plan / Test Execution?

---

## Auth flow

```bash
TOKEN=$(curl -s -X POST https://xray.cloud.getxray.app/api/v2/authenticate \
  -H "Content-Type: application/json" \
  -d '{"client_id":"'"$XRAY_CLIENT"'","client_secret":"'"$XRAY_SECRET"'"}' \
  | tr -d '"')
```

Token expires in ~1 hour; re-auth as needed.

## Core entities

- **Test** — a single test case (steps or generic description)
- **Test Set** — a named collection of tests (regression suite, smoke suite)
- **Test Plan** — scope of testing for a release; tracks progress
- **Test Execution** — a specific run of a set/plan, with pass/fail per test

## Common operations

### Create a Test (with steps)

```graphql
mutation {
  createTest(
    testType: { name: "Manual" }
    steps: [
      { action: "Navigate to login", result: "Login page shown" }
      { action: "Enter valid credentials", result: "Dashboard shown" }
    ]
    jira: { fields: { summary: "Login happy path", project: { key: "KEY" } } }
  ) {
    test { issueId jira(fields: ["key"]) }
    warnings
  }
}
```

### Create a Test Execution

```graphql
mutation {
  createTestExecution(
    testIssueIds: ["1001", "1002"]
    jira: { fields: { summary: "Release 2.3 smoke", project: { key: "KEY" } } }
  ) {
    testExecution { issueId }
    warnings
  }
}
```

### Update execution results

Each test in an execution gets a status: PASS / FAIL / EXECUTING / TODO / ABORTED.

```graphql
mutation {
  updateTestRunStatus(id: "{runId}", status: "PASSED") {
    warnings
  }
}
```

### Query — all tests in a test set

```graphql
{
  getTestSet(issueId: "TESTSET-123") {
    tests(limit: 100) {
      results { issueId jira(fields: ["key", "summary"]) }
    }
  }
}
```

## Rules

- Tokens are short-lived — cache the token + refresh on 401
- Use Test Plans to track release-scoped progress; use Test Sets for stable suites that outlast a release
- For automated tests imported via CI, prefer `importExecution` mutation (bulk) over per-test updates
- Deduplicate: search by summary before creating new tests

## What you do NOT do

- You don't duplicate test cases that already exist — search first
- You don't tie test IDs to hardcoded issue keys in test code — map via labels or Xray IDs
- You don't re-auth per request — cache the token
