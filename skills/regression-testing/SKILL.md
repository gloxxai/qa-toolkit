---
name: regression-testing
description: Plan and execute regression test cycles before releases, large merges, or infrastructure changes. Use when multiple changes are shipping and you need confidence nothing that worked yesterday is broken today.
user-invocable: true
argument-hint: "[release or change window]"
---

## MANDATORY PREPARATION

1. **The change window** — which PRs / deploys / infra changes are in scope?
2. **Release criticality** — blocker or rolling out to a subset?
3. **Capacity** — how many hours/days of QA time available?
4. **Automation state** — what's covered by specs vs needs manual?

---

## Phase 1 — Scope the regression set

1. List every change in the window.
2. For each change, list the flows it could affect (directly touched + callers + data consumers).
3. Union + dedup → your regression surface.
4. Rank by blast radius — top of list runs first.

## Phase 2 — Assign coverage

| Flow | Automation coverage | Manual needed? |
|---|---|---|
| {flow} | {spec name} | Smoke only / Full manual |
| {new flow} | None | Full manual pass |

## Phase 3 — Execute

- Automated specs: run in CI, collect results
- Manual: execute per plan, capture evidence
- Log all failures (flakes included) for retrospective

## Phase 4 — Report

```
## Regression Report — {window}

**Changes in scope:** {count} PRs, {count} infra changes
**Regression surface:** {count} flows — {automated} + {manual}

**Results:**
- Automated: X/Y specs green, Z flakes (filed)
- Manual: A/B steps passed, C failed (bugs filed)

**Regressions found:** {count}
**Go / no-go:** Go / Hold / {conditions}
```

## Rules

- "Full regression" is unsustainable — always risk-rank and skip low-blast surface
- Flakes get filed as bugs, not ignored
- Dependencies ship + tests run after — never assume a dep bump is invisible
- Sequence merges: regression after each, not only at the end

## What you do NOT do

- You don't regress the entire product for a small PR
- You don't declare "go" if any S1/S2 regressions are open
- You don't treat flake as pass — re-run, diagnose, decide
