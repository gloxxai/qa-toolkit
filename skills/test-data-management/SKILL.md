---
name: test-data-management
description: Create, seed, reset, and tear down test data (users, accounts, records, fixtures) for QA workflows. Use when tests fail because data is stale, missing, or conflicting — or when building a reliable per-run data strategy.
user-invocable: true
argument-hint: "[data type: users | accounts | fixtures | reset]"
---

## MANDATORY PREPARATION

1. **Data type** — user, account, product, record, full environment?
2. **Where it lives** — DB, API, JSON fixtures, factory functions?
3. **Idempotency need** — must re-run produce identical state?
4. **Teardown strategy** — per-test, per-run, or manual?

---

## The four strategies

### 1. Static fixtures (simplest)

- JSON / YAML files committed to the repo (`fixtures/`)
- Best for: read-only data, UI snapshot tests, API contract tests
- Worst for: stateful flows (checkout, sign-up, anything that mutates)

### 2. API seeding (most reliable)

- Test setup calls the backend's create endpoints via a service/admin token
- Best for: anything the UI would take too long to create
- Pattern: `cy.loginByApi(role, email)` custom command; `request(app).post('/api/seed')`

### 3. Database seed scripts

- SQL or migration-style scripts run before the suite
- Best for: large, structurally complex seed data (product catalog, permission grids)
- Watch for: drift between seed script and production schema

### 4. Factory functions

- Code that generates a consistent-but-unique record per call
- Best for: tests that need "a user" without caring about identity
- Pattern: `const user = factory.user({ role: 'admin' })`

## Uniqueness

Every test run needs unique data to avoid conflicts:

```js
const email = `test-${Date.now()}-${Math.random().toString(36).slice(2)}@example.com`;
```

Or use a test-data library (Faker.js) with a seed for reproducibility.

## Teardown

Three patterns:

- **Per-test cleanup** — each test deletes its own data (cleanest, slowest)
- **Per-run cleanup** — a `globalTeardown` hook wipes test-scoped data at the end
- **Isolated env** — tests run against an ephemeral env that's fully reset between runs (Docker, preview deploys)

Choose based on how much cross-test pollution you can tolerate.

## Rules

- Never use production data in tests — use seeded or factory'd subsets
- Every test user has an obvious prefix (`qa-`, `test-`) so cleanup is safe
- If a test depends on specific data, that data is created IN the test, not assumed present
- Reset strategy documented in the test README

## What you do NOT do

- You don't hardcode production-looking user emails or account numbers
- You don't leave test data in production (use prefix conventions)
- You don't assume shared state across tests (unless explicitly sharing fixtures)
- You don't delete data blindly — confirm it's test-scoped before wiping
