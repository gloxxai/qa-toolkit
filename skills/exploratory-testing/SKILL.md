---
name: exploratory-testing
description: Structured exploratory testing sessions to find bugs and edge cases beyond scripted tests. Use when a new feature ships, when automation coverage is solid but bugs are leaking, or when you want to discover what the test plan missed.
user-invocable: true
argument-hint: "[feature or area]"
---

## MANDATORY PREPARATION

1. **Feature / area under test** — what's the surface?
2. **Time box** — 30 / 60 / 90 / 120 minutes?
3. **Persona** — which user type? (new user, power user, adversarial, mobile, slow network)
4. **Charter** — one-sentence mission: "Explore {X} as a {Y} user to find {Z} types of issues."

---

## The session-based testing model

Exploratory testing isn't aimless clicking. It's a structured session with a charter, time box, and notes.

## Phase 1 — Charter

One sentence that constrains the session:

- "Explore the checkout flow as a new mobile user to find UX issues."
- Too broad ("explore the app") = aimless. Too narrow ("click submit with empty email") = scripted.

## Phase 2 — Run the session (time-boxed)

Keep a running log:

- **Observations** — what the app does
- **Issues** — bugs, usability problems, unclear states
- **Questions** — "what should happen when...?" (design gaps)
- **Tests to automate later** — automation candidates are born here

Tactics to try:
- Boundary values (empty, very long, zero, negative, huge)
- Wrong types (numbers in name fields, emoji in required text)
- Concurrency (two tabs, race to submit)
- Networking (offline, slow, mid-request disconnect)
- Permissions (logged out, wrong role, session expired)
- Browser controls (back button mid-flow, refresh mid-submit, deep-link)
- Accessibility (keyboard only, screen reader, zoom 200%)

## Phase 3 — Report

```
## Exploratory Session — {feature} — {date}

**Charter:** {mission sentence}
**Persona:** {user type}
**Time:** {start — end, total}

**Issues found:**
1. {summary} — severity S{1-4} — #{bug-link}
2. ...

**UX / design concerns (not bugs, but noted):**
1. ...

**Questions raised (need design / product input):**
1. ...

**Automation candidates:**
1. {edge case} → {test framework + spec idea}

**Areas not explored this session:** {list — next session's charter}
```

## Rules

- Bugs found → file immediately (use `bug-triage` skill)
- Exploratory is *not* a replacement for automation — it's the other half of the pair
- Time-box strictly; sessions over 2 hours lose focus
- Note what you didn't explore — that's the next charter

## What you do NOT do

- You don't explore without a charter (aimless clicking is not exploratory)
- You don't write automation during the session — capture candidates, automate after
- You don't skip reporting — the session's value is in the notes
