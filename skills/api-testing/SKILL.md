---
name: api-testing
description: Backend API testing patterns including REST, GraphQL, authentication setup, schema validation, and error-path coverage. Use when testing a backend contract directly — faster feedback than E2E, or covering edge cases E2E can't reach.
user-invocable: true
argument-hint: "[endpoint or API surface]"
---

## MANDATORY PREPARATION

Gather:
1. **API surface** — REST endpoint, GraphQL query/mutation, gRPC method?
2. **Auth** — bearer token, cookie session, API key, OAuth?
3. **Contract source** — OpenAPI spec, GraphQL schema, Postman collection, docs?
4. **Test data** — existing fixtures, or do you seed per test?
5. **Idempotency** — safe to re-run, or does it create persistent state?

---

## When to use API tests instead of E2E

| Use API tests | Use E2E (Cypress / Playwright) |
|---|---|
| Testing all error paths (500s, 401s, rate limits) | Testing user-visible composition of API calls |
| Testing data contracts (schema, types, field presence) | Validating rendering and browser behavior |
| High-volume scenarios (rate limits, pagination edges) | The question is "does the feature work" not "does the API work" |
| Feedback loop needs to be <30s | |

## Phase 1 — Orient to the API

- **REST**: `curl -i` for headers, `-v` for verbose. `jq` to parse JSON.
- **GraphQL**: POST with `{ query, variables }` body. Introspect schema if docs are thin.
- **Auth**: bearer → `-H "Authorization: Bearer $TOKEN"`, cookie → `-b cookies.txt`.

## Phase 2 — Test structure

```js
describe('POST /api/users', () => {
  it('creates a user with valid input (201)', async () => {
    const res = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${token}`)
      .send({ name: 'Test', email: 'test@example.com' });
    expect(res.status).toBe(201);
    expect(res.body).toMatchObject({ id: expect.any(String), name: 'Test' });
  });

  it('rejects missing email (400)', async () => {
    const res = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${token}`)
      .send({ name: 'Test' });
    expect(res.status).toBe(400);
    expect(res.body.error).toMatch(/email/i);
  });

  it('requires auth (401)', async () => {
    const res = await request(app).post('/api/users').send({ name: 'Test' });
    expect(res.status).toBe(401);
  });
});
```

## Phase 3 — Coverage checklist per endpoint

For every endpoint, cover:

- [ ] Happy path — valid input, expected status + response shape
- [ ] Each required field missing → validation error
- [ ] Each required field invalid type → validation error
- [ ] Unauthorized (no token) → 401
- [ ] Forbidden (wrong role) → 403
- [ ] Not found (bad ID) → 404
- [ ] Conflict (duplicate) → 409 if applicable
- [ ] Malformed input (bad JSON, wrong types) → 400
- [ ] Idempotency — same request twice produces same result

## Phase 4 — Schema validation

- Validate response against a schema (Zod, JSON Schema, AJV).
- Catches subtle drift — field type changes, renames, removals.
- Failure should include: expected schema, actual response, path of divergence.

## Phase 5 — GraphQL-specific

- Query the introspection schema periodically; diff to catch silent API changes.
- Test both queries and mutations, plus subscriptions if used.
- Check error shape (`errors` array) and data-and-errors combined responses.

## What you do NOT do

- You don't test happy path only — error paths are where API bugs hide
- You don't skip auth variations — "user hit endpoint with no token" is a real scenario
- You don't hardcode IDs that expire — use setup/teardown
- You don't commit tokens or secrets — use env vars
