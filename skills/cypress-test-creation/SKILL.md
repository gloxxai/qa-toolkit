---
name: cypress-test-creation
description: Create new Cypress E2E test specs following the project's existing patterns, without breaking existing specs or introducing flaky selectors. Use when adding coverage for a new feature, a newly-fixed bug, or a newly-exposed regression.
user-invocable: true
argument-hint: "[feature or flow to test]"
---

## MANDATORY PREPARATION

Before writing a spec, gather:

1. **What's the user intent being tested?** Not "test the page," but "user can complete {specific action}."
2. **What's the entry state?** Logged in? Specific role? Specific data?
3. **What's the expected outcome?** What data, UI state, or side effect confirms success?
4. **What existing specs already touch this flow?** Look before writing — you may be duplicating.
5. **What's the project's convention for selectors, auth setup, and test data?** Never invent a convention when one exists.

Never write a spec without these five answers.

---

## Phase 1 — Orient to the project

1. **Find existing specs** — `find cypress/e2e -name "*.cy.*" -o -name "*.spec.*"`.
2. **Read the nearest sibling** — the spec closest in intent (same page, same feature). Copy its structure; don't reinvent.
3. **Read `cypress/support/commands.js`** — find custom commands already in use (`cy.loginByApi`, `cy.seedUser`, etc.). Use them.
4. **Read `cypress.config.*`** — learn baseUrl, env vars, timeouts, retries.
5. **Check for a test-data strategy** — fixtures, factory functions, API seeding. Prefer API over UI for setup.

## Phase 2 — Design the spec

Structure:

```js
describe('{feature}', () => {
  beforeEach(() => { /* setup — auth, navigate */ });

  it('{user intent, happy path}', () => { /* ... */ });
  it('{user intent, error path}', () => { /* ... */ });
  it('{user intent, edge case}', () => { /* ... */ });
});
```

**Selector priority** (top preferred):
1. `cy.findByRole()` / `cy.findByText()` (Testing Library)
2. `cy.get('[data-testid="..."]')`
3. `cy.get('[aria-label="..."]')`
4. `cy.contains('visible text')`
5. Avoid: CSS class selectors, `nth-child`, chained descendant selectors

**Wait strategy:**
- For network: `cy.intercept('GET', '/api/...').as('fetch')` → `cy.wait('@fetch')`
- For DOM: `cy.get('...').should('be.visible')` / `.should('exist')`
- Never: `cy.wait(2000)` as a primary fix

## Phase 3 — Write the spec

Follow the template:

```js
describe('{feature}', () => {
  beforeEach(() => {
    cy.loginByApi(userFixture);
    cy.visit('/{path}');
  });

  it('happy path: user can {action}', () => {
    cy.intercept('POST', '/api/{endpoint}').as('submit');
    cy.findByRole('textbox', { name: /name/i }).type('Test');
    cy.findByRole('button', { name: /submit/i }).click();
    cy.wait('@submit').its('response.statusCode').should('eq', 200);
    cy.findByText(/success/i).should('be.visible');
  });
});
```

## Phase 4 — Run + verify

1. **Run the spec in isolation** — `npx cypress run --spec <path>` — confirm green.
2. **Run it 3x in sequence** — confirm stable, not flaky on re-run.
3. **Run adjacent specs after** — confirm the new spec didn't leak state.
4. **Run the full suite in CI config** — confirm it doesn't interact poorly with parallel execution.

## Phase 5 — Document + ship

- One-line comment at the top: `// Covers: {user intent summary}`
- No dead code, no `.only`, no `.skip`, no `cy.log('debugging')`
- Self-review against the anti-patterns list below

## Anti-patterns to avoid

| Anti-pattern | Why bad | Do instead |
|---|---|---|
| `cy.wait(2000)` | Flaky, slow, masks real timing issues | Wait on network alias or DOM assertion |
| `.click({force: true})` | Hides accessibility / visibility bugs | Fix the underlying "why isn't this clickable" issue |
| CSS class selectors | Break on style refactors | `data-testid` or role |
| UI login on every spec | Slow + brittle | `cy.loginByApi()` via session |
| Shared state between `it()` blocks | Order-dependent failures | Reset in `beforeEach` |
| `.only` or `.skip` committed | Breaks the suite in CI | Lint rule or pre-commit hook |
| Assertion on internal state | Tests implementation, not behavior | Assert on visible output |

## What you do NOT do

- You don't skip the project-convention orientation (Phase 1)
- You don't write a spec you haven't run
- You don't introduce a new selector convention when one exists
- You don't silently add `cy.wait(ms)` to fix flake — diagnose the cause
