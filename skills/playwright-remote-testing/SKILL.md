---
name: playwright-remote-testing
description: Create and run Playwright tests against remote environments (staging, preview deploys, production smoke) where cookie-based auth or persistent browser state is required. Use when testing a deployed app rather than a local dev server.
user-invocable: true
argument-hint: "[target environment or flow]"
---

## MANDATORY PREPARATION

Gather:

1. **Target environment URL** — staging, preview, production-read-only?
2. **Authentication method** — cookie, JWT, SSO, OAuth, API key?
3. **Is persistent state required?** — logged-in user, pre-seeded data, specific feature flag?
4. **What's allowed to mutate?** — some envs are read-only; others allow writes.
5. **What's the safety net?** — can you delete test data afterward, or is it destructive?

---

## Phase 1 — Set up the persistent browser profile

Remote tests often need persistent state across runs (session cookies, OAuth tokens, SSO state, stored preferences). Use Playwright's `launchPersistentContext`:

```js
const { chromium } = require('@playwright/test');
const path = require('path');

const USER_DATA_DIR = path.join(__dirname, '../.playwright-profile');

module.exports = {
  async launch() {
    return chromium.launchPersistentContext(USER_DATA_DIR, {
      headless: false,
      viewport: { width: 1440, height: 900 },
    });
  },
};
```

Add `.playwright-profile/` to `.gitignore`.

## Phase 2 — Bootstrap auth once

On first run, complete login manually in the persistent profile. Subsequent runs reuse the stored session.

```js
// auth-bootstrap.js — run once
const { launch } = require('./profile');
(async () => {
  const ctx = await launch();
  const page = await ctx.newPage();
  await page.goto('https://staging.example.com/login');
  // ... complete auth manually in the opened window ...
  console.log('Auth stored. Close when done.');
})();
```

For automated re-auth, use Playwright's `storageState` API:

```js
// Save after manual login
await page.context().storageState({ path: 'auth.json' });

// Reuse in tests
const browser = await chromium.launch();
const ctx = await browser.newContext({ storageState: 'auth.json' });
```

## Phase 3 — Write tests against the stored profile

```js
const { test, expect } = require('@playwright/test');

test('user can view dashboard', async ({ browser }) => {
  const ctx = await browser.newContext({ storageState: 'auth.json' });
  const page = await ctx.newPage();
  await page.goto('https://staging.example.com/dashboard');
  await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();
  await ctx.close();
});
```

## Phase 4 — Safety guardrails for remote runs

- **Never run destructive tests against production** — enforce with env-var guards:
  ```js
  if (process.env.TARGET === 'production' && testConfig.destructive) {
    throw new Error('Destructive tests blocked in production');
  }
  ```
- **Rate limit** — remote envs may throttle; add `test.describe.configure({ mode: 'serial' })` for sensitive paths.
- **Use test accounts**, never real users.
- **Clean up after yourself** — if the test creates data, it deletes it.
- **Snapshot on failure** — `page.screenshot()` in `afterEach` on fail for debugging.

## Phase 5 — Run + report

- Local run: `npx playwright test --project=remote-staging`
- CI run: headless, with stored `STORAGE_STATE` env var pointing to a pre-auth'd storage state
- Report: Playwright HTML reporter, archived per-run

## What you do NOT do

- You don't commit the persistent profile or `auth.json` to git (contains session tokens)
- You don't run tests that mutate production without explicit write-mode flags
- You don't skip the auth bootstrap because "it'll work"
- You don't use real user accounts — always test accounts with identifiable prefixes

## Reference

- `playwright.config.*` — project definitions per environment
- `.playwright-profile/` — persistent state directory (gitignored)
- Playwright storage state: https://playwright.dev/docs/auth
